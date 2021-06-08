# 다대다 관계

- [다대다 관계란?](#다대다-관계란)
- [다대다 관계 저장](#다대다-관계-저장)
- [다대다 관계 삭제](#다대다-관계-삭제)
- [캐스케이드가 있는 관계의 소프트 삭제](#캐스케이드가-있는-관계의-소프트-삭제)
- [다대다 관계 로드](#다대다-관계-로드)
- [양방향 관계](#양방향-관계)
- [사용자 정의 속성을 사용한 다대다 관계](#사용자-정의-속성을-사용한-다대다-관계)

## 다대다 관계란?

다대다는 A에 B의 여러 인스턴스가 포함되고 B에 A의 여러 인스턴스가 포함되는 관계입니다. 예를 들어 `Question` 및 `Category` 항목을 살펴 보겠습니다. 질문에는 여러 범주가 있을 수 있으며 각 범주에는 여러 질문이 있을 수 있습니다.

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

    @ManyToMany(() => Category)
    @JoinTable()
    categories: Category[];

}
```

`@ManyToMany` 관계에는 `@JoinTable()`이 필요합니다. 관계의 한쪽(소유)에 `@JoinTable`을 넣어야합니다.

이 예제는 다음 테이블을 생성합니다.

```shell
+-------------+--------------+----------------------------+
|                        category                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                        question                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| title       | varchar(255) |                            |
| text        | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|              question_categories_category               |
+-------------+--------------+----------------------------+
| questionId  | int(11)      | PRIMARY KEY FOREIGN KEY    |
| categoryId  | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

## 다대다 관계 저장

[캐스케이드](./relations.md#cascades)를 사용 설정하면 `save` 호출을 한번만 사용하여 이 관계를 저장할 수 있습니다.

```typescript
const category1 = new Category();
category1.name = "animals";
await connection.manager.save(category1);

const category2 = new Category();
category2.name = "zoo";
await connection.manager.save(category2);

const question = new Question();
question.title = "dogs";
question.text = "who let the dogs out?";
question.categories = [category1, category2];
await connection.manager.save(question);
```

## 다대다 관계 삭제

[캐스케이드](./relations.md#cascades)를 사용 설정하면 `save` 호출을 한번만 사용하여 이 관계를 삭제할 수 있습니다.

두 레코드간의 다대다 관계를 삭제하려면 해당 필드에서 제거하고 레코드를 저장하십시오.

```typescript
const question = getRepository(Question);
question.categories = question.categories.filter(category => {
    category.id !== categoryToRemove.id
})
await connection.manager.save(question)
```

조인 테이블의 레코드만 제거됩니다. `question` 및 `categoryToRemove` 레코드는 계속 존재합니다.

## 캐스케이드가 있는 관계의 소프트 삭제

이 예는 계단식 소프트 삭제가 어떻게 작동하는지 보여줍니다.

```typescript
const category1 = new Category();
category1.name = "animals";

const category2 = new Category();
category2.name = "zoo";

const question = new Question();
question.categories = [category1, category2];
const newQuestion =  await connection.manager.save(question);

await connection.manager.softRemove(newQuestion);
```

이 예에서는 `category1` 및 `category2`에 대해 `save` 또는 `softRemove`를 호출하지 않았지만 다음과 같이 `캐스케이드` 옵션이 `true`로 설정되면 자동으로 저장되고 일시 삭제됩니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable} from "typeorm";
import {Category} from "./Category";

@Entity()
export class Question {

    @PrimaryGeneratedColumn()
    id: number;

    @ManyToMany(() => Category, category => category.questions, {
        cascade: true
    })
    @JoinTable()
    categories: Category[];

}
```

## 다대다 관계 로드

카테고리가 있는 질문을 로드하려면 `FindOptions`에서 관계를 지정해야 합니다.

```typescript
const questionRepository = connection.getRepository(Question);
const questions = await questionRepository.find({ relations: ["categories"] });
```

또는 `QueryBuilder`를 사용하여 조인할 수 있습니다.

```typescript
const questions = await connection
    .getRepository(Question)
    .createQueryBuilder("question")
    .leftJoinAndSelect("question.categories", "category")
    .getMany();
```

`FindOptions`를 사용할 때 즉시로딩(eager) 관계를 지정할 필요가 없습니다. 항상 자동으로 로드됩니다.

## 양방향 관계

관계는 단방향 및 양방향일 수 있습니다. 단방향 관계는 한쪽에서만 관계 데코레이터와의 관계입니다. 양방향 관계는 관계의 양쪽에 있는 데코레이터와의 관계입니다.

우리는 단방향 관계를 생성했습니다. 양방향으로 만들어 보겠습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany} from "typeorm";
import {Question} from "./Question";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(() => Question, question => question.categories)
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

    @ManyToMany(() => Category, category => category.questions)
    @JoinTable()
    categories: Category[];

}
```

우리는 우리의 관계를 양방향으로 만들었습니다. 역 관계에는 `@JoinTable`이 없습니다. `@JoinTable`은 관계의 한쪽에만 있어야 합니다.

양방향 관계를 사용하면 `QueryBuilder`를 사용하여 양쪽에서 관계를 결합할 수 있습니다.

```typescript
const categoriesWithQuestions = await connection
    .getRepository(Category)
    .createQueryBuilder("category")
    .leftJoinAndSelect("category.questions", "question")
    .getMany();
```

## 사용자 정의 속성을 사용한 다대다 관계

In case you need to have additional properties in your many-to-many relationship, you have to create a new entity yourself.
For example, if you would like entities `Post` and `Category` to have a many-to-many relationship with an additional `order` column, then you need to create an entity `PostToCategory` with two `ManyToOne` relations pointing in both directions and with custom columns in it:
다대다 관계에 추가 속성이 필요한 경우 직접 새 엔터티를 만들어야합니다. 예를 들어 `Post` 및 `Category` 항목이 추가 `order` 컬럼과 다대다 관계를 갖도록 하려면 두개의 `ManyToOne` 관계가 가리키는 항목 `PostToCategory`를 생성해야합니다. 양방향 및 맞춤 컬럼 포함:

```typescript
import { Entity, Column, ManyToOne, PrimaryGeneratedColumn } from "typeorm";
import { Post } from "./post";
import { Category } from "./category";

@Entity()
export class PostToCategory {
    @PrimaryGeneratedColumn()
    public postToCategoryId!: number;

    @Column()
    public postId!: number;

    @Column()
    public categoryId!: number;

    @Column()
    public order!: number;

    @ManyToOne(() => Post, post => post.postToCategories)
    public post!: Post;

    @ManyToOne(() => Category, category => category.postToCategories)
    public category!: Category;
}
```

또한 `Post` 및 `Category`에 다음과 같은 관계를 추가해야합니다.

```typescript
// category.ts
...
@OneToMany(() => PostToCategory, postToCategory => postToCategory.category)
public postToCategories!: PostToCategory[];

// post.ts
...
@OneToMany(() => PostToCategory, postToCategory => postToCategory.post)
public postToCategories!: PostToCategory[];
```
