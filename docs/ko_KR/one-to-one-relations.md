# 일대일 관계

일대일은 A에 B의 인스턴스가 하나만 포함되고 B에 A의 인스턴스가 하나만 포함된 관계입니다. 예를 들어 `User` 및 `Profile` 엔티티를 살펴보겠습니다. 사용자는 단일 프로필만 가질 수 있으며 단일 프로필은 단일 사용자만 소유합니다.

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

    @OneToOne(() => Profile)
    @JoinColumn()
    profile: Profile;

}
```
여기서는 `profile`에 `@OneToOne`을 추가하고 대상 관계 유형을 `Profile`로 지정합니다. 우리는 또한 필수적이고 관계의 한쪽에만 설정되어야하는 `@JoinColumn`을 추가했습니다. `@JoinColumn`을 설정한 쪽에서 해당 쪽의 테이블에는 대상 엔터티 테이블에 대한 "관계 ID"와 외래 키가 포함됩니다.

이 예제는 다음 테이블을 생성합니다.

```shell
+-------------+--------------+----------------------------+
|                        profile                          |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| gender      | varchar(255) |                            |
| photo       | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| profileId   | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

다시 말하지만, `@JoinColumn`은 관계의 한쪽, 즉 데이터베이스 테이블에 외래 키가 있어야하는 쪽에만 설정되어야합니다.

이러한 관계를 저장하는 방법의 예:

```typescript
const profile = new Profile();
profile.gender = "male";
profile.photo = "me.jpg";
await connection.manager.save(profile);

const user = new User();
user.name = 'Joe Smith';
user.profile = profile;
await connection.manager.save(user);
```

[캐스케이드](./relations.md#cascades)를 활성화하면 하나의 `save`호출만으로 이 관계를 저장할 수 있습니다.

내부 프로필과 함께 사용자를 로드하려면 `FindOptions`에서 관계를 지정해야합니다.

```typescript
const userRepository = connection.getRepository(User);
const users = await userRepository.find({ relations: ["profile"] });
```

또는 `QueryBuilder`를 사용하여 조인할 수 있습니다.

```typescript
const users = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.profile", "profile")
    .getMany();
```

관계에 대해 즉시로딩(eager)을 활성화하면 관계를 지정하거나 조인할 필요가 없습니다. 항상 자동으로 로드됩니다.

관계는 단방향 및 양방향일 수 있습니다. 단방향은 한쪽에서만 관계 데코레이터와의 관계입니다. 양방향은 관계의 양쪽에 있는 데코레이터와의 관계입니다.

우리는 단방향 관계를 생성했습니다. 양방향으로 만들어 보겠습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, OneToOne} from "typeorm";
import {User} from "./User";

@Entity()
export class Profile {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    gender: string;

    @Column()
    photo: string;

    @OneToOne(() => User, user => user.profile) // 두번째 매개변수로 역변 지정
    user: User;

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

    @OneToOne(() => Profile, profile => profile.user) // 두번째 매개변수로 역변 지정
    @JoinColumn()
    profile: Profile;

}
```

우리는 우리의 관계를 양방향으로 만들었습니다. 역 관계에는 `@JoinColumn`이 없습니다. `@JoinColumn`은 외래 키를 소유할 테이블에서 관계의 한쪽에만 있어야합니다.

양방향 관계를 사용하면 `QueryBuilder`를 사용하여 양쪽에서 관계를 결합할 수 있습니다.

```typescript
const profiles = await connection
    .getRepository(Profile)
    .createQueryBuilder("profile")
    .leftJoinAndSelect("profile.user", "user")
    .getMany();
```
