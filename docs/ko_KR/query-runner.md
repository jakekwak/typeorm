# 쿼리 실행기와 작업하기

- [`QueryRunner`란?](#queryrunner란)
- [새 queryRunner 만들기](#새-queryrunner-만들기)
- [queryRunner 사용하기](#queryrunner-사용하기)
- [`QueryRunner` 작업하기](#queryrunner-작업하기)

## `QueryRunner`란?

데이터베이스와의 상호 작용은 연결을 설정한 후에 만 가능합니다. TypeORM의 `Connection`은 보이는 것처럼 데이터베이스 연결을 설정하지 않고 대신 연결 풀을 설정합니다. 실제 데이터베이스 연결에 관심이 있다면 `QueryRunner`를 사용해야합니다. `QueryRunner`의 각 인스턴스는 별도의 격리된 데이터베이스 연결입니다. 쿼리 실행기를 사용하면 단일 데이터베이스 연결을 사용하여 실행할 쿼리를 제어하고 데이터베이스 트랜잭션을 수동으로 제어할 수 있습니다.

## 새 queryRunner 만들기

`QueryRunner`의 새 인스턴스를 만들려면 먼저 `Connection` 설명서에 설명된 방법중 하나를 사용하여 연결 풀을 만들어야합니다. 연결이 설정되면 `createQueryRunner` 함수를 사용하여 격리된 연결을 만듭니다.

`createQueryRunner` 단일 데이터베이스 연결에서 쿼리를 수행하는 데 사용되는 쿼리 실행기를 만듭니다.


```typescript
import { getConnection, QueryRunner } from 'typeorm';
// createConnection이 호출되고 해결되면 사용할 수 있습니다.
const connection: Connection = getConnection();

const queryRunner: QueryRunner = connection.createQueryRunner();
```
## queryRunner 사용하기

`QueryRunner`의 인스턴스를 만든 후 connect를 사용하여 연결을 활성화합니다.

```typescript
import { getConnection, QueryRunner } from 'typeorm';
// createConnection이 호출되고 해결되면 사용할 수 있습니다.
const connection: Connection = getConnection();

const queryRunner: QueryRunner = connection.createQueryRunner();

await queryRunner.connect(); // 연결을 수행
```

`QueryRunner`는 격리된 데이터베이스 연결을 관리하는데 사용되므로 연결 풀에서 다시 사용할 수 있도록 더 이상 필요하지 않을 때 해제해야합니다. 연결이 해제된 후에는 쿼리 실행기 메서드를 사용할 수 없습니다.

## `QueryRunner` 작업하기

queryRunner를 설정하면 `Connection` 인터페이스와 유사한 인터페이스로 사용할 수 있습니다.

```typescript
import { getConnection, QueryRunner } from 'typeorm';
import { User } from "../entity/User";

export class UserController {


    @Get("/users")
    getAll(): Promise<User[]> {
        // createConnection이 호출되고 해결되면 사용할 수 있습니다.
        const connection: Connection = getConnection();

        const queryRunner: QueryRunner = connection.createQueryRunner();

        await queryRunner.connect(); // 연결을 수행

        const users = await queryRunner.manager.find(User);

        await queryRunner.release(); // 릴리스 연결

        return users;
    }

}
```
