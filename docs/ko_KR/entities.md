

# Entities

- [엔티티 란?](#엔티티-란)
- [엔티티 컬럼](#엔티티-컬럼)
  - [기본 컬럼(Primary Columns)](#기본-컬럼primary-columns)
  - [특수 컬럼](#특수-컬럼)
  - [공간 컬럼(Spatial Columns)](#공간-컬럼spatial-columns)
- [컬럼 타입](#컬럼-타입)
  - [`mysql` / `mariadb`의 컬럼 타입](#mysql--mariadb의-컬럼-타입)
  - [`postgres`의 컬럼 타입](#postgres의-컬럼-타입)
  - [`cockroachdb`의 컬럼 타입](#cockroachdb의-컬럼-타입)
  - [`sqlite` / `cordova` / `react-native` / `expo`의 컬럼 타입](#sqlite--cordova--react-native--expo의-컬럼-타입)
  - [`mssql`의 컬럼 타입](#mssql의-컬럼-타입)
  - [`oracle`의 컬럼 타입](#oracle의-컬럼-타입)
  - [`enum` 컬럼 타입](#enum-컬럼-타입)
  - [`set` 컬럼 타입](#set-컬럼-타입)
  - [`simple-array` 컬럼 타입](#simple-array-컬럼-타입)
  - [`simple-json` 컬럼 타입](#simple-json-컬럼-타입)
  - [생성된 값이 있는 컬럼](#생성된-값이-있는-컬럼)
- [컬럼 옵션](#컬럼-옵션)
- [엔티티 상속](#엔티티-상속)
- [트리 엔티티](#트리-엔티티)
  - [인접 목록(Adjacency list)](#인접-목록adjacency-list)
  - [클로저 테이블](#클로저-테이블)

## 엔티티 란?

엔티티는 데이터베이스 테이블(또는 MongoDB를 사용하는 경우 컬렉션)에 매핑되는 클래스입니다. 새 클래스를 정의하여 엔티티를 만들고 `@Entity()`로 표시할 수 있습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

}
```

그러면 다음 데이터베이스 테이블이 생성됩니다.

```shell
+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| firstName   | varchar(255) |                            |
| lastName    | varchar(255) |                            |
| isActive    | boolean      |                            |
+-------------+--------------+----------------------------+
```

기본 엔터티는 열과 관계로 구성됩니다. 각 엔터티에는 기본 컬럼(또는 MongoDB를 사용하는 경우 ObjectId 컬럼)이 있어야 합니다(**MUST**).

각 엔티티는 연결 옵션에 등록되어야합니다.

```typescript
import {createConnection, Connection} from "typeorm";
import {User} from "./entity/User";

const connection: Connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: [User]
});
```

또는 모든 엔티티가 내부에 있는 전체 디렉토리를 지정할 수 있습니다. 그러면 모두 로드됩니다.

```typescript
import {createConnection, Connection} from "typeorm";

const connection: Connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: ["entity/*.js"]
});
```

`User` 엔터티에 대해 대체 테이블 이름을 사용하려면 `@Entity`: `@Entity("my_users")`에 지정할 수 있습니다. 애플리케이션의 모든 데이터베이스 테이블에 기본 접두사를 설정하려면 연결 옵션에서 `entityPrefix`를 지정할 수 있습니다.

엔티티 생성자를 사용할 때 인수는 **선택 사항이어야 합니다**. ORM은 데이터베이스에서 로드할 때 엔티티 클래스의 인스턴스를 생성하므로 생성자 인수를 인식하지 못합니다.

[데코레이터 참조](./decorator-reference.md)에서 `@Entity` 매개변수에 대해 자세히 알아보세요.

## 엔티티 컬럼

데이터베이스 테이블은 컬럼으로 구성되므로 엔티티도 컬럼으로 구성되어야합니다. `@Column`으로 표시한 각 엔티티 클래스 속성은 데이터베이스 테이블 칼럼에 매핑됩니다.

### 기본 컬럼(Primary Columns)

각 항목에는 하나 이상의 기본 컬럼이 있어야합니다. 여러 타입의 기본 컬럼이 있습니다.

* `@PrimaryColumn()` 모든 타입의 값을 취하는 기본 컬럼을 만듭니다. 컬럼 타입을 지정할 수 있습니다. 컬럼 타입을 지정하지 않으면 속성 타입에서 유추됩니다. 아래 예제는 저장하기 전에 수동으로 할당해야 하는 타입으로 `int`를 사용하여 id를 생성합니다.

```typescript
import {Entity, PrimaryColumn} from "typeorm";

@Entity()
export class User {

    @PrimaryColumn()
    id: number;


}
```

* `@PrimaryGeneratedColumn()` 값이 자동 증가 값으로 자동 생성되는 기본 컬럼을 만듭니다. `auto-increment` / `serial` / `sequence` (데이터베이스에 따라 다름)로 `int` 컬럼을 생성합니다. 저장하기 전에 값을 수동으로 할당할 필요가 없습니다. 값이 자동으로 생성됩니다.

```typescript
import {Entity, PrimaryGeneratedColumn} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;


}
```

* `@PrimaryGeneratedColumn("uuid")` 값이 `uuid`로 자동 생성되는 기본 컬럼을 만듭니다. Uuid는 고유한 문자열 ID입니다. 저장하기 전에 값을 수동으로 할당할 필요가 없습니다. 값이 자동으로 생성됩니다.

```typescript
import {Entity, PrimaryGeneratedColumn} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn("uuid")
    id: string;


}
```

복합 기본 컬럼도 가질 수 있습니다.

```typescript
import {Entity, PrimaryColumn} from "typeorm";

@Entity()
export class User {

    @PrimaryColumn()
    firstName: string;

    @PrimaryColumn()
    lastName: string;

}
```

`save`를 사용하여 항목을 저장하면 항상 주어진 항목 ID(또는 ID)를 사용하여 데이터베이스에서 항목을 찾으려고합니다. ID/IDs가 발견되면 데이터베이스에서 이 행을 업데이트합니다. ID/IDs가 있는 행이 없으면 새 행이 삽입됩니다.

ID로 엔티티를 찾으려면`manager.findOne` 또는`repository.findOne`을 사용할 수 있습니다. 예:

```typescript
// 단일 기본 키로 ID로 하나 찾기
const person = await connection.manager.findOne(Person, 1);
const person = await connection.getRepository(Person).findOne(1);

// 복합 기본 키로 ID로 하나 찾기
const user = await connection.manager.findOne(User, { firstName: "Timber", lastName: "Saw" });
const user = await connection.getRepository(User).findOne({ firstName: "Timber", lastName: "Saw" });
```

### 특수 컬럼

추가 기능을 사용할 수 있는 몇가지 특수 컬럼 타입이 있습니다.


* `@CreateDateColumn`은 엔티티 삽입 날짜로 자동 설정되는 특수 컬럼입니다. 이 컬럼은 설정할 필요가 없습니다. 자동으로 설정됩니다.

* `@UpdateDateColumn`은 엔티티 관리자 또는 저장소의 `save`를 호출할 때마다 엔티티의 업데이트 시간으로 자동 설정되는 특수 컬럼입니다. 이 컬럼은 설정할 필요가 없습니다. 자동으로 설정됩니다.

* `@DeleteDateColumn`은 엔티티 관리자 또는 저장소의 소프트 삭제를 호출할 때마다 엔티티의 삭제 시간으로 자동 설정되는 특수 컬럼입니다. 이 컬럼은 설정할 필요가 없습니다. 자동으로 설정됩니다. @DeleteDateColumn이 설정된 경우 기본 범위는 "삭제되지 않음"입니다.

* `@VersionColumn` 엔티티 관리자 또는 저장소의 `save`을 호출할 때마다 엔티티 버전(증분 번호)으로 자동 설정되는 특수 컬럼입니다. 이 컬럼은 설정할 필요가 없습니다. 자동으로 설정됩니다.

### 공간 컬럼(Spatial Columns)

MS SQL, MySQL / MariaDB 및 PostgreSQL은 모두 공간 컬럼을 지원합니다. TypeORM의
각각에 대한 지원은 특히 컬럼에 따라 데이터베이스마다 약간씩 다릅니다. 이름은 데이터베이스마다 다릅니다.

MS SQL 및 MySQL / MariaDB의 TypeORM 지원은 도형이 [잘 알려진 텍스트로 제공 될 것으로 예상합니다.(WKT)](https://en.wikipedia.org/wiki/Well-known_text)이므로 도형(geometry) 컬럼에는 `string` 타입으로 태그를 지정해야합니다.

TypeORM의 PostgreSQL 지원은 [GeoJSON](http://geojson.org/)을 교환 형식으로 사용하므로 [`geojson` 타입](https://www.npmjs.com/package/@types/geojson)를 가져온 후 도형 컬럼에 `object`또는 `Geometry`(또는 하위 클래스, 예: `Point`)로 태그를 지정해야합니다.

TypeORM은 옳은 일을하려고 하지만 언제 값이 삽입되는지 또는 PostGIS 함수의 결과가 지오메트리로 처리되어야 하는지를 항상 결정할 수 있는 것은 아닙니다. 결과적으로 값이 GeoJSON에서 PostGIS `지오메트리`로 변환되고 `json`으로 GeoJSON으로 변환되는 다음과 유사한 코드를 작성할 수 있습니다.

```typescript
const origin = {
  type: "Point",
  coordinates: [0, 0]
};

await getManager()
    .createQueryBuilder(Thing, "thing")
    // 문자열화 된 GeoJSON을 테이블 사양과 일치하는 SRID를 가진 지오메트리로 변환
    .where("ST_Distance(geom, ST_SetSRID(ST_GeomFromGeoJSON(:origin), ST_SRID(geom))) > 0")
    .orderBy({
        "ST_Distance(geom, ST_SetSRID(ST_GeomFromGeoJSON(:origin), ST_SRID(geom)))": {
            order: "ASC"
        }
    })
    .setParameters({
      // GeoJSON 문자열화
      origin: JSON.stringify(origin)
    })
    .getMany();

await getManager()
    .createQueryBuilder(Thing, "thing")
    // 지오메트리 결과를 GeoJSON으로 변환하고 JSON으로 처리
    // (TypeORM이 이를 역 직렬화하도록 알 수 있음)
    .select("ST_AsGeoJSON(ST_Buffer(geom, 0.1))::json geom")
    .from("thing")
    .getMany();
```


## 컬럼 타입

TypeORM은 가장 일반적으로 사용되는 모든 데이터베이스 지원 컬럼 타입을 지원합니다. 컬럼 타입은 데이터베이스 타입에 따라 다릅니다. 이는 데이터베이스 스키마의 모양에 대해 더 많은 유연성을 제공합니다.

컬럼 타입을 `@Column`의 첫번째 매개 변수로 지정하거나 `@Column`의 컬럼 옵션에 지정할 수 있습니다. 예를 들면 다음과 같습니다.

```typescript
@Column("int")
```

또는

```typescript
@Column({ type: "int" })
```

추가 타입 매개변수를 지정하려면 컬럼 옵션을 통해 수행할 수 있습니다.
예를 들면:

```typescript
@Column("varchar", { length: 200 })
```

또는

```typescript
@Column({ type: "int", width: 200 })
```

> `bigint` 타입에 대한 참고사항: SQL 데이터베이스에서 사용되는 `bigint` 컬럼 타입은 일반 `number` 타입에 맞지 않고 대신 `string`에 속성을 매핑합니다.

### `mysql` / `mariadb`의 컬럼 타입

`bit`, `int`, `integer`, `tinyint`, `smallint`, `mediumint`, `bigint`, `float`, `double`,
`double precision`, `dec`, `decimal`, `numeric`, `fixed`, `bool`, `boolean`, `date`, `datetime`,
`timestamp`, `time`, `year`, `char`, `nchar`, `national char`, `varchar`, `nvarchar`, `national varchar`,
`text`, `tinytext`, `mediumtext`, `blob`, `longtext`, `tinyblob`, `mediumblob`, `longblob`, `enum`, `set`,
`json`, `binary`, `varbinary`, `geometry`, `point`, `linestring`, `polygon`, `multipoint`, `multilinestring`,
`multipolygon`, `geometrycollection`

### `postgres`의 컬럼 타입

`int`, `int2`, `int4`, `int8`, `smallint`, `integer`, `bigint`, `decimal`, `numeric`, `real`,
`float`, `float4`, `float8`, `double precision`, `money`, `character varying`, `varchar`,
`character`, `char`, `text`, `citext`, `hstore`, `bytea`, `bit`, `varbit`, `bit varying`,
`timetz`, `timestamptz`, `timestamp`, `timestamp without time zone`, `timestamp with time zone`,
`date`, `time`, `time without time zone`, `time with time zone`, `interval`, `bool`, `boolean`,
`enum`, `point`, `line`, `lseg`, `box`, `path`, `polygon`, `circle`, `cidr`, `inet`, `macaddr`,
`tsvector`, `tsquery`, `uuid`, `xml`, `json`, `jsonb`, `int4range`, `int8range`, `numrange`,
`tsrange`, `tstzrange`, `daterange`, `geometry`, `geography`, `cube`, `ltree`

### `cockroachdb`의 컬럼 타입

`array`, `bool`, `boolean`, `bytes`, `bytea`, `blob`, `date`, `numeric`, `decimal`, `dec`, `float`,
`float4`, `float8`, `double precision`, `real`, `inet`, `int`, `integer`, `int2`, `int8`, `int64`,
`smallint`, `bigint`, `interval`, `string`, `character varying`, `character`, `char`, `char varying`,
`varchar`, `text`, `time`, `time without time zone`, `timestamp`, `timestamptz`, `timestamp without time zone`,
`timestamp with time zone`, `json`, `jsonb`, `uuid`

> Note: CockroachDB returns all numeric data types as `string`. However if you omit column type and define your property as
 `number` ORM will `parseInt` string into number.


### `sqlite` / `cordova` / `react-native` / `expo`의 컬럼 타입

`int`, `int2`, `int8`, `integer`, `tinyint`, `smallint`, `mediumint`, `bigint`, `decimal`,
`numeric`, `float`, `double`, `real`, `double precision`, `datetime`, `varying character`,
`character`, `native character`, `varchar`, `nchar`, `nvarchar2`, `unsigned big int`, `boolean`,
`blob`, `text`, `clob`, `date`

### `mssql`의 컬럼 타입

`int`, `bigint`, `bit`, `decimal`, `money`, `numeric`, `smallint`, `smallmoney`, `tinyint`, `float`,
`real`, `date`, `datetime2`, `datetime`, `datetimeoffset`, `smalldatetime`, `time`, `char`, `varchar`,
`text`, `nchar`, `nvarchar`, `ntext`, `binary`, `image`, `varbinary`, `hierarchyid`, `sql_variant`,
`timestamp`, `uniqueidentifier`, `xml`, `geometry`, `geography`, `rowversion`

### `oracle`의 컬럼 타입

`char`, `nchar`, `nvarchar2`, `varchar2`, `long`, `raw`, `long raw`, `number`, `numeric`, `float`, `dec`,
`decimal`, `integer`, `int`, `smallint`, `real`, `double precision`, `date`, `timestamp`, `timestamp with time zone`,
`timestamp with local time zone`, `interval year to month`, `interval day to second`, `bfile`, `blob`, `clob`,
`nclob`, `rowid`, `urowid`

### `enum` 컬럼 타입

`enum` 컬럼 유형은 `postgres` 및 `mysql`에서 지원됩니다. 가능한 다양한 컬럼 정의가 있습니다.

typescript 열거형 사용:

```typescript
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost"
}

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: UserRole,
        default: UserRole.GHOST
    })
    role: UserRole

}
```
> 참고 : 문자열, 숫자 및 이기종 열거형이 지원됩니다.

열거형 값과 함께 배열 사용:

```typescript
export type UserRoleType = "admin" | "editor" | "ghost",

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: ["admin", "editor", "ghost"],
        default: "ghost"
    })
    role: UserRoleType
}
```

### `set` 컬럼 타입

`set` 컬럼 타입은 `mariadb` 및 `mysql`에서 지원됩니다. 가능한 다양한 컬럼 정의가 있습니다.

typescript 열거형 사용:

```typescript
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost"
}

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "set",
        enum: UserRole,
        default: [UserRole.GHOST, UserRole.EDITOR]
    })
    roles: UserRole[]

}
```

`set` 값이 있는 배열 사용:

```typescript
export type UserRoleType = "admin" | "editor" | "ghost",

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "set",
        enum: ["admin", "editor", "ghost"],
        default: ["ghost", "editor"]
    })
    roles: UserRoleType[]
}
```

### `simple-array` 컬럼 타입

단일 문자열 열에 기본 배열 값을 저장할 수 있는 `simple-array`이라는 특수 컬럼 타입이 있습니다. 모든 값은 쉼표로 구분됩니다. 예를 들면 :

```typescript
@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column("simple-array")
    names: string[];

}
```

```typescript
const user = new User();
user.names = [
    "Alexander",
    "Alex",
    "Sasha",
    "Shurik"
];
```

단일 데이터베이스 열에 `Alexander, Alex, Sasha, Shurik`값으로 저장됩니다. 데이터베이스에서 데이터를 로드할 때 이름은 저장 한 것처럼 이름 배열로 반환됩니다.

작성하는 값에는 쉼표가 **없어야 합니다**.

### `simple-json` 컬럼 타입

JSON.stringify를 통해 데이터베이스에 저장할 수 있는 모든 값을 저장할 수 있는 `simple-json`이라는 특수 컬럼 타입이 있습니다. 데이터베이스에 json 타입이 없고 번거로움 없이 객체를 저장하고 로드하려는 경우 매우 유용합니다.

예를 들면 :

```typescript
@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column("simple-json")
    profile: { name: string, nickname: string };

}
```

```typescript
const user = new User();
user.profile = { name: "John", nickname: "Malkovich" };
```

단일 데이터베이스 컬럼에 `{"name": "John", "nickname": "Malkovich"}`값으로 저장됩니다. 데이터베이스에서 데이터를 로드할 때 JSON.parse를 통해 객체 / 배열 / 기본 요소를 다시 갖게됩니다.

### 생성된 값이 있는 컬럼

`@Generated` 데코레이터를 사용하여 생성된 값으로 컬럼을 생성할 수 있습니다. 예를 들면:

```typescript
@Entity()
export class User {

    @PrimaryColumn()
    id: number;

    @Column()
    @Generated("uuid")
    uuid: string;

}
```

`uuid` 값은 자동으로 생성되어 데이터베이스에 저장됩니다.

"uuid" 외에 "increment" 및 "rowid"(CockroachDB 전용) 생성 타입도 있지만 이러한 타입의 생성을 사용하는 일부 데이터베이스 플랫폼에는 몇가지 제한이 있습니다(예: 일부 데이터베이스는 하나의 증가컬럼만 가질 수 있거나 기본 키가 되려면 증분이 필요합니다.)

## 컬럼 옵션

컬럼 옵션은 항목 컬럼에 대한 추가 옵션을 정의합니다. `@Column`에 컬럼 옵션을 지정할 수 있습니다.

```typescript
@Column({
    type: "varchar",
    length: 150,
    unique: true,
    // ...
})
name: string;
```

`ColumnOptions`에서 사용 가능한 옵션 목록:

* `type: ColumnType` - 컬럼 타입. [위](#column-types)에 나열된 타입중 하나.
* `name: string` - 데이터베이스 테이블의 컬럼 이름입니다. 기본적으로 컬럼 이름은 속성 이름에서 생성됩니다. 자신의 이름을 지정하여 변경할 수 있습니다.

* `length: number` - 컬럼 타입의 길이. 예를 들어 `varchar(150)` 타입을 생성하려면 컬럼 타입과 길이 옵션을 지정합니다.
* `width: number` - 컬럼 타입의 표시 너비. [MySQL 정수 유형](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)에만 사용됩니다.
* `onUpdate: string` - `ON UPDATE` 트리거. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html)에서만 사용됩니다.
* `nullable: boolean` - 데이터베이스에서 `NULL` 또는 `NOT NULL` 컬럼을 만듭니다. 기본적으로 컬럼은 `nullable: false`입니다.
* `update: boolean` - 컬럼 값이 "save" 작업으로 업데이트되었는지 여부를 나타냅니다. false 인 경우 객체를 처음 삽입할 때만이 값을 쓸 수 있습니다. 기본값은 `true`입니다.
* `insert: boolean` - 객체를 처음 삽입할 때 컬럼 값이 설정되었는지 여부를 나타냅니다. 기본값은 `true`입니다.
* `select: boolean` - 쿼리를 만들 때 기본적으로 이 컬럼을 숨길 지 여부를 정의합니다. `false`로 설정하면 컬럼 데이터가 표준 쿼리에 표시되지 않습니다. 기본 열은 `select: true`입니다.
* `default: string` - 데이터베이스 수준 컬럼의 `DEFAULT` 값을 추가합니다.
* `primary: boolean` - 컬럼을 기본으로 표시합니다. `@PrimaryColumn`을 사용하는 경우에도 동일합니다.
* `unique: boolean` - 컬럼을 고유한 컬럼으로 표시합니다 (고유 제약조건 생성).
* `comment: string` - 데이터베이스의 컬럼 주석입니다. 모든 데이터베이스 타입에서 지원되지는 않습니다.
* `precision: number` - 값에 대해 저장되는 최대 자릿수인 10진수(정확히 숫자) 컬럼의 정밀도(10진수 컬럼에만 적용됨). 일부 컬럼 타입에서 사용됩니다.
* `scale: number` - 소수점 오른쪽의 자릿수를 나타내며 정밀도보다 크지 않아야하는 10진수(정확히 숫자) 컬럼(10진수 컬럼에만 적용됨)의 스케일입니다. 일부 컬럼 타입에서 사용됩니다.
* `zerofill: boolean` - 숫자 컬럼에 `ZEROFILL` 속성을 추가합니다. MySQL에서만 사용됩니다. true 인 경우 MySQL은 자동으로 이 컬럼에 `UNSIGNED` 속성을 추가합니다.
* `unsigned: boolean` - 숫자 열에 `UNSIGNED` 속성을 추가합니다. MySQL에서만 사용됩니다.
* `charset: string` - 컬럼 문자 집합을 정의합니다. 모든 데이터베이스 타입에서 지원되지는 않습니다.
* `collation: string` - 컬럼 데이터 정렬을 정의합니다.
* `enum: string[]|AnyEnum` - 허용되는 열거형 값 목록을 지정하기 위해 `enum` 컬럼 타입에 사용됩니다. 값 배열을 지정하거나 열거형 클래스를 지정할 수 있습니다.
* `enumName: string` - 사용 된 열거형의 이름을 정의합니다.
* `asExpression: string` - 생성된 컬럼 표현식. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)에서만 사용됩니다.
* `generatedType: "VIRTUAL"|"STORED"` - 생성된 컬럼 타입. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)에서만 사용됩니다.
* `hstoreType: "object"|"string"` - `HSTORE` 컬럼의 반환 타입입니다. 값을 문자열 또는 객체로 반환합니다. [Postgres](https://www.postgresql.org/docs/9.6/static/hstore.html)에서만 사용됩니다.
* `array: boolean` - 배열이 될 수 있는 postgres 및 cockroachdb 컬럼 타입에 사용됩니다 (예: int[]).
* `transformer: { from(value: DatabaseType): EntityType, to(value: EntityType): DatabaseType }` - 임의 유형 `EntityType`의 속성을 데이터베이스에서 지원하는 `DatabaseType` 타입으로 마샬링하는데 사용됩니다. 트랜스포머 배열도 지원되며 쓰기시에는 자연스러운 순서로, 읽을 때는 역순으로 적용됩니다. 예: `[lowercase, encrypt]`는 먼저 문자열을 소문자로 쓴 다음 쓸 때 암호화하고 해독한 다음 읽을 때 아무것도하지 않습니다.

참고: 대부분의 컬럼 옵션은 RDBMS 전용이며 `MongoDB`에서는 사용할 수 없습니다.

## 엔티티 상속

You can reduce duplication in your code by using entity inheritance.

For example, you have `Photo`, `Question`, `Post` entities:

```typescript
@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

    @Column()
    size: string;

}

@Entity()
export class Question {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

    @Column()
    answersCount: number;

}

@Entity()
export class Post {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

    @Column()
    viewCount: number;

}
```

As you can see all those entities have common columns: `id`, `title`, `description`. To reduce duplication and produce a better abstraction we can create a base class called `Content` for them:

```typescript
export abstract class Content {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

}
@Entity()
export class Photo extends Content {

    @Column()
    size: string;

}

@Entity()
export class Question extends Content {

    @Column()
    answersCount: number;

}

@Entity()
export class Post extends Content {

    @Column()
    viewCount: number;

}
```

상위 항목(상위 항목도 다른 항목을 확장할 수 있음)의 모든 컬럼(관계, 삽입 등)은 최종 항목에서 상속되고 생성됩니다.

## 트리 엔티티

TypeORM supports the Adjacency list and Closure table patterns of storing tree structures.

### 인접 목록(Adjacency list)

인접 목록은 자체 참조가 있는 간단한 모델입니다. 이 접근 방식의 장점은 단순성이며, 단점은 조인 제한으로 인해 한 번에 큰 트리를 로드할 수 없다는 것입니다.

예:

```typescript
import {Entity, Column, PrimaryGeneratedColumn, ManyToOne, OneToMany} from "typeorm";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @ManyToOne(type => Category, category => category.children)
    parent: Category;

    @OneToMany(type => Category, category => category.parent)
    children: Category[];
}

```

### 클로저 테이블

클로저 테이블은 부모와 자식 간의 관계를 특별한 방식으로 별도의 테이블에 저장합니다. 읽기와 쓰기 모두에서 효율적입니다.
클로저 테이블에 대해 자세히 알아 보려면 [Bill Karwin의 멋진 프레젠테이션](https://www.slideshare.net/billkarwin/models-for-hierarchical-data)을 참조하세요.

예:

```typescript
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("closure-table")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;

    @TreeLevelColumn()
    level: number;
}
```
