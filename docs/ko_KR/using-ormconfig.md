# Using Configuration Sources

- [구성 파일에서 새 연결 만들기](#구성-파일에서-새-연결-만들기)
- [`ormconfig.json` 사용](#ormconfigjson-사용)
- [`ormconfig.js` 사용](#ormconfigjs-사용)
- [환경 변수 사용](#환경-변수-사용)
- [`ormconfig.yml` 사용](#ormconfigyml-사용)
- [`ormconfig.xml` 사용](#ormconfigxml-사용)
- [Typeorm에서 사용하는 구성 파일](#typeorm에서-사용하는-구성-파일)
- [ormconfig에 정의된 옵션 재정의](#ormconfig에-정의된-옵션-재정의)

## 구성 파일에서 새 연결 만들기

대부분의 경우 연결 옵션을 별도의 구성 파일에 저장하려고 합니다. 편리하고 쉽게 관리할 수 있습니다. TypeORM은 여러 구성 소스를 지원합니다. 애플리케이션의 루트 디렉토리(`package.json` 근처)에 `ormconfig.[format]` 파일을 만들고 구성을 배치한 다음 앱에서 구성을 전달하지 않고 `createConnection()`을 호출하기만하면 됩니다.

```typescript
import {createConnection} from "typeorm";

// createConnection 메소드는 ormconfig 파일 또는
// 환경 변수에서 연결 옵션을 자동으로 읽습니다.
const connection = await createConnection();
```

지원되는 ormconfig 파일 형식은 다음과 같습니다: `.json`, `.js`, `.ts`, `.env`, `.yml` and `.xml`.

## `ormconfig.json` 사용

프로젝트 루트(`package.json` 근처)에 `ormconfig.json`을 만듭니다. 다음 내용이 있어야 합니다.

```json
{
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test"
}
```

[ConnectionOptions](./connection-options.md)에서 다른 옵션을 지정할 수 있습니다.

여러 연결을 생성하려면 단일 배열에 여러 연결을 생성하면됩니다.

```json
[{
   "name": "default",
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test"
}, {
   "name": "second-connection",
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test"
}]
```

## `ormconfig.js` 사용

프로젝트 루트(`package.json` 근처)에 `ormconfig.js`를 만듭니다. 다음 내용이 있어야 합니다.

```javascript
module.exports = {
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test"
}
```

또는 환경에서 지원하는 경우 ECMAScript 모듈 형식을 사용할 수 있습니다.

```javascript
export default {
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test"
}
```

[ConnectionOptions](./connection-options.md)에서 다른 옵션을 지정할 수 있습니다.
여러 연결을 만들려면 단일 배열에 여러 연결을 만들고 반환하면 됩니다.

## 환경 변수 사용

프로젝트 루트(`package.json` 근처)에 `.env` 또는 `ormconfig.env`를 만듭니다. 다음 내용이 있어야 합니다.

```ini
TYPEORM_CONNECTION = mysql
TYPEORM_HOST = localhost
TYPEORM_USERNAME = root
TYPEORM_PASSWORD = admin
TYPEORM_DATABASE = test
TYPEORM_PORT = 3000
TYPEORM_SYNCHRONIZE = true
TYPEORM_LOGGING = true
TYPEORM_ENTITIES = entity/*.js,modules/**/entity/*.js
```

설정할 수 있는 사용 가능한 환경변수 목록 :

* TYPEORM_CACHE
* TYPEORM_CACHE_ALWAYS_ENABLED
* TYPEORM_CACHE_DURATION
* TYPEORM_CACHE_OPTIONS
* TYPEORM_CONNECTION
* TYPEORM_DATABASE
* TYPEORM_DEBUG
* TYPEORM_DRIVER_EXTRA
* TYPEORM_DROP_SCHEMA
* TYPEORM_ENTITIES
* TYPEORM_ENTITIES_DIR
* TYPEORM_ENTITY_PREFIX
* TYPEORM_HOST
* TYPEORM_LOGGER
* TYPEORM_LOGGING
* TYPEORM_MAX_QUERY_EXECUTION_TIME
* TYPEORM_MIGRATIONS
* TYPEORM_MIGRATIONS_DIR
* TYPEORM_MIGRATIONS_RUN
* TYPEORM_MIGRATIONS_TABLE_NAME
* TYPEORM_PASSWORD
* TYPEORM_PORT
* TYPEORM_SCHEMA
* TYPEORM_SID
* TYPEORM_SUBSCRIBERS
* TYPEORM_SUBSCRIBERS_DIR
* TYPEORM_SYNCHRONIZE
* TYPEORM_URL
* TYPEORM_USERNAME
* TYPEORM_UUID_EXTENSION

`TYPEORM_CACHE`는 부울 또는 캐시 타입의 문자열이어야 합니다.

`ormconfig.env`는 개발중에만 사용해야합니다. 프로덕션에서 이러한 모든 값을 실제 환경변수로 설정할 수 있습니다.

`env` 파일 또는 환경변수를 사용하여 여러 연결을 정의할 수 없습니다. 앱에 여러 연결이 있는 경우 대체 구성 저장소 형식을 사용합니다.

드라이버별 옵션을 전달해야하는 경우(예: MySQL의 경우 `charset`의 경우 JSON 형식의 `TYPEORM_DRIVER_EXTRA` 변수를 사용할 수 있습니다.

```
TYPEORM_DRIVER_EXTRA='{"charset": "utf8mb4"}'
```
## `ormconfig.yml` 사용

프로젝트 루트(`package.json` 근처)에 `ormconfig.yml`을 만듭니다. 다음 내용이 있어야 합니다.

```yaml
default: # 기본 연결
    host: "localhost"
    port: 3306
    username: "test"
    password: "test"
    database: "test"

second-connection: # 다른 연결
    host: "localhost"
    port: 3306
    username: "test"
    password: "test"
    database: "test2"
```

사용 가능한 모든 연결 옵션을 사용할 수 있습니다.

## `ormconfig.xml` 사용

프로젝트 루트(`package.json` 근처)에 `ormconfig.xml`을 만듭니다. 다음 내용이 있어야 합니다.

```xml
<connections>
    <connection type="mysql" name="default">
        <host>localhost</host>
        <username>root</username>
        <password>admin</password>
        <database>test</database>
        <port>3000</port>
        <logging>true</logging>
    </connection>
    <connection type="mysql" name="second-connection">
        <host>localhost</host>
        <username>root</username>
        <password>admin</password>
        <database>test2</database>
        <port>3000</port>
        <logging>true</logging>
    </connection>
</connections>
```

사용 가능한 모든 연결 옵션을 사용할 수 있습니다.

## Typeorm에서 사용하는 구성 파일

경우에 따라 다른 형식을 사용하여 여러 구성을 사용할 수 있습니다. `getConnectionOptions()`를 호출하거나 연결 옵션없이 `createConnection()`을 사용하려고하면 Typeorm은 다음 순서로 구성 로드를 시도합니다.

1. 환경변수에서. Typeorm은 dotEnv를 사용하여 `.env` 파일을 로드하려고 시도합니다. 환경 변수 `TYPEORM_CONNECTION` 또는 `TYPEORM_URL`이 설정된 경우 Typeorm은 이 메서드를 사용합니다.
2. `ormconfig.env`에서.
3. 다른 `ormconfig.[format]` 파일에서 순서대로 `[js, ts, json, yml, yaml, xml]`.

Typeorm은 발견된 첫번째 유효한 메서드를 사용하고 다른 메서드는 로드하지 않습니다. 예를 들어 Typeorm은 환경에서 구성이 발견된 경우 `ormconfig.[format]` 파일을 로드하지 않습니다.

## ormconfig에 정의된 옵션 재정의

때때로 ormconfig 파일에 정의된 값을 재정의하거나 구성에 TypeScript/JavaScript 로직을 추가할 수 있습니다.

이러한 경우 ormconfig에서 옵션을 로드하고 `ConnectionOptions`를 빌드한 다음 `createConnection` 함수에 전달하기 전에 해당 옵션으로 원하는 모든 작업을 수행할 수 있습니다.


```typescript
// ormconfig 파일(또는 ENV 변수)에서 연결 옵션 읽기
const connectionOptions = await getConnectionOptions();

// connectionOptions로 작업을 수행합니다.
// 예를 들어 사용자 지정 명명 전략 또는 사용자 지정 로거 추가
Object.assign(connectionOptions, { namingStrategy: new MyNamingStrategy() });

// 수정된 연결 옵션을 사용하여 연결 생성
const connection = await createConnection(connectionOptions);
```
