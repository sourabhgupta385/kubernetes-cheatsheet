# Label selectors

Unlike names and UIDs, labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

Via a label selector, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.

The API currently supports two types of selectors: 
- equality-based
- set-based. 

A label selector can be made of multiple requirements which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical AND (&&) operator.

## Equality-based requirement

- Equality- or inequality-based requirements allow filtering by label keys and values. 
- Matching objects must satisfy all of the specified label constraints, though they may have additional labels as well. 
- Three kinds of operators are admitted 
  - =
  - ==
  - !=
  - The first two represent equality (and are simply synonyms), while the latter represents inequality. 

For example:

```
environment = production
tier != frontend
```

The former selects all resources with key equal to environment and value equal to production. The latter selects all resources with key equal to tier and value distinct from frontend, and all resources with no labels with the tier key. One could filter for resources in production excluding frontend using the comma operator: environment=production,tier!=frontend

One usage scenario for equality-based label requirement is for Pods to specify node selection criteria. For example, the sample Pod below selects nodes with the label “accelerator=nvidia-tesla-p100”.

```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

## Set-based requirement

- Set-based label requirements allow filtering keys according to a set of values. 
- Three kinds of operators are supported: 
  - in
  - notin
  - xists (only the key identifier). 

For example:

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

The first example selects all resources with key equal to environment and value equal to production or qa. The second example selects all resources with key equal to tier and values other than frontend and backend, and all resources with no labels with the tier key. The third example selects all resources including a label with key partition; no values are checked. The fourth example selects all resources without a label with key partition; no values are checked. Similarly the comma separator acts as an AND operator. So filtering resources with a partition key (no matter the value) and with environment different than  qa can be achieved using partition,environment notin (qa). The set-based label selector is a general form of equality since environment=production is equivalent to environment in (production); similarly for != and notin.

Set-based requirements can be mixed with equality-based requirements. For example: partition in (customerA, customerB),environment!=qa.

## Resources that support equity-based requirements

- service 
- replicationcontroller

## Resources that support set-based requirements

- Job
- Deployment
- ReplicaSet
- DaemonSet

```
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}    
```

matchLabels is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is “key”, the operator is “In”, and the values array contains only “value”. matchExpressions is a list of pod selector requirements. Valid operators include In, NotIn, Exists, and DoesNotExist. The values set must be non-empty in the case of In and NotIn. All of the requirements, from both matchLabels and matchExpressions are ANDed together -- they must all be satisfied in order to match.