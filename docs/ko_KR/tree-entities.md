# 트리 엔티티

TypeORM은 트리 구조를 저장하기 위해 인접 목록 및 클로저 테이블 패턴을 지원합니다. 계층 구조 테이블에 대한 자세한 내용은 [Bill Karwin의 멋진 프레젠테이션](https://www.slideshare.net/billkarwin/models-for-hierarchical-data)을 참조하세요.

- [인접 목록](#인접-목록)
- [중첩 세트](#중첩-세트)
- [구체화된 경로(일명 경로 열거)](#구체화된-경로일명-경로-열거)
- [클로저 테이블](#클로저-테이블)
- [트리 엔티티와 작업하기](#트리-엔티티와-작업하기)


## 인접 목록

인접 목록은 자체 참조가 있는 간단한 모델입니다. 이 접근 방식의 이점은 단순성이며, 결점은 조인 제한으로 인해 한번에 큰 트리를 로드할 수 없다는 것입니다. 인접 목록의 이점과 사용에 대한 자세한 내용은 [Matthew Schinckel의 이 기사](http://schinckel.net/2014/09/13/long-live-adjacency-lists/)를 참조하십시오.

예:

```typescript
import {Entity, Column, PrimaryGeneratedColumn, ManyToOne, OneToMany} from "typeorm";

@Entity()
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @ManyToOne(type => Category, category => category.children)
    parent: Category;

    @OneToMany(type => Category, category => category.parent)
    children: Category[];
}

```

## 중첩 세트

중첩 세트는 데이터베이스에 트리 구조를 저장하는 또 다른 패턴입니다. 읽기에는 매우 효율적이지만 쓰기에는 좋지 않습니다. 중첩 세트에 여러 루트를 가질 수 없습니다.

예:

```typescript
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("nested-set")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;
}
```

## 구체화된 경로(일명 경로 열거)

구체화된 경로(경로 열거라고도 함)는 데이터베이스에 트리 구조를 저장하는 또 다른 패턴입니다. 간단하고 효과적입니다.

예:

```typescript
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("materialized-path")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;
}
```

## 클로저 테이블

클로저 테이블은 부모와 자식간의 관계를 특별한 방식으로 별도의 테이블에 저장합니다. 읽기와 쓰기 모두에서 효율적입니다.

예:

```typescript
import {Entity, Tree, Column, PrimaryGeneratedColumn, TreeChildren, TreeParent, TreeLevelColumn} from "typeorm";

@Entity()
@Tree("closure-table")
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @TreeChildren()
    children: Category[];

    @TreeParent()
    parent: Category;
}
```

선택적 매개변수 `options`를 `@Tree("closure-table", options) `로 설정하여 클로저 테이블 이름 및/또는 클로저 테이블 컬럼 이름을 지정할 수 있습니다. `ancestorColumnName` 및 `descandantColumnName`은 기본 컬럼의 메타 데이터를 수신하고 컬럼 이름을 반환하는 콜백 함수입니다.

```ts
@Tree("closure-table", {
    closureTableName: "category_closure",
    ancestorColumnName: (column) => "ancestor_" + column.propertyName,
    descendantColumnName: (column) => "descendant_" + column.propertyName,
})
```

> 노트:
> 구성요소의 부모 업데이트 또는 제거는 아직 구현되지 않았습니다 ([이 문제 참조](https://github.com/typeorm/typeorm/issues/2032)). 이러한 작업중 하나를 수행하려면 클로저 테이블을 명시적으로 업데이트해야 합니다.

## 트리 엔티티와 작업하기

바인드 트리 엔티티를 서로에게 연결하려면 하위 엔티티를 부모로 설정하고 저장하는 것이 중요합니다.

예를 들면 :

```typescript
const manager = getManager();

const a1 = new Category("a1");
a1.name = "a1";
await manager.save(a1);

const a11 = new Category();
a11.name = "a11";
a11.parent = a1;
await manager.save(a11);

const a12 = new Category();
a12.name = "a12";
a12.parent = a1;
await manager.save(a12);

const a111 = new Category();
a111.name = "a111";
a111.parent = a11;
await manager.save(a111);

const a112 = new Category();
a112.name = "a112";
a112.parent = a11;
await manager.save(a112);
```

이러한 트리를 로드하려면 `TreeRepository`를 사용하십시오.

```typescript
const manager = getManager();
const trees = await manager.getTreeRepository(Category).findTrees();
```

`trees`는 다음과 같습니다.

```json
[{
    "id": 1,
    "name": "a1",
    "children": [{
        "id": 2,
        "name": "a11",
        "children": [{
            "id": 4,
            "name": "a111"
        }, {
            "id": 5,
            "name": "a112"
        }]
    }, {
        "id": 3,
        "name": "a12"
    }]
}]
```

`TreeRepository`를 통해 트리 엔터티로 작업하는 다른 특수 메서드가 있습니다.

* `findTrees` - 모든 자식, 자식의 자식등과 함께 데이터베이스의 모든 트리를 반환합니다.

```typescript
const treeCategories = await repository.findTrees();
// 내부에 하위 범주가 있는 루트 범주를 반환합니다.
```

* `findRoots` - 루트는 조상이 없는 엔티티입니다. 그들 모두를 찾습니다. 자식 잎(leaf)을 로드하지 않습니다.

```typescript
const rootCategories = await repository.findRoots();
// 내부에 하위 범주가 없는 루트 범주를 반환합니다.
```

* `findDescendants` - 지정된 엔티티의 모든 하위(하위 항목)를 가져옵니다. 모두 플랫 배열로 반환합니다.

```typescript
const children = await repository.findDescendants(parentCategory);
// parentCategory의 모든 직접 하위 범주(중첩된 범주 없음)를 반환합니다.
```

* `findDescendantsTree` - 지정된 엔티티의 모든 하위(하위 항목)를 가져옵니다. 서로 중첩된 트리로 반환합니다.

```typescript
const childrenTree = await repository.findDescendantsTree(parentCategory);
// parentCategory의 모든 직접 하위 범주(중첩 범주 포함)를 반환합니다.
```

* `createDescendantsQueryBuilder` - 트리에서 항목의 하위 항목을 가져 오는데 사용되는 쿼리 작성기를 만듭니다.

```typescript
const children = await repository
    .createDescendantsQueryBuilder("category", "categoryClosure", parentCategory)
    .andWhere("category.type = 'secondary'")
    .getMany();
```

* `countDescendants` - 엔터티의 하위 항목 수를 가져옵니다.

```typescript
const childrenCount = await repository.countDescendants(parentCategory);
```

* `findAncestors` - 지정된 엔티티의 모든 상위(조상)를 가져옵니다. 모두 플랫 배열로 반환합니다.

```typescript
const parents = await repository.findAncestors(childCategory);
// 모든 직접 childCategory의 상위 범주를 반환합니다("부모의 부모" 제외).
```

* `findAncestorsTree` - 지정된 엔티티의 모든 상위(조상)를 가져옵니다. 서로 중첩된 트리로 반환합니다.

```typescript
const parentsTree = await repository.findAncestorsTree(childCategory);
// 모든 직접 childCategory의 상위 범주를 반환합니다("부모의 부모" 포함).
```

* `createAncestorsQueryBuilder` - 트리에서 항목의 상위 항목을 가져 오는데 사용되는 쿼리 작성기를 만듭니다.

```typescript
const parents = await repository
    .createAncestorsQueryBuilder("category", "categoryClosure", childCategory)
    .andWhere("category.type = 'secondary'")
    .getMany();
```

* `countAncestors` - 엔터티의 조상 수를 가져옵니다.

```typescript
const parentsCount = await repository.countAncestors(childCategory);
```
