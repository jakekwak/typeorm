# 쿼리 캐싱

`QueryBuilder` 메소드 (`getMany`, `getOne`, `getRawMany`, `getRawOne` 및 `getCount`)로 선택한 결과를 캐시할 수 있습니다.

  `find`, `findAndCount`, `findByIds`, `count`와 같은 `Repository` 메소드로 선택한 결과를 캐시할 수도 있습니다.

캐싱을 활성화하려면 연결 옵션에서 명시적으로 활성화해야 합니다.

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: true
}
```

처음으로 캐시를 활성화하는 경우 데이터베이스 스키마를 동기화해야합니다(CLI, 마이그레이션 또는 `synchronize` 연결 옵션 사용).

그런 다음 `QueryBuilder`에서 모든 쿼리에 대해 쿼리 캐시를 활성화할 수 있습니다.

```typescript
const users = await connection
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache(true)
    .getMany();
```

동등한 `Repository` 쿼리:

```typescript
const users = await connection
    .getRepository(User)
    .find({
        where: { isAdmin: true },
        cache: true
    });
```

그러면 모든 관리자를 가져오고 결과를 캐시하는 쿼리가 실행됩니다. 다음에 동일한 코드를 실행하면 캐시에서 모든 관리자를 가져옵니다. 기본 캐시 수명은 '1000ms'와 같습니다. 1초. 이는 쿼리 작성기 코드가 호출된 후 1초 후에 캐시가 유효하지 않음을 의미합니다. 실제로 이는 사용자가 3초 이내에 사용자 페이지를 150번 열면 이 기간 동안 3개의 쿼리만 실행됨을 의미합니다. 1초 캐시 창 동안 삽입된 사용자는 사용자에게 반환되지 않습니다.

`QueryBuilder`를 통해 캐시 시간을 수동으로 변경할 수 있습니다.

```typescript
const users = await connection
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache(60000) // 1 minute
    .getMany();
```

또는 `Repository`를 통해 :

```typescript
const users = await connection
    .getRepository(User)
    .find({
        where: { isAdmin: true },
        cache: 60000
    });
```

또는 연결 옵션에서 전역적으로 :

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        duration: 30000 // 30 seconds
    }
}
```

또한 `QueryBuilder`를 통해 `캐시 ID`를 설정할 수 있습니다.

```typescript
const users = await connection
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache("users_admins", 25000)
    .getMany();
```

또는 `Repository`에서:

```typescript
const users = await connection
    .getRepository(User)
    .find({
        where: { isAdmin: true },
        cache: {
            id: "users_admins",
            milliseconds: 25000
        }
    });
```

이를 통해 캐시를 세부적으로 제어할 수 있습니다.
예를 들어 새 사용자를 삽입할 때 캐시된 결과를 지웁니다.

```typescript
await connection.queryResultCache.remove(["users_admins"]);
```


기본적으로 TypeORM은 `query-result-cache`라는 별도의 테이블을 사용하고 모든 쿼리와 결과를 여기에 저장합니다. 테이블 이름은 구성 가능하므로 tableName 속성에 다른 값을 지정하여 변경할 수 있습니다.

예:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "database",
        tableName: "configurable-table-query-result-cache"
    }
}
```

단일 데이터베이스 테이블에 캐시를 저장하는 것이 효과적이지 않은 경우 캐시 유형을 "redis" 또는 "ioredis"로 변경할 수 있으며 TypeORM은 대신 모든 캐시된 레코드를 redis에 저장합니다.

예:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "redis",
        options: {
            host: "localhost",
            port: 6379
        }
    }
}
```

"옵션"은 사용중인 유형에 따라 [node_redis 특정 옵션](https://github.com/NodeRedis/node_redis#options-object-properties) 또는 [ioredis 특정 옵션](https://github.com/luin/ioredis/blob/master/API.md#new-redisport-host-options)이 될 수 있습니다.


IORedis의 클러스터 기능을 사용하여 redis-cluster에 연결하려는 경우 다음을 수행하여 연결할 수도 있습니다.

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    cache: {
        type: "ioredis/cluster",
        options: {
            startupNodes: [
                {
                    host: 'localhost',
                    port: 7000,
                },
                {
                    host: 'localhost',
                    port: 7001,
                },
                {
                    host: 'localhost',
                    port: 7002,
                }
            ],
            options: {
                scaleReads: 'all',
                clusterRetryStrategy: function (times) { return null },
                redisOptions: {
                    maxRetriesPerRequest: 1
                }
            }
        }
    }
}
```

IORedis 클러스터 생성자의 첫번째 인수로 옵션을 계속 사용할 수 있습니다.

```typescript
{
    ...
    cache: {
        type: "ioredis/cluster",
        options: [
            {
                host: 'localhost',
                port: 7000,
            },
            {
                host: 'localhost',
                port: 7001,
            },
            {
                host: 'localhost',
                port: 7002,
            }
        ]
    },
    ...
}
```

내장된 캐시 공급자가 요구사항을 충족하지 않는 경우 `QueryResultCache` 인터페이스를 구현하는 새 객체를 반환해야하는 `provider` 팩토리 함수를 사용하여 자체 캐시 공급자를 지정할 수도 있습니다.

```typescript
class CustomQueryResultCache implements QueryResultCache {
    constructor(private connection: Connection) {}
    ...
}
```

```typescript
{
    ...
    cache: {
        provider(connection) {
            return new CustomQueryResultCache(connection);
        }
    }
}
```

캐시 오류를 무시하고 캐시 오류가 발생할 경우 쿼리가 데이터베이스로 전달되도록 하려면 ignoreErrors 옵션을 사용할 수 있습니다.
예:

```typescript
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "redis",
        options: {
            host: "localhost",
            port: 6379
        },
        ignoreErrors: true
    }
}
```

`typeorm cache: clear`를 사용하여 캐시에 저장된 모든 것을 지울 수 있습니다.
