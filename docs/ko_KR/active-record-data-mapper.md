# 액티브 레코드 대 데이터 매퍼

- [액티브 레코드 패턴은 무엇입니까?](#액티브-레코드-패턴은-무엇입니까)
- [데이터 매퍼 패턴이란 무엇입니까?](#데이터-매퍼-패턴이란-무엇입니까)
- [어느 것을 선택해야합니까?](#어느-것을-선택해야합니까)

## 액티브 레코드 패턴은 무엇입니까?

TypeORM에서는 Active Record와 Data Mapper 패턴을 모두 사용할 수 있습니다.

Active Record 접근 방식을 사용하면 모델 자체내에서 모든 쿼리 메서드를 정의하고 모델 메서드를 사용하여 객체를 저장, 제거 및 로드합니다.

간단히 말해서 Active Record 패턴은 모델 내에서 데이터베이스에 액세스하는 접근 방식입니다.
Active Record 패턴에 대한 자세한 내용은 [위키피디어](https://en.wikipedia.org/wiki/Active_record_pattern)에서 확인할 수 있습니다.

예:

```typescript
import {BaseEntity, Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User extends BaseEntity {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

}
```

모든 액티브 레코드 엔터티는 엔터티 작업을 위한 메서드를 제공하는 `BaseEntity` 클래스를 확장해야합니다. 그러한 엔티티와 작업하는 방법의 예:

```typescript

// AR 엔티티를 저장하는 방법의 예
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await user.save();

// AR 엔티티를 제거하는 방법의 예
await user.remove();

// AR 엔티티를 로드하는 방법의 예
const users = await User.find({ skip: 2, take: 5 });
const newUsers = await User.find({ isActive: true });
const timber = await User.findOne({ firstName: "Timber", lastName: "Saw" });
```

`BaseEntity`는 표준 `Repository`의 대부분의 메소드를 가지고 있습니다. 대부분의 경우 액티브 레코드 엔터티와 함께 `Repository` 또는 `EntityManager`를 사용할 필요가 없습니다.

이제 사용자를 성과 이름으로 반환하는 함수를 만들고 싶다고 가정해 보겠습니다. `User` 클래스에서 정적 메서드와 같은 함수를 만들 수 있습니다.

```typescript
import {BaseEntity, Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User extends BaseEntity {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

    static findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany();
    }

}
```

다른 방법과 마찬가지로 사용하십시오.

```typescript
const timber = await User.findByName("Timber", "Saw");
```

## 데이터 매퍼 패턴이란 무엇입니까?

TypeORM에서는 Active Record와 Data Mapper 패턴을 모두 사용할 수 있습니다.

데이터 매퍼 접근 방식을 사용하면 "리포지토리" 라는 별도의 클래스에 모든 쿼리 메서드를 정의하고 리포지토리를 사용하여 개체를 저장, 제거 및 로드합니다. 데이터 매퍼에서 엔티티는 매우 멍청합니다. 속성을 정의하기만하면 "더미" 메소드가 있을 수 있습니다.

간단히 말해, 데이터 매퍼는 모델이 아닌 저장소내에서 데이터베이스에 액세스하는 접근 방식입니다. 데이터 매퍼에 대한 자세한 내용은 [위키피디아](https://en.wikipedia.org/wiki/Data_mapper_pattern)에서 확인할 수 있습니다.

예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

}
```
그러한 엔티티와 작업하는 방법의 예:

```typescript
const userRepository = connection.getRepository(User);

// DM 엔티티를 저장하는 방법의 예
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await userRepository.save(user);

// DM 엔티티를 제거하는 방법의 예
await userRepository.remove(user);

// DM 엔티티를 로드하는 방법의 예
const users = await userRepository.find({ skip: 2, take: 5 });
const newUsers = await userRepository.find({ isActive: true });
const timber = await userRepository.findOne({ firstName: "Timber", lastName: "Saw" });
```

이제 사용자를 성과 이름으로 반환하는 함수를 만들고 싶다고 가정해 보겠습니다. 이러한 기능을 "사용자 정의 저장소"에 만들 수 있습니다.

```typescript
import {EntityRepository, Repository} from "typeorm";
import {User} from "../entity/User";

@EntityRepository()
export class UserRepository extends Repository<User> {

    findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany();
    }

}
```

다음과 같이 사용하십시오.

```typescript
const userRepository = connection.getCustomRepository(UserRepository);
const timber = await userRepository.findByName("Timber", "Saw");
```

[커스텀 리포지토리](./custom-repository.md)에 대해 자세히 알아보세요.

## 어느 것을 선택해야합니까?

결정은 당신에게 달려 있습니다. 두 전략에는 각각 장단점이 있습니다.

소프트웨어 개발과 관련하여 항상 명심해야 할 한가지는 애플리케이션을 유지관리하는 방법입니다. `Data Mapper` 접근방식은 유지관리에 도움이 되며 **더 큰 앱**에서 더 효과적입니다. `Active record` 접근방식은 **작은 앱**에서 잘 작동하는 단순성을 유지하는 데 도움이됩니다. 그리고 단순성은 항상 더 나은 유지관리의 핵심입니다.
