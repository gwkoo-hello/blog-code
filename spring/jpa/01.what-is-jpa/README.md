# JPA 란 무엇이고 왜 사용해야 하는가

## JPA?

- Java Persistence API (인터페이스의 모음이다.)
- Java 진영의 **ORM** 기술 표준
- JPA 2.1 표준 명세를 구현한 3가지 구현체 - Hibernate, EclipseLink, DataNucleus

## JPA 버전

- JPA 1.0(JSR 220) 2006년: 초기버전. 복합 키와 연관관계 기능이 부족
- JPA 2.0(JSR 317) 2009년: 대부분의 ORM 기능을 포함, JPA Criteria 추가
- JPA 2.1(JSR 338) 2013년: 스토어드 프로시저 접근, 컨버터(Converter), 엔티티 그래프 기능이 추가

## ORM 이란 ?

- Object-relational mapping(객체 관계 매핑)
- 객체는 객체대로 설계, 관계형 데이터베이스는 관계형 데이터베이스대로 설계를 해서 ORM 프레임워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재 (Typescript, Python...등도 존재함)

## JPA는 애플리케이션과 JDBC 사이에서 동작

패러다임 불일치를 해결하기 위해 Java Application 영역에서 객체에 데이터를 설정 후 JPA를 통해 DB 접근을 한다. 중간에 JPA 계층이 별도로 존재

## JPA를 왜 사용해야 하는가 ?

- SQL 중심적인 개발에서 객체 중심으로 개발한다.
- 생산성과 유지보수 측면에서 앞선다. (기본적인 SQL을 생성할 필요가 없고, 테이블에 컬럼 추가시 모든 SQL을 찾아서 추가해줘야 하는 불편함)
- 패러다임의 불일치를 해결한다. (객체의 상속, 객체 그래프 탐색(member.getTeam 형태의 멤버안에 팀을 조회 가능)
- 성능이 좋다
    - **1차 캐시와 동일성 보장**
        - 같은 트렌잭션 안에서는 같은 엔티티를 반환(조회 성능 향상)
        - DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장
    - **트랜젝션을 지원하는 쓰기 지연(트랜젝션 커밋전까지는 DB에 접근하지 않는다.)**
        - 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
        - JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송
        - update, delete로 인한 **로우락** 시간 최소화 (update, delete등의 쿼리가 정상 수행될 때까지 기다리는 시간을 커밋시 발생시키기 때문에 비지니스 로직이 바로 수행가능하다.)

        ```java
        transaction.begin(); // 트랜젝션 시작
        
        changeMemeber(memberA);
        deleteMember(memberB);
        비지니스_로직_수행(); // 비지니스 로직 수행동안 DB로우 락이 걸리지 않는다.
        
        // 커밋하는 순간 데이터베이스에 update, delete SQL을 보낸다.
        transaction.commit(); // 트랜잭션 커밋
        ```

    - **지연 로딩(Lazy Loading)**
- 데이터 접근 추상화와 벤더의 독립성(Dialect를 통해 DB별 방언을 충족한다)
- 동일한 트랜잭션에서 조회한 같은키의 엔티티는 객체비교 시 같음을 보장한다.

## JPA 구동 방식

1. `Persistence` 에서 설정 정보를 조회 한다. (`persistence.xml`)
2. `EntityManagerFactory`를 생성 한다. (애플리케이션 전체에서 한개만 생성해서 공유)
3. 팩토리를 통해 `EntityManager`들을 생성해서 호출한다.