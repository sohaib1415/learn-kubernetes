# Deployment

- Deployment talks to pod via replicaSet not directly to pods
- Deployment object act as a supervisor for pods, giving you fine-grained control over how and when a new pod is rolled out, updated and roll back to a previous state

## Use cases for Deployments

- __Create a deployment to rollout a replicaSet__
- __Declare the new state of pod__ by updating the PodTemplateSpec of the deployment
- __Rollback to an earlier deployment revision__
- __Scale up/down__
- __Pause/resume the deployment to apply multiple fixes__
  
If there are problem in deployment k8 will automatically rollback to previous version, however you can also rollback to specific version using --to-revision
e.g kubectl rollout undo deploy/mydeployment --to-revision=2

> The name of the replicaSet is always formatted as [DeploymentName-Random String]

```yaml
kubctl create deploy webapp1 --image=nginx:1.16-alpine-perl --dry-run -o yaml > webapp1.yaml
cat webapp1.yaml
kubectl create -f webapp1.yaml
kubectl get deploy -o=wide
kubectl describe deploy/mydeployment # step wise how deployment create replicaSets and pods
kubectl logs -f <pod> # check what is running inside container
kubectl exec <pod> -- cat /etc/os-release
```

## Scaling Up Deployments

```yaml
kubectl scale --replicas=30 deploy webapp1
```

## Perform Rollout and Rollbacks

- In case of Rollback no. of pods remains same e.g v1 contains 2 pods and v2 contains 3 pods then after rollback from v2 to v1 no. of pods remains 3  

```yaml
cp webapp1.yaml webapp2.yaml
nano webapp2.yaml # update docker image version from nginx:1.16-alpine-perl to nginx:1.17-alpine-perl
kubectl create -f webapp2.yaml # output: deployment.apps/webapp1 configured means it roll out from nginx:1.16-alpine-perl to nginx:1.17-alpine-perl
kubectl rollout status deploy webapp1 # deployment "webapp1" successfully rolled out
kubectl rollout history deploy webapp1 

kubectl rollout undo deploy webapp1 # output: deployment.apps/webapp1 rolled out
kubectl rollout undo deploy/webapp1 --to-revision=2 # go to specific revision
```