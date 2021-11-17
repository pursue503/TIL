# JPA 엔티티 매핑

## 👋 목차

- 객체와 테이블 매핑
- 데이터베이스 스키마 자동생성
- 필드와 컬럼 매핑
- 기본 키 매핑

---

## 엔티티 매핑

### 엔티티 매핑 소개

- 객체와 테이블 매핑: @Entity, @Table
- 필드와 컬럼 매핑: @Column
- 기본 키 매핑: @Id
- 연관관계 매핑: @ManyToOne, @JoinColumn

### 객체와 테이블 매핑

### @Entity

- @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수

**주의 사항**

- 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
- final 클래스, enum, interface, inner 클래스 사용X
- 저장할 필드에 final 사용 X

```java
@Entity
public class Member {
	@Id
	private Long id;
	private String name;
	
	public Member() {

	}

	//getter setter...

}
```

### 필드와 컬럼 매핑

@Column: 컬럼 매핑

- name: 필드와 매핑할 테이블의 컬럼이름 ( default: 객체의 필드 이름 )
- insertable, updatable: 등록, 변경 가능 여부 ( default: true )
- nullable(DDL): null값의 허용 여부를 설정한다 false 면 not null 제약 조건이 붙는다.
- unique(DDL):  유니크 제약 조건을 걸때 사용한다.
- columnDefinition: 데이터 베이스 컬럼 정보를 직접 줄 수 있다.
- length(DDL): 문자 길이 제약조건, String 타입에만 사용한다 ( default: 255)
- precision, scale(DDL): BigDecimal 타입에서 사용한다

@Temporal: 날짜 타입 매핑

- 자바 8 이상부터는 년월는 LocalDate → DATE
- 자바 8 이상부터 년월일은 LocalDateTime  → TIMESTAMP
- 하이버네이트가 맞춰서 넣어준다.

@Enumerated: enum 타입 매핑

- **ORDINAL 사용 X**
- ORDINAL: enum 순서를 데이터베이스에 저장
- STRING: enum 이름을 데이터베이스에 저장

@Lob: BLOB, CLOB 매핑

@Transient 특정 필드를 컬럼에서 제거

- 필드 매핑X
- 데이터베이스에 저장X, 조회X
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

---

## 데이터 베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 → 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 생성된 **DDL은 개발 장비에서만 사용**
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

**DDL 자동 생성 옵션**

```
hibernate.hbm2ddl.auto
```

- create: 기존 테이블 삭제 후 다시 생성( DROP + CREATE )
- create-drop: Create 와 같으나 종료시점에 테이블 DROP
- update: 변경분만 반영( 운영 DB에는 사용하면 안됨 )
- validate: 엔티티와 테이블이 정상 매핑되었는지만 확인
- none: 사용하지 않음

### 주의 사항

- 운영 장비에는 절대 create, create-drop, update 사용하면 안된다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none