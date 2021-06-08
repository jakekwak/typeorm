# 뷰 엔티티

- [뷰 엔티티란 무엇입니까?](#뷰-엔티티란-무엇입니까)
- [뷰 엔티티 컬럼](#뷰-엔티티-컬럼)
- [완전한 예](#완전한-예)

## 뷰 엔티티란 무엇입니까?

뷰 엔터티는 데이터베이스 뷰에 매핑되는 클래스입니다. 새 클래스를 정의하고 `@ViewEntity()`로 표시하여 뷰 엔터티를 만들 수 있습니다.

`@ViewEntity()` 다음 옵션을 허용합니다.

* `name` - 뷰 이름. 지정하지 않으면 엔티티 클래스 이름에서 뷰 이름이 생성됩니다.
* `database` - 선택한 DB 서버의 데이터베이스 이름.
* `schema` - 스키마 이름.
* `expression` - 뷰 정의. **필수 매개 변수**.

`expression` 적절하게 이스케이프된 컬럼과 테이블이 있는 문자열일 수 있으며 사용된 데이터베이스에 따라 다름(예: postgres):

```typescript
@ViewEntity({
    expression: `
        SELECT "post"."id" AS "id", "post"."name" AS "name", "category"."name" AS "categoryName"
        FROM "post" "post"
        LEFT JOIN "category" "category" ON "post"."categoryId" = "category"."id"
    `
})
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
```

**참고:** 매개 변수 바인딩은 드라이버 제한으로 인해 지원되지 않습니다. 대신 리터럴 매개 변수를 사용하십시오.

```typescript
@ViewEntity({
    expression: (connection: Connection) => connection.createQueryBuilder()
        .select("post.id", "id")
        .addSelect("post.name", "name")
        .addSelect("category.name", "categoryName")
        .from(Post, "post")
        .leftJoin(Category, "category", "category.id = post.categoryId")
        .where("category.name = :name", { name: "Cars" })  // <-- 이것은 틀렸습니다.
        .where("category.name = 'Cars'")                   // <-- 그리고 이것은 옳습니다.
})
```

각보기 엔티티는 연결 옵션에 등록되어야 합니다.

```typescript
import {createConnection, Connection} from "typeorm";
import {UserView} from "./entity/UserView";

const connection: Connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: [UserView]
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

## 뷰 엔티티 컬럼

뷰의 데이터를 올바른 엔티티 컬럼으로 매핑하려면 `@ViewColumn()` 데코레이터로 엔티티 컬럼을 표시하고 이러한 컬럼을 select 문 별칭으로 지정해야합니다.

문자열 표현식 정의가 있는 예 :

```typescript
import {ViewEntity, ViewColumn} from "typeorm";

@ViewEntity({
    expression: `
        SELECT "post"."id" AS "id", "post"."name" AS "name", "category"."name" AS "categoryName"
        FROM "post" "post"
        LEFT JOIN "category" "category" ON "post"."categoryId" = "category"."id"
    `
})
export class PostCategory {

    @ViewColumn()
    id: number;

    @ViewColumn()
    name: string;

    @ViewColumn()
    categoryName: string;

}
```

QueryBuilder를 사용한 예:

```typescript
import {ViewEntity, ViewColumn} from "typeorm";

@ViewEntity({
    expression: (connection: Connection) => connection.createQueryBuilder()
        .select("post.id", "id")
        .addSelect("post.name", "name")
        .addSelect("category.name", "categoryName")
        .from(Post, "post")
        .leftJoin(Category, "category", "category.id = post.categoryId")
})
export class PostCategory {

    @ViewColumn()
    id: number;

    @ViewColumn()
    name: string;

    @ViewColumn()
    categoryName: string;

}
```

## 완전한 예

두 개의 항목과 이러한 항목의 집계 데이터를 포함하는 뷰를 만들어 보겠습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToOne, JoinColumn} from "typeorm";
import {Category} from "./Category";

@Entity()
export class Post {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    categoryId: number;

    @ManyToOne(() => Category)
    @JoinColumn({ name: "categoryId" })
    category: Category;

}
```

```typescript
import {ViewEntity, ViewColumn, Connection} from "typeorm";

@ViewEntity({
    expression: (connection: Connection) => connection.createQueryBuilder()
        .select("post.id", "id")
        .addSelect("post.name", "name")
        .addSelect("category.name", "categoryName")
        .from(Post, "post")
        .leftJoin(Category, "category", "category.id = post.categoryId")
})
export class PostCategory {

    @ViewColumn()
    id: number;

    @ViewColumn()
    name: string;

    @ViewColumn()
    categoryName: string;

}
```

그런 다음 이러한 테이블을 데이터로 채우고 PostCategory 뷰에서 모든 데이터를 요청합니다.

```typescript
import {getManager} from "typeorm";
import {Category} from "./entity/Category";
import {Post} from "./entity/Post";
import {PostCategory} from "./entity/PostCategory";

const entityManager = getManager();

const category1 = new Category();
category1.name = "Cars";
await entityManager.save(category1);

const category2 = new Category();
category2.name = "Airplanes";
await entityManager.save(category2);

const post1 = new Post();
post1.name = "About BMW";
post1.categoryId = category1.id;
await entityManager.save(post1);

const post2 = new Post();
post2.name = "About Boeing";
post2.categoryId = category2.id;
await entityManager.save(post2);

const postCategories = await entityManager.find(PostCategory);
const postCategory = await entityManager.findOne(PostCategory, { id: 1 });
```

`postCategories`의 결과는 다음과 같습니다.

```
[ PostCategory { id: 1, name: 'About BMW', categoryName: 'Cars' },
  PostCategory { id: 2, name: 'About Boeing', categoryName: 'Airplanes' } ]
```

그리고 `postCategory`에서 :

```
PostCategory { id: 1, name: 'About BMW', categoryName: 'Cars' }
```
