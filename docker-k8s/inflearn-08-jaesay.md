# 볼륨

쿠버네티스는 다양한 유형의 볼륨을 지원한다. 파드는 여러 볼륨 유형을 동시에 사용할 수 있다. 임시 볼륨 유형은 파드의 수명을 갖지만, 퍼시스턴트 볼륨은 파드의 수명을 넘어 존재한다. 파드가 더 이상 존재하지 않으면, 쿠버네티스는 임시(ephemeral) 볼륨을 삭제하지만, 퍼시스턴트(persistent) 볼륨은 삭제하지 않는다. 볼륨의 종류와 상관없이, 파드 내의 컨테이너가 재시작되어도 데이터는 보존된다.

기본적으로 볼륨은 디렉터리이며, 일부 데이터가 있을 수 있으며, 파드 내 컨테이너에서 접근할 수 있다. 디렉터리의 생성 방식, 이를 지원하는 매체와 내용은 사용된 특정 볼륨의 유형에 따라 결정된다.

## 볼륨 유형

### emptyDir

하나의 포드 안에서 멀티 컨테이너 간 데이터를 공유하고자 할 때 사용한다.

컨테이너와 같이 사라진다.

### hostPath

호스트 노드에 있는 디렉토리를 포드에 마운트시키는 것과 같이 호스트 노드와 파드 사이에 파일시스템을 공유한다.  

파드가 배치되는 노드는 스케쥴링에 따라 달라질 수 있기 때문에 데이터를 유지하기 위해서 사용하지 않고 노드에 있는 데이터를 포드에 제공하고 싶을 때 주로 사용한다.

따라서 노드의 리소스/로그를 확인한다던지 모니터링 용도로 많이 쓰인다. 예를 들어 GCP에서 쿠버네티스 클러스터를 만들면 노드마다 노드 및 컨테이너들의 로그를 수집하기 위해 데몬셋으로 fluentd가 떠있다.

> kubectl get pod fluentd-pod-name -n kube-system -o yaml
  ```
  volumes:
  - hostPath:
      path: /var/run/google-fluentbit/pos-files
      type: ""
    name: varrun
  - hostPath:
      path: /var/log # 노드 로그
      type: ""
    name: varlog
  - hostPath:
      path: /var/lib/kubelet/pods
      type: ""
    name: varlibkubeletpods
  - hostPath:
      path: /var/lib/docker/containers # 컨테이너 로그
      type: ""
    name: varlibdockercontainers
  ```
