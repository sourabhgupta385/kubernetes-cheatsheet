# How Pods manage multiple containers

- Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service. 
- The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. 
- The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.
- Note that grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. 
- You should use this pattern only in specific instances in which your containers are tightly coupled. 
- For example, you might have a container that acts as a web server for files in a shared volume, and a separate “sidecar” container that updates those files from a remote source, as in the following diagram:

<img src="multiple-container-pod-eg.svg" width="1000">
<img src="multiple-container-pod-eg.svg">

Some Pods have `init containers` as well as `app containers`. Init containers run and complete before the app containers are started.

Pods provide two kinds of shared resources for their constituent containers: 

- Networking
- Storage

### Networking

- Each Pod is assigned a unique IP address for each address family. 
- Every container in a Pod shares the network namespace, including the IP address and network ports. 
- Containers inside a Pod can communicate with one another using `localhost`. 
- When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports).

### Storage

- A Pod can specify a set of shared storage volumes. 
- All containers in the Pod can access the shared volumes, allowing those containers to share data. 
- Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted.

