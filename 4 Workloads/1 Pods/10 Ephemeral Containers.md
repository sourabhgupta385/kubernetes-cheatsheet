# Ephemeral Containers

- Ephemeral containers differ from other containers in that they lack guarantees for resources or execution, and they will never be automatically restarted, so they are not appropriate for building applications. 
- Ephemeral containers are described using the same `ContainerSpec` as regular containers, but many fields are incompatible and disallowed for ephemeral containers.
    - Ephemeral containers may not have ports, so fields such as `ports, livenessProbe, readinessProbe` are disallowed.
    - Pod resource allocations are immutable, so setting `resources` is disallowed.
- Ephemeral containers are created using a special `ephemeralcontainers` handler in the API rather than by adding them directly to `pod.spec`, so it’s not possible to add an ephemeral container using `kubectl edit`.
- Like regular containers, you may not change or remove an ephemeral container after you have added it to a Pod.

## Uses for ephemeral containers
- Ephemeral containers are useful for interactive troubleshooting when `kubectl exec` is insufficient because a container has crashed or a container image doesn’t include debugging utilities.
- In particular, `distroless` images enable you to deploy minimal container images that reduce attack surface and exposure to bugs and vulnerabilities. Since distroless images do not include a shell or any debugging utilities, it’s difficult to troubleshoot distroless images using kubectl exec alone.
- When using ephemeral containers, it’s helpful to enable process namespace sharing so you can view processes in other containers.

## Ephemeral containers API

Ephemeral containers are created using the `ephemeralcontainers` subresource of Pod, which can be demonstrated using kubectl --raw. First describe the ephemeral container to add as an `EphemeralContainers` list:

```
{
    "apiVersion": "v1",
    "kind": "EphemeralContainers",
    "metadata": {
            "name": "example-pod"
    },
    "ephemeralContainers": [{
        "command": [
            "sh"
        ],
        "image": "busybox",
        "imagePullPolicy": "IfNotPresent",
        "name": "debugger",
        "stdin": true,
        "tty": true,
        "terminationMessagePolicy": "File"
    }]
}
```

To update the ephemeral containers of the already running `example-pod`:

```
kubectl replace --raw /api/v1/namespaces/default/pods/example-pod/ephemeralcontainers  -f ec.json
```

You can view the state of the newly created ephemeral container using kubectl describe:

```
kubectl describe pod example-pod
```
```
...
Ephemeral Containers:
  debugger:
    Container ID:  docker://cf81908f149e7e9213d3c3644eda55c72efaff67652a2685c1146f0ce151e80f
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:9f1003c480699be56815db0f8146ad2e22efea85129b5b5983d0e0fb52d9ab70
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
    State:          Running
      Started:      Thu, 29 Aug 2019 06:42:21 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
...
```

You can interact with the new ephemeral container in the same way as other containers using kubectl attach, kubectl exec, and kubectl logs, for example:

```
kubectl attach -it example-pod -c debugger
```