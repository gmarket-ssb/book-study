## 쿠버네티스란?
- 구글이 내부적으로 몰래 사용하던 보그(Borg)에서 경험을 쌓고나서 출시한 오픈소스 시스템
- 많은 양의 서비스를 관리하는 목적

### 특징
-  기본 인프라를 추상화하여 애플리케이션의 쉬운 개발, 배포, 관리

### 장점
- 애플리케이션 배포 단순화
- 하드웨어 활용 극대화
- 상태 확인 및 자가 치유
- 오토스케일링
- 애플리케이션 개발 단순화

> *즉, 개발자 + 운영자를 도와 프로세스를 단순화하여 DevOps가 원활하게 돌아가기 위한 시스템*

### 클러스터 아키텍쳐
<img width="1002" alt="image" src="https://user-images.githubusercontent.com/106303141/184526652-1e398bb4-787e-469d-8198-7448fd1944c8.png">

### 쿠버네티스에서 애플리케이션 실행
<img width="955" alt="image" src="https://user-images.githubusercontent.com/106303141/184527269-f4257f43-daa7-4b4c-a5da-b1a2c384cf62.png">

### 포드(POD)란?
![image](https://user-images.githubusercontent.com/106303141/185937876-525cc725-6e53-4545-b684-96ebada9b08e.png)

디플로이 -> 레플리카셋 -> 포드

### 쿠버네티스 설치전 주의사항
- 최소 2GB이상의 RAM (4GB추천, 넉넉하게 8GB)
- 2CPU * 3개 
- 네트워크 통신이 가능해야함, (방화벽 시 포트오픈)
- 유니크한 *호스트명*, MAC주소, product_uuid를 갖고있어야 함
- 쿠버네티스는 인스턴스를 최대한 성능 100%로 이끌어내는 것을 목표로 함 (개발 PC에서 직접 설치하면 후회하게 될 듯, 스왑기능을 제거할 수는 있음)



---



## 실습과정
### 방법 (택 1)
- 구글 클라우드 플랫폼 (GCP)
- VMWare
- AWS EKS
- VirtualBox

### 쿠버네티스 설치
```
# kube_install.sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 설치 후 노드 복제 (클러스터 구성)
#### 목표
- 마스터노드 (1개) -< 워커노드 (2개)

#### 스왑기능 비활성화
메모리(RAM)이 모자랄때 하드디스크의 일부를 RAM처럼 쓰는 기능
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

#### 도커 cgroups -> systemd로 변경하기
cgroups는 리눅스 커널의 종류로, 컨테이너나 프로세스가 리눅스 자원에 접근하는 것을 제한하는 기능
```
sudo docker info | grep -i cgroup # 현재 도커의 cgroup 확인

cat <<EOF | sudo tee /etc/docker/daemon.json # 도커 데몬의 설정파일 구성
#{
#  "exec-opts": ["native.cgroupdriver=systemd"]
#}
#EOF

service docker restart # 도커 재시작
```

#### 마스터 노드 초기화
```
kubeadm init
```

![image](https://user-images.githubusercontent.com/106303141/185929570-02c05591-2fb8-4fdd-bdbb-b1d357c49a2a.png)


### ngnix 애플리케이션 생성과 확인
![image](https://user-images.githubusercontent.com/106303141/184595213-00ddb740-9532-4229-8bfc-a25231c93f14.png)

![image](https://user-images.githubusercontent.com/106303141/184595842-df3a0ae1-ae94-4887-9ff2-13f68cbe835c.png)

### 쿠버네티스에서 앱 실행해보기 (Go언어)

- 쿠버네티스에서 올릴 컨테이너 이미지를 하나 생성
- 포트 8080에서  HTTP서버를 시작

앱 작성 후 실행
```
apt install golang
go get github.com/julienschmidt/httprouter
go build main.go
main
```
![image](https://user-images.githubusercontent.com/106303141/185933816-993f2e8a-7e30-4e2e-be0a-8364a2caa0df.png)

dockerfile
```
FROM golang:1.11
WORKDIR /usr/src/app
COPY main  /usr/src/app
CMD ["/usr/src/app/main"]
```

도커 빌드 및 푸시
```
sudo docker build -t gasbugs/http-go .
sudo docker login
sudo docker push gasbugs/http-go
sudo docker run -d -p 8080:8080 --rm http-go
curl 127.0.0.1:8080
```

![image](https://user-images.githubusercontent.com/106303141/185936182-3901d591-26ac-4a29-92b6-582d0b92da0d.png)


## 디플로이먼트, 포드, 서비스가 동작하는 방식 이해

- 사실 실제로 포드도 직접 만들지 않음
- kubectl create deploy 명령을 실행하여 디플로이먼트 생성
- 디플로이먼트가 실제 포드 객체를 생성
- 디플로이먼트가 관리하는 포드의 포트를 노출하라고 명령 필요

![image](https://user-images.githubusercontent.com/106303141/185939314-8ff37fdb-783e-4c33-aa51-7a1c8bb3d1b2.png)

포드에서 외부로 서비스를 노출하기에는 적합하지 않다. 따라서 expose해주는 서비스를 구성한다.

### 서비스의 역할
- 포드는 언제든지 사라질 수 있다. ( 강사는 포드를 가축으로 비유하며 애정들이지 말라고 함.... ㅠ )
- 포드가 다시 시작되면 IP와 ID가 변경된다.
- 서비스의 정적 IP를 사용하여, 변화하는 포드의 IP들을 연결하여 포워딩하여 노출시킨다.

![image](https://user-images.githubusercontent.com/106303141/185943497-bcb9f337-9d24-4c4c-b0fc-81c16b52b310.png)

서비스가 여러 포드를 연결하며 로드밸런싱해주고 있다.
