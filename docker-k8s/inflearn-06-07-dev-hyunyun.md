# kube-system 컴포넌트

## kube-system 컴포넌트 이해

<img width="247" alt="image" src="https://user-images.githubusercontent.com/106303141/189862902-a80c5655-1eb5-4cd2-ab44-7f178b35bd4b.png">

### 큐브 API 서버
1. 쿠버네티스 시스템 컴포넌트는 오직 API 서버와 통신
2. 컴포넌트 끼리 서로 직접 통신 X
3. etcd 와 통신하는 유일한 컴포넌트 API 서버
4. RESTful API를 통해 클러스트 상태를 쿼리, 수정할 수 있는 기능 제공

> ### 구체적인 역할
> 1. 권한 및 인증 플러그인을 통한 클라이언트 인증
> 2. 승인 제어 플러그인을 통한 리소스 확인/수정
