# Annotations

You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata.

## Attaching metadata to objects

- You can use either labels or annotations to attach metadata to Kubernetes objects. 
- Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects. 
- The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.