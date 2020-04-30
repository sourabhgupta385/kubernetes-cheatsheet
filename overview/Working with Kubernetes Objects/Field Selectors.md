# Field Selectors

Field selectors let you select Kubernetes resources based on the value of one or more resource fields. Here are some examples of field selector queries:

```
metadata.name=my-service
metadata.namespace!=default
status.phase=Pending
```

This kubectl command selects all Pods for which the value of the status.phase field is Running:

```
kubectl get pods --field-selector status.phase=Running
```

## Supported fields

Supported field selectors vary by Kubernetes resource type. All resource types support the metadata.name and metadata.namespace fields. Using unsupported field selectors produces an error. For example:

```
kubectl get ingress --field-selector foo.bar=baz

Error from server (BadRequest): Unable to find "ingresses" that match label selector "", field selector "foo.bar=baz": "foo.bar" is not a known field selector: only "metadata.name", "metadata.namespace"
```

## Supported operators

You can use the =, ==, and != operators with field selectors (= and == mean the same thing). This kubectl command, for example, selects all Kubernetes Services that aren’t in the default namespace:

```
kubectl get services  --all-namespaces --field-selector metadata.namespace!=default
```

## Chained selectors

As with label and other selectors, field selectors can be chained together as a comma-separated list. This kubectl command selects all Pods for which the status.phase does not equal Running and the spec.restartPolicy field equals Always:

```
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```