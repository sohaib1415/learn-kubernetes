# Pod

- smallest unit in k8s
- group on one or more containers deployed together on same host
- a cluster is groups of nodes
- in k8s control unit is pod not containers
- consists of one or mote tightly coupled containers
- pods run on node which is control by master
- cannot start container without a pod

## Multi container pods

- share access to memory space
- connect to each other via localhost:<container_port>
- share access to the same volume
- container with in pod are deployed in all-or-nothing manner
- entire pod is hosted on the same node(schedular will decide which node)
- no auto-healing or auto-scaling(by default)
- pods crashes

### Init containers

- init containers are specialized container that eub before app containers in a pod
- init container always run to completion
- of a pods init container fails, k8s repeatedly restarts the pod unit the init container succeeds
- init containers do not support readiness probe

#### Use Cases

- seeding a db(creting tables and column before feeding data)
- delaying the application launch until the dependencies are ready
- clone a git repo into a volume
- generate configuration file dynamically
  
```yaml
#init.yml

```

## Pod lifecycle

pending -> running -> succeeded -> failed -> completed -> unknown

- the phase of a pod is a simple, high-level summary of where the pod is inits lifecycle
  
- __pending__
  - the pod has been accepted by the k8s system, but its not running
  - one or more of the container images is still downloading
  - if the pod can not be scheduled because of resource constraints
- __running__
  - the pod has been bound to a node
  - all of the containers have been created
  - at least one container is still running or is in the process of starting or restarting
- __succeeded__
  - all containers in the pod have terminated in success and will not be restarted
- __failed__
  - all containers in the pod have terminated and at least one container has terminated in failure
  - the container either exited with non-zero status or was terminated by the system
- __unknown__
  - state of the pod could not be obtainer
  - typically due to an error in network or communicating with the host of the pod
- __completed__
  - the pod has run to completion as there is nothing to keep it running. e.g completed jobs

### Pod conditions

- A pod has a pod status, which has an array of pod conditions through which the pod has or has not passed
- using `kubectl describe pod <pod_name>` you can get conditions of pod. Following are the types:

- __Pod scheduled__
  - pods has been scheduled to a node
- __Ready__
  - the pod is able to serve request and will be added to the load balancing pods if all matching services
- __Initialized__
  - all init containers have started successfully
- __Unscheduled__
  - the schedular cannot schedule the pod right now, e.g due to lacking of resources or other constraints
- __Container Ready__
  - all containers in the pod are ready