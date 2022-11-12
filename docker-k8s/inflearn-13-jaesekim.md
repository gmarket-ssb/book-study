# 리소스 로깅과 모니터링

쿠버네티스 모니터링 플랫폼은 두개로 구분할 수 있음

- 쿠버네티스가 기본적으로 지원하는 파이프라인
    - [metrics-server 설치](https://github.com/kubernetes-sigs/metrics-server)
- 프로메테우스와 같은 서드파티 모니터링 시스템을 활용하여 구축된 파이프라인
- 참고: [https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/monitoring_architecture.md](https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/monitoring_architecture.md)

애플리케이션 로그는 노드의 `/var/log`에 저장된다. `/var/log/containers` 에 포드명_네임스페이스_컨테이너명.log로 저장된다고 한다..

`kubectl logs` 명령어를 사용하여 컨테이너의 로그(or 단일 컨테이너 포드)를 볼 수 있다.

kube-apiserver가 죽어서 명령어를 못쓰게 될 경우 `docker log` 를 통해 확인 가능하다. containerd(container runtime)를 통해 컨테이너가 만들어지기 떄문에 docker 로그와 동일하다.

## [Kubernetes Dashboard](https://github.com/kubernetes/dashboard)

쿠버네티스 클러스터를 관리하기 위한 웹 UI 대시보드


## 프로메테우스 + 그라파나를 통한 모니터링

### [프로메테우스](https://prometheus.io/) 

- 모니터링 및 알람에 사용되는 오픈소스 애플리케이션
- CNCF graduated project
- 시간에 따라 데이터를 저장하는 시계열 디비(Time Series Database, TSDB)에 metrics 저장
- pull 방식을 통한 metrics 수집
    - e.g. /actuactor/prometheus endpoint에서 metrics 수집

### [그라파나](https://grafana.com/)

- 데이터 시각화 및 분석을 위한 오픈소스 애플리케이션

### helm을 사용하여 프로메테우스 + 그라파나 구축

- [https://blog.naver.com/isc0304/222515904650](https://blog.naver.com/isc0304/222515904650)
