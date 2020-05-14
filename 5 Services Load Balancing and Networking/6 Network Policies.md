# Network Policies

- A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.
- NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods.
- Network policies are implemented by the network plugin. 
- To use network policies, you must be using a networking solution which supports NetworkPolicy. 
- Creating a NetworkPolicy resource without a controller that implements it will have no effect.

## Isolated and Non-isolated Pods

- By default, pods are non-isolated; they accept traffic from any source.
- Pods become isolated by having a NetworkPolicy that selects them. 
- Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. 
- Other pods in the namespace that are not selected by any NetworkPolicy will continue to accept all traffic.
- Network policies do not conflict; they are additive. 
- If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies’ ingress/egress rules. Thus, order of evaluation does not affect the policy result.

## The NetworkPolicy resource
An example NetworkPolicy might look like this:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

the example NetworkPolicy:

- isolates “role=db” pods in the “default” namespace for both ingress and egress traffic (if they weren’t already isolated)
- (Ingress rules) allows connections to all pods in the “default” namespace with the label “role=db” on TCP port 6379 from:
    - any pod in the “default” namespace with the label “role=frontend”
    - any pod in a namespace with the label “project=myproject”
    - IP addresses in the ranges 172.17.0.0–172.17.0.255 and 172.17.2.0–172.17.255.255 (ie, all of 172.17.0.0/16 except 172.17.1.0/24)
- (Egress rules) allows connections from any pod in the “default” namespace with the label “role=db” to CIDR 10.0.0.0/24 on TCP port 5978

## Behavior of to and from selectors
There are four kinds of selectors that can be specified in an ingress from section or egress to section:

- **podSelector**: This selects particular Pods in the same namespace as the NetworkPolicy which should be allowed as ingress sources or egress destinations.
- **namespaceSelector**: This selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations.
- **namespaceSelector and podSelector**: A single to/from entry that specifies both namespaceSelector and podSelector selects particular Pods within particular namespaces. Be careful to use correct YAML syntax; this policy:
```
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
  ...
```
contains a single from element allowing connections from Pods with the label role=client in namespaces with the label user=alice. But this policy:
```
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
  ...
```
contains two elements in the from array, and allows connections from Pods in the local Namespace with the label role=client, or from any Pod in any namespace with the label user=alice.

When in doubt, use kubectl describe to see how Kubernetes has interpreted the policy.

## Default policies

By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace. The following examples let you change the default behavior in that namespace.

#### Default deny all ingress traffic
You can create a “default” isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any ingress traffic to those pods.

```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
This ensures that even pods that aren’t selected by any other NetworkPolicy will still be isolated. This policy does not change the default egress isolation behavior.

#### Default allow all ingress traffic
If you want to allow all traffic to all pods in a namespace (even if policies are added that cause some pods to be treated as “isolated”), you can create a policy that explicitly allows all traffic in that namespace.
```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```

#### Default deny all egress traffic
You can create a “default” egress isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any egress traffic from those pods.

```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```
This ensures that even pods that aren’t selected by any other NetworkPolicy will not be allowed egress traffic. This policy does not change the default ingress isolation behavior.

#### Default allow all egress traffic
If you want to allow all traffic from all pods in a namespace (even if policies are added that cause some pods to be treated as “isolated”), you can create a policy that explicitly allows all egress traffic in that namespace.
```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
spec:
  podSelector: {}
  egress:
  - {}
  policyTypes:
  - Egress
```

#### Default deny all ingress and all egress traffic
You can create a “default” policy for a namespace which prevents all ingress AND egress traffic by creating the following NetworkPolicy in that namespace.
```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
This ensures that even pods that aren’t selected by any other NetworkPolicy will not be allowed ingress or egress traffic.