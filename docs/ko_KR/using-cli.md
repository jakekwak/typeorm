# CLI 사용하기

- [CLI 설치하기](#cli-설치하기)
- [새 TypeORM 프로젝트 초기화](#새-typeorm-프로젝트-초기화)
- [Create a new entity](#create-a-new-entity)
- [새 구독자 만들기](#새-구독자-만들기)
- [새 마이그레이션 만들기](#새-마이그레이션-만들기)
- [기존 테이블 스키마에서 마이그레이션 생성](#기존-테이블-스키마에서-마이그레이션-생성)
- [마이그레이션 실행](#마이그레이션-실행)
- [Revert migrations](#revert-migrations)
- [마이그레이션 표시](#마이그레이션-표시)
- [데이터베이스 스키마 동기화](#데이터베이스-스키마-동기화)
- [실제로 실행하지 않고 동기화 데이터베이스 스키마 쿼리 기록](#실제로-실행하지-않고-동기화-데이터베이스-스키마-쿼리-기록)
- [데이터베이스 스키마 삭제](#데이터베이스-스키마-삭제)
- [SQL 쿼리 실행](#sql-쿼리-실행)
- [캐시 지우기](#캐시-지우기)
- [버전 확인](#버전-확인)

## CLI 설치하기
### 엔티티 파일이 자바스크립트에 있는 경우
로컬 typeorm 버전이 있는 경우 설치할 글로벌 버전과 일치하는지 확인하십시오.

`npm i -g typeorm`을 사용하여 typeorm을 전역적으로 설치합니다. 설치할 필요가 없는 경우 각 명령에 `npx typeorm <params>`를 사용하도록 선택할 수도 있습니다.

### 엔티티 파일이 타이프스크립트에 있는 경우
이 CLI 도구는 자바스크립트로 작성되었으며 노드에서 실행됩니다. 엔터티 파일이 typescript에 있는 경우 CLI를 사용하기 전에 해당 파일을 javascript로 변환해야합니다. 자바스크립트만 사용하는 경우 이 섹션을 건너뛸 수 있습니다.

다음과 같이 작업을 쉽게하기 위해 프로젝트에서 ts-node를 설정할 수 있습니다.

ts-node를 전역으로 설치합니다.

```
npm install -g ts-node
```

package.json의 스크립트 섹션 아래에 typeorm 명령 추가

```
"scripts": {
    ...
    "typeorm": "node --require ts-node/register ./node_modules/typeorm/cli.js"
}
```

[module-alias](https://github.com/ilearnio/module-alias)와 같은 더 많은 모듈을 로드하려면 `--require my-module-supporting-register`를 더 추가할 수 있습니다.

그런 다음 다음과 같은 명령을 실행할 수 있습니다.

```
npm run typeorm migration:run
```

대시가 있는 매개변수를 npm 스크립트에 전달해야 하는 경우 - 뒤에 추가해야합니다. 예를 들어 *생성*이 필요한 경우 명령은 다음과 같습니다.
```
npm run typeorm migration:generate -- -n migrationNameHere
```

### 문서를 읽는 방법

설명서의 자세한 내용을 줄이기 위해 다음 섹션에서는 전역적으로 설치된 typeorm CLI를 사용합니다. CLI를 설치한 방법에 따라 명령 시작시 `typeorm`을 `npx typeorm` 또는 `npm run typeorm`으로 바꿀 수 있습니다.

## 새 TypeORM 프로젝트 초기화

이미 모든 것이 설정되어 있는 새 프로젝트를 만들 수 있습니다.

```
typeorm init
```

TypeORM을 사용하여 기본 프로젝트에 필요한 모든 파일을 만듭니다.

* .gitignore
* package.json
* README.md
* tsconfig.json
* ormconfig.json
* src/entity/User.ts
* src/index.ts

그런 다음 `npm install`을 실행하여 모든 종속성을 설치할 수 있습니다. 모든 종속성이 설치되면 `ormconfig.json`을 수정하고 자체 데이터베이스 설정을 삽입해야합니다. 그런 다음 `npm start`를 실행하여 애플리케이션을 실행할 수 있습니다.

모든 파일은 현재 디렉토리에 생성됩니다. 특수 디렉토리에 생성하려면 `--name`을 사용할 수 있습니다.

```
typeorm init --name my-project
```

사용하는 특정 데이터베이스를 지정하려면 `--database`를 사용할 수 있습니다.

```
typeorm init --database mssql
```

Express를 사용하여 기본 프로젝트를 생성할 수도 있습니다.

```
typeorm init --name my-project --express
```

docker를 사용하는 경우 다음을 사용하여 `docker-compose.yml` 파일을 생성할 수 있습니다.

```
typeorm init --docker
```

`typeorm init`는 TypeORM 프로젝트를 설정하는 가장 쉽고 빠른 방법입니다.


## Create a new entity

CLI를 사용하여 새 엔티티를 생성할 수 있습니다.

```
typeorm entity:create -n User
```

여기서 `User`는 엔티티 파일 및 클래스 이름입니다. 명령어를 실행하면 프로젝트의 `entitiesDir`에 비어있는 새 항목이 생성됩니다. 프로젝트의 `entitiesDir`을 설정하려면 연결 옵션에 추가해야합니다.

```
{
    cli: {
        entitiesDir: "src/entity"
    }
}
```

[연결 옵션](./connection-options.md)에 대해 자세히 알아보세요. 서로 다른 디렉터리에 여러 엔터티가 있는 다중모듈 프로젝트 구조가 있는 경우 엔터티를 생성하려는 CLI 명령의 경로를 제공할 수 있습니다.


```
typeorm entity:create -n User -d src/user/entity
```

[엔티티](./entities.md)에 대해 자세히 알아보세요.

## 새 구독자 만들기

CLI를 사용하여 새 구독자를 만들 수 있습니다.

```
typeorm subscriber:create -n UserSubscriber
```

여기서 `UserSubscriber`는 구독자 파일 및 클래스 이름입니다. 다음 명령을 실행하면 프로젝트의 `subscribersDir`에 비어있는 새 구독자가 생성됩니다. `subscribersDir`을 설정하려면 연결 옵션에 추가해야합니다.

```
{
    cli: {
        subscribersDir: "src/subscriber"
    }
}
```

[연결 옵션](./connection-options.md)에 대해 자세히 알아보세요. 다른 디렉토리에 여러 구독자가 있는 다중 모듈 프로젝트 구조가 있는 경우 구독자를 생성할 CLI 명령에 대한 경로를 제공할 수 있습니다.


```
typeorm subscriber:create -n UserSubscriber -d src/user/subscriber
```

[구독자](./listeners-and-subscribers.md)에 대해 자세히 알아보세요.

## 새 마이그레이션 만들기

CLI를 사용하여 새 마이그레이션을 생성할 수 있습니다.

```
typeorm migration:create -n UserMigration
```

여기서 `UserMigration`은 마이그레이션 파일 및 클래스 이름입니다. 명령을 실행하면 프로젝트의 `migrationsDir`에 비어있는 새 마이그레이션이 생성됩니다. `migrationsDir`을 설정하려면 연결 옵션에 추가해야합니다.

```
{
    cli: {
        migrationsDir: "src/migration"
    }
}
```

[연결 옵션](./connection-options.md)에 대해 자세히 알아보세요. 여러 디렉터리에 여러 마이그레이션이 있는 다중모듈 프로젝트 구조가 있는 경우 마이그레이션을 생성하려는 CLI 명령에 대한 경로를 제공할 수 있습니다.

```
typeorm migration:create -n UserMigration -d src/user/migration
```

[마이그레이션](./migrations.md)에 대해 자세히 알아보세요.

## 기존 테이블 스키마에서 마이그레이션 생성

자동 마이그레이션 생성은 새 마이그레이션 파일을 만들고 데이터베이스를 업데이트하기 위해 실행해야하는 모든 SQL 쿼리를 작성합니다.

생성된 변경 사항이 없으면 명령이 코드 1로 종료됩니다.

```
typeorm migration:generate -n UserMigration
```

경험상 각 엔터티 변경 후 마이그레이션을 생성하는 것이 좋습니다.

[마이그레이션](./migrations.md)에 대해 자세히 알아보세요.

## 마이그레이션 실행

보류중인 모든 마이그레이션을 실행하려면 다음 명령을 사용하십시오.

```
typeorm migration:run
```

[마이그레이션](./migrations.md)에 대해 자세히 알아보세요.

## Revert migrations

가장 최근에 실행된 마이그레이션을 되돌리려면 다음 명령을 사용하십시오.

```
typeorm migration:revert
```

이 명령은 마지막으로 실행된 마이그레이션만 실행 취소합니다.
이 명령을 여러번 실행하여 여러 마이그레이션을 되돌릴 수 있습니다.

[마이그레이션](./migrations.md)에 대해 자세히 알아보세요.

## 마이그레이션 표시

모든 마이그레이션 및 실행 여부를 표시하려면 다음 명령을 사용하십시오.

```
typeorm migration:show
```

[X] = 마이그레이션이 실행되었습니다.

[ ] = 마이그레이션이 보류중 / 적용되지 않음

이 명령은 적용되지 않은 마이그레이션이 있는 경우에도 오류 코드를 반환합니다.

## 데이터베이스 스키마 동기화

데이터베이스 스키마를 동기화하려면 다음을 사용하십시오.

```
typeorm schema:sync
```

프로덕션에서이 명령을 실행하는데 주의하십시오. 현명하게 사용하지 않으면 스키마 동기화로 인해 데이터가 손실될 수 있습니다. 프로덕션에서 실행하기 전에 실행할 SQL 쿼리를 확인하십시오.

## 실제로 실행하지 않고 동기화 데이터베이스 스키마 쿼리 기록

`schema: sync`가 실행할 SQL 쿼리를 확인하려면 다음을 사용하십시오.

```
typeorm schema:log
```

## 데이터베이스 스키마 삭제

데이터베이스 스키마를 완전히 삭제하려면 다음을 사용하십시오.

```
typeorm schema:drop
```

이 명령은 데이터베이스에서 데이터를 완전히 제거하므로 프로덕션에서 주의하십시오.

## SQL 쿼리 실행

다음을 사용하여 데이터베이스에서 직접 원하는 SQL 쿼리를 실행할 수 있습니다.

```
typeorm query "SELECT * FROM USERS"
```

## 캐시 지우기

`QueryBuilder` 캐싱을 사용하는 경우 때때로 캐시에 저장된 모든 항목을 지우고 싶을 수 있습니다. 다음 명령을 사용하여 수행할 수 있습니다.

```
typeorm cache:clear
```

## 버전 확인

다음을 실행하여 설치한 typeorm 버전(로컬 및 글로벌 모두)을 확인할 수 있습니다.

```
typeorm version
```
