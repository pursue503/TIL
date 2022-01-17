# 3장 사용자 및 권한

### 3.1 사용자 식별

**Mysql 사용자 식별 특징**

- 아이디 말고 어느 아이피에서 접속했는지도 확인
- Mysql 8.0 부터는 권한을 묶어서 관리하는 ROLE 도입
- Mysql 에서 계정을 언급할 때는 아이디와 호스트를 함께 명시해야함

```sql

-- LocalHost 에서만 접속가능한 계정
`svc_id'@'127.0.0.1`

```

**Mysql 에서 동일한 아이디가 있을 시 사용자 인증을 위해 어떤 계정을 선택하는가?**

```sql
-- 192.168.0.10 IP에서만 접속가능
-- 해당 아이디 비밀번호는 123
`svc_id'@'192.168.0.10`

-- 모든 아이피에서 접근가능
-- 해당 아이디 비밀번호는 abc
`scv_id'@'%`
```

192.168.0.10 아이피 에서 비밀번호 abc로 접속할경우

- Mysql 은 항상 작은 범위부터 선택한다.
- 해당 로그인은 실패한다.

---

### 3.2  사용자 계정 관리

**시스템 계정과 일반 계정**

Mysql 8.0 부터 시스템계정과 일반계정으로 구분

- 시스템 계정은 데이터베이스 서버 관리자용 계정
- 일반계정은 응용프로그램이나 개발자를 위한 계정

**시스템계정** 

- 계정 관리(계정 생성 및 삭제, 계정 권한 부여 및 제거)
- 다른 세션(Connection) 또는 그 세션에서 실행중인 쿼리를 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

**지금 메모 부터 구분**

- 사용자 : Mysql 을 사용하는 주체 ( 유저 또는 프로그래머 )
- 계정: Mysql 서버에 로그인하기 위한 식별자

---

### 3.2.2 계정 생성

Mysql 5.7 까지는 GRANT 명령으로 권한부여와 동시에 계정생성이 가능했었다.

Mysql 8.0 부터는 계정생성은 Create User 권한은 GRANT 명령으로 구분해서 실행하도록 변경되었다.

**계정 생성 옵션**

- 계정의 인증 방식과 비밀번호
- 비밀번화 관련 옵션(비밀번호 유효기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간)
- 기본 역할
- SSL 옵션
- 계정 잠금 여부

일반적으로 많이 사용되는 CREATE USER 옵션

```sql
CREATE USER `user'@'%` 
IDENTIFIED WITH 'mysql_native_password' BY 'password'
REQUIRE NONE 
PASSWORD EXPIRE INTERVAL 30 DAY
ACCOUNT UNLOCK
PASSWORD HISTORY DEFAUlT
PASSWORD REUSE INTERVAL DEFAULT
PASSWORD REQUIRE CURRENT DEFAULT;
```

**IDENTIFIED WITH**

사용자 인증 방식과 비밀번호 설정

IDENTIFIED WITH 뒤에는 인증방식을 명시해야함

- Native Pluggable Authentication:
    
    Mysql 5.7 버전까지 기본으로 사용되던 방식 단순히 비밀번호에 대한 해시(SHA-1 알고리즘) 값을 저장해두고, 클라이언트가 보낸 값과 해시 값이 일치하는지 비교
    
- Caching SHA-2 Pluggable Authentication:
    
    Mysql 5.6버전에 도입 8.0 버전부터 보안되서 SHA-2(256비트) 알고리즘 사용 
    
- PAM Pluggable Authentication:
    
    유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할 수 있게 해주는 인증방식 Mysql 엔터프라이즈 에디션에서만 가능
    
- LDAP Pluggable Authentication:
    
    LDAP 을 이용한 외부 인증을 사용할 수 있게 하는 방식 엔터프라이즈 에디션에서만 가능
    

**REQUIRE**

- DB 서버 접속 시 SSL/TLS 채널 사용 여부를 설정
- DEFAULT 비 암호화 채널 연결

**PASSWORD EXPIRE**

비밀 번호 유효기간 설정 옵션

명시하지 않으면 default_password_lifeitem 시스템 변수에 저장된 기간으로 유효기간 설정

- PASSWORD EXPIRE: 계정 생성과 동시에 비밀번호의 만료 처리
- PASSWORD EXPIRE NAVER: 계정 비밀번호의 만료 기간 없음
- PASSWORD EXPIRE DEFAULT: default_password_lifetime 시스템 변수에 저장된 기간으로 설정
- PASSWORD EXPIRE INTERVAL n DAY: 유효기간을 n 일자로 설정

**PASSWORD HISTORY**

한 번 사용했던 비밀번호를 재사용 못하게 하는 옵션

- PASSWORD HISTORY DEFAULT:
    
    password_history 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장 
    
    저장된 이력에 남아있는 비밀번호는 재사용 불가
    
- PASSWORD HISTORY n: 이력을 최근 n개까지만 저장

**PASSWORD REUSE INTERVAL**

한 번 사용한 비번의 재사용 금지 기간 설정

- PASSWORD REUSE INTERVAL DEFAULT: password_reuse_interval 변수 기간
- PASSWORD REUSE INTERVAL n DAY: n 일 이후에 재사용가능

### 비밀번호 관리

validate_password 컴포넌트를 이용하기

설치 방법

```sql
-- mysql server connect

mysql> INSTALL COMPONENT 'file://component_validate_password';
Query OK, 0 rows affected (0.04 sec)

mysql> select * from mysql.component;
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password |
+--------------+--------------------+------------------------------------+
1 row in set (0.00 sec)

mysql> show global variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.03 sec)

mysql>
```

- LOW : 비밀번호의 길이만 검증
- MIDIUM: 비밀번호 길이, 숫자 대소문자 ,특수문자 배합 검증
- STRONG: MIDIUM + 금칙어 포함됐는지 검증

---

### 3.2.2 이중 비밀번호 사용하기

8.0 버전부터 2개의 비밀번호 사용가능

최근 비밀번호는 Primary 이전 비밀번호는 Secondary

**설정법**

RETAIN CURRENT PASSWORD 옵션을 사용한다.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password'

ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
```

**Secondary 비밀번호 삭제**

```sql
ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
```

---

### 권한

```sql
-- 글로벌 권한
GRANT SUPER ON *.* TO 'user'@'localhost';

-- DB 권한
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON employees.* TO 'user'@'localhost';

```

### 역할

Mysql 8.0부터 추가된 역할 기능

각 역할을 생성하고 유저를 생성하면서 역할을 부여해봅니다.

```sql
-- role_emp_read, role_emp_wirte 역할 생성

CREATE ROLE role_emp_read, role_emp_write;

-- 역할에 권한 부여
GRANT SELECT ON employees.* TO role_emp_read;

GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;

-- 유저 생성
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'abc1234!A';
CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'abc1234!A';

-- 권한 부여
GRANT role_emp_read TO reader@'127.0.0.1';
GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
```

해당 아이디로 가도 권한이 없음

매번 권한을 들어가서 SET 으로 설정하거나 아래의 글로벌 변수를 조작하여 사용 가능

```sql
SHOW VARIABLES LIKE '%roles_on%';

SET GLOBAL activate_all_roles_on_login=ON;
```

**참고**

**계정 == 역할 이다 mysql 서버는 이를 구분하지 않으므로**

**꼭 앞에 식벽가능한 프리픽스를 주자**

---