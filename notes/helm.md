# HELM(package manager)

- package manager tasks:
  - automate installation
  - version management
  - dependency management (like jenkins needs jdk and python needs for ansible)
  - automate un-installation

- Introduced first time in 2015
- Helm 2 (client-server(tiller) architecture) because lacking of RBAC in k8s
- Helm 3 client only
- Helm helps you manage k8s applications with `Helm Charts`, which helps you define, install and upgrade even the most complex k8s applications
- helm is the k8s equivalent of yums in centos or apt in ubuntu
- helm is now an official k8s project and is part of CNCF
- the main building block of helm based deployments are helm charts. These charts describe a configurable set of dynamically generated k8s resources
- the chart can either be stored locally or fetched from remote chart repositories
- helm written in `Golang` language
  
## Why use HELM?

- Writing and maintaining k8s yaml manifest for all required k8s objects can be a tedious and time consuming
- For a simplest deployments, you would need at least 3 yaml files with duplicated and hardcoded values. Helm simplifies this process and create a single package that can be advertised to your cluster
- `Helm 3` was release in Nov. 2019
- Helm k8s automatically maintains a database of all versions of your release, so whenever something goes wrong during deployment, rolling back to tge previous versions is just one command away

### Some keywork to understand Helm

__Chart__ A chart is a helm package. It contains all of the resources definitions necessary to run an application, tool or service inside of a k8s cluster
OR
Helm charts are simply k8s yaml manifest combined into a single package that can be advertized to your k8s cluster

__Release__ A release is an instance of a chart running in a k8s cluster. One chart can oftten be onstalled many times into the same cluster and each time it is installed a new release is created

consider a mysql chart, if you can install that chart twice each one will have its own release; which will in turn have its own release name

| Revision  | Request         |
| --------- |:---------------:|
| 1         | installed chart |
| 2         | upgrade to 1.x  |

> helm keep tracks of all chart execution(install/upgrade/rollback)

__Repository__ location where package charts can be stored. There are two types:

1. Local Repository
2. Remote Repository(Helm hub)

## HELM 3 Architecture

- Helm-3 is a single-service/client architecture. One executable is responsible for implementing helm, this is no client-server split, nor is the core processing logic distributed among components.
- Implementation of Helm 3 is a single command line, client with no in-cluster server or controller. This tool expose command line operations and unilaterally handles the package management process
  
## Helm commands and concepts

__helm repo__: interact with charts repo. Helm-3 no longer ships with a default chart repository

```sh
helm repo list
helm repo add <name><url>
helm repo remove <name>
helm search # for finding charts for e.g helm search repo <chart>
helm show # information of the chart e.g helm show <values|chart|readme|all><chart_name>
helm install <release name><chart name> # install a package

# wait until subjects are ready e.g helm install mychart stable/tomcat --wait --timeout 10s
helm create <chart name>

```

```sh
To download helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# To verify the installation use the following command
which helm


# Let’s create Our First Helm Chart


helm repo add stable https://charts.helm.sh/stable
helm search repo jenkins
helm show values stable/jenkins # jenkins repo name

helm install testchart2 stable/tomcat --set service.type=NodePort

# to uninstall helm
# which helm ( to see which folder its installed )
rm -rf /usr/local/bin/helm


helm create helloworld
ls
tree helloworld
rm -rf helloworld # remove helloworld directory

helm install testjenkins stable/jenkins
helm install testjenkins2 stable/jenkins
helm install testtomcat stable/tomcat
kubectl get all
```

There are two ways to pass configuration data during install

1. __-- set__ specify overrides on the command line
2. __-- values(or `r)__ specify yaml file with overrides line

```sh
helm get # info after install chart
hel get <all|manifest|values><release name>
helm get manifest <chart name>
helm status   # display the status of the named release
helm history  # fetch release history
helm delete   # uninstall deployed release
helm upgrade  # upgrade release
helm rollback # rollback a release to any previous version
helm pull     # download a chart from a repository but not install
helm pull --untar <chart name>
# install from loca archive
helm install mychart tomcat-0.4.3.tgz
# install from an unpacked chart
helm install mychart
# install from url
helm install mychart <url>

```

### HELM hub 

https://artifacthub.io/
