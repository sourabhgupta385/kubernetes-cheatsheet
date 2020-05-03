# Runtime Class

RuntimeClass is a feature for selecting the container runtime configuration. The container runtime configuration is used to run a Pod’s containers.

## Motivation

You can set a different RuntimeClass between different Pods to provide a balance of performance versus security. For example, if part of your workload deserves a high level of information security assurance, you might choose to schedule those Pods so that they run in a container runtime that uses hardware virtualization. You’d then benefit from the extra isolation of the alternative runtime, at the expense of some additional overhead.

You can also use RuntimeClass to run different Pods with the same container runtime but with different settings.

## Setup

Ensure the `RuntimeClass` feature gate is enabled (it is by default). The `RuntimeClass` feature gate must be enabled on apiservers and kubelets.

#### 1. Configure the CRI implementation on nodes

The configurations available through RuntimeClass are Container Runtime Interface (CRI) implementation dependent. See the corresponding documentation https://kubernetes.io/docs/concepts/containers/runtime-class/#cri-configuration

The configurations have a corresponding `handler` name, referenced by the RuntimeClass.

#### 2. Create the corresponding RuntimeClass resources

The configurations setup in step 1 should each have an associated handler name, which identifies the configuration. For each handler, create a corresponding RuntimeClass object.

The RuntimeClass resource currently only has 2 significant fields: the RuntimeClass name (metadata.name) and the handler (handler). The object definition looks like this:

```
apiVersion: node.k8s.io/v1beta1  # RuntimeClass is defined in the node.k8s.io API group
kind: RuntimeClass
metadata:
  name: myclass  # The name the RuntimeClass will be referenced by
  # RuntimeClass is a non-namespaced resource
handler: myconfiguration  # The name of the corresponding CRI configuration
```

## Usage
Once RuntimeClasses are configured for the cluster, using them is very simple. Specify a `runtimeClassName` in the Pod spec. For example:

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  # ...
```

This will instruct the Kubelet to use the named `RuntimeClass` to run this pod. If the named RuntimeClass does not exist, or the CRI cannot run the corresponding handler, the pod will enter the `Failed` terminal phase.

If no `runtimeClassName` is specified, the default RuntimeHandler will be used, which is equivalent to the behavior when the RuntimeClass feature is disabled.

## Scheduling

As of Kubernetes v1.16, RuntimeClass includes support for heterogenous clusters through its `scheduling` fields. Through the use of these fields, you can ensure that pods running with this RuntimeClass are scheduled to nodes that support it. To use the scheduling support, you must have the `RuntimeClass admission controller` enabled (the default, as of 1.16).

To ensure pods land on nodes supporting a specific RuntimeClass, that set of nodes should have a common label which is then selected by the `runtimeclass.scheduling.nodeSelector` field. The RuntimeClass’s nodeSelector is merged with the pod’s nodeSelector in admission, effectively taking the intersection of the set of nodes selected by each. If there is a conflict, the pod will be rejected.

If the supported nodes are tainted to prevent other RuntimeClass pods from running on the node, you can add `tolerations` to the RuntimeClass. As with the nodeSelector, the tolerations are merged with the pod’s tolerations in admission, effectively taking the union of the set of nodes tolerated by each.

## Pod Overhead

You can specify overhead resources that are associated with running a Pod. Declaring overhead allows the cluster (including the scheduler) to account for it when making decisions about Pods and resources.
To use Pod overhead, you must have the `PodOverhead` feature gate enabled (it is on by default).

Pod overhead is defined in RuntimeClass through the `overhead` fields. Through the use of these fields, you can specify the overhead of running pods utilizing this RuntimeClass and ensure these overheads are accounted for in Kubernetes.

