# 유효성 검사 사용

유효성 검사를 사용하려면 [class-validator](https://github.com/pleerock/class-validator)를 사용하세요.
TypeORM과 함께 클래스 유효성 검사기를 사용하는 방법의 예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
import {Contains, IsInt, Length, IsEmail, IsFQDN, IsDate, Min, Max} from "class-validator";

@Entity()
export class Post {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    @Length(10, 20)
    title: string;

    @Column()
    @Contains("hello")
    text: string;

    @Column()
    @IsInt()
    @Min(0)
    @Max(10)
    rating: number;

    @Column()
    @IsEmail()
    email: string;

    @Column()
    @IsFQDN()
    site: string;

    @Column()
    @IsDate()
    createDate: Date;

}
```

유효성 검사:

```typescript
import {getManager} from "typeorm";
import {validate} from "class-validator";

let post = new Post();
post.title = "Hello"; // 통과해서는 안됩니다
post.text = "this is a great post about hell world"; // 통과해서는 안됩니다
post.rating = 11; // 통과해서는 안됩니다
post.email = "google.com"; // 통과해서는 안됩니다
post.site = "googlecom"; // 통과해서는 안됩니다

const errors = await validate(post);
if (errors.length > 0) {
    throw new Error(`Validation failed!`);
} else {
    await getManager().save(post);
}
```