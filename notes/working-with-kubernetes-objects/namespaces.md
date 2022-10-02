# Namespaces

- Namespace is a group of related elements that each hace a unique name or identifier
- Namespace is used to uniquely identifying one or more names from other similar names of different ibhects, groups or the namespace in general
- k8s namespace help different project teams or customer to share a k8s cluster and provides
- a scope for every names
- a mechanism to attach authorization and policy to a subsection of the cluster
- we can define resource quota on specifying gow many resources eac namespace can use
- most k8s resources (e.g pods, services,rc, and others) are in some namespaces and low level resources such as nodes and pv are not in any namespace
- ns are intended for use in environments with many users spread across multiple teams or projects. for cluster with a few to tens of users you should not need to create or think about namespaces and all

```yaml
apiVersion: v1
kind: Namespace
metadata:
   name: dev
   labels:
     name: dev

---
# vi pod.yml

kind: Pod                              
apiVersion: v1                     
metadata:                           
  name: testpod                  
spec:                                    
  containers:                      
    - name: c00                     
      image: ubuntu              
      command: ["/bin/bash", "-c", "while true; do echo Technical Guftgu; sleep 5 ; done"]
  restartPolicy: Never       
---
kubectl apply -f devns.yaml
kubectl apply -f pod.yaml -n dev
$ kubectl config set-context $(kubectl config current-context) --namespace=dev
$ kubectl config view | grep namespace: dev

---
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
