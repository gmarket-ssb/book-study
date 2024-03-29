## 노드 OS 업그레이드

커널 업그레이드, libc 업그레이드, 하드웨어 복구 등 노드를 재부팅해야 하는 상황이 있다. 노드를 갑자기 내리면 노드가 장애난 것을 인지하고 다른 노드로 파드을 옮기는 시간(default: 5분)동안 장애가 발생하게 된다. 따라서 아래 절차와 같이 노드를 계획성 있게 내려야 한다.

1. drain node
    - 해당 노드에 있는 파드들을 다른 노드로 이동(배수)시킨다. 만약 다른 노드에 자원이 없다면 pending 상태가 된다.
    - cordon node 명령어가 함께 실행된다. 해당 노드에 POD이 들어올 수도 나갈 수도 없게 (저지선을) 만든다. 해당 노드로 스케쥴링되지 않는다. (SchedulingDisabled)
2. update node
    - 노드를 업그레이드 한다. 1번 실행 시 해당 노드에는 파드가 없기 때문에 안정적으로 업그레이드 가능하다.
3. uncordon node
    - 해당 노드에 다시 파드들이 스케쥴링될 수 있다. 노드를 내릴 때 자원을 할당받지 못한 pending 상태의 파드들은 해당 노드로 배치된다.

### 예제
```bash
# 파드 이동 확인을 위해 deployment 생성
kubectl create deploy http-spring --image=jaesay/http-spring --replicas=10

# 노드별 각각 5개 씩 배치
root@master0:~# kubectl get all -o wide
NAME                               READY   STATUS              RESTARTS   AGE   IP          NODE    NOMINATED NODE   READINESS GATES
pod/http-spring-766c586cb9-2h8rs   1/1     Running             0          8s    10.40.0.5   node1   <none>           <none>
pod/http-spring-766c586cb9-2mcjm   1/1     Running             0          8s    10.32.0.4   node0   <none>           <none>
pod/http-spring-766c586cb9-5qhbf   1/1     Running             0          43s   10.40.0.1   node1   <none>           <none>
pod/http-spring-766c586cb9-8dkzb   0/1     ContainerCreating   0          8s    <none>      node1   <none>           <none>
pod/http-spring-766c586cb9-9k5bp   0/1     ContainerCreating   0          8s    <none>      node0   <none>           <none>
pod/http-spring-766c586cb9-9v5ld   0/1     ContainerCreating   0          8s    <none>      node1   <none>           <none>
pod/http-spring-766c586cb9-t9vxh   0/1     ContainerCreating   0          8s    <none>      node0   <none>           <none>
pod/http-spring-766c586cb9-tfqzt   1/1     Running             0          8s    10.40.0.3   node1   <none>           <none>
pod/http-spring-766c586cb9-tr8wf   0/1     ContainerCreating   0          8s    <none>      node0   <none>           <none>
pod/http-spring-766c586cb9-xn9lf   1/1     Running             0          8s    10.32.0.2   node0   <none>           <none>

# drain node0
root@master0:~# kubectl drain node0 --ignore-daemonsets
node/node0 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-tl5w2, kube-system/weave-net-jpd2x
evicting pod default/http-spring-766c586cb9-t9vxh
evicting pod default/http-spring-766c586cb9-2mcjm
evicting pod default/http-spring-766c586cb9-9k5bp
evicting pod default/http-spring-766c586cb9-tr8wf
evicting pod default/http-spring-766c586cb9-xn9lf
evicting pod ingress-nginx/ingress-nginx-controller-6d74cfb66b-vck99
pod/http-spring-766c586cb9-tr8wf evicted
pod/http-spring-766c586cb9-9k5bp evicted
pod/http-spring-766c586cb9-xn9lf evicted
pod/http-spring-766c586cb9-2mcjm evicted
pod/http-spring-766c586cb9-t9vxh evicted
pod/ingress-nginx-controller-6d74cfb66b-vck99 evicted
node/node0 drained

# kube-proxy와 weave-net은 여전히 node0에 남아 있음, 굳이 빼낼 필요 없어서 그런것같다고 함
root@master0:~# kubectl get pod --all-namespaces -o wide
NAMESPACE       NAME                                        READY   STATUS    RESTARTS      AGE     IP                NODE      NOMINATED NODE   READINESS GATES
default         http-spring-766c586cb9-2h8rs                1/1     Running   0             4m46s   10.40.0.5         node1     <none>           <none>
default         http-spring-766c586cb9-4dvtr                1/1     Running   0             95s     10.40.0.7         node1     <none>           <none>
default         http-spring-766c586cb9-5qhbf                1/1     Running   0             5m21s   10.40.0.1         node1     <none>           <none>
default         http-spring-766c586cb9-7jn47                1/1     Running   0             95s     10.40.0.10        node1     <none>           <none>
default         http-spring-766c586cb9-8dkzb                1/1     Running   0             4m46s   10.40.0.4         node1     <none>           <none>
default         http-spring-766c586cb9-9v5ld                1/1     Running   0             4m46s   10.40.0.2         node1     <none>           <none>
default         http-spring-766c586cb9-ls8bw                1/1     Running   0             95s     10.40.0.11        node1     <none>           <none>
default         http-spring-766c586cb9-pknlh                1/1     Running   0             95s     10.40.0.9         node1     <none>           <none>
default         http-spring-766c586cb9-rg86p                1/1     Running   0             95s     10.40.0.8         node1     <none>           <none>
default         http-spring-766c586cb9-tfqzt                1/1     Running   0             4m46s   10.40.0.3         node1     <none>           <none>
ingress-nginx   ingress-nginx-controller-6d74cfb66b-l9rlh   1/1     Running   0             95s     10.40.0.6         node1     <none>           <none>
kube-system     coredns-565d847f94-dt76p                    1/1     Running   0             41m     10.38.0.2         master0   <none>           <none>
kube-system     coredns-565d847f94-xdkcg                    1/1     Running   0             41m     10.38.0.1         master0   <none>           <none>
kube-system     etcd-master0                                1/1     Running   0             47m     192.168.138.128   master0   <none>           <none>
kube-system     kube-apiserver-master0                      1/1     Running   0             47m     192.168.138.128   master0   <none>           <none>
kube-system     kube-controller-manager-master0             1/1     Running   0             46m     192.168.138.128   master0   <none>           <none>
kube-system     kube-proxy-gfs28                            1/1     Running   0             46m     192.168.138.128   master0   <none>           <none>
kube-system     kube-proxy-sbnfh                            1/1     Running   0             7m36s   192.168.138.130   node1     <none>           <none>
kube-system     kube-proxy-tl5w2                            1/1     Running   0             35m     192.168.138.129   node0     <none>           <none>
kube-system     kube-scheduler-master0                      1/1     Running   0             46m     192.168.138.128   master0   <none>           <none>
kube-system     weave-net-4sqbt                             2/2     Running   1 (77d ago)   77d     192.168.138.128   master0   <none>           <none>
kube-system     weave-net-jpd2x                             2/2     Running   1 (77d ago)   77d     192.168.138.129   node0     <none>           <none>
kube-system     weave-net-mgxj9                             2/2     Running   1 (77d ago)   77d     192.168.138.130   node1     <none>           <none>

# node0는 cordon 명령어로 인해 SchedulingDisabled
root@master0:~# kubectl get node
NAME      STATUS                     ROLES           AGE   VERSION
master0   Ready                      control-plane   77d   v1.25.4
node0     Ready,SchedulingDisabled   <none>          77d   v1.25.4
node1     Ready                      <none>          77d   v1.25.0

# 필요한 업데이트 진행 (e.g. 쿠버네티스 버전 업데이트)

# node0에 다시 스케쥴링 되도록 uncordon 진행
root@master0:~# kubectl uncordon node0
node/node0 uncordoned

root@master0:~# kubectl get node
NAME      STATUS   ROLES           AGE   VERSION
master0   Ready    control-plane   77d   v1.25.4
node0     Ready    <none>          77d   v1.25.4
node1     Ready    <none>          77d   v1.25.0

# node1에 자원이 충분하여 pending된 파드들이 없기 떄문에 node0로 재이동하지는 않음
root@master0:~# kubectl get pod -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE   READINESS GATES
http-spring-766c586cb9-2h8rs   1/1     Running   0          14m   10.40.0.5    node1   <none>           <none>
http-spring-766c586cb9-4dvtr   1/1     Running   0          11m   10.40.0.7    node1   <none>           <none>
http-spring-766c586cb9-5qhbf   1/1     Running   0          15m   10.40.0.1    node1   <none>           <none>
http-spring-766c586cb9-7jn47   1/1     Running   0          11m   10.40.0.10   node1   <none>           <none>
http-spring-766c586cb9-8dkzb   1/1     Running   0          14m   10.40.0.4    node1   <none>           <none>
http-spring-766c586cb9-9v5ld   1/1     Running   0          14m   10.40.0.2    node1   <none>           <none>
http-spring-766c586cb9-ls8bw   1/1     Running   0          11m   10.40.0.11   node1   <none>           <none>
http-spring-766c586cb9-pknlh   1/1     Running   0          11m   10.40.0.9    node1   <none>           <none>
http-spring-766c586cb9-rg86p   1/1     Running   0          11m   10.40.0.8    node1   <none>           <none>
http-spring-766c586cb9-tfqzt   1/1     Running   0          14m   10.40.0.3    node1   <none>           <none>
```


## 쿠버네티스 버전 업데이트

v{MAJOR}.{MINOR}.{PATCH}

- e.g. v.1.14.1
- MAJOR
- MINOR: 특징, 기능 (API 버전명이 달라지는 경우도 있다.)
- PATCH: 버그 픽스

모든 버그를 고치기 위해 alpha와 beta로 나누어 출시 후 정식 버전 진행

- alpha: 추가되는 기능들을 disable한 형태로 배포
- beta: 추가 기능들을 enable한 형태로 배포
- release: 안정화된 버전을 배포

패키지마다 모든 컴포넌트는 동일한 버전으로 배포

- API 서버, 컨트롤러 매니저, 스케줄러, 큐블렛, 큐브프록시 등
- ETCD, 네트워크, CoreDNS 등 외부 프로젝트는 별도로 관리

보통 apiserver 기준으로 -1 마이너버전까지 호환이 된다. 따라서 마스터의 마이너 버전을 하나올리고 워커노드의 마이너버전을 하나 올리면 된다.

예제

- [https://blog.naver.com/isc0304/222506604980](https://blog.naver.com/isc0304/222506604980) (vm에 구성된 cluster를 통해 진행)

## 백업과 복원
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

`kubectl get all` 명령어를 통해 yaml 파일을 백업하고 그 외 정보(configmap, secret, pvc 등)은 etcd db의 스냅샷을 통해 백업한다.

pv 등은 일반적인 방법으로 백업

### 예제

```bash
# 파드의 정보파일 yaml 추출
root@master0:~# kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml

# etcd 옵션 확인
root@master0:/home/user01/etcd-v3.4.22-linux-amd64# ETCDCTL_API=3 ./etcdctl --help

# 기본적으로 tls 통신하기 때문에 인증서와 키를 옵션으로 줘야함, 쿠버네티스 etcd 정보는 /etc/kubernetes/pki/etcd/ 에 저장되어 있음
root@master0:/home/user01/etcd-v3.4.22-linux-amd64# sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key snapshot save snapshotdb
{"level":"info","ts":1668867684.9010208,"caller":"snapshot/v3_snapshot.go:119","msg":"created temporary db file","path":"snapshotdb.part"}
{"level":"info","ts":"2022-11-19T14:21:24.914Z","caller":"clientv3/maintenance.go:200","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1668867684.914573,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching snapshot","endpoint":"127.0.0.1:2379"}
{"level":"info","ts":"2022-11-19T14:21:25.205Z","caller":"clientv3/maintenance.go:208","msg":"completed snapshot read; closing"}
{"level":"info","ts":1668867685.2091548,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched snapshot","endpoint":"127.0.0.1:2379","size":"4.5 MB","took":0.307965633}
{"level":"info","ts":1668867685.2092974,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","path":"snapshotdb"}
Snapshot saved at snapshotdb

# 스냅샷 상태 정보 확인
root@master0:/home/user01/etcd-v3.4.22-linux-amd64# ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key snapshot status snapshotdb
1d7ebb1, 47669, 1005, 4.5 MB

# 스냅샷 상태 정보 테이블 형태로 출력
root@master0:/home/user01/etcd-v3.4.22-linux-amd64# ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key snapshot status snapshotdb --write-out=table
+---------+----------+------------+------------+
|  HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+---------+----------+------------+------------+
| 1d7ebb1 |    47669 |       1005 |     4.5 MB |
+---------+----------+------------+------------+

# snapshotdb 생성 확인
root@master0:/home/user01/etcd-v3.4.22-linux-amd64# ls
Documentation  etcd  etcdctl  README-etcdctl.md  README.md  READMEv2-etcdctl.md  snapshotdb

# etcd 백업 파일 복원 명령
root@master0:~# sudo ETCDCTL_API=3 /home/user01/etcd-v3.4.22-linux-amd64/etcdctl --endpoints=127.0.0.1:2379 \
> --cacert /etc/kubernetes/pki/etcd/ca.crt \
> --cert /etc/kubernetes/pki/etcd/server.crt \
> --key /etc/kubernetes/pki/etcd/server.key \
> --data-dir /var/lib/etcd-restore \
> --initial-cluster='master0=https://127.0.0.1:2380' \
> --name=master0 \
> --initial-cluster-token this-is-token \
> --initial-advertise-peer-urls https://127.0.0.1:2380 \
> snapshot restore ~/snapshotdb 
{"level":"info","ts":1668868553.5857525,"caller":"snapshot/v3_snapshot.go:296","msg":"restoring snapshot","path":"/root/snapshotdb","wal-dir":"/var/lib/etcd-restore/member/wal","data-dir":"/var/lib/etcd-restore","snap-dir":"/var/lib/etcd-restore/member/snap"}
{"level":"info","ts":1668868553.6189327,"caller":"mvcc/kvstore.go:388","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":46985}
{"level":"info","ts":1668868553.6368642,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"63018d27b801e4a4","local-member-id":"0","added-peer-id":"90cab5eb03ee2ff3","added-peer-peer-urls":["https://127.0.0.1:2380"]}
{"level":"info","ts":1668868553.6488965,"caller":"snapshot/v3_snapshot.go:309","msg":"restored snapshot","path":"/root/snapshotdb","wal-dir":"/var/lib/etcd-restore/member/wal","data-dir":"/var/lib/etcd-restore","snap-dir":"/var/lib/etcd-restore/member/snap"}

# member 디렉토리 있는지 확인
root@master0:~# cd /var/lib/etcd-restore/
root@master0:/var/lib/etcd-restore# ls
member

# etcd.yaml 스태틱파드 수정
# 디렉토리 모두 변경: /var/lib/etcd-->/var/lib/etcd-restore
# 옵션 추가: --initial-cluster-token=this-is-token
root@master0:~# sudo vim/etc/kubernetes/manifests/etcd.yaml

# etcd가 부팅되고 1분 정도 기다린 후 kubectl 동작 확인
root@master0:/var/lib/etcd-restore# kubectl get pod
^C

# yaml로 추출한 deployment, service 등 정보들도 자동으로 복구되어 있다. 확인차 kubectl create -f 를 통해 남은 파일을 복구한다.

```
