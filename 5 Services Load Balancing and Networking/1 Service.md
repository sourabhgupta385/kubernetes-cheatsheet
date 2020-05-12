# Service

Each Pod gets its own IP address, however in a Deployment, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them “backends”) provides functionality to other Pods (call them “frontends”) inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload?

Enter Services.

Service is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this pattern is called a micro-service). The set of Pods targeted by a Service is usually determined by a `selector`.

## Defining a Service

For example, suppose you have a set of Pods that each listen on TCP port 9376 and carry a label `app=MyApp`

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

- This specification creates a new Service object named “my-service”, which targets TCP port 9376 on any Pod with the `app=MyApp` label.
- Kubernetes assigns this Service an IP address (sometimes called the “cluster IP”), which is used by the Service proxies (see Virtual IPs and service proxies below).
- The controller for the Service selector continuously scans for Pods that match its selector, and then POSTs any updates to an Endpoint object also named “my-service”.
- The default protocol for Services is TCP.
- As many Services need to expose more than one port, Kubernetes supports multiple port definitions on a Service object. Each port definition can have the same protocol, or a different one.

## Services without selectors

Services most commonly abstract access to Kubernetes Pods, but they can also abstract other kinds of backends. For example:

- You want to have an external database cluster in production, but in your test environment you use your own databases.
- You want to point your Service to a Service in a different Namespace or on another cluster.
- You are migrating a workload to Kubernetes. Whilst evaluating the approach, you run only a proportion of your backends in Kubernetes.

In any of these scenarios you can define a Service without a Pod selector. For example:
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
Because this Service has no selector, the corresponding Endpoint object is not created automatically. You can manually map the Service to the network address and port where it’s running, by adding an Endpoint object manually:
```
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

Accessing a Service without a selector works the same as if it had a selector. In the example above, traffic is routed to the single endpoint defined in the YAML: 192.0.2.42:9376 (TCP).      

## Virtual IPs and service proxies

Every node in a Kubernetes cluster runs a `kube-proxy`. `kube-proxy` is responsible for implementing a form of virtual IP for Services of type other than ExternalName.

There are three types of proxy modes:

- User space proxy mode
- iptables proxy mode
- IPVS proxy mode

#### User space proxy mode

- In this mode, kube-proxy watches the Kubernetes master for the addition and removal of Service and Endpoint objects. 
- For each Service it opens a port (randomly chosen) on the local node. 
- Any connections to this “proxy port” are proxied to one of the Service’s backend Pods (as reported via Endpoints). 
- kube-proxy takes the SessionAffinity setting of the Service into account when deciding which backend Pod to use.
- Lastly, the user-space proxy installs iptables rules which capture traffic to the Service’s clusterIP (which is virtual) and port. 
- The rules redirect that traffic to the proxy port which proxies the backend Pod.
- By default, kube-proxy in userspace mode chooses a backend via a round-robin algorithm.

<p align="center">
  <img src="services-userspace-overview.svg" width="500">
</p> 

#### iptables proxy mode

- In this mode, kube-proxy watches the Kubernetes control plane for the addition and removal of Service and Endpoint objects. 
- For each Service, it installs iptables rules, which capture traffic to the Service’s clusterIP and port, and redirect that traffic to one of the Service’s backend sets. 
- For each Endpoint object, it installs iptables rules which select a backend Pod.
- By default, kube-proxy in iptables mode chooses a backend at random.
- Using iptables to handle traffic has a lower system overhead, because traffic is handled by Linux `netfilter` without the need to switch between userspace and the kernel space. This approach is also likely to be more reliable.

<p align="center">
  <img src="services-iptables-overview.svg" width="500">
</p> 

**Difference between iptables proxy mode and User space proxy mode**

- If kube-proxy is running in iptables mode and the first Pod that’s selected does not respond, the connection fails. 
- This is different from userspace mode: in that scenario, kube-proxy would detect that the connection to the first Pod had failed and would automatically retry with a different backend Pod.

#### IPVS proxy mode

- In ipvs mode, kube-proxy watches Kubernetes Services and Endpoints, calls `netlink` interface to create IPVS rules accordingly and synchronizes IPVS rules with Kubernetes Services and Endpoints periodically.
- This control loop ensures that IPVS status matches the desired state. 
- When accessing a Service, IPVS directs traffic to one of the backend Pods.
- The IPVS proxy mode is based on `netfilter` hook function that is similar to iptables mode, but uses a hash table as the underlying data structure and works in the kernel space. 
- That means kube-proxy in IPVS mode redirects traffic with lower latency than kube-proxy in iptables mode, with much better performance when synchronising proxy rules. 
- Compared to the other proxy modes, IPVS mode also supports a higher throughput of network traffic.

<p align="center">
  <img src="services-ipvs-overview.svg" width="500">
</p> 

## Multi-Port Services
For some Services, you need to expose more than one port. Kubernetes lets you configure multiple port definitions on a Service object. When using multiple ports for a Service, you must give all of your ports names so that these are unambiguous. For example:
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

## Discovering services

Kubernetes supports 2 primary modes of finding a Service:
- environment variables
- DNS

#### Environment variables

- When a Pod is run on a Node, the kubelet adds a set of environment variables for each active Service.
- It supports both Docker links compatible variables (see makeLinkVariables) and simpler `{SVCNAME}_SERVICE_HOST` and `{SVCNAME}_SERVICE_PORT` variables, where the Service name is upper-cased and dashes are converted to underscores.

For example, the Service `"redis-master"` which exposes TCP port 6379 and has been allocated cluster IP address 10.0.0.11, produces the following environment variables:
```
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

#### DNS
- You can (and almost always should) set up a DNS service for your Kubernetes cluster using an add-on.
- A cluster-aware DNS server, such as CoreDNS, watches the Kubernetes API for new Services and creates a set of DNS records for each one. 
- If DNS has been enabled throughout your cluster then all Pods should automatically be able to resolve Services by their DNS name.
- For example, if you have a Service called "my-service" in a Kubernetes Namespace "my-ns", the control plane and the DNS Service acting together create a DNS record for "my-service.my-ns". 
- Pods in the "my-ns" Namespace should be able to find it by simply doing a name lookup for my-service ("my-service.my-ns" would also work).
- Pods in other Namespaces must qualify the name as my-service.my-ns. 
- These names will resolve to the cluster IP assigned for the Service.

## Publishing Services (ServiceTypes)

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that’s outside of your cluster.

Kubernetes `ServiceTypes` allow you to specify what kind of Service you want. The default is ClusterIP.

`Type` values and their behaviors are:

- **ClusterIP**: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.

- **NodePort**: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
- **LoadBalancer**: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.On cloud providers which support external load balancers, setting the type field to LoadBalancer provisions a load balancer for your Service. The actual creation of the load balancer happens asynchronously, and information about the provisioned balancer is published in the Service’s .status.loadBalancer field. For example:
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```
- **ExternalName**: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.This Service definition, for example, maps the my-service Service in the prod namespace to my.database.example.com:
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

## Supported protocols

**TCP**
You can use TCP for any kind of Service, and it’s the default network protocol.

**UDP**
You can use UDP for most Services. For type=LoadBalancer Services, UDP support depends on the cloud provider offering this facility.

**HTTP**
If your cloud provider supports it, you can use a Service in LoadBalancer mode to set up external HTTP / HTTPS reverse proxying, forwarded to the Endpoints of the Service.

**PROXY protocol**
If your cloud provider supports it (eg, AWS), you can use a Service in LoadBalancer mode to configure a load balancer outside of Kubernetes itself, that will forward connections prefixed with PROXY protocol.

The load balancer will send an initial series of octets describing the incoming connection, similar to this example
```
PROXY TCP4 192.0.2.202 10.0.42.7 12345 7\r\n
```
followed by the data from the client.

**SCTP**
Kubernetes supports SCTP as a protocol value in Service, Endpoint, NetworkPolicy and Pod definitions as an alpha feature. To enable this feature, the cluster administrator needs to enable the SCTPSupport feature gate on the apiserver, for example, --feature-gates=SCTPSupport=true