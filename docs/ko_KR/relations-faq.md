# 관계 FAQ

- [자기 참조 관계를 만드는 방법](#자기-참조-관계를-만드는-방법)
- [관계를 결합하지 않고 관계 ID를 사용하는 방법](#관계를-결합하지-않고-관계-id를-사용하는-방법)
- [엔티티에 관계를 로드하는 방법](#엔티티에-관계를-로드하는-방법)
- [관계 속성 초기화 방지](#관계-속성-초기화-방지)
- [외래키 제약조건 생성방지](#외래키-제약조건-생성방지)

## 자기 참조 관계를 만드는 방법

자기 참조 관계는 자신과 관련된 관계입니다. 이것은 트리와 같은 구조에 엔티티를 저장할 때 유용합니다. 또한 "인접 목록" 패턴은 자체 참조 관계를 사용하여 구현됩니다. 예를 들어, 애플리케이션에서 카테고리 트리를 작성하려고 합니다. 카테고리는 카테고리를 중첩할 수 있고 중첩된 카테고리는 다른 카테고리를 중첩할 수 있습니다. 여기에서는 자체 참조 관계가 편리합니다. 기본적으로 자체 참조 관계는 엔티티 자체를 대상으로하는 정규 관계입니다.

예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany} from "typeorm";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

    @ManyToOne(type => Category, category => category.childCategories)
    parentCategory: Category;

    @OneToMany(type => Category, category => category.parentCategory)
    childCategories: Category[];

}
```

## 관계를 결합하지 않고 관계 ID를 사용하는 방법

때로는 로드하지 않고 관련 객체의 객체 ID를 갖고 싶을 때가 있습니다.

예를 들면 :

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class Profile {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    gender: string;

    @Column()
    photo: string;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn} from "typeorm";
import {Profile} from "./Profile";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToOne(type => Profile)
    @JoinColumn()
    profile: Profile;

}
```

`profile`에 가입하지 않은 사용자를 로드하면 사용자 객체에 프로필에 대한 정보가 없습니다.

프로필 ID:

```javascript
User {
  id: 1,
  name: "Umed"
}
```

그러나 때때로 이 사용자의 전체 프로필을 로드하지 않고 이 사용자의 "프로필 ID"가 무엇인지 알고 싶을 때가 있습니다. 이렇게 하려면 관계에 의해 생성된 컬럼과 정확히 일치하는 이름이 `@Column`인 항목에 다른 속성을 추가하기만하면 됩니다.

예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn} from "typeorm";
import {Profile} from "./Profile";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column({ nullable: true })
    profileId: number;

    @OneToOne(type => Profile)
    @JoinColumn()
    profile: Profile;

}
```

그게 전부입니다. 다음에 사용자 개체를 로드하면 프로필 ID가 포함됩니다.

```javascript
User {
  id: 1,
  name: "Umed",
  profileId: 1
}
```

## 엔티티에 관계를 로드하는 방법

엔티티 관계를 로드하는 가장 쉬운 방법은 `FindOptions`에서 `relations` 옵션을 사용하는 것입니다.

```typescript
const users = await connection.getRepository(User).find({ relations: ["profile", "photos", "videos"] });
```

대안적이고 더 유연한 방법은 `QueryBuilder`를 사용하는 것입니다.

```typescript
const user = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.profile", "profile")
    .leftJoinAndSelect("user.photos", "photo")
    .leftJoinAndSelect("user.videos", "video")
    .getMany();
```

`QueryBuilder`를 사용하면 `leftJoinAndSelect` 대신 `innerJoinAndSelect`를 수행할 수 있습니다 (`LEFT JOIN`과 `INNER JOIN`의 차이점을 배우려면 SQL 문서를 참조하십시오), 조건별로 관계 데이터를 결합하고 순서를 지정할 수 있습니다. .

[`QueryBuilder`](./select-query-builder.md)에 대해 자세히 알아보세요.

## 관계 속성 초기화 방지

때로는 관계 속성을 초기화하는 것이 유용합니다. 예를 들면 다음과 같습니다.

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
    categories: Category[] = []; // = [], 여기에서 초기화

}
```

그러나 TypeORM 엔티티에서는 문제가 발생할 수 있습니다. 문제를 이해하기 위해 먼저 초기화 세트없이 질문 엔터티를 로드해 보겠습니다. 질문을로드 하면 다음과 같은 객체가 반환됩니다.

```javascript
Question {
    id: 1,
    title: "Question about ..."
}
```

이제 이 객체 `categories`를 내부에 저장하면 설정되지 않았기 때문에 건드리지 않습니다.

그러나 초기화가 있는 경우 로드된 객체는 다음과 같습니다.

```javascript
Question {
    id: 1,
    title: "Question about ...",
    categories: []
}
```

객체를 저장할 때 데이터베이스에 질문에 바인딩된 카테고리가 있는지 확인하고 모든 카테고리를 분리합니다. 왜? `[]`와 같은 관계 또는 그 안에 있는 항목은 무언가가 제거된 것으로 간주되기 때문에 객체가 객체에서 제거되었는지 여부를 확인할 다른 방법이 없습니다.

따라서 이와 같은 객체를 저장하면 문제가 발생합니다. 이전에 설정된 모든 카테고리가 제거됩니다.

이 행동을 피하는 방법? 엔터티에서 배열을 초기화하지 마십시오. 생성자에도 동일한 규칙이 적용됩니다. 생성자에서도 초기화하지 마십시오.

## 외래키 제약조건 생성방지

때로는 성능상의 이유로 엔터티간 관계를 원할 수 있지만 외래키 제약조건은 없습니다. `createForeignKeyConstraints`옵션(기본값: true)으로 외래키 제약조건을 생성해야 하는지 정의할 수 있습니다.

```typescript
import {Entity, PrimaryColumn, Column, ManyToOne} from "typeorm";
import {Person} from "./Person";

@Entity()
export class ActionLog {

    @PrimaryColumn()
    id: number;

    @Column()
    date: Date;

    @Column()
    action: string;

    @ManyToOne(type => Person, {
        createForeignKeyConstraints: false
    })
    person: Person;

}
```
