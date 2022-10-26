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

