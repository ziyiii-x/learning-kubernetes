### Download minikube
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
$ chmod +x ./minikube-darwin-amd64
$ sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```
[minikube download method](https://minikube.sigs.k8s.io/docs/start/)
### Download kubectl
```
$ curl -LO "https://dl.k8s.io/release/v1.27.0/bin/darwin/arm64/kubectl"
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl 
$ sudo chown root: /usr/local/bin/kubectl
```
[kubectl download method](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)
### Open docker
open the docker desktop and follow the instruction
```
$ softwareupdate --install-rosetta
$ sudo hdiutil attach Docker.dmg
$ sudo /Volumes/Docker/Docker.app/Contents/MacOS/install
$ sudo hdiutil detach /Volumes/Docker
```
[docker setup method](https://docs.docker.com/desktop/install/mac-install/)
### Start cluster and check info
```
$ minikube start --driver=docker
$ kubectl cluster-info
```
```
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   default
```
### When minikube goes wrong
```
$ docker system prune
$ minikube delete
$ minikube start --driver=docker
```
