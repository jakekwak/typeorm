# 엔티티 정의 분리

- [스키마 정의](#스키마-정의)
- [스키마 확장](#스키마-확장)
- [스키마를 사용하여 데이터 쿼리 / 삽입](#스키마를-사용하여-데이터-쿼리--삽입)

## 스키마 정의

데코레이터를 사용하여 모델에서 바로 엔티티와 해당 컬럼을 정의할 수 있습니다. 그러나 어떤 사람들은 TypeORM에서 "엔티티 스키마"라고 하는 별도의 파일내에 엔터티와 해당 컬럼을 정의하는 것을 선호합니다.

간단한 정의 예:

```ts
import {EntitySchema} from "typeorm";

export const CategoryEntity = new EntitySchema({
    name: "category",
    columns: {
        id: {
            type: Number,
            primary: true,
            generated: true
        },
        name: {
            type: String
        }
    }
});
```

관계가 있는 예:

```ts
import {EntitySchema} from "typeorm";

export const PostEntity = new EntitySchema({
    name: "post",
    columns: {
        id: {
            type: Number,
            primary: true,
            generated: true
        },
        title: {
            type: String
        },
        text: {
            type: String
        }
    },
    relations: {
        categories: {
            type: "many-to-many",
            target: "category" // CategoryEntity
        }
    }
});
```

복잡한 예:

```ts
import {EntitySchema} from "typeorm";

export const PersonSchema = new EntitySchema({
    name: "person",
    columns: {
        id: {
            primary: true,
            type: "int",
            generated: "increment"
        },
        firstName: {
            type: String,
            length: 30
        },
        lastName: {
            type: String,
            length: 50,
            nullable: false
        },
        age: {
            type: Number,
            nullable: false
        }
    },
    checks: [
        { expression: `"firstName" <> 'John' AND "lastName" <> 'Doe'` },
        { expression: `"age" > 18` }
    ],
    indices: [
        {
            name: "IDX_TEST",
            unique: true,
            columns: [
                "firstName",
                "lastName"
            ]
        }
    ],
    uniques: [
        {
            name: "UNIQUE_TEST",
            columns: [
                "firstName",
                "lastName"
            ]
        }
    ]
});
```

엔터티를 형식이 안전하게 유지하려면 모델을 정의하고 스키마 정의에 지정할 수 있습니다.

```ts
import {EntitySchema} from "typeorm";

export interface Category {
    id: number;
    name: string;
}

export const CategoryEntity = new EntitySchema<Category>({
    name: "category",
    columns: {
        id: {
            type: Number,
            primary: true,
            generated: true
        },
        name: {
            type: String
        }
    }
});
```

## 스키마 확장

`Decorator` 접근방식을 사용하면 기본 컬럼을 추상 클래스로 `확장`하고 간단히 확장할 수 있습니다. 예를 들어, `id`, `createdAt` 및 `updatedAt` 컬럼은 이러한 `BaseEntity`에 정의될 수 있습니다. 자세한 내용은 [구체적인 테이블 상속](entity-inheritance.md##구체적인-테이블-상속)에 대한 문서를 참조하세요.

`EntitySchema` 접근방식을 사용할 때는 불가능합니다. 그러나 `Spread Operator` (`...`)를 유리하게 사용할 수 있습니다.

위의 `Category` 예를 다시 생각해보세요. 기본 컬럼 설명을 `추출`하여 다른 스키마에서 재사용할 수 있습니다. 다음과 같은 방법으로 수행할 수 있습니다.

```ts
import {EntitySchemaColumnOptions} from "typeorm";

export const BaseColumnSchemaPart = {
  id: {
    type: Number,
    primary: true,
    generated: true,
  } as EntitySchemaColumnOptions,
  createdAt: {
    name: 'created_at',
    type: 'timestamp with time zone',
    createDate: true,
  } as EntitySchemaColumnOptions,
  updatedAt: {
    name: 'updated_at',
    type: 'timestamp with time zone',
    updateDate: true,
  } as EntitySchemaColumnOptions,
};
```

이제 다음과 같이 다른 스키마 모델에서 `BaseColumnSchemaPart`를 사용할 수 있습니다.

```ts
export const CategoryEntity = new EntitySchema<Category>({
    name: "category",
    columns: {
        ...BaseColumnSchemaPart,
        // 이제 CategoryEntity에 정의된 id, createdAt, updatedAt 컬럼이 있습니다!
        // 또한 다음과 같은 새로운 필드가 정의됩니다.
        name: {
            type: String
        }
    }
});
```

`Category` 인터페이스에도 `extended` 컬럼을 추가해야 합니다(예: `export interface Category extend BaseEntity`를 통해).

## 스키마를 사용하여 데이터 쿼리 / 삽입

물론 데코레이터를 사용하는 것처럼 저장소 또는 엔티티 관리자에서 정의된 스키마를 사용할 수 있습니다. 데이터를 가져오거나 데이터베이스를 조작하기 위해 이전에 정의된 `Category` 예제(`Interface` 및 `CategoryEntity` 스키마 포함)를 고려하십시오.

```ts
// request data
const categoryRepository = getRepository<Category>(CategoryEntity);
const category = await categoryRepository.findOne(1); // 카테고리가 올바르게 입력되었습니다!

// 데이터베이스에 새 카테고리 삽입
const categoryDTO = {
  // ID는 자동 생성됩니다. 위의 스키마 참조
  name: 'new category',
};
const newCategory = await categoryRepository.save(categoryDTO);
```
