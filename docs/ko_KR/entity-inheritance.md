# 엔티티 상속

- [구체적인 테이블 상속](#구체적인-테이블-상속)
- [단일 테이블 상속](#단일-테이블-상속)
- [임베디드 사용](#임베디드-사용)

## 구체적인 테이블 상속

엔티티 상속 패턴을 사용하여 코드의 중복을 줄일 수 있습니다. 가장 간단하고 효과적인 방법은 구체적인 테이블 상속입니다.

예를 들어 `Photo`, `Question`, `Post` 항목이 있습니다.

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
```

```typescript
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
```

```typescript
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

보시다시피 모든 항목에는 `id`, `title`, `description`과 같은 공통 컬럼이 있습니다. 중복을 줄이고 더 나은 추상화를 생성하기 위해 `Content`라는 기본 클래스를 만들 수 있습니다.

```typescript
export abstract class Content {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

}
```

```typescript
@Entity()
export class Photo extends Content {

    @Column()
    size: string;

}
```

```typescript
@Entity()
export class Question extends Content {

    @Column()
    answersCount: number;

}
```

```typescript
@Entity()
export class Post extends Content {

    @Column()
    viewCount: number;

}
```

상위 항목(상위 항목도 다른 항목을 확장할 수 있음)의 모든 컬럼(관계, 삽입 등)은 최종항목에서 상속되고 생성됩니다.

이 예에서는 `photo`, `question` 및 `post`의 3개 테이블을 만듭니다.

## 단일 테이블 상속

TypeORM은 단일 테이블 상속도 지원합니다. 단일 테이블 상속은 고유한 속성을 가진 여러 클래스가 있지만 데이터베이스에서는 동일한 테이블에 저장되는 경우 패턴입니다.

```typescript
@Entity()
@TableInheritance({ column: { type: "varchar", name: "type" } })
export class Content {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

}
```

```typescript
@ChildEntity()
export class Photo extends Content {

    @Column()
    size: string;

}
```

```typescript
@ChildEntity()
export class Question extends Content {

    @Column()
    answersCount: number;

}
```

```typescript
@ChildEntity()
export class Post extends Content {

    @Column()
    viewCount: number;

}
```

그러면 `content`라는 단일 테이블이 생성되고 사진, 질문 및 게시물의 모든 인스턴스가이 테이블에 저장됩니다.

## 임베디드 사용

`임베디드 컬럼`을 사용하여 앱의 중복을 줄이는 놀라운 방법이 있습니다 (상속보다 구성(Composition) 사용).
임베디드 엔티티에 대한 자세한 내용은 [여기](./embedded-entities.md)를 참조하십시오.
