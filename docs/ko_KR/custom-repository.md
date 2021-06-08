# 커스텀 리포지토리

데이터베이스 작업을 위한 메서드를 포함해야 하는 사용자 지정 저장소를 만들 수 있습니다. 일반적으로 사용자 지정 저장소는 단일 엔터티에 대해 생성되며 특정 쿼리를 포함합니다. 예를 들어, 주어진 성과 이름으로 사용자를 검색하는 `findByName(firstName: string,  lastName: string)`이라는 메서드가 필요하다고 가정해 보겠습니다. 이 방법의 가장 좋은 위치는 `Repository`입니다. 그래서 우리는 `userRepository.findByName(...)`과 같이 부를 수 있습니다. 사용자 지정 저장소를 사용하여 이를 수행할 수 있습니다.

사용자 지정 저장소를 만드는 방법에는 여러가지가 있습니다.

* [Custom repository extends standard Repository](#custom-repository-extends-standard-repository)
* [Custom repository extends standard AbstractRepository](#custom-repository-extends-standard-abstractrepository)
* [Custom repository without extends](#custom-repository-without-extends)
* [Using custom repositories in transactions](#using-custom-repositories-in-transactions-or-why-custom-repositories-cannot-be-services)

## 사용자 지정 저장소는 표준 저장소를 확장합니다.

커스텀 저장소를 만드는 첫번째 방법은 `Repository`를 확장하는 것입니다.

예:

```typescript
import {EntityRepository, Repository} from "typeorm";
import {User} from "../entity/User";

@EntityRepository(User)
export class UserRepository extends Repository<User> {

    findByName(firstName: string, lastName: string) {
        return this.findOne({ firstName, lastName });
    }

}
```

그런 다음 다음과 같이 사용할 수 있습니다.

```typescript
import {getCustomRepository} from "typeorm";
import {UserRepository} from "./repository/UserRepository";

const userRepository = getCustomRepository(UserRepository); // 또는 connection.getCustomRepository 또는 manager.getCustomRepository()
const user = userRepository.create(); // const user = new User();와 동일
user.firstName = "Timber";
user.lastName = "Saw";
await userRepository.save(user);

const timber = await userRepository.findByName("Timber", "Saw");
```

보시다시피 `getCustomRepository`를 사용하여 저장소를 "가져올" 수 있습니다.
내부에 생성된 모든 메소드와 표준 엔티티 저장소의 모든 메소드에 액세스할 수 있습니다.

## 사용자 지정 저장소는 표준 AbstractRepository를 확장합니다.

사용자 지정 저장소를 만드는 두번째 방법은 `AbstractRepository`를 확장하는 것입니다.

```typescript
import {EntityRepository, AbstractRepository} from "typeorm";
import {User} from "../entity/User";

@EntityRepository(User)
export class UserRepository extends AbstractRepository<User> {

    createAndSave(firstName: string, lastName: string) {
        const user = new User();
        user.firstName = firstName;
        user.lastName = lastName;
        return this.manager.save(user);
    }

    findByName(firstName: string, lastName: string) {
        return this.repository.findOne({ firstName, lastName });
    }

}
```

그런 다음 다음과 같이 사용할 수 있습니다.

```typescript
import {getCustomRepository} from "typeorm";
import {UserRepository} from "./repository/UserRepository";

const userRepository = getCustomRepository(UserRepository); // 또는 connection.getCustomRepository 또는 manager.getCustomRepository()
await userRepository.createAndSave("Timber", "Saw");
const timber = await userRepository.findByName("Timber", "Saw");
```

이 타입의 저장소와 이전 저장소의 차이점은 `Repository`에 있는 모든 메소드를 노출하지 않는다는 것입니다. `AbstractRepository`에는 공개 메소드가 없으며 자체 공개 메소드에서 사용할 수 있는 `manager` 및 `repository`와 같은 보호된 메소드만 있습니다. `AbstractRepository`를 확장하는 것은 표준 `Repository`가 가진 모든 메소드를 공개하고 싶지 않은 경우 유용합니다.

## 확장이 없는 사용자 정의 저장소

저장소를 만드는 세번째 방법은 아무것도 확장하지 않는 것입니다.
그러나 항상 엔티티 관리자 인스턴스를 허용하는 생성자를 정의하십시오.

```typescript
import {EntityRepository, Repository, EntityManager} from "typeorm";
import {User} from "../entity/User";

@EntityRepository()
export class UserRepository {

    constructor(private manager: EntityManager) {
    }

    createAndSave(firstName: string, lastName: string) {
        const user = new User();
        user.firstName = firstName;
        user.lastName = lastName;
        return this.manager.save(user);
    }

    findByName(firstName: string, lastName: string) {
        return this.manager.findOne(User, { firstName, lastName });
    }

}
```

그런 다음 다음과 같이 사용할 수 있습니다.

```typescript
import {getCustomRepository} from "typeorm";
import {UserRepository} from "./repository/UserRepository";

const userRepository = getCustomRepository(UserRepository); // 또는 connection.getCustomRepository 또는 manager.getCustomRepository()
await userRepository.createAndSave("Timber", "Saw");
const timber = await userRepository.findByName("Timber", "Saw");
```

이 타입의 저장소는 아무것도 확장하지 않습니다. `EntityManager`를 허용해야 하는 생성자만 정의하면됩니다. 그런 다음 저장소 메서드의 모든 곳에서 사용할 수 있습니다. 또한 이러한 타입의 저장소는 특정 엔터티에 바인딩되지 않으므로 내부에 여러 엔터티를 사용할 수 있습니다.

## 트랜잭션에서 사용자 지정 저장소 사용 또는 사용자 지정 저장소가 서비스가 될 수 없는 이유

앱에 사용자 지정 저장소(일반 저장소 또는 엔티티 관리자와 마찬가지로)의 단일 인스턴스가 없기 때문에 사용자 지정 저장소는 서비스가 될 수 없습니다. 앱에 여러 연결이 있을 수 있다는 사실 외에도(엔티티 관리자와 저장소가 다른 경우) 저장소와 관리자는 트랜잭션에서도 다릅니다.

예를 들면 :

```typescript
await connection.transaction(async manager => {
    // 트랜잭션에서 반드시 트랜잭션이 제공하는 관리자 인스턴스를 사용해야합니다.
    // 이 관리자는 독점적이고 트랜잭션적이므로 글로벌 관리자, 저장소 또는 사용자 정의 저장소를 사용할 수 없습니다.
    // 사용자 정의 저장소를 서비스로 수행한다고 가정하면 "관리자" 속성이 있어야합니다.
    // EntityManager의 고유한 인스턴스이지만 전역 EntityManager 인스턴스가 없으며
    // 사용자 정의 관리자가 각 EntityManager에 고유하고 서비스가 될 수 없는 이유일 수 없습니다.
    // 이것은 또한 문제없이 트랜잭션에서 사용자 정의 저장소를 사용할 수 있는 기회를 엽니다.

    const userRepository = manager.getCustomRepository(UserRepository); // 여기에서 GLOBAL set Custom Repository를 사용하지 마십시오!
    await userRepository.createAndSave("Timber", "Saw");
    const timber = await userRepository.findByName("Timber", "Saw");
});
```
