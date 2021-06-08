# 쿼리 작성기를 사용하여 업데이트

`QueryBuilder`를 사용하여 `UPDATE` 쿼리를 만들 수 있습니다.

예:

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .update(User)
    .set({ firstName: "Timber", lastName: "Saw" })
    .where("id = :id", { id: 1 })
    .execute();
```

이는 데이터베이스의 엔티티를 업데이트하는 성능 측면에서 가장 효율적인 방법입니다.

### 원시 SQL 지원

SQL 쿼리를 실행해야하는 경우에는 함수 스타일 값을 사용해야합니다.


```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .update(User)
    .set({
        firstName: "Timber",
        lastName: "Saw",
        age: () => "age + 1"
    })
    .where("id = :id", { id: 1 })
    .execute();
```

이 구문은 값을 이스케이프하지 않으므로 이스케이프를 직접 처리해야합니다.
