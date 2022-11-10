# 리소스 로깅과 모니터링

쿠버네티스 모니터링 플랫폼은 두개로 구분할 수 있음

- 쿠버네티스가 기본적으로 지원하는 파이프라인
    - [metrics-server 설치](https://github.com/kubernetes-sigs/metrics-server)
- 프로메테우스와 같은 서드파티 모니터링 시스템을 활용하여 구축된 파이프라인
- 참고: [https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/monitoring_architecture.md](https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/monitoring_architecture.md)

애플리케이션 로그는 노드의 `/var/log`에 저장된다. `/var/log/containers` 에 포드명_네임스페이스_컨테이너명.log로 저장된다고 한다..

`kubectl logs` 명령어를 사용하여 컨테이너의 로그(or 단일 컨테이너 포드)를 볼 수 있다.

kube-apiserver가 죽어서 명령어를 못쓰게 될 경우 `docker log` 를 통해 확인 가능하다. containerd(container runtime)를 통해 컨테이너가 만들어지기 떄문에 docker 로그와 동일하다.
