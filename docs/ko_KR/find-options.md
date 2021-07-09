# Find 옵션

- [기본 옵션](#기본-옵션)
- [고급 옵션](#고급-옵션)
- [고급 옵션 결합](#고급-옵션-결합)

## 기본 옵션

모든 저장소 및 관리자 `find` 메소드는 `QueryBuilder`를 사용하지 않고 필요한 데이터를 쿼리하는데 사용할 수 있는 특수 옵션을 허용합니다.

* `select` - 선택해야하는 주 개체의 속성을 나타냅니다.

```typescript
userRepository.find({ select: ["firstName", "lastName"] });
```

* `relations` - 관계는 기본 엔티티와 함께 로드되어야 합니다. 하위 관계도 로드할 수 있습니다 (join 및 leftJoinAndSelect의 약어).

```typescript
userRepository.find({ relations: ["profile", "photos", "videos"] });
userRepository.find({ relations: ["profile", "photos", "videos", "videos.video_attributes"] });
```

* `join` - 엔터티에 대해 조인을 수행해야합니다. "관계"의 확장 버전.

```typescript
userRepository.find({
    join: {
        alias: "user",
        leftJoinAndSelect: {
            profile: "user.profile",
            photo: "user.photos",
            video: "user.videos"
        }
    }
});
```

* `where` - 엔터티를 쿼리해야하는 간단한 조건.

```typescript
userRepository.find({ where: { firstName: "Timber", lastName: "Saw" } });
```
포함된 엔터티에서 컬럼을 쿼리하는 것은 정의된 계층 구조와 관련하여 수행되어야합니다. 예:

```typescript
userRepository.find({ where: { name: { first: "Timber", last: "Saw" } } });
```

OR 연산자로 쿼리:

```typescript
userRepository.find({
  where: [
    { firstName: "Timber", lastName: "Saw" },
    { firstName: "Stan", lastName: "Lee" }
  ]
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "user" WHERE ("firstName" = 'Timber' AND "lastName" = 'Saw') OR ("firstName" = 'Stan' AND "lastName" = 'Lee')
```

* `order` - 선택 순서.

```typescript
userRepository.find({
    order: {
        name: "ASC",
        id: "DESC"
    }
});
```

* `withDeleted` - `softDelete` 또는 `softRemove`로 소프트 삭제된 항목을 포함합니다; 예: `@DeleteDateColumn` 컬럼이 설정되어 있습니다. 기본적으로 일시 삭제된 엔티티는 포함되지 않습니다.

```typescript
userRepository.find({
    withDeleted: true
});
```

`find` 여러 항목을 반환하는 메서드(`find`, `findAndCount`, `findByIds`)도 다음 옵션을 허용합니다.

* `skip` - 엔티티를 가져와야하는 오프셋(페이지 매김).

```typescript
userRepository.find({
    skip: 5
});
```

* `take` - 리미트(페이지 매김) - 가져와야하는 최대 엔티티 수입니다.

```typescript
userRepository.find({
    take: 10
});
```

** MSSQL에서 typeorm을 사용 중이고 `take` 또는 `limit`를 사용하려면 `order`도 사용해야합니다. 그렇지 않으면 다음 오류가 표시됩니다. `'FETCH 문에서 NEXT 옵션을 잘못 사용했습니다.'`

```typescript
userRepository.find({
    order: {
        columnName: 'ASC'
        },
    skip: 0,
    take: 10
})
```



* `cache` - 쿼리 결과 캐싱을 활성화하거나 비활성화합니다. 자세한 정보와 옵션은 [캐싱](./caching.md)을 참조하십시오.

```typescript
userRepository.find({
    cache: true
})
```

* `lock` - 쿼리에 대한 잠금 메커니즘을 사용합니다. `findOne` 메서드에서만 사용할 수 있습니다. `lock`은 다음과 같이 정의할 수 있는 객체입니다.

```ts
{ mode: "optimistic", version: number|Date }
```
또는

```ts
{ mode: "pessimistic_read"|"pessimistic_write"|"dirty_read"|"pessimistic_partial_write"|"pessimistic_write_or_fail"|"for_no_key_update" }
```

예:

```typescript
userRepository.findOne(1, {
    lock: { mode: "optimistic", version: 1 }
})
```

잠금모드 지원 및 변환되는 SQL 문은 아래표에 나열되어 있습니다(빈 셀은 지원되지 않음을 나타냄). 지정된 잠금모드가 지원되지 않는 경우 `LockNotSupportedOnGivenDriverError` 오류가 발생합니다.

```text
|                 | pessimistic_read         | pessimistic_write       | dirty_read    | pessimistic_partial_write   | pessimistic_write_or_fail   | for_no_key_update   |
| --------------- | --------------------     | ----------------------- | ------------- | --------------------------- | --------------------------- | ------------------- |
| MySQL           | LOCK IN SHARE MODE       | FOR UPDATE              | (nothing)     | FOR UPDATE SKIP LOCKED      | FOR UPDATE NOWAIT           |                     |
| Postgres        | FOR SHARE                | FOR UPDATE              | (nothing)     | FOR UPDATE SKIP LOCKED      | FOR UPDATE NOWAIT           | FOR NO KEY UPDATE   |
| Oracle          | FOR UPDATE               | FOR UPDATE              | (nothing)     |                             |                             |                     |
| SQL Server      | WITH (HOLDLOCK, ROWLOCK) | WITH (UPDLOCK, ROWLOCK) | WITH (NOLOCK) |                             |                             |                     |
| AuroraDataApi   | LOCK IN SHARE MODE       | FOR UPDATE              | (nothing)     |                             |                             |                     |

```

찾기 옵션의 전체 예:

```typescript
userRepository.find({
    select: ["firstName", "lastName"],
    relations: ["profile", "photos", "videos"],
    where: {
        firstName: "Timber",
        lastName: "Saw"
    },
    order: {
        name: "ASC",
        id: "DESC"
    },
    skip: 5,
    take: 10,
    cache: true
});
```


## 고급 옵션

TypeORM은 더 복잡한 비교를 생성하는데 사용할 수 있는 많은 내장 연산자를 제공합니다.

* `Not`

```ts
import {Not} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: Not("About #1")
})
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "title" != 'About #1'
```

* `LessThan`

```ts
import {LessThan} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: LessThan(10)
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "likes" < 10
```

* `LessThanOrEqual`

```ts
import {LessThanOrEqual} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: LessThanOrEqual(10)
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "likes" <= 10
```

* `MoreThan`

```ts
import {MoreThan} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: MoreThan(10)
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "likes" > 10
```

* `MoreThanOrEqual`

```ts
import {MoreThanOrEqual} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: MoreThanOrEqual(10)
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "likes" >= 10
```

* `Equal`

```ts
import {Equal} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: Equal("About #2")
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "title" = 'About #2'
```

* `Like`

```ts
import {Like} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: Like("%out #%")
});
```
다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "title" LIKE '%out #%'
```

* `ILike`

```ts
import {ILike} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: ILike("%out #%")
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "title" ILIKE '%out #%'
```

* `Between`

```ts
import {Between} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: Between(1, 10)
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "likes" BETWEEN 1 AND 10
```

* `In`

```ts
import {In} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: In(["About #2", "About #3"])
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "title" IN ('About #2','About #3')
```

* `Any`

```ts
import {Any} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: Any(["About #2", "About #3"])
});
```

다음 쿼리를 실행합니다. (Postgres 표기법):

```sql
SELECT * FROM "post" WHERE "title" = ANY(['About #2','About #3'])
```

* `IsNull`

```ts
import {IsNull} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: IsNull()
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "title" IS NULL
```

* `Raw`

```ts
import {Raw} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: Raw("dislikes - 4")
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "likes" = "dislikes" - 4
```

가장 간단한 경우에는 등호기호 바로 뒤에 원시 쿼리가 삽입됩니다. 그러나 함수를 사용하여 비교 논리를 완전히 다시 작성할 수도 있습니다.

```ts
import {Raw} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    currentDate: Raw(alias =>`${alias} > NOW()`)
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "currentDate" > NOW()
```

사용자 입력을 제공해야 하는 경우 SQL 주입 취약점을 생성할 수 있으므로 사용자 입력을 쿼리에 직접 포함해서는 안됩니다. 대신 `Raw` 함수의 두번째 인수를 사용하여 쿼리에 바인딩할 매개변수 목록을 제공할 수 있습니다.

```ts
import {Raw} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    currentDate: Raw(alias =>`${alias} > :date`, { date: "2020-10-06" })
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "currentDate" > '2020-10-06'
```

배열인 사용자 입력을 제공해야하는 경우 특수 표현식 구문을 사용하여 SQL 문의 값 목록으로 바인딩할 수 있습니다.

```ts
import {Raw} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    title: Raw(alias =>`${alias} IN (:...titles)`, { titles: ["Go To Statement Considered Harmful", "Structured Programming"] })
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE "titles" IN ('Go To Statement Considered Harmful', 'Structured Programming')
```

## 고급 옵션 결합

또한 이러한 연산자를 `Not` 연산자와 결합할 수 있습니다.

```ts
import {Not, MoreThan, Equal} from "typeorm";

const loadedPosts = await connection.getRepository(Post).find({
    likes: Not(MoreThan(10)),
    title: Not(Equal("About #2"))
});
```

다음 쿼리를 실행합니다.

```sql
SELECT * FROM "post" WHERE NOT("likes" > 10) AND NOT("title" = 'About #2')
```
