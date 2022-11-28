# 쿠버네티스 실전 애플리케이션 개발

[https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)

# [마스터 노드 HA 구성](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/)

단일 마스터를 클러스터에 구성하면 단일 실패 지점이 생기기 떄문에 HA 보장을 위해 클러스터 구조로 구성할 수 있다.

## HA 클러스터 구성 방법 두가지

### Stacked etcd topology

컨트롤 플레인과 etcd 맴버가 같은 노드에 묶임

중복성을 유지하기 위해 최소 3개의 컨트롤 플레인 노드가 필요

etcd와 컨트롤프레인의 중복성이 같이 손상받지만 단순

![stacked-etcd-topology](https://user-images.githubusercontent.com/19777164/204255835-a63d4994-5a87-4acd-ba2a-48c2937fae05.svg)

### External etcd topology

컨트롤 플레인과 etcd 맴버를 분리

호스트 개수가 두배 필요

![external-ectd-topology](https://user-images.githubusercontent.com/19777164/204255804-9bf131cb-1dec-4478-bdd4-221009a535c2.svg)

## [HAProxy](https://www.haproxy.com/)

오픈 소스 로드 밸런서

기존의 하드웨어 스위치를 대체하는 소프트웨어 로드 밸런서로, 네트워크 스위치에서 제공하는 L4, L7 기능 및 로드 밸런서 기능을 제공한다.

참고: [https://d2.naver.com/helloworld/284659](https://d2.naver.com/helloworld/284659)

## 참고

### k8s 마스터 로드밸런싱을 위한 HA proxy 설치하기

[https://blog.naver.com/isc0304/221984189195](https://blog.naver.com/isc0304/221984189195)

### 쿠버네티스 HA(고가용성) 컨트롤플레인 구성 실습

[https://blog.naver.com/isc0304/222513017037](https://blog.naver.com/isc0304/222513017037)
