# Control Plane-Node Communication

There can be two communication paths:

- Node to Control Plane
- Control Plane to Node

## Node to Control Plane

- All communication paths from the nodes to the control plane terminate at the apiserver (none of the other master components are designed to expose remote services). 
- In a typical deployment, the apiserver is configured to listen for remote connections on a secure HTTPS port (443) with one or more forms of client authentication enabled.
- Nodes should be provisioned with the public root certificate for the cluster such that they can connect securely to the apiserver along with valid client credentials.
- Pods that wish to connect to the apiserver can do so securely by leveraging a service account so that Kubernetes will automatically inject the public root certificate and a valid bearer token into the pod when it is instantiated. 
- The `kubernetes` service (in all namespaces) is configured with a virtual IP address that is redirected (via kube-proxy) to the HTTPS endpoint on the apiserver.
- The control plane components also communicate with the cluster apiserver over the secure port.
- The default operating mode for connections from the nodes and pods running on the nodes to the control plane is secured by default and can run over untrusted and/or public networks.

## Control Plane to node

- There are two primary communication paths from the control plane (apiserver) to the nodes:
    - The first is from the apiserver to the kubelet process which runs on each node in the cluster. 
    - The second is from the apiserver to any node, pod, or service through the apiserver’s proxy functionality.

#### apiserver to kubelet

The connections from the apiserver to the kubelet are used for:

- Fetching logs for pods.
- Attaching (through kubectl) to running pods.
- Providing the kubelet’s port-forwarding functionality.

These connections terminate at the kubelet’s HTTPS endpoint. By default, the apiserver does not verify the kubelet’s serving certificate, which makes the connection subject to **man-in-the-middle** attacks, and unsafe to run over untrusted and/or public networks.

To verify this connection, use the `--kubelet-certificate-authority` flag to provide the apiserver with a root certificate bundle to use to verify the kubelet’s serving certificate.

If that is not possible, use `SSH tunneling` between the apiserver and kubelet if required to avoid connecting over an untrusted or public network.

Finally, Kubelet authentication and/or authorization should be enabled to secure the kubelet API.

#### apiserver to nodes, pods, and services

- The connections from the apiserver to a node, pod, or service default to plain HTTP connections and are therefore neither authenticated nor encrypted. 
- They can be run over a secure HTTPS connection by prefixing `https:` to the node, pod, or service name in the API URL, but they will not validate the certificate provided by the HTTPS endpoint nor provide client credentials so while the connection will be encrypted, it will not provide any guarantees of integrity. 
- These connections are not currently safe to run over untrusted and/or public networks.

### SSH tunnels

- Kubernetes supports SSH tunnels to protect the control plane to nodes communication paths. 
- In this configuration, the apiserver initiates an SSH tunnel to each node in the cluster (connecting to the ssh server listening on port 22) and passes all traffic destined for a kubelet, node, pod, or service through the tunnel. 
- This tunnel ensures that the traffic is not exposed outside of the network in which the nodes are running.
- SSH tunnels are currently deprecated so you shouldn’t opt to use them unless you know what you are doing. 
- The Konnectivity service is a replacement for this communication channel.

### Konnectivity service

- As a replacement to the SSH tunnels, the Konnectivity service provides TCP level proxy for the control plane to Cluster communication. 
- The Konnectivity consists of two parts
    - the Konnectivity server
    - the Konnectivity agents
- Konnectivity server run in the control plane network.
- Konnectivity agents run in the nodes network. 
- The Konnectivity agents initiate connections to the Konnectivity server and maintain the connections. 
- All control plane to nodes traffic then goes through these connections.





