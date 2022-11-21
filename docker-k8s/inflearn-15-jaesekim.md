## [인증](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

쿠버네티스의 account에는 두가지 타입이 있다.

- user: 개발자 및 데브옵스 팀 등 쿠버네티스를 사용하는 사용자들이 인증을 위해 사용
- service account: 포드 외 애플리케이션들이 인증을 위해 사용

### static file token

스태틱 파일 안에 토큰을 넣고 해당 토큰을 사용하여 인증한다. kube-apiserver yaml 내에 토큰 파일 위치를 작성하기 때문에 토큰을 수정하려면 kube-apiserver를 재시작해야 한다.

예제

```bash
user01@master0:~$ sudo -i

# static file token 작성
root@master0:~# vim somefile.csv
password1,user1,uid001,"group1"
password2,user2,uid002
password3,user3,uid003
password4,user4,uid004

# 파일을 읽기 위해선 hostpath에 위치한 곳으로 옮겨야 한다.
root@master0:~# mv ./somefile.csv /etc/kubernetes/pki/

# kube-apiserver에 아래 내용추가
root@master0:/etc/kubernetes/manifests# vim /etc/kubernetes/manifests/kube-apiserver.yaml
- --token-auth-file=/etc/kubernetes/pki/somefile.csv

# 에러가 있는지 로그로 확인
# ref. https://github.com/kubernetes/kubeadm/issues/2695
root@master0:/etc# crictl --help
root@master0:/etc# crictl logs --help
root@master0:/etc# crictl ps
root@master0:/etc# crictl logs d5c05e3004bbf

# kube-apiserver 잘뜨는지 확인
root@master0:/etc/kubernetes/manifests# watch "sudo crictl ps | grep api"

# user 설정하기 (kubectl에 등록하고 사용하는 방법)
# 아이디/비밀번호를 입력...
root@master0:/etc/kubernetes/manifests# kubectl config set-credentials user1 --token=password1
User "user1" set.

# 어떤 클러스터와 유저를 맵핑할지..
root@master0:/etc/kubernetes/manifests# kubectl config set-context user1-context --cluster=kubernetes --namespace=frontend --user=user1
Context "user1-context" created.

# 유저 전환
root@master0:/etc/kubernetes/manifests# kubectl config use-context user1-context
Switched to context "user1-context".

# 인가 에러
root@master0:/etc/kubernetes/manifests# kubectl get pod --user user1
# or
root@master0:/etc/kubernetes/manifests# kubectl get pod
Error from server (Forbidden): pods is forbidden: User "user1" cannot list resource "pods" in API group "" in the namespace "frontend"

# admin으로 재전환
root@master0:/etc/kubernetes/manifests# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

```
