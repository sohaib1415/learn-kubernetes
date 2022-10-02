# Storage

Ways to provide both long-term and temporary storage to Pods in your cluster.

## Storage Requirements

1. Storage doesn't depends on pod lifecycle
2. Storage must be available on all nodes
3. Storage needs to survive in case of cluster crashes

> For DB persistence use remote storage

## Volumes

All data stored inside a container. If container crashes, the data also get deleted. However kubelet will restart the container with clean state.
To overcome this problem, k8s uses `volumes`. A volume is essentially a directory backed by some storage medium which is defined by `volume type`

In k8s volumes are attached with pods and shared among the containers of that pod. If container delete the data remains preserved but if pods deletes volume get also deleted.

### Types of Volume

#### local type

- __EmptyDir__ [:link:](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)
  - An emptyDir volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that node.
  - use for intercommunication of containers within pod. Containers writes on same volume via mount path.
  - All Containers in the same Pod can read and write in the same emptyDir volume
  - When a Pod is restarted or removed, the data in the emptyDir is lost forever.
    - use-cases: When data persistent is not important some algorithm or mathematical computations  
      - scratch space, for a sort algorithm for example. 
      - when a long computation needs to be done in memory as a cache

  > Note: emptyDir volume should NOT be used for persisting data (database, application data, etc

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-emptyDir
  spec:
    - name: c1
    image: centos
    command: ["/bin/bash", "-c", "sleep 15000"]
    volumeMounts:     # Mount definition inside the container
      - name: xchange
        mountPath: "/tmp/xchange"          
  - name: c2
    image: centos
    command: ["/bin/bash", "-c", "sleep 10000"]
    volumeMounts:
      - name: xchange
        mountPath: "/tmp/data"
  volumes:                                                   
  - name: xchange
    emptyDir: {}
  ```

  ```yaml
  # exercise
  kubectl apply -f emptyDir.yaml 
  kubectl exec my-emptyDir -c c1 -it -- /bin/bash
  cd /tmp/xchange
  vi testfile

  kubectl exec my-emptyDir -c c2 -it -- /bin/bash
  cd /tmp/data
  ls # testfile exist
  ```

- __HostPath__ [:link:](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
  - use to access the pod/container from host machine
  - A hostPath volume mounts a file or directory from the node's filesystem into the Pod.
  - You can specify whether the file/directory must already exist on the node or should be created on pod startup. 
    - You can do it using a type attribute in the config file.
    - type: Directory defines that the directory must already exist on the host, so you will have to create it there manually first, before using the hostPath. 
    - Other values for type are DirectoryOrCreate, File, FileOrCreate. Where *OrCreate will be created dynamically if it doesn't already exist on the host.
  - __Disadvantages__
    - Node suitable if nodes are multiple
    - Pods created from the same pod template may behave differently on different nodes because of different hostPath file/dir contents on those nodes
    - Files or directories created with HostPath on the host are only writable by root. Which means, you either need to run your container process as root or modify the file permissions on the host to be writable by non-root user, which may lead to security issues
  > NOT use hostPath volume type for StatefulSets
  > Caution: The FileOrCreate mode does not create the parent directory of the file. If the parent directory of the mounted file does not exist

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-hostPath
  spec:
    containers:
    - image: my-app-image
      name: my-app
      volumeMounts:
        - mountPath: /temp/hostPath
          name: test-volume
    volumes:
    - name: test-volume 
      hostPath:
      path: /temp/data  #directory on host
      type: Directory #optional
  ```

  ```yaml
  # exercise
  kubectl apply -f hostPath.yaml 
  cd /temp/data
  echo "test" >testfile
  kubectl exec my-hostPath -c c1 -it -- ls /temp-hostPath # testfile shown
  ```

- __file sharing__ such as nfs
- __cloud provider specific__ like awselasticblockstore and azuredisk
- __distributed file system__ e.g glusterfs and cephfs
- __special purpose type__ like secret, gitrepo

### The problem

when a pod is re-created, the data (of a database application for example) is lost.

#### How to do it 👩🏻‍💻

So, you need to create and configure the actual physical storage and manage it by yourself.
3 different Kubernetes volume components, that you need to use to connect the actual physical storage to your pod, so that the application inside the container can access it.

### 3 Volume Components

#### Persistent Volume

- this is cluster level object means volume is accessible to all pods
- a cluster resource, like CPU or RAM, which is created and provisioned by administrators
- by creating persistent volumes you can use this actual physical storages. So in the persistent volume specification section you can define which storage back-end you want to use to create that storage abstraction or storage resource for applications

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myebsvol
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  awsElasticBlockStore: # storage back-end
    volumeID:           # YAHAN APNI EBS VOLUME ID DAALO
    fsType: ext4
```

##### Access modes for Volumes

- ReadWriteOnce(RWO): volume can be mounted as read-write by single node
- ReadOnlyMany(ROX): volume can be mounted read-only by many nodes
- ReadWriteMany(RWX): volume can be mounted as read-write by many nodes

> PV outside of the namespaces

#### Persistent Volume Claim

user's or pod's request for a persistent volume
![alt text](images/pvc.PNG "Logo Title Text 1") 

#### Storage Class

- A StorageClass is a Kubernetes resource that enables dynamic storage provisioning. The administrator configures the StorageClass, which can then no longer be modified.
- First the StorageClass is created, then the PersistentVolumeClaim and finally the Pod.
- When a PVC is created, Kubernetes creates a PersistentVolume and binds it to the PVC automatically, depending on the VolumeBindingMode used in the StorageClass configuration. These three Kubernetes objects are required to check the test case of a StorageClass.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: my-storageclass
provisioner: kubernetes.io/aws-ebs
volumeBidingMode: WaitForFirstConsumer
```

The apiVersion, kind, and metadata are common properties. The new, additional properties are:
Provisioner: This property is essential, and it must be specified. It is a field that determines which volume plugin should be used to provision the PVs. There are many volume plugins that are classified as either internal or external provisioners. (e.g., GCEPersistentDisk, Azuredisk, Azurefile, AWSElasticBlockStore, NFS, Flexvolume, Local, etc.). The first four use internal provisioner plugins, while the last three are external.
You can find more volume plugins and their provisioners here [Kubernetes official documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/).

__volumeBindingMode:__ This property has two values, Immediate and WaitForFirstConsumer.

1. __Immediate Mode:__ This mode involves the automatic volume binding of PVs to PVCs with a StorageClass once a PVC is created.
2. __WaitForFirstConsumer Mode:__ This mode will delay the binding and dynamic provisioning of PVs, until a Pod that will use it is created.

##### Multiple and different volume types in single pod

pod can actually use multiple volumes of different types simultaneously let's say you have an elasticsearch application or pod running in your cluster that needs a configuration file mounted through a config map needs a certificate let's say client certificate mounted as a secret and it needs database storage let's say which is backed with AWS elastic block storage so in this case you can configure all three inside your pod or deployment

To persist data and kubernetes:

1. admins need to configure storage for the cluster
2. create persistent volumes
3. developers then can claim them using PVCs

but consider a cluster with hundreds of applications
where things get deployed daily and storage is needed for these applications so developers need to ask admins to
create persistent volumes they need for applications before deploying them and admins then may have to manually request
storage from cloud or storage provider and create hundreds of persistent volumes for all the applications that need storage manually and that can be tedious time-consuming and can get messy very quickly.
So to make this process more efficient there is a third component of kubernetes persistence called
storage class.
It basically creates or provisions persistent volumes
dynamically whenever PVC claims it and this way creating or provisioning volumes in a cluster may be automated.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

example file where we have the kind storage class storage class creates
persistent volumes dynamically in the
background so remember we define storage
back-end in the persistent volume
component now we have to define it in
the storage class component and we do
that using the provisional attribute
which is the main part of the storage
class configuration because it tells
kubernetes which provisioner to be used
for a specific storage platform or cloud
provider to create the persistent volume
component out of it
so storage class is basically another abstraction level that abstracts the underlying storage provider as well as parameters for that storage characteristics for the storage
like what disk type or etc.

![alt text](images/storage-class-usage.PNG "Logo Title Text 1")

so now when a pod claims storage through PVC. The PVC will request that storage from storage class which then will provision or create persistent volume that meets the needs of that claim using provisioner from the actual storage back-end.

##### Provisioner [:link:](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner)

Each StorageClass has a provisioner that determines what volume plugin is used for provisioning PVs. This field must be specified.

##### Stages

1. Create a StorageClass
2. Create a PVC for storage request from StorageClass
3. Create a Pod to consume the claim by referencing the PVC in the Pod


#### ConfigMap and Secret as Volume Types

ConfigMap and Secret components can be used to:

- create individual key-value pairs, like db credentials or db url but also create configuration files, that can be mounted into the pod as volumes
- both of them are local volumes but unlike the rest these two aren't created by a PV and PVC but rather own components and managed by kubernetes itself.
- consider a case where you need a configuration file for your Prometheus pod or maybe a message broker service like mosquito or consider when you need a certificate file mounted inside your application in both cases you need a file available to your pod so how this works is that you create config map or secret component and you can mount that into your pod and into your container the same way as you would mount persistent volume claim.

```yaml
# exercise

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myebsvolclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: pvdeploy
spec:
  replicas: 1
  selector: # tells the controller which pods to watch to
    matchLabels:
     app: mypv
  template:
    metadata:
      labels:
        app: mypv
    spec:
      containers:
      - name: shell
        image: centos
        command: ["bin/bash", "-c", "sleep 10000"]
        volumeMounts:
        - name: mypd
          mountPath: "/tmp/persistent"
      volumes:
        - name: mypd
          persistentVolumeClaim:
            claimName: myebsvolclaim
```

### Health Check/Liveness Probe

- a pod is considered to be ready when all of its containers are ready
- in order tpo verify if a container in a pod is healthy and ready to serve traffic, k8s provides a health check mechanism
- Health checks carried out by the kubelet to determine when to restart a container for liveness probe and used by services and deployments to determine if a pod should recieve traffic
- For running health checks we use sommands specific to that application
- If the commands succeed its return 0 and kubelet consider the container to be alive and healthy
- If the commands fails its return a non-zero value, the kubelet kills the pod and recreate it

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: mylivenessprobe
spec:
  containers:
  - name: liveness
    image: ubuntu
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 1000
    livenessProbe:                                          
      exec:
        command:                                         
        - cat                
        - /tmp/healthy
      initialDelaySeconds: 5          
      periodSeconds: 5                                 
      timeoutSeconds: 30       
```