# Connection Options

- [`ConnectionOptions` 란?](#connectionoptions-란)
- [일반적인 연결 옵션](#일반적인-연결-옵션)
- [`mysql` / `mariadb` 연결 옵션](#mysql--mariadb-연결-옵션)
- [`postgres` / `cockroachdb` 연결 옵션](#postgres--cockroachdb-연결-옵션)
- [`sqlite` 연결 옵션](#sqlite-연결-옵션)
- [`better-sqlite3` 연결 옵션](#better-sqlite3-연결-옵션)
- [`capacitor` 연결 옵션](#capacitor-연결-옵션)
- [`cordova` 연결 옵션](#cordova-연결-옵션)
- [`react-native` 연결 옵션](#react-native-연결-옵션)
- [`nativescript` 연결 옵션](#nativescript-연결-옵션)
- [`mssql` 연결 옵션](#mssql-연결-옵션)
- [`mongodb` 연결 옵션](#mongodb-연결-옵션)
- [`sql.js` 연결 옵션](#sqljs-연결-옵션)
- [`expo` 연결 옵션](#expo-연결-옵션)
- [연결 옵션 example](#연결-옵션-example)

## `ConnectionOptions` 란?

연결 옵션은 `createConnection`에 전달하거나 `ormconfig` 파일에서 정의하는 연결 구성입니다. 데이터베이스마다 고유한 연결 옵션이 있습니다.

## 일반적인 연결 옵션

* `type` - 데이터베이스 타입. 사용하는 데이터베이스 엔진을 지정해야합니다. 가능한 값은 "mysql", "postgres", "cockroachdb", "mariadb", "sqlite", "better-sqlite3", "capacitor", "cordova", "nativescript", "oracle", "mssql", "mongodb ", "sqljs ", "react-native"입니다. 이 옵션은 **필수**입니다.

* `name` - 연결 이름. `getConnection(name: string)` 또는 `ConnectionManager.get(name: string)`을 사용하여 필요한 연결을 얻는데 사용합니다. 다른 연결의 연결 이름은 동일할 수 없습니다. 모두 고유해야합니다. 연결 이름이 제공되지 않으면 "기본값"이라고 합니다.

* `extra` - 기본 드라이버에 전달할 추가 연결 옵션입니다. 기본 데이터베이스 드라이버에 추가 설정을 전달하려는 경우 사용하십시오.

* `entities` - 이 연결에 로드 및 사용할 엔티티 또는 엔티티 스키마입니다. 로드할 엔티티 클래스, 엔티티 스키마 클래스 및 디렉토리 경로를 모두 허용합니다. 디렉토리는 glob 패턴을 지원합니다.
예: `entities: [Post, Category, "entity/*.js", "modules/**/entity/*.js"]`.
[엔티티](./entities.md)에 대해 자세히 알아보기.
[엔티티 스키마](separating-entity-definition.md)에 대해 자세히 알아보기.

* `subscribers` - 이 연결에 로드 및 사용할 구독자입니다. 로드할 엔티티 클래스와 디렉토리를 모두 허용합니다. 디렉토리는 glob 패턴을 지원합니다.
예: `subscribers: [PostSubscriber, AppSubscriber, "subscriber/*.js", "modules/**/subscriber/*.js"]`.
[구독자](listeners-and-subscribers.md)에 대해 자세히 알아보기.

* `migrations` - 이 연결에 로드 및 사용할 마이그레이션입니다. 로드할 마이그레이션 클래스와 디렉토리를 모두 허용합니다. 디렉토리는 glob 패턴을 지원합니다.
예: `migrations: [FirstMigration, SecondMigration, "migration/*.js", "modules/**/migration/*.js"]`.
[마이그레이션](./migrations.md)에 대해 자세히 알아보기.

* `logging` - 로깅이 활성화되었는지 여부를 나타냅니다. `true`로 설정하면 쿼리 및 오류 로깅이 활성화됩니다. 사용할 수 있는 다양한 타입의 로깅을 지정할 수도 있습니다 (예: `["query", "error", "schema"]`).
[로깅](./logging.md)에 대해 자세히 알아보기.

* `logger` - 로깅 목적으로 사용되는 로거. 가능한 값은 "advanced-console", "simple-console" 및 "file"입니다. 기본값은 "advanced-console"입니다. `Logger` 인터페이스를 구현하는 로거 클래스를 지정할 수도 있습니다.
[로깅](./logging.md)에 대해 자세히 알아보기.

* `maxQueryExecutionTime` - 쿼리 실행 시간이 주어진 최대 실행 시간(밀리 초)을 초과하면 로거는 이 쿼리를 기록합니다.

* `namingStrategy` - 데이터베이스에서 테이블과 컬럼의 이름을 지정하는 데 사용할 이름 지정 전략입니다. [이름 지정 전략](./naming-strategy.md)에 대해 자세히 알아보세요.

* `entityPrefix` - 이 데이터베이스 연결의 모든 테이블(또는 컬렉션)에 주어진 문자열을 접두사로 붙입니다.

* `entitySkipConstructor` - 데이터베이스에서 엔터티를 역 직렬화할 때 TypeORM이 생성자를 건너뛰어야 하는지 여부를 나타냅니다. 생성자를 호출하지 않으면 전용 속성과 기본 속성이 모두 예상대로 작동하지 않습니다.

* `dropSchema` - 연결이 설정될 때마다 스키마를 삭제합니다. 이 옵션에 주의하고 **프로덕션에서 사용하지 마십시오**. 그렇지 않으면 모든 프로덕션 데이터가 **손실**됩니다. 이 옵션은 디버그 및 개발 중에 유용합니다.

* `synchronize` - 애플리케이션을 시작할 때마다 데이터베이스 스키마를 자동으로 만들어야하는지 여부를 나타냅니다. 이 옵션에 주의하고 프로덕션에서 사용하지 마십시오. 그렇지 않으면 프로덕션 데이터가 손실될 수 있습니다. 이 옵션은 디버그 및 개발 중에 유용합니다. 대신 CLI를 사용하고 schema: sync 명령을 실행할 수 있습니다. MongoDB 데이터베이스의 경우 MongoDB는 스키마가 없기 때문에 스키마를 생성하지 않습니다. 대신 인덱스를 생성하는 것만으로 동기화됩니다.

* `migrationsRun` - 애플리케이션을 시작할 때마다 마이그레이션을 자동으로 실행해야 하는지 여부를 나타냅니다. 또는 CLI를 사용하고 migration:run 명령을 실행할 수 있습니다.

* `migrationsTableName` - 실행된 마이그레이션에 대한 정보를 포함할 데이터베이스의 테이블 이름입니다. 기본적으로 이 테이블을 "마이그레이션" 이라고합니다.

* `cache` - 엔티티 결과 캐싱을 사용합니다. 여기에서 캐시 유형 및 기타 캐시 옵션을 구성할 수도 있습니다. 캐싱에 대한 자세한 내용은 [여기](./caching.md)를 참조하십시오.

* `cli.entitiesDir` - CLI에서 기본적으로 엔티티를 생성해야하는 디렉토리입니다.

* `cli.migrationsDir` - CLI에서 기본적으로 마이그레이션을 만들어야하는 디렉토리입니다.

* `cli.subscribersDir` - CLI에서 기본적으로 구독자를 만들어야하는 디렉터리입니다.

## `mysql` / `mariadb` 연결 옵션

* `url` - 연결을 수행하는 연결 URL입니다. 다른 연결 옵션은 url에서 설정된 매개변수보다 우선합니다.

* `host` - 데이터베이스 호스트.

* `port` - 데이터베이스 호스트 포트. 기본 mysql 포트는 `3306`입니다.

* `username` - 데이터베이스 사용자 이름.

* `password` - 데이터베이스 비밀번호.

* `database` - 데이터베이스 이름.

* `charset` - 연결을 위한 문자 집합입니다. 이를 MySQL의 SQL 수준에서 "데이터 정렬"이라고 합니다 (예: utf8_general_ci). SQL 레벨 문자 세트가 지정되면(예: utf8mb4) 해당 문자 세트에 대한 기본 데이터 정렬이 사용됩니다.(기본값: `UTF8_GENERAL_CI`).

* `timezone` - MySQL 서버에 구성된 시간대. 이것은 서버 날짜/시간 값을 JavaScript Date 객체로 또는 그 반대로 타입 캐스트하는 데 사용됩니다. `local`, `Z` 또는 `+HH:MM` 또는 `-HH:MM` 형식의 오프셋일 수 있습니다. (기본값: `local`)

* `connectTimeout` - MySQL 서버에 처음 연결하는 동안 시간 초과가 발생하기 전의 밀리 초입니다.(기본값: `10000`)

* `acquireTimeout` - MySql 서버에 처음 연결하는 동안 시간 초과가 발생하기 전의 밀리 초입니다. connectTimeout이 아닌 TCP 연결 시간 제한을 제어한다는 점에서 `connectTimeout`과 다릅니다.(기본값: `10000`)

* `insecureAuth` - 이전(안전하지 않은) 인증 방법을 요청하는 MySQL 인스턴스에 대한 연결을 허용합니다. (기본값: `false`)

* `supportBigNumbers` - 데이터베이스에서 큰 숫자(`BIGINT` 및 `DECIMAL` 컬럼)를 처리할 때 이 옵션을 활성화해야 합니다 (기본값: `true`).

* `bigNumberStrings` - `supportBigNumbers` 와 `bigNumberStrings`를 모두 활성화하면 큰 숫자(`BIGINT` 및 `DECIMAL` 열)가 항상 JavaScript String 객체로 반환됩니다(기본값: `true`). `supportBigNumbers`를 활성화하고 `bigNumberStrings`를 비활성화하면 [JavaScript Number objects](http://ecma262-5.com/ELS5_HTML.htm#Section_8.5) (이는 `[-2^53, +2^53]` 범위를 초과할 때 발생합니다. 그렇지 않으면 Number 객체로 반환됩니다. 이 옵션은 `supportBigNumbers`가 비활성화된 경우 무시됩니다.

* `dateStrings` - 날짜 유형(`TIMESTAMP`, `DATETIME`, `DATE`)이 JavaScript Date 객체로 확장되지 않고 문자열로 반환되도록 강제합니다. true/false 또는 문자열로 유지할 타입 이름 배열일 수 있습니다.(기본값: `false`)

* `debug` - 프로토콜 세부 사항을 stdout에 인쇄합니다. true/false 또는 인쇄해야 하는 패킷 타입 이름의 배열일 수 있습니다. (기본값 :`false`)

* `trace` - 라이브러리 입구의 호출 사이트 ("긴 스택 추적")를 포함하도록 오류에 대한 스택 추적을 생성합니다. 대부분의 통화에 대해 약간의 성능 저하.(기본값: `true`)

* `multipleStatements` - 쿼리당 여러 개의 mysql 문을 허용합니다. 이것에 주의하십시오. SQL 인젝션 공격의 범위를 증가시킬 수 있습니다.(기본값: `false`)

* `legacySpatialSupport` - MySQL 8에서 제거된 GeomFromText 및 AsText와 같은 공간 함수를 사용합니다.(기본값: true)

* `flags` - 기본 플래그 외에 사용할 연결 플래그 목록입니다. 기본 항목을 블랙리스트에 올릴 수도 있습니다. 자세한 내용은 [연결 플래그](https://github.com/mysqljs/mysql#connection-flags)를 확인하세요.

* `ssl` -  ssl 매개변수가 있는 객체 또는 ssl 프로필의 이름을 포함하는 문자열. [SSL 옵션](https://github.com/mysqljs/mysql#ssl-options)을 참조하세요.

## `postgres` / `cockroachdb` 연결 옵션

* `url` - 연결을 수행하는 연결 URL입니다. 다른 연결 옵션은 url에서 설정된 매개 변수보다 우선합니다.

* `host` - 데이터베이스 호스트.

* `port` - 데이터베이스 호스트 포트. 기본 postgres 포트는 `5432`입니다.

* `username` - 데이터베이스 사용자 이름.

* `password` - 데이터베이스 비밀번호.

* `database` - 데이터베이스 이름.

* `schema` - 스키마 이름. 기본값은 "public"입니다.

* `connectTimeoutMS` - postgres 서버에 처음 연결하는 동안 시간 초과가 발생하기 전의 밀리 초입니다. `undefined` 또는 `0`으로 설정된 경우 시간 제한이 없습니다. 기본값은 `undefined`입니다.

* `ssl` - SSL 매개 변수가있는 개체입니다. [TLS/SSL](https://node-postgres.com/features/ssl)을 참조하세요.

* `uuidExtension` - UUID를 생성할 때 사용할 Postgres 확장입니다. 기본값은 `uuid-ossp`입니다. `uuid-ossp` 확장을 사용할 수없는 경우 `pgcrypto`로 변경할 수 있습니다.

* `poolErrorHandler` - 기본 풀이 `'error'` 이벤트를 생성할 때 호출되는 함수입니다. 단일 매개변수(오류 인스턴스)를 사용하고 기본적으로 `warn`수준의 로깅을 사용합니다.

* `logNotifications` - postgres 서버 [notice messages](https://www.postgresql.org/docs/current/plpgsql-errors-and-messages.html) 및 [notification events](https://www.postgresql.org/docs/current/sql-notify.html)은 `info` 레벨(기본값: `false`)로 클라이언트 로그에 포함되어야 합니다.

## `sqlite` 연결 옵션

* `database` - 데이터베이스 경로. 예: "./mydb.sql"

## `better-sqlite3` 연결 옵션

* `database` - 데이터베이스 경로. 예: "./mydb.sql"

* `statementCacheSize` - 쿼리 속도를 높이기위한 sqlite 문의 캐시 크기(기본값 100).

* `prepareDatabase` - typeorm에서 데이터베이스를 사용하기 전에 실행할 함수입니다. 여기에서 원래 better-sqlite3 Database 객체에 액세스할 수 있습니다.

## `capacitor` 연결 옵션

* `database` - 데이터베이스 이름(capacitor-sqlite는 접미사 `SQLite.db`를 추가함)

* `driver` - capacitor-sqlite 인스턴스. 예를 들면 `new SQLiteConnection(CapacitorSQLite)`.

## `cordova` 연결 옵션

* `database` - 데이타베이스 이름

* `location` - 데이터베이스를 저장할 위치. 옵션은 [cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage#opening-a-database)를 참조하세요.

## `react-native` 연결 옵션
* `database` - 데이타베이스 이름

* `location` - 데이터베이스를 저장할 위치. 옵션은 [react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage#opening-a-database)를 참조하세요.

## `nativescript` 연결 옵션
* `database` - 데이타베이스 이름

## `mssql` 연결 옵션

* `url` - 연결을 수행하는 연결 URL입니다. 다른 연결 옵션은 url에서 설정된 매개변수를 대체합니다.

* `host` - 데이터베이스 호스트.

* `port` - 데이터베이스 호스트 포트. 기본 mssql 포트는 `1433`입니다.

* `username` - 데이터베이스 사용자 이름.

* `password` - 데이터베이스 비밀번호.

* `database` - 데이타베이스 이름.

* `schema` - 스키마 이름. 기본값은 "public"입니다.

* `domain` - 도메인을 설정하면 드라이버가 도메인 로그인을 사용하여 SQL Server에 연결합니다.

* `connectionTimeout` - 연결 시간 제한(ms) (기본값: `15000`).

* `requestTimeout` - 요청 시간 제한(ms) (기본값: `15000`). 참고: msnodesqlv8 드라이버는 1초 미만의 시간 초과를 지원하지 않습니다.

* `stream` - 레코드 세트/행을 콜백 인수로 한 번에 모두 반환하는 대신 스트림합니다 (기본값: `false`). 각 요청에 대해 독립적으로 스트리밍을 활성화 할 수도 있습니다(`request.stream = true`). 많은 양의 행으로 작업하려는 경우 항상 `true`로 설정하십시오.

* `pool.max` - 풀에 있을 수 있는 최대 연결 수(기본값: `10`).

* `pool.min` - 풀에있을 수있는 최소 연결 수(기본값: `0`).

* `pool.maxWaitingClients` - 허용되는 최대 대기 요청 수, 추가 취득 호출은 이벤트 루프의 향후 주기에서 오류와 함께 콜백됩니다.

* `pool.testOnBorrow` -  풀이 리소스를 클라이언트에게 제공하기 전에 유효성을 검사해야합니다. `factory.validate` 또는 `factory.validateAsync`를 지정해야합니다.

* `pool.acquireTimeoutMillis` - `acquire` 호출은 시간 초과 전에 리소스를 기다릴 최대 밀리 초입니다.(기본값은 제한 없음), 제공된 경우 0이 아닌 양의 정수여야합니다.

* `pool.fifo` - 참이면 가장 오래된 리소스가 먼저 할당됩니다. false이면 가장 최근에 릴리스된 리소스가 가장 먼저 할당됩니다. 이는 사실상 풀의 동작을 대기열에서 스택으로 바꿉니다. 부울(기본값 `true`).

* `pool.priorityRange` - 1과 x 사이의 int - 설정된 경우 차용자는 사용 가능한 리소스가 없는 경우 대기열에서 상대적 우선 순위를 지정할 수 있습니다. 예를 참조하십시오. (기본값 `1`).

* `pool.autostart` - boolean, 생성자가 호출되면 풀이 리소스 생성 등을 시작해야합니다 (기본값 `true`).

* `pool.evictionRunIntervalMillis` - 퇴거 확인을 실행하는 빈도입니다. 기본값: `0`(실행되지 않음).

* `pool.numTestsPerRun` - 각 제거 실행을 확인할 리소스 수입니다. 기본값: `3`.

* `pool.softIdleTimeoutMillis` - 최소한 "최소 유휴" 객체 인스턴스가 풀에 남아 있다는 추가 조건과 함께 유휴 객체 축출기(있는 경우)가 제거할 수 있기 전에 객체가 풀에서 유휴 상태로 있을 수 있는 시간입니다. 기본값은 `-1`(아무것도 제거할 수 없음)입니다.

* `pool.idleTimeoutMillis` -  유휴 시간으로 인해 제거될 수 있기 전에 객체가 풀에서 유휴 상태로 있을 수 있는 최소 시간입니다. `softIdleTimeoutMillis`를 대체합니다. 기본값: `30000`.

 * `pool.errorHandler` - 기본 풀이 `'error'` 이벤트를 생성 할 때 호출되는 함수입니다. 단일 매개변수(오류 인스턴스)를 사용하고 기본적으로 `경고` 수준의 로깅을 사용합니다.

* `options.fallbackToDefaultDb` - 기본적으로 `options.database`에 의한 데이터베이스 요청에 액세스할 수 없는 경우 오류와 함께 연결이 실패합니다. 그러나 `options.fallbackToDefaultDb`가 `true`로 설정되면 사용자의 기본 데이터베이스가 대신 사용됩니다(기본값: `false`).

* `options.instanceName` - 연결할 인스턴스 이름입니다. SQL Server Browser 서비스는 데이터베이스 서버에서 실행 중이어야 하며 데이터베이스 서버의 UDP 포트 1434에 연결할 수 있어야 합니다. `port`와 상호 배타적입니다.(기본값 없음).

* `options.enableAnsiNullDefault` - true이면 초기 SQL에 SET ANSI_NULL_DFLT_ON ON이 설정됩니다. 즉, 새 열은 기본적으로 nullable이됩니다. 자세한 내용은 [T-SQL 문서](https://msdn.microsoft.com/en-us/library/ms187375.aspx)를 참조하세요.(기본값: `true`).

* `options.cancelTimeout` - 요청 취소(중단)가 실패한 것으로 간주되기 전까지의 시간(밀리 초)입니다 (기본값: `5000`).

* `options.packetSize` - TDS 패킷의 크기(서버와의 협상에 따름). 2의 거듭 제곱이어야합니다(기본값: `4096`).

* `options.useUTC` - UTC 또는 현지 시간으로 시간 값을 전달할지 여부를 결정하는 부울입니다.(기본값: `true`).

* `options.abortTransactionOnError` - 주어진 트랜잭션 실행 중에 오류가 발생하면 트랜잭션을 자동으로 롤백할지 여부를 결정하는 부울입니다. 연결의 초기 SQL 단계 ([documentation](http://msdn.microsoft.com/en-us/library/ms188792.aspx))에서 `SET XACT_ABORT` 값을 설정합니다.

* `options.localAddress` - SQL Server에 연결할 때 사용할 네트워크 인터페이스(ip 주소)를 나타내는 문자열입니다.

* `options.useColumnNames` - 행을 배열로 반환할지 키-값 컬렉션으로 반환할지를 결정하는 부울입니다.(기본값: `false`).

* `options.camelCaseColumns` - 반환된 열 이름의 첫 글자가 소문자로 변환되는지('true') 여부를 제어하는 부울입니다.  `columnNameReplacer`를 제공하면 이 값은 무시됩니다.(기본값: `false`).

* `options.isolationLevel` - 트랜잭션이 실행될 기본 격리 수준입니다. 격리 수준은 `require('tedious').ISOLATION_LEVEL`에서 사용할 수 있습니다.
   * `READ_UNCOMMITTED`
   * `READ_COMMITTED`
   * `REPEATABLE_READ`
   * `SERIALIZABLE`
   * `SNAPSHOT`

   (디폴트: `READ_COMMITTED`)

* `options.connectionIsolationLevel` - 새 연결에 대한 기본 격리 수준입니다. 이 설정으로 모든 트랜잭션 외 쿼리가 실행됩니다. 격리 수준은 `require('tedious').ISOLATION_LEVEL`에서 사용할 수 있습니다.
   * `READ_UNCOMMITTED`
   * `READ_COMMITTED`
   * `REPEATABLE_READ`
   * `SERIALIZABLE`
   * `SNAPSHOT`

   (디폴트: `READ_COMMITTED`)

* `options.readOnlyIntent` - 연결이 SQL Server 가용성 그룹에서 읽기 전용 액세스를 요청할지 여부를 결정하는 부울입니다. 자세한 내용은 여기를 참조하십시오.(기본값: `false`).

* `options.encrypt` - 연결을 암호화할지 여부를 결정하는 부울입니다. Windows Azure를 사용하는 경우 true로 설정합니다. (기본값 :`false`).

* `options.cryptoCredentialsDetails` - 암호화를 사용하면 [tls.createSecurePair](http://nodejs.org/docs/latest/api/tls.html#tls_tls_createsecurepair_credentials_isserver_requestcert_rejectunauthorized)를 호출할 때 첫번째 인수에 사용할 객체를 제공할 수 있습니다(기본값: `{}`).

* `options.rowCollectionOnDone` - true 일 때 Requests의 `done*` 이벤트에서 수신된 행을 노출하는 부울입니다. 완료 참조, [doneInProc](http://tediousjs.github.io/tedious/api-request.html#event_doneInProc) 및 [doneProc](http://tediousjs.github.io/tedious/api-request.html# event_doneProc). (기본값: `false`)

   주의: 많은 행이 수신되는 경우 이 옵션을 활성화하면 과도한 메모리 사용량이 발생할 수 있습니다.

* `options.rowCollectionOnRequestCompletion` - true 일 때 Requests의 완료 콜백에서 수신된 행을 노출하는 부울입니다. [새 요청](http://tediousjs.github.io/tedious/api-request.html#function_newRequest)을 참조하십시오. (기본값: `false`)

   주의: 많은 행이 수신되는 경우 이 옵션을 활성화하면 과도한 메모리 사용량이 발생할 수 있습니다.

* `options.tdsVersion` - 사용할 TDS의 버전입니다. 서버가 지정된 버전을 지원하지 않으면 협상된 버전이 대신 사용됩니다. 버전은 `require('tedious').TDS_VERSION`에서 사용할 수 있습니다.
   * `7_1`
   * `7_2`
   * `7_3_A`
   * `7_3_B`
   * `7_4`

  (default: `7_4`)

* `options.debug.packet` - 패킷 세부 정보를 설명하는 텍스트와 함께 `debug` 이벤트를 내보낼 지 여부를 제어하는 부울입니다 (기본값: `false`).

* `options.debug.data` - 패킷 데이터 세부 정보를 설명하는 텍스트와 함께 `debug` 이벤트를 내보낼 지 여부를 제어하는 부울 (기본값: `false`)입니다.

* `options.debug.payload` - 패킷 페이로드 세부 정보를 설명하는 텍스트와 함께 `debug` 이벤트를 내보낼 지 여부를 제어하는 부울 (기본값: `false`)입니다.

* `options.debug.token` - 토큰 스트림 토큰을 설명하는 텍스트와 함께 `debug` 이벤트를 내보낼 지 여부를 제어하는 부울입니다 (기본값: `false`).

## `mongodb` 연결 옵션

* `url` - 연결을 수행하는 연결 URL입니다. 다른 연결 옵션은 url에서 설정된 매개변수를 대체합니다.

* `host` - 데이터베이스 호스트.

* `port` - 데이터베이스 호스트 포트. 기본 mongodb 포트는 `27017`입니다.

* `username` - 데이터베이스 사용자 이름(`auth.user` 대체).

* `password` - 데이터베이스 비밀번호(`auth.password` 대체).

* `database` - 데이타베이스 이름.

* `poolSize` - 각 개별 서버 또는 프록시 연결에 대한 최대 풀 크기를 설정합니다.

* `ssl` - SSL 연결을 사용합니다(ssl을 지원하는 mongodb 서버가 필요함). 기본값: `false`.

* `sslValidate` - ca에 대해 mongodb 서버 인증서의 유효성을 검사합니다(ssl을 지원하는 mongodb 서버, 2.4 이상이 필요함). 기본값: `true`.

* `sslCA` - 버퍼 또는 문자열로 유효한 인증서 배열 (ssl을 지원하는 mongodb 서버, 2.4 이상이 필요함).

* `sslCert` - 우리가 제시하고자 하는 인증서를 포함하는 문자열 또는 버퍼 (ssl 지원, 2.4 이상이있는 mongodb 서버가 필요함)

* `sslKey` - 제시하려는 인증서 개인 키를 포함하는 문자열 또는 버퍼 (ssl 지원, 2.4 이상을 지원하는 mongod 서버가 필요함).

* `sslPass` - 인증서 비밀번호를 포함하는 문자열 또는 버퍼 (ssl 지원, 2.4 이상을 지원하는 mongodb 서버가 있어야 함).

* `autoReconnect` - 오류 발생시 다시 연결하십시오. 기본값: `true`.

* `noDelay` - TCP 소켓 NoDelay 옵션. 기본값: `true`.

* `keepAlive` - TCP 소켓에서 keepAlive를 시작하기 전에 대기하는 시간 (밀리 초)입니다. 기본값: `30000`.

* `connectTimeoutMS` - TCP 연결 시간 초과 설정. 기본값: `30000`.

* `socketTimeoutMS` - TCP 소켓 제한 시간 설정. 기본값: `360000`.

* `reconnectTries` - 서버가 #회 재 연결을 시도합니다. 기본값: `30`.

* `reconnectInterval` - 서버는 재시도 사이에 #밀리 초를 기다립니다. 기본값: `1000`.

* `ha` - 고 가용성 모니터링을 켭니다. 기본값: `true`.

* `haInterval` - 각 복제 세트 상태 확인 사이의 시간입니다. 기본값: `10000,5000`.

* `replicaSet` - 연결할 복제 세트의 이름입니다.

* `acceptableLatencyMS` - NEAREST를 사용할 때 선택할 서버 범위를 설정합니다 (가장 낮은 ping ms + 대기 시간 차단, 예: 범위 1 ~ (1 + 15) ms). 기본값: `15`.

* `secondaryAcceptableLatencyMS` - NEAREST를 사용할 때 선택할 서버 범위를 설정합니다 (가장 낮은 ping ms + 대기 시간 차단, 예: 범위 1 ~ (1 + 15) ms). 기본값: `15`.

* `connectWithNoPrimary` - 기본이없는 경우에도 드라이버를 연결해야하는지 여부를 설정합니다. 기본값: `false`.

* `authSource` - 데이터베이스 인증이 다른 databaseName에 종속된 경우.

* `w` - 쓰기 문제.

* `wtimeout` - 쓰기 문제 시간 초과 값입니다.

* `j` - 저널 쓰기 문제를 지정하십시오. 기본값: `false`.

* `forceServerObjectId` - 서버가 드라이버 대신 _id 값을 할당하도록합니다. 기본값: `false`.

* `serializeFunctions` - 모든 개체의 함수를 직렬화합니다. 기본값: `false`.

* `ignoreUndefined` - BSON 직렬 변환기가 정의되지 않은 필드를 무시할지 여부를 지정합니다. 기본값: `false`.

* `raw` - 문서 결과를 원시 BSON 버퍼로 반환합니다. 기본값: `false`.

* `promoteLongs` - Long 값이 53 비트 해상도에 맞는 경우 숫자로 승격합니다. 기본값: `true`.

* `promoteBuffers` - 바이너리 BSON 값을 네이티브 노드 버퍼로 승격합니다. 기본값: `false`.

* `promoteValues` - 가능한 경우 BSON 값을 기본 유형으로 승격하고 래퍼 유형 만 수신하려면 false로 설정합니다. 기본값: `true`.

* `domainsEnabled` - 성능 적중을 방지하기 위해 기본적으로 비활성화된 현재 도메인에서 콜백 래핑을 활성화합니다. 기본값: `false`.

* `bufferMaxEntries` - 작업 연결을 포기하기 전에 드라이버가 버퍼링할 작업 수에 대한 한도를 설정합니다. 기본값은 무제한인 -1입니다.

* `readPreference` - 선호하는 읽기 환경 설정입니다.
  * `ReadPreference.PRIMARY`
  * `ReadPreference.PRIMARY_PREFERRED`
  * `ReadPreference.SECONDARY`
  * `ReadPreference.SECONDARY_PREFERRED`
  * `ReadPreference.NEAREST`

* `pkFactory` - 사용자 지정 _id 키 생성을위한 기본 키 팩토리 개체입니다.

* `promiseLibrary` - Bluebird와 같이 애플리케이션이 사용하려는 Promise 라이브러리 클래스는 ES6와 호환되어야합니다.

* `readConcern` - 컬렉션에 대한 읽기 문제를 지정합니다. (MongoDB 3.2 이상 만 지원됨).

* `maxStalenessSeconds` - 보조 읽기에 대해 maxStalenessSeconds 값을 지정합니다. 최소값은 90 초입니다.

* `appname` - 이 MongoClient 인스턴스를 생성한 애플리케이션의 이름입니다. MongoDB 3.4 이상은 각 연결을 설정할 때 서버 로그에 이 값을 인쇄합니다. 느린 쿼리 로그 및 프로필 컬렉션에도 기록됩니다.

* `loggerLevel` - 드라이버 로거에서 사용하는 로그 수준을 지정합니다 (`error/warn/info/debug`).

* `logger` - 고객 로거 메커니즘을 지정하고 앱 수준 로거를 사용하여 로깅하는데 사용할 수 있습니다.

* `authMechanism` - MongoDB가 연결을 인증하는 데 사용할 인증 메커니즘을 설정합니다.

## `sql.js` 연결 옵션

* `database`: 가져와야하는 원시 UInt8Array 데이터베이스입니다.

* `sqlJsConfig`: sql.js에 대한 선택적 초기화 구성.

* `autoSave`: 자동 저장을 비활성화할지 여부입니다. true로 설정하면 변경이 발생하고 `location`이 지정될 때 데이터베이스가 주어진 파일 위치(Node.js) 또는 LocalStorage 요소(브라우저)에 저장됩니다. 그렇지 않으면 `autoSaveCallback`을 사용할 수 있습니다.

* `autoSaveCallback`: 데이터베이스가 변경되고 `autoSave`가 활성화되면 호출되는 함수입니다. 이 함수는 데이터베이스를 나타내는 `UInt8Array`를 가져옵니다.

* `location`: 데이터베이스를 로드하고 저장할 파일 위치입니다.

* `useLocalForage`: [localforage 라이브러리](https://github.com/localForage/localForage)를 사용하여 브라우저 환경에서 로컬 동기화 저장소 방법을 사용하는 대신 indexedDB에서 비동기 적으로 데이터베이스를 저장하고 로드할 수 있습니다. localforage 노드 모듈을 프로젝트에 추가하고 localforage.js를 페이지로 가져와야합니다.

## `expo` 연결 옵션

* `database` - 데이터베이스의 이름입니다. 예: "mydb".
* `driver` - Expo SQLite 모듈. 예: `require('expo-sqlite')`.

## 연결 옵션 example

Here is a small example of 연결 옵션 for mysql:

```typescript
{
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    logging: true,
    synchronize: true,
    entities: [
        "entity/*.js"
    ],
    subscribers: [
        "subscriber/*.js"
    ],
    entitySchemas: [
        "schema/*.json"
    ],
    migrations: [
        "migration/*.js"
    ],
    cli: {
        entitiesDir: "entity",
        migrationsDir: "migration",
        subscribersDir: "subscriber"
    }
}
```
