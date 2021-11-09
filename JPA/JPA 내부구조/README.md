# JPA 내부 구조

### JPA에서 가장 중요한 2가지

- 객체와 관계형 데이터베이스 매핑하기
- 영속성 컨텍스트

---

## 영속성 컨텍스트

- JPA를 이애하는데 가장 중요한 영어
- 엔티티를 영구 저장하는 환경 이라는 뜻
- EntityManager.persist(entity);

### 엔티티 매니저? 영속성 컨텍스트?

- 영속성 컨텍스트는 논리적인 개념
- 눈에 보이지 않는다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근

### 엔티티의 생명주기

- 비영속(new/transient)
    - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태 - new 로 첫 생성
- 영속 (managed)
    - persist한 상황
- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제 (removed)
    - 삭제된 상태
    

### 비영속

객체를 생성만 하고 있는 상태.

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled.png)

### 영속 상태

entityManager를 통해서 persist한 상태

아직 db에 저장된 상태는 아님 , 트랜잭션 커밋을 해야 db에 쿼리를 날림

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%201.png)

### 준영속, 삭제

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%202.png)

---

## 영속성 컨텍스트의 이점

- **1차 캐시**
- **동일성(identity)보장**
- **트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)**
- **변경감지(Dirty Checking)**
- **지연 로딩(Lazy Loading)**

### 1차 캐시

JPA에서 조회가 일어날시 우선적으로 1차 캐시에서 조회합니다.

그 후 1차캐시에 존재하면 그안에서 데이터를 반환합니다.

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%203.png)

**1차 캐시에 없을경우.**

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%204.png)

하지만 JPA는 트랜잭션단위로 동작하기 떄문에 트랜잭션이 끝나면 엔티티 매니저가 반환되며

크게 도움은 없다고 합니다.

### 영속 엔티티의 동일성 보장

1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭
션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

```java
Member member1 = entityManager.find(Member.class, "member1"):
Member member2 = entityManager.find(Member.class, "member1"):

System.out.println ( member1 == member2 ); // true 동일하다.

```

### 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)

```java

EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.

transaction.begin(); // [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

persist 를 하는순간 JPA가 이를 분석해서 쿼리를 생성하고 쓰기지연 SQL 저장소에 저장시켜둔다.

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%205.png)

그 후 커밋을 하는 순간 생성된 쿼리를 DB로 전송한다 이를 flush 라고 부른다.

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%206.png)

### 엔티티 수정 **변경감지(Dirty Checking)**

동작원리

최초로 조회 했을때 (1차캐시) 그대의 값을 스냅샷에 저장해둔다.

트랜잭션이 끝날때 엔티티와 스냅샷을 비교하고, 변경이 이뤄졌다면

Update SQL 을 생성해서 DB로 보낸다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.

transaction.begin(); // [트랜잭션] 시작

//영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

//영속 엔티티 데이터 수정
memberA.setUsername("h1");
memberA.setAge(10);

//em.update(member) 이런 코드가 없어도 된다.

transaction.commit(); // 트랜잭셩 커밋
```

![Untitled](JPA%20%E1%84%82%E1%85%A2%E1%84%87%E1%85%AE%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%209885ed46bd9c42f5ac62a255581dd13d/Untitled%207.png)

---

## 플러시

영속성 컨텍스트의 변경내용을 데이터베이스에 반영

### 플러시 발생

- 변경 감지
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송 ( 등록, 수정, 삭제 쿼리 )

---

### 영속성 컨텍스트를 플러시하는 방법

- em.flush() - 직접 호출
- 트랜잭션 커밋 - 플러시 자동 호출
- JPQL 쿼리 실행 - 플러시 자동 호출

플러시를 한다고 1차 캐시가 사라지는건 아니다.

**주의! 플러시는!**

- 영속성 컨텍스트를 비우지 않음
- 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화
- 트랜잭션이라는 작업 단위가 중요 → 커밋직전에만 동기화 하면 됨.

---

## 준영속 상태

- 영속 → 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
- 영속성 컨텍스트가 제공하는 기능을 사용 못함

### 준영속 상태로 만드는 방법

- entityManager.detach(entity)
    - 특정 엔티티만 준영속 상태로 전환
- entityManager.clear()
    - 영속성 컨텍스트를 완전히 초기화
- entityManager.close()
    - 영속성 컨텍스트를 종료

```java
// detach 예시
Member member = em.find(Member.class, 1L);
member.setName("AAAA");

entityManager.detach(member); // 준영속 상태로 진입하여 update가 안됨

```