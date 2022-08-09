## Architecture
![image](https://user-images.githubusercontent.com/57446639/183696349-c8ff2c90-c0a1-4090-9e5e-14515b0096a4.png)
- Monolithic Architecture: 서비스가 하나의 애플리케이션으로 돌아가는 구조
- MicroService Architecture: 애플리케이션의 각각의 기능을 분리하여 개발 및 관리하는 구조


분산 시스템 환경을 많이 사용하면서 Transaction 보장, 테스트, 배포, 관리가 복잡해졌다.
이 문제를 해결하기 위해 나온게 "이미지" "도커" "쿠버네티스" 라고 볼 수 있다.
- 도커를 사용하면 컨테이너를 편리하게 쓸 수 있게 해주고
- 쿠버네티스를 사용하면 도커를 잘 다룰 수 있게 해준다.


## 용어 정리
이미지: 필요한 프로그램과 라이브러리, 소스를 설치한 뒤 만든 하나의 파일
컨테이너: 이미지를 격리하여 독립된 공간에서 실행한 가상 환경
- 가상머신처럼 하드웨어를 전부 구현하지 않기 때문에 매우 빠른 실행이 가능하다.
- VM 과 컨테이너의 실행 시 CPU 사용량 비교
  - ![image](https://user-images.githubusercontent.com/57446639/183699034-d5150198-b3e9-43d4-9727-1a49f36aa612.png)
  - 출처: https://arxiv.org/pdf/1807.01842.pdf


## Docker
> https://docs.docker.com/
컨테이너 기술을 지원하는 다양한 프로젝트 중에 하나, 사실상 '컨테이너 기술의 표준'
애플리케이션에 국한되지 않고 의존성 및 파일 시스템까지 패키징하여 빌드, 배포, 실행을 단순화
![image](https://user-images.githubusercontent.com/57446639/183700034-7f3f2729-17eb-4f01-8f3a-c17d59ad96b4.png)


## Kubernates, k8s
2014년에 나온 컨테이너 기반 오픈소스 가상화 프로젝트
