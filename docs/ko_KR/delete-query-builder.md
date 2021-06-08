# 쿼리 작성기를 사용하여 삭제

- [쿼리 작성기를 사용하여 삭제](#쿼리-작성기를-사용하여-삭제)
    - [`Delete`](#delete)
    - [`Soft-Delete`](#soft-delete)
    - [`Restore-Soft-Delete`](#restore-soft-delete)

### `Delete`

`QueryBuilder`를 사용하여 `DELETE` 쿼리를 만들 수 있습니다.

예 :

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .delete()
    .from(User)
    .where("id = :id", { id: 1 })
    .execute();
```

데이터베이스에서 항목을 삭제하는 성능 측면에서 가장 효율적인 방법입니다.

---

### `Soft-Delete`

QueryBuilder에 소프트 삭제 적용

```typescript
import {createConnection} from "typeorm";
import {Entity} from "./entity";

createConnection(/*...*/).then(async connection => {

    await connection
      .getRepository(Entity)
      .createQueryBuilder()
      .softDelete()

}).catch(error => console.log(error));
```

### `Restore-Soft-Delete`

또는 `restore()` 메서드를 사용하여 일시 삭제된 행을 복구할 수 있습니다.

```typescript
import {createConnection} from "typeorm";
import {Entity} from "./entity";

createConnection(/*...*/).then(async connection => {

    await connection
      .getRepository(Entity)
      .createQueryBuilder()
      .restore()

}).catch(error => console.log(error));
```
