# 7장. 트랜잭션

트랜잭션은 신뢰성을 지닌 시스템 구현하기 위한 다양한 문제들을 단순화하는 메커니즘으로 채택돼 왔다. 트랜잭션을 사용함으로써 애플리케이션에서 어느정도의 잠재적인 오류 시나리오와 동시성 문제를 무시할 수 있다.

모든 애플리케이션에서 트랜잭션이 필요하지는 않으며 때로는 트랙잰션적인 보장을 완화하거나 아예 쓰지 않는게 이득이다. 예를 들어 성능을 향상시키거나 가용성을 높일 수 있다.

## 애매모호한 트랜잭션의 개념

새로운 데이터베이스 중 다수는 트랜잭션을 완전히 포기하거나 과거에 인식되던 것보다 훨씬 약한 보장을 의미하는 단어로 트랜잭션의 의미를 재정의했다. 비관계형 데이터베이스는 새로운 데이터베이스 모델을 채택하고 기본적으로 복제와 파티셔닝 기능을 제공히였고 기존 트랜잭션 개념에 많은 영향을 미쳤다.

### ACID의 의미

트랜잭션이 제공하는 안전성 보장은 흔히 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)을 의미하는 약어인 ACID로 잘 알려져 있다.

오늘날 현실에서는 데이터베이스마다 ACID 구현이 제각각이여서 ACID를 준수한다고 할 때 그 시스템에서 실제로 어떤 것을 기대할 수 있는지 분명하지 않다. 거의 마켓팅 용어가 돼버렸다.

**원자성(Atomicity)**

원자성은 컴퓨터의 여러 분야에서 비슷하지만 미묘하게 다른 것을 의미한다. 쓰레드에서는 원자성은 동시성과도 관련이 있다.

여러 쓰기 작업이 하나의 원자적인 트랜잭션으로 묶여 있는데 결함 때문에 완료(커밋(commit))될 수 없다면 어보트(abort)되고 데이터베이스는 이 트랜잭션에서 지금까지 실행한 쓰기를 무시하거나 취소해야 한다.

**일관성(Consistency)**

일관성이란 단어는 굉장히 여러 의미로 쓰인다. (e.g. 복제 일관성, 최종적 일관성, 일관성 해싱 등)

ACID 일관성은 유효한 데이터베이스에서 시작하고 트랙잭션에서 실행된 모든 쓰기가 유효성을 보존한다면 불변식이 항상 만족해야 한다는 의미이다. 

일관성은 실제로는 데이터베이스 속성이 아닌 애플리케이션의 속성이다. 일관성의 아이디어는 애플리케이션의 불변식에 의존하며 일관성을 유지하도록 트랜잭션을 올바르게 정의하는 것도 애플리케이션의 책임이다. 

**격리성(Isolation)**

ACID에서 격리성은 실행되는 트랜잭션은 서로 격리된다는 것을 의미한다.

직렬성 격리(serializable isolation)는 성능 손해를 동반하므로 현실에서는 거의 사용되지 않는다. 오라클 11g 같은 대중적인 데이터베이스 중에는 아예 구현조차 하지 않는 것도 있다. 직렬성이라는 격리 수준은 있지만 실제로는 직렬성보다 보장이 약한  스냅숏 격리를 구현한 것이다.

**지속성(Durability)**

지속성은 트랜잭션이 성공적으로 커밋됐다면 하드웨어 결함이 발생하거나 데이터베이스가 죽더라도 트랜잭션에서 기록한 모든 데이터는 손실되지 않는다는 보장이다.

단일 노드 데이터베이스에서 지속성은 일반적으로 데이터가 하드디스크나 SSD 같은 비휘발성 자장소에 기록됐다는 뜻이다.

복제 기능이 있는 데이터베이스에서 지속성은 데이터가 성공적으로 다른 노드 몇 개에 복사됐다는 것을 의미한다.

완벽한 지속성은 존재하지 않는다. 여러가지 기법을 함께서 위험을 줄여야 한다.
### 단일 객체 연산과 다중 객체 연산

ACID에서 원자성과 격리성은 클라이언트가 한 트랜잭션 내에서 여러 번의 쓰기를 하면 데이터베이스가 어떻게 해야 하는지를 서술한다.

원자성은 트랜잭션을 실패하면 부분 실패를 걱정할 필요 없게 해준다.

다중성은 일관성이 때진 중간 지점을 보는 일을 없게 해준다.

**다중 객체 연산**

다중 객체 트랜잭션은 흔히 데이터의 여러 조각이 동기화된 상태로 유지돼야 할 때 필요하다.

관계형 데이터베이스는 전형적으로 클라이언트와 데이터베이스 서버 사이의 TCP 연결을 기반으로 하고 어떤 특정 연결 내에서 BEGIN TRANSACTION 문과 COMMIT 문 사이의 모든 것은 같은 트랙잭션에 속하는 것으로 여겨진다.

비관계형 데이터베이스는 연산을 묶는 방법이 없는 경우가 많다. 다중 put(multi-put) 연산을 제공할 수도 있지만 반드시 트랜잭션 시맨틱을 뜻하지 않는다. 어떤 키에 대한 연산은 성공하고 어떤 키에 대한 연산은 실패해서 데이터베이스가 부분적으로 갱신될 수 있다.

**단일 객체 쓰기**

원자성과 격리성은 단일 객체를 변경하는 경우에도 적용된다. 예를 들어 20KB의 JSON 문서를 데이터베이스에 쓸 때 첫 10KB를 보낸 후 네트워크 연결이 끊길 때 어떻게 처리할지가 있다.

저장소 엔진들은 거의 보편적으로 한 노드에 존재하는 키-값 쌍 같은 단일 객체 수준에서 원자성과 격리성을 제공하는 것을 목표로 한다. 원자성은 장애 복구(crash recovery)용 로그로 격리성은 각 객체 잠금을 사용해 구현할 수 있다.

어떤 데이터베이스는 증가 연산처럼 더 복잡한 원자적 연산을 제공하기도 한다. (e.g. 원자적 증가(atomic imcrement))

이러한 단일 객체 연산은 여러 클라이언트에서 동시에 같은 객체에 쓰려고 할 때 갱신 손실(lost update)을 방지하므로 유용하다.

트랜잭션은 보통 다중 객체에 대한 다중 연산을 하나의 실행 단위로 묶는 매커니즘으로 이해된다.

**다중 객체 트랜잭션의 필요성**

많은 경우에 여러 개의 다른 객체에 실행되는 쓰기 작업은 코디네이션돼야 한다.

- 외래키가 참조하는 객체가 유효한 상태로 유지되도록 보장해야 한다.
- 비정규화된 정보를 갱신할 때 한번에 여러 문서를 갱신해야 한다.
- 보조 색인은 값을 변경할 때마다 색인도 갱신되어야 한다.

트랜잭션이 없더라도 이런 애플리케이션들을 구현할 수는 있지만 원자성이 없으면 오류 처리가 훨씬 복잡해지고 격리성이 없으면 동시성 문제가 생길 수 있다.

**오류와 어보트 처리**

ACID 데이터베이스는 오류가 생기면 어보트되고 안전하게 재시도할 수 있다는 철학을 바탕으로 한다.

모든 시스템이 ACID 데이터베이스 철학을 따르지는 않는다. 예를 들어 리더 없는 복제를 사용하는 데이터스토어는 최선을 다하는(best effort) 원칙을 기반으로 훨씬 더 많은 일을 하고 이미 한 일은 책임지지 않는다.

어보트된 트랜잭션을 재시도하는 것은 간단하고 효과적인 오류 처리 메커니즘이지만 완벽하지는 않다. 예를 들어 애플리케이션에서 추가적인 중복제거 매커니즘가 필요할 수 있다.
### 완화된 격리 수준(**Weak Isolation Levels)**

동시성 문제(경쟁조건)는 트랜잭션이 다른 트랙잭션에서 동시에 변경한 데이터를 읽거나 두 트랜잭션이 동시에 같은 데이터를 변경하려고 할 때만 나타난다.

동시성 버그는 타이밍에 운이 없을 때만 촉발되기 떄문에 테스트로 발견하기 어렵고 일반적으로 재현 및 추론이 어렵다. 이런 어려움 때문에 데이터베이스는 오랫동안 트랜잭션 격리를 제공함으로써 애플리케이션 개발자들에게 동시성 문제를 감추려고 했다.

직렬성 격리는 데이터베이스가 여러 트랜잭션들이 직렬적으로 실행되는 것(즉 동시성 없이 한 번에 트랜잭션 하나만 실행)과 동일한 결과가 나오도록 보장한다는 것을 의미한다. 하지만 직렬성 격리는 성능 비용이 있고 많은 데이터베이스들은 어떤 동시성 이슈로부터는 보호해주지만 모든 이슈로부터 보호해주지는 않는, 완화된 격리 수준을 사용한다.

**더티읽기(dirty reads)**

다른 트랜잭션에서 커밋되지 않은 데이터를 볼 수 있으면 이를 더티 읽기라고 부른다.

더티 읽기를 사용하면 다음 문제가 있다.

- 부분적으로 갱신된 상태를 보는 것은 사용자에게 혼란을 주며..다른 트랜잭션들이 잘못된 결정을 하는 원인이 될 수도 있다.
- 트랜잭션이 어보트되면 그때까지 쓴 내용은 모두 롤백돼야 한다.

**더티쓰기(dirty writes)**

여러 트랜잭션이 데이터베이스에 있는 동일한 객체를 동시에 갱신하려할때 아직 커밋되지 않은 트랜잭션에서 쓴것을 나중에 실행된 쓰기 작업이 덮어쓰는 것을 더티 쓰기라고 부른다.

두개 이상의 트랜잭션이 데이터베이스에 있는 동일한 객체를 동시에 갱신하려할때 아직 커밋되지 않은 트랜잭션의 쓴 값을 다른 트랜잭션이 덮어 쓰는 것을 말한다. 보통 먼저 쓴 트랜잭션이 커밋되거나 어보트될 때 까지 두 번째 쓰기를 지연시키는 방법을 사용한다.

더티쓰기를 방지함으로써 앨리스와 밥이 동시에 같은 차를 살 때 목록에 구매자 값은 밥으로 송장은 앨리스에게 전송되는 것과 같은 동시성 문제를 회피할 수 있다.

![스크린샷 2022-10-20 오전 12 03 15](https://user-images.githubusercontent.com/19777164/196729132-018d28af-5e9b-4057-8765-0debbbf14a03.png)

두 번의 카운터 증가 사이에 발생하는 경쟁조건은 막지 못한다. 이 경우는 첫번째 트랜잭션이 커밋된 후 두번째 쓰기가 일어났으므로 더티쓰기가 아니다.

![스크린샷 2022-10-20 오전 12 04 59](https://user-images.githubusercontent.com/19777164/196729544-a3df7c22-6898-41e1-af9b-a201c78c1492.png)

**커밋 후 읽기(Read Committed)**

커밋 후 읽기는 오라클 11g, 포스트그레스큐엘, SQL 서버 2012, 멤SQL(MemSQL) 등 많은 데이터베이스에서 기본 설정이다.

- 데이터베이스에서 읽을 때 커밋된 데이터만 보게 된다(더티 읽기가 없음).
- 데이터베이스에 쓸 때 커밋된 데이터만 덮어쓰게 된다(더티 쓰기가 없음).

가장 흔한 방법으로 데이터베이스는 로우 수준 잠금을 사용해 더티 쓰기를 방지한다. 트랜잭션에서 특정 객체(로우나 문서)를 변경하고 싶다면 잠금을 획득해야 하고 트랜잭션이 커밋되거나 어보트될 때까지 잠금을 보유하고 있어야 한다. 오직 한 트랜잭션만 어떤 주어진 객체에 대한 잠금을 보유할 수 있다.

쓰여진 모든 객체에 대해 데이터베이스는 과거에 커밋된 값과 현재 쓰기 잠금을 갖고 있는 트랜잭션에서 쓴 새로운 값을 기억하여 더티 읽기를 방지한다. 해당 트랜잭션이 실행 중인 동안 그 객체를 읽는 다른 트랜잭션들은 과거의 값을, 새 값이 커밋된 후에는 새 값을 읽을 수 있게 된다.  읽기 잠금은 읽기만 실행하는 여러 트랜잭션들이 오랫동안 실행되는 쓰기 트랜잭션 하나가 완료될 떄까지 기다려야 하기 떄문에 운용성이 나빠 현실에서는 잘 동작하지 않는다.