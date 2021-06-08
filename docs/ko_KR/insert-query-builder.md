# 쿼리 작성기를 사용하여 삽입

`QueryBuilder`를 사용하여 `INSERT` 쿼리를 만들 수 있습니다.

예:

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .insert()
    .into(User)
    .values([
        { firstName: "Timber", lastName: "Saw" },
        { firstName: "Phantom", lastName: "Lancer" }
     ])
    .execute();
```

이것은 데이터베이스에 행을 삽입하는 성능 측면에서 가장 효율적인 방법입니다. 이 방법으로 대량 삽입을 수행할 수도 있습니다.

### 원시 SQL 지원

SQL 쿼리를 실행해야 하는 경우에는 함수 스타일 값을 사용해야합니다.


```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .insert()
    .into(User)
    .values({
        firstName: "Timber",
        lastName: () => "CONCAT('S', 'A', 'W')"
    })
    .execute();
```

이 구문은 값을 이스케이프하지 않으므로 이스케이프를 직접 처리해야합니다.
