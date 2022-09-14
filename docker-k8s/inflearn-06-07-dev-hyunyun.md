# kube-system 컴포넌트

## kube-system 컴포넌트 이해

<img width="247" alt="image" src="https://user-images.githubusercontent.com/106303141/189862902-a80c5655-1eb5-4cd2-ab44-7f178b35bd4b.png">

### 큐브 API 서버
1. 쿠버네티스 시스템 컴포넌트는 오직 API 서버와 통신
2. 컴포넌트 끼리 서로 직접 통신 X
3. etcd 와 통신하는 유일한 컴포넌트 API 서버
4. RESTful API를 통해 클러스트 상태를 쿼리, 수정할 수 있는 기능 제공

> ### 구체적인 역할
> 1. 권한 및 인증 플러그인을 통한 클라이언트 인증
> 2. 승인 제어 플러그인을 통한 리소스 확인/수정

### 큐브 컨트롤러 매니저
1. 쿠버네티스 안의 자원 및 정보와 같은 것들을 다양한 컨트롤러를 통해 관리
  * 레플리케이션 매니저, 리플리카셋, 데몬셋, 잡, 디플로이먼트 등

### 큐브 스케줄러
1. 일반적으로 실행할 노드를 직접 정해주지 않음.
2. 요청받은 리소스를 어느 노드에 실행할지 결정하는 역할
3. 현재 노드의 상태를 점검하고 최상의 노드를 찾아 배치
4. 다수의 포드를 배치하는 경우에는 라운드로빈을 사용하여 분산
> ### 라운드 로빈 스케줄링
> 프로세스들 사이에 우선순위를 두지 않고, 시간단위로 CPU를 할당하는 방식

## 스태틱 포드

### 스태틱 포드의 필요성
1. 스태틱포드 : kubelet이 직접 실행하는 포드
2. 포드를 apiserver를 통해 삭제해도, 노드에서 스태틱포드가 있는지 없는지 확인하여 다시 만들어준다.
3. 특정 경로에 yaml파일을 구성하여 설정한다.

![image](https://user-images.githubusercontent.com/106303141/190142023-f059b8b4-f7f0-4aa9-afef-440ed9ab8ac9.png)
큐브 시스템 설정 기본경로
> 기본경로의 yaml 파일은 함부로 수정할 시, 복구가 어려울 수 있으므로, 스냅샷이나 백업을 따로 하는것을 권장한다.

```
vim /etc/systemd/system/kubelet.service.d
# 10-kubeadm.conf 확인
```

![image](https://user-images.githubusercontent.com/106303141/190142994-bbfafe37-f482-4dee-b13d-1466132bb07f.png)

> kubelet에 대한 환경 명령어를 확인 할 수 있다.
> KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml 을 확인

![image](https://user-images.githubusercontent.com/106303141/190143369-cd44246e-a985-49e9-a6ad-07b141e44d2e.png)
> staticPodPath를 확인

실습을 위해 /etc/kubernates/manifests 경로에 static-pod.yaml을 새로 생성한다.
![image](https://user-images.githubusercontent.com/106303141/190144175-898f96e9-2741-4d85-8973-e586d790321a.png)

![image](https://user-images.githubusercontent.com/106303141/190144591-b31b874c-501b-4f8b-a9f8-a370f3429b27.png)


## etcd

쿠버네티스 데이터베이스이며, 쿠버네티스의 전체 설정 정보를 저장한다.

![image](https://user-images.githubusercontent.com/106303141/190146258-a82fff7d-a0a5-4118-ae0c-111c68ecc98c.png)


1. Key-Value 데이터 셋으로 이루어져있다.
2. etcdctl이라는 클라이언트 오픈소스가 있으므로 활용

![image](https://user-images.githubusercontent.com/106303141/190145923-6a0ddc8d-8e9c-4230-b97d-65728d81a8ca.png)


