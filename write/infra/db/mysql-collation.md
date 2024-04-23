# Mysql Collation 불일치

```
2024-04-14 16:31:35.723 ERROR 8 --- [o-8080-exec-202] o.h.engine.jdbc.spi.SqlExceptionHelper   : Illegal mix of collations (utf8mb4_general_ci,IMPLICIT) and (utf8_general_ci,COERCIBLE) for operation '='
```

위 에러코드 처럼 "Illegal mix of collations" 오류는 데이터베이스 쿼리에서 서로 다른 문자열 콜레이션(collation)을 사용하여 비교하려고 할 때 발생한다.

* 콜레이션은 문자열 비교 및 정렬 규칙을 정의하며, 각 데이터베이스나 테이블에 대해 설정할 수 있음
* utf8\_general\_ci는 UTF-8 문자셋을 사용하는 일반적인 콜레이션으로, utf8mb4\_general\_ci는 UTF-8 문자셋을 사용하는데, 이는 4바이트 문자(이모지 등)를 지원한다.
* 해당 오류는 보통 데이터베이스나 테이블의 콜레이션 설정이 서로 일치하지 않을 때 발생함
* 현재 에러는 `utf8mb4_general_ci` collation의 문자열을 `utf8_general_ci` collation의 데이터베이스에 삽입하려고 시도했다는 것
* 문자열이 `utf8mb4_general_ci`로 판단되는 기준은 주로 그 문자열이 어떤 유니코드 문자를 포함하고 있는지에 따라 결정
  * 이모지, 다양한 아시아 언어의 문자, 아랍 문자 등
* 데이터베이스의 콜레이션 설정을 다시 검토하고, 데이터베이스 내에서 일관성 있게 설정되도록 보장해야함.

## DB collation 설정점검

* utf8mb4는 MySQL 5.5, MariaDB 5.5 이후 버전에서 사용할 수 있음.

기존의 테이블은 utf8로 처리하고 이모티콘이 저장되는 테이블만 utf8mb4로 저장하거나 변경하면 된다. 그리고 my.cnf에서 기존의 utf8부분이 설정된 부분은 막게 아래처럼 utf8mb4로 해준다.

```
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

참고로 `character-set-client-handshake` 옵션은 클라이언트의 문자셋을 무시하고 서버쪽 문자셋을 이용하는 것이다. 만약 기존 데이터베이스나 테이블이 CHARSET이 UTF8로 되었다면 utf8mb4로 변경해야 한다.

```
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;

ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## springboot 에서는 여전히 문제가 발생한다...

* sql 편집기에서 insert를 실행하면 동작이 잘 되었다
* 하지만 springboot+JPA 에서 save를 하면 여전히 `(utf8mb4_general_ci,IMPLICIT) and (utf8_general_ci,COERCIBLE) for operation '='` 해당 에러가 났다
* 이쯤되면 원인은 서버가 아니라 클라이언트쪽에서 연결 시도 시 collation 관련 원인이 맞다고 좁혀진다
* 하지만 `skip-character-set-client-handshake` 옵션이 my.cnf에 있는데..?
  * 해당 옵션은 MySQL 서버에서 클라이언트와의 문자 세트 핸드셰이크를 생략하는 데 사용된다
  * 이 옵션을 사용하면 클라이언트의 문자 세트 설정을 무시하고 서버에서 지정한 문자 세트를 사용한다
  * 따라서 springboot 혹은 sql편집기 등 client에서 어떤 설정을 하던간에 서버쪽 문자세트를 사용해야하는것이 아닌가?
* JPA와 같은 프레임워크를 사용하는 경우에는 충분하지 않을 수 있다&#x20;
* 프레임워크 내부에서도 문자 세트 설정을 관리하고 있기 때문에, 프레임워크와 데이터베이스 간의 일관성을 유지하기 위해 다른 방법을 추가로 적용해야 할 수 있다&#x20;
* 이러한 경우에는 properties 혹은 yml파일에 `spring.datasource.hikari.connection-init-sql` 속성을 사용하여 데이터베이스와 클라이언트 간의 일관성을 유지하는 것이 좋은 방법일 수 있다.

아래 설정으로 해결

```
spring.datasource.hikari.connection-init-sql=SET NAMES utf8mb4 COLLATE utf8mb4_general_ci;
```

