# `EntityManager` API

* `connection` - `EntityManager`에서 사용하는 연결입니다.

```typescript
const connection = manager.connection;
```

* `queryRunner` - `EntityManager`에서 사용하는 쿼리 실행기입니다. EntityManager의 트랜잭션 인스턴스에서만 사용됩니다.

```typescript
const queryRunner = manager.queryRunner;
```

* `transaction` - 단일 데이터베이스 트랜잭션에서 여러 데이터베이스 요청이 실행되는 트랜잭션을 제공합니다. 자세히 알아보기 [트랜잭션](./transactions.md).

```typescript
await manager.transaction(async manager => {
    // 참고: 주어진 관리자 인스턴스를 사용하여 모든 데이터베이스 작업을 수행해야합니다.
    // 이 인스턴스는 이 트랜잭션과 함께 작동하는 EntityManager의 특수 인스턴스이며 여기서 기다리는 것을 잊지 마십시오.
});
```

* `query` - Executes a raw SQL query.

```typescript
const rawData = await manager.query(`SELECT * FROM USERS`);
```

* `createQueryBuilder` - Creates a query builder use to build SQL queries.
Learn more about [QueryBuilder](select-query-builder.md).

```typescript
const users = await manager.createQueryBuilder()
    .select()
    .from(User, "user")
    .where("user.name = :name", { name: "John" })
    .getMany();
```

* `hasId` - 지정된 엔터티에 기본 컬럼 속성이 정의되어 있는지 확인합니다.

```typescript
 if (manager.hasId(user)) {
    // ... do something
 }
```

* `getId` - 주어진 엔터티의 기본 컬럼 속성 값을 가져옵니다. 엔터티에 복합 기본 키가 있는 경우 반환된 값은 기본 컬럼의 이름과 값이 있는 객체가됩니다.

```typescript
const userId = manager.getId(user); // userId === 1
```

* `create` - `User`의 새 인스턴스를 만듭니다. 선택적으로 새로 생성된 사용자 객체에 기록될 사용자 속성이 있는 객체 리터럴을 허용합니다.

```typescript
const user = manager.create(User); // const user = new User();와 동일
const user = manager.create(User, {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}); // const user = new User(); user.firstName = "Timber"; user.lastName = "Saw";와 동일
```

* `merge` - 여러 항목을 단일 항목으로 병합합니다.

```typescript
const user = new User();
manager.merge(User, user, { firstName: "Timber" }, { lastName: "Saw" }); // user.firstName = "Timber"; user.lastName = "Saw";와 동일
```

* `preload` - 주어진 일반 자바스크립트 객체에서 새 엔티티를 만듭니다. 엔터티가 이미 데이터베이스에 있는 경우 엔터티(및 이와 관련된 모든 항목)를 로드하고 모든 값을 지정된 객체의 새 값으로 바꾼 다음 새 엔터티를 반환합니다. 새 객체는 실제로 데이터베이스 객체에서 로드되고 모든 속성은 새 객체에서 대체됩니다.

```typescript
const partialUser = {
    id: 1,
    firstName: "Rizzrak",
    profile: {
        id: 1
    }
};
const user = await manager.preload(User, partialUser);
// user는 partialUser 속성 값이 있는 partialUser의 모든 누락된 데이터를 포함합니다.
// { id: 1, firstName: "Rizzrak", lastName: "Saw", profile: { id: 1, ... } }
```

* `save` - 주어진 엔티티 또는 엔티티 배열을 저장합니다. 엔터티가 이미 데이터베이스에 있는 경우 업데이트됩니다. 엔티티가 아직 데이터베이스에 존재하지 않는 경우 삽입됩니다. 주어진 모든 엔티티를 단일 트랜잭션으로 저장합니다(엔티티 관리자가 트랜잭션이 아닌 경우). 정의되지 않은 모든 속성을 건너뛰기 때문에 부분 업데이트도 지원합니다. 값을 `NULL`로 만들려면 속성을 `null`과 같도록 수동으로 설정해야합니다.

```typescript
await manager.save(user);
await manager.save([
    category1,
    category2,
    category3
]);
```

* `remove` - 주어진 엔티티 또는 엔티티 배열을 제거합니다. 단일 트랜잭션에서 주어진 모든 엔티티를 제거합니다(엔티티의 경우 관리자가 트랜잭션이 아님).

```typescript
await manager.remove(user);
await manager.remove([
    category1,
    category2,
    category3
]);
```

* `insert` - 새 엔티티 또는 엔티티 배열을 삽입합니다.

```typescript
await manager.insert(User, {
    firstName: "Timber",
    lastName: "Timber"
});

await manager.insert(User, [{
    firstName: "Foo",
    lastName: "Bar"
}, {
    firstName: "Rizz",
    lastName: "Rak"
}]);
```

* `update` - 주어진 업데이트 옵션 또는 엔티티 ID로 엔티티를 부분적으로 업데이트합니다.

```typescript
await manager.update(User, { firstName: "Timber" }, { firstName: "Rizzrak" });
// UPDATE user SET firstName = Rizzrak WHERE firstName = Timber 실행합니다

await manager.update(User, 1, { firstName: "Rizzrak" });
// UPDATE user SET firstName = Rizzrak WHERE id = 1 실행합니다.
```

* `delete` - 엔티티 ID, ID 또는 주어진 조건으로 엔티티를 삭제합니다.

```typescript
await manager.delete(User, 1);
await manager.delete(User, [1, 2, 3]);
await manager.delete(User, { firstName: "Timber" });
```

* `count` - 주어진 옵션과 일치하는 엔티티를 계산합니다. 페이지 매김에 유용합니다.

```typescript
const count = await manager.count(User, { firstName: "Timber" });
```

* `increment` - 주어진 옵션과 일치하는 엔티티의 제공된 값으로 일부 컬럼을 증가시킵니다.

```typescript
await manager.increment(User, { firstName: "Timber" }, "age", 3);
```

* `decrement` - 주어진 옵션과 일치하는 제공된 값으로 일부 컬럼을 줄입니다.
```typescript
await manager.decrement(User, { firstName: "Timber" }, "age", 3);
```

* `find` - 주어진 옵션과 일치하는 엔티티를 찾습니다.

```typescript
const timbers = await manager.find(User, { firstName: "Timber" });
```

* `findAndCount` - 주어진 찾기 옵션과 일치하는 엔티티를 찾습니다. 또한 주어진 조건과 일치하는 모든 엔티티를 계산하지만 페이지 매김 설정(from 및 take 옵션)은 무시합니다.

```typescript
const [timbers, timbersCount] = await manager.findAndCount(User, { firstName: "Timber" });
```

* `findByIds` - ID로 여러 엔티티를 찾습니다.

```typescript
const users = await manager.findByIds(User, [1, 2, 3]);
```

* `findOne` - 일부 ID 또는 찾기 옵션과 일치하는 첫번째 엔티티를 찾습니다.

```typescript
const user = await manager.findOne(User, 1);
const timber = await manager.findOne(User, { firstName: "Timber" });
```

* `findOneOrFail` - 일부 ID 또는 찾기 옵션과 일치하는 첫번째 엔티티를 찾습니다. 일치하는 것이 없으면 반환된 Promise를 거부합니다.

```typescript
const user = await manager.findOneOrFail(User, 1);
const timber = await manager.findOneOrFail(User, { firstName: "Timber" });
```

* `clear` - 주어진 테이블에서 모든 데이터를 지웁니다(잘라내기(truncates) / 삭제(drops)).

```typescript
await manager.clear(User);
```

* `getRepository` - 특정 엔티티에 대한 작업을 수행하기 위해 `Repository`를 가져옵니다. [리포지토리](./working-with-repository.md)에 대해 자세히 알아보세요.

```typescript
const userRepository = manager.getRepository(User);
```

* `getTreeRepository` - 특정 엔티티에 대한 작업을 수행하기 위해 `TreeRepository`를 가져옵니다. [리포지토리](./working-with-repository.md)에 대해 자세히 알아보세요.

```typescript
const categoryRepository = manager.getTreeRepository(Category);
```

* `getMongoRepository` - 특정 엔티티에 대한 작업을 수행하기 위해 `MongoRepository`를 가져옵니다. [MongoDB](./mongodb.md)에 대해 자세히 알아보세요.

```typescript
const userRepository = manager.getMongoRepository(User);
```

* `getCustomRepository` - 사용자 정의 엔티티 저장소를 가져옵니다. [커스텀 리포지토리](./custom-repository.md)에 대해 자세히 알아보세요.

```typescript
const myUserRepository = manager.getCustomRepository(UserRepository);
```

* `release` - 엔티티 관리자의 쿼리 실행기를 해제합니다. 쿼리 실행기를 수동으로 만들고 관리할 때만 사용됩니다.

```typescript
await manager.release();
```
