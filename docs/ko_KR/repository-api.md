# Repository APIs

- [`Repository` API](#repository-api)
- [`TreeRepository` API](#treerepository-api)
- [`MongoRepository` API](#mongorepository-api)

## `Repository` API

* `manager` - 이 저장소에서 사용하는 `EntityManager`입니다.

```typescript
const manager = repository.manager;
```

* `metadata` - 이 저장소에서 관리하는 엔티티의 `EntityMetadata`입니다. [엔티티 메타데이터의 트랜잭션](./entity-metadata.md)에 대해 자세히 알아보세요.

```typescript
const metadata = repository.metadata;
```

* `queryRunner` - `EntityManager`에서 사용하는 쿼리 실행기입니다. EntityManager의 트랜잭션 인스턴스에서만 사용됩니다.

```typescript
const queryRunner = repository.queryRunner;
```

* `target` - 이 저장소에서 관리하는 대상 엔티티 클래스입니다. EntityManager의 트랜잭션 인스턴스에서만 사용됩니다.

```typescript
const target = repository.target;
```

* `createQueryBuilder` - SQL 쿼리를 작성하는데 사용할 쿼리 작성기를 만듭니다. [QueryBuilder](select-query-builder.md)에 대해 자세히 알아보세요.

```typescript
const users = await repository
    .createQueryBuilder("user")
    .where("user.name = :name", { name: "John" })
    .getMany();
```

* `hasId` - 지정된 엔터티의 기본 컬럼 속성이 정의되었는지 확인합니다.

```typescript
 if (repository.hasId(user)) {
    // ... do something
 }
```

* `getId` - 지정된 엔터티의 기본 컬럼 속성값을 가져옵니다. 엔터티에 복합 기본키가 있는 경우 반환된 값은 기본 컬럼의 이름과 값이 있는 객체가됩니다.

```typescript
const userId = repository.getId(user); // userId === 1
```

* `create` - `User`의 새 인스턴스를 만듭니다. 새로 생성된 사용자 객체에 기록될 사용자 속성이 있는 객체 리터럴을 선택적으로 허용합니다.

```typescript
const user = repository.create(); // const user = new User();와 같음
const user = repository.create({
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}); // const user = new User(); user.firstName = "Timber"; user.lastName = "Saw";와 같음
```

* `merge` - 여러 항목을 단일 항목으로 병합합니다.

```typescript
const user = new User();
repository.merge(user, { firstName: "Timber" }, { lastName: "Saw" }); // user.firstName = "Timber"; user.lastName = "Saw";와 같음
```

* `preload` - 주어진 일반 자바스크립트 객체에서 새 엔티티를 만듭니다. 엔터티가 데이터베이스에 이미 존재하는 경우 엔터티(및 이와 관련된 모든 항목)를 로드하고 모든 값을 지정된 개체의 새 값으로 바꾼 다음 새 엔터티를 반환합니다. 새 엔티티는 실제로 데이터베이스에서 로드된 엔티티이며 모든 속성이 새 객체에서 대체됩니다. 주어진 객체와 유사한 객체에는 객체를 찾기위한 객체 ID / 기본키가 있어야합니다. 주어진 ID를 가진 엔티티를 찾을 수 없는 경우 `undefined`를 반환합니다.

```typescript
const partialUser = {
    id: 1,
    firstName: "Rizzrak",
    profile: {
        id: 1
    }
};
const user = await repository.preload(partialUser);
// user는 partialUser 속성 값이 있는 partialUser의 모든 누락된 데이터를 포함합니다.
// { id: 1, firstName: "Rizzrak", lastName: "Saw", profile: { id: 1, ... } }
```

* `save` - 주어진 엔티티 또는 엔티티 배열을 저장합니다. 엔티티가 이미 데이터베이스에 있는 경우 업데이트됩니다. 엔터티가 데이터베이스에 없으면 삽입됩니다. 주어진 모든 엔티티를 단일 트랜잭션에 저장합니다(엔티티의 경우 관리자는 트랜잭션이 아님). 정의되지 않은 모든 속성을 건너뛰기 때문에 부분 업데이트도 지원합니다. 저장된 엔티티 / 엔티티들을 반환합니다.

```typescript
await repository.save(user);
await repository.save([
    category1,
    category2,
    category3
]);
```

* `remove` - 주어진 엔티티 또는 엔티티 배열을 제거합니다. 단일 트랜잭션에서 주어진 모든 엔티티를 제거합니다(엔티티의 경우 관리자가 트랜잭션이 아님). 제거된 엔티티 / 엔티티들을 반환합니다.

```typescript
await repository.remove(user);
await repository.remove([
    category1,
    category2,
    category3
]);
```

* `insert` - 새 엔티티 또는 엔티티 배열을 삽입합니다.

```typescript
await repository.insert({
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
await repository.update({ firstName: "Timber" }, { firstName: "Rizzrak" });
// UPDATE user SET firstName = Rizzrak WHERE firstName = Timber 실행

await repository.update(1, { firstName: "Rizzrak" });
// UPDATE user SET firstName = Rizzrak WHERE id = 1 실행
```

* `delete` - 엔티티 ID, ID 또는 주어진 조건으로 엔티티를 삭제합니다.

```typescript
await repository.delete(1);
await repository.delete([1, 2, 3]);
await repository.delete({ firstName: "Timber" });
```

* `softDelete` 와 `restore` - ID로 행을 소프트 삭제 및 복원

```typescript
const repository = connection.getRepository(Entity);
// 항목 삭제
await repository.softDelete(1);
// 그리고 복원을 사용하여 복원 할 수 있습니다.
await repository.restore(1);
```

* `softRemove` 와 `recover` - 이것은 `softDelete` 및 `restore`의 대안입니다.
```typescript
// softRemove를 사용하여 일시 삭제할 수 있습니다.
const entities = await repository.find();
const entitiesAfterSoftRemove = await repository.softRemove(entities);

// 그리고 복구를 사용하여 복구할 수 있습니다.
await repository.recover(entitiesAfterSoftRemove);
```


* `count` - 주어진 옵션과 일치하는 엔티티를 계산합니다. 페이지 매김에 유용합니다.

```typescript
const count = await repository.count({ firstName: "Timber" });
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
const timbers = await repository.find({ firstName: "Timber" });
```

* `findAndCount` - 주어진 찾기 옵션과 일치하는 엔티티를 찾습니다. 또한 주어진 조건과 일치하는 모든 항목을 계산하지만 페이지 매김 설정(`skip` 및 `take` 옵션)을 무시합니다.

```typescript
const [timbers, timbersCount] = await repository.findAndCount({ firstName: "Timber" });
```

* `findByIds` - ID로 여러 엔티티를 찾습니다.

```typescript
const users = await repository.findByIds([1, 2, 3]);
```

* `findOne` - 일부 ID 또는 찾기 옵션과 일치하는 첫번째 엔티티를 찾습니다.

```typescript
const user = await repository.findOne(1);
const timber = await repository.findOne({ firstName: "Timber" });
```

* `findOneOrFail` - 일부 ID 또는 찾기 옵션과 일치하는 첫번째 엔티티를 찾습니다. 일치하는 것이 없으면 반환된 Promise를 거부합니다.

```typescript
const user = await repository.findOneOrFail(1);
const timber = await repository.findOneOrFail({ firstName: "Timber" });
```

> 참고: `findOne` 및 `findOneOrFail`을 호출하기 전에 `id` 또는 `FindOptions` 값이 `null` 또는 `undefined`가 아닌지 확인하는 것이 좋습니다. `null`또는 `undefined`를 전달하면 쿼리가 저장소의 모든 항목과 일치하고 첫번째 레코드를 반환합니다.

* `query` - 원시 SQL 쿼리를 실행합니다.

```typescript
const rawData = await repository.query(`SELECT * FROM USERS`);
```

* `clear` - 주어진 테이블에서 모든 데이터를 지 웁니다 (잘라내기 / 삭제).

```typescript
await repository.clear();
```
### 추가 옵션

선택적 `SaveOptions`는 `save`의 매개변수로 전달할 수 있습니다.

* `data` -  persist 메소드로 전달할 추가 데이터입니다. 이 데이터는 가입자에게 사용될 수 있습니다.
* `listeners`: boolean - 이 작업에 대해 리스너 및 구독자가 호출되는지 여부를 나타냅니다. 기본적으로 활성화되어 있으며 저장 / 제거 옵션에서 `{listeners: false}`를 설정하여 비활성화 할 수 있습니다.
* `transaction`: boolean - 기본적으로 트랜잭션이 활성화되고 지속성 작업의 모든 쿼리가 트랜잭션으로 래핑됩니다. 지속성 옵션에서 `{transaction: false}`를 설정하여 이 동작을 비활성화 할 수 있습니다.
* `chunk`: number - 저장 실행을 여러 청크 그룹으로 나눕니다. 예를 들어, 100,000 개의 객체를 저장하고 싶지만 저장하는데 문제가 있는 경우 10개 그룹의 10,000개 객체로 나누고(`{chunk: 10000}` 설정) 각 그룹을 개별적으로 저장할 수 있습니다. 이 옵션은 기본 드라이버 매개변수 번호 제한에 문제가 있을 때 매우 큰 삽입을 수행하는 데 필요합니다.
* `reload`: boolean - 지속성 작업중에 지속되는 엔터티를 다시 로드해야 하는지 여부를 결정하는 플래그입니다. RETURNING / OUTPUT 문을 지원하지 않는 데이터베이스에서만 작동합니다. 기본적으로 활성화됩니다.

Example:
```typescript
// 사용자는 사용자 엔티티의 배열을 포함합니다.
userRepository.save(users, {chunk: users.length / 1000});
```

선택적 `RemoveOptions`는 `remove` 및 `delete`의 매개변수로 전달할 수 있습니다.

* `data` - remove 메소드로 전달할 추가 데이터입니다. 이 데이터는 가입자에게 사용될 수 있습니다.
* `listener`: boolean - 이 작업에 대해 리스너 및 구독자가 호출되는지 여부를 나타냅니다. 기본적으로 활성화되어 있으며 저장 / 제거 옵션에서 `{listeners: false}`를 설정하여 비활성화할 수 있습니다.
* `transaction`: boolean - 기본적으로 트랜잭션이 활성화되고 지속성 작업의 모든 쿼리가 트랜잭션으로 래핑됩니다. 지속성 옵션에서 `{transaction: false}`를 설정하여 이 동작을 비활성화할 수 있습니다.
* `chunk`: number - 저장 실행을 여러 청크 그룹으로 나눕니다. 예를 들어, 100,000개의 객체를 저장하고 싶지만 저장하는데 문제가 있는 경우 `{chunk: 10000}`를 설정하여 객체를 10,000개의 객체로 구성된 10개 그룹으로 나누고 각 그룹을 개별적으로 저장할 수 있습니다. 이 옵션은 기본 드라이버 매개변수 번호 제한에 문제가 있을 때 매우 큰 삽입을 수행하는데 필요합니다.

Example:
```typescript
// 사용자는 사용자 엔티티의 배열을 포함합니다.
userRepository.remove(users, {chunk: entities.length / 1000});
```

## `TreeRepository` API

`TreeRepository` API는 [트리 엔티티 문서](./tree-entities.md#트리-엔티티와-작업하기)를 참조하세요.

## `MongoRepository` API

`MongoRepository` API는 [MongoDB 문서](./mongodb.md)를 참조하세요.
