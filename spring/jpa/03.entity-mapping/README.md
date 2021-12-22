# 엔티티 매핑
- [엔티티 매핑](#------)
  * [객체와 테이블 매핑](#----------)
    + [@Entity](#-entity)
    + [@Table](#-table)
  * [데이터베이스 스키마 자동 생성](#----------------)
    + [hibernate.hbm2ddl.auto](#hibernatehbm2ddlauto)
    + [주의 사항](#-----)
  * [필드와 컬럼 매핑](#---------)
  * [기본 키 매핑](#-------)
  

## 객체와 테이블 매핑
### @Entity
- `@Entity`가 붙은 클래스는 JPA가 관리하는 엔티티라고 한다
- JPA를 사용해서 테이블과 매핑할 클래스는 해당 어노테이션을 붙인다.
- **기본 생성자**가 필수적으로 필요하다.
- 속성
  - name 
    - `@Entity(name="{entityName}")` 형태로 사용한다.
    - JPA에서 사용할 엔티티 이름을 지정한다. (show_sql 옵션 확인 시 테이블 alias 형태로 사용)
    - 기본값은 클래스의 이름을 그대로 사용한다.
    - 가능한 기본값을 그대로 사용하는것을 권장.
### @Table
- `@Table`은 엔티티와 매핑할 테이블을 지정한다.
- 속성
  - name
    - 매핑할 테이블 이름 (엔티티 이름을 기본값으로 사용)
  - catalog
    - 데이터베이스 catalog 매핑
  - schema
    - 데이터베이스 schema 매핑
  - uniqueConstraints(DDL)
    - DDL 생성 시에 유니크 제약조건 생성
    
## 데이터베이스 스키마 자동 생성
- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 → 객체중심
- 데이터베이스 방언(Dialect)을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
    - 오라클 varchar2 mysql varchar ... 등 데이터베이스에 맞는 타입으로 생성
- 이렇게 **생성된 DDL은 개발 장비에서만 사용**
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용
### hibernate.hbm2ddl.auto (자동 생성 옵션명)
- create
    - 기존테이블 삭제 후 다시 생성(DROP + CREATE)
- create-drop
    - create와 같으나 애플리케이션 종료시점에 테이블 DROP
    - 주로 테스트케이스를 작성하는 환경에서 사용
- update
    - 변경분만 반영(운영 DB에는 사용하면 안됨)
- validate
    - 엔티티와 테이블이 정상 매핑되었는지만 확인
- none
    - 사용하지 않음
### 주의 사항
- **운영 장비에는 절대 create, create-drop, update를 사용하면 안된다.**
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none
- 데이터베이스에 직접적인 작업을 하는것을 애플리케이션에 일임할 경우
  데이터베이스에 문제 또는 서비스 장애로 이어지는 문제를 예상할 수 없기 때문에 사용하지 않는다.
  alter table을 한다고 가정했을시 운영에 존재하는 데이터에 따라 예상시간을 가늠할 수 없고 그 시간동안 해당 테이블에 접근이 불가 할 수 있고 그로 인해 서비스에 장애로 이어질 수 있기 때문이다.
- 대부분의 규모가 있는 회사 또는 서비스라면 애플리케이션 계정에서 DDL이 불가 할 것이다.
  계정에 권한관리를 통해 접근제어를 하는게 맞다.

## 필드와 컬럼 매핑
- `@Column`
  - 컬럼 매핑
  - `@Column(name = "{컬럼명}")` 형태로 사용한다.
  - 속성
    - name
      - 매핑할 테이블의 컬럼 이름 (기본값: 객체의 필드명)
    - insertable, updatable
      - 등록, 변경 가능 여부 (기본값: True)
      - `@Column(insertable = true, updatable = false)`
    - nullable(DDL)
      - `null` 값의 허용 여부를 설정한다. `false`로 설정하면 DDL 생성 시에 `not null` 제약조건이 붙는다.
      - `@Column(nullable = false)`
    - unique(DDL)
      - `@Table`의 `uniqueConstraints`와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸때 사용.
      - 컬럼에 사용 시 유니크키의 이름을 설정하기 힘들기 때문에 잘 사용하지 않는다.
    - columnDefinition(DDL)
      - 컬럼정보를 직접 줄 수 있다.
      - `@Column(columnDefinition = "varchar(1000) default 'EMPTY'")`
    - length(DDL)
      - 문자 길이 제약조건, String 타입에만 사용한다.
    - precision(DDL), scale(DDL)
      - `BigDecimal` 타입에서 사용한다(`BigInteger`도 사용할 수 있다). 
      - `precision`은 소수점을 포함한 전체 자 릿수를, `scale`은 소수의 자릿수 다. 참고로 `double`, `float` 타입에는 적용되지 않는다. 아주 큰 숫자나 정밀한 소수를 다루어야 할 때만 사용한다.
      - 기본값: precision = 19, scale = 2
- `@Temporal`
  - 날짜 타입 매핑
  - Date, Time, Timestamp 3가지 타입이 존재한다.
  - `@Temporal(TemporalType.TIMESTAMP)` 형태로 사용한다.
  - `LocalDate`, `LocalDateTime`을 사용할 때는 생략 가능하다.
  - 속성
    - value
      - `TemporalType.DATE`
        - 날짜, 데이터베이스 Date 타입과 매핑 (yyyy-MM-dd)
      - `TemporalType.TIME`
        - 시간, 데이터베이스 Time타입과 매핑 (HH:mi:ss)
      - `TemporalType.TIMESTAMP`
        - timestamp 타입과 매핑 (yyyy-MM-dd HH:mi:ss)
- `@Enumerated`
  - enum 타입 매핑
  - `@Enumerated(EnumType.STRING)` 형태로 사용한다.
  - 속성
    - value
      - EnumType.ORDINAL
        - enum의 순서를 데이터베이스에 저장한다. (0,1,2 형태로 순서를 저장한다.)
      - EnumType.STRING
        - enum의 이름을 데이터베이스에 저장한다.
      - **ORDINAL 사용하지 않는것을 권장!!! 타입이 추가되서 순서가 꼬일경우... 망합니다. **
- `@Lob`
  - `BLOB`, `CLOB` 타입으로 매핑된다. 
  - 문자 타입일 경우 `CLOB` 타입으로 매핑, 나머지는 `BLOB` 매핑
- `@Transient`
  - 해당 필드를 컬럼에 매핑하지 않음(엔티티 내에서 DB에 컬럼과 매핑되지 않은 필드로 사용한다.)

## 기본 키 매핑