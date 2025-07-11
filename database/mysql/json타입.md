> 참고: [MySQL JSON 공식 문서](https://dev.mysql.com/doc/refman/8.0/en/json.html)

## MySQL JSON 타입의 특징

MySQL에서는 JSON을 단순 문자열로 저장하지 않고, 전용 JSON 타입으로 저장할 수 있다.
이는 단순히 문자열에 {} 넣는 것보다 성능과 기능 면에서 유리하다.

- JSON 컬럼에 저장되는 JSON 문서는 **자동으로 유효성 검사(Validation)** 가 이루어지기 때문에 만약 저장하려는 데이터가 올바른 JSON 형식이 아니면 오류가 발생한다.
  - ex) {name: "belluga"} 처럼 key에 쌍따옴표가 없는 JSON은 문법적으로 잘못된 JSON이기 때문에 저장이 안 되고 오류가 난다.
  
- JSON 데이터를 문자열로 저장하면 읽을 때마다 **파싱(parse)** 이 필요한데, JSON 타입은 내부적으로 바이너리 형식으로 저장되기 때문에 전체를 다 읽지 않고 필요한 값만 빠르게 찾아 읽을 수 이다.

JSON 컬럼에 저장되는 JSON 문서의 최대 크기는 MySQL 설정 변수인 `max_allowed_packet` 값으로 설정할 수 있다.
(`max_allowed_packet` 은 MySQL 서버가 한 번에 처리할 수 있는 최대 패킷 크기를 의미하며, 기본값은 보통 4MB이다)

MySQL 8.0.13 이전 버전에서는 JSON 컬럼에 NULL이 아닌 **기본값(default value)** 을 설정할 수 없었는데 8.0.13 부터는 가능해졌다. (ex: CREATE TABLE table (data JSON DEFAULT '{}'))

JSON을 위한 다양한 SQL 함수가 제공된다.


SON 컬럼은 내부적으로 바이너리로 저장되기 때문에 직접 인덱스를 걸 수 없다. 
대신 JSON에서 값을 뽑아내는 **생성된 컬럼(generated column)** 을 만들고 그 컬럼에 인덱스를 설정할 수 있다.


MySQL 8.0.17 이상에서는 InnoDB 스토리지 엔진이 JSON 배열 안의 각 원소에 대해 인덱스를 부여할 수 있는 기능을 지원한다.

## Partial Updates of JSON Values