# 2장 설치와 설정

### Mac (M1) docker Mysql 설치 방법

docker desktop 가 설치되었다는 기준하의 작성되었습니다.

[https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql)

도커 허브의 mysql을 가보면 os가 linux/amd64 로 지정되어있습니다

m1 버전은 플랫폼을 명시해주지 않으면 arm 버전으로 찾는거로 알고있습니다 ( 이 부분은 확실하지 않습니다. )

아래 명령어를 터미널을 열어 작성해줍시다.

```bash

# 플랫폼 명시해주기
$ docker pull --platform linux/amd64 mysql

# 다운 받은 이미지 실행하기
$ docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=<password> -d -p 3306:3306 mysql:latest
```

---

### 2.4.2 MySQL 시스템 변수의 특징

MySql 서버는 기동하면서 설정 파일의 내용을 읽어 메모리나 작동 방식을 초기화하고 

접속된 사용자를 제어하기 위해 이러한 값을 별도로 저장해둔다.

MySql 서버에서는 이렇게 저장된 값을 시스템 변수 ( System Variables ) 라고 한다.

각 변수는 아래의 쿼리로 조회가 가능하다 

```sql

SHOW GLOBAL VARIABLES;
SHOW VARIABLES;

```

해당 쿼리 조회로 나오는 설정들의 스코프들은 아래 사이트를 통해 더 정확하게 적용 범위를 알 수 있다.

[https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)

시스템 변수가 가지는 5가지 속성의 의미는 다음과 같다.

- Cmd-Line:
    
    MySql 서버의 명령행 인자로 설정될 수 있는지 여부를 나타낸다. 즉 이값이 Yes 이면 명령행 인자로 이 시스템 변수의 값을 변경하는 것이 가능하다는 의미다.
    
- Option file:
    
    MySQL의 설정 파일인 my.cnf (또는my.ini)로 제어할 수 있는지 여부를 나타낸다. 
    
    옵션 파일 이나 설정 파일 또는 컨피규레이션 파일등은 전부my.cnf (또는my.ini)
    
    파일을 지칭하는것 으로 같은의미로 사용된다.
    
- System Var:
    
    시스템 변수인지 아닌지를 나타낸다.
    
- Var Scope:
    
    시스템 변수의 적용 범위를 나타낸다. 이 시스템 변수가 영향을 미치는 곳이 MySql 서버 전체인지
    
    아니면 MySql 서버와 클라이언트의 커넥션 ( 또는 세션 ) 만인지를 구분한다.
    
    어떤 변수는 세션과 글로벌 ( Both ) 모두 적용 되기도 한다.
    
- Dynamic:
    
    시스템 변수가 동적인지 정적인지 구분하는 변수이며 동적변수와 정적변수의 차이로만 설명된다 ( 책에서 )
    

### 2.4.4 정적 변수와 동적 변수

Mysql 서버의 시스템 변수는 Mysql 서버가 기동 중인 상태에서 변경 

가능한지에 따라 동적 변수와 정적 변수로 구분된다.

Mysql 서버의 시스템 변수는 디스크에 저장돼 있는 설정 파일(my.cnf 또는 my.ini)을 변경하는 경우와

이미 기동 중인 Mysql 서버의 메모리에 있는 시스템 변수를 변경하는 경우로 구분할 수 있다.

디스크에 저장된 설정 파일은 내용은 변경하더라도 서버를 재시작 하기 전까지는 적용되지 않는다.

하지만 SHOW 명령으로 서버에 적용된 변숫값을 확인하거나 SET 명령을 통해 값을 바꿀 수도 있다.

```sql
SHOW GLOBAL VARIBLES LIKE '%max_connections%';

SET GLOBAL max_connections=500;

```

하지만 SET 명령어로 변경한 시스템 변숫값은 설정파일에 반영되는게 아니기에

서버가 재기동되면 초기화된다. 영구히 저장하려면 반드시 설정파일을 변경하자.

Mysql 8.0 버전부터는 SET PERSIST 명령어를 이용하면 실행중인 서버의 시스템 변수를

변경함과 동시에 자동으로 설정파일로도 기록된다.

### SET PERSIST

Mysql 8.0 버전부터 가능하다.

현재 글로벌 변수도 변경하고 재시작시 불러오는 설정파일에도 적용된다.

```sql
SET PERSIST max_connections=5000;

SHOW GLOBAL VARIABLES LIKE 'max_connections';
```

### SET PERSIST_ONLY

SET PERSIST_ONLY 명령은 정적인 변수의 값을 영구적으로 변경하고자 할 때도 사용 할 수 있다.

해당 명령은 실행중인 Mysql 서버에서 정적인 변수에 대해서 cnf 파일에 기록해두고

서버가 재시작 할 때부터 그 설정을 불러오도록 할 수 있다.

```sql
SET PERSIST innodb_doublewrite=ON;
```

해당 명령으로 시스템 변수를 변경하면 JSON 포맷의 mysqld-auto.cnf 파일이 생성된다.

해당 파일에는 누가 언제 어떤 변수를 변경하였는지 기록된다.