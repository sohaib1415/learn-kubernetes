# Policies

Policies you can configure that apply to groups of resources.

## Managing compute resources for containers

- A pod in k8s will run with no limits on CPU and memory
- you can optionally specify how much CPU and memory(RAM) each container needs
- Schedular decides about which nodes to place pods, only if the node has enough CPU resources available to satisfy the pod CPU request
- CPU is specified in cores
- Memory is specified in bytes

Two types of constraints can be set for each resource type - Request and limits

a `request` is the amount of that resources that the system will guarantee for the container and k8s will use this value to decide on which mode to place the pod

a `limit` is the max amount of resources that k8s will allow the container to use

- If _request is not set_ for a container, then `request=limits`
- If _limit is not set_, then if default to `1`

__Default range__ (custom namespace)
CPU:
  request - 0.5
  limit - 1
Memory:
  request - 500M
  limit - 1G

- CPU values are specified in 'Milli CPU' and memory in MiB
- 1CPU=1000mi or 0.5 CPU

## ResourceQuota

A k8s cluster can be divided into namespaces. If a container is created in a namespace that has a default CPU limit, and the container does not specify its owb CPU limit, then the container is assigned the default CPU limit

Namespaces can bve assigned ResourceQuota objects, this will limit the amount of usage allowed to the objects in that namespace, You can limit:

1. compute
2. memory
3. storage

- Here are 2 restrictions that a resource quota imposes on a namespace:

1. Every container that runs in the namespace must have its own CPU limit
2. The total amount of CPU used by all containers in the namespace must not exceed a specified limit

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resources
spec:
  containers:
  - name: resource
    image: centos
    command: ["/bin/bash", "-c", "while true; do echo Technical-Guftgu; sleep 5 ; done"]
    resources:                                          
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
```

```kubernetes
kubectl apply -f podresources.yaml
kubectl config view | grep namespace
kubectl get pods
kubectl describe pod resources
kubectl delete -f podresources.yaml
```

```yaml
# resourcequota.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
   name: myquota
spec:
  hard:
    limits.cpu: "400m"
    limits.memory: "400Mi"
    requests.cpu: "200m"
    requests.memory: "200Mi"
---
# testpod.yaml

kind: Deployment
apiVersion: apps/v1
metadata:
  name: deployments
spec:
  replicas: 3
  selector:      
    matchLabels:
     objtype: deployment
  template:
    metadata:
      name: testpod8
      labels:
        objtype: deployment
    spec:
     containers:
       - name: c00
         image: ubuntu
         command: ["/bin/bash", "-c", "while true; do echo Technical-Guftgu; sleep 5 ; done"]
         resources:
            requests:
              cpu: "200m"
```

```sh
kubectl apply -f resourcequota.yaml
kubectl apply -f testpod.yaml
kubectl get deploy
kubectl get pods
kubectl get rc # given error resource quota limit exceeded
kubectl delete -f resourcequota.yaml
kubectl delete -f testpod.yaml
```

```yaml
# cpudefault.yaml

apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

```kubernetes
kubectl apply -f resourcequota.yaml
kubectl apply -f pod.yaml
kubectl get pods
kubectl delete -f pod.yaml
```

```yaml
# cpu2.yml
apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo-2
spec:
  containers:
  - name: default-cpu-demo-2-ctr
    image: nginx
    resources:
      limits:
        cpu: "1"
```

```kubernetes
kubectl apply -f cpu2.yaml
kubectl get pods
kubectl describe pod default-cpu-demo-2
kubectl delete -f cpu2.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo-3
spec:
  containers:
  - name: default-cpu-demo-3-ctr
    image: nginx
    resources:
      requests:
        cpu: "0.75"
```

```kubernetes
kubectl apply -f cpu3.yaml
kubectl get pods
kubectl describe pod mypod
kubectl delete -f cpu3.yaml
```

## Horizontal Pod Autoscaler

- K8s has the possibility to automatically scale pods based on observer CPU utilization, which is Horizontal Pod Autoscaling

- You provide CPU limit let say 20%(pod/container CPU limit) which means, when the average CPU limits of all the pods with containers exceeds from this limit create a new pod
- HPA get this information using an API called metrics-server which is open sources
- Metrics-server contains the information of resources(CPU/memory/storage) utilization in every pod
- scaling can be done only for scalable objects like controller, deployment or replicaSet
- HPA is implemented as a k8s API resource and a controller
- the controller periodically adjust the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user
- the HPA is implemented as a controlled loop with a period controlled by controller manager's-horizontal-pod-autoscalar-sync-period flag(default 30s)
- during each period, the controller manager queries the resource utilization against the matrices specifier in each horizontal-pod-autoscalar definition
- for per-pod resource metrics like CPU, the controller fetches the metrics from the resource metrics API for each pod targeted by the HorizontalPodAutoScaler
- Then if a target raw-value is set, the raw metrics values are used directly. the controller then takes the mean of the utilization or the raw value across all targeted pods and produces a ratio used to scale the number of desired replicas
- cool down period to wait before another downscale operation can be performed is controller by -- horizontal-pod-autoscaler-downtime-stabilization flag(default 5mins)

- metric server needs to be deployed in the cluster to provide metrics via the resources metrics API
- pass metrics-server installation argument --kublet-insecure-tts to avoid certificate requirement for metrics-server

### Install metrics server

$ wget -O metricserver.yml https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

vi metricserver.yml # add --kublet-insecure-tts in deployment > spec > container > args

```yaml

# deployhpa.yml

kind: Deployment
apiVersion: apps/v1
metadata:
   name: mydeploy
spec:
   replicas: 1
   selector:
    matchLabels:
     name: deployment
   template:
     metadata:
       name: testpod8
       labels:
         name: deployment
     spec:
      containers:
        - name: c00
          image: httpd
          ports:
          - containerPort: 80
          resources:
            limits:
              cpu: 500m
            requests:
              cpu: 200m
```

```sh
kubectl apply -f deployhpa.yml
$ kubectl autoscale deployment mydeploy --cpu-percent=20 --min=1 --max=10 # output: horizontalpodautoscaler.autoscaling/mydeploy autoscaled

$ kubectl exec mydeploy-5fddffdc88-5j477 -it -- /bin/bash
```
