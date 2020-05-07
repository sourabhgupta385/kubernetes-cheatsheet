# Pod Preset

A Pod Preset is an API resource for injecting additional runtime requirements into a Pod at creation time. You use label selectors to specify the Pods to which a given Pod Preset applies.

Using a Pod Preset allows pod template authors to not have to explicitly provide all information for every pod. This way, authors of pod templates consuming a specific service do not need to know all the details about that service.

## How It Works

Kubernetes provides an admission controller (`PodPreset`) which, when enabled, applies Pod Presets to incoming pod creation requests. When a pod creation request occurs, the system does the following:

- Retrieve all `PodPresets` available for use.
- Check if the label selectors of any `PodPreset` matches the labels on the pod being created.
- Attempt to merge the various resources defined by the `PodPreset` into the Pod being created.
- On error, throw an event documenting the merge error on the pod, and create the pod without any injected resources from the `PodPreset`.
- Annotate the resulting modified Pod spec to indicate that it has been modified by a PodPreset. The annotation is of the form 

```
podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>"
```

- Each Pod can be matched by zero or more Pod Presets; and each PodPreset can be applied to zero or more pods. 
- When a PodPreset is applied to one or more Pods, Kubernetes modifies the Pod Spec. 
- For changes to `Env`, `EnvFrom`, and `VolumeMounts`, Kubernetes modifies the `container spec` for all containers in the Pod
- For changes to `Volume`, Kubernetes modifies the `Pod Spec`.