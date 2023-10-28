## KindでK8sクラスタを構築
---

### 1. ツールのインストール
#### 1.1 install kind from release binaries

参考リンク：
・https://kind.sigs.k8s.io/docs/user/quick-start/  
・https://github.com/kubernetes-sigs/kind

```
$ [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
```
`kind version`で結果（`kind v0.20.0 go1.20.4 linux/amd64`）が表示されたらkindインストールの成功になります。

#### 1.2 install docker-ce
参考リンク：
・https://docs.docker.jp/engine/installation/linux/docker-ce/ubuntu.html#id5
・https://qiita.com/TsuyoshiUshio@github/items/2a81a55bceed5fa11c4f

apt が HTTPS を通してリポジトリを使えるように、パッケージをインストールします。
```
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```
Docker の公式 GPG 鍵を追加します。
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
鍵の fingerprint（フィンガープリント）が `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88` と表示されるのを確認します。（！廃止されたのでいらないかも）
```
$ sudo apt-key fingerprint 0EBFCD88

Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```
以降のコマンドでは stable （安定版）リポジトリをセットアップします。もしも edge や testing リポジトリからビルドしたものをインストールしたい場合でも、常に stable リポジトリが必要です。 edge や testing リポジトリを追加するには、以降のコマンドの stable 文字のあとに edge か testing （あるいは両方）を追加します。
```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
docker CE のインストール
```
$ sudo apt-get update
$ sudo apt-get install docker-ce
```
`sudo docker run hello-world`でHello from Docker!が表示されましたらインストールの成功になります。

sudoしないとdockerコマンドが使えないのでsudoなしでも使えるようにする
```
$ docker ps
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied

# 自ユーザを docker グループに追加する
$ sudo usermod -aG docker $USER

# マシンを再起動
$ sudo shutdown -r now

# id コマンドで docker グループに入っていることを確認する
$ id
uid=1000(xie) gid=1000(xie) groups=1000(xie),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),999(docker)

$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS         PORTS                       NAMES
2d4ac030ba50   kind-image:modified   "/usr/local/bin/entr…"   31 minutes ago   Up 2 minutes   127.0.0.1:36813->6443/tcp   test-control-plane
```

#### 1.3 install Golang
参考リンク：
・https://go.dev/doc/install
・https://go.dev/dl/

remove any existing installation of Go 
```
$ sudo rm -r /usr/bin/go
$ sudo rm -r /usr/local/go
```
install with binaries
```
$ cd ~
$ wget https://dl.google.com/go/go1.21.3.linux-amd64.tar.gz
$ sudo tar -C /usr/local/ -xzf go1.21.3.linux-amd64.tar.gz go/
```
set environment variables for GO
```
$ export PATH=$PATH:/usr/local/go/bin
$ export GOPATH=${HOME}/go
$ export GOPATH_K8S=${HOME}/go/src/k8s.io/kubernetes
```
`go version`で結果（`go version go1.21.3 linux/amd64`）が表示されたらgoインストールの成功になります。

#### 1.4 clone Kubernetes
```
$ git clone https://github.com/<gituser>/kubernetes ${GOPATH_K8S}
$ cd ${GOPATH_K8S}
```

#### 1.5 install kubectl
参考リンク：
・https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

>ここでインストールするkubectlはホストからクラスタに接続しようとしているコマンドで、kind buildしてcreateされたクラスタないで生成されたのはクラスタ内部で使用するコマンドになりますので、ホストでインストールする必要があります。

```
$ curl -LO "https://dl.k8s.io/release/$(curl -LS https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
```
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
```
or 
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
```
$ kubectl version --client
#使う際にはsudo kubectlで
```

#### 1.6 install important dependencies
```
$ sudo apt-get update
$ sudo apt install make
```

### 2. Make A Cluster with kind
#### 2.1 build kubernetes image
```
$ cd ${GOPATH_K8S}
$ sudo kind build node-image ~/go/src/k8s.io/kubernetes --image kind-image:test
```
log:
```
Starting to build Kubernetes
+++ [1023 01:31:18] Verifying Prerequisites....
+++ [1023 01:31:19] Building Docker image kube-build:build-5c5fb3e2b8-5-v1.29.0-go1.21.3-bullseye.0
+++ [1023 01:31:38] Syncing sources to container
+++ [1023 01:31:45] Running build command...
+++ [1023 01:31:49] Building go targets for linux/amd64
    k8s.io/kubernetes/cmd/kube-apiserver (static)
    k8s.io/kubernetes/cmd/kube-controller-manager (static)
    k8s.io/kubernetes/cmd/kube-scheduler (static)
    k8s.io/kubernetes/cmd/kube-proxy (static)
    k8s.io/kubernetes/cmd/kubectl (static)
    k8s.io/kubernetes/cmd/kubeadm (static)
    k8s.io/kubernetes/cmd/kubectl (static)
    k8s.io/kubernetes/cmd/kubelet (non-static)
+++ [1023 01:34:39] Syncing out of container
+++ [1023 01:35:08] Building images: linux-amd64
+++ [1023 01:35:08] Starting docker build for image: kube-apiserver-amd64
+++ [1023 01:35:08] Starting docker build for image: kube-controller-manager-amd64
+++ [1023 01:35:08] Starting docker build for image: kube-scheduler-amd64
+++ [1023 01:35:08] Starting docker build for image: kube-proxy-amd64
+++ [1023 01:35:08] Starting docker build for image: kubectl-amd64
+++ [1023 01:36:18] Deleting docker image registry.k8s.io/kube-controller-manager-amd64:v1.25.0-alpha.3.9753_478c934c1ace78
+++ [1023 01:36:21] Deleting docker image registry.k8s.io/kubectl-amd64:v1.25.0-alpha.3.9753_478c934c1ace78
+++ [1023 01:36:26] Deleting docker image registry.k8s.io/kube-scheduler-amd64:v1.25.0-alpha.3.9753_478c934c1ace78
+++ [1023 01:36:39] Deleting docker image registry.k8s.io/kube-proxy-amd64:v1.25.0-alpha.3.9753_478c934c1ace78
+++ [1023 01:36:39] Deleting docker image registry.k8s.io/kube-apiserver-amd64:v1.25.0-alpha.3.9753_478c934c1ace78
+++ [1023 01:36:41] Docker builds done
Finished building Kubernetes
Building node image ...
Building in container: kind-build-1698025007-283647148
Image "kind-image:test" build completed.
```

#### 2.2 Make a yaml file for your cluster
```
$ mkdir -p ~/kindconfigs
$ cd ~/kindconfigs
$ vim cluster.yaml
```

(image list: https://github.com/kubernetes-sigs/kind/releases)
and put the following yaml inside
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-v1.28
nodes:
- role: control-plane
  image: kind-image:test
  kubeadmConfigPatches:
  - |
    kind: ClusterConfiguration
    apiServer:
      extraArgs:
        v: "10"
    scheduler:
      extraArgs:
        v: "10"
    controllerManager:
      extraArgs:
        v: "10"
```

#### 2.3 Create the cluster with your yaml
```
$ sudo kind create cluster --config cluster.yaml 
```
log:
```
Creating cluster "kind-v1.28" ...
 ✓ Ensuring node image (kind-image:test) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind-v1.28"
You can now use your cluster with:

kubectl cluster-info --context kind-kind-v1.28

Thanks for using kind! 😊

```
```
$ sudo kind get clusters
$ kubectl cluster-info --context kind-kind
```
```
# control-plane内部のコンテナ（K8sコンポーネント）を確認するには

$ sudo docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                       NAMES
9ad1f32f7923   kind-image:test   "/usr/local/bin/entr…"   43 minutes ago   Up 43 minutes   127.0.0.1:44321->6443/tcp   kind-v1.28-control-plane


# containerIDをコピー
9ad1f32f7923 

$ sudo docker exec -it 9ad1f32f7923 bash
root@kind-v1:/# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
b41a7a00c774f       ce18e076e9d4b       47 minutes ago      Running             local-path-provisioner    0                   4abdc64bf84d7       local-path-provisioner-6f8956fb48-fk742
e6685cad20282       cbb01a7bd410d       47 minutes ago      Running             coredns                   0                   a1efe19e1a414       coredns-58bc4d5885-smmx2
799f7a64dc557       cbb01a7bd410d       47 minutes ago      Running             coredns                   0                   c391909655f53       coredns-58bc4d5885-qcwn6
5f775e02c72e5       b0b1fa0f58c6e       47 minutes ago      Running             kindnet-cni               0                   8b0e8f5dc0be9       kindnet-8xrs7
d0be8179f8808       5165125f68c6e       47 minutes ago      Running             kube-proxy                0                   fcfaf7151fd7a       kube-proxy-kzrg9
4074dc6f3bf2b       73deb9a3f7025       48 minutes ago      Running             etcd                      0                   1d071715bc32b       etcd-kind-v1.28-control-plane
c8194a569c6db       059c6c77b3adc       48 minutes ago      Running             kube-controller-manager   0                   fff84c1b79437       kube-controller-manager-kind-v1.28-control-plane
4f115d8482a45       b17528eae78de       48 minutes ago      Running             kube-scheduler            0                   026b0020302f9       kube-scheduler-kind-v1.28-control-plane
77614c762b4bb       f0cc49b6fc380       48 minutes ago      Running             kube-apiserver            0                   a633baf55a371       kube-apiserver-kind-v1.28-control-plane
# docker内部のコンテナはcontainerdというランタイムで動いているので、crictlコマンドを使う
```
```
# logを確認するためには

$ cd ~
$ mkdir test1
$ cd test1/

$ sudo kind export logs . -n kind-v1.28
Exporting logs for cluster "kind-v1.28" to:
.

$ ls
docker-info.txt  kind-v1.28-control-plane  kind-version.txt
$ cd kind-v1.28-control-plane
$ ls
alternatives.log  containers  inspect.json  kubelet.log             pods
containerd.log    images.log  journal.log   kubernetes-version.txt  serial.log
# ここでlogファイルをいろいろ確認できる
```

#### 2.4 (if needed!)Delete clusters
```
kind delete cluster --name kind-v1.28
```
