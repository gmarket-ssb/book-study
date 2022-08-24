# 02. 데이터 모델과 질의 언어

> “내 언어의 한계는 내 세계의 한계를 의미한다” - 비트겐슈타인

데이터 모델은 소프트웨어 개발에서 제일 중요한 부분
- 소프트웨어가 어떻게 작성됐는지와 해결하려는 문제를 어떻게 생각해야 하는지에 영향

대부분의 애플리케이션은 하나의 데이터 모델을 다른 데이터 모델 위에 계층을 둬서 만드는 것이 일반적

각 계층의 핵심적 문제는 다음 하위 계층 관점에서 데이터 모델을 __표현__ 하는 방법
- 애플리케이션 개발자는 현실을 보고 객체나 데이터구조, API를 모델링 (for 애플리케이션)
- 데이터 구조 저장 시에는 범용 데이터 모델로 표현 (JSON/XML 문서, 관계형 DB 테이블, 그래프 모델) => 2장 내용
- DB SW 개발하는 엔지니어는 위 데이터를 메모리나 디스크, 네트워크 상의 바이트 단위로 표현하는 방법 결정 (표현은 다양한방법으로 질의,탐색,조작,처리 등을 가능하게함) => 3장
- HW 개발 엔지니어는 전류, 빛의파동, 자기장등의 관점에서 바이트를 표현하는 방법 발견

복잡한 어플리케이션에서는 여러 단계의 API를 더 둘 수 있지만, 결국 각 계층은 명확한 모델로 하위 계층의 복잡성을 숨긴다. 이를 추상화라고 한다.

이런 추상화는 다른 그룹의 사람들이 효율적으로 일할 수 있게 돕는다. (DBA 와 게발자 등의 관계)

----
## 관계형 모델과 문서 모델

### 관계형 모델

1970년 에드가 코드가 제안한 Relation(Table) + Tuple(Row) 모델
- RDBMS + SQL 로 오늘날까지 주로 사용됨
- 메인프레임에서 수행된 비지니스 처리를 위해 사용되기 시작
- 트랜잭션(은행 거래, 항공권 예약 등), 일괄처리(급여 지불, 고객 송장 작성 등) 오늘날의 상당수 서비스들이 관계형 데이터베이스를 통해 제공된다

관계형 모델의 목표
> “정리된 인터페이스 뒤로 구현 세부사항을 숨기는 것”

접근 방식의 경쟁
- 네트워크 모델, 계층 모델 제시 (1970-80s)
- 객체 데이터 베이스 반짝 (1980-90s)
- XML 데이터베이스 인기no (2000s)
- 결국은 관계형 모델이 오랜시간 우위 지속
- NoSQL은? (2010s) => NoSQL의 탄생

컴퓨터의 발전 & 네트워크화로 인한 변화
- 컴퓨터의 목적이 다양해짐
- 관계형 데이터베이스 => 비즈니스 데이터 처리를 넘어 더 다양한 목적으로 사용
- 

### NoSQL의 탄생
NoSQL 이란?
- 처음엔 인기 트위터 해시태그 #NoSQL
- “Not Only SQL” 로 재해석됨

- 대규모 데이터셋이나 높은 쓰기 처리량 등에서 관계형 데이터베이스 뛰어난 확장성
- 상용 DBMS보다 오픈소스에 대한 선호
- 특수한 질의 동작
- 동적이고 표현력이 풍부한 데이터 모델

### 다중 저장소 지속성(polyglot persistence)
- 애플리케이션의 요구사항에 따라 적절한 다양한 데이터 저장소를 함께 사용
- https://trino.io/
  ![image](https://user-images.githubusercontent.com/8626130/186179134-624a9f4d-0460-4422-87be-c840763d6441.png)

### 객체 관계형 불일치

대부분의 애플리케이션은 OOP 언어로 개발된다
- 이를 RDB에 저장하려면 거추장스러운 변환 계층이 필요

> Impedance mismatch
> 두 회로를 연결시 저항이 불일치할때 발생하는 문제
> OOP로 작성된 모델 <-> 관계형 모델간 괴리로 발생하는 문제

Hibernate같은 ORM 프레임워크를 이용해 boilerplate code(전환을 위해 필요한 상용구 코드들)를 상당수 숨길 수 있지만 두 모델의 차이를 완전히 숨길 수는 없다

#### 이력서의 표현

- 관계형 모델 

<img width="980" alt="image" src="https://user-images.githubusercontent.com/8626130/186180380-8d675e12-9ed1-44a6-80c6-be855230706f.png">

- JSON

문서로 표현하기 적합하다. 관계형 모델의 경우에는 다중 질의를 해야 하나, JSON의 경우에는 모든 관련정보를 하나의 위치에서 표현하고 있어서 한번의 질의로 확인 할 수 있다.
-> 지역성(Locality)이 더 낫다고 한다.

다만 스키마 유연성에서의 단점이 있다. (뒤에서 설명)

  <img width="921" alt="image" src="https://user-images.githubusercontent.com/8626130/186180922-1cc2b853-34d6-41f7-b813-25571d3deb63.png">
  <img width="1003" alt="image" src="https://user-images.githubusercontent.com/8626130/186180975-02399bc5-1afe-4b42-ad61-e93b2965cd4c.png">

### 다대일과 다대다관계
- 이력서 예제에서 region_id와 inderstry_id를 평문 대신 ID로 저장사례

의미 있는 정보를 한곳에만 저장하고 이를 ID로 참조 join을 통해 참조
- 중복제거 (정규화)
- 데이터 중복 시 중복된 데이터를 모두 수정해야 해서 쓰기 오버헤드 발생

중복된 데이터를 제거하면 다대일(Many to One) 관계가 필요하다.
문서 모델의 경우에는 다대일 관계가 적합하지 않다.

<img width="680" alt="image" src="https://user-images.githubusercontent.com/8626130/186184064-165f3736-789b-4f63-b226-0127eab05a98.png">

문서 데이터베이스의 JOIN 은 보통 지원하지 않는다.
- Application 에서 직접 구현하는 방법도 있다.

개발 초기에 Join 할 일이 없어, 문서모델로 충분하다고 하더라도 점차 시간이 지나며 기능이 추가되고, 데이터가 점차 상호 연결 될 수 있다.

#### 다대다 관계
<img width="980" alt="image" src="https://user-images.githubusercontent.com/8626130/186186404-c34a8a6f-996d-4ba8-acf3-b6556ac53c9c.png">

----
## 문서 데이터베이스는 역사를 반복하고 있나?
Many-to-many관계를 표현하는 방법에 대한 논쟁

- 관계형 모델의 경우 Many-to-many와 다중 Join이 수월
- NoSQL 문서 모델의 경우 명확한 해결 방법이 없다

### IBM IMS - 계층모델
- 문서 모델과 비슷하게 One-to-many에서 잘 작동
- JSON 과 유사

Many-to-many는 아래 중 하나 선택
1. 데이터 중복
2. 레코드간 참조를 수동으로 해결

이를 해결하기 위해 두가지 모델이 대립
- 관계형 모델
- 네트워크 모델
- 여기서 관계형 모델이 주도권을 잡고 세상을 지배했다

### 네트워크 모델 - CODASYL 모델
- https://ko.wikipedia.org/wiki/CODASYL

![image](https://user-images.githubusercontent.com/8626130/186189806-e0dfd80d-8b1c-435b-82eb-fc43dc034398.png)

- 다중 부모 가능
- Many-to-one, Many-to-many 관계 모델링 가능
- 레코드간 연결은 Foreign key 보다는 프로그래밍 언어의 Pointer에 가까움
- 최상위 레코드에서부터 연속된 연결경로를 따라간다.
- Many-to-many관계에서 다중 부모를 가지는 경우 맨 앞에서 다양한 접근 경로를 계속 추적해야함

> 알고리즘을 잘 작성하면 해결됬겠지만, 이때는 1970년도... 테이프 디스크 사용하던시절
![image](https://user-images.githubusercontent.com/8626130/186191253-47782d63-08d4-43a2-83d0-5d14c97b9bff.png)

수동 접근 경로
- 직접 경로를 사용자가 구현한다. 제한된 하드웨어 성능을 효율적으로 사용 가능
- 데이터베이스 질의와 갱신을 위한 코드가 복잡하고 유연하지 못함
- 원하는 데이터에 대한 경로가 없다면, 접근 경로를 변경하기 위해 수 많은 질의 코드 확인 후 재작성 필요

### 관계형 모델

- 중첩된 구조와 데이터를 위해 따라가야 하는 복잡한 접근 경로가 없음
- 조건과 일치하는 테이블의 일부 또는 전부를 조회 가능
- Foreign key와 무관하게 임의의 테이블에 데이터 삽입 가능
- 접근 경로는 Query Optimizer가 자동으로 만든다
- Index를 생성하고 사용하기 위해 질의를 바꿀 필요가 없다(Optimizer가 자동 선택)

### 문서 데이터베이스와 비교
- 오늘날에는 관계형 DB의 FK 는 문서 DB에서 Document Reference로 유사하게 지원한다.
- https://www.mongodb.com/docs/v5.3/reference/database-references/
- https://www.bearpooh.com/164

----
## 관계형 데이터베이스와 오늘날의 문서 데이터베이스
- 내결함성과 동시성처리를 포함해 고려해야 할 차이점이 많다.
- 본 장에서는 데이터 모델의 차이점에만 집중한다.

### 문서모델의 선호이유 
- 스키마유연성, 지역성에 기인한 더 나은 성능, 어플리케이션의 데이터 구조와 더 가까움 등

### 관계형모델의 선호이유
- 조인, 다대일, 다대다를 더 잘 지원함 -> 중복성 최소화, 기능확장에 대응

### 어떤 데이터 모델이 애플리케이션 코드를 더 간단하게 할까?
a) 데이터가 문서랑 비슷한 구조일 경우 => 문서 모델
- 여러 테이블로 찢는(shredding) 관계형 기법은 불필요한 복잡도
b) 다대다 관계 사용 => 관계형 사용
- 비정규화 데이터의 일관성을 유지하기 위한 코드 복잡도 및 다중 요청 (성능 ↓)
c) 상호 연결이 많은 데이터 => 관계형은 무난, 그래프 모델 사용

### 문서 모델에서의 스키마 유연성
스키마 강요
- JSON (문서, 관계형) : 스키마 강요 X
- XML (관계형) : 선택적 스키마 유효성 검사 포함

문서 데이터베이스는 스키마리스(Schemaless) ?
- 임의의 키와 값을 문서에 추가 가능
- 읽을 때 필드 존재 여부를 보장하지 않음

#### 스키마 접근 방식
- 쓰기 스키마 (schema-on-write) : 스키마는 명시적이고 DB는 모든 데이터가 스키마를 따름을 보장 (RDB 접근 방식, Statically typed) -> 엄격한 타입 제한과 구조화, 별도의 Alter Table 필요

  <img width="930" alt="image" src="https://user-images.githubusercontent.com/8626130/186197929-730021aa-d15c-46fb-a8f6-6d6fcfeafa4e.png">

- 읽기 스키마 (schema-on -read) : 데이터 구조는 암묵적이고 데이터를 읽을 때만 해석
(= Dynamically typed) -> 읽는 Application의 데이터 필드드만 추가하면 됨

  <img width="868" alt="image" src="https://user-images.githubusercontent.com/8626130/186197868-6813d977-ad77-408b-b4bb-d9f228142cf6.png">

### 질의를 위한 데이터 지역성
1. 저장소 지역성 (storage locality)
- 한번에 해당 문서의 많은 부분을 필요로 하는 경우 좋다.

2. 문서 모델의 저장소 지역성
- 문서의 작은 부분 접근시에도 전체 문서 적재(디스크->메모리)
- 문서를 갱신시에 전체 문서를 재작성
- 큰 문서에서는 낭비일 수도 (갱신시에도 전체 문서 적재)
- 다만, 문서의 크기를 바꾸지 않은 수정은 쉽게 수행 됨
- 문서는 작게 유지하면서 크기가 커지는 쓰기를 피히도록 권장

https://www.mongodb.com/blog/post/schema-design-for-time-series-data-in-mongodb

3. 문서 모델이 아닌 경우의 지역성
- 구글의 Spanner DB 의 로우 교차 배치 스키마 (ref $28)
- 오라클의 다중 테이블 색인 클러스터 테이블(multi-table index cluster table)
- 빅테이블(Bigtable) 데이터 모델의 칼럼 패밀리(column-family) 개념

### 문서 데이터베이스와 관계형 데이터베이스의 통합
- 다양한 RDB가 문서 지원함
- https://www.postgresql.org/docs/current/datatype-json.html

----
## 데이터를 위한 질의 언어
명령형 언어는 특정 순서로 특정 연산을 수행하라고 일일이 모두 정의하고 지시한다.

- 결과를 결정하기 위한 알고리즘을 모두 지정
- 멀티 코어 병렬 처리가 어려움
- cf) 네트워크 모델류의 DB와 IMS 가 이런 형태를 사용함
- ex) 특정 html태그를 javascript로 DOM API 를 사용해서 스타일링하는 것. 더 좋은 api를 사용하려면 코드바꿔야함


SQL, 관계대수(relational algebra) 같은 선언형 언어는 결과를 충족해야하는 조건과 변환을 지정해주기만 하면 된다.
- 내부적으로 어떤 순서로 어떤 연산을 수행할지는 쿼리 최적화기가 알아서 한다.
- 멀티 코어 병렬 처리도 알아서 활용
- ex) 특정 html태그를 css(선언형)로 꾸미는것. 브라우저 벤더가 알아서 최적화함.

참고 : https://sjh836.tistory.com/196

### 맵리듀스 질의
- 10장 진행
- 많은 문서를 대상으로 읽기 전용 질의 수행시 사용 함

![image](https://user-images.githubusercontent.com/8626130/186202483-583f3ff0-909d-4bd5-9ecd-03d7f002d6dc.png)


POSTGRES

<img width="872" alt="image" src="https://user-images.githubusercontent.com/8626130/186202233-f1743df4-2334-4d0d-9808-ea6a2f5b4c04.png">

> 상어과에 속하는 종만 보이도록 관측치를 필터링 한 다음, 관측치가 발생한 달력의 월로 그룹화하고, 해당 달의 모든 관측치에 보여진 동물 수를 합친다.


MongoDB

<img width="762" alt="image" src="https://user-images.githubusercontent.com/8626130/186202879-33948ea9-a1d3-4fa7-98d6-34115010188c.png">

### 그래프형 데이터 모델
<img width="943" alt="image" src="https://user-images.githubusercontent.com/8626130/186203282-8568e4a4-d712-44a1-a8ff-e9237514526e.png">

정점(Vertex, node, entity), 간선(edge, 관계, 호(arc)) 객체로 이뤄진다.
- Neo4J, Titan 등에서 사용된다.

각 객체생성
<img width="689" alt="image" src="https://user-images.githubusercontent.com/8626130/186203869-2f480f55-b2fc-4918-add0-19b13c5ef4b8.png">

#### 사이퍼 질의 언어
그래프형 데이터 모델에서 사용하는 질의 언어

데이터 삽입
- 맨 왼쪽 정점과 간선 표현 대륙 -> 북아메리카 -> 국가 -> 미국 -> 주 -> 아이다호
<img width="785" alt="image" src="https://user-images.githubusercontent.com/8626130/186204447-ad6bfc26-50c1-456d-91b8-4e4c1b1c0361.png">

데이터 조회
- 미국에서 유럽으로 이민 온 사람을 찾는 사이퍼 질의
<img width="1006" alt="image" src="https://user-images.githubusercontent.com/8626130/186205259-0e7a2e6a-cc7c-42a6-854b-1e70edac091f.png">

- 다음 두 가지 조건을 만족하는 정점(Person)을 찾아라
1. person은 어떤 정점을 향하는 BORN_IN 유출 간선을 가진다. 이 정점에서 name 속성이 United States 인 Location 유형의 정점에 도달 할 때 까지 일련의 WITHIN 유출 간선을 따라간다.
2. 같은 Person 정점은 LIVES_IN 유출 간선도 가진다. 이 간선과 WITHIN 유출 간선을 따라가면 결국 name 속성이 Europe 인 Location 유형의 정점에 도달한다.

## 트리플 저장소 모델
RDF
![image](https://user-images.githubusercontent.com/8626130/186206096-1fc932a3-6eaa-4f15-b3a1-52b39785843b.png)
- https://www.w3.org/TR/rdf-sparql-query/
- https://ko.wikipedia.org/wiki/%EC%8B%9C%EB%A7%A8%ED%8B%B1_%EC%9B%B9
- https://github.com/solid
- https://solidproject.org/
