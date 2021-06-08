# 트랜잭션

- [트랜잭션 생성 및 사용](#트랜잭션-생성-및-사용)
  - [격리 수준 지정](#격리-수준-지정)
- [트랜잭션 데코레이터](#트랜잭션-데코레이터)
- [`QueryRunner`를 사용하여 단일 데이터베이스 연결상태 생성 및 제어](#queryrunner를-사용하여-단일-데이터베이스-연결상태-생성-및-제어)

## 트랜잭션 생성 및 사용

트랜잭션은 `Connection` 또는 `EntityManager`를 사용하여 생성됩니다.

예:

```typescript
import {getConnection} from "typeorm";

await getConnection().transaction(async transactionalEntityManager => {

});
```

또는

```typescript
import {getManager} from "typeorm";

await getManager().transaction(async transactionalEntityManager => {

});
```

트랜잭션에서 실행하려는 모든 것은 콜백에서 실행되어야 합니다.

```typescript
import {getManager} from "typeorm";

await getManager().transaction(async transactionalEntityManager => {
    await transactionalEntityManager.save(users);
    await transactionalEntityManager.save(photos);
    // ...
});
```

트랜잭션에서 작업할 때 가장 중요한 제한은 **항상** 제공된 엔티티 관리자 인스턴스를 사용하는 것입니다.
이 예에서는 `transactionalEntityManager`입니다.
전역 관리자(`getManager` 또는 연결 관리자)를 사용하면 문제가 발생합니다.
또한 글로벌 관리자 또는 연결을 사용하여 쿼리를 실행하는 클래스를 사용할 수 없습니다.
모든 작업은 제공된 트랜잭션 엔티티 관리자를 사용하여 **반드시** 실행해야합니다.

### 격리 수준 지정

트랜잭션에 대한 격리수준을 지정하려면 첫 번째 매개변수로 제공하면됩니다.

```typescript
import {getManager} from "typeorm";

await getManager().transaction("SERIALIZABLE", transactionalEntityManager => {

});
```

격리 수준 구현은 모든 데이터베이스에서 독립적이지 **않습니다**.

다음 데이터베이스 드라이버는 표준 격리 수준(`READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`)을 지원합니다.

* MySQL
* Postgres
* SQL Server

**SQlite**는 기본적으로 트랜잭션을 `SERIALIZABLE`로 설정하지만 *공유 캐시모드*가 활성화된 경우 트랜잭션은 `READ UNCOMMITTED` 격리 수준을 사용할 수 있습니다.

**Oracle**은 `READ COMMITTED` 및 `SERIALIZABLE` 격리수준만 지원합니다.


## 트랜잭션 데코레이터

There are a few decorators which can help you organize your transactions -
`@Transaction`, `@TransactionManager` and `@TransactionRepository`.

`@Transaction` wraps all its execution into a single database transaction,
and `@TransactionManager` provides a transaction entity manager which must be used to execute queries inside this transaction:
거래를 구성하는데 도움이되는 데코레이터가 몇 가지 있습니다 - `@Transaction`, `@TransactionManager` 및 `@TransactionRepository`.

`@Transaction`은 모든 실행을 단일 데이터베이스 트랜잭션으로 래핑합니다. 그리고 `@TransactionManager`는 이 트랜잭션 내에서 쿼리를 실행하는 데 사용해야하는 트랜잭션 엔티티 관리자를 제공합니다.

```typescript
@Transaction()
save(@TransactionManager() manager: EntityManager, user: User) {
    return manager.save(user);
}
```

격리 수준:

```typescript
@Transaction({ isolation: "SERIALIZABLE" })
save(@TransactionManager() manager: EntityManager, user: User) {
    return manager.save(user);
}
```

**반드시** 항상 `@ TransactionManager`에서 제공하는 관리자를 사용해야합니다.

그러나 `@TransactionRepository`를 사용하여 트랜잭션 저장소(내부에서 트랜잭션 엔티티 관리자를 사용함)를 삽입할 수도 있습니다.

```typescript
@Transaction()
save(user: User, @TransactionRepository(User) userRepository: Repository<User>) {
    return userRepository.save(user);
}
```

`Repository`, `TreeRepository` 및 `MongoRepository`(`@TransactionRepository(Entity) entityRepository: Repository<Entity>` 사용)와 같은 내장 TypeORM의 저장소 또는 `@TransactionRepository() customRepository: CustomRepository`를 사용하여 사용자 정의 저장소(내장 TypeORM의 저장소 클래스를 확장하는 클래스)를 모두 삽입 할 수 있습니다.

## `QueryRunner`를 사용하여 단일 데이터베이스 연결상태 생성 및 제어

`QueryRunner`는 단일 데이터베이스 연결을 제공합니다. 트랜잭션은 쿼리 실행기를 사용하여 구성됩니다. 단일 트랜잭션은 단일 쿼리 실행기에서만 설정할 수 있습니다. 쿼리 실행기 인스턴스를 수동으로 만들고 이를 사용하여 트랜잭션 상태를 수동으로 제어할 수 있습니다.

예:

```typescript
import {getConnection} from "typeorm";

// 연결을 얻고 새 쿼리 실행기를 만듭니다.
const connection = getConnection();
const queryRunner = connection.createQueryRunner();

// 새로운 쿼리 실행기를 사용하여 실제 데이터베이스 연결 설정
await queryRunner.connect();

// 이제 쿼리 실행기에서 모든 쿼리를 실행할 수 있습니다. 예를 들면 다음과 같습니다.
await queryRunner.query("SELECT * FROM users");

// 쿼리 실행기에 의해 생성된 연결로 작동하는 엔티티 관리자에 액세스할 수도 있습니다.
const users = await queryRunner.manager.find(User);

// 이제 새 트랜잭션을 열 수 있습니다.
await queryRunner.startTransaction();

try {

    // 이 트랜잭션에 대해 몇 가지 작업을 실행합니다.
    await queryRunner.manager.save(user1);
    await queryRunner.manager.save(user2);
    await queryRunner.manager.save(photos);

    // 지금 트랜잭션 커밋:
    await queryRunner.commitTransaction();

} catch (err) {

    // s오류가 있습니다. 변경사항을 롤백하겠습니다.
    await queryRunner.rollbackTransaction();

} finally {

    // 수동으로 생성된 쿼리 실행기를 해제해야합니다.
    await queryRunner.release();
}
```

`QueryRunner`에서 트랜잭션을 제어하는 3가지 방법이 있습니다.


* `startTransaction` - 쿼리 실행기 인스턴스 내에서 새 트랜잭션을 시작합니다.
* `commitTransaction` - 쿼리 실행기 인스턴스를 사용하여 만든 모든 변경사항을 커밋합니다.
* `rollbackTransaction` - 쿼리 실행기 인스턴스를 사용하여 만든 모든 변경사항을 롤백합니다.

[Query Runner](./query-runner.md)에 대해 자세히 알아보세요.
