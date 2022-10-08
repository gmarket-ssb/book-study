# 파드 디자인 패턴과 잡(Job) 실행

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

어댑터패턴: 클래스의 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴으로, 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.

메인컨테이너에 대한 변경 사항 없이 어댑터 컨테이너를 통해 서비스를 변경할 수 있다.

main container가 로그를 남기면 adapter container가 json 포맷으로 변경하여 전달한다.

- [https://github.com/bbachi/k8s-adaptor-container-pattern](https://github.com/bbachi/k8s-adaptor-container-pattern)
