# TTL Controller for Finished Resources

- The TTL controller provides a TTL (time to live) mechanism to limit the lifetime of resource objects that have finished execution. 
- TTL controller only handles Jobs for now, and may be expanded to handle other resources that will finish execution, such as Pods and custom resources.
- Alpha Disclaimer: this feature is currently alpha, and can be enabled with both kube-apiserver and kube-controller-manager feature gate TTLAfterFinished.

## TTL Controller

- A cluster operator can use this feature to clean up finished Jobs (either `Complete or Failed`) automatically by specifying the `.spec.ttlSecondsAfterFinished` field of a Job, as shown in below example. 
- The TTL controller will assume that a resource is eligible to be cleaned up TTL seconds after the resource has finished, in other words, when the TTL has expired. 
- When the TTL controller cleans up a resource, it will delete it cascadingly, that is to say it will delete its dependent objects together with it. 
- Note that when the resource is deleted, its lifecycle guarantees, such as finalizers, will be honored.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

The Job `pi-with-ttl` will be eligible to be automatically deleted, 100 seconds after it finishes.

If the field is set to 0, the Job will be eligible to be automatically deleted immediately after it finishes. If the field is unset, this Job won’t be cleaned up by the TTL controller after it finishes.

