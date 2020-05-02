# Node Controller

- The node controller is a Kubernetes master component which manages various aspects of nodes.
- The node controller has multiple roles in a node’s life
    - The first is assigning a CIDR block to the node when it is registered (if CIDR assignment is turned on).
    - The second is keeping the node controller’s internal list of nodes up to date with the cloud provider’s list of available machines. 
        - When running in a cloud environment, whenever a node is unhealthy, the node controller asks the cloud provider if the VM for that node is still available. If not, the node controller deletes the node from its list of nodes.
     - The third is monitoring the nodes’ health. 
- The node controller is responsible for updating the NodeReady condition of NodeStatus to ConditionUnknown when a node becomes unreachable (i.e. the node controller stops receiving heartbeats for some reason, for example due to the node being down), and then later evicting all the pods from the node (using graceful termination) if the node continues to be unreachable. (The default timeouts are 40s to start reporting ConditionUnknown and 5m after that to start evicting pods.) 
- The node controller checks the state of each node every --node-monitor-period seconds.

### Heartbeats
- Heartbeats, sent by Kubernetes nodes, help determine the availability of a node. 
- There are two forms of heartbeats: 
    - updates of NodeStatus
    - updates of Lease object
- Each Node has an associated Lease object in the `kube-node-lease` namespace. 
- Lease is a lightweight resource, which improves the performance of the node heartbeats as the cluster scales.
- The kubelet is responsible for creating and updating the NodeStatus and a Lease object.
- The kubelet updates the NodeStatus either when there is change in status, or if there has been no update for a configured interval. 
- The default interval for NodeStatus updates is 5 minutes (much longer than the 40 second default timeout for unreachable nodes).
- The kubelet creates and then updates its Lease object every 10 seconds (the default update interval). 
- Lease updates occur independently from the NodeStatus updates. If the Lease update fails, the kubelet retries with exponential backoff starting at 200 milliseconds and capped at 7 seconds.