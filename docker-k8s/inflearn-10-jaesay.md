# 애플리케이션 스케줄링과 라이프사이클 관리

## 애플리케이션의 환경변수를 관리하는 방법

컨피그맵, 시크릿을 통해 애플리케이션 코드와 별도로 구성 데이터를 설정하고 쉽게 관리할 수 있다.

도커 이미지를 만들때 환경변수를 고려해서 설계하기 때문에 쿠버네티스에서도 환경변수를 알아야 불편함이 없다

### ****컨테이너를 위한 환경 변수 정의하기****

구성파일에 `env`나 `envFrom` 필드를 통해 파드를 생성할 때 파드 안에서 동작하는 컨테이너를 위한 환경 변수를 설정할 수 있다.

여러 환경에 데이터를 정의하거나 유지, 관리가 어려움

### ConfigMap

컨피그맵은 키-값 쌍으로 기밀이 아닌 데이터를 저장하는 데 사용하는 API 오브젝트이다. 파드는 볼륨에서 환경 변수, 커맨드-라인 인수 또는 구성 파일로 컨피그맵을 사용할 수 있다.

컨피그맵을 사용하면 컨테이너 이미지에서 환경별 구성을 분리하여, 애플리케이션을 쉽게 이식할 수 있다.

파일을 통해 ConfigMap을 생성하는 방법이 가장 일반적이다.

**파드설정을 통해 ConfigMap을 사용하는 방법**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-configmap
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    # envFrom: # Use envFrom to define all of the ConfigMap's data as container environment variables
    #   - configMapRef:
    #       name: special-config
    env:
    - name: DEMO_GREETING # 환경변수명
      valueFrom:
        configMapKeyRef:
          name: map-name # configmap name과 매칭
          key: test # configmap test 키의 value와 DEMO_GREETING의 value가 맵핑
---
apiVersion: v1
data:
  test: "1234"
kind: ConfigMap
metadata:
  creationTimestamp: "2022-09-30T13:56:30Z"
  name: map-name
  namespace: default
  resourceVersion: "754380"
  uid: a01a7c52-ad38-4f09-b35f-99b7cf81b116
```

**볼륨을 통해 ConfigMap 사용하는 방법**

코어 dns가 이런식으로 데이터를 전달한다.

파드 설정을 통해 ConfigMap을 사용한 환경변수는 컨테이너를 재시작해야지만 다시 변수가 들어가지만 마운트해서 설정하는 경우는 1분마다 리프레시가 된다.  외부에서도 관리 가능하다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volumes-configmap
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config # configmap의 데이터를 볼륨을 마운트해서 전달
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want to add to the container
        name: special-config
```

### 시크릿

시크릿은 컨피그맵과 유사하지만 특별히 기밀 데이터를 보관하기 위한 것이다. 시크릿 같은 경우는 좀 더 안전한 처리를 위해 인코딩해서 데이터를 처리한다.

```yaml
apiVersion: v1
data:
  password: MTIzNHNzZHNh
  username: YWRtaW4=
kind: Secret
metadata:
  name: db-user-pass
---
apiVersion: v1
kind: Pod
metadata:
  name: envar-secret
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: user # 환경변수명
      valueFrom:
        secretKeyRef:
          name: db-user-pass # secret name과 매칭
          key: username # secret 키(username)의 value와 환경변수명(user)의 value가 맵핑
    - name: pass
      valueFrom:
        secretKeyRef:
          name: db-user-pass
          key: password
```

쿠버네티스 시크릿은 기본적으로 API 서버의 기본 데이터 저장소(etcd)에 암호화되지 않은 상태로 저장된다. 따라서 권한있는 파드나 권한있는 사용자는 확인해볼수 있다.

yaml 파일로 만들 경우 수동으로 base64 인코딩을 해야하고 커멘드로 저장하는 경우는 평문으로 저장해도 된다. 결과는 같다.

환경변수로 들어갈 때에는 ConfigMap과 저장한 것과 동일하게 디코딩된 평문으로 들어가게 된다.

## 초기 명령어 및 아규먼트 전달과 실행

도커에서 초기 명령어 및 아규먼트 전달과 실행하기 위해서는 Dockerfile의 초기명령어를 셋팅하고 빌드해야 한다.

쿠버네티스에서는 파드를 생성할 때 command나 arguments를 전달하면 컨테이너가 부팅된 뒤 실행된다.

## 한 포드에 멀티컨테이너

같은 네트워크 인터페이스, IPC, 볼륨등을 공유한다.

로컬호스트를 통해 통신할 수 있다. 하나의 파드는 네트워크 인터페이스를 띄워주고 유지하기 위해 퍼즈를 하고 슬립한다. 멀티 컨테이너는 퍼즈를 통해 만들어진 네트워크 인터페이스를 공유한다.

컨테이너가 밀접한 실행이 있는경우에만 한 포드에 넣는것을 권장한다. (e.g. 사이드카)

## 초기화 컨테이너(Init Containers)

다수의 컨테이너를 올리는 방법중에 하나지만 사용의 쓰임새가 다르다. 먼저 실행되어야하는 서비스 등이 있는 경우 (계속 리스타트 반복하면서 기다리지 않고..) 점검을 먼저 한 다음에 주컨테이너가 실행 가능하다고 판단되면 실행한다.

```Bash
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          3m54s
myapp-pod   0/1     Init:1/2   0          4m40s
myapp-pod   0/1     PodInitializing   0          4m41s
myapp-pod   1/1     Running           0          4m42s
```
## 시스템 리소스 요구사항과 제한 설정
### 파드 및 컨테이너 리소스 관리
주로 관리자가 설정한다.

애플리케이션 설정과 밀접한 CPU와 메모리 설정을 제한한다.

포드의 디스크를 설정하는 방법도 있지만 가득차는 경우는 없다고 봐도 무방하기 떄문에..설정을 하지 않는다. 대게 별도 스토리지에 대한 주로 스토리지를 제한한다. ⇒ 조사해보기

파드에서 컨테이너에 대한 리소스 요청(request) 을 지정하면, kube-scheduler는 이 정보를 사용하여 파드가 배치될 노드를 결정한다. 컨테이너에 대한 리소스 제한(limit) 을 지정하면, kubelet은 실행 중인 컨테이너가 설정한 제한보다 많은 리소스를 사용할 수 없도록 해당 제한을 적용한다. 또한 kubelet은 컨테이너가 사용할 수 있도록 해당 시스템 리소스의 최소 요청 량을 예약한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: rsc-nginx
  name: rsc-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rsc-nginx
  template:
    metadata:
      labels:
        app: rsc-nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests: # 리소스 요청 설정
            memory: "200Mi"
            cpu: "1m"
          limits: # 리소스 제한 설정
            memory: "400Mi"
            cpu: "2m"
        ports:
        - containerPort: 80
```
kubectl top pod rsc-nginx-68c9547c-fl9n2
```Bash
NAME                       CPU(cores)   MEMORY(bytes)   
rsc-nginx-68c9547c-fl9n2   0m           2Mi
```

**CPU**

CPU는 코어 단위로 지정

CPU 0.5가 있는 컨테이너 는 CPU 1 개를 요구하는 절반의 CPU 

CPU 0.1은 100m과 동일한 기능 (m은 1/1000 CPU)

CPU는 1 하이퍼 스레딩 기능이 있는 베어 메탈 인텔 프로세서의 하이퍼 스레드

클라우드마다 이름이 다르다. (e.g. 1 AWS vCPU, 1 GCP 코어, 1 Azure vCore, 1 IBM vCPU)

작게 만드는 추세..⇒ 조사해보기

**Memory**

메모리는 바이트 단위로 지정

K, M, G의 단위는 1000씩 증가 (잘 사용안함)

Ki, Mi, Gi의 단위는 1024씩 증가 (보통 요걸로 사용, 우리가 이해하는 MB, KB, GB)

### ****리밋 레인지(Limit Range)****

기본적으로 컨테이너는 쿠버네티스 클러스터에서 무제한 컴퓨팅 리소스로 실행된다.

쿠버네티스 1.10 버전부터 리밋레인지 지원이 기본적으로 활성화되었다.

네임 스페이스에서 파드 또는 컨테이너별로 리소스를 제한할 수 있다. limitrange-demo 네임스페이스에서 컨테이너는 CPU는 200-800m 설정가능하며 설정 안할 시 default로 요청/제한 800m이 설정되고 memory는 500Mi-1Gi 설정가능하며 default로 요청/제한 1Gi 설정된다.

```bash
# kubectl describe limitrange -n limitrange-demo
Name:       cpu-min-max-demo-lr
Namespace:  limitrange-demo
Type        Resource  Min   Max   Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---   ---------------  -------------  -----------------------
Container   cpu       200m  800m  800m             800m           -

Name:       mem-min-max-demo-lr
Namespace:  limitrange-demo
Type        Resource  Min    Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---    ---  ---------------  -------------  -----------------------
Container   memory    500Mi  1Gi  1Gi              1Gi            -
```

리소스 쿼터 설정 시 프로젝트를 진행할 때 제한된 리소스 안에서 활동하게 할 수 있다. 네임스페이스의 모든 컨테이너의 합을 제한할 수 있다.

```bash
# kubectl describe resourcequota -n quota-mem-cpu-example
Name:            mem-cpu-demo
Namespace:       quota-mem-cpu-example
Resource         Used   Hard
--------         ----   ----
limits.cpu       800m   2
limits.memory    800Mi  2Gi
requests.cpu     400m   1
requests.memory  600Mi  1Gi
```

네임스페이스에서 스토리지클래스별 최소 및 최대 스토리지 요청을 지정할 수 있다.

## 데몬셋(DaemonSet)

리눅스에서는 벡그라운드 프로세스를 일컫는다. 쿠버네티스에서도 쿠버네티스 전반의 걸쳐서 시스템을 지원하기 위한 포드를 말한다. 

노드 당 포드를 하나씩 구성된다. 그러한 특성에 맞게 노드의 리소스 관리용으로 많이 사용되며 노드의 파일을 공유하는 스토리지인 hostPath랑 많이 쓰인다.

노드 당 하나의 포드이기 떄문에 Deployment처럼 개수를 설정할 필요 없다.

kube-proxy가 데몬셋으로 만든 쿠버네티스에서 기본적으로 활동중인 파드이다.

```bash
# kubectl get ds -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   31d
```
## ****테인트(Taints)와 톨러레이션(Tolerations)****

노드가 갖고 있는 오점(taint)을 용인(toleration)하는 옵션을 파드에 설정하면 해당 노드에 스케줄될 수 있다. 스케줄을 허용하지만 보장하지는 않는다. 스케줄러는 그 기능의 일부로서 다른 매개변수를 고려한다.

동일한 노드에 여러 테인트를, 동일한 파드에 여러 톨러레이션을 둘 수 있다. 쿠버네티스가 여러 테인트 및 톨러레이션을 처리하는 방식은 필터와 같다.

- NoSchedule: 노드에 해당 테인트가 적용되기 전 있었던 파드들은 영향받지 않고 테인트가 적용된다.
- PreferNoSchedule: 해당 테인트가 적용된 노드 말고 배치가능한 다른 노드가 있으면 거기에 배치하고 없으면 해당 노드에 배치한다.
- NoExecute: 해당 노드에 있던 파드들이 다른 노드에 배치되고 테인트가 적용된다.

## 수동 스케줄링

특수한 환경의 경우 특정 노드에서 실행되도록 선호하도록 파드를 제한한다.

- 블록체인이나 딥러닝 시스템을 위해 GPU 서비스가 필요한 경우
- SSD가 있는 노드에서 파드가 실행하기 위한 경우

일반적으로 스케줄러는 자동으로 합리적인 배치를 수행하므로 이러한 제한은 필요하지 않다.

`nodeName` 필드를 사용하여 노드를 직접 지정할 수도 있고 노드의 `label` 을 추가하고 `nodeSelector` 를 사용하여 노드를 선택할 수 있다.

## ****다중 스케줄러(Multiple Schedulers)****

쿠버네티스는 kube-scheduler를 기본 스케줄러로 사용한다. 만일 기본 스케줄러가 사용자의 필요를 만족시키지 못한다면 직접 스케줄러를 구현하여 사용할 수 있다.

기본 스케줄러와 함께 여러 스케줄러를 동시에 실행 가능하다.

멀티플 스케쥴러를 설정하려면 쿠버네티스 api를 사용하기 위해 권한이 필요하고 `schedulerName` 을 통해서 지정하면 해당 스케줄러한테 스케줄링을 받을 수 있다.
