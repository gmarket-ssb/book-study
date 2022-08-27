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
<br><br>


### 쿠버네티스 워크로드



### 그 외
- kubeadmn 을 설치하면 kubectl 이 같이 설치된다.
- kubectl client 프로그램. 쿠버네티스에 접속할 수 있게 해준다.
