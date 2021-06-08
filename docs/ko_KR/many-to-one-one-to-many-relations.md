# 다대일 / 일대다 관계

다대일 / 일대다는 A에 B의 여러 인스턴스가 포함되어 있지만 B에는 A의 인스턴스가 하나만 포함된 관계입니다.
예를 들어 `User` 및 `Photo` 엔티티를 살펴 보겠습니다.
사용자는 여러장의 사진을 가질 수 있지만 각 사진은 한명의 사용자만 소유합니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, ManyToOne} from "typeorm";
import {User} from "./User";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

    @ManyToOne(() => User, user => user.photos)
    user: User;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column, OneToMany} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(() => Photo, photo => photo.user)
    photos: Photo[];

}
```

여기서는 `photos` 속성에 `@OneToMany`를 추가하고 대상 관계 타입을 `Photo`로 지정했습니다. `@ManyToOne` / `@OneToMany` 관계에서 `@JoinColumn`을 생략할 수 있습니다. `@OneToMany`는 `@ManyToOne` 없이는 존재할 수 없습니다. `@OneToMany`를 사용하려면 `@ManyToOne`이 필요합니다. 그러나 그 반대는 필요하지 않습니다. `@ManyToOne` 관계만 신경 쓰는 경우 관련 엔터티에 `@OneToMany`없이 정의할 수 있습니다. `@ManyToOne`을 설정하는 경우 - 관련 엔티티에 "관계 ID"와 외래 키가 있습니다.

이 예제는 다음 테이블을 생성합니다.

```shell
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| url         | varchar(255) |                            |
| userId      | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

이러한 관계를 저장하는 방법의 예:

```typescript
const photo1 = new Photo();
photo1.url = "me.jpg";
await connection.manager.save(photo1);

const photo2 = new Photo();
photo2.url = "me-and-bears.jpg";
await connection.manager.save(photo2);

const user = new User();
user.name = "John";
user.photos = [photo1, photo2];
await connection.manager.save(user);
```

또는 다음을 수행할 수 있습니다.

```typescript
const user = new User();
user.name = "Leo";
await connection.manager.save(user);

const photo1 = new Photo();
photo1.url = "me.jpg";
photo1.user = user;
await connection.manager.save(photo1);

const photo2 = new Photo();
photo2.url = "me-and-bears.jpg";
photo2.user = user;
await connection.manager.save(photo2);
```

[캐스케이드](./relations.md#cascades)를 사용하면 `save` 호출을 한번만 사용하여 이 관계를 저장할 수 있습니다.

내부에 사진이 있는 사용자를 로드하려면 `FindOptions`에서 관계를 지정해야합니다.

```typescript
const userRepository = connection.getRepository(User);
const users = await userRepository.find({ relations: ["photos"] });

// 또는 반대쪽에서

const photoRepository = connection.getRepository(Photo);
const photos = await photoRepository.find({ relations: ["user"] });
```

또는 `QueryBuilder`를 사용하여 조인할 수 있습니다.

```typescript
const users = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .getMany();

// 또는 반대쪽에서

const photos = await connection
    .getRepository(Photo)
    .createQueryBuilder("photo")
    .leftJoinAndSelect("photo.user", "user")
    .getMany();
```

관계에 대해 즉시 로딩을 활성화하면 관계를 지정하거나 조인할 필요가 없습니다. 항상 자동으로 로드됩니다.
