# Pod Topology Spread Constraints

You can use __topology spread constraints__ to control how Pods are spread across your cluster among failure-domains such as regions, zones, nodes, and other user-defined topology domains. This can help to achieve high availability as well as efficient resource utilization.

## Prerequisites

#### Enable Feature Gate
The `EvenPodsSpread` feature gate must be enabled for the API Server and scheduler.

#### Node Labels

Topology spread constraints rely on node labels to identify the topology domain(s) that each Node is in. For example, a Node might have labels: `node=node1,zone=us-east-1a,region=us-east-1`

Suppose you have a 4-node cluster with the following labels:

NAME    STATUS   ROLES    AGE     VERSION   LABELS
node1   Ready    <none>   4m26s   v1.16.0   node=node1,zone=zoneA
node2   Ready    <none>   3m58s   v1.16.0   node=node2,zone=zoneA
node3   Ready    <none>   3m17s   v1.16.0   node=node3,zone=zoneB
node4   Ready    <none>   2m43s   v1.16.0   node=node4,zone=zoneB
Then the cluster is logically viewed as below:

+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+

## Spread Constraints for Pods

#### API

The field `pod.spec.topologySpreadConstraints` is introduced in 1.16 as below:

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  topologySpreadConstraints:
    - maxSkew: <integer>
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
```

You can define one or multiple `topologySpreadConstraint` to instruct the kube-scheduler how to place each incoming Pod in relation to the existing Pods across your cluster. The fields are:

- **maxSkew** 
    - It describes the degree to which Pods may be unevenly distributed. 
    - It’s the maximum permitted difference between the number of matching Pods in any two topology domains of a given topology type. 
    - It must be greater than zero.
- **topologyKey** 
    - It is the key of node labels. 
    - If two Nodes are labelled with this key and have identical values for that label, the scheduler treats both Nodes as being in the same topology. 
    - The scheduler tries to place a balanced number of Pods into each topology domain.
- **whenUnsatisfiable** 
    - It indicates how to deal with a Pod if it doesn’t satisfy the spread constraint:
        - `DoNotSchedule` (default) tells the scheduler not to schedule it.
        - `ScheduleAnyway` tells the scheduler to still schedule it while prioritizing nodes that minimize the skew.
- **labelSelector** is used to find matching Pods. Pods that match this label selector are counted to determine the number of Pods in their corresponding topology domain.

You can read more about this field by running 

```
kubectl explain Pod.spec.topologySpreadConstraints
```

## Example: One TopologySpreadConstraint

Suppose you have a 4-node cluster where 3 Pods labeled `foo:bar` are located in node1, node2 and node3 respectively (P represents Pod):

+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
|   P   |   P   |   P   |       |
+-------+-------+-------+-------+

If we want an incoming Pod to be evenly spread with existing Pods across zones, the spec can be given as:

```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
  labels:
    foo: bar
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        foo: bar
  containers:
  - name: pause
    image: k8s.gcr.io/pause:3.1
```

- `topologyKey`: `zone` implies the even distribution will only be applied to the nodes which have label pair “zone:” present. 
- `whenUnsatisfiable`: `DoNotSchedule` tells the scheduler to let it stay pending if the incoming Pod can’t satisfy the constraint.

If the scheduler placed this incoming Pod into `zoneA`, the Pods distribution would become [3, 1], hence the actual skew is 2 (3 - 1) - which violates `maxSkew: 1`. In this example, the incoming Pod can only be placed onto `zoneB`:

+---------------+---------------+      +---------------+---------------+
|     zoneA     |     zoneB     |      |     zoneA     |     zoneB     |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |  OR  | node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
|   P   |   P   |   P   |   P   |      |   P   |   P   |  P P  |       |
+-------+-------+-------+-------+      +-------+-------+-------+-------+

You can tweak the Pod spec to meet various kinds of requirements:

- Change `maxSkew` to a bigger value like “2” so that the incoming Pod can be placed onto “zoneA” as well.
- Change `topologyKey` to “node” so as to distribute the Pods evenly across nodes instead of zones. In the above example, if maxSkew remains “1”, the incoming Pod can only be placed onto “node4”.
- Change `whenUnsatisfiable: DoNotSchedule` to `whenUnsatisfiable: ScheduleAnyway` to ensure the incoming Pod to be always schedulable (suppose other scheduling APIs are satisfied). However, it’s preferred to be placed onto the topology domain which has fewer matching Pods. (Be aware that this preferability is jointly normalized with other internal scheduling priorities like resource usage ratio, etc.)

## Example: Multiple TopologySpreadConstraints

This builds upon the previous example. Suppose you have a 4-node cluster where 3 Pods labeled foo:bar are located in node1, node2 and node3 respectively (P represents Pod):

+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
|   P   |   P   |   P   |       |
+-------+-------+-------+-------+

You can use 2 `TopologySpreadConstraints` to control the Pods spreading on both zone and node:

```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
  labels:
    foo: bar
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        foo: bar
  - maxSkew: 1
    topologyKey: node
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        foo: bar
  containers:
  - name: pause
    image: k8s.gcr.io/pause:3.1
```

In this case, to match the first constraint, the incoming Pod can only be placed onto “zoneB”; while in terms of the second constraint, the incoming Pod can only be placed onto “node4”. Then the results of 2 constraints are ANDed, so the only viable option is to place on “node4”.

Multiple constraints can lead to conflicts. Suppose you have a 3-node cluster across 2 zones:

+---------------+-------+
|     zoneA     | zoneB |
+-------+-------+-------+
| node1 | node2 | node3 |
+-------+-------+-------+
|  P P  |   P   |  P P  |
+-------+-------+-------+

If you apply above to this cluster, you will notice “mypod” stays in Pending state. This is because: to satisfy the first constraint, “mypod” can only be put to “zoneB”; while in terms of the second constraint, “mypod” can only put to “node2”. Then a joint result of “zoneB” and “node2” returns nothing.

To overcome this situation, you can either increase the `maxSkew` or modify one of the constraints to use `whenUnsatisfiable: ScheduleAnyway`