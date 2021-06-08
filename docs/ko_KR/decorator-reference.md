# Decorators reference

- [엔티티 데코레이터](#엔티티-데코레이터)
    - [`@Entity`](#entity)
    - [`@ViewEntity`](#viewentity)
- [컬럼 데코레이터](#컬럼-데코레이터)
    - [`@Column`](#column)
    - [`@PrimaryColumn`](#primarycolumn)
    - [`@PrimaryGeneratedColumn`](#primarygeneratedcolumn)
    - [`@ObjectIdColumn`](#objectidcolumn)
    - [`@CreateDateColumn`](#createdatecolumn)
    - [`@UpdateDateColumn`](#updatedatecolumn)
    - [`@DeleteDateColumn`](#deletedatecolumn)
    - [`@VersionColumn`](#versioncolumn)
    - [`@Generated`](#generated)
- [관계 데코레이터](#관계-데코레이터)
    - [`@OneToOne`](#onetoone)
    - [`@ManyToOne`](#manytoone)
    - [`@OneToMany`](#onetomany)
    - [`@ManyToMany`](#manytomany)
    - [`@JoinColumn`](#joincolumn)
    - [`@JoinTable`](#jointable)
    - [`@RelationId`](#relationid)
- [구독자 및 리스너 데코레이터](#구독자-및-리스너-데코레이터)
    - [`@AfterLoad`](#afterload)
    - [`@BeforeInsert`](#beforeinsert)
    - [`@AfterInsert`](#afterinsert)
    - [`@BeforeUpdate`](#beforeupdate)
    - [`@AfterUpdate`](#afterupdate)
    - [`@BeforeRemove`](#beforeremove)
    - [`@AfterRemove`](#afterremove)
    - [`@EventSubscriber`](#eventsubscriber)
- [기타 데코레이터](#기타-데코레이터)
    - [`@Index`](#index)
    - [`@Unique`](#unique)
    - [`@Check`](#check)
    - [`@Exclusion`](#exclusion)
    - [`@Transaction`, `@TransactionManager` 및 `@TransactionRepository`](#transaction-transactionmanager-및-transactionrepository)
    - [`@EntityRepository`](#entityrepository)

## 엔티티 데코레이터

#### `@Entity`

모델을 엔티티로 표시합니다. 엔티티는 데이터베이스 테이블로 변환되는 클래스입니다. 엔터티에 테이블 이름을 지정할 수 있습니다.

```typescript
@Entity("users")
export class User {
```

이 코드는 "users"라는 데이터베이스 테이블을 생성합니다.

몇가지 추가 엔티티 옵션을 지정할 수도 있습니다.

* `name` - 테이블 이름. 지정하지 않으면 엔티티 클래스 이름에서 테이블 이름이 생성됩니다.
* `database` - 선택한 DB 서버의 데이터베이스 이름.
* `schema` - 스키마 이름.
* `engine` - 테이블 생성 중 설정할 데이터베이스 엔진 (일부 데이터베이스에서만 작동)
* `synchronize` - `false`로 표시된 항목은 스키마 업데이트에서 건너뜁니다.
* `orderBy` - `find` 작업 및 `QueryBuilder`를 사용할 때 항목의 기본 순서를 지정합니다.

예:

```typescript
@Entity({
    name: "users",
    engine: "MyISAM",
    database: 'example_dev',
    schema: 'schema_with_best_tables',
    synchronize: false,
    orderBy: {
        name: "ASC",
        id: "DESC"
    }
})
export class User {
```

[엔티티](entities.md)에 대해 자세히 알아보세요.

#### `@ViewEntity`

뷰 엔터티는 데이터베이스 뷰에 매핑되는 클래스입니다.

`@ViewEntity ()`는 다음 옵션을 허용합니다.

* `name` - 뷰 이름. 지정하지 않으면 엔티티 클래스 이름에서 뷰 이름이 생성됩니다.
* `database` - 선택한 DB 서버의 데이터베이스 이름.
* `schema` - 스키마 이름.
* `expression` - 뷰 정의. **필수 매개 변수**.

`expression`은 사용된 데이터베이스 (예: postgres)에 따라 적절하게 이스케이프된 컬럼과 테이블이 있는 문자열일 수 있습니다.

```typescript
@ViewEntity({
    expression: `
        SELECT "post"."id" "id", "post"."name" AS "name", "category"."name" AS "categoryName"
        FROM "post" "post"
        LEFT JOIN "category" "category" ON "post"."categoryId" = "category"."id"
    `
})
export class PostCategory {
```

또는 QueryBuilder의 인스턴스

```typescript
@ViewEntity({
    expression: (connection: Connection) => connection.createQueryBuilder()
        .select("post.id", "id")
        .addSelect("post.name", "name")
        .addSelect("category.name", "categoryName")
        .from(Post, "post")
        .leftJoin(Category, "category", "category.id = post.categoryId")
})
export class PostCategory {
```

**참고:** 매개변수 바인딩은 드라이버 제한으로 인해 지원되지 않습니다. 대신 리터럴 매개변수를 사용하십시오.

```typescript
@ViewEntity({
    expression: (connection: Connection) => connection.createQueryBuilder()
        .select("post.id", "id")
        .addSelect("post.name", "name")
        .addSelect("category.name", "categoryName")
        .from(Post, "post")
        .leftJoin(Category, "category", "category.id = post.categoryId")
        .where("category.name = :name", { name: "Cars" })  // <-- this is wrong
        .where("category.name = 'Cars'")                   // <-- and this is right
})
export class PostCategory {
```

[뷰 엔티티](view-entities.md)에 대해 자세히 알아보세요.

## 컬럼 데코레이터

#### `@Column`

엔터티의 속성을 테이블 컬럼으로 표시합니다.

예:

```typescript
@Entity("users")
export class User {

    @Column({ primary: true })
    id: number;

    @Column({ type: "varchar", length: 200, unique: true })
    firstName: string;

    @Column({ nullable: true })
    lastName: string;

    @Column({ default: false })
    isActive: boolean;
}
```

`@ Column`은 사용할 수 있는 몇가지 옵션을 허용합니다.

* `type: ColumnType` - 컬럼 타입. [지원되는 컬럼 타입](entities.md#column-types) 중 하나입니다.
* `name: string` - 데이터베이스 테이블의 컬럼 이름입니다. 기본적으로 컬럼 이름은 속성 이름에서 생성됩니다. 자신의 이름을 지정하여 변경할 수 있습니다.
* `length: string|number` - 컬럼 타입의 길이. 예를 들어 `varchar(150)` 타입을 생성하려면 컬럼 타입과 길이 옵션을 지정합니다.
* `width: number` - 컬럼 타입의 표시 너비. [MySQL 정수 유형](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)에만 사용됩니다.
* `onUpdate: string` - `ON UPDATE` 트리거. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html)에서만 사용됩니다.
* `nullable: boolean` - 데이터베이스에서 `NULL`또는 `NOT NULL` 컬럼을 만듭니다. 기본적으로 컬럼은 `nullable: false` 입니다.
* `update: boolean` - 컬럼 값이 "save" 작업으로 업데이트되었는지 여부를 나타냅니다. false인 경우 객체를 처음 삽입할 때만 이 값을 쓸 수 있습니다. 기본값은 `true`입니다.
* `insert: boolean` - 객체를 처음 삽입할 때 컬럼 값이 설정되었는지 여부를 나타냅니다. 기본값은 `true`입니다.
* `select: boolean` - 쿼리를 만들 때 기본적으로 이 컬럼을 숨길 지 여부를 정의합니다. `false`로 설정하면 컬럼 데이터가 표준 쿼리에 표시되지 않습니다. 기본 열은 `select: true`입니다.
* `default: string` - 데이터베이스 수준 컬럼의 `DEFAULT`값을 추가합니다.
* `primary: boolean` - 컬럼을 기본으로 표시합니다. `@PrimaryColumn`을 사용하는 것과 동일합니다.
* `unique: boolean` - 컬럼을 고유한 컬럼으로 표시합니다 (고유 제약조건 생성). 기본값은 `false`입니다.
* `comment: string` - 데이터베이스의 컬럼의 주석입니다. 모든 데이터베이스 타입에서 지원되지는 않습니다.
* `precision: number` - 값에 대해 저장되는 최대 자릿수인 10진수(정확히 숫자) 컬럼의 정밀도(10진수 컬럼에만 적용됨). 일부 컬럼 타입에서 사용됩니다.
* `scale: number` - 소수점 오른쪽의 자릿수를 나타내며 정밀도보다 크지 않아야 하는 10진수(정확히 숫자) 컬럼 (10진수 컬럼에만 적용됨)의 스케일입니다. 일부 컬럼 타입에서 사용됩니다.
* `zerofill: boolean` - 숫자 컬럼에 `ZEROFILL` 속성을 추가합니다. MySQL에서만 사용됩니다. `true`인 경우 MySQL은 자동으로 이 컬럼에 `UNSIGNED` 속성을 추가합니다.
* `unsigned: boolean` - 숫자 컬럼에 `UNSIGNED` 속성을 추가합니다. MySQL에서만 사용됩니다.
* `charset: string` - 컬럼 문자집합을 정의합니다. 모든 데이터베이스 타입에서 지원되지는 않습니다.
* `collation: string` - 컬럼 데이터 정렬.
* `enum: string[]|AnyEnum` - 허용되는 열거형 값 목록을 지정하기 위해 `enum` 컬럼 타입에 사용됩니다. 값 배열을 지정하거나 열거형 클래스를 지정할 수 있습니다.
* `enumName: string` - 생성된 열거형 타입의 이름입니다. 지정하지 않으면 TypeORM은 엔티티 및 컬럼 이름에서 열거형 유형을 생성하므로 다른 테이블에서 동일한 열거형 타입을 사용하려는 경우 필요합니다.
* `asExpression: string` - 생성된 컬럼 표현식. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)에서만 사용됩니다.
* `generatedType: "VIRTUAL"|"STORED"` - 생성된 컬럼 타입. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)에서만 사용됩니다.
* `hstoreType: "object"|"string"` - `HSTORE` 컬럼의 반환타입입니다. 값을 문자열 또는 객체로 반환합니다. [Postgres](https://www.postgresql.org/docs/9.6/static/hstore.html)에서만 사용됩니다.
* `array: boolean` - 배열이 될 수 있는 postgres 및 cockroachdb 컬럼 타입에 사용됩니다(예: int[]).
* `transformer: ValueTransformer|ValueTransformer[]` - 데이터베이스를 읽거나 쓸 때 이 컬럼을 마샬링하는 데 사용할 값 변환기(또는 값 변환기 배열)를 지정합니다. 배열의 경우 값 변환기는 entityValue에서 databaseValue로 자연 순서로 적용되고 databaseValue에서 entityValue로 역순으로 적용됩니다.
* `spatialFeatureType: string` - 공간 컬럼(spatial column)에 대한 제약으로 사용되는 선택적 기능 타입(`Point`, `Polygon`, `LineString`, `Geometry`)입니다. 지정하지 않으면 `Geometry`가 제공된 것처럼 작동합니다. PostgreSQL에서만 사용됩니다.
* `srid: number` - 공간 컬럼에 대한 제약조건으로 사용되는 선택적 [공간 참조 ID](https://postgis.net/docs/using_postgis_dbmanagement.html#spatial_ref_sys) 지정하지 않으면 기본값은 `0`입니다. 표준 지리 좌표 (WGS84 데이텀의 위도/경도)는 [EPSG 4326](http://spatialreference.org/ref/epsg/wgs-84/)에 해당합니다. PostgreSQL에서만 사용됩니다.

[엔티티 컬럼](entities.md#entity-columns)에 대해 자세히 알아보세요.

#### `@PrimaryColumn`

엔터티의 속성을 테이블 기본 열로 표시합니다. `@Column` 데코레이터와 동일하지만 `primary` 옵션을 true로 설정합니다.

예:

```typescript
@Entity()
export class User {

    @PrimaryColumn()
    id: number;

}
```

[엔티티 컬럼](entities.md#entity-columns)에 대해 자세히 알아보세요.

#### `@PrimaryGeneratedColumn`

엔터티의 속성을 테이블 생성 기본 컬럼으로 표시합니다. 생성된 컬럼은 기본이며 해당값은 자동생성됩니다.

예:

```typescript
@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

}
```

두가지 생성 전략이 있습니다.

* `increment` - AUTO_INCREMENT / SERIAL / SEQUENCE (데이터베이스 유형에 따라 다름)를 사용하여 증분 번호를 생성합니다.
* `uuid` - 고유한 `uuid` 문자열을 생성합니다.
* `rowid` - [CockroachDB](https://www.cockroachlabs.com/docs/stable/serial.html)에만 해당됩니다. 값은 `unique_rowid()` 함수를 사용하여 자동으로 생성됩니다. 이렇게하면 `INSERT` 또는 `UPSERT` 작업을 실행하는 노드의 현재 타임 스탬프 및 ID에서 64 비트 정수가 생성됩니다.
> 참고: `rowid` 생성 전략이 있는 속성은 `string` 데이터 타입이어야 합니다.

기본 생성 전략은 `increment`입니다. 다른 전략으로 변경하려면 데코레이터에 첫 번째 인수로 전달하면됩니다.

```typescript
@Entity()
export class User {

    @PrimaryGeneratedColumn("uuid")
    id: string;

}
```

[엔티티 컬럼](entities.md#entity-columns)에 대해 자세히 알아보세요.

#### `@ObjectIdColumn`

엔터티의 속성을 ObjectID로 표시합니다. 이 데코레이터는 MongoDB에서만 사용됩니다. MongoDB의 모든 항목에는 ObjectID 컬럼이 있어야 합니다.

예:

```typescript
@Entity()
export class User {

    @ObjectIdColumn()
    id: ObjectID;

}
```

[MongoDB](mongodb.md)에 대해 자세히 알아보세요.

#### `@CreateDateColumn`

엔티티의 생성시간으로 자동 설정되는 특수 컬럼입니다. 이 컬럼에 값을 쓸 필요가 없습니다. 자동으로 설정됩니다.

예:

```typescript
@Entity()
export class User {

    @CreateDateColumn()
    createdDate: Date;

}
```

#### `@UpdateDateColumn`

엔티티 관리자 또는 저장소에서 `save`를 호출할 때마다 엔티티 업데이트 시간으로 자동 설정되는 특수 컬럼입니다. 이 컬럼에 값을 쓸 필요가 없습니다. 자동으로 설정됩니다.

```typescript
@Entity()
export class User {

    @UpdateDateColumn()
    updatedDate: Date;

}
```

#### `@DeleteDateColumn`

엔티티 관리자 또는 저장소의 일시 삭제를 호출할 때마다 엔티티 삭제시간에 자동으로 설정되는 특수 컬럼입니다. 이 컬럼은 설정할 필요가 없습니다. 자동으로 설정됩니다.

TypeORM의 자체 소프트 삭제 기능은 전역 범위를 활용하여 데이터베이스에서 "삭제되지 않은" 항목만 가져옵니다.

`@DeleteDateColumn`이 설정된 경우 기본 범위는 "삭제되지 않음"입니다.

```typescript
@Entity()
export class User {

    @DeleteDateColumn()
    deletedDate: Date;

}
```

#### `@VersionColumn`

엔티티 관리자 또는 저장소에서 `save`를 호출할 때마다 엔티티의 버전(증분 번호)으로 자동 설정되는 특수 컬럼입니다. 이 컬럼에 값을 쓸 필요가 없습니다. 자동으로 설정됩니다.

```typescript
@Entity()
export class User {

    @VersionColumn()
    version: number;

}
```

#### `@Generated`

컬럼을 생성된 값으로 표시합니다. 예를 들면:

```typescript
@Entity()
export class User {

    @Column()
    @Generated("uuid")
    uuid: string;

}
```

값은 데이터베이스에 엔티티를 삽입하기 전에 한번만 생성됩니다.

## 관계 데코레이터

#### `@OneToOne`

일대일은 A가 B의 인스턴스를 한번만 포함하고 B는 A의 인스턴스를 하나만 포함하는 관계입니다. 예를 들어 `User` 및 `Profile` 엔티티를 보겠습니다. 사용자는 단일 프로필만 가질 수 있으며 단일 프로필은 단일 사용자만 소유합니다.

예:

```typescript
import {Entity, OneToOne, JoinColumn} from "typeorm";
import {Profile} from "./Profile";

@Entity()
export class User {

    @OneToOne(type => Profile, profile => profile.user)
    @JoinColumn()
    profile: Profile;

}
```

[일대일 관계](one-to-one-relations.md)에 대해 자세히 알아보세요.

#### `@ManyToOne`

다대일 / 일대다는 A에 B의 여러 인스턴스가 포함되어 있지만 B에는 A의 인스턴스가 하나만 포함된 관계입니다. 예를 들어 `User` 및 `Photo` 엔티티를 살펴 보겠습니다. 사용자는 여러장의 사진을 가질 수 있지만 각 사진은 한명의 사용자만 소유합니다.

예:

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

[다대일 / 일대다 관계](many-to-one-one-to-many-relations.md)에 대해 자세히 알아보십시오.

#### `@OneToMany`

다대일 / 일대다는 A에 B의 여러 인스턴스가 포함되어 있지만 B에는 A의 인스턴스가 하나만 포함된 관계입니다. 예를 들어 `User` 및 `Photo` 엔티티를 살펴 보겠습니다. 사용자는 여러장의 사진을 가질 수 있지만 각 사진은 한명의 사용자만 소유합니다.

예:

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

[다대일 / 일대다 관계](many-to-one-one-to-many-relations.md)에 대해 자세히 알아보십시오.

#### `@ManyToMany`

다대다는 A에 B의 여러 인스턴스가 포함되고 B에 A의 여러 인스턴스가 포함되는 관계입니다. 예를 들어 `Question` 및 `Category` 항목을 살펴 보겠습니다. 질문에는 여러 카테고리가 있을 수 있으며 각 카테고리에는 여러 질문이 있을 수 있습니다.

예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable} from "typeorm";
import {Category} from "./Category";

@Entity()
export class Question {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

    @ManyToMany(type => Category)
    @JoinTable()
    categories: Category[];

}
```

[다대다 관계](many-to-many-relations.md)에 대해 자세히 알아보십시오.

#### `@JoinColumn`

외래 키가 있는 조인 컬럼을 포함하는 관계의 측면을 정의하고 조인 컬럼 이름과 참조된 컬럼 이름을 사용자 지정할 수 있습니다.

예:

```typescript
@Entity()
export class Post {

    @ManyToOne(type => Category)
    @JoinColumn({
        name: "cat_id",
        referencedColumnName: "name"
    })
    category: Category;

}
```

#### `@JoinTable`

`many-to-many` 관계에 사용되며 "정션" 테이블의 조인 컬럼을 설명합니다. 정션 테이블은 관련 엔터티를 참조하는 컬럼과 함께 TypeORM에 의해 자동으로 생성되는 특수한 별도의 테이블입니다. `@JoinColumn` 데코레이터를 사용하여 접합 테이블 내부의 컬럼 이름과 참조된 컬럼을 변경할 수 있습니다. 생성된 "정션" 테이블의 이름을 변경할 수도 있습니다.

예:

```typescript
@Entity()
export class Post {

    @ManyToMany(type => Category)
    @JoinTable({
        name: "question_categories",
        joinColumn: {
            name: "question",
            referencedColumnName: "id"
        },
        inverseJoinColumn: {
            name: "category",
            referencedColumnName: "id"
        }
    })
    categories: Category[];

}
```

대상 테이블에 복합 기본키가 있는 경우 속성 배열을 `@JoinTable` 데코레이터로 보내야합니다.

#### `@RelationId`

특정 관계의 ID(또는 ID)를 속성에 로드합니다. 예를 들어 `Post`항목에 다대일 `category`가 있는 경우 `@RelationId`로 새 속성을 표시하여 새 카테고리 ID를 가질 수 있습니다.

예:

```typescript
@Entity()
export class Post {

    @ManyToOne(type => Category)
    category: Category;

    @RelationId((post: Post) => post.category) // 대상 관계를 지정해야합니다.
    categoryId: number;

}
```

이 기능은 `다대다`를 포함한 모든 종류의 관계에서 작동합니다.

```typescript
@Entity()
export class Post {

    @ManyToMany(type => Category)
    categories: Category[];

    @RelationId((post: Post) => post.categories)
    categoryIds: number[];

}
```

관계 ID는 표현에만 사용됩니다. 기본 관계는 값을 연결할 때 추가/제거/변경되지 않습니다.

## 구독자 및 리스너 데코레이터

#### `@AfterLoad`

엔터티에 임의의 이름으로 메서드를 정의하고 `@AfterLoad`로 표시하면 TypeORM이 `QueryBuilder` 또는 저장소/관리자 찾기 메서드를 사용하여 엔터티가 로드될 때마다 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterLoad()
    updateCounters() {
        if (this.likesCount === undefined)
            this.likesCount = 0;
    }
}
```

[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@BeforeInsert`

엔티티에 임의의 이름으로 메소드를 정의하고 `@BeforeInsert`로 표시하면 TypeORM이 저장소/관리자 `save`를 사용하여 엔티티가 삽입되기 전에 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @BeforeInsert()
    updateDates() {
        this.createdDate = new Date();
    }
}
```
[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@AfterInsert`

엔티티에 임의의 이름으로 메소드를 정의하고 `@AfterInsert`로 표시하면 TypeORM이 저장소/관리자 `save`를 사용하여 엔티티가 삽입된 후 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterInsert()
    resetCounters() {
        this.counters = 0;
    }
}
```

[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@BeforeUpdate`

엔티티에 임의의 이름으로 메소드를 정의하고 `@BeforeUpdate`로 표시하면 TypeORM이 저장소/관리자 `save`를 사용하여 기존 엔티티가 업데이트되기 전에 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @BeforeUpdate()
    updateDates() {
        this.updatedDate = new Date();
    }
}
```

[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@AfterUpdate`

엔티티에 임의의 이름으로 메소드를 정의하고 `@AfterUpdate`로 표시할 수 있으며 TypeORM은 저장소/관리자 `save`를 사용하여 기존 엔티티가 업데이트된 후이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterUpdate()
    updateCounters() {
        this.counter = 0;
    }
}
```

[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@BeforeRemove`

엔티티에 임의의 이름으로 메소드를 정의하고 `@BeforeRemove`로 표시하면 TypeORM이 저장소/관리자 `remove`를 사용하여 엔티티를 제거하기 전에 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @BeforeRemove()
    updateStatus() {
        this.status = "removed";
    }
}
```

[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@AfterRemove`

엔티티에 임의의 이름으로 메소드를 정의하고 `@AfterRemove`로 표시하면 TypeORM이 저장소/관리자 `remove`를 사용하여 엔티티가 제거된 후 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterRemove()
    updateStatus() {
        this.status = "removed";
    }
}
```

[리스너](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

#### `@EventSubscriber`

클래스를 특정 엔터티 이벤트 또는 엔터티의 이벤트를 수신할 수 있는 이벤트 구독자로 표시합니다. 이벤트는 `QueryBuilder` 및 저장소/관리자 메소드를 사용하여 시작됩니다.

예:

```typescript
@EventSubscriber()
export class PostSubscriber implements EntitySubscriberInterface<Post> {


    /**
     * 이 구독자가 Post 이벤트만 수신함을 나타냅니다.
     */
    listenTo() {
        return Post;
    }

    /**
     * 포스트 삽입 전에 호출됩니다.
     */
    beforeInsert(event: InsertEvent<Post>) {
        console.log(`BEFORE POST INSERTED: `, event.entity);
    }

}
```

`EntitySubscriberInterface`의 모든 메소드를 구현할 수 있습니다. 엔티티를 수신하려면 `listenTo` 메소드를 생략하고 `any`를 사용하면 됩니다.

```typescript
@EventSubscriber()
export class PostSubscriber implements EntitySubscriberInterface {

    /**
     * 엔티티 삽입 전에 호출됩니다.
     */
    beforeInsert(event: InsertEvent<any>) {
        console.log(`BEFORE ENTITY INSERTED: `, event.entity);
    }

}
```

[구독자](listeners-and-subscribers.md)에 대해 자세히 알아보세요.

## 기타 데코레이터

#### `@Index`

이 데코레이터를 사용하면 특정 컬럼에 대한 데이터베이스 인덱스를 만들 수 있습니다. 또한 컬럼을 고유하게 표시할 수도 있습니다. 이 데코레이터는 컬럼 또는 엔티티 자체에 적용할 수 있습니다. 단일 컬럼에 대한 인덱스가 필요한 경우 컬럼에 사용하고 여러 열에 대한 단일 인덱스가 필요한 경우 엔터티에 사용합니다.

예:

```typescript
@Entity()
export class User {

    @Index()
    @Column()
    firstName: string;

    @Index({ unique: true })
    @Column()
    lastName: string;
}
```
```typescript
@Entity()
@Index(["firstName", "lastName"])
@Index(["lastName", "middleName"])
@Index(["firstName", "lastName", "middleName"], { unique: true })
export class User {

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    middleName: string;
}
```

[색인](indices.md)에 대해 자세히 알아보세요.

#### `@Unique`

이 데코레이터를 사용하면 특정 컬럼에 대한 데이터베이스 고유 제약조건을 만들 수 있습니다. 이 데코레이터는 엔티티 자체에만 적용할 수 있습니다. 엔터티 필드이름(데이터베이스 컬럼 이름 아님)을 인수로 지정해야합니다.

예:

```typescript
@Entity()
@Unique(["firstName"])
@Unique(["lastName", "middleName"])
@Unique("UQ_NAMES", ["firstName", "lastName", "middleName"])
export class User {

    @Column({ name: 'first_name' })
    firstName: string;

    @Column({ name: 'last_name' })
    lastName: string;

    @Column({ name: 'middle_name' })
    middleName: string;
}
```

> 참고: MySQL은 고유한 제약조건을 고유 인덱스로 저장합니다.

#### `@Check`

이 데코레이터를 사용하면 특정 컬럼에 대한 데이터베이스 검사 제약조건을 만들 수 있습니다. 이 데코레이터는 엔티티 자체에만 적용할 수 있습니다.

예:

```typescript
@Entity()
@Check(`"firstName" <> 'John' AND "lastName" <> 'Doe'`)
@Check(`"age" > 18`)
export class User {

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    age: number;
}
```

> 참고: MySQL은 검사 제약조건을 지원하지 않습니다.

#### `@Exclusion`

이 데코레이터를 사용하면 특정 컬럼에 대한 데이터베이스 제외 제약조건을 만들 수 있습니다. 이 데코레이터는 엔티티 자체에만 적용할 수 있습니다.

예:

```typescript
@Entity()
@Exclusion(`USING gist ("room" WITH =, tsrange("from", "to") WITH &&)`)
export class RoomBooking {

    @Column()
    room: string;

    @Column()
    from: Date;

    @Column()
    to: Date;
}
```

> 참고 : PostgreSQL만 제외 제약조건을 지원합니다.

#### `@Transaction`, `@TransactionManager` 및 `@TransactionRepository`

`@Transaction`은 메서드에서 사용되며 모든 실행을 단일 데이터베이스 트랜잭션으로 래핑합니다. 모든 데이터베이스 쿼리는 `@TransactionManager` 제공 관리자 또는 `@TransactionRepository`가 삽입된 트랜잭션 저장소를 사용하여 수행되어야 합니다.

예:

```typescript

@Transaction()
save(@TransactionManager() manager: EntityManager, user: User) {
    return manager.save(user);
}
```

```typescript
@Transaction()
save(user: User, @TransactionRepository(User) userRepository: Repository<User>) {
    return userRepository.save(user);
}
```

```typescript
@Transaction()
save(@QueryParam("name") name: string, @TransactionRepository() userRepository: UserRepository) {
    return userRepository.findByName(name);
}
```

> 참고: 트랜잭션 내부의 모든 작업은 제공된 `EntityManager` 인스턴스 또는 삽입된 저장소만 사용해야 합니다.

다른 쿼리 소스(글로벌 관리자, 글로벌 리포지토리 등)를 사용하면 버그와 오류가 발생합니다.

[트랜잭션](transactions.md)에 대해 자세히 알아보세요.

#### `@EntityRepository`

사용자 정의 클래스를 엔티티 저장소로 표시합니다.

예:

```typescript
@EntityRepository()
export class UserRepository {

    /// ... 사용자 정의 저장소 방법 ...

}
```

`connection.getCustomRepository` 또는 `entityManager.getCustomRepository` 메소드를 사용하여 커스텀으로 생성된 저장소를 얻을 수 있습니다.

[커스텀 엔티티 리포지토리](custom-repository.md)에 대해 자세히 알아보세요.

----

> 참고: 일부 데코레이터(`@Tree`, `@ChildEntity` 등)는 그렇지 않습니다.

현재 실험적인 것으로 취급되기 때문에 이 참조에 문서화되어 있습니다. 향후 문서를 볼 수 있습니다.
