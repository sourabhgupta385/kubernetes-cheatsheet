# Working with Pods

- When a Pod gets created (directly by you, or indirectly by a controller), it is scheduled to run on a Node in your cluster. 
- The Pod remains on that node until the process is terminated, the pod object is deleted, the Pod is evicted for lack of resources, or the node fails.

__Restarting a container in a Pod should not be confused with restarting a Pod. A Pod is not a process, but an environment for running a container. A Pod persists until it is deleted.__

- Pods do not, by themselves, self-heal. 
- If a Pod is scheduled to a Node that fails, or if the scheduling operation itself fails, the Pod is deleted; likewise, a Pod won’t survive an eviction due to a lack of resources or Node maintenance. 

Kubernetes uses a higher-level abstraction, called a controller, that handles the work of managing the relatively disposable Pod instances. Thus, while it is possible to use Pod directly, it’s far more common in Kubernetes to manage your pods using a controller.

## Pods and controllers

You can use workload resources to create and manage multiple Pods for you. 
A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. 
For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.

Here are some examples of workload resources that manage one or more Pods:

- Deployment
- StatefulSet
- DaemonSet