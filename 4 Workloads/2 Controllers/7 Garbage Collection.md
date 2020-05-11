# Garbage Collection

The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.

## Owners and dependents

Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called dependents of the owner object. Every dependent object has a metadata.ownerReferences field that points to the owning object.

Sometimes, Kubernetes sets the value of ownerReference automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the ownerReference field of each Pod in the ReplicaSet. In 1.8, Kubernetes automatically sets the value of ownerReference for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job and CronJob.

You can also specify relationships between owners and dependents by manually setting the ownerReference field.

## Controlling how the garbage collector deletes dependents

- When you delete an object, you can specify whether the object’s dependents are also deleted automatically. 
- Deleting dependents automatically is called `cascading deletion`. 
- There are two modes of cascading deletion: `background and foreground`.

If you delete an object without deleting its dependents automatically, the dependents are said to be `orphaned`.

#### Foreground cascading deletion

In foreground cascading deletion, the root object first enters a “deletion in progress” state. In the “deletion in progress” state, the following things are true:

- The object is still visible via the REST API
- The object’s `deletionTimestamp` is set
- The object’s `metadata.finalizers` contains the value “foregroundDeletion”.

Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents. Once the garbage collector has deleted all “blocking” dependents (objects with `ownerReference.blockOwnerDeletion=true`), it deletes the owner object.

#### Background cascading deletion

In background cascading deletion, Kubernetes deletes the owner object immediately and the garbage collector then deletes the dependents in the background.

### Setting the cascading deletion policy

To control the cascading deletion policy, set the propagationPolicy field on the deleteOptions argument when deleting an Object. Possible values include `“Orphan”, “Foreground”, or “Background”`.

Here’s an example that deletes dependents in background:
```
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
  -H "Content-Type: application/json"
```  
Here’s an example that deletes dependents in foreground:
```
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
  -H "Content-Type: application/json"
```
Here’s an example that orphans dependents:
```
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
  -H "Content-Type: application/json"
```

kubectl also supports cascading deletion. To delete dependents automatically using kubectl, set `--cascade to true`. To orphan dependents, set `--cascade to false`. The default value for --cascade is `true`.

Here’s an example that orphans the dependents of a ReplicaSet:
```
kubectl delete replicaset my-repset --cascade=false
```