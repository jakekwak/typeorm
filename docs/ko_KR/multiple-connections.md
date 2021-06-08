# Multiple connections, databases, schemas and replication setup

- [여러 연결 사용](#여러-연결-사용)
- [단일 연결에서 여러 데이터베이스 사용](#단일-연결에서-여러-데이터베이스-사용)
- [단일 연결에서 여러 스키마 사용](#단일-연결에서-여러-스키마-사용)
- [복제](#복제)

## 여러 연결 사용

여러 데이터베이스를 사용하는 가장 간단한 방법은 다른 연결을 만드는 것입니다.

```typescript
import {createConnections} from "typeorm";

const connections = await createConnections([{
    name: "db1Connection",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "db1",
    entities: [__dirname + "/entity/*{.js,.ts}"],
    synchronize: true
}, {
    name: "db2Connection",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "db2",
    entities: [__dirname + "/entity/*{.js,.ts}"],
    synchronize: true
}]);
```

이 접근 방식을 사용하면 보유한 데이터베이스의 수에 관계없이 연결할 수 있으며 각 데이터베이스에는 자체 구성, 자체 엔터티, 전체 ORM 범위 및 설정이 있습니다.

각 연결에 대해 새로운 `Connection` 인스턴스가 생성됩니다. 만드는 각 연결에 대해 고유한 이름을 지정해야 합니다.

연결 옵션은 ormconfig 파일에서 로드할 수도 있습니다. ormconfig 파일에서 모든 연결을 로드할 수 있습니다.

```typescript
import {createConnections} from "typeorm";

const connections = await createConnections();
```

또는 이름으로 만들 연결을 지정할 수 있습니다.

```typescript
import {createConnection} from "typeorm";

const connection = await createConnection("db2Connection");
```

연결 작업을 할 때 특정 연결을 얻으려면 연결 이름을 지정해야합니다.

```typescript
import {getConnection} from "typeorm";

const db1Connection = getConnection("db1Connection");
// 이제 "db1"데이터베이스로 작업 할 수 있습니다.

const db2Connection = getConnection("db2Connection");
// 이제 "db2"데이터베이스로 작업 할 수 있습니다.
```

이 접근 방식을 사용하면 다른 로그인 자격 증명, 호스트, 포트 및 데이터베이스 타입 자체로 여러 연결을 구성할 수 있다는 이점이 있습니다. 단점은 여러 연결 인스턴스를 관리하고 작업해야 한다는 것입니다.

## 단일 연결에서 여러 데이터베이스 사용

여러 연결을 생성하지 않고 단일 연결에서 여러 데이터베이스를 사용하려는 경우 사용하는 엔터티별로 데이터베이스 이름을 지정할 수 있습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity({ database: "secondDB" })
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity({ database: "thirdDB" })
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

}
```

`User` 엔티티는 `secondDB` 데이터베이스 내부에 생성되고 `Photo` 엔티티는 `thirdDB` 데이터베이스 내부에 생성됩니다. 다른 모든 엔티티는 기본 연결 데이터베이스에 생성됩니다.

다른 데이터베이스에서 데이터를 선택하려면 엔터티만 제공하면 됩니다.

```typescript
const users = await connection
    .createQueryBuilder()
    .select()
    .from(User, "user")
    .addFrom(Photo, "photo")
    .andWhere("photo.userId = user.id")
    .getMany(); // userId는 데이터베이스간 요청이므로 외래 키가 아닙니다.
```

This code will produce following sql query (depend on database type):

```sql
SELECT * FROM "secondDB"."user" "user", "thirdDB"."photo" "photo"
    WHERE "photo"."userId" = "user"."id"
```

엔티티 대신 테이블 경로를 지정할 수도 있습니다.

```typescript
const users = await connection
    .createQueryBuilder()
    .select()
    .from("secondDB.user", "user")
    .addFrom("thirdDB.photo", "photo")
    .andWhere("photo.userId = user.id")
    .getMany(); // userId는 데이터베이스간 요청이므로 외래 키가 아닙니다.
```

이 기능은 mysql 및 mssql 데이터베이스에서만 지원됩니다.

## 단일 연결에서 여러 스키마 사용

애플리케이션에서 여러 스키마를 사용할 수 있으며 각 항목에 `schema`를 설정하기 만하면됩니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity({ schema: "secondSchema" })
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity({ schema: "thirdSchema" })
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

}
```

`User` 엔티티는 `secondSchema` 스키마 내에 생성되고 `Photo` 엔티티는 `thirdSchema` 스키마 내에 생성됩니다. 다른 모든 엔터티는 기본 연결 스키마에 생성됩니다.

다른 스키마에서 데이터를 선택하려면 엔터티만 제공하면됩니다.

```typescript
const users = await connection
    .createQueryBuilder()
    .select()
    .from(User, "user")
    .addFrom(Photo, "photo")
    .andWhere("photo.userId = user.id")
    .getMany(); // userId는 데이터베이스간 요청이므로 외래 키가 아닙니다.
```

이 코드는 다음 SQL 쿼리를 생성합니다(데이터베이스 유형에 따라 다름).

```sql
SELECT * FROM "secondSchema"."question" "question", "thirdSchema"."photo" "photo"
    WHERE "photo"."userId" = "user"."id"
```

엔티티 대신 테이블 경로를 지정할 수도 있습니다.

```typescript
const users = await connection
    .createQueryBuilder()
    .select()
    .from("secondSchema.user", "user") // in mssql you can even specify a database: secondDB.secondSchema.user
    .addFrom("thirdSchema.photo", "photo") // in mssql you can even specify a database: thirdDB.thirdSchema.photo
    .andWhere("photo.userId = user.id")
    .getMany();
```

이 기능은 postgres 및 mssql 데이터베이스에서만 지원됩니다. mssql에서 다음과 같이 스키마와 데이터베이스를 결합할 수도 있습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity({ database: "secondDB", schema: "public" })
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

}
```

## 복제

TypeORM을 사용하여 읽기/쓰기 복제를 설정할 수 있습니다. 복제 연결 설정의 예:

```typescript
{
  type: "mysql",
  logging: true,
  replication: {
    master: {
      host: "server1",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    },
    slaves: [{
      host: "server2",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }, {
      host: "server3",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }]
  }
}
```

모든 스키마 업데이트 및 쓰기 작업은 `master` 서버를 사용하여 수행됩니다.
찾기 메소드 또는 선택 쿼리 작성기로 수행되는 모든 단순 쿼리는 임의의 `slave`인스턴스를 사용합니다.
질의 방법으로 수행되는 모든 질의는 `master`인스턴스를 사용하여 수행됩니다.

쿼리 빌더에서 생성한 SELECT에서 명시적으로 master를 사용하려는 경우 다음 코드를 사용할 수 있습니다.

```typescript
const masterQueryRunner = connection.createQueryRunner("master");
try {
    const postsFromMaster = await connection.createQueryBuilder(Post, "post")
        .setQueryRunner(masterQueryRunner)
        .getMany();
} finally {
      await masterQueryRunner.release();
}

```

원시 쿼리에서 `slave`를 사용하려면 쿼리 실행기를 명시적으로 지정해야합니다.

```typescript

const slaveQueryRunner = connection.createQueryRunner("slave");
try {
    const userFromSlave = await slaveQueryRunner.query('SELECT * FROM users WHERE id = $1', [userId], slaveQueryRunner);
} finally {
    return slaveQueryRunner.release();
}
```

`QueryRunner`에 의해 생성된 연결은 명시적으로 해제되어야합니다.

복제는 mysql, postgres 및 sql 서버 데이터베이스에서 지원됩니다.

Mysql은 심층 구성을 지원합니다.

```typescript
{
  replication: {
    master: {
      host: "server1",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    },
    slaves: [{
      host: "server2",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }, {
      host: "server3",
      port: 3306,
      username: "test",
      password: "test",
      database: "test"
    }],

    /**
    * true이면 연결이 실패할 때 PoolCluster가 재 연결을 시도합니다.(기본값: true)
    */
    canRetry: true,

    /**
     * 연결이 실패하면 노드의 errorCount가 증가합니다.
     * errorCount가 removeNodeErrorCount보다 크면 PoolCluster에서 노드를 제거합니다. (기본값 : 5)
     */
    removeNodeErrorCount: 5,

    /**
     * 연결이 실패하면 다른 연결 시도가 이루어지기까지의 시간(밀리 초)을 지정합니다.
     * 0으로 설정하면 노드가 대신 제거되고 다시 사용되지 않습니다. (기본값: 0)
     */
     restoreNodeTimeout: 0,

    /**
      * 슬레이브 선택 방법을 결정합니다.
      * RR: 번갈아 선택(Round-Robin).
      * RANDOM: 랜덤 기능으로 노드를 선택합니다.
      * ORDER: 무조건 사용 가능한 첫번째 노드를 선택합니다.
     */
    selector: "RR"
  }
}
```
