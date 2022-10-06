# 애플리케이션 스케줄링과 라이프사이클 관리

## 환경 변수 설정

1. YAML파일이나 다른 리소스로 쿠버네티스 환경 변수를 전달
2. 하드 코딩된 환경 변수는 여러 환경에 데이터를 정의하거나 유지, 관리가 어려움
3. ConfigMap은 외부에 컨테이너 설정을 저장할 수 있음

![image](https://user-images.githubusercontent.com/106303141/193984391-e436ce04-8eff-44a6-9d54-28f922f992fc.png)
> 각 POD마다 환경변수를 매번 변경하기에는 번거로우므로, 외부 환경(ConfigMap, Secret)을 통해 설정한다.

![image](https://user-images.githubusercontent.com/106303141/193984596-0a75a0f3-8ebb-4bd1-8d8e-0a3cdc0baee1.png)
> 1. 첫번째는 정적인 환경변수
> 2. 나머지는 외부에서 참조하여 불러들이는 환경변수
> >  1. ConfigMap은 일반 변수
> >  2. Secret은 인코딩(base64)된 변수 (OAuth토큰 및 ssh키 등)

![image](https://user-images.githubusercontent.com/106303141/193985876-bdfc0632-da9c-4553-bb37-e28b87a9ff1a.png)
> valueFrom 을 envFrom으로 바꾸면 configMap의 모든 key-value를 가져올 수 있다.

## ConfigMap을 활용한 디렉토리 마운트

```
kubectl create -f https://kubernetes.io/examples/configmap/configmap-multikeys.yaml
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```
apiVersion: v1
kind: Pod
metadata:
  name: volumes-configmap
  labels:
    purpose: demonstarte-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io.google-samples/node-hello:1.0
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
```

> 그 후, 해당 컨테이너 bash로 들어가면 /etc/config에 SPECIAL_LEVEL과 SPECIAL_TYPE을 확인할 수 있다.

## 초기 명령어 및 아규먼트 전달과 실행

> Pod를 생성할 때 **spec.containers.command**와 **spec.contaioners.args**에 실행하기 원하는 인자를 전달하면 컨테이너가 부팅된 뒤 실행

![image](https://user-images.githubusercontent.com/106303141/194210669-992826a7-2bc2-4978-867f-cf050b5086d1.png)

> $를 사용하여 명령 내용 변경 가능

![image](https://user-images.githubusercontent.com/106303141/194210830-4f8d2b87-836e-470d-b66c-f98f76d94ecc.png)

## 한 POD에 멀티 컨테이너

1. 하나의 포드를 사용하는 경우 같은 네트워크 인터페이스와 IPC(Inter-Process Communication), 볼륨 등을 공유
2. 데이터의 지역성을 보장, 여러 개의 응용프로그램이 결합된 형태로 POD 구성

## init 컨테이너

1. 포드 컨테이너 실행 전에 초기화 역할을 하는 컨테이너
2. 완전히 초기화가 진행된 다음에야 주 컨테이너를 실행
3. Init 컨테이너가 실패하면, 성공할때까지 포드를 반복해서 재시작
4. restartPolicy에 Never를 하면 재시작하지 않음

![image](https://user-images.githubusercontent.com/106303141/194214511-e031b255-bcf7-4f76-9163-8b4da159022a.png)

![image](https://user-images.githubusercontent.com/106303141/194214936-3e261030-635d-4738-ac2b-6f8a9836b130.png)

## 시스템 리소스 요구사항과 제한 설정

### 리소스 요청 설정 방법
```
spec.containers[].resources.requests.cpu
spec.containers[].resources.requests.memory
```
### 리소스 제한 설정 방법
```
spec.containers[].resources.limits.cpu
spec.containers[].resources.limits.memory
```

![image](https://user-images.githubusercontent.com/106303141/194215757-96e04739-d4dc-4292-b1d1-7b35b34a0e3a.png)

> CPU는 코어단위: m(millicpu)
> > ex) 0.1 = 100m과 동일

> 메모리는 바이트 단위: Ti, Gi, Mi, Ki, T, G, M, K...

### limitRanges

1. 네임 스페이스에서 포드 또는 컨테이너별로 리소스를 제한하는 정책
    * 타입별 (Container, Pod, PersistentVolumeClaim)
    * 네임 스페이스별 (ResourceQuota)
2. 리미트 레인지의 기능
    * 포드나 컨테이너당 최소 및 최대 사용량 제한
    * PersistentVolumeClaim 당 최소 및 최대 스토리지 사용량 제한
    * 리소스에 대한 요청과 제한 사이의 비율 적용
    * 디폴트 requests/limit을 설정하고 런타임 중인 컨테이너에 자동으로 입력
3. 적용방법
    * Apiserver 옵션에 --enable-admission-plugins=LimitRange를 설정

## 데몬셋

1. 레플리카셋은 무작위 노드에 포드를 지정하여 생성
2. 데몬셋은 각 하나의 노드에 하나의 포드만을 구성
3. **kube-proxy가 데몬셋으로 만든 쿠버네티스에서 기본적으로 활동중인 포드**

![image](https://user-images.githubusercontent.com/106303141/194222695-99d694af-e6a3-4433-8afd-9b16a22322db.png)

> 무조건적으로 노드마다 데몬셋에서 만든 파드가 할당되므로, 노드를 관리하기 위한 목적으로 활용할 수 있다.
> ex) 노드파일 공유(hostpath), kube-proxy, 로그 수집 등

### Taint와 Toleration의 원리

> Taint: "오점을 남기다. 더럽히다."

> Toleration: "용인하다 견디다"

1. Taint는 노드에 설정하여, "어떤" 오점을 남김
2. Toleration은 Taint에서 남긴 오점을 "용인"하는 역할

![image](https://user-images.githubusercontent.com/106303141/194224002-212c49fa-9909-4161-80d4-f86d515fcd62.png)
> ex) Master node에는 기본적으로 POD를 배치할수 없으나, 배치하고 싶은 경우

#### Taint Effect

![image](https://user-images.githubusercontent.com/106303141/194226148-33cddf52-5d54-4cc1-b4b7-343454dabfe8.png)

1. NoSchedule
    > Taint가 있으면, 노드에 Pod를 스케줄하지 않는다.

    > Taint가 없으나, PreferNoSchedule(안할 수 있으면 안하겠다...) 이펙트가 있는 Taint가 하나 이상 있으면, 노드에 Pod를 스케줄하지 않으려고 **시도**한다.
2. NoExecute
    > Taint가 있으면, 노드에서 Pod를 축출(이미 실행중인 경우)하고, 스케줄 되지 않는다.(아직 실행안된 경우)

## 수동 스케줄링: 원하는 포드를 원하는 노드에

1. 특수한 환경의 경우 특정 노드에서 실행하게끔 포드를 제한
2. 일반적으로 스케줄러는 자동으로 합리적인 배치를 수행하므로 이러한 제한은 필요하지 않음
3. 더 많은 제어가 필요할 수 있는 몇가지 케이스
    * SSD가 있는 노드에서 포드가 실행하기 위한 경우 (파일 입출력이 빨라야 할 때)
    * 블록체인이나 딥러닝 시스템을 위해 GPU서비스가 필요한 경우 (CPU가 아닌 GPU메모리를 사용해야 할 때)
    * 서비스의 성능을 극대화하기 위해 하나의 노드에 필요한 포드를 모두 배치 (성능이 최적화 된 노드의 경우)

> 즉, 노드에 종속적으로 POD를 배치해야 할 때 사용

![image](https://user-images.githubusercontent.com/106303141/194227455-796d100c-aac0-4fe7-8e02-57dcfcf367de.png)

> nodeName을 적으면 간단하게 구현된다.

### 노드 셀렉터를 활용한 스케줄링

* 특정 하드웨어를 가진 노드에서 포드를 실행하고자 하는 경우

![image](https://user-images.githubusercontent.com/106303141/194227745-e87d2905-3933-43af-a7d3-cc0360f548bb.png)

![image](https://user-images.githubusercontent.com/106303141/194227779-3a4117be-b6e0-487b-b15e-8aeefa5cc478.png)


