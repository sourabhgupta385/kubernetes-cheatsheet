# Node Components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

1. kubelet
        An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

        The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

2. kube-proxy
        kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

        kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

        kube-proxy uses the operating system packet filtering layer if there is one and it’s available. Otherwise, kube-proxy forwards the traffic itself.

3. Container Runtime
        The container runtime is the software that is responsible for running containers.

        Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface).