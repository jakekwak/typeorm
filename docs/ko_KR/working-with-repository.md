# 리포지토리란?

`Repository`는 `EntityManager`와 비슷하지만 그 작업은 구체적인 엔티티로 제한됩니다.

`getRepository(Entity)`, `Connection#getRepository` 또는 `EntityManager#getRepository`를 통해 저장소에 액세스할 수 있습니다.

예:

```typescript
import {getRepository} from "typeorm";
import {User} from "./entity/User";

const userRepository = getRepository(User); // getConnection().getRepository() 또는 getManager().getRepository()를 통해 가져올 수도 있습니다.
const user = await userRepository.findOne(1);
user.name = "Umed";
await userRepository.save(user);
```

세가지 유형의 저장소가 있습니다.
* `Repository` - 모든 엔티티에 대한 일반 저장소.
* `TreeRepository` - 트리 엔터티에 사용되는 `Repository`의 확장인 저장소 (`@Tree` 데코레이터로 표시된 엔티티와 같음). 트리 구조로 작업하는 특별한 방법이 있습니다.
* `MongoRepository` - MongoDB에서만 사용되는 특수 기능이 있는 저장소.
