# 영속성 관리 (영속성 컨텍스트)
- [영속성 관리 (영속성 컨텍스트)](#-----------------)
- [영속성 컨텍스트](#--------)
    * [엔티티의 생명주기](#---------)
    * [영속성 컨텍스트의 이점](#------------)
        + [**1차 캐시**](#--1------)
        + [**트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)**](#--------------------transactional-write-behind---)
        + [**변경 감지 (Dirty Checking)**](#---------dirty-checking---)
        + [**지연로딩 (Lazy Loading)**](#--------lazy-loading---)
    * [플러시](#---)
        + [플러시 모드 옵션](#---------)
    * [준영속 상태](#------)
        + [준영속 상태로 만드는 방법](#--------------)

# 영속성 컨텍스트
- JPA를 이해하는데 가장 중요한 단어
- `Entity`를 영구 저장하는 환경 이라는 논리적인 개념이다.
- `EntityManager.persist(entity);`
    - `persist()` 호출 시 DB에 저장하는 것이 아니라 영속성 컨텍스트에 영속화 한다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근

## 엔티티의 생명주기
- **비영속 (new/transient)**
    - 영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태

        ```java
        // 객체를 생성한 상태(비영속)
        Member member = new Member();
        member.setId("member1");
        member.setName("hello koo");
        ```
- **영속 (managed)**
    - 영속성 컨텍스트에 **관리**되는 상태
        ```java
        // 객체를 생성한 상태(비영속)
        Member member = new Member();
        member.setId("member1");
        member.setName("hello koo");
        
        EntityManager em = emf.createEntityManager();
        em.getTransaction().begin();
        
        // 객체를 저장한 상태(영속) DB에 저장되지 않음
        em.persist(member);
        ```
- **준영속 (detached)**
    - 영속성 컨텍스트에 저장되었다가 **분리**된 상태
- **삭제 (removed)**
    - **삭제**된 상태
## 영속성 컨텍스트의 이점

### **1차 캐시**

- 1개의 트랜젝션 안에서만 쓰고 반환하기 때문에 성능에 굉장히 큰 영향을 주지는 않는다.
- 나는 mybatis랑 비교했을 시에는 좀 크게 다가왔다. (저장, 수정시)
- **동일성(identity) 보장**
    - 1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공 - 1차 캐시가 있기 때문에 동일성을 보장할 수 잇다.
        ```java
        Member a = em.find(Member.class, "member1");
        Member b = em.find(Member.class, "member1");
        System.out.println(a == b); //동일성 비교 true
        ```
### **트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)**

- `em.persist(memberA);` 수행 시
- 쓰기 지연 SQL 저장소라는 곳에 `INSERT SQL`을 생성해서 쌓아두며, 동시에 1차 캐시에 저장한다.
- `transaction.commit();` 수행 시 (JPA에서는 `flush`라고 부름)
    - DB에 SQL을 실행한다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();// 엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.

transaction.begin(); // [트랜잭션] 시작

em.persist(memberA);
em.persist(memberB); // 여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

// 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

- **Hibernate**에서는 `batch_size`라는 설정을 통해서 `sql`을 모아서 `write`할 수 있다.

### **변경 감지 (Dirty Checking)**

- `Entity`의 `Data`만 변경해도 `update`쿼리가 실행 되는걸 말함. (`em.persist();` 호출하지 않아도 됨.)
1. flush()가 수행되면 엔티티와 스냅샷을 비교한다. (1차 캐시에 존재)
2. 스냅샷을 비교해서 값이 다를경우 `UPDATE SQL` 을 **쓰기 지연 SQL 저장소에** **생성**한다.
3. **쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송** (등록, 수정, 삭제 쿼리)

### **지연로딩 (Lazy Loading)**

- 뒤에서 설명

## 플러시

- **영속성 컨텍스트의 변경 내용을 데이터베이스에 반영**
    - 트랜잭션의 작업을 데이터베이스와 동기화를 수행하는 작업
- 플러시 발생 시 수행
    1. 변경 감지
    2. 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
    3. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송
- 영속성 컨텍스트를 플러시 하는 방법
    - entityManager.flush() - 직접 호출
    - 트랜잭션 커밋 - 플러시 자동 호출
    - JPQL 쿼리 실행 - 플러시 자동 호출
        - JPQL은 쿼리 실행전에 flush를 호출한다 무적권
- `em.flush()`가 실행되어도 1차 캐시가 날라가지는 않는다.

### 플러시 모드 옵션

FlushModeType.AUTO (Default)

- 커밋이나 쿼리를 실행할 때 플러시 - 이거만 쓰세요.!!!

FlushModeType.COMMIT

- 커밋할 경우에만 플러시

## 준영속 상태

- 영속 → 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
- 영속성 컨텍스트가 제공하는 기능을 사용 못함(1차캐시, 더티체킹)

### 준영속 상태로 만드는 방법

- `em.detach(entity);`
    - 특정 엔티티만 준영속 상태로 전환
- `em.clear();`
    - 영속성 컨텍스트를 완전히 초기화
- `em.close();`
    - 영속성 컨텍스트를 종료