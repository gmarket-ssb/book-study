## [인증](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

쿠버네티스의 account에는 두가지 타입이 있다.

- user: 일반 유저를 위한 계정(e.g. 개발자, 데브옵스팀)
- service account: 포드나 애플리케이션을 위한 계정

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

### Service Account

기본적으로 default sa 사용

- 아무 권한도 없으며 여기에 권한을 주는건 권장되지 않음
- 권한이 필요하다면 새로 sa 만들고 권한 부여

파드를 생성하면 자동으로 볼륨 마운트를 통해 ca.crt, namespace, token 가 제공

```yaml
spec:
  containers:
  ...
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-7ltkl
      readOnly: true
  ...
  volumes:
  - name: kube-api-access-7ltkl
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

이제는 sa 만들어도 시크릿이 자동 생성되지 않음

- [https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it](https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it)

예제

```bash
jaeseong@jaeseongui-MacBookPro yaml % kubectl exec -it nx -- bash
root@nx:/# ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt	namespace  token

# 환경변수에 쿠버네티스 관련정보가 이것저것 들어있음
root@nx:/var/run/secrets/kubernetes.io/serviceaccount# printenv
KUBERNETES_SERVICE_HOST=10.96.0.1 # kube-apiserver의 클러스터 내부주소

root@nx:/var/run/secrets/kubernetes.io/serviceaccount# TOKEN=$(cat token)
root@nx:/var/run/secrets/kubernetes.io/serviceaccount# curl -X GET https://$KUBERNETES_SERVICE_HOST/api --header "Authorization: Bearer $TOKEN" --insecure

# 서비스어카운트를 통해 쿠버네티스 API 호출
# 노드포트는 6443(미니큐브는 8443)으로 열리지만 내부에서는 443으로 열림
curl -X GET https://$KUBERNETES_SERVICE_HOST/api --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "192.168.64.5:8443"
    }
  ]

curl -X GET https://$KUBERNETES_SERVICE_HOST/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:sa1\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

참고

- pod sa: [https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- curl sa: [https://kubernetes.io/ko/docs/tasks/administer-cluster/access-cluster-api/](https://kubernetes.io/ko/docs/tasks/administer-cluster/access-cluster-api/)
- api reference: [https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/)
