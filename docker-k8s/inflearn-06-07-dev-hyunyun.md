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


## Service

### 포드의 문제점
1. 포드는 IP 주소의 지속적인 변동, 로드밸런싱을 관리해줄 또다른 개체가 필요
2. 포드는 개별의 쿠버네티스 네트워크를 갖고있으므로, 외부랑 통신이 안된다.
3. 이 문제를 해결하기위해 서비스 라는 리소스가 존재

![image](https://user-images.githubusercontent.com/106303141/190166076-c1ea4f99-9aef-4459-ac32-df4579f99867.png)

### 서비스의 요구사항
1. 포드의 IP가 변경될 때 마다 재설정 하지 않도록 해야 함

![image](https://user-images.githubusercontent.com/106303141/190167617-874388ca-a918-4ae4-adaa-b90fb3dab049.png)

### 서비스의 세션 고정하기
1. 다수의 포드 환경에서 웹서비스의 세션 유지를 위해 클라이언트 IP를 그대로 유지해주는 방법
2. sessionAffinity: ClientIP 옵션을 통해 해결

![image](https://user-images.githubusercontent.com/106303141/190168183-e35d8219-2e22-4f2d-a39e-3a4fee25edbb.png)

### 서비스 노출하는 세 가지 방법
1. NodePort: 노드 자체 포트를 사용하여 포드로 리다이렉션
2. LoadBalancer: 외부 게이트웨이를 사용해 노드 포트로 리다이렉션 (L4)
3. Ingress: 하나의 IP 주소를 통해 여러 서비스를 제공 (L7)

> Cloud 서비스에서는 LoadBalanecer와 Ingress방법 둘다 기본 지원되나, VirtualBox나 VmWare에서는 지원되지 않아, 구현에는 어렵다.

#### 노드포트 서비스 패킷 흐름

![image](https://user-images.githubusercontent.com/106303141/190180464-280512bb-88c5-4f89-b9a1-aa4236032d38.png)

## 쿠버네티스 네트워크 모델
1. 한 포드의 다수의 컨테이너끼리 통신
![image](https://user-images.githubusercontent.com/106303141/190310824-572ebf93-a197-4f12-92ff-0435ca3cf819.png)
![image](https://user-images.githubusercontent.com/106303141/190311802-3d558bed-4b58-419d-8d18-cc6f675e9d3a.png)

2. 포드끼리 통신
> 포드끼리의 통신을 위해서는 CNI플러그인이 필요(ACI, AOS, AWS VPN CNI, CNI-Genie, GCE, flannel, Weave Net, Calico 등)
![image](https://user-images.githubusercontent.com/106303141/190312280-2090836a-12b2-4ad7-8bf0-1941d7ca4586.png)
> Weavenet 네트워크 모델


3. 포드와 서비스 사이의 통신

![image](https://user-images.githubusercontent.com/106303141/190313683-79e1ba5e-3c52-42cb-aed3-a90c4f0333f7.png)


4. 외부 클라이언트와 서비스 사이의 통신

![image](https://user-images.githubusercontent.com/106303141/190314054-8785963e-f90f-4205-8c8f-7aad7bf7bd4b.png)






