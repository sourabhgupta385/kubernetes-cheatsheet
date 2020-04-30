# Control Plane Components

The control plane’s components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment’s replicas field is unsatisfied).

Control plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all control plane components on the same machine, and do not run user containers on this machine. See Building High-Availability Clusters for an example multi-master-VM setup.

- **kube-apiserver**
    - The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. 
    - The API server is the front end for the Kubernetes control plane.
    - The main implementation of a Kubernetes API server is kube-apiserver. 
    - kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances. 
    - You can run several instances of kube-apiserver and balance traffic between those instances.

- **etcd**
    - Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

- **kube-scheduler**
    - Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.
    - Factors taken into account for scheduling decisions include: 
        - individual and collective resource requirements
        - hardware/software/policy constraints
        - affinity and anti-affinity specifications
        - data locality, inter-workload interference, and deadlines.

- **kube-controller-manager**
    - Control Plane component that runs controller processes.
    - Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
    - These controllers include:
        - **Node controller:** Responsible for noticing and responding when nodes go down.
        - **Replication controller:** Responsible for maintaining the correct number of pods for every replication controller object in the system.
        - **Endpoints controller:** Populates the Endpoints object (that is, joins Services & Pods).
        - **Service Account & Token controllers:** Create default accounts and API access tokens for new namespaces.

- **cloud-controller-manager**
    - A Kubernetes control plane component that embeds cloud-specific control logic. 
    - The cloud controller manager lets you link your cluster into your cloud provider’s API, and separates out the components that interact with that cloud platform from components that just interact with your cluster.
    - As with the kube-controller-manager, the cloud-controller-manager combines several logically independent control loops into a single binary that you run as a single process. 
    - You can scale horizontally (run more than one copy) to improve performance or to help tolerate failures.
     -The following controllers can have cloud provider dependencies:
         - **Node controller:** For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
         - **Route controller:** For setting up routes in the underlying cloud infrastructure
         - **Service controller:** For creating, updating and deleting cloud provider load balancers
