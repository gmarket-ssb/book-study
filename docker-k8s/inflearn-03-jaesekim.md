## VM 환경에 Cluster 구성하기(우분투 20.04)

### 고정 IP 변경

```bash
# 관리자 권한으로 변경
user01@user01:~$ sudo -i

root@user01:~# ip addr

# 고정 IP로 변경, 우분투에서는 netplan을 사용해서 ip를 변경한다.
root@user01:~# vim /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      dhcp4: no
      dhcp6: no
      addresses: [192.168.138.128/24]
      gateway4: 192.168.138.2 # vmware 사용시 gw가 2번으로 열려있음
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
  version: 2

# 셋팅한 netplan 적용
root@user01:~# netplan apply
```

### 마스터 노드, 워커 노드 공통사항 설치

```bash
# 도커와 kubeadm 설치
# 1. 도커 설치, 도커를 설치하면 컨테이너 런타임(containerd)이 설치됨
# 파드에서 컨테이너를 실행하기 위해, 쿠버네티스는 컨테이너 런타임을 사용한다.
root@user01:~# apt update && apt install docker.io -y

# 스왑 기능 제거, 바로 스왑기능을 제거 하고 리부트한 이후에도 계속 적용하기 위해 아래 두가지 명령어 모두 실행
# 바로 적용, 리부트한 이후 적용 X
root@user01:~# sudo swapoff -a
# 리부트한 이후부터 계속 적용
root@user01:~# sudo sed -i '/ swap / s/^/#/' /etc/fstab

# 1.22 버전부터 kubelet 데몬에서 사용하는 컨테이너 그룹명과 도커의 그룹명이 일치하지 않아서 문제가 발생한다.
# cgroupfs를 systemd 로 변경해야 한다.
# ref. https://stackoverflow.com/questions/43794169/docker-change-cgroup-driver-to-systemd
root@user01:~# docker info | grep -i group
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
WARNING: No swap limit support

root@user01:~# vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

root@user01:~# service docker restart
root@user01:~# docker info | grep -i group
 Cgroup Driver: systemd
 Cgroup Version: 1
WARNING: No swap limit support

root@user01:~# vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

root@user01:~# service docker restart
root@user01:~# docker info | grep -i group
 Cgroup Driver: systemd
 Cgroup Version: 1
WARNING: No swap limit support
```

### 이후 셋팅 에러시 원복을 위해 해당 이미지 스냅샷 생성

### 마스터 노드, 워커 노드 공통사항 설치 셋팅된 이미지로 node0, node1 생성

### 마스터 노드 호스트 네임 변경

```bash
# 호스트이름 변경
# 노드 ID로 호스트이름을 사용 (노드가 같은 호스트이름이면 검색 시 하나만 나옴)
root@user01:~# hostnamectl -h
root@user01:~# hostnamectl set-hostname master0
# 아래까지 해야 sudo 명령어 사용시 지연시간이 없음
root@user01:~# vim /etc/hosts
127.0.0.1 localhost
127.0.1.1 master0 # user01 -> master0 이름 변경

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

root@user01:~# exit
logout
user01@user01:~$ sudo -i
[sudo] password for user01: 
# 이름변경 확인
root@master0:~#
```

### node0, node1도 고정 IP로 변경 및 호스트네임 설정

### 마스터노드에 kubeadm 설치하기

설정을 리셋(kubeadm reset)해야할 경우에는 $HOME/.kube/config를 삭제한 후 진행해야 한다. 그렇지 않으면 인증서 관련 에러가 발생한다.

워커노드에서 에러나서 kubeadm reset할 경우는 그냥 해도 되는것같음

```bash
# master 노드에서 kubeadm init
# static pod 등 마스터 노드에 필요한 기능을 셋팅한다. (마스터 노드 활성화)
# init 이 성공하면 아래와 같은 메시지를 출력
# 1. regular user 등록방법
# 2. pod network 를 구성하는 CNI를 구성하는 방법
# 3. 워커노드를 join하는 방법
root@master0:~# kubeadm init
[init] Using Kubernetes version: v1.25.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master0] and IPs [10.96.0.1 192.168.138.128]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost master0] and IPs [192.168.138.128 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost master0] and IPs [192.168.138.128 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.004929 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master0 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node master0 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: e0rv69.i1xdsf2ug2axmlef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.138.128:6443 --token e0rv69.i1xdsf2ug2axmlef \
        --discovery-token-ca-cert-hash sha256:b39c4fb8f1fb74ea1ddde0964b4684793f6a8547f308f41acced10b542bab1b9

# regular user 등록 후 kubectl 명령어가 정상동작하게 된다
root@master0:~#   mkdir -p $HOME/.kube
root@master0:~#   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@master0:~#   sudo chown $(id -u):$(id -g) $HOME/.kube/config
root@master0:~# 
root@master0:~# 
root@master0:~# kubectl get node
NAME      STATUS     ROLES           AGE     VERSION
master0   NotReady   control-plane   4m13s   v1.25.0
```

### node0, node1 워커노드에서 join 수행

```bash
# 각 node에서 join 명령 수행
root@node0:~# kubeadm join 192.168.138.128:6443 --token axfikx.7fshk2maixxcglvn \
>         --discovery-token-ca-cert-hash sha256:67999421b79d609d77b7777e5c21b439e410f6e82b2abe420aa4b449650f85c9

root@node1:~# kubeadm join 192.168.138.128:6443 --token axfikx.7fshk2maixxcglvn \
>         --discovery-token-ca-cert-hash sha256:67999421b79d609d77b7777e5c21b439e410f6e82b2abe420aa4b449650f85c9

root@master0:~# kubectl get node
NAME      STATUS     ROLES           AGE     VERSION
master0   NotReady   control-plane   6m3s    v1.25.0
node0     NotReady   <none>          2m37s   v1.25.0
node1     NotReady   <none>          104s    v1.25.0
```

### weavenet 설치

```bash
# weavenet 설치
root@master0:~# kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

# 1분정도 후에 READY로 변경됨
root@master0:~# kubectl get node
NAME      STATUS   ROLES           AGE     VERSION
master0   Ready    control-plane   8m7s    v1.25.0
node0     Ready    <none>          4m41s   v1.25.0
node1     Ready    <none>          3m48s   v1.25.0

```
