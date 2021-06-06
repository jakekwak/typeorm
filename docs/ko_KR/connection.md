# Connection과 작업하기

- ['커넥션'이란?](#커넥션이란)
- [새로운 커넥션 생성](#새로운-커넥션-생성)
- [`ConnectionManager` 사용하기](#connectionmanager-사용하기)
- [커넥션과 작업하기](#커넥션과-작업하기)

## '커넥션'이란?

데이터베이스와의 상호작용은 연결을 설정한 후에 만 가능합니다. TypeORM의 `Connection`은 보이는 것처럼 데이터베이스 연결을 설정하지 않고 대신 연결 풀을 설정합니다. 실제 데이터베이스 연결에 관심이 있다면 `QueryRunner` 문서를 참조하십시오. `QueryRunner`의 각 인스턴스는 별도의 격리된 데이터베이스 연결입니다. Connection Pool 설정은 `Connection`의 `connect`메소드가 호출되면 설정됩니다. `createConnection` 함수를 사용하여 연결을 설정하면 `connect` 메소드가 자동으로 호출됩니다. 연결해제 (풀의 모든 연결 닫기)는 `close`가 호출 될 때 이루어집니다. 일반적으로 애플리케이션 부트스트랩에서 연결을 한 번만 생성하고 데이터베이스 작업을 완전히 마친 후에 연결을 닫아야합니다. 실제로 사이트에 대한 백엔드를 구축하고 백엔드 서버가 항상 실행중인 경우 연결을 닫지 않습니다.

## 새로운 커넥션 생성

연결을 만드는 방법에는 여러 가지가 있습니다. 가장 간단하고 일반적인 방법은 `createConnection` 및 `createConnections` 함수를 사용하는 것입니다.

`createConnection`은 단일 연결을 생성합니다.

```typescript
import {createConnection, Connection} from "typeorm";

const connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
});
```
단일 `url` 속성과 `type` 속성도 작동합니다.

```js
createConnection({
    type: 'postgres',
    url: 'postgres://test:test@localhost/test'
})
```

`createConnections`는 여러 연결을 만듭니다.

```typescript
import {createConnections, Connection} from "typeorm";

const connections = await createConnections([{
    name: "default",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
}, {
    name: "test2-connection",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test2"
}]);
```

이 두 함수는 전달하는 연결 옵션을 기반으로 `Connection`을 만들고 `connect`메서드를 호출합니다. 프로젝트의 루트에 [ormconfig.json](./using-ormconfig.md) 파일을 만들 수 있습니다.
연결 옵션은 이러한 방법에 의해 이 파일에서 자동으로 읽혀집니다. 프로젝트의 루트는 `node_modules` 디렉토리가있는 레벨과 같습니다.

```typescript
import {createConnection, createConnections, Connection} from "typeorm";

// 여기서 createConnection은 ormconfig.json / ormconfig.js /
// ormconfig.yml / ormconfig.env / ormconfig.xml 파일
// 또는 특수 환경 변수에서 연결 옵션을로드합니다.
const connection: Connection = await createConnection();

// 생성 할 연결의 이름을 지정할 수 있습니다
// (이름을 생략하면 지정된 이름없이 연결이 생성됩니다).
const secondConnection: Connection = await createConnection("test2-connection");

// createConnection 대신 createConnections를 호출하면
// ormconfig 파일에 정의 된 모든 연결을 초기화하고 반환합니다.
const connections: Connection[] = await createConnections();
```

연결마다 이름이 달라야합니다. 기본적으로 연결 이름이 지정되지 않은 경우 `default`와 같습니다. 일반적으로 여러 데이터베이스 또는 여러 연결 구성을 사용할 때 여러 연결을 사용합니다.

연결을 만든 후에는 `getConnection`함수를 사용하여 앱에서 어디서나 연결할 수 있습니다.

```typescript
import {getConnection} from "typeorm";

// createConnection이 호출되고 해결되면 사용할 수 있습니다.
const connection = getConnection();

// 여러 연결이 있는 경우 이름으로 연결할 수 있습니다.
const secondConnection = getConnection("test2-connection");
```

연결을 저장하고 관리하기 위해 추가 클래스 / 서비스를 생성하지 마십시오. 이 기능은 이미 TypeORM에 포함되어 있습니다.
불필요한 추상화를 과도하게 설계하고 만들 필요가 없습니다.

## `ConnectionManager` 사용하기

`ConnectionManager`클래스를 사용하여 연결을 생성할 수 있습니다. 예를 들면:

```typescript
import {getConnectionManager, ConnectionManager, Connection} from "typeorm";

const connectionManager = getConnectionManager();
const connection = connectionManager.create({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
});
await connection.connect(); // 연결을 수행
```

이것은 연결을 만드는 일반적인 방법은 아니지만 일부 사용자에게 유용할 수 있습니다. 예를 들어, 연결을 만들고 인스턴스를 저장하려고 하지만 실제 "연결"이 설정되는 시기를 제어해야 하는 사용자입니다. 또한 고유한 `ConnectionManager`를 만들고 유지할 수 있습니다.

```typescript
import {getConnectionManager, ConnectionManager, Connection} from "typeorm";

const connectionManager = new ConnectionManager();
const connection = connectionManager.create({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
});
await connection.connect(); // 연결을 수행
```

그러나 이렇게하면 더 이상 `getConnection()`을 사용할 수 없습니다. 연결 관리자 인스턴스를 저장하고 `connectionManager.get`을 사용하여 필요한 연결을 가져와야 합니다.

일반적으로 이 방법을 피하고 애플리케이션에서 불필요한 복잡함을 피하고, 정말로 필요하다고 생각하는 경우에만 `ConnectionManager`를 사용하십시오.

## 커넥션과 작업하기

연결을 설정하면 `getConnection` 함수를 사용하여 앱의 어느 곳에서나 사용할 수 있습니다.

```typescript
import {getConnection} from "typeorm";
import {User} from "../entity/User";

export class UserController {

    @Get("/users")
    getAll() {
        return getConnection().manager.find(User);
    }

}
```

`ConnectionManager#get`을 사용하여 연결할 수도 있지만 대부분의 경우 `getConnection()`을 사용하면 충분합니다.

Connection을 사용하면 특히 연결의 `EntityManager` 및 `Repository`를 사용하여 엔티티와 함께 데이터베이스 작업을 실행합니다. 이에 대한 자세한 내용은 [엔티티 관리자 및 리포지토리](working-with-entity-manager.md) 문서를 참조하십시오.

그러나 일반적으로 `Connection`을 많이 사용하지 않습니다. 대부분의 경우 연결 객체를 직접 사용하지 않고 연결을 만들고 `getRepository()` 및 `getManager()`를 사용하여 연결의 관리자 및 저장소에 액세스합니다.

```typescript
import {getManager, getRepository} from "typeorm";
import {User} from "../entity/User";

export class UserController {

    @Get("/users")
    getAll() {
        return getManager().find(User);
    }

    @Get("/users/:id")
    getAll(@Param("id") userId: number) {
        return getRepository(User).findOne(userId);
    }

}
```
