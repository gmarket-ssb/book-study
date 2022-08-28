## 미니큐브와 쿠버네티스 워크로드


### 미니큐브
쿠버네티스는 기본적으로 하나의 master 노드와 여러 개의 woker 노드를 세팅하여 사용해야한다. 이는 결국 쿠버네티스가 무거운 이유 중에 하나가 된다.<br>
<img src="https://user-images.githubusercontent.com/57446639/187021677-299a8b56-b57a-4bc6-bd4a-478362e8f89c.png" width="650"/>
<br><br>

결국 테스트를 하거나 가볍게 돌리고 싶을 때도 무거운 쿠버네티스 환경을 만들어야 했는데,<br>
이를 해결하고자 가벼운 쿠버네티스 구현체인 "미니큐브" 가 등장한다.
<br><br>

미니큐브는 하나의 노드 안에서 쿠버네티스를 가볍게 사용할 수 있도록 하며,<br>
<img src="https://user-images.githubusercontent.com/57446639/187021707-e28d5986-2002-49d6-9ebc-4fb20f43a801.png" width="650"/>
<br><br>

손쉬운 설치 방법 및 가이드를 공식 홈페이지에서 지원한다.
- install guide: https://minikube.sigs.k8s.io/docs/start/
- usage guide: https://kubernetes.io/ko/docs/tutorials/hello-minikube/
<br><br>

참고로 minukube 설치하면 kubectl 이 함께 설치되며,<br>
<img src="https://user-images.githubusercontent.com/57446639/187022507-e8d49cb5-e951-4570-a126-fcef09950b0b.png" width="500"/>
<br>

`$kubectl get nodes` 명령어로 띄워진 노드를 확인해보면, 미니큐브 노드 하나만 띄워진 것을 확인할 수 있다.<br>
<img src="https://user-images.githubusercontent.com/57446639/187022449-44387b0e-e904-480d-8a00-7203281ca19d.png" width="500"/>
<br><br><br>


### 쿠버네티스 워크로드
#### pod
pod 는 표준 Kubernetes 모듈의 기본 구성요소이며, 배포 가능한 가장 작은 Kubernetes 객체다.<br>
<img src="https://user-images.githubusercontent.com/57446639/187022794-9cd7cdc4-ef07-4258-b85a-59730d57c241.png" width="650"/>
- pod 는 밀접하게 연관된 프로세스를 함께 실행하고 마치 하나의 환경에서 동작하는 것처럼 보이게 한다.
- pod 의 모든 컨테이너는 동일한 네트워크 및 UTS 네임스페이스에서 실행된다.
- 같은 호스트 및 네트워크 인터페이스를 공유하므로 포트 충돌 가능성이 있으니 유의해야 한다.

<img src="https://user-images.githubusercontent.com/57446639/187026473-c235fd76-a2c7-4651-9bed-0ddb8683187a.png" width="500"/><br>
- 그러므로 위 그림과 같이 두 가지의 컨테이너가 밀접한 실행이 필요한 경우(ex. 파일 시스템 및 네트워크를 공유해야하는 경우, 네트워크 통신이 매우 빨라야 하는 경우) 에서만 한 pod 에 넣도록 하자.

pod 의 정의 요소
```yaml
apiVersion: apps/v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    name: nginx
    image: nginx:latest
```
- apiVersion: 쿠버네티스 api 버전
- kind: 어떤 리소스 유형인지 결정(포드 레플리카 컨트롤러, 서비스 등)
- metamata: 포드와 관련된 이름, 네임스페이스, 라벨 그 밖의 정보 존재
- spec: 컨테이너, 볼륨 등의 정보
- stateful: 포드의 상태, 각 컨테이너의 설명 및 상태, 포드 내부의 IP 및 그 밖의 기본 정보 등

<br><br>

### 그 외
- kubeadmn 을 설치하면 kubectl 이 같이 설치된다.
- kubectl client 프로그램. 쿠버네티스에 접속할 수 있게 해준다.
