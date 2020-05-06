# Pod Lifecycle

This document describes the lifecycle of a Pod.

## Pod phase

A Pod’s status field is a PodStatus object, which has a phase field.

The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle.
Here are the possible values for phase:

Value     |	Description
--------- | ------------
Pending	  | The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.
Running	  | The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.
Succeeded |	All Containers in the Pod have terminated in success, and will not be restarted.
Failed	  | All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.
Unknown	  | For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.

## Pod conditions

A Pod has a PodStatus, which has an array of PodConditions through which the Pod has or has not passed. Each element of the PodCondition array has six possible fields:

- The `lastProbeTime` field provides a timestamp for when the Pod condition was last probed.
- The `lastTransitionTime` field provides a timestamp for when the Pod last transitioned from one status to another.
- The `message` field is a human-readable message indicating details about the transition.
- The `reason` field is a unique, one-word, CamelCase reason for the condition’s last transition.
- The `status` field is a string, with possible values “True”, “False”, and “Unknown”.
- The `type` field is a string with the following possible values:
    - `PodScheduled`: the Pod has been scheduled to a node;
    - `Ready`: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services;
    - `Initialized`: all init containers have started successfully;
    - `ContainersReady`: all containers in the Pod are ready.

## Container probes

A Probe is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a Handler implemented by the Container. There are three types of handlers:

- **ExecAction:** Executes a specified command inside the Container. The diagnostic is considered successful if the command exits with a status code of 0.
- **TCPSocketAction:** Performs a TCP check against the Container’s IP address on a specified port. The diagnostic is considered successful if the port is open.
- **HTTPGetAction:** Performs an HTTP Get request against the Container’s IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.

Each probe has one of three results:

- `Success`: The Container passed the diagnostic.
- `Failure`: The Container failed the diagnostic.
- `Unknown`: The diagnostic failed, so no action should be taken.

The kubelet can optionally perform and react to three kinds of probes on running Containers:

- `livenessProbe`: 
    - Indicates whether the Container is running. 
    - If the liveness probe fails, the kubelet kills the Container, and the Container is subjected to its restart policy. 
    - If a Container does not provide a liveness probe, the default state is Success.
- `readinessProbe`: 
    - Indicates whether the Container is ready to service requests. 
    - If the readiness probe fails, the endpoints controller removes the Pod’s IP address from the endpoints of all Services that match the Pod. 
    - The default state of readiness before the initial delay is Failure. 
    - If a Container does not provide a readiness probe, the default state is Success.
- `startupProbe`: 
    - Indicates whether the application within the Container is started. 
    - All other probes are disabled if a startup probe is provided, until it succeeds. 
    - If the startup probe fails, the kubelet kills the Container, and the Container is subjected to its restart policy. 
    - If a Container does not provide a startup probe, the default state is Success.

## Container States

To check state of container, you can use `kubectl describe pod [POD_NAME]`. State is displayed for each container within that Pod.

There are three possible states of containers:

- **Waiting**: 
    - Default state of container. 
    - If container is not in either Running or Terminated state, it is in Waiting state. 
    - A container in Waiting state still runs its required operations, like pulling images, applying Secrets, etc. 
- **Running**: 
    - Indicates that the container is executing without issues. 
    - The postStart hook (if any) is executed prior to the container entering a Running state. 
    - This state also displays the time when the container entered Running state.
- **Terminated**: 
    - Indicates that the container completed its execution and has stopped running. 
    - A container enters into this when it has successfully completed execution or when it has failed for some reason. 
    - Regardless, a reason and exit code is displayed, as well as the container’s start and finish time. 
    - Before a container enters into Terminated, preStop hook (if any) is executed.

## Restart policy

A PodSpec has a restartPolicy field with possible values 
- Always
- OnFailure
- Never

The default value is `Always`. `restartPolicy` applies to all Containers in the Pod.

## Pod lifetime

There are different kinds of resources for creating Pods:

- Use a `Deployment, ReplicaSet or StatefulSet` for Pods that are not expected to terminate, for example, web servers.
- Use a `Job` for Pods that are expected to terminate once their work is complete; for example, batch computations. `Jobs` are appropriate only for Pods with restartPolicy equal to OnFailure or Never.
- Use a `DaemonSet` for Pods that need to run one per eligible node.