# 인덱스

- [컬럼 인덱스](#컬럼-인덱스)
- [고유한 인덱스](#고유한-인덱스)
- [여러 컬럼이 있는 인덱스](#여러-컬럼이-있는-인덱스)
- [공간 인덱스](#공간-인덱스)
- [동기화 비활성화](#동기화-비활성화)

## 컬럼 인덱스

인덱스를 만들려는 컬럼에 `@Index`를 사용하여 특정 컬럼에 대한 데이터베이스 인덱스를 생성할 수 있습니다. 엔터티의 모든 컬럼에 대한 인덱스를 만들 수 있습니다.

예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column, Index} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Index()
    @Column()
    firstName: string;

    @Column()
    @Index()
    lastName: string;
}
```

인덱스 이름을 지정할 수도 있습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, Index} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Index("name1-idx")
    @Column()
    firstName: string;

    @Column()
    @Index("name2-idx")
    lastName: string;
}
```

## 고유한 인덱스

고유한 인덱스을 생성하려면 인덱스 옵션에 `{unique: true}`를 지정해야합니다.

> 참고: CockroachDB는 고유 인덱스를 `UNIQUE` 제약조건으로 저장합니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column, Index} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Index({ unique: true })
    @Column()
    firstName: string;

    @Column()
    @Index({ unique: true })
    lastName: string;
}
```

## 여러 컬럼이 있는 인덱스

여러 컬럼으로 색인을 생성하려면 항목 자체에 `@Index`를 입력하고 색인에 포함되어야하는 모든 컬럼 속성 이름을 지정해야합니다.

예:

```typescript
import {Entity, PrimaryGeneratedColumn, Column, Index} from "typeorm";

@Entity()
@Index(["firstName", "lastName"])
@Index(["firstName", "middleName", "lastName"], { unique: true })
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    middleName: string;

    @Column()
    lastName: string;

}
```

## 공간 인덱스

MySQL과 PostgreSQL(PostGIS를 사용할 수 있는 경우)은 모두 공간 인덱스를 지원합니다.

MySQL의 컬럼에 공간 인덱스를 만들려면 공간 타입(`geometry`, `point`, `linestring`, `polygon`, `multipoint`, `multilinestring`, `multipolygon`, `geometrycollection`)을 사용하는 열에 `spatial: true`가 있는 `Index`를 추가합니다.

```typescript
@Entity()
export class Thing {
    @Column("point")
    @Index({ spatial: true })
    point: string;
}
```

PostgreSQL의 컬럼에 공간 인덱스를 만들려면 공간 타입(`geometry`, `geography`)을 사용하는 컬럼에 `spatial: true`가 있는 `Index`를 추가합니다.

```typescript
export interface Geometry {
  type: "Point";
  coordinates: [Number, Number];
}

@Entity()
export class Thing {
    @Column("geometry", {
      spatialFeatureType: "Point",
      srid: 4326
    })
    @Index({ spatial: true })
    point: Geometry;
}
```

## 동기화 비활성화

TypeORM은 다양한 데이터베이스 특성과 기존 데이터베이스 색인에 대한 정보를 가져와 자동으로 동기화하는데 여러 문제가 있기 때문에 일부 색인옵션 및 정의(예: `lower`, `pg_trgm`)를 지원하지 않습니다. 이러한 경우 원하는 인덱스 서명을 사용하여 수동으로(예: 마이그레이션에서) 인덱스를 만들어야합니다. 동기화중에 TypeORM이 이러한 인덱스를 무시하도록하려면 `@Index` 데코레이터에서 `synchronize: false` 옵션을 사용하십시오.

예를 들어 대소 문자를 구분하지 않는 비교를 사용하여 색인을 생성합니다.

```sql
CREATE INDEX "POST_NAME_INDEX" ON "post" (lower("name"))
```

그 후에는 다음 스키마 동기화시 삭제되지 않도록 이 인덱스에 대한 동기화를 비활성화해야합니다.

```ts
@Entity()
@Index("POST_NAME_INDEX", { synchronize: false })
export class Post {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

}
```
