# Cluster Architecture

![alt text](../images/k8s-architecture-online.png "Logo Title Text 1")

## Working with K8s

- we create manifest(.yaml/.json file)
- apply this to cluster to bring it to desired state
- pods runs on node which is controlled be master

### Role of master node

- K8s cluster contains container running on Bare metal/ VM instances/ Cloud instances/ all mix
- k8s designates one or more of these as master and all others as workers
- the master is now going to run set of k8s processes. These processes will ensure smooth functioning of cluster. These processes all called __Control Plane__
- Can be multi master for high availability
- Master run control plane to run cluster smoothly

#### Components of Control Plane

1. __Kube-api-server__(for all communications)
   1. directly interact with user(through manifest)
   2. meant to scale automatically as per load
   3. front-end of Control Plane
2. __etcd__
   1. store meta data(machine understand metadata) and status of cluster
   2. consistent and high-available store(key-value store)
   3. source of touch for cluster state(info about state of cluster)
   4. etcd has following features:
      1. __Fully Replicated__ The entire state is available on every node in the cluster
      2. __Secure__ Implements automate TLS with optional client certificate authentication
      3. __Fast__ Benchmarked at 10000 writes per second
3. __Kube Scheduler__
   1. when user make request for the creation & management of pods, kube-schedular is going to take action on the requests
   2. Handles pods creation and management
   3. kube-schedular match/assign any node to create and run pods
   4. watches for newly created pods that have no node assigned for every pod that the schedular discovers, the schedular become responsible for finding best node for that pod to run on
   5. gets the information for hardware configuration from configuration files and schedules the pods on nodes accordingly
4. __Control Manager__
   1. make sure actual and desire state are same
   2. Two possible choices for control manager
      1. if k8s on cloud, then it will cloud-control-manager
      2. if k8s is on non-cloud, then it will kube-control-manager

#### Components on master that runs controller

1. __Node-controller__ for checking the cloud provider to determine if a node has been detected in the cloud after it stops responding
1. __Route-controller__ responsible for setting up networks routes on your cloud
1. __Service-controller__ responsible for load balancer on your cloud against services of type load balancer
1. __Volume-controller__ for creating, attaching and mounting volumes and interacting with the cloud provider to orchestrate volume

## Nods

nodes are going to run 3 important piece of software/process:

1. __Kublet__
   - agent running on node
   - listen to k8s master(e.g pod creation request)
   - use port 10255
   - sends success/fail reports to master

2. __Container Engine__
   - works with kubelet
   - pulling images
   - start/stop containers
   - exposing containers on ports specified in manifest
3. __Kube-Proxy__
   - assign ip to each pod
   - it is required to assign ip to pods(dynamic)
   - kube-proxy run on each node and this make sure that each pod will get its own unique ip address
  