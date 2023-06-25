# k3d-skaffold-kustomize-demo 

An example local [k3s](https://github.com/rancher/k3s) development environment using [kustomize](https://github.com/kubernetes-sigs/kustomize), [skaffold](https://github.com/GoogleContainerTools/skaffold) and [k3d](https://github.com/rancher/k3d). 

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [Features](#features)
* [Prerequisites](#prerequisites)
* [Usage](#usage)
* [Kustomize configuration](#kustomize-configuration)


<!-- FEATURES -->
## Features
- Bootstraps k3s cluster in Docker using k3d
- Skaffold loads docker images directly into the k3s cluster
- Skaffold uses kustomize for building and deploying k8s manifests using [local-dev](#kustomize-directory-structure-based-layout) overlay. 
- An example `node.js` app will be bootstrapped with [File sync](https://skaffold.dev/docs/filesync/) and [Port forward](https://skaffold.dev/docs/pipeline-stages/port-forwarding/) enabled

<!-- PREREQUISITES -->
## Prerequisites
> **NB.** The setup is tested on `MacOS & Linux with brew installed`.

The first prerequisite is to install go-task in order to make the setup a bit easier:

```sh
### MacOS
brew install go-task/tap/go-task

### Linux
$ sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
```

The following prerequisites are used in order to create and manage the local K3s cluster:

- [Docker](https://docs.docker.com/engine/install/ubuntu/) or [Docker Desktop](https://docs.docker.com/get-docker/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)
- [Skaffold](https://skaffold.dev/docs/getting-started/#installing-skaffold)
- [K3d](https://github.com/rancher/k3d)
- [Go Task](https://taskfile.dev/installation) (Note: Go Task as a more modern iteration of the Makefile utility)

If you don't have them installed yet you can install them using install-prerequisites task:

```sh
task install-prerequisites
```

<!-- USAGE -->
## Usage
Create k3s cluster:
> **NB** If you want to change the amount of k3s agents argument e.g. `k3d:create-cluster -- <number_of_agents>`
```sh
$ task k3d:create-cluster
```
Make sure your KUBECONFIG points to k3s cluster context (if not already):
```sh
$ kubectl get nodes
```
Start the local development environment:
```sh
$ task skaffold:dev
```
An example node.js app is available at:
```sh
localhost:3000

$ curl localhost:3000
Hello World!
```
Make some changes to `src/index.js` and they will be synchronized to the pod(s) running the app.

Delete the k3s cluster:
```sh
task k3d:delete-cluster
```
Delete images that are built by Skaffold and stored on the local Docker daemon:
```sh
task docker:rmi
```

<!-- KUSTOMIZE CONFIGURATION -->
## Kustomize configuration
Kustomize configuration is based on `Directory Structure Based Layout` in order to use multiple environments with different configuration. In order to use different clusters remember to specify the corresponding context before applying changes using Skaffold.
```sh
├── base
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── local-dev
    │   ├── deployment-patch.yaml
    │   ├── hpa-patch.yaml
    │   ├── kustomization.yaml
    ├── prod
    │   ├── deployment-patch.yaml
    │   ├── hpa-patch.yaml
    │   └── kustomization.yaml
    └── staging
        ├── deployment-patch.yaml
        ├── hpa-patch.yaml
        └── kustomization.yaml        

        
```
Note: kustomize for prod and staging, need changes of skaffold.yaml file -> kubeContext & (OPTIONAL) portForward: resourceName: if using kustomize EXAMPLE 2 , kustomization files, ETC.
