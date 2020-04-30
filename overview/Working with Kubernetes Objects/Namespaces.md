# Namespaces

Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.

## Not All Objects are in a Namespace

- Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. - Low-level resources, such as nodes and persistentVolumes, are not in any namespace.

To see which Kubernetes resources are and aren’t in a namespace:

```
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```