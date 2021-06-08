# MongoDB

- [MongoDB 지원](#mongodb-지원)
- [엔터티 및 컬럼 정의](#엔터티-및-컬럼-정의)
- [하위 문서 정의 (임베디드 문서)](#하위-문서-정의-임베디드-문서)
- [`MongoEntityManager` 및 `MongoRepository` 사용](#mongoentitymanager-및-mongorepository-사용)

## MongoDB 지원

TypeORM에는 기본 MongoDB 지원이 있습니다. 대부분의 TypeORM 기능은 RDBMS 전용이며 이 페이지에는 모든 MongoDB 전용 기능 문서가 포함되어 있습니다.

## 엔터티 및 컬럼 정의

엔터티와 컬럼을 정의하는 것은 관계형 데이터베이스에서와 거의 동일하지만, 주요 차이점은 `@PrimaryColumn` 또는 `@PrimaryGeneratedColumn` 대신 `@ObjectIdColumn`을 사용해야 한다는 것입니다.

간단한 엔티티 예:

```typescript
import {Entity, ObjectID, ObjectIdColumn, Column} from "typeorm";

@Entity()
export class User {

    @ObjectIdColumn()
    id: ObjectID;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

}
```

그리고 이것이 앱을 부트스트랩하는 방법입니다.

```typescript
import {createConnection, Connection} from "typeorm";

const connection: Connection = await createConnection({
    type: "mongodb",
    host: "localhost",
    port: 27017,
    database: "test"
});
```

## 하위 문서 정의 (임베디드 문서)

MongoDB는 객체와 객체를 객체 내부(또는 문서 내부의 문서)에 저장하므로 TypeORM에서 동일한 작업을 수행할 수 있습니다.

```typescript
import {Entity, ObjectID, ObjectIdColumn, Column} from "typeorm";

export class Profile {

    @Column()
    about: string;

    @Column()
    education: string;

    @Column()
    career: string;

}
```

```typescript
import {Entity, ObjectID, ObjectIdColumn, Column} from "typeorm";

export class Photo {

    @Column()
    url: string;

    @Column()
    description: string;

    @Column()
    size: number;

    constructor(url: string, description: string, size: number) {
        this.url = url;
        this.description = description;
        this.size = size;
    }

}
```

```typescript
import {Entity, ObjectID, ObjectIdColumn, Column} from "typeorm";

@Entity()
export class User {

    @ObjectIdColumn()
    id: ObjectID;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column(type => Profile)
    profile: Profile;

    @Column(type => Photo)
    photos: Photo[];

}
```

이 엔티티를 저장하는 경우:

```typescript
import {getMongoManager} from "typeorm";

const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.profile = new Profile();
user.profile.about = "About Trees and Me";
user.profile.education = "Tree School";
user.profile.career = "Lumberjack";
user.photos = [
    new Photo("me-and-trees.jpg", "Me and Trees", 100),
    new Photo("me-and-chakram.jpg", "Me and Chakram", 200),
];

const manager = getMongoManager();
await manager.save(user);
```

다음 문서가 데이터베이스에 저장됩니다.

```json
{
    "firstName": "Timber",
    "lastName": "Saw",
    "profile": {
        "about": "About Trees and Me",
        "education": "Tree School",
        "career": "Lumberjack"
    },
    "photos": [
        {
            "url": "me-and-trees.jpg",
            "description": "Me and Trees",
            "size": 100
        },
        {
            "url": "me-and-chakram.jpg",
            "description": "Me and Chakram",
            "size": 200
        }
    ]
}
```

## `MongoEntityManager` 및 `MongoRepository` 사용

`EntityManager` 내에서 대부분의 메소드를 사용할 수 있습니다(`query` 및 `transaction`과 같은 RDBMS 전용 제외).

예를 들면 :

```typescript
import {getManager} from "typeorm";

const manager = getManager(); // 또는 connection.manager
const timber = await manager.findOne(User, { firstName: "Timber", lastName: "Saw" });
```

For MongoDB there is also a separate `MongoEntityManager` which extends `EntityManager`.

```typescript
import {getMongoManager} from "typeorm";

const manager = getMongoManager(); // 또는 connection.mongoManager
const timber = await manager.findOne(User, { firstName: "Timber", lastName: "Saw" });
```

Just like separate like `MongoEntityManager` there is a `MongoRepository` with extended `Repository`:

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User); // 또는 connection.getMongoRepository
const timber = await userRepository.findOne({ firstName: "Timber", lastName: "Saw" });
```

find()에서 고급 옵션을 사용하십시오.

Equal:

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
const timber = await userRepository.find({
  where: {
    firstName: {$eq: "Timber"},
  }
});
```

LessThan:

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
const timber = await userRepository.find({
  where: {
    age: {$lt: 60},
  }
});
```

In:

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
const timber = await userRepository.find({
  where: {
    firstName: {$in: ["Timber","Zhang"]},
  }
});
```

Not in:

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
const timber = await userRepository.find({
  where: {
firstName: {$not: {$in: ["Timber","Zhang"]}},
}
});
```

Or:

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
const timber = await userRepository.find({
  where: {
    $or: [
        {firstName:"Timber"},
        {firstName:"Zhang"}
      ]
  }
});
```

하위 문서 쿼리

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
// Education Tree School로 사용자 쿼리
const users = await userRepository.find({
  where: {
   'profile.education': { $eq: "Tree School"}
  }
});
```

하위 문서 배열 쿼리

```typescript
import {getMongoRepository} from "typeorm";

const userRepository = getMongoRepository(User);
// Query users with photos of size less than 500
const users = await userRepository.find({
  where: {
   'photos.size': { $lt: 500}
  }
});

```


`MongoEntityManager`와 `MongoRepository`에는 유용한 MongoDB 관련 메서드가 많이 포함되어 있습니다.

#### `createCursor`

MongoDB의 결과를 반복하는 데 사용할 수 있는 쿼리에 대한 커서를 만듭니다.

#### `createEntityCursor`

MongoDB의 결과를 반복하는 데 사용할 수 있는 쿼리에 대한 커서를 만듭니다. 이는 각 결과를 엔티티 모델로 변환하는 수정된 버전의 커서를 리턴합니다.

#### `aggregate`

컬렉션에 대해 집계 프레임워크 파이프라인을 실행합니다.

#### `bulkWrite`

유창한 API없이 bulkWrite 작업을 수행하십시오.

#### `count`

db에서 쿼리에 일치하는 문서 수를 계산합니다.

#### `createCollectionIndex`

db 및 컬렉션에 인덱스를 생성합니다.

#### `createCollectionIndexes`

컬렉션에 여러 인덱스를 생성합니다.이 방법은 MongoDB 2.6 이상에서만 지원됩니다. 이전 버전의 MongoDB에서는 지원되지 않는 명령 오류가 발생합니다. 색인 사양은 http://docs.mongodb.org/manual/reference/command/createIndexes/ 에 정의되어 있습니다.

#### `deleteMany`

MongoDB에서 여러 문서를 삭제합니다.

#### `deleteOne`

MongoDB에서 문서를 삭제합니다.

#### `distinct`

distinct 명령은 컬렉션 전체에서 지정된 키에 대한 고유값 목록을 반환합니다.

#### `dropCollectionIndex`

이 컬렉션에서 인덱스를 삭제합니다.

#### `dropCollectionIndexes`

컬렉션에서 모든 인덱스를 삭제합니다.

#### `findOneAndDelete`

하나의 원자적(atomic) 작업으로 문서를 찾아서 삭제하고 작업 기간 동안 쓰기 잠금이 필요합니다.

#### `findOneAndReplace`

문서를 찾아서 하나의 원자적(atomic) 작업으로 교체하고 작업 기간 동안 쓰기 잠금이 필요합니다.

#### `findOneAndUpdate`

문서를 찾아 하나의 원자적(atomic) 작업으로 업데이트하고 작업 기간 동안 쓰기 잠금이 필요합니다.

#### `geoHaystackSearch`

컬렉션에서 geo haystack 인덱스를 사용하여 지역 검색을 실행합니다.

#### `geoNear`

컬렉션에서 항목을 검색하려면 geoNear 명령을 실행합니다.

#### `group`

컬렉션에서 그룹 명령을 실행합니다.

#### `collectionIndexes`

컬렉션의 모든 인덱스를 검색합니다.

#### `collectionIndexExists`

컬렉션에 인덱스가 있는지 검색

#### `collectionIndexInformation`

이 컬렉션 인덱스 정보를 검색합니다.

#### `initializeOrderedBulkOp`

In order 대량 쓰기 작업을 시작하면 작업이 추가된 순서대로 순차적으로 실행되어 유형의 각 스위치에 대해 새 작업이 생성됩니다.

#### `initializeUnorderedBulkOp`

비 순차 일괄 쓰기 작업을 시작합니다. 모든 작업은 순서없이 실행되는 삽입 / 업데이트 / 제거 명령에 버퍼링됩니다.

#### `insertMany`

MongoDB에 문서 배열을 삽입합니다.

#### `insertOne`

MongoDB에 단일 문서를 삽입합니다.

#### `isCapped`

컬렉션이 제한 컬렉션인 경우 반환합니다.

#### `listCollectionIndexes`

컬렉션에 대한 모든 인덱스 정보 목록을 가져옵니다.

#### `mapReduce`

컬렉션 전체에서 Map Reduce를 실행합니다. out에 대한 인라인 옵션은 컬렉션이 아닌 결과 배열을 반환합니다.

#### `parallelCollectionScan`

전체 컬렉션의 병렬 읽기를 허용하는 컬렉션에 대해 N 개의 병렬 커서를 반환합니다. 반환된 결과에 대한 주문 보장이 없습니다.

#### `reIndex`

컬렉션의 모든 인덱스 다시 색인화 경고: reIndex는 차단 작업(인덱스가 포그라운드에서 다시 작성됨)이며 대규모 컬렉션의 경우 속도가 느립니다.

#### `rename`

기존 컬렉션의 이름을 변경합니다.

#### `replaceOne`

MongoDB에서 문서를 바꿉니다.

#### `stats`

모든 수집 통계를 가져옵니다.

#### `updateMany`

필터를 기반으로 컬렉션내의 여러 문서를 업데이트합니다.

#### `updateOne`

필터를 기반으로 컬렉션내의 단일 문서를 업데이트합니다.
