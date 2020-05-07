# Disruptions

Pods do not disappear until someone (a person or a controller) destroys them, or there is an unavoidable hardware or system software error.

There can be two types of disruptions:

- **Voluntary**: When there is an unavoidable hardware or system software error. Some Examples:
    - a hardware failure of the physical machine backing the node
    - cluster administrator deletes VM (instance) by mistake
    - cloud provider or hypervisor failure makes VM disappear
    - a kernel panic
    - the node disappears from the cluster due to cluster network partition
    - eviction of a pod due to the node being out-of-resources.
- **Involuntary**: These include both actions initiated by the application owner and those initiated by a Cluster Administrator. Typical application owner and Cluster Administrator actions include:
    - deleting the deployment or other controller that manages the pod
    - updating a deployment’s pod template causing a restart
    - directly deleting a pod (e.g. by accident)
    - Draining a node for repair or upgrade.
    - Draining a node from a cluster to scale the cluster down (learn about Cluster Autoscaling ).
    - Removing a pod from a node to permit something else to fit on that node.

## Dealing with Disruptions

Here are some ways to mitigate involuntary disruptions:

- Ensure your pod requests the resources it needs.
- Replicate your application if you need higher availability. (Learn about running replicated stateless and stateful applications.)
- For even higher availability when running replicated applications, spread applications across racks (using anti-affinity) or across zones (if using a multi-zone cluster.)

Kubernetes offers features to help run highly available applications at the same time as frequent voluntary disruptions. We call this set of features **__Disruption Budgets__**.

## How Disruption Budgets Work

An Application Owner can create a `PodDisruptionBudget` object (PDB) for each application. A PDB limits the number of pods of a replicated application that are down simultaneously from voluntary disruptions.

A PDB specifies the number of replicas that an application can tolerate having, relative to how many it is intended to have. For example, a Deployment which has a .spec.replicas: 5 is supposed to have 5 pods at any given time. If its PDB allows for there to be 4 at a time, then the Eviction API will allow voluntary disruption of one, but not two pods, at a time.

## PDB Example

Consider a cluster with 3 nodes, node-1 through node-3. The cluster is running several applications. One of them has 3 replicas initially called pod-a, pod-b, and pod-c. Another, unrelated pod without a PDB, called pod-x, is also shown. Initially, the pods are laid out as follows:

node-1          | node-2          | node-3
pod-a available | pod-b available |	pod-c available
pod-x available	|                 |

All 3 pods are part of a deployment, and they collectively have a PDB which requires there be at least 2 of the 3 pods to be available at all times.

For example, assume the cluster administrator wants to reboot into a new kernel version to fix a bug in the kernel. The cluster administrator first tries to drain node-1 using the `kubectl drain` command. That tool tries to evict pod-a and pod-x. This succeeds immediately. Both pods go into the terminating state at the same time. This puts the cluster in this state:

node-1 draining   |	node-2          | node-3
pod-a terminating |	pod-b available	| pod-c available
pod-x terminating |		            |

The deployment notices that one of the pods is terminating, so it creates a replacement called pod-d. Since node-1 is cordoned, it lands on another node. Something has also created pod-y as a replacement for pod-x.

(Note: for a StatefulSet, pod-a, which would be called something like pod-0, would need to terminate completely before its replacement, which is also called pod-0 but has a different UID, could be created. Otherwise, the example applies to a StatefulSet as well.)

Now the cluster is in this state:

node-1 draining   |	node-2          | node-3
pod-a terminating |	pod-b available	| pod-c available
pod-x terminating |	pod-d starting	| pod-y

At some point, the pods terminate, and the cluster looks like this:

node-1 drained   | node-2          | node-3
                 | pod-b available | pod-c available
            	 | pod-d starting  | pod-y          

At this point, if an impatient cluster administrator tries to drain node-2 or node-3, the drain command will block, because there are only 2 available pods for the deployment, and its PDB requires at least 2. After some time passes, pod-d becomes available.

The cluster state now looks like this:

node-1 drained	 | node-2          | node-3
            	 | pod-b available | pod-c available
	             | pod-d available | pod-y

Now, the cluster administrator tries to drain node-2. The drain command will try to evict the two pods in some order, say pod-b first and then pod-d. It will succeed at evicting pod-b. But, when it tries to evict pod-d, it will be refused because that would leave only one pod available for the deployment.

The deployment creates a replacement for pod-b called pod-e. Because there are not enough resources in the cluster to schedule pod-e the drain will again block. The cluster may end up in this state:

node-1 drained	 | node-2	       | node-3	          | no node
	             | pod-b available | pod-c available  | pod-e pending
	             | pod-d available | pod-y	          |

At this point, the cluster administrator needs to add a node back to the cluster to proceed with the upgrade.

You can see how Kubernetes varies the rate at which disruptions can happen, according to:

- how many replicas an application needs
- how long it takes to gracefully shutdown an instance
- how long it takes a new instance to start up
- the type of controller
- the cluster’s resource capacity