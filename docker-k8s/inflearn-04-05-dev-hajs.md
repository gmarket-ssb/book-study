# 미니큐브와 쿠버네티스 워크로드


## 미니큐브
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


## 쿠버네티스 워크로드
### pod
pod 는 표준 Kubernetes 모듈의 기본 구성요소이며, 배포 가능한 가장 작은 Kubernetes 객체다.<br>
<img src="https://user-images.githubusercontent.com/57446639/187022794-9cd7cdc4-ef07-4258-b85a-59730d57c241.png" width="650"/>
- pod 는 밀접하게 연관된 프로세스를 함께 실행하고 마치 하나의 환경에서 동작하는 것처럼 보이게 한다.
- pod 의 모든 컨테이너는 동일한 네트워크 및 UTS 네임스페이스에서 실행된다.
- 같은 호스트 및 네트워크 인터페이스를 공유하므로 포트 충돌 가능성이 있으니 유의해야 한다.
<br>

<img src="https://user-images.githubusercontent.com/57446639/187026473-c235fd76-a2c7-4651-9bed-0ddb8683187a.png" width="500"/><br>
- 그러므로 위 그림과 같이 두 가지의 컨테이너가 밀접한 실행이 필요한 경우(ex. 파일 시스템 및 네트워크를 공유해야하는 경우, 네트워크 통신이 매우 빨라야 하는 경우) 에서만 한 pod 에 넣도록 하자.
<br><br>
  
#### pod 의 정의 요소
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```
- apiVersion: 객체를 만드는 데 사용되는 Kubernetes API 버전
- kind: 원하는 객체를 정의, 어떤 리소스 유형인지 결정(포드 레플리카 컨트롤러, 서비스 등)
- metamata: 객체를 식별할 수 있도록 포드와 관련된 이름, 고유 ID, 네임스페이스(optional), 라벨 등을 사용
- spec: 컨테이너, 볼륨 등의 정보
- status: 포드의 상태, 각 컨테이너의 설명 및 상태, 포드 내부의 IP 및 그 밖의 기본 정보 등
  - 유저가 작성하는 정보가 아님
<br><br>

#### pod 을 보조하는 기능 (Liveness, Readiness and Startup Probes)
> https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- Liveness
  - 컨테이너가 살았는지 판단하고 다시 시작하는 기능
  - 컨테이너의 상태를 스스로 판단하여 교착 상태에 빠진 컨테이너를 재시작
- Readiness
  - 포드가 준비된 상태에 있는지 확인하고 정상 서비스를 시작하는 기능
  - 포드가 적절하게 준비되지 않은 경우 로드밸런싱을 하지 않음
- Startup Probe
  - 애플리케이션의 시작 시기를 확인하여 가용성을 높이는 기능
  - Liveness, Readiness 기능을 비활성화
<br><br>

#### 레이블과 셀렉터
레이블(라벨)
- 모든 리소스를 구성하는 매우 간단하면서도 강력한 쿠버네티스 기능
- 리소스에 첨부하는 임의의 키/값 쌍(ex. app:test)
- 레이블 셀렉터를 사용하면 각종 리소스를 필터링하여 선택할 수 있음
- 모든 사람이 쉽게 이해할 수 있는 체계적인 시스템 구축 가능
  - app: 애플리케이션 구성 요소
  - rel: 애플리케이션의 버전 지정
  - <img src="https://user-images.githubusercontent.com/57446639/187056919-5e63e5ce-5359-40f0-b7da-60d2b4855c51.png" width="650"/><br>
<br><br>

#### ReplicationController, ReplicaSet
> https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
ReplicationController
- 구버전(~v1.8) 에서 사용하던 컨트롤러
- 포드가 항상 실행되도록 유지하는 쿠버네티스 리소스
- 포드를 감지하고 대체 포드를 생성

ReplicationController 세 가지 요소
1. ReplicationController 가 관리하는 포드 범위를 결정하는 **레이블 셀렉터**
2. 실행해야 하는 포드의 수를 결정하는 **복제본 수**
3. 새로운 포드의 모양을 설명하는 **포드 템플릿**

ReplicationController vs ReplicaSet
- 거의 동일하게 동작하지만, ReplicaSet 이 더 풍부한 포드 셀렉터 사용이 가능하다.
- 정확히는 ReplicationController 는 동일 기반 선택기만 지원하는 반면, ReplicaSet 은 세트 기반 선택기를 지원한다는 것이다.

<img src="https://user-images.githubusercontent.com/57446639/187057613-6e152439-433f-4b2e-b166-669f584f0390.png" width="650"/><br>
- ReplicationController: 특정 레이블을 포함하는 포드가 일치하는지 확인
- ReplicaSet: 특정 레이블이 없거나 해당 값과 관계없이 특정 레이블 키를 포함하는 포드를 매치하는지 확인

<img src="https://user-images.githubusercontent.com/57446639/187057212-4dab5895-65cf-4a99-9612-7261e0449c5b.png" width="650"/><br>
<img src="https://user-images.githubusercontent.com/57446639/187057687-105b6921-5cb1-43f7-8753-5a17df1cceea.png" width="650"/><br><br>

#### Deployment




<br><br><br>

## 명령어 모음
- `$kubectl get pod`: 띄워진 pod 확인
- `$kubectl get pod -w`: pod 정보를 실시간(watch)으로 확인
- `$kubectl get pod [{podName}] -o yaml`: 띄워진 pod 를 디테일하게 확인
- `$kubectl create -f {yamlName}.yaml`: yaml 파일에 작성한 정보로 pod 를 생성
- `$kubectl delete -f {yamlName}.yaml`: yaml 파일에 작성한 정보로 pod 를 제거
- `$kubectl delete pod {podName}`: 해당 pod 를 제거
- `$kubectl delete pod --all`: 모든 pod 를 제거
- `$kubectl port-forward {podName} 8888:8080`: 8888 포트를 8080 포트로 포트 포워딩
- `$kubectl label pod {podName} test=foo`: test=foo 라는 레이블 추가
- `$kubectl label pod {podName} rel=beta --overwrite`: 기존 레이블 수정
- `$kubectl label pod {podName} rel-`: 레이블 삭제
- `$kubectl get pod --show-labels`: pod 의 레이블 확인
- `$kubectl get pod --show-labels -l 'env'`: pod 의 특정 레이블 필터링 검색
<br><br><br>

## 참고
- kubeadmn 을 설치하면 kubectl 이 같이 설치된다.
- kubectl client 프로그램. 쿠버네티스에 접속할 수 있게 해준다.
