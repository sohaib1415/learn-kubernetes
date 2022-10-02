
# Services, Load Balancing, and Networking

Concepts and resources behind networking in Kubernetes.

> create service via command of already created deployment kubectl expose deploy webapp1 --port=80

### K8s Networking

k8s networking addresses four concerns:

- containers with in pod use networking to communicate via loopback: containers with same pod communicate through localhost:<container_port>
- communication between different pods: Pod to pod communication is done through <pod_ip>:<container_port> & by default pod ip does not accessible outside the node
- the service resources let you expose an application running in pods to be reachable from outside your cluster
- you can also use services to publish services only for consumption inside your cluster

### Services

- It is logical bridge between pods and end user which provides virtual IP(VIP)
- Services allows user connect to container through VIP
- VIP is not actual IP but it purpose is purely forward the traffic to one or more pods
- Kube-proxy is the one which keeps the mapping between VIP and pods up-to-date
- Labels are used to select pods used against service
- Creating a service will create an endpoint to access the pods/applications in it
- Set of pods that are targeted by service is usually determined by selector
- valid ports for services are between 30000 and 32767

#### Service Types

- __ClusterIP__ exposes VIP only reachable within cluster and mainly used to communicate between components of micro services
- __NodePort__ make a service accessible from outside the cluster; exposes the service on same port of each selected node in the cluster using NAT 
- __Load Balancer__ Load balancer created by cloud providers that will route external traffic to every node on the nodePort.eg ELB on AWS
- __Headless__ Create several endpoints that are used to produce DNS records. Each DNS record is bound to a pod

> create service via command of already created deployment kubectl expose deploy webapp1 --port=80