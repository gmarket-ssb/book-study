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
  labels:
    purpose: demonstrate-envars
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
  labels:
    purpose: demonstrate-envars
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
