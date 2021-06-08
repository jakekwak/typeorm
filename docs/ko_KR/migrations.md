# 마이그레이션

- [마이그레이션 작동 방식](#마이그레이션-작동-방식)
- [새 마이그레이션 만들기](#새-마이그레이션-만들기)
- [마이그레이션 실행 및 되돌리기](#마이그레이션-실행-및-되돌리기)
- [마이그레이션 생성](#마이그레이션-생성)
- [커넥션 옵션](#커넥션-옵션)
- [마이그레이션 API를 사용하여 마이그레이션 작성](#마이그레이션-api를-사용하여-마이그레이션-작성)

## 마이그레이션 작동 방식

프로덕션에 들어가면 모델 변경사항을 데이터베이스에 동기화해야합니다. 일반적으로 프로덕션에서 스키마 동기화를 위해 `synchronize: true`를 사용하는 것은 안전하지 않습니다. 데이터베이스에서 데이터를 얻습니다. 여기에 마이그레이션이 도움이됩니다.

마이그레이션은 데이터베이스 스키마를 업데이트하는 SQL 쿼리가있는 단일 파일입니다. 기존 데이터베이스에 새로운 변경 사항을 적용합니다.

이미 데이터베이스와 게시물 엔티티가 있다고 가정해 보겠습니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Post {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

}
```

그리고 당신의 엔티티는 변경없이 몇달동안 프로덕션상태로 있었습니다. 데이터베이스에 수천개의 게시물이 있습니다.

이제 새 릴리스를 만들고 `title`을 `name`으로 변경해야합니다. 당신은 무엇을 하시겠습니까?

다음 SQL 쿼리(postgres 언어)를 사용하여 새 마이그레이션을 만들어야합니다.

```sql
ALTER TABLE "post" ALTER COLUMN "title" RENAME TO "name";
```

이 SQL 쿼리를 실행하면 데이터베이스 스키마가 새 코드베이스와 함께 작동할 준비가됩니다. TypeORM은 이러한 SQL 쿼리를 작성하고 필요할 때 실행할 수 있는 위치를 제공합니다. 이 장소를 `migrations`라고 합니다.

## 새 마이그레이션 만들기

**전제 조건**: [CLI 설치하기](./using-cli.md#installing-cli)

새 마이그레이션을 만들기 전에 연결 옵션을 올바르게 설정해야합니다.

```json
{
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "username": "test",
    "password": "test",
    "database": "test",
    "entities": ["entity/*.js"],
    "migrationsTableName": "custom_migration_table",
    "migrations": ["migration/*.js"],
    "cli": {
        "migrationsDir": "migration"
    }
}
```

여기에서 세 가지 옵션을 설정합니다.

* `"migrationsTableName": "migrations"` - 마이그레이션 테이블 이름이 `"migrations"`와 달라야하는 경우에만 이 옵션을 지정하십시오.
* `"migrations": ["migration/*.js"]` - typeorm이 주어진 `"migration"`디렉토리에서 마이그레이션을 로드해야 함을 나타냅니다.
* `"cli": { "migrationsDir": "migration" }` - CLI가 `"migration"` 디렉토리에 새 마이그레이션을 만들어야 함을 나타냅니다.

연결 옵션을 설정하면 CLI를 사용하여 새 마이그레이션을 만들 수 있습니다.

```
typeorm migration:create -n PostRefactoring
```

여기서 `PostRefactoring`은 마이그레이션의 이름입니다 - 원하는 이름을 지정할 수 있습니다. 명령어를 실행하면 "migration" 디렉토리에 `{TIMESTAMP}-PostRefactoring.ts`라는 이름의 새 파일이 생성된 것을 볼 수 있습니다. 여기서 `{TIMESTAMP}`는 마이그레이션이 생성된 현재 타임 스탬프입니다. 이제 파일을 열고 여기에 마이그레이션 SQL 쿼리를 추가할 수 있습니다.

마이그레이션 내에 다음 내용이 표시되어야합니다.

```typescript
import {MigrationInterface, QueryRunner} from "typeorm";

export class PostRefactoringTIMESTAMP implements MigrationInterface {

    async up(queryRunner: QueryRunner): Promise<void> {

    }

    async down(queryRunner: QueryRunner): Promise<void> {

    }


}
```

마이그레이션 코드로 채워야하는 두 가지 방법이 있습니다: `up` 및 `down`.
`up`은 마이그레이션을 수행하는 데 필요한 코드를 포함해야합니다.
`down`은 변경된 `up`을 되돌려야합니다.
`down` 메서드는 마지막 마이그레이션을 되돌리는 데 사용됩니다.

`up`과 `down` 안에는 `QueryRunner` 객체가 있습니다.
모든 데이터베이스 작업은 이 객체를 사용하여 실행됩니다.
[query runner](./query-runner.md)에 대해 자세히 알아보세요.

`Post` 변경사항으로 마이그레이션이 어떻게 보이는지 살펴 보겠습니다.

```typescript
import {MigrationInterface, QueryRunner} from "typeorm";

export class PostRefactoringTIMESTAMP implements MigrationInterface {

    async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.query(`ALTER TABLE "post" RENAME COLUMN "title" TO "name"`);
    }

    async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.query(`ALTER TABLE "post" RENAME COLUMN "name" TO "title"`); // reverts things made in "up" method
    }
}
```

## 마이그레이션 실행 및 되돌리기

프로덕션에서 실행할 마이그레이션이 있으면 CLI 명령을 사용하여 실행할 수 있습니다.

```
typeorm migration:run
```

**`typeorm migration:create` 및 `typeorm migration:generate`는 `o` 플래그를 사용하지 않는 한 `.ts` 파일을 생성합니다 ([마이그레이션 생성](#generating-migrations)에서 자세히 참조). `migration:run` 및 `migration:revert` 명령은 `.js` 파일에서만 작동합니다. 따라서 명령을 실행하기 전에 typescript 파일을 컴파일해야합니다.** 또는 `typeorm`과 함께 `ts-node`를 사용하여 `.ts` 마이그레이션 파일을 실행할 수 있습니다.

`ts-node`를 사용한 예:

```
ts-node --transpile-only ./node_modules/typeorm/cli.js migration:run
```

`node_modules`를 직접 사용하지 않는`ts-node` 예:

```
ts-node $(yarn bin typeorm) migration:run
```

이 명령은 보류중인 모든 마이그레이션을 실행하고 타임 스탬프 순서에 따라 순서대로 실행합니다.
즉, 생성된 마이그레이션의 `up` 메서드로 작성된 모든 SQL 쿼리가 실행됩니다.
그게 전부입니다! 이제 데이터베이스 스키마가 최신 상태입니다.

어떤 이유로 든 변경사항을 되돌리려면 다음을 실행할 수 있습니다.

```
typeorm migration:revert
```

이 명령은 최근에 실행된 마이그레이션에서 `down`을 실행합니다.
여러 마이그레이션을 되돌려야 하는 경우 이 명령을 여러번 호출해야합니다.

## 마이그레이션 생성

TypeORM은 스키마 변경사항으로 마이그레이션 파일을 자동으로 생성할 수 있습니다.

`title` 컬럼이 있는 `Post` 엔티티가 있고 `title` 이름을 `name`으로 변경했다고 가정해 보겠습니다.
다음 명령을 실행할 수 있습니다.

```
typeorm migration:generate -n PostRefactoring
```

그리고 다음 내용으로`{TIMESTAMP}-PostRefactoring.ts`라는 새 마이그레이션을 생성합니다.

```typescript
import {MigrationInterface, QueryRunner} from "typeorm";

export class PostRefactoringTIMESTAMP implements MigrationInterface {

    async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.query(`ALTER TABLE "post" ALTER COLUMN "title" RENAME TO "name"`);
    }

    async down(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.query(`ALTER TABLE "post" ALTER COLUMN "name" RENAME TO "title"`);
    }


}
```

또는 `o`(`--outputJs`의 별칭) 플래그를 사용하여 마이그레이션을 자바스크립트 파일로 출력할 수도 있습니다. 이것은 TypeScript 추가 패키지가 설치되지 않은 Javascript 전용 프로젝트에 유용합니다. 이 명령은 다음 내용으로 새 마이그레이션 파일 `{TIMESTAMP}-PostRefactoring.js`를 생성합니다.

```javascript
const { MigrationInterface, QueryRunner } = require("typeorm");

module.exports = class PostRefactoringTIMESTAMP {

    async up(queryRunner) {
        await queryRunner.query(`ALTER TABLE "post" ALTER COLUMN "title" RENAME TO "name"`);
    }

    async down(queryRunner) {
        await queryRunner.query(`ALTER TABLE "post" ALTER COLUMN "title" RENAME TO "name"`);
    }
}
```

직접 쿼리를 작성할 필요는 없습니다.
마이그레이션 생성에 대한 경험적 규칙은 모델을 **각** 변경한 후에 마이그레이션을 생성하는 것입니다. 생성된 마이그레이션 쿼리에 여러 줄 형식을 적용하려면 `p`(`--pretty`의 별칭) 플래그를 사용합니다.

## 커넥션 옵션

기본값이 아닌 다른 연결에 대해 마이그레이션을 실행/되돌려 야하는 경우 `-c`(`--connection`의 별칭)를 사용하고 구성 이름을 인수로 전달합니다.

```
typeorm -c <your-config-name> migration:{run|revert}
```

## 마이그레이션 API를 사용하여 마이그레이션 작성

API를 사용하여 데이터베이스 스키마를 변경하려면 `QueryRunner`를 사용할 수 있습니다.

예:

```ts
import {MigrationInterface, QueryRunner, Table, TableIndex, TableColumn, TableForeignKey } from "typeorm";

export class QuestionRefactoringTIMESTAMP implements MigrationInterface {

    async up(queryRunner: QueryRunner): Promise<void> {
        await queryRunner.createTable(new Table({
            name: "question",
            columns: [
                {
                    name: "id",
                    type: "int",
                    isPrimary: true
                },
                {
                    name: "name",
                    type: "varchar",
                }
            ]
        }), true)

        await queryRunner.createIndex("question", new TableIndex({
            name: "IDX_QUESTION_NAME",
            columnNames: ["name"]
        }));

        await queryRunner.createTable(new Table({
            name: "answer",
            columns: [
                {
                    name: "id",
                    type: "int",
                    isPrimary: true
                },
                {
                    name: "name",
                    type: "varchar",
                },
                {
                  name: 'created_at',
                  type: 'timestamp',
                  default: 'now()'
                }
            ]
        }), true);

        await queryRunner.addColumn("answer", new TableColumn({
            name: "questionId",
            type: "int"
        }));

        await queryRunner.createForeignKey("answer", new TableForeignKey({
            columnNames: ["questionId"],
            referencedColumnNames: ["id"],
            referencedTableName: "question",
            onDelete: "CASCADE"
        }));
    }

    async down(queryRunner: QueryRunner): Promise<void> {
        const table = await queryRunner.getTable("answer");
        const foreignKey = table.foreignKeys.find(fk => fk.columnNames.indexOf("questionId") !== -1);
        await queryRunner.dropForeignKey("answer", foreignKey);
        await queryRunner.dropColumn("answer", "questionId");
        await queryRunner.dropTable("answer");
        await queryRunner.dropIndex("question", "IDX_QUESTION_NAME");
        await queryRunner.dropTable("question");
    }

}
```

---

```ts
getDatabases(): Promise<string[]>
```

시스템 데이터베이스를 포함하여 사용가능한 모든 데이터베이스 이름을 반환합니다.

---

```ts
getSchemas(database?: string): Promise<string[]>
```

- `database` - 데이터베이스 매개 변수가 지정된 경우 해당 데이터베이스의 스키마를 반환합니다.

시스템 스키마를 포함하여 사용가능한 모든 스키마 이름을 반환합니다. SQLServer 및 Postgres에만 유용합니다.

---

```ts
getTable(tableName: string): Promise<Table|undefined>
```

- `tableName` - 로드할 테이블 이름

데이터베이스에서 주어진 이름으로 테이블을 로드합니다.

---

```ts
getTables(tableNames: string[]): Promise<Table[]>
```

- `tableNames` - 로드할 테이블 이름

데이터베이스에서 주어진 이름으로 테이블을 로드합니다.

---

```ts
hasDatabase(database: string): Promise<boolean>
```

- `database` - 검사할 데이터베이스 이름

주어진 이름의 데이터베이스가 존재하는지 확인합니다.

---

```ts
hasSchema(schema: string): Promise<boolean>
```

- `schema` - 확인할 스키마의 이름

주어진 이름의 스키마가 존재하는지 확인합니다. SqlServer 및 Postgres에만 사용됩니다.

---

```ts
hasTable(table: Table|string): Promise<boolean>
```

- `table` - 테이블 객체 또는 이름

테이블이 있는지 확인합니다.

---

```ts
hasColumn(table: Table|string, columnName: string): Promise<boolean>
```

- `table` - 테이블 객체 또는 이름
- `columnName` - 확인할 컬럼 이름

테이블에 컬럼이 존재하는지 확인합니다.

---

```ts
createDatabase(database: string, ifNotExist?: boolean): Promise<void>
```

- `database` - 데이터베이스 이름
- `ifNotExist` - `true`인 경우 생성을 건너뛰고, 데이터베이스가 이미 존재하는 경우 오류 발생

새 데이터베이스를 만듭니다.

---

```ts
dropDatabase(database: string, ifExist?: boolean): Promise<void>
```

- `database` - 데이터베이스 이름
- `ifExist` - `true`이면 삭제를 건너뛰고, 그렇지 않으면 데이터베이스를 찾을 수 없으면 오류를 발생시킵니다.

데이터베이스를 삭제합니다.

---

```ts
createSchema(schemaPath: string, ifNotExist?: boolean): Promise<void>
```

- `schemaPath` - 스키마 이름. SqlServer의 경우 스키마 경로(예: 'dbName.schemaName')를 매개변수로 사용할 수 있습니다. 스키마 경로가 전달되면 지정된 데이터베이스에 스키마를 생성합니다.
- `ifNotExist` - `true`이면 생성을 건너뛰고, 스키마가 이미 있으면 오류를 발생시킵니다.

새 테이블 스키마를 생성합니다.

---

```ts
dropSchema(schemaPath: string, ifExist?: boolean, isCascade?: boolean): Promise<void>
```

- `schemaPath` - 스키마 이름. SqlServer의 경우 스키마 경로(예: 'dbName.schemaName')를 매개변수로 사용할 수 있습니다. 스키마 경로가 전달되면 지정된 데이터베이스에서 스키마를 삭제합니다.
- `ifExist` - `true`이면 삭제를 건너뛰고, 스키마를 찾을 수 없으면 오류를 발생시킵니다.
- `isCascade` - `true` 인 경우 스키마에 포함된 객체(테이블, 함수 등)를 자동으로 삭제합니다. Postgres에서만 사용됩니다.

테이블 스키마를 삭제합니다.

---

```ts
createTable(table: Table, ifNotExist?: boolean, createForeignKeys?: boolean, createIndices?: boolean): Promise<void>
```

- `table` - 테이블 객체.
- `ifNotExist` - `true`이면 생성을 건너뛰고, 테이블이 이미 있으면 오류를 발생시킵니다. 기본값 `false`
- `createForeignKeys` - 테이블 생성시 외래키가 생성되는지 여부를 나타냅니다. 기본값 `true`
- `createIndices` - 테이블 생성시 인덱스 생성 여부를 나타냅니다. 기본값 `true`

새 테이블을 만듭니다.

---

```ts
dropTable(table: Table|string, ifExist?: boolean, dropForeignKeys?: boolean, dropIndices?: boolean): Promise<void>
```

- `table` - 삭제할 테이블 객체 또는 테이블 이름
- `ifExist` - `true`이면 삭제를 건너뛰고, 그렇지 않으면 테이블이 없으면 오류가 발생합니다.
- `dropForeignKeys` - 테이블 삭제시 외래키를 삭제할지 여부를 나타냅니다. 기본값 `true`
- `dropIndices` - 테이블 삭제시 인덱스를 삭제할지 여부를 나타냅니다. 기본값 `true`

테이블을 삭제합니다.

---

```ts
renameTable(oldTableOrName: Table|string, newTableName: string): Promise<void>
```

- `oldTableOrName` - 이름을 바꿀 이전 테이블 객체 또는 이름
- `newTableName` - 새 테이블 이름

테이블의 이름을 바꿉니다.

---

```ts
addColumn(table: Table|string, column: TableColumn): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `column` - 새 컬럼

새 컬럼을 추가합니다.

---

```ts
addColumns(table: Table|string, columns: TableColumn[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `columns` - 새 컬럼들

새 컬럼들을 추가합니다.

---

```ts
renameColumn(table: Table|string, oldColumnOrName: TableColumn|string, newColumnOrName: TableColumn|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `oldColumnOrName` - 이전 컬럼. TableColumn 객체 또는 컬럼 이름 허용
- `newColumnOrName` - 새 컬럼. TableColumn 객체 또는 컬럼 이름 허용

Renames a column.

---

```ts
changeColumn(table: Table|string, oldColumn: TableColumn|string, newColumn: TableColumn): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `oldColumn` -  이전 컬럼. TableColumn 객체 또는 컬럼 이름 허용
- `newColumn` -  새 컬럼. TableColumn 객체 허용

테이블의 컬럼을 변경합니다.

---

```ts
changeColumns(table: Table|string, changedColumns: { oldColumn: TableColumn, newColumn: TableColumn }[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `changedColumns` - 변경된 컬럼의 배열.
  + `oldColumn` - 이전 TableColumn 객체
  + `newColumn` - 새로운 TableColumn 객체

테이블의 컬럼들을 변경합니다.

---

```ts
dropColumn(table: Table|string, column: TableColumn|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `column` - 삭제할 TableColumn 개체 또는 열 이름

테이블에서 컬럼을 삭제합니다.

---

```ts
dropColumns(table: Table|string, columns: TableColumn[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `columns` - 삭제할 TableColumn 객체의 배열

테이블의 컬럼 배열을 삭제합니다.

---

```ts
createPrimaryKey(table: Table|string, columnNames: string[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `columnNames` - array of column names which will be primary

Creates a new primary key.

---

```ts
updatePrimaryKeys(table: Table|string, columns: TableColumn[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `columns` - 업데이트될 TableColumn 객체의 배열

복합 기본키를 업데이트합니다.

---

```ts
dropPrimaryKey(table: Table|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름

기본키를 삭제합니다.

---

```ts
createUniqueConstraint(table: Table|string, uniqueConstraint: TableUnique): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `uniqueConstraint` - 생성할 TableUnique 객체

Creates new unique constraint.

> 참고: MySQL에서는 고유한 제약조건을 고유한 인덱스로 저장하므로 MySQL에서는 작동하지 않습니다. 대신 `createIndex()` 메소드를 사용하십시오.

---

```ts
createUniqueConstraints(table: Table|string, uniqueConstraints: TableUnique[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `uniqueConstraints` - 생성될 TableUnique 객체의 배열

새로운 고유 제약조건을 만듭니다.

> 참고: MySQL에서는 고유한 제약조건을 고유한 인덱스로 저장하므로 MySQL에서는 작동하지 않습니다. 대신 `createIndices()` 메소드를 사용하십시오.

---

```ts
dropUniqueConstraint(table: Table|string, uniqueOrName: TableUnique|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `uniqueOrName` - 삭제할 TableUnique 객체 또는 고유 제약 조건 이름

고유한 제약조건을 삭제합니다.

> 참고: MySQL에서는 고유한 제약조건을 고유한 인덱스로 저장하므로 MySQL에서는 작동하지 않습니다. 대신 `dropIndex()`메소드를 사용하십시오.

---

```ts
dropUniqueConstraints(table: Table|string, uniqueConstraints: TableUnique[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `uniqueConstraints` - 삭제할 TableUnique 객체의 배열

고유한 제약 조건을 삭제합니다.

> 참고: MySQL에서는 고유한 제약조건을 고유한 인덱스로 저장하므로 MySQL에서는 작동하지 않습니다. 대신 `dropIndices()` 메소드를 사용하십시오.

---

```ts
createCheckConstraint(table: Table|string, checkConstraint: TableCheck): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `checkConstraint` - TableCheck 객체

새로운 검사 제약을 생성합니다.

> Note: MySQL does not support check constraints.

---

```ts
createCheckConstraints(table: Table|string, checkConstraints: TableCheck[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `checkConstraints` - TableCheck 객체의 배열

새로운 검사 제약들을 생성합니다.

> 참고: MySQL은 검사 제약조건을 지원하지 않습니다.

---

```ts
dropCheckConstraint(table: Table|string, checkOrName: TableCheck|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `checkOrName` - TableCheck 객체 또는 검사 제약조건 이름

체크 제약조건을 삭제합니다.

> 참고 : MySQL은 검사 제약조건을 지원하지 않습니다.

---

```ts
dropCheckConstraints(table: Table|string, checkConstraints: TableCheck[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `checkConstraints` - TableCheck 객체의 배열

체크 제약조건들을 삭제합니다.

> 참고: MySQL은 검사 제약 조건을 지원하지 않습니다.

---

```ts
createForeignKey(table: Table|string, foreignKey: TableForeignKey): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `foreignKey` - TableForeignKey 객체

새 외래키를 만듭니다.

---

```ts
createForeignKeys(table: Table|string, foreignKeys: TableForeignKey[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `foreignKeys` - TableForeignKey 객체의 배열

새 외래키 배열을 만듭니다.

---

```ts
dropForeignKey(table: Table|string, foreignKeyOrName: TableForeignKey|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `foreignKeyOrName` - TableForeignKey 개체 또는 외래키 이름

외래키를 삭제합니다.

---

```ts
dropForeignKeys(table: Table|string, foreignKeys: TableForeignKey[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `foreignKeys` - TableForeignKey 객체의 배열

외래키들을 삭제합니다.

---

```ts
createIndex(table: Table|string, index: TableIndex): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `index` - TableIndex 객체

새로운 인덱스를 생성합니다.

---

```ts
createIndices(table: Table|string, indices: TableIndex[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `indices` - TableIndex 객체의 배열

새로운 인덱스들을 생성합니다.

---

```ts
dropIndex(table: Table|string, index: TableIndex|string): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `index` - TableIndex 객체 또는 인덱스 이름

인덱스를 삭제합니다.

---

```ts
dropIndices(table: Table|string, indices: TableIndex[]): Promise<void>
```

- `table` - 테이블 객체 또는 이름
- `indices` - TableIndex 객체의 배열

인덱스들을 삭제합니다.

---

```ts
clearTable(tableName: string): Promise<void>
```

- `tableName` - 테이블 이름

모든 테이블 내용을 지웁니다.

> 참고: 이 작업은 트랜잭션에서 되돌릴 수 없는 SQL의 TRUNCATE 쿼리를 사용합니다.

---

```ts
enableSqlMemory(): void
```

SQL 쿼리가 실행되지 않고 대신 쿼리 실행기 내부의 특수 변수에 기억되는 특수 쿼리 실행기 모드를 활성화합니다.
`getMemorySql()` 메소드를 사용하여 기억된 SQL을 얻을 수 있습니다.

---

```ts
disableSqlMemory(): void
```

SQL 쿼리가 실행되지 않는 특수 쿼리 실행기 모드를 비활성화합니다. 이전에 기억된 SQL은 플러시됩니다.

---

```ts
clearSqlMemory(): void
```

기억된 모든 SQL 문을 플러시합니다.

---

```ts
getMemorySql(): SqlInMemory
```

- `upQueries` 및 `downQueries` SQL 문 배열과 함께 `SqlInMemory` 객체를 반환합니다.

메모리에 저장된 SQL을 가져옵니다. SQL의 매개변수는 이미 대체되었습니다.

---

```ts
executeMemoryUpSql(): Promise<void>
```

기억된 up SQL 쿼리를 실행합니다.

---

```ts
executeMemoryDownSql(): Promise<void>
```

기억된 down SQL 쿼리를 실행합니다.

---
