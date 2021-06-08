# EntityManager란?

`EntityManager`를 사용하여 모든 엔티티를 관리(삽입, 업데이트, 삭제, 로드 등)할 수 있습니다. EntityManager는 한 장소에 있는 모든 엔티티 저장소의 모음과 같습니다.

`getManager()`또는 `Connection`을 통해 엔티티 관리자에 액세스할 수 있습니다.

사용 방법 예:

```typescript
import {getManager} from "typeorm";
import {User} from "./entity/User";

const entityManager = getManager(); // getConnection().manager를 통해서도 얻을 수 있습니다.
const user = await entityManager.findOne(User, 1);
user.name = "Umed";
await entityManager.save(user);
```
