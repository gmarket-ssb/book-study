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
