# 파드 디자인 패턴과 잡(Job) 실행

주컨테이너를 실시간으로 보조하기 위해 사이드카 컨테이너, 어댑터 컨테이너, 앰배서더 컨테이너가 사용할 수 있다.

## 사이드카 컨테이너

사이드카 패턴: 이 패턴은 오토바이에 연결된 사이드카와 유사하므로 사이드카라고 합니다. 패턴에서 사이드카는 상위 애플리케이션에 연결되고 애플리케이션에 대한 지원 기능을 제공합니다. 또한 사이드카는 상위 애플리케이션과 동일한 수명 주기를 공유하므로 상위 애플리케이션과 함께 만들어지고 사용 중지됩니다.

- [https://learn.microsoft.com/ko-kr/azure/architecture/patterns/sidecar](https://learn.microsoft.com/ko-kr/azure/architecture/patterns/sidecar)

쿠버네티스에서는 파드의 파일시스템을 공유하는 컨테이너를 말한다.

app-container가 파일시스템에 로그를 저장하면 streaming container가 stdout으로 스트리밍한다.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar
spec:
  containers:
  - name: nginx # app-container
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: varlognginx
      mountPath: /var/log/nginx 
  - name: sidecar-access # access.log streaming container
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/nginx/access.log']
    volumeMounts:
    - name: varlognginx
      mountPath: /var/log/nginx
  - name: sidecar-error # error.log streaming container
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/nginx/error.log'] 
    volumeMounts:
    - name: varlognginx
      mountPath: /var/log/nginx
  volumes: # emptyDir을 통해 멀티컨테이너 간 파일 시스템을 공유한다.
  - name: varlognginx 
    emptyDir: {}
```

비지박스(BusyBox)는 유닉스의 명령어 일부를 집대성하여, 타 운영체제에서 유닉스 명령어를 사용할 수 있도록 만들어 주는 소프트웨어

## 어댑터 컨테이너

어댑터 패턴: 클래스의 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴으로, 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.

메인컨테이너에 대한 변경 사항 없이 어댑터 컨테이너를 통해 서비스를 변경할 수 있다.

main container가 로그를 남기면 adapter container가 json 포맷으로 변경하여 전달한다.

- [https://github.com/bbachi/k8s-adaptor-container-pattern](https://github.com/bbachi/k8s-adaptor-container-pattern)

## 앰배서더 컨테이너

앰배서더는 국가나 단체의 이름을 붙이고 해당 국가를 대표하거나 공익단체를 홍보하는 직책을 뜻하는 말이다. 쿠버네이스에서는 파드를 대표한다.

파드에 앰배서더 컨테이너를 배치하여 통신을 대신해주는 역할을 한다. 파드에 앰배서더 컨테이너를 배치하여 인증, 로깅 등 공통적인 역할들을 대신주고 주컨테이너는 기능에만 집중한다.

main container에 GET / 요청을 하면 ambassador container를 통해 다른 서비스에 접근하여 응답을 받아온다.

- [https://github.com/bbachi/k8s-ambassador-container-pattern](https://github.com/bbachi/k8s-ambassador-container-pattern)

## Job과 크론잡

### 잡

사용자를 대신하여 파드 집합을 관리하는 워크로드 리소스이다.

잡은 하나 이상의 파드를 생성하고 지정된 수의 파드가 성공적으로 종료될 때까지 계속해서 파드의 실행을 재시도한다. 반복횟수와 병렬 실행 개수를 지정할 수 있다. 백오프 제한은 기본적으로 6으로 설정되어 있다.

잡이 완료되면 파드가 더 이상 생성되지도 않지만, 일반적으로는 삭제되지도 않는다. 이를 유지하면 완료된 파드의 로그를 계속 보며 에러, 경고 또는 다른 기타 진단 출력을 확인할 수 있다. 잡 오브젝트는 완료된 후에도 상태를 볼 수 있도록 남아 있다. 상태를 확인한 후 이전 잡을 삭제하는 것은 사용자의 몫이다.

### 크론잡

크론잡은 특정시점마다 잡을 만들고 잡은 파드가 만들어 작업을 수행한다. 크론잡은 오직 그 일정에 맞는 잡 생성에 책임이 있고, 잡은 그 잡이 대표하는 파드 관리에 책임이 있다.

`spec.concurrencyPolicy`로 동시성 정책을 설정할 수 있다.
