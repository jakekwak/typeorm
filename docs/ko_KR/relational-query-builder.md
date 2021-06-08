# 관계와 작업하기

`RelationQueryBuilder`는 관계 작업을 할 수 있는 특별한 타입의 `QueryBuilder`입니다. 이를 사용하면 엔티티를 로드할 필요없이 데이터베이스에서 엔티티를 서로 바인딩하거나 관련 엔티티를 쉽게 로드할 수 있습니다.

예를 들어 `Post` 항목이 있고 `categories`라는 `Category`와 다대다 관계가 있습니다. 이 다대다 관계에 새 카테고리를 추가해 보겠습니다.

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post)
    .add(category);
```

이 코드는 다음을 수행하는 것과 동일합니다.

```typescript
import {getRepository} from "typeorm";

const postRepository = getRepository(Post);
const post = await postRepository.findOne(1, { relations: ["categories"] });
post.categories.push(category);
await postRepository.save(post);
```

그러나 부피가 큰 `save` 메서드 호출을 호출하는 것과 달리 최소한의 작업을 수행하고 데이터베이스의 항목을 바인딩하기 때문에 더 효율적입니다.

또한 이러한 접근방식의 또 다른 이점은 모든 관련 엔터티를 푸시하기 전에 로드할 필요가 없다는 것입니다. 예를 들어, 단일 게시물에 10,000 개의 카테고리가 있는 경우이 목록에 새 게시물을 추가하는 것이 문제가 될 수 있습니다. 이렇게하는 표준 방법은 모든 10,000 개의 카테고리가 포함된 게시물을 로드하고, 새 카테고리를 푸시하고, 저장해. 이는 매우 높은 성능 비용을 초래하고 기본적으로 생산 결과에 적용할 수 없습니다. 그러나 `RelationQueryBuilder`를 사용하면 이 문제가 해결됩니다.

또한 항목 ID를 대신 사용할 수 있으므로 항목을 "바인딩"할 때 실제로 항목을 사용할 필요가 없습니다. 예를 들어, id = 3 인 카테고리를 id = 1 인 게시물에 추가해 보겠습니다.

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(1)
    .add(3);
```

복합 기본키를 사용하는 경우 이를 id 맵으로 전달해야 합니다. 예를 들면 다음과 같습니다.

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of({ firstPostId: 1, secondPostId: 3 })
    .add({ firstCategoryId: 2, secondCategoryId: 4 });
```

추가하는 것과 동일한 방법으로 엔티티를 제거할 수 있습니다.

```typescript
import {getConnection} from "typeorm";

// 이 코드는 주어진 게시물에서 카테고리를 제거합니다.
await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // 게시물 ID도 사용할 수 있습니다.
    .remove(category); // 카테고리 ID도 사용할 수 있습니다.
```

관련항목 추가 및 제거는 `다대다` 및 `일대다` 관계에서 작동합니다. `일대일` 및 `다대일` 관계의 경우 대신 `set`를 사용합니다.

```typescript
import {getConnection} from "typeorm";

// 이 코드는 주어진 게시물의 카테고리를 설정합니다.
await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // 게시물 ID도 사용할 수 있습니다.
    .set(category); // 카테고리 ID도 사용할 수 있습니다.
```

관계 설정을 해제하려면(null로 설정) `set` 메서드에 `null`을 전달하면 됩니다.

```typescript
import {getConnection} from "typeorm";

// 이 코드는 주어진 게시물의 카테고리를 설정 해제합니다.
await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // 게시물 ID도 사용할 수 있습니다.
    .set(null);
```

관계 업데이트 외에도 관계형 쿼리 작성기를 사용하여 관계형 엔터티를 로드할 수 있습니다. 예를 들어, `Post` 엔티티 내부에 다대다 `categories` 관계와 다대일 `user` 관계가 있다고 가정해 보겠습니다. 이러한 관계를 로드하려면 다음 코드를 사용할 수 있습니다.

```typescript
import {getConnection} from "typeorm";

const post = await getConnection().manager.findOne(Post, 1);

post.categories = await getConnection()
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // 게시물 ID도 사용할 수 있습니다.
    .loadMany();

post.author = await getConnection()
    .createQueryBuilder()
    .relation(Post, "user")
    .of(post) // 게시물 ID도 사용할 수 있습니다.
    .loadOne();
```
