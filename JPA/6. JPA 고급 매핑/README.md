# JPA κ³ κΈ λ§€ν

## **π λͺ©μ°¨**

- μμκ΄κ³ λ§€ν
- @MappedSuperclass

---

## μμκ΄κ³ λ§€ν

- κ΄κ³ν λ°μ΄ν°λ² μ΄μ€λ μμ κ΄κ³X
- μνΌ νμ μλΈνμ κ΄κ³λΌλ λͺ¨λΈλ§ κΈ°λ²μ΄ κ°μ²΄ μμκ³Ό μ μ¬
- μμκ΄κ³ λ§€ν: κ°μ²΄μ μμκ³Ό κ΅¬μ‘°μ DBμ μνΌνμ μλΈνμ κ΄κ³λ₯Ό λ§€ν
- μνΌ νμ μλΈνμ λΌλ¦¬ λͺ¨λΈμ μ€μ  λ¬Όλ¦¬ λͺ¨λΈλ‘ κ΅¬ννλ λ°©λ²
    - κ°κ° νμ΄λΈλ‘ λ³ν β μ‘°μΈ μ λ΅
    - ν΅ν© νμ΄λΈλ‘ λ³ν β λ¨μΌ νμ΄λΈ μ λ΅
    - μλΈνμ νμ΄λΈλ‘ λ³ν β κ΅¬ν ν΄λμ€λ§λ€ νμ΄λΈ μ λ΅

### μ£Όμ μ΄λΈνμ΄μ

- @Inheritance(strategy = inheritanceType.XXX)
    - JOINED: μ‘°μΈ μ λ΅
    - SINGLE_TABLE: λ¨μΌ νμ΄λΈ μ λ΅
    - TABLE_PER_CLASS: κ΅¬ν ν΄λμ€λ§λ€ νμ΄λΈ μ λ΅
- DiscriminatorColumn(name= "DTYPE")
    - μμ νμ΄λΈμ μ»¬λΌμ΄ μκΈ°κ³  μ΄λ€ μν°ν°κ° λ€μ΄μλμ§ μ μ μλ€.
- DiscriminatorValue("XXX")
    - νμ μν°ν°μμ λ³μΉ­μΌλ‘ DTYPEμ λ€μ΄κ°κ±Έ μ ν΄ μ€ μ μλ€.

---

### λ¨μΌ νμ΄λΈ μ λ΅

κΈ°λ³Έ μ λ΅μ΄ SINGLE_TABLE  μλμ μ½λ μ€νμ Item ννμ΄λΈμ λͺ¨λ  μ»¬λΌμ΄ λ€μ΄κ°

- μ₯μ 
    - μ‘°μΈμ΄ νμ μμΌλ―λ‘ μΌλ°μ μΌλ‘ μ‘°ν μ±λ₯μ΄ λΉ λ¦
    - μ‘°ν μΏΌλ¦¬κ° λ¨μν¨
- λ¨μ 
    - μμ μν°ν°κ° λ§€νν μ»¬λΌμ λͺ¨λ null νμ©
    - λ¨μΌ νμ΄λΈμ λͺ¨λ  κ²μ μ μ₯νλ―λ‘ νμ΄λΈμ΄ μ»€μ§ μ μλ€. μν©μ λ°λΌμ μ‘°ν μ±λ₯μ΄ μ€νλ € λλ €μ§ μ μλ€.

```java
@Entity
@Inheritance(strategy = inheritanceType.SINGLE_TABLE)
abstract class Item {
	@Id
	private Long id;
	private String name;
	private int price;
}

@Entity
class Album extends Item {
	private String artist
}

@Entity
class Movie extends Item {
	private String director;
	private String actor;
}

@Entity
class Book extends Item {
	private String author;
	private String isbn;
}
```

![Untitled](image/Untitled.png)

---

### μ‘°μΈ νμ΄λΈ μ λ΅

JOINED νλ©΄ μλ ERDμ²λΌ νμ΄λΈμ΄ μμ±λ¨.

- μ₯μ 
    - νμ΄λΈ μ κ·ν
    - μΈλ ν€ μ°Έμ‘° λ¬΄κ²°μ± μ μ½μ‘°κ±΄ νμ©κ°λ₯
    - μ μ₯κ³΅κ° ν¨μ¨ν
- λ¨μ 
    - μ‘°νμ μ‘°μΈμ λ§μ΄ μ¬μ©, μ±λ₯ μ ν
    - μ‘°ν μΏΌλ¦¬κ° λ³΅μ‘ν¨.
    - λ°μ΄ν° μ μ₯μ INSERT SQL 2λ² νΈμΆ

```java
@Entity
@Inheritance(strategy = inheritanceType.JOINED)
abstract class Item {
	@Id
	private Long id;
	private String name;
	private int price;
}

@Entity
class Album extends Item {
	private String artist
}

@Entity
class Movie extends Item {
	private String director;
	private String actor;
}

@Entity
class Book extends Item {
	private String author;
	private String isbn;
}
```

![Untitled](image/Untitled%201.png)

```

MOVE λ‘ save μ

insert item values ? ? ?
insert move values ? ? ? μ΄λ κ² λ€μ΄κ°λ€.

```

---

### κ΅¬ν ν΄λμ€λ§λ€ νμ΄λΈ μ λ΅

- μ΄ μ λ΅μ λ°μ΄ν°λ² μ΄μ€ μ€κ³μμ ORM μ λ¬Έκ° λ λ€ μΆμ² X
- μ₯μ 
    - μλΈ νμμ λͺννκ² κ΅¬λΆν΄μ μ²λ¦¬ ν  λ ν¨κ³Όμ 
    - not null μ μ½μ‘°κ±΄ μ¬μ© κ°λ₯
- λ¨μ 
    - μ¬λ¬ μμ νμ΄λΈμ ν¨κ» μ‘°νν  λ μ±λ₯μ΄ λλ¦Ό(UNION SQL νμ)
    - μμ νμ΄λΈμ ν΅ν©ν΄μ μΏΌλ¦¬νκΈ° μ΄λ €μ

```java
@Entity
@Inheritance(strategy = inheritanceType.TABLE_PER_CLASS)
abstract class Item {
	@Id
	private Long id;
	private String name;
	private int price;
}

@Entity
class Album extends Item {
	private String artist
}

@Entity
class Movie extends Item {
	private String director;
	private String actor;
}

@Entity
class Book extends Item {
	private String author;
	private String isbn;
}
```

![Untitled](image/Untitled%202.png)

---

## MappedSuperclass

- κ³΅ν΅ λ§€ν μ λ³΄ κ° νμν  λ μ¬μ©(id, name)

```java
@MpaaedSuperclass
class BaseEntity {
	LocalDateTime create;
	LocalDateTime update;
}

class Member extends BaseEntity {

}

```

![Untitled](image/Untitled%203.png)

- μμκ΄κ³ λ§€νX
- μν°ν°X, νμ΄λΈκ³Ό λ§€νX
- λΆλͺ¨ ν΄λμ€λ₯Ό μμλ°λ μμ ν΄λμ€μ λ§€ν μ λ³΄λ§ μ κ³΅
- μ‘°ν, κ²μλΆκ°(em.find(BaseEntity)λΆκ°
- μ§μ  μμ±ν΄μ μ¬μ©ν  μΌμ΄ μμΌλ―λ‘ μΆμ ν΄λμ€ κΆμ₯
- νμ΄λΈκ³Ό κ΄κ³ μκ³ , λ¨μν μν°ν°κ° κ³΅ν΅μΌλ‘ μ¬μ©νλ λ§€ν μ λ³΄λ₯Ό λͺ¨μΌλ μ­ν 
- μ£Όλ‘ λ±λ‘μΌ, μμ μΌ, λ±λ‘μ, μμ μ κ°μ μ μ²΄ μν°ν°μμ κ³΅ν΅μΌλ‘ μ μ©νλ μ λ³΄λ₯Ό λͺ¨μ λ μ¬μ©
- μ°Έκ³ : @Entity ν΄λμ€λ μν°ν°λ @MappedSuperclassλ‘ μ§μ ν ν΄λμ€λ§ μμ κ°λ₯