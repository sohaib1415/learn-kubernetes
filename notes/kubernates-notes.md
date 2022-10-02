# Kubernetes

---

- [Kubernetes](#kubernetes)
  - [History](#history)
    - [Online platform for K8s](#online-platform-for-k8s)
    - [Cloud based K8s services](#cloud-based-k8s-services)
    - [K8s installation tools](#k8s-installation-tools)
    - [Problems with out a container management tools](#problems-with-out-a-container-management-tools)
  - [Features of K8s](#features-of-k8s)
  - [Higher level k8s objects](#higher-level-k8s-objects)
    - [Setup Kubernetes on AWS](#setup-kubernetes-on-aws)
      - [Common commands for master and worker nodes](#common-commands-for-master-and-worker-nodes)
      - [BOOTSTRAPPING THE MASTER NODE (IN MASTER)](#bootstrapping-the-master-node-in-master)
      - [CONFIGURE WORKER NODES (IN NODES)](#configure-worker-nodes-in-nodes)
        - [GO TO MASTER AND RUN THIS COMMAND](#go-to-master-and-run-this-command)
    - [Install Minikube on AWS](#install-minikube-on-aws)
  - [Kubernetes Objects](#kubernetes-objects)
    - [Labels and selectors](#labels-and-selectors)
    - [Replication Controller](#replication-controller)
    - [Replica Set](#replica-set)
    - [Deployment](#deployment)
    - [Services](#services)
    - [Volume](#volume)
    - [Configurations](#configurations)
    - [Namespace](#namespace)
    - [Resource Quota](#resource-quota)
    - [Job](#job)
    - [HELM](#helm)
    - [Scheduling](#scheduling)
    - [Understand the role of DaemonSets](#understand-the-role-of-daemonsets)
    - [K8s Logging and Monitoring](#k8s-logging-and-monitoring)
      - [Monitor Cluster Component](#monitor-cluster-component)
      - [Monitor Applications](#monitor-applications)
      - [Cluster component logs](#cluster-component-logs)
      - [Application Logs](#application-logs)

---

- Open-source container management tool which automates container deployment, container scaling and load balancing
- It schedules, run and manages isolated container which are running in virtual, physical and cloud machines

## History

- Google developed internal system called 'borg (later named as Omega) to deploy and managed thousands of google applications and services on their clusters(group of containers)
- In 2014 Google introduced K8s an open-source platform written in Golang and later donated to CNCF(cloud native computing foundation)

### Online platform for K8s

- K8s playground
- Play with k8s
- play with k8s classroom

### Cloud based K8s services

- GKS -> Google Kubernetes Services
- AKS -> Azure Kubernetes Services
- EKS -> Elastic Kubernetes Services

### K8s installation tools

- minikube
- kubeadm

### Problems with out a container management tools

- containers can not communicate with each other
- auto scaling and load balancing was not possible

## Features of K8s

- Orchestration(clustering of any number of container in different networks)
- Auto scaling(vertical scaling: adding resources on same container and horizontal scaling: adding containers/hosts) 
- Auto Healing
- Load Balancing
- Platform independent(cloud/virtual/on-premise)
- Fault tolerance(node/pod failure)
- Rollback
- Health monitoring of containers
- Batch execution

| Features | Kubernetes | Docker Swarm |
| --- | --- | --- |
Installation and Cluster configurations | Complicated and time consuming | Fast and easy
Supports | support almost all containers type Rocket, ContainerD, Docker | Docker only
GUI | available | Not
Data Volumes | only shared with containers in same pod | Can be shared with any other container
Updates and Roll Back | process scheduling to maintain services while updating | Progressive updates of service health monitoring through out the update
Auto Scaling | vertical and horizontal | Not
Logging and Monitoring | Inbuilt tools present for monitoring | Used 3rd party tools like Splunk

## Higher level k8s objects

used to achieve above issues(by default non auto scaling)

1. __Replication Set__: scaling and healing
2. __Deployment__: Versioning and rollback
3. __Service__: static(non-ephemeral) IP and networking
4. __Volume__: non-ephemeral storage(outside node)

> kubetl -> single cloud
> kubeadm -> on premise
> kubefed -> federated
> For Master node min specs: 2 CPU, 4GB ram

- Two pods in a node can talk via <Pod_IP>:<port>. If pod dies IP get change for this __Service__ is used
- Service is a permanent IP address to pod
- Life cycle of service and pod is not connected
- For a application to be accessible outside the cluster __External Service__ needs to be configured
- External Service is a service that opens the communication from external sources (e.g Web app service)
- Internal Service is a service that don't access through external sources (e.g Db service)
- For user friendly url __Ingress__ is used. Request first goes to ingress then it forward to service
- Settings of external configurations/environment variables/endpoints __Config Map__ are used to avoid re-build of docker images for small change.
- __ConfigMap__ is for non-confidential data only
- __Secret__ is for confidential data only
- __Secret__ stored data in base64 format ```echo -n password | base64```
- configMap and secret are accessible via env variables or properties file
- __Volume__ persistent of data in case of loss physical storage(local/remote(outside cluster/sns/nas/cloud)) is used
- To avoid zero down time in case of disaster or application update distributed deployment is used
- Load Balancer type of service is used to distributed deployment structure where one service point to multiple pod
- To achieve distributed deployment a blue print of pods is used which is called __Deployment__
- __StatefulSet__ is used instead of __Deployment__ for stateful objects .i.e. where state lock is maintained like db service where two service access same database
- Configuration of stateFulSets is tedious thats why recommendation is to use database from outside the cluster

> Pod is layer of abstraction over containers and Deployment is layer of abstraction on pod
> Deployment: for stateless applications
> StatefulSet: for stateful applications

```sh
kubectl get all # view all the cluster k8s objects
kubectl logs <pod> # logs of container
kubectl describe svc webapp-service # for detailed output of serviceSohaib
kubectl get pods -o wide # for matching endpoints in above command with ip in this command
kubectl get node -o wide # check the ip of node for nodePort service type configuration

```

### Setup Kubernetes on AWS

#### Common commands for master and worker nodes

```sh
sudo su
apt-get update

# now install https package
apt-get install apt-transport-https

# this https package is needed for intra cluster communication(particularly from control plane to pods)

# now install docker
apt install docker.io -y #available in ubuntu 16.0 
systemctl start docker
systemctl enable docker

# setup an open GPG key. This is required for intra cluster communication. It will be added to source key on this node i.e. when k8s send signed info to our hosts, it is going to accept information because this open GPG key is present in the same source location

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

# now install kubernetes
# edit source list file by:
apt-get install nano
nano /etc/apt/sources.list.d/kubernetes.list
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
#exit from nano -> ctrl +x -> ctrl+y -> enter

apt-get update # install all packages
apt-get install -y kubelet kubeadm kubectl kubernetes-cni

```

#### BOOTSTRAPPING THE MASTER NODE (IN MASTER)

```sh
kubeadm init 
#you will get one long command started from "kubeadm join 172.31.6... ". Copy this command to run in nodes and save in notepad

# create both .kube and its parent directories(-p)
mkdir -p $HOME/.kube

#copy configuration to .kube directory(in config file)
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# provide user permissions to config file
chown $(id -u):$(id -g) $HOME/.kube/config

# deploy flannel node network for its repository path. Flannel is going to place a binary in each node
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

#### CONFIGURE WORKER NODES (IN NODES)

```sh
# COPY LONG CODE PROVIDED MY MASTER IN NODE NOW LIKE CODE GIVEN BELOW

e.g- kubeadm join 172.31.6.165:6443 --token kl9fhu.co2n90v3rxtqllrs --discovery-token-ca-cert-hash sha256:b0f8003d23dbf445e0132a53d7aa1922bdef8d553d9eca06e65c928322b3e7c0
```

##### GO TO MASTER AND RUN THIS COMMAND

```sh
kubectl get nodes
```



### Install Minikube on AWS

```yml

sudo su

# Now install docker

sudo apt update && apt -y install docker.io

# install Kubectl

curl -LO https://storage.googleapis.com/kubern... -s https://storage.googleapis.com/kubern... && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl

install Minikube

curl -Lo minikube https://storage.googleapis.com/miniku... && chmod +x minikube && sudo mv minikube /usr/local/bin/


#----------------------

kind: Pod                              
apiVersion: v1                     
metadata:                           
  name: testpod                  
spec:                                    
  containers:                      
    - name: c00                     
      image: ubuntu              
      command: ["/bin/bash", "-c", "while true; do echo Hello-Bhupinder; sleep 5 ; done"]
  restartPolicy: Never         # Defaults to Always

kubectl apply -f pod1.yml


#----------------------

MULTI CONTAINER POD ENVIRONMENT 

kind: Pod
apiVersion: v1
metadata:
  name: testpod3
spec:
  containers:
    - name: c00
      image: ubuntu
      command: ["/bin/bash", "-c", "while true; do echo Technical-Guftgu; sleep 5 ; done"]
    - name: c01
      image: ubuntu
      command: ["/bin/bash", "-c", "while true; do echo Hello-Bhupinder; sleep 5 ; done"]

#----------------------

POD ENVIRONMENT  VARIABLES


kind: Pod
apiVersion: v1
metadata:
  name: environments
spec:
  containers:
    - name: c00
      image: ubuntu
      command: ["/bin/bash", "-c", "while true; do echo Hello-Bhupinder; sleep 5 ; done"]
      env:                        # List of environment variables to be used inside the pod
      - name: MYNAME
        value: BHUPINDER

#----------------------

POD WITH PORTS

kind: Pod
apiVersion: v1
metadata:
  name: testpod4
spec:
  containers:
    - name: c00
      image: httpd
      ports:
       - containerPort: 80  
```

## Kubernetes Objects

### Labels and selectors

- Labels are the mechanism you use to organize k8s objects
- Label is a key-value pair without any pre-defined meaning that can be attached to the objects
- Label are similar to tags in AWS or git where you use a name to quick refrence
- Multiple label can be added to a single object
  
### [Replication Controller](workloads/workload-resources/replication-controller.md)

### [Replica Set](workloads/workload-resources/replicaset.md)

### [Deployment](workloads/workload-resources/deployment.md)

### [Services](services,load-balancing-and-networking/Services,Load-Balancing-and-Networking.md)

### [Volume](storage/storage.md)

### [Configurations](configuraitons/configMaps-and-secrets.md)

### [Namespace](working-with-kubernetes-objects/namespaces.md)

### [Resource Quota](policies/resource-quotas.md)

### [Job](workloads/workload-resources/job.md)

### [HELM](helm.md)

### Scheduling

- scheduling is done through labels

```yaml
kubectl label node k8s-2 disktype=ssd # output: node/k8s-2 labeled 
kubectl get nodes --show-labels=true
kubectl create -f webapp1.yaml # this file already contains 
# nodeSelector:
#   disctype: "ssd" 
kubectl get pods -o=wide
# all the pods are created on k8s-2 node

```

### Understand the role of DaemonSets

- DaemonSets are simply a way to run a single pod on each and every node in the cluster

```yaml
# by default it is running on master node as well. to avoid running on master comment below code on daemonset menifest file
spec:
 template:
  spec:
   tolerations:
   - key: node-role.kubernetes.io/master
     effect: NoSchedule
```

### K8s Logging and Monitoring

#### Monitor Cluster Component

- __Node Problem Detector__ : install via yaml file and create daemonSets that runs on every node tp monitor node health
- __Metrices-server__: must be enabled to collect metrices. kubectl top command is used

```yaml
kubectl top node --sort-by=memory
```

#### Monitor Applications

- __Liveness probe__ determine if container is running (internal to the container)
- __Readiness probe__ determine if the container is ready for service requests (external to the container)

#### Cluster component logs

- __Master__ - /var/log/ or /var/log/containers
  - kube-apiserver.log, kube-scheduler.log, kube-controller-manager.log
- __Workers__ - /var/log or /var/log/containers
  - kubelet.log, kube-proxy.log

#### Application Logs

