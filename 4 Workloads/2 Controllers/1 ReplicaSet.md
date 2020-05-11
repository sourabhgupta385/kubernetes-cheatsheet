# ReplicaSet

A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

## How a ReplicaSet works

- A ReplicaSet is defined with fields, including:
    - a selector that specifies how to identify Pods it can acquire
    - a number of replicas indicating how many Pods it should be maintaining
    - a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. 
- A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. 
- When a ReplicaSet needs to create new Pods, it uses its Pod template.
- A ReplicaSet identifies new Pods to acquire by using its selector.

## When to use a ReplicaSet

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, it is recommend using Deployments instead of directly using ReplicaSets.

## Example

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

Saving this manifest into `frontend.yaml` and submitting it to a Kubernetes cluster will create the defined ReplicaSet and the Pods that it manages.
```
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```
You can then get the current ReplicaSets deployed:
```
kubectl get rs
```
And see the frontend one you created:
```
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```

## Non-Template Pod acquisitions

While you can create bare Pods with no problems, it is strongly recommended to make sure that the bare Pods do not have labels which match the selector of one of your ReplicaSets. The reason for this is because a ReplicaSet is not limited to owning Pods specified by its template-- it can acquire other Pods in the manner specified in the previous sections.

Take the previous frontend ReplicaSet example, and the Pods specified in the following manifest:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: hello1
    image: gcr.io/google-samples/hello-app:2.0

---

apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    tier: frontend
spec:
  containers:
  - name: hello2
    image: gcr.io/google-samples/hello-app:1.0
```
As those Pods do not have a Controller (or any object) as their owner reference and match the selector of the frontend ReplicaSet, they will immediately be acquired by it.

Suppose you create the Pods after the frontend ReplicaSet has been deployed and has set up its initial Pod replicas to fulfill its replica count requirement:
```
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```
The new Pods will be acquired by the ReplicaSet, and then immediately terminated as the ReplicaSet would be over its desired count.

Fetching the Pods:
```
kubectl get pods
```
The output shows that the new Pods are either already terminated, or in the process of being terminated:
```
NAME             READY   STATUS        RESTARTS   AGE
frontend-b2zdv   1/1     Running       0          10m
frontend-vcmts   1/1     Running       0          10m
frontend-wtsmm   1/1     Running       0          10m
pod1             0/1     Terminating   0          1s
pod2             0/1     Terminating   0          1s
```
If you create the Pods first:
```
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```
And then create the ReplicaSet however:
```
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```
You shall see that the ReplicaSet has acquired the Pods and has only created new ones according to its spec until the number of its new Pods and the original matches its desired count. As fetching the Pods:
```
kubectl get pods
```
Will reveal in its output:
```
NAME             READY   STATUS    RESTARTS   AGE
frontend-hmmj2   1/1     Running   0          9s
pod1             1/1     Running   0          36s
pod2             1/1     Running   0          36s
```
In this manner, a ReplicaSet can own a non-homogenous set of Pods

## Working with ReplicaSets

#### Deleting a ReplicaSet and its Pods

To delete a ReplicaSet and all of its Pods, use
```
kubectl delete
```

#### Deleting just a ReplicaSet

You can delete a ReplicaSet without affecting any of its Pods using
```
kubectl delete --cascade=false
```
#### Isolating Pods from a ReplicaSet
You can remove Pods from a ReplicaSet by changing their labels. 

#### Scaling a ReplicaSet
- A ReplicaSet can be easily scaled up or down by simply updating the `.spec.replicas` field. 
- The ReplicaSet controller ensures that a desired number of Pods with a matching label selector are available and operational.

#### ReplicaSet as a Horizontal Pod Autoscaler Target
A ReplicaSet can also be a target for `Horizontal Pod Autoscalers (HPA)`. That is, a ReplicaSet can be auto-scaled by an HPA. Here is an example HPA targeting the ReplicaSet we created in the previous example.

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

Saving this manifest into `hpa-rs.yaml` and submitting it to a Kubernetes cluster should create the defined HPA that autoscales the target ReplicaSet depending on the CPU usage of the replicated Pods.
```
kubectl apply -f https://k8s.io/examples/controllers/hpa-rs.yaml
```
Alternatively, you can use the kubectl autoscale command to accomplish the same (and it’s easier!)
```
kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50
```

