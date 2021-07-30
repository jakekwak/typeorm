# 관계

- [관계는 무엇입니까?](#관계는-무엇입니까)
- [관계 옵션](#관계-옵션)
- [캐스케이드](#캐스케이드)
  - [캐스케이드 옵션](#캐스케이드-옵션)
- [`@JoinColumn` 옵션](#joincolumn-옵션)
- [`@JoinTable` 옵션](#jointable-옵션)

## 관계는 무엇입니까?

관계를 사용하면 관련 엔터티와 쉽게 작업할 수 있습니다. 여러 타입의 관계가 있습니다.

* [one-to-one](./one-to-one-relations.md) `@OneToOne` 사용
* [many-to-one](./many-to-one-one-to-many-relations.md) `@ManyToOne` 사용
* [one-to-many](./many-to-one-one-to-many-relations.md) `@OneToMany` 사용
* [many-to-many](./many-to-many-relations.md) `@ManyToMany` 사용

## 관계 옵션

관계에 대해 지정할 수 있는 몇가지 옵션이 있습니다.

* `eager: boolean` - true로 설정하면 이 항목에서 `find*` 메서드 또는 `QueryBuilder`를 사용할 때 항상 기본 항목과 함께 관계가 로드됩니다.
* `cascade: boolean | ("insert" | "update")[]` - true로 설정하면 관련 객체가 데이터베이스에 삽입되고 업데이트됩니다. [cascade options](#cascade-options) 배열을 지정할 수도 있습니다.
* `onDelete: "RESTRICT"|"CASCADE"|"SET NULL"` - 참조된 객체가 삭제될 때 외래 키의 작동 방식을 지정합니다.
* `primary: boolean` - 이 관계의 컬럼이 기본 컬럼인지 여부를 나타냅니다.
* `nullable: boolean` - 이 관계의 컬럼이 Null을 허용하는지 여부를 나타냅니다. 기본적으로 nullable입니다.
* `orphanedRowAction: "nullify" | "delete"` - 하위 행이 상위에서 제거될 때 하위 행이 고아(기본값) 또는 삭제되어야 하는지 여부를 결정합니다.

## 캐스케이드

캐스케이드 예 :

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany} from "typeorm";
import {Question} from "./Question";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(type => Question, question => question.categories)
    questions: Question[];

}
```

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

    @ManyToMany(type => Category, category => category.questions, {
        cascade: true
    })
    @JoinTable()
    categories: Category[];

}
```

```typescript
const category1 = new Category();
category1.name = "ORMs";

const category2 = new Category();
category2.name = "Programming";

const question = new Question();
question.title = "How to ask questions?";
question.text = "Where can I ask TypeORM-related questions?";
question.categories = [category1, category2];
await connection.manager.save(question);
```

이 예에서 볼 수 있듯이 `category1`과 `category2`에 대해 `save`를 호출하지 않았습니다. `cascade`를 true로 설정했기 때문에 자동으로 삽입됩니다.

명심하십시오 - 큰 힘에는 큰 책임이 따릅니다. 캐스케이드는 관계에 대해 작업하는 좋고 쉬운 방법처럼 보일 수 있지만 원하지 않는 객체가 데이터베이스에 저장될 때 버그 및 보안 문제를 가져올 수도 있습니다. 또한 새 객체를 데이터베이스에 저장하는 덜 명시적인 방법을 제공합니다.

### 캐스케이드 옵션

`cascade` 옵션은 `boolean` 또는 계단식 옵션 배열 `( "insert" | "update" | "remove" | "soft-remove" | "recover")[]`로 설정할 수 있습니다.

기본값은 `false`로, 캐스케이드가 없음을 의미합니다. `cascade: true`를 설정하면 전체 캐스케이드가 활성화됩니다. 배열을 제공하여 옵션을 지정할 수도 있습니다.

예를 들면 :

```typescript
@Entity(Post)
export class Post {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

    // 카테고리에 대한 전체 캐스케이드.
    @ManyToMany(type => PostCategory, {
        cascade: true
    })
    @JoinTable()
    categories: PostCategory[];

    // 여기서 캐스케이드 삽입은 이 관계에 설정된 새 PostDetails 인스턴스가 있는 경우
    // 이 Post 엔티티를 저장할 때 db에 자동으로 삽입됨을 의미합니다.
    @ManyToMany(type => PostDetails, details => details.posts, {
        cascade: ["insert"]
    })
    @JoinTable()
    details: PostDetails[];

    // 여기서 캐스케이드 업데이트는 기존 PostImage에 변경 사항이 있는 경우
    // 이 Post 엔티티를 저장할 때 db로 자동 업데이트됨을 의미합니다.
    @ManyToMany(type => PostImage, image => image.posts, {
        cascade: ["update"]
    })
    @JoinTable()
    images: PostImage[];

    // 여기서 캐스케이드 삽입 및 업데이트는 새 PostInformation 인스턴스 또는 기존 인스턴스에 대한
    // 업데이트가 있는 경우 이 Post 엔티티를 저장할 때 자동으로 삽입되거나 업데이트됨을 의미합니다.
    @ManyToMany(type => PostInformation, information => information.posts, {
        cascade: ["insert", "update"]
    })
    @JoinTable()
    informations: PostInformation[];
}
```

## `@JoinColumn` 옵션

`@JoinColumn`은 외래 키가 있는 조인 컬럼을 포함하는 관계의 측면을 정의할뿐만 아니라 조인 컬럼 이름과 참조된 컬럼 이름을 사용자 지정할 수도 있습니다.

`@ JoinColumn`을 설정하면 데이터베이스에 `propertyName+referencedColumnName` 이라는 컬럼이 자동으로 생성됩니다.

예를 들면:

```typescript
@ManyToOne(type => Category)
@JoinColumn() // 이 데코레이터는 @ManyToOne의 경우 선택 사항이지만 @OneToOne에는 필수입니다.
category: Category;
```

이 코드는 데이터베이스에 `categoryId` 컬럼을 생성합니다.
데이터베이스에서 이 이름을 변경하려면 사용자 지정 조인 컬럼 이름을 지정할 수 있습니다.

```typescript
@ManyToOne(type => Category)
@JoinColumn({ name: "cat_id" })
category: Category;
```

조인 컬럼은 항상 다른 컬럼에 대한 참조입니다(외래 키 사용). 기본적으로 관계는 항상 관련 엔터티의 기본 컬럼을 참조합니다. 관련 엔터티의 다른 컬럼과 관계를 만들려면 `@JoinColumn`에서도 지정할 수 있습니다.

```typescript
@ManyToOne(type => Category)
@JoinColumn({ referencedColumnName: "name" })
category: Category;
```

이제 관계가 `id`대신 `Category`항목의 `name`을 참조합니다. 해당 관계의 컬럼 이름은 `categoryName`이 됩니다.

여러 컬럼을 결합할 수도 있습니다. 기본적으로 관련 엔터티의 기본 컬럼을 참조하지 않습니다. 참조된 컬럼 이름을 제공해야 합니다.

```typescript
@ManyToOne(type => Category)
@JoinColumn([
    { name: "category_id", referencedColumnName: "id" },
    { name: "locale_id", referencedColumnName: "locale_id" }
])
category: Category;
```

## `@JoinTable` 옵션

`@JoinTable`은 `다대다` 관계에 사용되며 "정션" 테이블의 조인 컬럼을 설명합니다. 정션 테이블은 관련 엔터티를 참조하는 컬럼과 함께 TypeORM에 의해 자동으로 생성되는 특수한 별도 테이블입니다. `JoinColumn`과 `inverseJoinColumn`을 사용하여 정션 테이블 내부의 컬럼 이름과 참조된 컬럼을 변경할 수 있습니다. 생성된 정션 테이블의 이름을 변경할 수도 있습니다.

```typescript
@ManyToMany(type => Category)
@JoinTable({
    name: "question_categories", // 이 관계의 정션 테이블에 대한 테이블 이름
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
```

대상 테이블에 복합 기본 키가 있는 경우 속성 배열을 `@JoinTable`로 전송해야 합니다.
