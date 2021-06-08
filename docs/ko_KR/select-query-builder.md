# 쿼리 작성기를 사용하여 선택

* [What is `QueryBuilder`](#what-is-querybuilder)
* [Important note when using the `QueryBuilder`](#important-note-when-using-the-querybuilder)
* [How to create and use a `QueryBuilder`](#how-to-create-and-use-a-querybuilder)
* [Getting values using QueryBuilder](#getting-values-using-querybuilder)
* [What are aliases for?](#what-are-aliases-for)
* [Using parameters to escape data](#using-parameters-to-escape-data)
* [Adding `WHERE` expression](#adding-where-expression)
* [Adding `HAVING` expression](#adding-having-expression)
* [Adding `ORDER BY` expression](#adding-order-by-expression)
* [Adding `GROUP BY` expression](#adding-group-by-expression)
* [Adding `LIMIT` expression](#adding-limit-expression)
* [Adding `OFFSET` expression](#adding-offset-expression)
* [Joining relations](#joining-relations)
* [Inner and left joins](#inner-and-left-joins)
* [Join without selection](#join-without-selection)
* [Joining any entity or table](#joining-any-entity-or-table)
* [Joining and mapping functionality](#joining-and-mapping-functionality)
* [Getting the generated query](#getting-the-generated-query)
* [Getting raw results](#getting-raw-results)
* [Streaming result data](#streaming-result-data)
* [Using pagination](#using-pagination)
* [Set locking](#set-locking)
* [Max execution time](#max-execution-time)
* [Partial selection](#partial-selection)
* [Using subqueries](#using-subqueries)
* [Hidden Columns](#hidden-columns)

## `QueryBuilder`란?

`QueryBuilder`는 TypeORM의 가장 강력한 기능중 하나입니다. 우아하고 편리한 구문을 사용하여 SQL 쿼리를 작성하고 실행하고 자동으로 변환된 엔티티를 얻을 수 있습니다.

`QueryBuilder`의 간단한 예:

```typescript
const firstUser = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne();
```

다음 SQL 쿼리를 작성합니다.

```sql
SELECT
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1
```

그리고 `User`의 인스턴스를 반환합니다.

```
User {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}
```

## `QueryBuilder` 사용시 중요 사항

`QueryBuilder`를 사용할 때 `WHERE` 표현식에 고유한 매개변수를 제공해야 합니다. **작동하지 않습니다** :

```TypeScript
const result = await getConnection()
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
    .leftJoinAndSelect('user.linkedCow', 'linkedCow')
    .where('user.linkedSheep = :id', { id: sheepId })
    .andWhere('user.linkedCow = :id', { id: cowId });
```

... 그러나 이것은:

```TypeScript
const result = await getConnection()
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
    .leftJoinAndSelect('user.linkedCow', 'linkedCow')
    .where('user.linkedSheep = :sheepId', { sheepId })
    .andWhere('user.linkedCow = :cowId', { cowId });
```

다른 매개 변수에 대해 `:id`를 두번 사용하는 대신 `:sheepId` 및 `:cowId`라는 이름을 고유하게 지정했습니다.

## `QueryBuilder` 생성 및 사용 방법

`Query Builder`를 만드는 방법에는 여러 가지가 있습니다.

* 연결 사용:

    ```typescript
    import {getConnection} from "typeorm";

    const user = await getConnection()
        .createQueryBuilder()
        .select("user")
        .from(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

* 엔티티 관리자 사용:

    ```typescript
    import {getManager} from "typeorm";

    const user = await getManager()
        .createQueryBuilder(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

* 리포지토리 사용:

    ```typescript
    import {getRepository} from "typeorm";

    const user = await getRepository(User)
        .createQueryBuilder("user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

사용 가능한 5가지 `QueryBuilder` 타입이 있습니다.

* `SelectQueryBuilder` - `SELECT` 쿼리를 작성하고 실행하는데 사용됩니다. 예:

    ```typescript
    import {getConnection} from "typeorm";

    const user = await getConnection()
        .createQueryBuilder()
        .select("user")
        .from(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

* `InsertQueryBuilder` - `INSERT` 쿼리를 작성하고 실행하는데 사용됩니다. 예:

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

* `UpdateQueryBuilder` - `UPDATE` 쿼리를 작성하고 실행하는데 사용됩니다. 예:

    ```typescript
    import {getConnection} from "typeorm";

    await getConnection()
        .createQueryBuilder()
        .update(User)
        .set({ firstName: "Timber", lastName: "Saw" })
        .where("id = :id", { id: 1 })
        .execute();
    ```
* `DeleteQueryBuilder` - `DELETE` 쿼리를 작성하고 실행하는데 사용됩니다. 예:

    ```typescript
    import {getConnection} from "typeorm";

    await getConnection()
        .createQueryBuilder()
        .delete()
        .from(User)
        .where("id = :id", { id: 1 })
        .execute();
    ```

* `RelationQueryBuilder` - 관계별 작업을 구축하고 실행하는 데 사용됩니다 [TBD].

다른 유형의 쿼리 작성기간에 전환할 수 있습니다. 그러면 다른 모든 방법과 달리 쿼리 작성기의 새 인스턴스가 생성됩니다.

## `QueryBuilder`를 사용하여 값 가져 오기

데이터베이스에서 단일 결과를 가져 오려면(예: ID 또는 이름으로 사용자 가져 오기) `getOne`을 사용해야합니다.

```typescript
const timber = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
    .getOne();
```

`getOneOrFail`은 데이터베이스에서 단일 결과를 가져 오지만 결과가 없으면 `EntityNotFoundError`가 발생합니다.

```typescript
const timber = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
    .getOneOrFail();
```

예를 들어 데이터베이스에서 여러 결과를 얻으려면 데이터베이스에서 모든 사용자를 가져 오려면 `getMany`를 사용하십시오.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .getMany();
```

선택 쿼리 작성기를 사용하여 얻을 수 있는 결과에는 **항목** 또는 **원시 결과**의 두가지 타입이 있습니다. 대부분의 경우 데이터베이스에서 실제 엔티티(예: 사용자)를 선택해야합니다. 이를 위해 `getOne`과 `getMany`를 사용합니다. 그러나 때로는 특정 데이터를 선택해야 할 때가 있습니다. *모든 사용자 사진의 합계*를 가정해 보겠습니다. 이 데이터는 엔티티가 아니라 원시 데이터라고 합니다. 원시 데이터를 얻으려면 `getRawOne` 및 `getRawMany`를 사용합니다.

예:

```typescript
const { sum } = await getRepository(User)
    .createQueryBuilder("user")
    .select("SUM(user.photosCount)", "sum")
    .where("user.id = :id", { id: 1 })
    .getRawOne();
```

```typescript
const photosSums = await getRepository(User)
    .createQueryBuilder("user")
    .select("user.id")
    .addSelect("SUM(user.photosCount)", "sum")
    .groupBy("user.id")
    .getRawMany();

// 결과는 다음과 같습니다: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
```

## 별칭은 무엇입니까?

`createQueryBuilder("user")`를 사용했습니다. 그러나 "사용자"는 무엇입니까? 일반 SQL 별칭입니다. 선택한 데이터로 작업하는 경우를 제외하고 모든 곳에서 별칭을 사용합니다.

`createQueryBuilder("user")`는 다음과 같습니다.

```typescript
createQueryBuilder()
    .select("user")
    .from(User, "user")
```

그러면 다음과 같은 SQL 쿼리가 생성됩니다.

```sql
SELECT ... FROM users user
```

이 SQL 쿼리에서 `users`는 테이블 이름이고 `user`는 이 테이블에 할당한 별칭입니다. 나중에 이 별칭을 사용하여 테이블에 액세스합니다.

```typescript
createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.name = :name", { name: "Timber" })
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user WHERE user.name = 'Timber'
```

쿼리 작성기를 만들 때 할당한 `user` 별칭을 사용하여 users 테이블을 사용했습니다.

하나의 쿼리 작성기는 하나의 별칭으로 제한되지 않으며 여러 별칭을 가질 수 있습니다. 각 선택에는 고유한 별칭이 있을 수 있으며, 각각 고유한 별칭이 있는 여러 테이블에서 선택할 수 있으며, 각각 고유한 별칭을 사용하여 여러 테이블을 조인할 수 있습니다. 이러한 별칭을 사용하여 선택한 테이블(또는 선택한 데이터)에 액세스할 수 있습니다.

## 매개변수를 사용하여 데이터 이스케이프

`where("user.name = :name", { name: "Timber" })`를 사용했습니다. `{ name: "Timber" }`는 무엇을 의미합니까? SQL 주입을 방지하기 위해 사용한 매개변수입니다. `where("user.name = '" + name + "')`라고 쓸 수 있었지만 SQL 주입에 대한 코드를 열기 때문에 안전하지 않습니다. 안전한 방법은 다음과 같은 특수 구문을 사용하는 것입니다. `where("user.name = :name", { name: "Timber" })`, 여기서 `:name`은 매개 변수 이름이고 값은 객체에 지정됩니다: `{name : "Timber"}`.

```typescript
.where("user.name = :name", { name: "Timber" })
```

다음에 대한 바로 가기입니다.

```typescript
.where("user.name = :name")
.setParameter("name", "Timber")
```

참고: 쿼리 작성기에서 다른 값에 대해 동일한 매개변수 이름을 사용하지 마십시오. 값을 여러번 설정하면 값이 무시됩니다.

특수 확장구문을 사용하여 값 배열을 제공하고 SQL 문에서 값 목록으로 변환되도록 할 수도 있습니다.

```typescript
.where("user.name IN (:...names)", { names: [ "Timber", "Cristal", "Lina" ] })
```

다음과 같이됩니다.

```sql
WHERE user.name IN ('Timber', 'Cristal', 'Lina')
```

## `WHERE` 표현식 추가

`WHERE` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .where("user.name = :name", { name: "Timber" })
```

다음을 생성합니다.

```sql
SELECT ... FROM users user WHERE user.name = 'Timber'
```

기존 `WHERE` 표현식에 `AND`를 추가할 수 있습니다.

```typescript
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .andWhere("user.lastName = :lastName", { lastName: "Saw" });
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user WHERE user.firstName = 'Timber' AND user.lastName = 'Saw'
```

기존 `WHERE` 표현식에 `OR`을 추가할 수 있습니다.

```typescript
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" });
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user WHERE user.firstName = 'Timber' OR user.lastName = 'Saw'
```

`WHERE` 표현식을 사용하여 `IN` 쿼리를 수행할 수 있습니다.

```typescript
createQueryBuilder("user")
    .where("user.id IN (:...ids)", { ids: [1, 2, 3, 4] })
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user WHERE user.id IN (1, 2, 3, 4)
```


`Brackets`를 사용하여 복잡한 `WHERE` 표현식을 기존 `WHERE`에 추가할 수 있습니다.

```typescript
createQueryBuilder("user")
    .where("user.registered = :registered", { registered: true })
    .andWhere(new Brackets(qb => {
        qb.where("user.firstName = :firstName", { firstName: "Timber" })
          .orWhere("user.lastName = :lastName", { lastName: "Saw" })
    }))
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user WHERE user.registered = true AND (user.firstName = 'Timber' OR user.lastName = 'Saw')
```

필요한만큼 `AND` 및 `OR` 표현식을 결합할 수 있습니다. `.where`를 두번 이상 사용하면 이전 `WHERE` 표현식이 모두 재정의됩니다.

참고: `orWhere`에 주의하세요. `AND` 및 `OR` 표현식과 함께 복잡한 표현식을 사용하는 경우, 허위없이 누적된다는 점에 유의하세요. 때로는 where 문자열을 대신 생성하고 `orWhere`를 사용하지 않아야합니다.

## `HAVING` 표현식 추가

`HAVING` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .having("user.name = :name", { name: "Timber" })
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user HAVING user.name = 'Timber'
```

기존 `HAVING` 표현식에 `AND`를 추가할 수 있습니다.

```typescript
createQueryBuilder("user")
    .having("user.firstName = :firstName", { firstName: "Timber" })
    .andHaving("user.lastName = :lastName", { lastName: "Saw" });
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' AND user.lastName = 'Saw'
```

기존 `HAVING` 표현식에 `OR`을 추가 할 수 있습니다.

```typescript
createQueryBuilder("user")
    .having("user.firstName = :firstName", { firstName: "Timber" })
    .orHaving("user.lastName = :lastName", { lastName: "Saw" });
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' OR user.lastName = 'Saw'
```

필요한만큼 `AND` 및 `OR` 표현식을 결합할 수 있습니다.
`.having`을 두번 이상 사용하면 이전 `HAVING` 표현식이 모두 재정의됩니다.

## `ORDER BY` 표현식 추가

`ORDER BY` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .orderBy("user.id")
```

다음을 생성합니다.

```sql
SELECT ... FROM users user ORDER BY user.id
```

순서 방향을 오름차순에서 내림차순(또는 반대)으로 변경할 수 있습니다.

```typescript
createQueryBuilder("user")
    .orderBy("user.id", "DESC")

createQueryBuilder("user")
    .orderBy("user.id", "ASC")
```

여러 `order-by` 기준을 추가할 수 있습니다.

```typescript
createQueryBuilder("user")
    .orderBy("user.name")
    .addOrderBy("user.id");
```

order-by 필드 맵을 사용할 수도 있습니다.

```typescript
createQueryBuilder("user")
    .orderBy({
        "user.name": "ASC",
        "user.id": "DESC"
    });
```

`.orderBy`를 두번 이상 사용하면 이전의 모든 `ORDER BY` 표현식이 재정의됩니다.

## `DISTINCT ON` 표현식 추가(Postgres만 해당)

`order-by` 표현식과 함께 `distinct-on`을 모두 사용할 때 `distinct-on` 표현식은 가장 왼쪽의 `order-by`와 일치해야 합니다.
구별식은 `order-by`와 동일한 규칙을 사용하여 해석됩니다. `order-by` 표현식없이 `distinct-on`을 사용하면 각 세트의 첫번째 행을 예측할 수 없습니다.

`DISTINCT ON` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .distinctOn(["user.id"])
    .orderBy("user.id")
```

다음을 생성합니다.

```sql
SELECT DISTINCT ON (user.id) ... FROM users user ORDER BY user.id
```

## `GROUP BY` 표현식 추가

`GROUP BY` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .groupBy("user.id")
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user GROUP BY user.id
```

더 많은 그룹별 기준을 추가하려면 `addGroupBy`를 사용하십시오.

```typescript
createQueryBuilder("user")
    .groupBy("user.name")
    .addGroupBy("user.id");
```

`.groupBy`를 두번 이상 사용하면 이전 `GROUP BY` 표현식이 모두 재정의됩니다.

## `LIMIT` 표현식 추가

`LIMIT` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .limit(10)
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user LIMIT 10
```

결과 SQL 쿼리는 데이터베이스 타입(SQL, mySQL, Postgres 등)에 따라 다릅니다. 참고: 조인 또는 하위 쿼리와 함께 복잡한 쿼리를 사용하는 경우 `LIMIT`가 예상대로 작동하지 않을 수 있습니다. 페이지 매김을 사용하는 경우 대신 `take`를 사용하는 것이 좋습니다.

## `OFFSET` 표현식 추가

SQL `OFFSET` 표현식을 추가하는 것은 다음과 같이 쉽습니다.

```typescript
createQueryBuilder("user")
    .offset(10)
```

다음 SQL 쿼리를 생성합니다.

```sql
SELECT ... FROM users user OFFSET 10
```

결과 SQL 쿼리는 데이터베이스 유형(SQL, mySQL, Postgres 등)에 따라 다릅니다.
참고: 조인 또는 하위 쿼리와 함께 복잡한 쿼리를 사용하는 경우 OFFSET이 예상대로 작동하지 않을 수 있습니다.
페이지 매김을 사용하는 경우 대신 `skip`를 사용하는 것이 좋습니다.

## 조인 관계

다음 항목이 있다고 가정해 보겠습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, OneToMany} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(type => Photo, photo => photo.user)
    photos: Photo[];
}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToOne} from "typeorm";
import {User} from "./User";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

    @ManyToOne(type => User, user => user.photos)
    user: User;
}
```

이제 "Timber"사용자의 모든 사진을 로드하려고 한다고 가정해 보겠습니다.

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

다음과 같은 결과를 얻을 수 있습니다.

```typescript
{
    id: 1,
    name: "Timber",
    photos: [{
        id: 1,
        url: "me-with-chakram.jpg"
    }, {
        id: 2,
        url: "me-with-trees.jpg"
    }]
}
```

보시다시피 `leftJoinAndSelect`는 자동으로 Timber의 모든 사진을 로드했습니다. 첫번째 인수는 로드하려는 관계이고 두번째 인수는 이 관계의 테이블에 할당하는 별칭입니다. 이 별칭은 쿼리 작성기의 어느 곳에서나 사용할 수 있습니다. 예를 들어, 제거되지 않은 모든 Timber 사진을 찍어 보겠습니다.

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .andWhere("photo.isRemoved = :isRemoved", { isRemoved: false })
    .getOne();
```

그러면 다음 SQL 쿼리가 생성됩니다.

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber' AND photo.isRemoved = FALSE
```

"where"를 사용하는 대신 조인 표현식에 조건을 추가할 수도 있습니다.

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

그러면 다음 SQL 쿼리가 생성됩니다.

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

## 이너 및 레프트 조인

`LEFT JOIN`대신 `INNER JOIN`을 사용하려면 대신 `innerJoinAndSelect`를 사용하세요.

```typescript
const user = await createQueryBuilder("user")
    .innerJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

그러면 다음이 생성됩니다.

```sql
SELECT user.*, photo.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

`LEFT JOIN`과 `INNER JOIN`의 차이점은 `INNER JOIN`이 사진이 없는 경우 사용자를 반환하지 않는다는 것입니다. `LEFT JOIN`은 사진이 없더라도 사용자를 반환합니다. 다양한 조인 타입에 대한 자세한 내용은 [SQL 문서](https://msdn.microsoft.com/en-us/library/zt8wzxy4.aspx)를 참조하세요.

## 선택하지 않고 조인

선택하지 않고 데이터를 결합할 수 있습니다.
이를 위해 `left Join` 또는 `inner Join`을 사용하십시오.

```typescript
const user = await createQueryBuilder("user")
    .innerJoin("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

그러면 다음이 생성됩니다.

```sql
SELECT user.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber'
```

사진이 있으면 Timber를 선택하지만 사진은 반환하지 않습니다.

## 엔티티 또는 테이블 조인

관계뿐만 아니라 관련되지 않은 다른 항목이나 테이블도 조인할 수 있습니다.

예:

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect(Photo, "photo", "photo.userId = user.id")
    .getMany();
```

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("photos", "photo", "photo.userId = user.id")
    .getMany();
```

## 결합 및 매핑 기능

`User` 엔터티에 `profile Photo`를 추가하고 `Query Builder`를 사용하여 모든 데이터를 해당 속성에 매핑할 수 있습니다.

```typescript
export class User {
    /// ...
    profilePhoto: Photo;

}
```

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndMapOne("user.profilePhoto", "user.photos", "photo", "photo.isForProfile = TRUE")
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

그러면 Timber의 프로필 사진이 로드되고 `user.profilePhoto`로 설정됩니다.
단일 엔티티를 로드하고 매핑하려면 `leftJoinAndMapOne`을 사용하십시오.
여러 엔티티를 로드하고 매핑하려면 `leftJoinAndMapMany`를 사용하십시오.

## 생성된 쿼리 가져 오기

때때로 `QueryBuilder`에 의해 생성된 SQL 쿼리를 얻고 싶을 수 있습니다.
이렇게 하려면 `getSql`을 사용하십시오.

```typescript
const sql = createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" })
    .getSql();
```

디버깅 목적으로 `printSql`을 사용할 수 있습니다.

```typescript
const users = await createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" })
    .printSql()
    .getMany();
```

이 쿼리는 사용자를 반환하고 사용된 SQL 문을 콘솔에 인쇄합니다.

## 원시 결과 얻기

선택 쿼리 작성기를 사용하여 얻을 수 있는 결과에는 **항목** 및 **원시 결과**의 두가지 유형이 있습니다. 대부분의 경우 데이터베이스에서 실제 엔티티(예: 사용자)를 선택해야 합니다. 이를 위해 `getOne`과 `getMany`를 사용합니다. 그러나 때때로 *모든 사용자 사진의 합계*와 같은 특정 데이터를 선택해야 합니다. 이러한 데이터는 엔티티가 아니라 원시 데이터라고 합니다. 원시 데이터를 얻으려면 `getRawOne` 및 `getRawMany`를 사용합니다.

예 :

```typescript
const { sum } = await getRepository(User)
    .createQueryBuilder("user")
    .select("SUM(user.photosCount)", "sum")
    .where("user.id = :id", { id: 1 })
    .getRawOne();
```

```typescript
const photosSums = await getRepository(User)
    .createQueryBuilder("user")
    .select("user.id")
    .addSelect("SUM(user.photosCount)", "sum")
    .groupBy("user.id")
    .getRawMany();

// 결과는 다음과 같습니다: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
```

## 스트리밍 결과 데이터

스트림을 반환하는 `stream`을 사용할 수 있습니다.
스트리밍은 원시 데이터를 반환하며 엔터티 변환을 수동으로 처리해야합니다.

```typescript
const stream = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .stream();
```

## 페이지 매김 사용

대부분의 경우 애플리케이션을 개발할 때 페이지 매김 기능이 필요합니다.
애플리케이션에 페이지 매김, 페이지 슬라이더 또는 무한 스크롤 구성 요소가 있는 경우 사용됩니다.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .take(10)
    .getMany();
```

그러면 처음 10명의 사용자에게 사진이 제공됩니다.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(10)
    .getMany();
```

이렇게하면 처음 10명의 사용자를 제외한 모든 사진이 제공됩니다.
이러한 방법을 결합할 수 있습니다.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(5)
    .take(10)
    .getMany();
```

이렇게하면 처음 5명의 사용자를 건너 뛰고 10명의 사용자가 추가됩니다.


`take`와 `skip`는 `limit`와 `offset`을 사용하는 것처럼 보일 수 있지만 그렇지 않습니다. 조인 또는 하위 쿼리가 있는 더 복잡한 쿼리가 있으면 `limit` 및 `offset`이 예상대로 작동하지 않을 수 있습니다. `take`와 `skip`을 사용하면 이러한 문제를 방지할 수 있습니다.

## 잠금 설정

QueryBuilder는 낙관적 잠금과 비관적 잠금을 모두 지원합니다.
비관적 읽기 잠금을 사용하려면 다음 방법을 사용하십시오.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("pessimistic_read")
    .getMany();
```

비관적 쓰기 잠금을 사용하려면 다음 방법을 사용하십시오.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("pessimistic_write")
    .getMany();
```

더티 읽기 잠금을 사용하려면 다음 방법을 사용하십시오.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("dirty_read")
    .getMany();
```

낙관적 잠금을 사용하려면 다음 방법을 사용하십시오.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("optimistic", existUser.version)
    .getMany();
```

낙관적 잠금은 `@Version` 및 `@UpdatedDate` 데코레이터와 함께 작동합니다.

## 최대 실행 시간

서버 충돌을 방지하기 위해 느린 쿼리를 삭제할 수 있습니다. 현재 MySQL 드라이버만 지원됩니다.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .maxExecutionTime(1000) // milliseconds.
    .getMany();
```

## 부분 선택

일부 엔터티 속성만 선택하려는 경우 다음 구문을 사용할 수 있습니다.

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .select([
        "user.id",
        "user.name"
    ])
    .getMany();
```

이것은 `user`의 `id`와 `name` 만 선택합니다.

## 하위 쿼리 사용

하위 쿼리를 쉽게 만들 수 있습니다. 하위 쿼리는 `FROM`, `WHERE` 및 `JOIN` 표현식에서 지원됩니다.

예:

```typescript
const qb = await getRepository(Post).createQueryBuilder("post");
const posts = qb
    .where("post.title IN " + qb.subQuery().select("user.name").from(User, "user").where("user.registered = :registered").getQuery())
    .setParameter("registered", true)
    .getMany();
```

동일한 작업을 수행하는 더 우아한 방법:

```typescript
const posts = await connection.getRepository(Post)
    .createQueryBuilder("post")
    .where(qb => {
        const subQuery = qb.subQuery()
            .select("user.name")
            .from(User, "user")
            .where("user.registered = :registered")
            .getQuery();
        return "post.title IN " + subQuery;
    })
    .setParameter("registered", true)
    .getMany();
```

또는 별도의 쿼리 작성기를 만들고 생성된 SQL을 사용할 수 있습니다.

```typescript
const userQb = await connection.getRepository(User)
    .createQueryBuilder("user")
    .select("user.name")
    .where("user.registered = :registered", { registered: true });

const posts = await connection.getRepository(Post)
    .createQueryBuilder("post")
    .where("post.title IN (" + userQb.getQuery() + ")")
    .setParameters(userQb.getParameters())
    .getMany();
```

다음과 같이 `FROM`에 하위 쿼리를 만들 수 있습니다.

```typescript
const userQb = await connection.getRepository(User)
    .createQueryBuilder("user")
    .select("user.name", "name")
    .where("user.registered = :registered", { registered: true });

const posts = await connection
    .createQueryBuilder()
    .select("user.name", "name")
    .from("(" + userQb.getQuery() + ")", "user")
    .setParameters(userQb.getParameters())
    .getRawMany();
```

또는 더 우아한 구문 사용:

```typescript
const posts = await connection
    .createQueryBuilder()
    .select("user.name", "name")
    .from(subQuery => {
        return subQuery
            .select("user.name", "name")
            .from(User, "user")
            .where("user.registered = :registered", { registered: true });
    }, "user")
    .getRawMany();
```

하위 선택을 "두번째 출처"로 추가하려면 `addFrom`을 사용하세요.

`SELECT` 문에서도 하위 선택을 사용할 수 있습니다.

```typescript
const posts = await connection
    .createQueryBuilder()
    .select("post.id", "id")
    .addSelect(subQuery => {
        return subQuery
            .select("user.name", "name")
            .from(User, "user")
            .limit(1);
    }, "name")
    .from(Post, "post")
    .getRawMany();
```
## 숨겨진 컬럼

쿼리중인 모델에 `select: false` 컬럼이 가지고있는 컬럼이 있는 경우 컬럼에서 정보를 검색하려면 `addSelect` 함수를 사용해야합니다.

다음 엔티티가 있다고 가정해 보겠습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column({select: false})
    password: string;
}
```

표준 `find` 또는 쿼리를 사용하면 모델에 대한 `password` 속성을 받지 못합니다. 그러나 다음을 수행하는 경우 :

```typescript
const users = await connection.getRepository(User)
    .createQueryBuilder()
    .select("user.id", "id")
    .addSelect("user.password")
    .getMany();
```

쿼리에서 `password` 속성을 얻게됩니다.
