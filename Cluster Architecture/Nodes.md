# Nodes

- A node is a worker machine in Kubernetes, previously known as a minion. 
- A node may be a VM or physical machine, depending on the cluster. 
- Each node contains the services necessary to run pods and is managed by the master components. 
- The services on a node include:
    - the container runtime
    - kubelet
    - kube-proxy

## Node Status

A node’s status contains the following information:

- Addresses
- Conditions
- Capacity and Allocatable
- Info

Node status and other details about a node can be displayed using the following command:

```
kubectl describe node <insert-node-name-here>
```

Each section is described in detail below.

#### Addresses

The usage of these fields varies depending on your cloud provider or bare metal configuration.

- **HostName:** The hostname as reported by the node’s kernel. Can be overridden via the kubelet `--hostname-override` parameter.
- **ExternalIP:** Typically the IP address of the node that is externally routable (available from outside the cluster).
- **InternalIP:** Typically the IP address of the node that is routable only within the cluster.

#### Conditions

The conditions field describes the status of all Running nodes. Examples of conditions include:

Node Condition     | Description
------------------ | -----------
Ready	           | True if the node is healthy and ready to accept pods, False if the node is not healthy and is not accepting pods, and Unknown if the node controller has not heard from the node in the last node-monitor-grace-period (default is 40 seconds)
MemoryPressure     | True if pressure exists on the node memory -- that is, if the node memory is low; otherwise False
PIDPressure	       | True if pressure exists on the processes -- that is, if there are too many processes on the node; otherwise False
DiskPressure       | True if pressure exists on the disk size -- that is, if the disk capacity is low; otherwise False
NetworkUnavailable | True if the network for the node is not correctly configured, otherwise False

- If the Status of the Ready condition remains `Unknown` or `False` for longer than the `pod-eviction-timeout` (an argument passed to the kube-controller-manager), all the Pods on the node are scheduled for deletion by the Node Controller. 
- The default eviction timeout duration is five minutes. 
- In some cases when the node is unreachable, the apiserver is unable to communicate with the kubelet on the node. The decision to delete the pods cannot be communicated to the kubelet until communication with the apiserver is re-established. In the meantime, the pods that are scheduled for deletion may continue to run on the partitioned node.
- In versions of Kubernetes prior to 1.5, the node controller would force delete these unreachable pods from the apiserver. 
- In 1.5 and higher, the node controller does not force delete pods until it is confirmed that they have stopped running in the cluster. 
- You can see the pods that might be running on an unreachable node as being in the `Terminating` or `Unknown` state. 
- In cases where Kubernetes cannot deduce from the underlying infrastructure if a node has permanently left a cluster, the cluster administrator may need to delete the node object by hand. 
- Deleting the node object from Kubernetes causes all the Pod objects running on the node to be deleted from the apiserver, and frees up their names.
- The node lifecycle controller automatically creates taints that represent conditions. The scheduler takes the Node’s taints into consideration when assigning a Pod to a Node. Pods can also have tolerations which let them tolerate a Node’s taints.

#### Capacity and Allocatable

- Describes the resources available on the node: 
    - CPU
    - memory
    - maximum number of pods that can be scheduled onto the node.
- The fields in the capacity block indicate the total amount of resources that a Node has. 
- The allocatable block indicates the amount of resources on a Node that is available to be consumed by normal Pods.

#### Info

- Describes general information about the node, such as kernel version, Kubernetes version (kubelet and kube-proxy version), Docker version (if used), and OS name. 
- This information is gathered by Kubelet from the node.

## Management

- When Kubernetes creates a node, it creates an object that represents the node. 
- After creation, Kubernetes checks whether the node is valid or not.
- Kubernetes creates a node object internally (the representation), and validates the node by health checking based on the `metadata.name` field. 
- If the node is valid -- that is, if all necessary services are running -- it is eligible to run a pod. Otherwise, it is ignored for any cluster activity until it becomes valid.
- Currently, there are three components that interact with the Kubernetes node interface: 
    - node controller
    - kubelet
    - kubectl