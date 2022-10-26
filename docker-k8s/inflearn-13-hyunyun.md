# 리소스 로깅과 모니터링

## 쿠버네티스 모니터링 시스템과 아키텍처

* 쿠버네티스를 지원하는 다양한 모니터링 플랫폼
  * Heapster (deprecated)
  * Metrics Service ( 쿠버네티스 v1.8 부터 Heapster 대신 Metrics Service로 대체함 )
  * cAdvisor
  * 프로메테우스
  * EFK

## 리소스 모니터링 도구

* **리소스** 메트릭 파이프라인
  * kubectl top 등의 유틸리티 관련된 메트릭들로 제한된 집합을 제공
  * 단기 메모리 저장소인 metrics-server에 의해 수집
  * metrics-server는 모든 노드를 발견하고 kubelet에 CPU와 메모리를 질의
  * kubelet은 kubelet에 통합된 cAdvisor를 통해 레거시 도커와 통합 후 metric-server 리소스 메트릭으로 노출
  * /metrics/resource/v1beta1 API를 사용

> ```kubectl top``` 요청 => API Server 질의 => metric server 질의 => kubelet 질의 => docker 프로세스로 진행

> 참고) metrics 서버는 기본적으로 TLS통신을 이용한다. 로컬에서 인증서 없이 진행할 경우 ```--kubelet-insecure-tls```와 ```--kubelet-preferred-address-types=InternalIP``` 매개변수를 추가하여 실행해야 한다. 

![image](https://user-images.githubusercontent.com/106303141/198029856-d43bfbd7-d0a0-44e1-8dc2-e4793b11004f.png)

* **완전한** 메트릭 파이프라인
  * 보다 풍부한 메트릭에 접근
  * 클러스터의 현재 상태를 기반으로 자동으로 스케일링하거나 클러스터를 조정
  * 모니터링 파이프라인은 kubelet에서 메트릭을 가져옴
  * CNCF 프로젝트인 프로메테우스가 대표적
  * custom.metrics.k8s.io, external.metrics.k8s.io API를 사용

> 리소스 메트릭 파이프라인 프로세스에서 metric server 대신 프로메테우스 사용

![image](https://user-images.githubusercontent.com/106303141/198026503-8cd50d6c-484a-4d64-b336-cda91f52ea3a.png)

참조링크 : https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/monitoring_architecture.md

## 애플리케이션 로그 관리

* Kubernetes 애플리케이션 로그 확인
  * 로그는 컨테이너 단위로 로그 확인 가능
  * 싱글 컨테이너 포드의 경우 포드까지만 지정하여 로그 확인
  * 멀티 컨테이너의 경우 포드 뒤에 컨테이너 이름까지 전달하여 로그 확인
  * ```kubectl logs <pod name> <옵션:container name>```

* kubeapi가 정상 동작하지 않는 경우
  * 쿠버네티스에서 돌아가는 리소스들은 모두 docker를 사용
  * 따라서 docker의 로깅 기능을 사용
  * ```docker ps -a```를 사용하여 조회 가능
  * ```docker logs <container id>```를 사용하여 로그 확인 가능

## 큐브 대시보드 설치와 사용

### Kubernetes Dashboard 설치

참조 링크 : https://github.com/kubernetes/dashboard

1. 다음 명령어를 사용하여 git에 올라온 yaml 파일을 바로 적용

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml```

2. kubernetes-dashboard가 올라간것을 확인

![image](https://user-images.githubusercontent.com/106303141/198032850-1c219a67-d921-4cc7-82b8-3937f24a27b1.png)

3. kubernetes-dashboard의 TYPE을 ClusterIP가 아닌 NodePort로 변경

```kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard```

![image](https://user-images.githubusercontent.com/106303141/198033156-7e7b90b0-42e7-4787-be7f-0d1c5c0ae23b.png)

![image](https://user-images.githubusercontent.com/106303141/198033308-f718db0a-593e-40be-bb8f-171a5f5db76c.png)

4. 포트확인하여 접속

![image](https://user-images.githubusercontent.com/106303141/198033703-bd2c4b17-7459-4072-b4ca-05b962eed3b6.png)

5. service account 토큰 조회 후 토큰 내용 확인

```kubectl get sa -n kubernetes-dashboard kubernetes-dashboard -o yaml```

![image](https://user-images.githubusercontent.com/106303141/198034357-8b2ca08f-363c-4c11-85fb-b9fab53ccf4d.png)

```kubectl describe secret -n kubernetes-dashboard kubernetes-dashboard-toke-6d4lk```

![image](https://user-images.githubusercontent.com/106303141/198034626-9b5a28c2-fed1-4eb4-bf18-5f0b27d320e4.png)

6. 토큰값 입력

![image](https://user-images.githubusercontent.com/106303141/198034885-3871e6a7-5f25-4782-a48d-7465a08f9ddb.png)
