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

![모니터링_구조](https://user-images.githubusercontent.com/19777164/201478808-67d02088-ded1-498b-9326-e3b8385daa11.png)

[프로메테우스](https://prometheus.io/) 

- 모니터링 및 알람에 사용되는 오픈소스 애플리케이션
- CNCF graduated project
- 시간에 따라 데이터를 저장하는 시계열 디비(Time Series Database, TSDB)에 metrics 저장
- pull 방식을 통한 metrics 수집
    - e.g. /actuactor/prometheus endpoint에서 metrics 수집

[그라파나](https://grafana.com/)

    - 데이터 시각화 및 분석을 위한 오픈소스 애플리케이션

구성예제

- [https://blog.naver.com/isc0304/222515904650](https://blog.naver.com/isc0304/222515904650)

## [Istio](https://istio.io/latest/)

Istio는 쿠버네티스 환경의 네트워크 메시 이슈를 보다 간편하게 해결하기 위해 지원하는 환경(서비스 메시)

마이크로서비스 간의 모든 네트워크 통신을 관찰하는 특수 사이드카 프록시(envoy)를 배치

## EFK

![스크린샷 2022-11-13 오후 3 29 47](https://user-images.githubusercontent.com/19777164/201509265-1b457a24-19a3-48b5-a18a-ccf995b4df8d.png)

일반적으로 엘라스틱스택을 사용하여 통합된 로그시스템을 구성

쿠버네티스에서는 CNCF 프로젝트인 FluentD를 사용하는 것이 유행

- Elasticsearch: 데이터베이스 & 검색엔진
    - index = database
        - 날짜가 suffix로 붙음 (`logstash-2022.11.13`)
    - field = column
        - `@timestamp` 는 수집기가 찍어준 시간을 의미
- Logstash: 데이터 수집기, 파이프라인 (F: FluentD)
- Kibana: 대시보드

구성예제

- [https://blog.naver.com/isc0304/221860255105](https://blog.naver.com/isc0304/221860255105)
- [https://blog.naver.com/isc0304/221879552183](https://blog.naver.com/isc0304/221879552183)

## [Jaeger(예거)](https://www.jaegertracing.io/)

분산 서비스 간 트랜잭션을 추적하는 오픈소스 소프트웨어 (e.g. Zipkin)

Distributed tracing을 지원하기 위한 기술들이 벤더에 종속되지 않는 표준 방식인 Open Tracing을 따름

CNCF 프로젝트

헬름 차트나 istio 등을 활용해 편리하게 설치 가능
