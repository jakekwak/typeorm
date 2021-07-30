<div align="center">
  <a href="http://typeorm.io/">
    <img src="https://github.com/typeorm/typeorm/raw/master/resources/logo_big.png" width="492" height="228">
  </a>
  <br>
  <br>
	<a href="https://app.circleci.com/pipelines/github/typeorm/typeorm">
		<img src="https://circleci.com/gh/typeorm/typeorm/tree/master.svg?style=shield">
	</a>
	<a href="https://badge.fury.io/js/typeorm">
		<img src="https://badge.fury.io/js/typeorm.svg">
	</a>
	<a href="https://david-dm.org/typeorm/typeorm">
		<img src="https://david-dm.org/typeorm/typeorm.svg">
	</a>
    <a href="https://codecov.io/gh/typeorm/typeorm">
        <img alt="Codecov" src="https://img.shields.io/codecov/c/github/typeorm/typeorm.svg">
    </a>
	<a href="https://join.slack.com/t/typeorm/shared_invite/zt-gej3gc00-hR~L~DqGUJ7qOpGy4SSq3g">
		<img src="https://img.shields.io/badge/chat-on%20slack-blue.svg">
	</a>
  <br>
  <br>
</div>

TypeORM은 NodeJS, Browser, Cordova, PhoneGap, Ionic, React Native, NativeScript, Expo 및 Electron 플랫폼에서 실행할 수있는 [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)입니다. TypeScript 및 JavaScript(ES5, ES6, ES7, ES8)와 함께 사용됩니다. 목표는 항상 최신 JavaScript 기능을 지원하고 추가 기능을 제공하는 것입니다. 몇 개의 테이블이 있는 소규모 애플리케이션부터 여러 데이터베이스가 있는 대규모 엔터프라이즈 애플리케이션에 이르기까지 데이터베이스를 사용하는 모든 종류의 애플리케이션을 개발하는 데 도움이됩니다.

TypeORM은 [Active Record](./docs/active-record-data-mapper.md#what-is-the-active-record-pattern) 및 [Data Mapper](./docs/active-record-data-mapper.md#what-is-the-data-mapper-pattern) 패턴은 현재 존재하는 다른 모든 JavaScript ORM과 달리 가장 생산적인 방식으로 고품질의 느슨하게 결합된 확장가능하고 유지관리 가능한 애플리케이션을 작성할 수 있음을 의미합니다.

 TypeORM은 [Hibernate](http://hibernate.org/orm/), [Doctrine](http://www.doctrine-project.org/) 및 [Entity Framework](https://www.asp.net/entity-framework)와 같은 다른 ORM의 영향을 많이받습니다.

## 특 징

* [DataMapper](./docs/active-record-data-mapper.md#what-is-the-data-mapper-pattern) 및 [ActiveRecord](./docs/active-record-data-mapper.md#what-is-the-active-record-pattern) 모두 지원 (선택 사항)
* 엔티티 및 컬럼
* 데이타베이스별 컬럼 타입
* 엔티티 관리자
* 리포지토리 및 사용자 지정 리포지토리
* 깨끗한 객체 관계형 모델
* 연관 (관계)
* eager 와 lazy 관계
* 단방향, 양방향 및 자기 참조 관계
* 다중 상속 패턴 지원
* 캐스케이드
* indices
* 트랜잭션
* 마이그레이션 및 자동 마이그레이션 생성
* 커넥션 풀링
* 복제(리플리케이션)
* 여러 데이터베이스 연결 사용
* 여러 데이타베이스 타입과 동작
* 교차 데이터베이스 및 교차 스키마 쿼리
* 우아한 구문, 유연하고 강력한 QueryBuilder
* 레프트 조인과 이너 조인
* 조인을 사용하는 쿼리에 대한 적절한 페이지 매김
* 쿼리 캐싱
* 원시 결과 스트리밍
* 로깅
* 리스너 및 구독자(후크)
* 마감 테이블 패턴 지원
* 모델 또는 개별 구성 파일의 스키마 선언
* json / xml / yml / env 형식의 연결 구성
* MySQL / MariaDB / Postgres / CockroachDB / SQLite / Microsoft SQL Server / Oracle / SAP Hana / sql.js 지원
* MongoDB NoSQL 데이터베이스 지원
* NodeJS / Browser / Ionic / Cordova / React Native / NativeScript / Expo / Electron 플랫폼에서 작동
* TypeScript 및 JavaScript 지원
* 생성된 코드는 성능이 뛰어나고 유연하며 깨끗하고 유지 관리가 가능합니다.
* 가능한 모든 모범 사례를 따릅니다.
* CLI

그리고 더...

TypeORM을 사용하면 모델이 다음과 같이 보입니다.

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    age: number;

}
```

그리고 도메인 로직은 다음과 같습니다.

```typescript
const repository = connection.getRepository(User);

const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.age = 25;
await repository.save(user);

const allUsers = await repository.find();
const firstUser = await repository.findOne(1); // find by id
const timber = await repository.findOne({ firstName: "Timber", lastName: "Saw" });

await repository.remove(timber);
```

또는 `ActiveRecord` 구현을 선호하는 경우 다음과 같이 사용할 수도 있습니다.

```typescript
import { Entity, PrimaryGeneratedColumn, Column, BaseEntity } from "typeorm";

@Entity()
export class User extends BaseEntity {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    age: number;

}
```

도메인 로직은 다음과 같습니다.

```typescript
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.age = 25;
await user.save();

const allUsers = await User.find();
const firstUser = await User.findOne(1);
const timber = await User.findOne({ firstName: "Timber", lastName: "Saw" });

await timber.remove();
```

## 설 치


1. npm 패키지를 설치합니다.

    `npm install typeorm --save`

2. `reflect-metadata` shim을 설치해야합니다.

    `npm install reflect-metadata --save`

    앱의 전역 위치 (예: `app.ts`)에서 가져옵니다.

    `import "reflect-metadata";`

3. 노드 타입을 설치해야 할 수도 있습니다.

    `npm install @types/node --save-dev`

4. 데이터베이스 드라이버를 설치합니다.

    * **MySQL** 또는 **MariaDB** 용

        `npm install mysql --save` (대신 `mysql2`를 설치할 수도 있습니다.)

    * **PostgreSQL** 또는 **CockroachDB** 용

        `npm install pg --save`

    * **SQLite** 용

        `npm install sqlite3 --save`

    * **Microsoft SQL Server** 용

        `npm install mssql --save`

    * **sql.js** 용

        `npm install sql.js --save`

    * **Oracle** 용

        `npm install oracledb --save`

        Oracle 드라이버가 작동하도록하려면 [해당](https://github.com/oracle/node-oracledb) 사이트의 설치 지침을 따라야합니다.

    * **SAP Hana** 용

        ```
        npm i @sap/hana-client
		npm i hdb-pool
        ```

        *[Neptune Software](https://www.neptune-software.com/)의 후원으로 SAP Hana 지원이 가능해졌습니다.*

    * **MongoDB** 용 (실험적인)

        `npm install mongodb --save`

    * **NativeScript**, **react-native** 및 **Cordova** 용

        [지원 플랫폼 문서](./docs/supported-platforms.md) 확인

    사용하는 데이터베이스에 따라 *하나* 만 설치하십시오.


##### TypeScript 설정

또한 TypeScript 버전 **3.3** 이상을 사용하고 있으며 `tsconfig.json`에서 다음 설정을 활성화했는지 확인하십시오.

```json
"emitDecoratorMetadata": true,
"experimentalDecorators": true,
```

컴파일러 옵션의 `lib` 섹션에서 `es6`을 활성화하거나 `@types`에서 `es6-shim`을 설치해야 할 수도 있습니다.

## 시작하기

TypeORM을 시작하는 가장 빠른 방법은 CLI 명령을 사용하여 시작 프로젝트를 생성하는 것입니다. 빠른 시작은 NodeJS 애플리케이션에서 TypeORM을 사용하는 경우에만 작동합니다. 다른 플랫폼을 사용하는 경우 [단계별 가이드](#단계별-가이드)로 진행하세요.

먼저 TypeORM을 전역으로 설치하십시오.

```
npm install typeorm -g
```

그런 다음 새 프로젝트를 만들 디렉터리로 이동하여 다음 명령을 실행합니다.

```
typeorm init --name MyProject --database mysql
```

여기서 `name`은 프로젝트 이름이고 `database`는 사용할 데이터베이스입니다. 데이터베이스는 다음 값 중 하나일 수 있습니다.
`mysql`, `mariadb`, `postgres`, `cockroachdb`, `sqlite`, `mssql`, `oracle`, `mongodb`, `cordova`, `react-native`, `expo`, `nativescript`.

이 명령은 `MyProject` 디렉토리에 다음 파일로 새 프로젝트를 생성합니다.

```
MyProject
├── src              // TypeScript 코드 위치
│   ├── entity       // 엔티티(데이터베이스 모델)가 저장되는 위치
│   │   └── User.ts  // 샘플 엔티티
│   ├── migration    // 마이그레이션이 저장되는 장소
│   └── index.ts     // 애플리케이션의 시작점
├── .gitignore       // 표준 gitignore 파일
├── ormconfig.json   // ORM 및 데이터베이스 연결 구성
├── package.json     // 노드 모듈 종속성
├── README.md        // 간단한 readme 파일
└── tsconfig.json    // TypeScript 컴파일러 옵션
```

> 기존 노드 프로젝트에서 `typeorm init`를 실행할 수도 있지만 주의하세요. 이미 가지고 있는 일부 파일을 덮어 쓸 수 있습니다.

다음 단계는 새 프로젝트 종속성을 설치하는 것입니다.

```
cd MyProject
npm install
```

설치가 진행되는 동안 `ormconfig.json` 파일을 편집하고 거기에 자신의 데이터베이스 연결 구성 옵션을 넣으십시오.

```json
{
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test",
   "synchronize": true,
   "logging": false,
   "entities": [
      "src/entity/**/*.ts"
   ],
   "migrations": [
      "src/migration/**/*.ts"
   ],
   "subscribers": [
      "src/subscriber/**/*.ts"
   ]
}
```

특히 대부분의 경우 `host`, `username`, `password`, `database` 및 `port` 옵션만 구성하면됩니다.

구성을 마치고 모든 노드 모듈이 설치되면 애플리케이션을 실행할 수 있습니다.

```
npm start
```

이제 애플리케이션이 성공적으로 실행되고 새 사용자를 데이터베이스에 삽입해야 합니다. 이 프로젝트로 계속 작업하고 필요한 다른 모듈을 통합하고 더 많은 항목을 만들 수 있습니다.

> `typeorm init --name MyProject --database mysql --express` 명령을 실행하여 Express가 설치된 고급 프로젝트를 생성 할 수 있습니다.

> `typeorm init --name MyProject --database postgres --docker` 명령을 실행하여 docker-compose 파일을 생성 할 수 있습니다.

## 단계별 가이드

ORM에서 무엇을 기대하십니까? 우선, 당신은 그것이 당신을 위해 데이터베이스 테이블을 생성하고 거의 유지하기 어려운 SQL 쿼리를 많이 작성하지 않고도 데이터를 찾고(find) / 삽입(insert) / 업데이트(update) / 삭제(delete)할 것으로 기대하고 있습니다. 이 가이드는 TypeORM을 처음부터 설정하고 ORM에서 기대하는 작업을 수행하는 방법을 보여줍니다.

### 모델 생성

데이터베이스 작업은 테이블 생성에서 시작됩니다.
TypeORM에게 데이터베이스 테이블을 생성하도록 어떻게 지시합니까?
대답은 - 모델을 통해입니다.
앱의 모델은 데이터베이스 테이블입니다.

예를 들어 `Photo`모델이 있습니다.

```typescript
export class Photo {
    id: number;
    name: string;
    description: string;
    filename: string;
    views: number;
    isPublished: boolean;
}
```

그리고 데이터베이스에 사진을 저장하려고합니다. 데이터베이스에 항목을 저장하려면 먼저 데이터베이스 테이블이 필요하며 데이터베이스 테이블은 모델에서 생성됩니다. 모든 모델이 아니라 *엔티티*로 정의한 모델만.

### 엔터티 만들기

*Entity*는 `@Entity` 데코레이터로 장식된 모델입니다.
이러한 모델에 대한 데이터베이스 테이블이 생성됩니다.
TypeORM으로 모든 곳에서 엔티티와 작업합니다.
로드 / 삽입 / 업데이트 / 제거 및 기타 작업을 수행 할 수 있습니다.

`Photo` 모델을 엔티티로 만들어 보겠습니다.

```typescript
import { Entity } from "typeorm";

@Entity()
export class Photo {
    id: number;
    name: string;
    description: string;
    filename: string;
    views: number;
    isPublished: boolean;
}
```

이제 `Photo` 엔터티에 대한 데이터베이스 테이블이 생성되고 앱의 어느 곳에서나 작업할 수 있습니다.
데이터베이스 테이블을 만들었지만 컬럼없이 존재할 수 있는 테이블은 무엇입니까?
데이터베이스 테이블에 몇 개의 컬럼을 생성해 보겠습니다.

### 테이블 컬럼 추가

데이터베이스 열을 추가하려면 `@Column` 데코레이터를 사용하여 만들려는 항목의 속성을 열로 데코레이션하면 됩니다.

```typescript
import { Entity, Column } from "typeorm";

@Entity()
export class Photo {

    @Column()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

이제 `id`, `name`, `description`, `filename`, `views` 및 `isPublished` 컬럼이 `photo` 테이블에 추가됩니다. 데이터베이스의 컬럼 타입은 사용한 속성타입(예: `number`는 `integer`로, `string`은 `varchar`로, `boolean`은 `bool`로 변환됩니다. 그러나 `@Column` 데코레이터에 컬럼타입을 명시적으로 지정하여 데이터베이스에서 지원하는 모든 컬럼타입을 사용할 수 있습니다.

컬럼이 있는 데이터베이스 테이블을 생성했지만 한가지가 남았습니다. 각 데이터베이스 테이블에는 기본 키가 있는 컬럼이 있어야 합니다.

### 기본 컬럼 만들기

각 항목에는 **반드시** 하나 이상의 기본 키 컬럼이 있습니다.
이것은 요구사항이며 피할 수 없습니다. 컬럼을 기본 키로 만들려면 `@PrimaryColumn` 데코레이터를 사용해야 합니다.

```typescript
import { Entity, Column, PrimaryColumn } from "typeorm";

@Entity()
export class Photo {

    @PrimaryColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### 자동 생성 컬럼 만들기

이제 id 컬럼이 자동 생성되기를 원한다고 가정해 보겠습니다 (자동 증가 / 시퀀스 / 직렬 / 생성된 ID 컬럼이라고 함). 그렇게하려면 `@PrimaryColumn` 데코레이터를 `@PrimaryGeneratedColumn` 데코레이터로 변경해야합니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### 컬럼 데이터 타입

다음으로 데이터 타입을 수정하겠습니다. 기본적으로 문자열은 데이터베이스 타입에 따라 varchar(255)와 유사한 타입으로 매핑됩니다. 숫자는 정수와 유사한 타입(데이터베이스 타입에 따라 다름)에 맵핑됩니다. 모든 컬럼이 varchar 또는 정수로 제한되는 것을 원하지 않습니다. 올바른 데이터 타입을 설정해 보겠습니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        length: 100
    })
    name: string;

    @Column("text")
    description: string;

    @Column()
    filename: string;

    @Column("double")
    views: number;

    @Column()
    isPublished: boolean;
}
```

컬럼 타입은 데이터베이스에 따라 다릅니다. 데이터베이스에서 지원하는 모든 컬럼 타입을 설정할 수 있습니다. 지원되는 컬럼 타입에 대한 자세한 내용은 [여기](./docs/entities.md#column-types)에서 찾을 수 있습니다.

### 데이터베이스에 대한 연결 생성

이제 엔터티가 생성되면 `index.ts`(또는 `app.ts`라고 부르는 파일) 파일을 만들고 여기에 연결을 설정하겠습니다.

```typescript
import "reflect-metadata";
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "test",
    entities: [
        Photo
    ],
    synchronize: true,
    logging: false
}).then(connection => {
    // here you can start to work with your entities
}).catch(error => console.log(error));
```

이 예에서는 MySQL을 사용하고 있지만 지원되는 다른 데이터베이스를 사용할 수 있습니다. 다른 데이터베이스를 사용하려면 옵션의 `type`을 사용중인 데이터베이스 타입으로 변경하십시오. `mysql`, `mariadb`, `postgres`, `cockroachdb`, `sqlite`, `mssql`, `oracle`, `cordova`, `nativescript`, `react-native`, `expo` 또는 `mongodb`. 또한 자신의 호스트, 포트, 사용자 이름, 암호 및 데이터베이스 설정을 사용해야합니다.

이 연결에 대한 엔티티 목록에 사진 엔티티를 추가했습니다. 연결에서 사용중인 각 엔티티가 여기에 나열되어야 합니다.

`synchronize`를 설정하면 애플리케이션을 실행할 때마다 엔티티가 데이터베이스와 동기화됩니다.

### 디렉토리에서 모든 엔티티 로드

나중에 더 많은 엔터티를 만들 때 구성의 엔터티에 추가해야합니다. 이것은 그다지 편리하지 않으므로 대신 모든 엔티티가 연결되고 연결에 사용되는 전체 디렉토리를 설정할 수 있습니다.

```typescript
import { createConnection } from "typeorm";

createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "test",
    entities: [
        __dirname + "/entity/*.js"
    ],
    synchronize: true,
}).then(connection => {
    // here you can start to work with your entities
}).catch(error => console.log(error));
```

그러나 이 접근 방식에 주의하십시오. `ts-node`를 사용하는 경우 대신 `.ts` 파일에 대한 경로를 지정해야합니다. `outDir`를 사용하는 경우 outDir 디렉토리 내에 `.js` 파일의 경로를 지정해야합니다. `outDir`을 사용중이고 엔티티를 제거하거나 이름을 바꿀 때 `outDir` 디렉토리를 지우고 프로젝트를 다시 컴파일하십시오. 소스 `.ts` 파일을 제거할 때 컴파일된 `.js` 버전이 출력 디렉토리에서 제거되지 않고 여전히 `outDir` 디렉토리에 있기 때문에 TypeORM에 의해 로드됩니다.

### 애플리케이션 실행

이제 `index.ts`를 실행하면 데이터베이스와의 연결이 초기화되고 사진에 대한 데이터베이스 테이블이 생성됩니다.


```shell
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(100) |                            |
| description | text         |                            |
| filename    | varchar(255) |                            |
| views       | int(11)      |                            |
| isPublished | boolean      |                            |
+-------------+--------------+----------------------------+
```

### 사진 생성 및 데이터베이스에 삽입

이제 새 사진을 만들어 데이터베이스에 저장해 보겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(connection => {

    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;

    return connection.manager
            .save(photo)
            .then(photo => {
                console.log("Photo has been saved. Photo id is", photo.id);
            });

}).catch(error => console.log(error));
```

엔티티가 저장되면 새로 생성 된 ID를 받게됩니다. `save`메소드는 전달한 동일한 객체의 인스턴스를 반환합니다. 객체의 새 복사본이 아니며 "id"를 수정하고 반환합니다.

### async/await 구문 사용

최신 ES8(ES2017) 기능을 활용하고 대신 async/await 구문을 사용하겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;

    await connection.manager.save(photo);
    console.log("Photo has been saved");

}).catch(error => console.log(error));
```

### 엔티티 관리자 사용

방금 새 사진을 만들어 데이터베이스에 저장했습니다. 우리는 그것을 저장하기 위해 `EntityManager`를 사용했습니다. 엔티티 관리자를 사용하면 앱의 모든 엔티티를 조작할 수 있습니다. 예를 들어 저장된 엔티티를 로드해 보겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let savedPhotos = await connection.manager.find(Photo);
    console.log("All photos from the db: ", savedPhotos);

}).catch(error => console.log(error));
```

`savedPhotos`는 데이터베이스에서 로드된 데이터가 있는 Photo 객체의 배열입니다.

[EntityManager](./docs/working-with-entity-manager.md)에 대해 자세히 알아보십시오.

### 저장소(리포지토리) 사용

이제 코드를 리팩터링하고 `EntityManager` 대신 `Repository`를 사용하겠습니다. 각 엔티티에는 엔티티와 함께 모든 작업을 처리하는 자체 저장소가 있습니다. 엔티티를 많이 다룰 때 저장소는 EntityManagers보다 사용하기 더 편리합니다.


```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;

    let photoRepository = connection.getRepository(Photo);

    await photoRepository.save(photo);
    console.log("Photo has been saved");

    let savedPhotos = await photoRepository.find();
    console.log("All photos from the db: ", savedPhotos);

}).catch(error => console.log(error));
```

[여기](./docs/working-with-repository.md)에서 저장소에 대해 자세히 알아보세요.

### 데이터베이스에서 로드

Repository를 사용하여 더 많은 로드 작업을 시도해 보겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let allPhotos = await photoRepository.find();
    console.log("All photos from the db: ", allPhotos);

    let firstPhoto = await photoRepository.findOne(1);
    console.log("First photo from the db: ", firstPhoto);

    let meAndBearsPhoto = await photoRepository.findOne({ name: "Me and Bears" });
    console.log("Me and Bears photo from the db: ", meAndBearsPhoto);

    let allViewedPhotos = await photoRepository.find({ views: 1 });
    console.log("All viewed photos: ", allViewedPhotos);

    let allPublishedPhotos = await photoRepository.find({ isPublished: true });
    console.log("All published photos: ", allPublishedPhotos);

    let [allPhotos, photosCount] = await photoRepository.findAndCount();
    console.log("All photos: ", allPhotos);
    console.log("Photos count: ", photosCount);

}).catch(error => console.log(error));
```

### 데이터베이스에서 업데이트

이제 데이터베이스에서 단일 사진을 로드하고 업데이트하고 저장해 보겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photoToUpdate = await photoRepository.findOne(1);
    photoToUpdate.name = "Me, my friends and polar bears";
    await photoRepository.save(photoToUpdate);

}).catch(error => console.log(error));
```

이제 `id = 1`인 사진이 데이터베이스에서 업데이트됩니다.

### 데이터베이스에서 제거

이제 데이터베이스에서 사진을 제거하겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photoToRemove = await photoRepository.findOne(1);
    await photoRepository.remove(photoToRemove);

}).catch(error => console.log(error));
```

이제 `id = 1`인 사진이 데이터베이스에서 제거됩니다.

### 일대일 관계 만들기

다른 클래스와 일대일 관계를 만들어 보겠습니다. `PhotoMetadata.ts`에 새 클래스를 만들어 보겠습니다. 이 PhotoMetadata 클래스는 사진의 추가 메타정보를 포함해야합니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToOne, JoinColumn } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class PhotoMetadata {

    @PrimaryGeneratedColumn()
    id: number;

    @Column("int")
    height: number;

    @Column("int")
    width: number;

    @Column()
    orientation: string;

    @Column()
    compressed: boolean;

    @Column()
    comment: string;

    @OneToOne(type => Photo)
    @JoinColumn()
    photo: Photo;
}
```

여기서는 `@OneToOne`이라는 새로운 데코레이터를 사용하고 있습니다. 이를 통해 두 개체간에 일대일 관계를 만들 수 있습니다. `type => Photo`는 관계를 맺고자하는 엔티티의 클래스를 반환하는 함수입니다. 언어 특성으로 인해 클래스를 직접 사용하는 대신 클래스를 반환하는 함수를 사용해야합니다. `() => Photo`로 쓸 수도 있지만 코드 가독성을 높이기 위해`type => Photo`를 규칙으로 사용합니다. 타입 변수 자체에는 아무것도 포함되지 않습니다.

또한 관계의 이 쪽이 관계를 소유함을 나타내는 `@JoinColumn` 데코레이터를 추가합니다. 관계는 단방향 또는 양방향 일 수 있습니다. 관계형의 한쪽만 소유할 수 있습니다. 관계의 소유자 측에서 `@JoinColumn` 데코레이터를 사용해야 합니다.

앱을 실행하면 새로 생성된 테이블이 표시되며 여기에는 사진 관계에 대한 외래 키가 있는 컬럼이 포함됩니다.

```shell
+-------------+--------------+----------------------------+
|                     photo_metadata                      |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| height      | int(11)      |                            |
| width       | int(11)      |                            |
| comment     | varchar(255) |                            |
| compressed  | boolean      |                            |
| orientation | varchar(255) |                            |
| photoId     | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

### 일대일 관계 저장

이제 사진과 메타 데이터를 저장하고 서로 연결해 보겠습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    // create a photo
    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.isPublished = true;

    // create a photo metadata
    let metadata = new PhotoMetadata();
    metadata.height = 640;
    metadata.width = 480;
    metadata.compressed = true;
    metadata.comment = "cybershoot";
    metadata.orientation = "portrait";
    metadata.photo = photo; // this way we connect them

    // get entity repositories
    let photoRepository = connection.getRepository(Photo);
    let metadataRepository = connection.getRepository(PhotoMetadata);

    // first we should save a photo
    await photoRepository.save(photo);

    // photo is saved. Now we need to save a photo metadata
    await metadataRepository.save(metadata);

    // done
    console.log("Metadata is saved, and relation between metadata and photo is created in the database too");

}).catch(error => console.log(error));
```

### 관계의 반대쪽

관계는 단방향 또는 양방향일 수 있습니다. 현재 PhotoMetadata와 Photo 간의 관계는 단방향입니다. 관계의 소유자는 PhotoMetadata이며 Photo는 PhotoMetadata에 대해 아무것도 모릅니다. 이로 인해 사진측에서 PhotoMetadata에 액세스하는 것이 복잡해집니다. 이 문제를 해결하려면 역 관계를 추가하고 PhotoMetadata와 Photo 간의 관계를 양방향으로 만들어야합니다. 엔티티를 수정해 보겠습니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToOne, JoinColumn } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class PhotoMetadata {

    /* ... other columns */

    @OneToOne(type => Photo, photo => photo.metadata)
    @JoinColumn()
    photo: Photo;
}
```

```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToOne } from "typeorm";
import { PhotoMetadata } from "./PhotoMetadata";

@Entity()
export class Photo {

    /* ... other columns */

    @OneToOne(type => PhotoMetadata, photoMetadata => photoMetadata.photo)
    metadata: PhotoMetadata;
}
```

`photo => photo.metadata`는 관계의 반대쪽 이름을 반환하는 함수입니다. 여기서는 Photo 클래스의 메타 데이터 속성이 Photo 클래스에 PhotoMetadata를 저장하는 위치임을 보여줍니다. 사진의 속성을 반환하는 함수를 전달하는 대신` "metadata"`와 같이 단순히 문자열을`@ OneToOne` 데코레이터에 전달할 수 있습니다. 그러나 우리는 리팩토링을 더 쉽게 만들기 위해이 함수 유형 접근 방식을 사용했습니다.

관계의 한쪽에만`@ JoinColumn` 데코레이터를 사용해야합니다. 이 데코레이터를 두는 쪽이 관계의 소유 쪽이 될 것입니다. 관계의 소유 측에는 데이터베이스의 외래 키가있는 열이 포함됩니다.

### 관계와 함께 객체 로드

이제 사진과 사진 메타 데이터를 단일 쿼리로 로드하겠습니다. 두가지 방법이 있습니다.`find*` 메소드를 사용하거나 `QueryBuilder` 기능을 사용하는 것입니다. 먼저 `find*`메소드를 사용해 보겠습니다. `find*` 메소드를 사용하면 `FindOneOptions` / `FindManyOptions` 인터페이스로 객체를 지정할 수 있습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photoRepository = connection.getRepository(Photo);
    let photos = await photoRepository.find({ relations: ["metadata"] });

}).catch(error => console.log(error));
```

여기에서 사진에는 데이터베이스의 사진 배열이 포함되고 각 사진에는 사진 메타데이터가 포함됩니다. [이 문서](./docs/find-options.md)에서 찾기 옵션에 대해 자세히 알아보세요.

찾기 옵션을 사용하는 것은 훌륭하고 간단하지만 더 복잡한 쿼리가 필요한 경우 대신 `QueryBuilder`를 사용해야 합니다. `QueryBuilder`를 사용하면 보다 복잡한 쿼리를 우아한 방식으로 사용할 수 있습니다.

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photos = await connection
            .getRepository(Photo)
            .createQueryBuilder("photo")
            .innerJoinAndSelect("photo.metadata", "metadata")
            .getMany();


}).catch(error => console.log(error));
```

`QueryBuilder`는 거의 모든 복잡한 SQL 쿼리를 생성하고 실행할 수 있습니다. `QueryBuilder`로 작업할 때 SQL 쿼리를 생성하는 것처럼 생각하십시오. 이 예에서 "photo"및 "metadata"는 선택한 사진에 적용된 별칭입니다. 별칭을 사용하여 선택한 데이터의 열과 속성에 액세스합니다.

### 캐스케이드를 사용하여 관련 개체 자동 저장

We can setup cascade options in our relations, in the cases when we want our related object to be saved whenever the other object is saved.
Let's change our photo's `@OneToOne` decorator a bit:

```typescript
export class Photo {
    /// ... other columns

    @OneToOne(type => PhotoMetadata, metadata => metadata.photo, {
        cascade: true,
    })
    metadata: PhotoMetadata;
}
```

`cascade`를 사용하면 사진을 별도로 저장하지 않고 메타데이터 객체를 별도로 저장할 수 있습니다. 이제 사진 객체를 간단히 저장할 수 있으며 계단식 옵션으로 인해 메타데이터 객체가 자동으로 저장됩니다.

```typescript
createConnection(options).then(async connection => {

    // create photo object
    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.isPublished = true;

    // create photo metadata object
    let metadata = new PhotoMetadata();
    metadata.height = 640;
    metadata.width = 480;
    metadata.compressed = true;
    metadata.comment = "cybershoot";
    metadata.orientation = "portrait";

    photo.metadata = metadata; // this way we connect them

    // get repository
    let photoRepository = connection.getRepository(Photo);

    // saving a photo also save the metadata
    await photoRepository.save(photo);

    console.log("Photo is saved, photo metadata is saved too.")

}).catch(error => console.log(error));
```

이제 이전과 같이 메타데이터의 `photo`속성 대신 사진의 `metadata`속성을 설정했습니다. `cascade`기능은 사진을 사진의 메타 데이터에 연결하는 경우에만 작동합니다. 메타데이터 쪽을 설정하면 메타데이터가 자동으로 저장되지 않습니다.

### 다대일 / 일대다 관계 생성

다대일 / 일대다 관계를 만들어 보겠습니다. 사진에 한명의 작성자가 있고 각 작성자가 여러장의 사진을 가질 수 있다고 가정해 보겠습니다. 먼저 `Author` 클래스를 만들어 보겠습니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToMany, JoinColumn } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class Author {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(type => Photo, photo => photo.author) // note: we will create author property in the Photo class below
    photos: Photo[];
}
```

`Author`는 관계의 역면을 포함합니다. `OneToMany`는 항상 관계의 반대편이며 관계의 다른쪽에 `ManyToOne`이 없으면 존재할 수 없습니다.

이제 관계의 소유자 측을 Photo 엔티티에 추가해 보겠습니다.

```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from "typeorm";
import { PhotoMetadata } from "./PhotoMetadata";
import { Author } from "./Author";

@Entity()
export class Photo {

    /* ... other columns */

    @ManyToOne(type => Author, author => author.photos)
    author: Author;
}
```

다대일 / 일대다 관계에서 소유자 측은 항상 다대일입니다. 이는 `@ManyToOne`을 사용하는 클래스가 관련 객체의 id를 저장한다는 의미입니다.

애플리케이션을 실행하면 ORM이 `author` 테이블을 생성합니다.


```shell
+-------------+--------------+----------------------------+
|                          author                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

또한 `photo` 테이블을 수정하여 새 `author` 컬럼을 추가하고 이에 대한 외래키를 만듭니다.

```shell
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| description | varchar(255) |                            |
| filename    | varchar(255) |                            |
| isPublished | boolean      |                            |
| authorId    | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

### 다대다 관계 생성

다대일 / 다대다 관계를 만들어 봅시다. 사진이 여러 앨범에 있을 수 있고 각 앨범에 여러 사진이 포함될 수 있다고 가정해 보겠습니다. `Album` 클래스를 만들어 보겠습니다.

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";

@Entity()
export class Album {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(type => Photo, photo => photo.albums)
    @JoinTable()
    photos: Photo[];
}
```

관계의 소유자측임을 지정하려면 `@JoinTable`이 필요합니다.

이제 `Photo` 클래스에 우리 관계의 반대쪽을 추가해 보겠습니다.

```typescript
export class Photo {
    /// ... other columns

    @ManyToMany(type => Album, album => album.photos)
    albums: Album[];
}
```

애플리케이션을 실행하면 ORM이 **album_photos_photo_albums** *정션 테이블*을 생성합니다.

```shell
+-------------+--------------+----------------------------+
|                album_photos_photo_albums                |
+-------------+--------------+----------------------------+
| album_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
| photo_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

ORM에서 연결에 `Album` 클래스를 등록하는 것을 잊지 마십시오.

```typescript
const options: ConnectionOptions = {
    // ... other options
    entities: [Photo, PhotoMetadata, Author, Album]
};
```

이제 데이터베이스에 앨범과 사진을 삽입해 보겠습니다.

```typescript
let connection = await createConnection(options);

// create a few albums
let album1 = new Album();
album1.name = "Bears";
await connection.manager.save(album1);

let album2 = new Album();
album2.name = "Me";
await connection.manager.save(album2);

// create a few photos
let photo = new Photo();
photo.name = "Me and Bears";
photo.description = "I am near polar bears";
photo.filename = "photo-with-bears.jpg";
photo.views = 1
photo.isPublished = true
photo.albums = [album1, album2];
await connection.manager.save(photo);

// now our photo is saved and albums are attached to it
// now lets load them:
const loadedPhoto = await connection
    .getRepository(Photo)
    .findOne(1, { relations: ["albums"] });
```

`loadedPhoto`는 다음과 같습니다.

```typescript
{
    id: 1,
    name: "Me and Bears",
    description: "I am near polar bears",
    filename: "photo-with-bears.jpg",
    albums: [{
        id: 1,
        name: "Bears"
    }, {
        id: 2,
        name: "Me"
    }]
}
```

### QueryBuilder 사용

QueryBuilder를 사용하여 거의 모든 복잡성의 SQL 쿼리를 작성할 수 있습니다. 예를 들어 다음과 같이 할 수 있습니다.

```typescript
let photos = await connection
    .getRepository(Photo)
    .createQueryBuilder("photo") // first argument is an alias. Alias is what you are selecting - photos. You must specify it.
    .innerJoinAndSelect("photo.metadata", "metadata")
    .leftJoinAndSelect("photo.albums", "album")
    .where("photo.isPublished = true")
    .andWhere("(photo.name = :photoName OR photo.name = :bearName)")
    .orderBy("photo.id", "DESC")
    .skip(5)
    .take(10)
    .setParameters({ photoName: "My", bearName: "Mishka" })
    .getMany();
```

이 쿼리는 "My"또는 "Mishka" 이름으로 게시된 모든 사진을 선택합니다.
위치 5(페이지 매기기 오프셋)에서 결과를 선택하고 10개의 결과(페이지 매기기 제한)만 선택합니다.
선택 결과는 내림차순으로 id로 정렬됩니다.
사진의 앨범은 레프트 조인되고 메타 데이터는 이너 조인됩니다.

애플리케이션에서 쿼리 작성기를 많이 사용하게됩니다.
QueryBuilder [여기](./docs/select-query-builder.md)에 대해 자세히 알아보십시오.

## 샘플

사용 예는 [sample](https://github.com/typeorm/typeorm/tree/master/sample)의 샘플을 참조하세요.

복제하고 시작할 수있는 몇 가지 저장소가 있습니다.

* [TypeScript와 함께 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/typescript-example)
* [JavaScript에서 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/javascript-example)
* [JavaScript 및 Babel과 함께 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/babel-example)
* [브라우저에서 TypeScript 및 SystemJS와 함께 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/browser-example)
* [Express 및 TypeORM 사용 방법 예](https://github.com/typeorm/typescript-express-example)
* [Koa 및 TypeORM 사용 방법 예](https://github.com/typeorm/typescript-koa-example)
* [MongoDB에서 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/mongo-typescript-example)
* [Cordova/PhoneGap 앱에서 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/cordova-example)
* [Ionic 앱에서 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/ionic-example)
* [React Native에서 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/react-native-example)
* [Nativescript-Vue와 함께 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/nativescript-vue-typeorm-sample)
* [Nativescript-Angular와 함께 TypeORM을 사용하는 방법의 예](https://github.com/betov18x/nativescript-angular-typeorm-example)
* [JavaScript를 사용하여 Electron에서 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/electron-javascript-example)
* [TypeScript를 사용하여 Electron과 함께 TypeORM을 사용하는 방법의 예](https://github.com/typeorm/electron-typescript-example)

## 익스텐션

TypeORM 작업을 단순화하고 다른 모듈과 통합하는 몇 가지 확장이 있습니다.

* [TypeORM + GraphQL 프레임워크](http://vesper-framework.com)
* [TypeORM 통합](https://github.com/typeorm/typeorm-typedi-extensions)과 [TypeDI](https://github.com/pleerock/typedi)
* [TypeORM 통합](https://github.com/typeorm/typeorm-routing-controllers-extensions) 과 [routing-controllers](https://github.com/pleerock/routing-controllers)
* 기존 데이터베이스에서 모델 생성 - [typeorm-model-generator](https://github.com/Kononnable/typeorm-model-generator)
* 픽스쳐 로더 - [typeorm-fixtures-cli](https://github.com/RobinCK/typeorm-fixtures)
* ER 다이어그램 생성기 - [typeorm-uml](https://github.com/eugene-manuilov/typeorm-uml/)
* Create/Drop database - [typeorm-extension](https://github.com/Tada5hi/typeorm-extension)

## 기여하기

[여기](https://github.com/typeorm/typeorm/blob/master/CONTRIBUTING.md)에서 기여에 대해 알아보고 [여기](https://github.com/typeorm/typeorm/blob/master/DEVELOPER.md)에서 개발 환경을 설정하는 방법을 알아보세요.

이 프로젝트는 다음과 같은 기여를 한 모든 사람들에게 감사합니다.

<a href="https://github.com/typeorm/typeorm/graphs/contributors"><img src="https://opencollective.com/typeorm/contributors.svg?width=890&showBtn=false" /></a>

## 스폰서

오픈 소스는 어렵고 시간이 많이 걸립니다. TypeORM의 미래에 투자하고 싶다면 스폰서가되어 핵심 팀이 TypeORM의 개선사항과 새로운 기능에 더 많은 시간을 할애 할 수 있습니다. [스폰서되기](https://opencollective.com/typeorm)

<a href="https://opencollective.com/typeorm" target="_blank"><img src="https://opencollective.com/typeorm/tiers/sponsor.svg?width=890"></a>

## 골드 스폰서

골드 스폰서가 되어 핵심 기여자로부터 프리미엄 기술 지원을 받으십시오. [골드 스폰서되기](https://opencollective.com/typeorm)

<a href="https://opencollective.com/typeorm" target="_blank"><img src="https://opencollective.com/typeorm/tiers/gold-sponsor.svg?width=890"></a>
