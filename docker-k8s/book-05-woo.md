# 마이크로서비스 간의 통신

마이크로서비스 앱은 보통 서로 자신의 영역을 책임지고 있는 여러개의 마이크로서비스로 구성되어 있으며 협업을 통해 복잡한 동작을 해낸다. 협업을 위해선 서로 통신할 방법이 필요하다.

마이크로서비스간 협업을 위해 필요한 도커, 도커 컴포즈를 다시 살펴보고, 개발 생산성 향상을 위해 라이브 리로드를 구성한다. 이전 챕터에서 마이크로서비스간 HTTP 요청을 통해 서비스를 제공했다면, 이번엔 직접 메시징(direct messaging)으로 확장해보고, 이를 RabbitMQ를 사용해 간접 메세징(indirect messaging)으로 변경하는 것을 진행한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcvh4fY%2FbtrLaeFt0vp%2FNn7Sz2E7XqDZCIlLHPxJg1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1pAuY%2FbtrK90ncVdm%2F4IRWJ2vRo33ZsUbTqBmD6k%2Fimg.png)

## 히스토리 마이크로서비스 소개

사용자가 시청한 내용을 기록하는 히스토리 서비스를 추가한다. viewed 라는 메세지 이름을 가진 메세지를 video-streaming 마이크로서비스가 history 마이크로서비스에게 전달하고 history 마이크로서비스는 이를 데이터베이스에 기록한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcf8t8B%2FbtrK7CNVmXQ%2FSlGukuK7PdVqSADYdEh3H1%2Fimg.jpg)

## 빠른 개발 주기를 위한 라이브 리로드

-   새로 만든 마이크로서비스에 라이브 리로드 추가
-   개발과 운영 환경의 도커파일 분리
    -   dev
        -   개발 편의를 위해 좀 더 많은 유틸리티를 활용할 수 있는 non-alpine 버전을 사용한다.
            -   라이브 리로드를 지원하는 실행환경을 사용한다.
-   개발과 운영 환경의 도커 컴포즈 파일 분리
    -   dev
        -   로컬 개발머신의 소스파일, npm 패키지 설치 파일을 마운트하여 재설치 시간 줄임

```
// Dockerfile-dev
FROM node:12.18.1-alpine

WORKDIR /usr/src/app
COPY package*.json ./

CMD npm config set cache-min 9999999 && \
    npm install && \
    npm run start:dev


// Dockerfile-prod
FROM node:12.18.1-alpine

WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=production
COPY ./src ./src
COPY ./videos ./videos

CMD npm start
```

```
// docker-compose.yml
version: '3'
services:

  video-streaming:
    image: video-streaming
    build: 
      context: ./video-streaming
      dockerfile: Dockerfile-dev
    container_name: video-streaming
    volumes:
      - /tmp/video-streaming/npm-cache:/root/.npm:z
      - ./video-streaming/src:/usr/src/app/src:z
      - ./video-streaming/videos:/usr/src/app/videos:z
    ports:
      - "4000:80"
    environment:
      - PORT=80
      - NODE_ENV=development
    restart: "no"

  history:
    image: history
    build: 
      context: ./history
      dockerfile: Dockerfile-dev
    container_name: history
    volumes:
      - /tmp/history/npm-cache:/root/.npm:z
      - ./history/src:/usr/src/app/src:z
    ports:
      - "4001:80"
    environment:
      - PORT=80
      - NODE_ENV=development
    restart: "no"
```

## 마이크로서비스의 통신 방법들

직접 메시징(direct messaging), 간접 메시징(direct messaging) 방식이 있으며, 소위 동기식, 비동기식으로 많이 알려져 있다.

-   직접 메세징 : 단순하게 하나의 마이크로서비스가 다른 마이크로서비스로 메세지를 직접 보내고, 바로 응답을 직접 받는 것. 통신의 양단에 두 개의 마이크로비스가 밀접한 연결성(tight coupling)을 갖는 단점이 있음.

\[그림 5.5\]

-   간접 메세징 : 두 마이크로서비스 사이에 중개 역할을 할 중재자를 추가. 통신하는 양측은 실제 서로에 대해 알 필요가 없어 느슨한 연결(looser coupling)이 구성됨.
    -   보내는 사람은 받는 사람이 누군인지 모르고, 받는 사람은 보내는 사람이 누군지 모른다 => 메세지 처리의 성공/실패 여부를 확인할 수 없기 때문에 즉시 응답을 확인해야 하는 경우 사용할 수 없다.
    -   메세지를 보낸 마이크로서비스가 보낸 후의 결과에 따른 동작을 신경 쓸 필요가 없을 경우 사용할 수 있다.
    -   브로드캐스트 할 경우 유용하다.

\[그림 5.6\]

## HTTP를 사용한 직접 메세징

도커 컴포즈는 내부적으로 DNS를 가지고 서비스 네임으로 서비스간 통신이 가능하다.

\[그림 5.8)

```
// 송신부
const postOptions = { // Options to the HTTP POST request.
  method: "POST", // Sets the request method as POST.
  headers: {
      "Content-Type": "application/json", // Sets the content type for the request's body.
  },
};

const requestBody = { // Body of the HTTP POST request.
  videoPath: videoPath 
};

const req = http.request( // Send the "viewed" message to the history microservice.
  "http://history/viewed",
  postOptions
);

req.on("close", () => {
  console.log("Sent 'viewed' message to history microservice.");
});

req.on("error", (err) => {
  console.error("Failed to send 'viewed' message!");
  console.error(err && err.stack || err);
});

req.write(JSON.stringify(requestBody)); // Write the body to the request.
req.end(); // End the request.

// /////////////////////////////////

// 수신부
app.post("/viewed", (req, res) => { // Handle the "viewed" message via HTTP POST request.
  const videoPath = req.body.videoPath; // Read JSON body from HTTP request.
  videosCollection.insertOne({ videoPath: videoPath }) // Record the "view" in the database.
    .then(() => {
        console.log(`Added video ${videoPath} to history.`);
        res.sendStatus(200);
    })
    .catch(err => {
        console.error(`Error adding video ${videoPath} to history.`);
        console.error(err && err.stack || err);
        res.sendStatus(500);
    });
});
```

### 순차적 직접 메세지

직접 메세징 방식이 가지는 잠재적인 방식은 여러개의 마이크로서비스에 대해서 발생하는 복잡한 동작을 조율할 수 있는 하나의 제어용 마이크로서비스를 둘 수 있다는 점. 전체적인 메세지 흐름을 조정하는 역할을 할 수 있다. 명시적 또는 미리 정해진 순서대로 여러 동작을 제어하기 좋다.

\[그림 5.9\]

하지만 한 번에 하나의 마이크로서비스만 대상으로 메세지를 전송할 수 있다. 여러 개의 마이크로서비스가 수신하도록 만들기 어렵다. 또한 두 개의 마이크로서비스가 밀접한 관련성을 가지게 된다.

## 래빗MQ를 사용한 간접 메시징

앱을 이해하기 어려운 구조로 만들 수 있찌만 보안, 시스템 용량 조정과 확장성, 신뢰성, 성능 면에서 장점을 가지고 있다.

\[그림 5.10\]

비디오 스트리밍 마이크로서비스는 히스토리 마이크로서비스와 직접적인 연결 없이 조회 메세지를 메세지 큐에 publish 하고, 히스토리 마이크로서비스는 때가 되면 큐에서 메세지를 가져간다.

-   래빗MQ를 사용하는 이유
    -   10년 이상 지난, 안정적이고 널리 알려진 소프트웨어
    -   message broker 통신을 위한 공개 표준 AMQP를 따름
    -   여러 인기있는 프로그래밍 언어를 지원하는 라이브러리를 지원함

\[그림 5.11\]  
\[그림 5.12\]

-   각각의 마이크로서비스에게 더 나은 제어권을 제공
    -   처리할 준비가 되었을 시점에 메세지를 소비할 수 있음. 만약 과부하 등 여러가지 이유로 메세지를 처리할 여건이 되지 않으면 나중에 처리할 수 있을 떄 까지 메세지를 소비하지 않을 수 있다.

```
  ...
  rabbit:
    image: rabbitmq:3.8.5-management
    container_name: rabbit
    ports:
      - "5672:5672"
      - "15672:15672"
    expose:
      - "5672"
      - "15672"
    restart: always
  ...
  ...
  history:
    ...
    environment:
    - RABBIT=amqp://guest:guest@rabbit:5672
    depends_on:
    - db
    - rabbit      <-- rabbitMQ 서비스에 의존하게 된다.
```

rabbitmq:{tag}-management 이름을 가지는 이미지는 대시보드를 포함하고있다. 경량화 등의 이유로 제공하지만, 운영상 필요한 부분도 있기 때문에 대시보드 버전을 사용하는 것도 좋다.

> 같은 버전 기준 106MB vs 94MB

\[그림 5.13\]

```
function connectRabbit() {
  console.log(`Connecting to RabbitMQ server at ${RABBIT}.`);
  return amqp.connect(RABBIT) // Connect to the RabbitMQ server.
    .then(messagingConnection => {
      console.log("Connected to RabbitMQ.");
      return messagingConnection.createChannel(); // Create a RabbitMQ messaging channel.
    });
}
```

위 상황에서 rabbitmq는 마이크로서비스에비해 무겁고 느려서 연결이 가능할 때 까지 시간이 오래 걸린다. 만약 rabbitMQ가 준비완료되지 않은 상태에서 마이크로서비스가 연결을 시도하면 오류와 함께 마이크로서비스가 중단된다.

내결함성을 가지도록 마이크로서비스를 만들어야 한다. RabbitMQ가 준비되기 까지 기다리고, RabbitMQ 서버가 여러가지 이유로 off되더라도 연결이 끊김으로 발생하는 문제를 처리하고 자동으로 최대한 빨리 다시 연결할 수 있도록 해야한다.

지금 단계에선 `wait-port` 툴을 이용해 간단하게 구현하고 추후에 더 나은 방법을 사용한다.

```
# history/Dockerfile-dev

...
CMD npm config set cache-min 9999999 && \
    npm install && \
    npx wait-port rabbit:5672 && \
    npm run start:dev
...
```

mongoDB 설치떈 왜 해당 작업을 하지 않았는가? -> mongoDB 라이브러리는 자등으로 다시 연결하도록 프로그래밍되어 있다.

### 단일 수신자를 위한 간접 메세징

단일 수신자(single recipient) 메세지 설정은 여러 전송자와 수신자가 포함될 수 있찌만 오직 하나의 마이크로서비스만이 개별 메세지를 수신하는 것을 보장한다. 즉 특정 작업을 오직 한 번만 앱에서 처리한다. -> 작업을 마이크로서비스 풀에 분배할 떄 유용하다.

-   단일 수신자 메세지 받기
    1.  큐를 확보(assert)한다. : 존재하는지 확인하며, 존재하지 않은 경우 큐를 생성한다.
    2.  메세지를 받을 경우 어떤식으로 동작하는지 정의한다.

```
function consumeViewdMessage(msg) {
  const json = JSON.parse(msg.content.toString());
  ...
)

messageChannel.assertQueue("viewed", {}) // viewed 라는 큐를 assert 한다.
  .then(() => {
    console.log("Asserted that the 'viewed' queue exists.");
    return messageChannel.consume("viewed", consumeViewedMessage);  // 메세지 발생 시 실행할 함수를 지정한다.
  });
```

-   단일 수신자용 메세지 보내기

```
messageChannel.publish("", "viewed", Buffer.from(jsonMsg));
```

### 다중 수신 메세지

다중 수신(multiple recipient)나 브로드캐스트(broadcast)같은 형태는 알림이 필요한 경우 사용하면 좋다. 하나의 마이크로서비스가 메세지를 보내고 잠재적으로 많은 다른 서비스들이 수신하는 것이다.

\[그림 5.14\]

```
// consume
return messageChannel.assertExchange("viewed", "fanout")            // 1
  .then(() => {
    return messageChannel.assertQueue("", { exclusive: true });        // 2
  }).then(response => {
    const queueName = response.queue;
    console.log(`Created queue ${queueName}, binding it to "viewed" exchange.`);
    return messageChannel.bindQueue(queueName, "viewed", "")        // 3
             .then(() => {
               return messageChannel.consume(queueName, consumeViewedMessage);
             });
  });
```

-   (1) 다중 수신을 위해선 교환기(exchange)가 필요하다. 교환기는 교환기가 받은 메세지를 다수의 큐로 전송하는 역할이므로 교환기에 붙일 큐가 필요하다.
-   (2) `assertQueue`에서 큐 이름을 지정하지 않으면, `assertQueue`를 호출한 마이크로서비스만 소유할 수 있는 anonymouse 큐가 생성되며 임의의 고유 이름을 가지게 된다.
-   (3) 이렇게 생성한 큐는 exchange에 바인딩되어야만 의미가 있다. 바인딩이 되어야 exchange가 받은 메세지를 큐로 전달받을 수 있다.

```
// publish : 내용은 단일 수신 방식과 같다.
messageChannel.publish("viewed", "", Buffer.from(jsonMsg)); // Publish message to the "viewed" exchange.
```

## 마이크로서비스 통신 다시보기

-   직접 메세징: HTTP
    -   특정 마이크로서비스 이름으로 메세지를 직접 전송한다.
    -   메세지 처리 성공/실패 확인이 필요하다.
    -   첫 번째 메세지 처리르 완료하면 다음 메세지를 순서대로 보낸다.
    -   하나의 마이크로서비스로 다른 마이크로서비스의 동작을 조정하고 싶다.
-   간접 메세징: RabbitMQ
    -   앱 전체의 여러 마이크로서비스로 시스템의 이벤트 메세지를 브로드캐스트하고 싶다.
    -   발신자와 수신자를 직접 연결할 필요 없게 만들고 싶다(독립적으로 쉽게 변경하고 개선하고 싶다)
    -   발신자와 수신자의 성능이 독립적이였으면 좋겠다.(전송자는 필요한 만큼 가능한 많이 생성, 수신자는 자신으 ㅣ속도에 맞게 메세지럴 처리)
    -   메세지 처리에 실패하면 자동으로 성공할 때 까지 메세지를 다시 가져가면 좋겠다(일시적 장애 때문에 메세지를 손실하면 안된다)
    -   메세지를 처리하는 여러 병렬 처리 작업자에 대한 부하 균형이 필요하다

# 운영 환경 구축

-   앱을 실행하는 운영 인프라를 만든다.
-   인프라 구성을 위해 테라폼을 사용한다.
-   쿠버네티스 클러스터를 만들어 마이크로서비스를 호스팅한다.
-   앱은 작은 규모일 때 배포가 쉽다. 커질스록 점점 어려워 진다.

\[그림 6.1\]

## 코드형 인프라

코드형 인프라, Infrastructure as Code는 코드를 사용하여 인프라를 생성하는 기술. 코드를 사용함으로써 재사용이 가능하고 원하는 만큼 반복해서 생성할 수 있음. 일종의 실행 가능한 문서 역할을 함.

절차를 다루는 것이 아니라 선언적인 언어를 사용하는 것. 즉 순서대로 실행할 작업이나 명령 보다는 인프라의 설정과 구성을 기술한다. 어려운 작업은 도구들이 수행한다.

인프라를 위한 코드는 깃과 같은 코드 레포지터리에 저장하고 그 위치에서 코드를 실행해 클라우드 기반의 인프라를 생성, 설정, 관리한다.

\[그림 6.2\]

### 쿠버네티스에 마이크로서비스 호스팅하기

-   쿠버네티스는 컨테이너 코에스트레이션 플랫폼
-   컨테이너의 배포와 확장성을 관리하고 자동화할 수 있음
-   쿠버네티스를 사용하는 이유
    -   특정 기업에 종속되지 않음 - 모든 주요 클라우드 기업들은 각자의 서비스에 적합한 자신만의 컨테이너 오케스트레이션 서비스를 제공하지만 이들 모두 매니지드 쿠버네티스 서비스를 제공하므로 굳이 특정 서비스를 사용할 필요가 없음. 쿠버네티스를 사용하면 어느 클라우드 서비스라도 사용 가능
    -   어느 클라우드 서비스에서 지식을 활용할 수 있음
    -   여러가지 방법으로 확장성 있는 앱을 만들게 해준다 - 10장, 11장에서 소개
    -   제공하는 자동화 API로 자동 배포 파이프라인을 만들 수 있다 - 7장 소개

### 쿠버네티스 동작 원리

-   쿠버네티스는 여러 노드(클라우드에선 가상머신으로 구성)로 이루어져 있다.
-   노드는 여러개의 팟을 호스팅한다. 팟은 쿠버네티스 computation의 기본 단위이다.

\[그림 6.3\]

-   각각의 팟은 하나 이상의 컨테이너를 포함한다. 사이드카 패턴 같은 것들을 구성할 수 있다.

\[그림 6.4\]

## 애저 CLI 사용하기

```
brew update && brew install azure-cli

az --version

az login
```

```
$ az aks get-versions --location westus --output table

KubernetesVersion    Upgrades
-------------------  -----------------------
1.24.3               None available
1.24.0               1.24.3
1.23.8               1.24.0, 1.24.3
1.23.5               1.23.8, 1.24.0, 1.24.3
1.22.11              1.23.5, 1.23.8
1.22.6               1.22.11, 1.23.5, 1.23.8
1.1.1(preview)

# prewview 버전인 아닌 버전을 사용하는 것이 안정적이다.
```

## 테라폼으로 인프라 만들기

HCL(Hashicorp Configuration Language)를 사용해 인프라 코드를 작성한다.

-   테라폼은 플러그인을 제공해 여러 종류의 클라우드 서비스를 HCL로 작성해 운영할 수 있도록 지원한다.
-   HCL은 yaml, json과 비슷한 형식이다.

\[그림 6.5\]

-   테라폼을 사용하는 이유
    -   클라우드 인프라를 신뢰할 수 있는 방법으로 반복해서 설정을 쉽게 다를 수 있다.
    -   플러그인을 제공함으로써 기능을 확장할 수 있는 유연성이 뛰어나다.
    -   다양한 클라우드 서비스를 제공한다.
    -   자동화된 배포 파이프라인을 만들기 위한 모든 작업을 해낼 수 있다.

```
# 설치(https://learn.hashicorp.com/tutorials/terraform/install-cli)

$ brew tap hashicorp/tap
$ brew install hashicorp/tap/terraform

$ terraform -help
```

## 애저 리소스 그룹 만들기

지금까지 생성했던 애저 리소스들(스토리지, 컨테이너 레지스트리 등)을 그룹화할 애저 리소스 그룹을 생성한다. - 테라폼으로 애저 리소스 그룹을 생성한다.

\[그림 6.7\]

테라폼은 반복적인 형태로 인프라를 구성하는 도구. 이를 진화하는 설계(evolutionary architecture)라고 한다. 인프라를 진화시키는 과정을 진행할 것이다.

\[그림 6.8\]

```
provider "azurerm" {
    version = "3.0.1"
    features {}
}
resource "azurerm_resource_group" "flixtube" {
  name     = "flixtube"
  location = "West US"
}
```

```
# 테라폼 환경을 초기화한다. 설치에 필요한 플러그인들을 인식해 다운로드(.terraform/~~)한다.
$ terraform init
```

-   init 명령 실행 이후 생성된 .trafform 폴더에 캐싱된 플러그인 파일들이 저장된다.
-   위에 작성한 `provider` 부분은 작성하지 않아도 다른 코드들을 보고 init 시에 최신 버전으로 세팅해주지만 버전 호환성등의 이유로 명시하는 것이 좋다.

```
# 실제 테라폼 코드를 인프라에 적용한다. 중간 확인 단계가 있다.
$ terraform apply
## apply 이후 생기는 ~.tfstate 파일에 적용된 상태를 반영하고 이는 곧 다음 명령어 실행의 input이 된다.

# 스크립트로 구성된 인프라를 삭제한다.
$ terraform destroy
```

[![asciicast](https://asciinema.org/a/521327.svg)](https://asciinema.org/a/521327)

\[그림 6.10\]

-   테라폼은 tfstate 파일을 통해 자신이 관리하는 인프라에 대해서만 관여한다.

## 컨테이너 레지스트리 생성

```
resource "azurerm_resource_group" "flixtube" {
  name     = "flixtube"
  location = "West US"
}
...
resource "azurerm_container_registry" "container_registry" {
  name                = "flixtube"
  resource_group_name = azurerm_resource_group.flixtube.name   <-- 위 resource 블록을 변수처럼 사용하였다
  location            = "westus"
  admin_enabled       = true
  sku                 = "Basic"
}

output "registry_hostname" {         <-- 출력을 생성한다
  value = azurerm_container_registry.container_registry.login_server
}

output "registry_un" {
  value = azurerm_container_registry.container_registry.admin_username
}

output "registry_pw" {
  value = azurerm_container_registry.container_registry.admin_password
  sensitive = true
}
```

[![asciicast](https://asciinema.org/a/qwlF514alsHu1mAFOpC35BRIU.svg)](https://asciinema.org/a/qwlF514alsHu1mAFOpC35BRIU)
