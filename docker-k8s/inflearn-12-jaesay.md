# 헬름 차트를 활용한 쿠버네티스 애플리케이션 패키지 배포

## HELM

[https://helm.sh/](https://helm.sh/)

The package managerfor Kubernetes

애플리케이션을 배포하기 위해서는 많은 리소스를 yaml 로 작성해야 한다. helm 차트를 통해 리소스의 매니페스트 파일을 패키지로 만들어 관리할 수 있다.

쿠버네티스 클러스터에 엑세스해서 작업할 수 있는 환경이 구성된 곳에 설치되어야 한다. 내부적으로 kubectl 사용해서라고 한다.

cloud shell에는 helm 설치되어 있다.

helm을 통해 배포하지만 기존과 같이 사용할 수 있다.

## 헬름차트 구성요소

[https://helm.sh/docs/chart_template_guide/getting_started/](https://helm.sh/docs/chart_template_guide/getting_started/)

```bash
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

## 헬름 명령어

[https://helm.sh/docs/helm/#helm-commands](https://helm.sh/docs/helm/#helm-commands)

```bash
# 헬름 차트 리렉토리(구성요소들) 생성
helm create mychart2

# 헬름차트 설정 확인
helm lint mychart2

# 헬름 차트 패키지(tgz를 만듦)
helm package mychart2

# 헬름 차트 패키지를 사용하여 index 파일 생성
helm repo index ./

# 저장소에 추가
helm repo add mycharts https://raw.githubusercontent.com/jaesay/study2022/main/devops-k8s-master/helm-charts

# 저장소 업데이트
helm repo update

# 저장소 차트 목록 확인
helm repo list

# 저장소 차트 이름으로 검색
helm search repo mycharts

# 차트를 통해 배포
helm install mychart-test mycharts/mychart2

# 배포 목록 확인
helm list

# 배포된 헬름차트 상태 확인
helm status mychart-test

# 배포 종료
helm uninstall mychart-test

# 저장소에서 제거
helm repo remove mycharts
```
