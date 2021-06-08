# Connection APIs

- [메인 API](#메인-api)
- [`Connection` API](#connection-api)
- [`ConnectionManager` API](#connectionmanager-api)

## 메인 API

* `createConnection()` - 새 연결을 만들고 글로벌 연결 관리자에 등록합니다. 연결 옵션 매개 변수가 생략되면 `ormconfig` 파일 또는 환경 변수에서 연결 옵션을 읽습니다.

```typescript
import {createConnection} from "typeorm";

const connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
});
```

* `createConnections()` - 여러 연결을 만들고 글로벌 연결 관리자에 등록합니다. 연결 옵션 매개변수가 생략되면 `ormconfig` 파일 또는 환경 변수에서 연결 옵션을 읽습니다.

```typescript
import {createConnections} from "typeorm";

const connection = await createConnections([{
    name: "connection1",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
}, {
    name: "connection2",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
}]);
```

* `getConnectionManager()` - 생성된 모든 연결(`createConnection()` 또는 `createConnections()` 사용)을 저장하는 연결 관리자를 가져옵니다.

```typescript
import {getConnectionManager} from "typeorm";

const defaultConnection = getConnectionManager().get("default");
const secondaryConnection = getConnectionManager().get("secondary");
```

* `getConnection()` - `createConnection` 메소드를 사용하여 생성된 연결을 가져옵니다.

```typescript
import {getConnection} from "typeorm";

const connection = getConnection();
// if you have named connection you can specify its name:
const secondaryConnection = getConnection("secondary-connection");
```

* `getEntityManager()` - 연결에서 `EntityManager`를 가져옵니다. 어떤 연결의 엔티티 관리자를 가져와야하는지 나타내기 위해 연결 이름을 지정할 수 있습니다.

```typescript
import {getEntityManager} from "typeorm";

const manager = getEntityManager();
// 이제 관리자 방법을 사용할 수 있습니다.

const secondaryManager = getEntityManager("secondary-connection");
// 보조 연결 관리자 방법을 사용할 수 있습니다.
```

* `getRepository()` - 연결에서 주어진 엔티티에 대한 `Repository`를 가져옵니다. 어떤 연결의 엔티티 관리자를 가져와야하는지 나타내기 위해 연결 이름을 지정할 수 있습니다.

```typescript
import {getRepository} from "typeorm";

const userRepository = getRepository(User);
// 이제 저장소 메소드를 사용할 수 있습니다.

const blogRepository = getRepository(Blog, "secondary-connection");
// 보조 연결 저장소 방법을 사용할 수 있습니다.
```

* `getTreeRepository()` - 연결에서 주어진 엔티티에 대한 `TreeRepository`를 가져옵니다. 어떤 연결의 엔티티 관리자를 가져와야하는지 나타내기 위해 연결 이름을 지정할 수 있습니다.

```typescript
import {getTreeRepository} from "typeorm";

const userRepository = getTreeRepository(User);
// 이제 저장소 메소드를 사용할 수 있습니다.

const blogRepository = getTreeRepository(Blog, "secondary-connection");
// 보조 연결 저장소 방법을 사용할 수 있습니다.
```

* `getMongoRepository()` - 연결에서 주어진 엔티티에 대한`MongoRepository`를 가져옵니다. 어떤 연결의 엔티티 관리자를 가져와야하는지 나타내기 위해 연결 이름을 지정할 수 있습니다.

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
// 이제 저장소 메소드를 사용할 수 있습니다.

const blogRepository = getMongoRepository(Blog, "secondary-connection");
// 보조 연결 저장소 방법을 사용할 수 있습니다.
```

## `Connection` API

* `name` - 연결 이름. 이름없는 연결을 만든 경우 "기본값"과 같습니다. 여러 연결로 작업하고 `getConnection(connectionName : string)`을 호출할 때 이 이름을 사용합니다.

```typescript
const connectionName: string = connection.name;
```

* `options` - 이 연결을 만드는 데 사용되는 연결 옵션입니다. [연결 옵션](./connection-options.md)에 대해 자세히 알아보세요.

```typescript
const connectionOptions: ConnectionOptions = connection.options;
// 사용하는 데이터베이스 드라이버에 따라 connectionOptions를 MysqlConnectionOptions
// 또는 다른 xxxConnectionOptions로 캐스팅할 수 있습니다.
```

* `isConnected` - 데이터베이스에 대한 실제 연결이 설정되었는지 여부를 나타냅니다.

```typescript
const isConnected: boolean = connection.isConnected;
```

* `driver` - 이 연결에 사용되는 기본 데이터베이스 드라이버입니다.

```typescript
const driver: Driver = connection.driver;
// 사용하는 데이터베이스 드라이버에 따라 connectionOptions를 MysqlDriver
// 또는 다른 xxxDriver로 캐스팅할 수 있습니다.
```

* `manager` - 연결 엔터티 작업에 사용되는 `EntityManager`. [엔티티 관리자 및 저장소](working-with-entity-manager.md)에 대해 자세히 알아보세요.

```typescript
const manager: EntityManager = connection.manager;
// 관리자 메소드를 호출할 수 있습니다. 예를 들어 다음을 찾을 수 있습니다.
const user = await manager.findOne(1);
```

* `mongoManager` - `MongoEntityManager`는 mongodb 연결에서 연결 엔티티로 작업하는데 사용되었습니다. MongoEntityManager에 대한 자세한 정보는 [MongoDB](./mongodb.md) 문서를 참조하십시오.

```typescript
const manager: MongoEntityManager = connection.mongoManager;
// 관리자 또는 mongodb-manager 특정 메소드를 호출할 수 있습니다. 예를 들어 다음을 찾을 수 있습니다.
const user = await manager.findOne(1);
```

* `connect` - 데이터베이스에 대한 연결을 수행합니다. `createConnection`을 사용하면 자동으로 `connect`를 호출하므로 직접 호출할 필요가 없습니다.

```typescript
await connection.connect();
```

* `close` - 데이터베이스와의 연결을 닫습니다. 일반적으로 애플리케이션이 종료될 때 이 메소드를 호출합니다.

```typescript
await connection.close();
```

* `synchronize` - 데이터베이스 스키마를 동기화합니다. 연결 옵션에 `synchronize: true`가 설정되면 이 메서드를 호출합니다. 일반적으로 응용 프로그램이 시작될 때이 메서드를 호출합니다.

```typescript
await connection.synchronize();
```

* `dropDatabase` - 데이터베이스와 모든 데이터를 삭제합니다. 이 방법은 모든 데이터베이스 테이블과 해당 데이터를 삭제하므로 프로덕션에서 이 방법에 주의하십시오. 데이터베이스 연결이 설정된 후에 만 사용할 수 있습니다.

```typescript
await connection.dropDatabase();
```

* `runMigrations` - 보류중인 모든 마이그레이션을 실행합니다.

```typescript
await connection.runMigrations();
```

* `undoLastMigration` - 마지막으로 실행한 마이그레이션을 되돌립니다.

```typescript
await connection.undoLastMigration();
```

* `hasMetadata` - 주어진 엔티티에 대한 메타데이터가 등록되었는지 확인합니다. [엔티티 메타데이터](./entity-metadata.md)에 대해 자세히 알아보세요.

```typescript
if (connection.hasMetadata(User))
    const userMetadata = connection.getMetadata(User);
```

* `getMetadata` - 주어진 엔티티의 `EntityMetadata`를 가져옵니다. 테이블 이름을 지정할 수도 있으며 이러한 테이블 이름을 가진 엔터티 메타데이터가 발견되면 반환됩니다. [엔티티 메타데이터](./entity-metadata.md)에 대해 자세히 알아보세요.

```typescript
const userMetadata = connection.getMetadata(User);
// 이제 사용자 엔티티에 대한 정보를 얻을 수 있습니다.
```

* `getRepository` - 주어진 엔티티의 `Repository`를 가져옵니다. 테이블 이름을 지정할 수도 있으며 주어진 테이블의 저장소가 발견되면 반환됩니다. [Repositories](working-with-repository.md)에 대해 자세히 알아보세요.

```typescript
const repository = connection.getRepository(User);
// 이제 저장소 메소드를 호출할 수 있습니다. 예를 들면 다음과 같습니다.
const users = await repository.findOne(1);
```

* `getTreeRepository` - 주어진 엔티티의 `TreeRepository`를 가져옵니다. 테이블 이름을 지정할 수도 있으며 주어진 테이블의 저장소가 발견되면 반환됩니다. [Repositories](working-with-repository.md)에 대해 자세히 알아보세요.

```typescript
const repository = connection.getTreeRepository(Category);
// 이제 트리 저장소 메소드(예: findTrees)를 호출 할 수 있습니다.
const categories = await repository.findTrees();
```

* `getMongoRepository` - 주어진 엔티티의 `MongoRepository`를 가져옵니다. 이 저장소는 MongoDB 연결의 엔티티에 사용됩니다. [MongoDB 지원](./mongodb.md)에 대해 자세히 알아보세요.

```typescript
const repository = connection.getMongoRepository(User);
// 이제 mongodb 특정 저장소 메소드(예: createEntityCursor)를 호출할 수 있습니다.
const categoryCursor = repository.createEntityCursor();
const category1 = await categoryCursor.next();
const category2 = await categoryCursor.next();
```

* `getCustomRepository` - 사용자 정의 저장소를 가져옵니다. [커스텀 리포지토리](./custom-repository.md)에 대해 자세히 알아보세요.

```typescript
const userRepository = connection.getCustomRepository(UserRepository);
// 이제 사용자 정의 저장소 - UserRepository 클래스 내에서 메소드를 호출할 수 있습니다.
const crazyUsers = await userRepository.findCrazyUsers();
```

* `transaction` - 단일 데이터베이스 트랜잭션에서 여러 데이터베이스 요청이 실행되는 단일 트랜잭션을 제공합니다. [트랜잭션](./transactions.md)에 대해 자세히 알아보세요.

```typescript
await connection.transaction(async manager => {
    // 참고: 주어진 관리자 인스턴스를 사용하여 모든 데이터베이스 작업을 수행해야합니다.
    // 이 트랜잭션과 함께 작동하는 EntityManager의 특수 인스턴스는
    // 여기에서 기다리는 것을 잊지 마십시오.
});
```

* `query` - 원시 SQL 쿼리를 실행합니다.

```typescript
const rawData = await connection.query(`SELECT * FROM USERS`);
```

* `createQueryBuilder` - 쿼리 작성에 사용할 수있는 쿼리 작성기를 만듭니다. [QueryBuilder](./select-query-builder.md)에 대해 자세히 알아보세요.

```typescript
const users = await connection.createQueryBuilder()
    .select()
    .from(User, "user")
    .where("user.name = :name", { name: "John" })
    .getMany();
```

* `createQueryRunner` - 단일 실제 데이터베이스 연결을 관리하고 사용하는데 사용되는 쿼리 실행기를 만듭니다. [QueryRunner](./query-runner.md)에 대해 자세히 알아보세요.

```typescript
const queryRunner = connection.createQueryRunner();

// 실제 데이터베이스 연결을 수행하는 connect를 호출한 후에 만 메소드를 사용할 수 있습니다.
await queryRunner.connect();

// .. 이제 쿼리 실행기로 작업하고 해당 메서드를 호출 할 수 있습니다.

// 매우 중요합니다. 작업을 마치면 쿼리 실행기를 릴리스하는 것을 잊지 마십시오.
await queryRunner.release();
```

## `ConnectionManager` API

* `create` - 새로운 연결을 생성하고 관리자에 등록합니다.

```typescript
const connection = connectionManager.create({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
});
```

* `get` - 이름으로 관리자에 저장된 이미 생성된 연결을 가져옵니다.

```typescript
const defaultConnection = connectionManager.get("default");
const secondaryConnection = connectionManager.get("secondary");
```

* `has` - 지정된 연결 관리자에 연결이 등록되어 있는지 확인합니다.

```typescript
if (connectionManager.has("default")) {
    // ...
}
```
