# 즉시로딩과 지연로딩 관계

- [즉시로딩 관계](#즉시로딩-관계)
- [지연로딩 관계](#지연로딩-관계)

## 즉시로딩 관계

즉시로딩 관계는 데이터베이스에서 엔티티를 로드할 때마다 자동으로 로드됩니다.

예를 들면 :

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
        eager: true
    })
    @JoinTable()
    categories: Category[];

}
```

이제 질문을 로드할 때 로드하려는 관계를 결합하거나 지정할 필요가 없습니다. 자동으로 로드됩니다.

```typescript
const questionRepository = connection.getRepository(Question);

// 질문은 카테고리로 로드됩니다.
const questions = await questionRepository.find();
```

즉시로딩 관계는 `find*` 메소드를 사용할 때만 작동합니다. `QueryBuilder`를 사용하는 경우 즉시로딩 관계가 비활성화되고 `leftJoinAndSelect`를 사용하여 관계를 로드해야 합니다. 즉시로딩 관계는 관계의 한쪽에서만 사용할 수 있으며 관계의 양쪽에 `eager: true`를 사용하는 것은 허용되지 않습니다.

## 지연로딩 관계

지연로딩 관계의 엔티티는 액세스하면 로드됩니다. 이러한 관계에는 유형으로 `Promise`가 있어야 합니다. Promise에 값을 저장하고 로드할 때 promise도 반환합니다. 예:

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
    questions: Promise<Question[]>;

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

    @ManyToMany(type => Category, category => category.questions)
    @JoinTable()
    categories: Promise<Category[]>;

}
```

`categories`는 Promise입니다. 지연로딩(lazy)이라는 의미이며, 값이 있는 promise만 저장할 수 있습니다.

이러한 관계를 저장하는 방법의 예:

```typescript
const category1 = new Category();
category1.name = "animals";
await connection.manager.save(category1);

const category2 = new Category();
category2.name = "zoo";
await connection.manager.save(category2);

const question = new Question();
question.categories = Promise.resolve([category1, category2]);
await connection.manager.save(question);
```

지연로딩 관계내에서 객체를 로드하는 방법의 예:

```typescript
const question = await connection.getRepository(Question).findOne(1);
const categories = await question.categories;
// 이제 "categories" 변수안에 모든 질문의 카테고리가 있습니다.
```

참고: 다른 언어(Java, PHP 등)에서 왔고 모든 곳에서 지연로딩 관계를 사용하는 데 사용되는 경우 주의하십시오. 이러한 언어는 비동기식이 아니며 지연로드는 다른 방식으로 이루어집니다. 그렇기 때문에 거기에서 promise를 사용하지 않는 것입니다. 자바스크립트와 Node.JS에서 지연로드된 관계를 원하면 promise를 사용해야합니다. 이것은 비표준 기술이며 TypeORM에서 실험적인 것으로 간주됩니다.