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

> Pod를 생성할 때 *spec.containers.command*와 *spec.contaioners.args*에 실행하기 원하는 인자를 전달하면 컨테이너가 부팅된 뒤 실행

![image](https://user-images.githubusercontent.com/106303141/194210669-992826a7-2bc2-4978-867f-cf050b5086d1.png)

> $를 사용하여 명령 내용 변경 가능

![image](https://user-images.githubusercontent.com/106303141/194210830-4f8d2b87-836e-470d-b66c-f98f76d94ecc.png)

## 한 POD에 멀티 컨테이너

1. 하나의 포드를 사용하는 경우 같은 네트워크 인터페이스와 IPC(Inter-Process Communication), 볼륨 등을 공유
2. 데이터의 지역성을 보장, 여러 개의 응용프로그램이 결합된 형태로 POD 구성

