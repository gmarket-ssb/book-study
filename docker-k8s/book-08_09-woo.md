## 제스트를 사용한 테스트

제스트는 페이스북에서 만든, 자바스크립트 코드를 테스트할 때 활용할 수 있다. 테스트 코드를 순서대로 로드하고, 테스트할 코드를 실행해서 예상대로 동작했는지 결과를 검증한다. 자바스크립트 테스트 프레임워크로 가장 인기있는 도구이며 빠른 병렬 테스트, 라이브 리로딩이 가능하다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcLU1Os%2FbtrPI8ACbaL%2FWHdZ0CDkxsKTb7n8aevGD0%2Fimg.jpg)

```
# 설치 : 운영환경에서 설치되지 않도록 --save-dev 사용
$ npm install --save-dev -jest

# 초기화
$ npx jest --init
```

-   최상위 폴더의 `jest.config.js` 파일에 테스트 관련 설정을 한다.
-   `clearMocks` 설정을 `true`로 두어 테스트간에 영향을 받지 않도록 한다.
-   `{filename}.test.js` 로 테스트 파일을 네이밍하는 것이 관례이다. jest에선 기본적으로 이 네이밍으로 테스트파일을 인식하며 설정에서 변경할 수 있다.
-   src 하위 디렉토리에 위치한 test 또는 tests 폴더 아래에 위치시킨다.

```
// src/math.js
function square(n) {
  return n * n;
}

module.exports = {
  square,
};
```

```
// src/math.test.js
const { square } = require("./math");
describe("square function", () => {
  test("can square two", () => {
    const result = square(2);
    expect(result).toBe(4);
  });
});
```

테스트 실행 및 결과 출력 화면

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FTVsXt%2FbtrOqjdMgSX%2FzZ0MerZTpKLQFsEWrmfIq0%2Fimg.png)

`--watchAll`, `--watch` 명령어를 사용하면 테스트 파일이 변경되었을 때 자동으로 인식해 테스트를 진행하는 `라이브 리로딩 기능`이 활성화 된다. `--watchAll`은 하나의 파일이라도 변경이 있으면 전체 테스트를, `--watch`는 변경된 파일의 테스트만 진행한다.

테스트가 성공 시 jest는 **종료코드 0** 을 반환하며 실패 시 **0이 아닌 코드**를 반환한다. 이 코드는 CD 플랫폼에서 테스트를 실행하여 테스트 후 과정을 어떻게 진행할지 판단할 때 사용된다.

규모가 큰 테스트는 코드가 예상대로 잘 동작하는지 의심의 여지가 없도록 증명이 가능한 하나의 자동화된 검증 시스템을 갖는 것. 가장 중요한 것은 코드가 진화함에 따라 앞으로도 코드가 잘 동작할 지 증명하는 것.

npm 스크립트에 jest를 사용한 테스트를 실행할 수 있도록 수정할 수 있다.

```
// npm.js
{
  "name": "example-1",
  "version": "1.0.0",
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watchAll"
  },
  "devDependencies": {
    "jest": "^26.2.2"
  },
  "dependencies": {}
}
```

## 단위 테스트

마이크로서비스의 단위테스트도 다른 아키텍처의 단위 테스트와 동일함. 나머지 코드와 격리된 단일 코드 단위로 테스트하는 것을 말함. 여기서 단위란 주로 '하나의 함수나 단일 기능을 테스트하는 것'. 단위 테스트에서 중요한 것은 격리(isolation). 구분된 기능의 코드를 테스트하고 코드 영역에 대한 테스트에 집중하는 것. 다른 종속적인 코드는 이미 테스트가 완료된 것으로 간주하고 진행. 격리는 단위 테스트를 빠르게 한다.

통합과 E2E 테스트는 코드를 격리하지 않는다. 이러한 테스트는 코드 모듈들의 통합을 시험하기 때문.

단위 테스트는 실제 HTTP 서버나 데이터베이스에 연결하지 않으며 여러번 실행해볼 수 있어야 한다. 또한 빠르게 끝나야 한다.

-   테스트 대상 코드 : [index.js](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-2/src/index.js)
-   테스트 코드 : [https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-2/src/index.test.js](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-2/src/index.test.js)
    -   mongodb 커넥션은 jest에서 지원하는 mock을 사용했다.
    -   [58~64: microservice starts web server on start up](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-2/src/index.test.js#L58-L64) : mocking한 listener의 값들을 확인했다.
    -   [66~74: /video route is handled](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-2/src/index.test.js#L66-L74) : 호출 여부를 판단
    -   [76~10: /video route retreives data via videos collection](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-2/src/index.test.js#L76-L105) : 호출 결과값을 비교

다음 단계로는 제스트를 이용해 통합 테스트를 작성한다. 이전처럼 마이크로서비스에서 코드를 직접 호출하는 대신, HTTP 요청으로 코드를 실행하여 테스트를하고, 이를 위해 http 요청 자바스크립트 라이브러리인 axios를 사용한다.

[https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-3/src/index.test.js#L58-L88](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-3/src/index.test.js#L58-L88)

## 통합 테스트

통합테스트란 단위 테스트처럼 격리된 모듈을 테스트하는 대산 하나의 연결된 형태로 기능의 연계된 동작을 테스트하는 것. 마이크로서비스의 통합테스트는 보통 모든 코드 모듈과 함꼐 관련된 코드 라이브러리까지 포함하는 전체적인 마이크로서비스를 테스트

코드 모듈이 상호 동작하면서 발생하는 문제점들을 단위테스트는 발견하지 못할 수 있음

mock 작업을 하지 않기 때문에 단위테스트보다 작성이 더 쉬울 수 있음. 환경적인 면에서 설정이 더 까다로울 수 있다. 실제 mongodb를 사용하거나 하나의 HTTP 서버를 여러 테스트가 공유해야되는 등.

1.  통합테스트를 위해서 mongodb를 설치한다.
2.  헬퍼 함수등을 사용해서 테스트에 필요한 초기 데이터(fixture)를 설정한다.

```
async function loadDatabaseFixture(collectionName, records) {
  await microservice.db.dropDatabase(); // Reset the test database.

  const collection = microservice.db.collection(collectionName);
  await collection.insertMany(records); // Insert the database fixture.
}
```

도커컴포즈를 사용하여 테스트 데이터를 가진 데이터베이스를 사용하고 Cypress라는 웹 페이지 테스트 도구를 사용한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbET98Z%2FbtrPI8UWY0z%2F3520wn5lhHKhdWQIzK2QNk%2Fimg.jpg)

E2E 테스트는 앱을 모두 먼저 시작하고 웹 브라우저로 테스트한다. 이렇게 진행하기 때문에 테스트 방식 중 가장 느리고 비용이 많이 들지만 고객 관점에서 앱의 앞단부터 검사해 나아가며 중요한 의미를 가진다.

싸이프러스 특징

-   일렉트론(Electron) 프레임워크를 기반으로 한다. -> 설치 시 시간, 용량이 필요하다.
-   웹 페이지의 테스트를 위한 모든 것을 제공한다.
-   사용자 인터페이스가 편하고 시각화가 가능해 진행사항을 관찰할 수 있다.
-   크롬을 기본적으로 사용하지만 실행 머신의 브라우저도 자동으로 감지해 여러 브라우저에서 테스트할 수 있다.
-   Headless를 지원한다.
-   Live Reload를 지원한다.
-   오픈소스이다.
-   용량이 커 CD 파이프라인에서 설치 시 시간이 오래걸릴 수 있다.

> cypress 같은 통합 테스트 툴은 사이즈가 큰편이라 별도의 프로젝트에서 구성을 많이한다.

```
$ npm install --save-dev cypress
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbqy8lW%2FbtrPIhd3EdZ%2FEAopZa9oQN9YeOdhrsKMN0%2Fimg.png)

-   싸이프러스는 브라우저상에서 실행되기 때문이 이전처럼 데이터 로딩을 위한 로직직을 별도로 실행시킬 수 없으므로 외부 API를 구성하고 Cypress에서 호출하도록 구성한다.
-   싸이프러스는 실행되면 초기 데이터 세팅을 위해 위 과정에서 생성한 API를 호출하고, 테스트 대상 웹을 실행한다.
-   데이터 세팅 모듈 정의 : [command.js](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-4/cypress/support/commands.js#L27-L68)
-   E2E 테스트 코드 작성 : [front-end.spec.js](https://github.com/bootstrapping-microservices/chapter-8/blob/master/example-4/cypress/integration/front-end.spec.js#L5-L26)

`npm test`로 위 테스트가 실행되도록 설정한다.

## CD 파이프라인 자동 테스트

개발자가 code repository에 변경 사항을 반영하면, 자동으로 코드의 품질을 검사하기 위한 테스트 세트를 실행하도록 구성한다. 테스트를 통과한 경우 코드는 운영으로 넘어가며 만약 실패하면 코드는 배포되지 않는다.

\[그림 8.13\]

위에서 정의한 `npm test`만 파이프라인에 끼워넣으면 테스트 성공/실패 여부에 따라 배포 여부가 결정되는 빌드 파이프라인을 구성할 수 있다.

```
image: hashicorp/terraform:0.12.29

pipelines:
    default:
      - step:
          name: Build and deploy
          services:
            - docker
          script:
            - export VERSION=$BITBUCKET_BUILD_NUMBER
            - cd video-streaming && npm install && npm test
            -  chmod +x ./scripts/deploy.sh
            - ./scripts/deploy.sh
```

# 플릭스튜브 탐색

[##_Image|kage@vrDhO/btrPJpvqrn0/R99UkY8rN5FTKKOzKqyRLk/img.png|CDM|1.3|{"originWidth":3602,"originHeight":2398,"style":"alignCenter","width":627,"height":417}_##]

## 개요

완성된 플릭스튜브 예제 앱

[##_Image|kage@VTjzm/btrPHyz856g/aESY8tHLkUNf9BMmSjNTM0/img.png|CDM|1.3|{"originWidth":4032,"originHeight":3024,"style":"alignCenter"}_##]

[예제 프로젝트](https://github.com/bootstrapping-microservices/chapter-9) 구조

\[그림 9.4\]

예제는 단순하게 만들기 위해 단일 코드 레포를 사용했다. 하지만 운영 환경의 마이크로서비스는 단일 코드 레포를 가지면 안된다. 독립적으로 배포할 수 있다는 마이크로서비스의 제일 큰 장점을 포기하는 것이기 때문이다. 실제 마이크로서비스는 대부분 서비스마다 구분된 레포를 가진다.

## 개발 환경에서 플릭스튜브 실행하기

\[그림 9.5\]

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FebveEv%2FbtrPIhE91xB%2Fll4xKkMYhnrUbsUT3rrBg1%2Fimg.png)

### 게이트웨이

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcetdjX%2FbtrPJd9Ckcp%2FrwWdw3GaLnI3uo4D5lK5w0%2Fimg.png)

클러스터 안으로 HTTP 요청을 전달하는 것. 더 나아가 API 라우팅 기능을 제공하는 게이트웨이도 있다.

-   프론트엔드 사용자에게 인터페이스를 제공한다.
-   현재는 인증을 지원하지 않지만 게이트웨이에서 인증을 제공해서 통로역할을 할 수 있다.
-   각각의 프론트엔드는 자신만의 게이트웨이를 가진다. 모바일은 모바일 게이트웨이, 앱은 앱 게이트웨이를 사용함으로써 프론트별 특성에 따른 서비스를 제공할 수있다. backends for front ends. ex) 앱은 별도의 인증 방식이 필요하다, 웹은 다른 보안 설정이 필요하다 등

[https://github.com/bootstrapping-microservices/chapter-9/blob/master/example-1/gateway/src/index.js#L17](https://github.com/bootstrapping-microservices/chapter-9/blob/master/example-1/gateway/src/index.js#L17)
