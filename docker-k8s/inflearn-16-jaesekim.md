# 클러스터 보안

## [Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

서버 침해사고에 대한 최소화하기 위해 컨테이너 내부의 유저 권한을 축소

- 침해사고를 당한 서비스가 모든 권한(root나 노드 커널 기능)으로 동작하는 경우 서비스를 탈취한 공격자는 그대로 컨테이너의 권한을 사용할 수 있기 때문에 권한을 최대한 축소하여 침해사고에 대한 확대를 방지 (최소 권한 정책에 따른 취약점 감소)

보안 컨텍스트 설정(주로 권한)은 주로 리눅스 기능을 통해 설정

- 권한 상승 가능 여부
- 프로세스 기본 UID/GID를 활용한 파일 등의 오브젝트의 액세스 제어
- Linux Capabilities를 활용한 커널 기능 추가
    - 도커는 노드에 있는 커널 기능을 공유해서 사용하기 때문에 리눅스의 커널 기능을 제한하는데 일부 허용하게 함
- 오브젝트에 보안 레이블을 지정하는 SELinux (Security Enhanced Linux) 기능
    - 복잡해서 잘 안쓴다고 함
- AppArmor : 프로그램 프로필을 사용하여 개별 프로그램의 기능 제한
- Seccomp : 프로세스의 시스템 호출을 필터링

## [NetworkPolicy](https://kubernetes.io/ko/docs/concepts/services-networking/network-policies/)

파드들 간에 통신 제한(방화벽 역할 수행)

CNI Plugin 중 NetworkPolicy 설정이 안되는 것이 있음(e.g. flannel)

서비스로 들어오는 통신들에도 잘 적용된다.

세부 조건을 정의하지 않을 경우 default로 deny all이 적용된다.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector: # 해당 파드로 network policy 적용
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress: # 들어오는 트래픽에 대한 정책 설정
    - from: # ipBlock, namespaceSelector, podSelector를 통해 규칙을 설정할 수 있고 3가지 규칙은 OR 조건으로 적용된다.
        - ipBlock: # 허용하고자 하는 ip 대역 설정 가능
            cidr: 172.17.0.0/16
            except: # 예외 항목 설정 가능
              - 172.17.1.0/24
        - namespaceSelector: # 허용되는 네임스페이스
            matchLabels:
              project: myproject
        - podSelector: # 허용되는 다른 파드
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress: # 나가는 트래픽에 대한 정책 설정
    - to: # 두가지 조건은 AND 조건으로 적용된다.
        - ipBlock: # 허용하고자 하는 ip 대역 설정
            cidr: 10.0.0.0/24
      ports: # 어떤 포트를 허용하는지 명시
        - protocol: TCP
          port: 5978
```
