# 엔터티 리스너 및 구독자

- [엔티티 리스너란?](#엔티티-리스너란)
  - [`@AfterLoad`](#afterload)
  - [`@BeforeInsert`](#beforeinsert)
  - [`@AfterInsert`](#afterinsert)
  - [`@BeforeUpdate`](#beforeupdate)
  - [`@AfterUpdate`](#afterupdate)
  - [`@BeforeRemove`](#beforeremove)
  - [`@AfterRemove`](#afterremove)
- [구독자란?](#구독자란)

## 엔티티 리스너란?

모든 엔터티에는 특정 엔터티 이벤트를 수신하는 사용자 지정 논리가 있는 메서드가 있을 수 있습니다. 수신하려는 이벤트에 따라 특수 데코레이터로 이러한 메서드를 표시해야합니다.

### `@AfterLoad`

엔터티에 임의의 이름으로 메서드를 정의하고 `@AfterLoad`로 표시하면 TypeORM이 `QueryBuilder` 또는 저장소 / 관리자 찾기 메서드를 사용하여 엔터티가 로드될 때마다 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterLoad()
    updateCounters() {
        if (this.likesCount === undefined)
            this.likesCount = 0;
    }
}
```

### `@BeforeInsert`

엔티티에 임의의 이름으로 메소드를 정의하고 `@BeforeInsert`로 표시하면 TypeORM이 저장소 / 관리자 `save`를 사용하여 엔티티가 삽입되기 전에 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @BeforeInsert()
    updateDates() {
        this.createdDate = new Date();
    }
}
```

### `@AfterInsert`

엔티티에 임의의 이름으로 메소드를 정의하고 `@AfterInsert`로 표시하면 TypeORM이 저장소 / 관리자 `save`를 사용하여 엔티티가 삽입된 후 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterInsert()
    resetCounters() {
        this.counters = 0;
    }
}
```

### `@BeforeUpdate`

엔티티에 임의의 이름으로 메소드를 정의하고 `@BeforeUpdate`로 표시하면 TypeORM이 저장소 / 관리자 `save`를 사용하여 기존 엔티티가 업데이트되기 전에 이를 호출합니다. 그러나 이것은 모델에서 정보가 변경된 경우에만 발생합니다. 모델을 수정하지 않고 `save`를 실행하면 `@BeforeUpdate`와 `@AfterUpdate`가 실행되지 않습니다.

예:

```typescript
@Entity()
export class Post {

    @BeforeUpdate()
    updateDates() {
        this.updatedDate = new Date();
    }
}
```

### `@AfterUpdate`

엔티티에 임의의 이름으로 메소드를 정의하고 `@ AfterUpdate`로 표시할 수 있으며 TypeORM은 저장소 / 관리자 `save`를 사용하여 기존 엔티티가 업데이트된 후 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterUpdate()
    updateCounters() {
        this.counter = 0;
    }
}
```

### `@BeforeRemove`

엔티티에 임의의 이름으로 메소드를 정의하고 `@BeforeRemove`로 표시하면 TypeORM이 저장소 / 관리자 `remove`를 사용하여 엔티티를 제거하기 전에 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @BeforeRemove()
    updateStatus() {
        this.status = "removed";
    }
}
```

### `@AfterRemove`

엔티티에 임의의 이름으로 메소드를 정의하고 `@AfterRemove`로 표시하면 TypeORM이 저장소 / 관리자 `remove`를 사용하여 엔티티가 제거된 후 이를 호출합니다.

예:

```typescript
@Entity()
export class Post {

    @AfterRemove()
    updateStatus() {
        this.status = "removed";
    }
}
```

## 구독자란?

특정 엔티티 이벤트 또는 엔티티 이벤트를 수신할 수 있는 이벤트 구독자로 클래스를 표시합니다. `QueryBuilder` 및 저장소 / 관리자 메소드를 사용하여 이벤트가 발생합니다.

예:

```typescript
@EventSubscriber()
export class PostSubscriber implements EntitySubscriberInterface<Post> {


    /**
     * 이 구독자가 Post 이벤트 만 수신함을 나타냅니다.
     */
    listenTo() {
        return Post;
    }

    /**
     * 포스트 삽입 전에 호출됩니다.
     */
    beforeInsert(event: InsertEvent<Post>) {
        console.log(`BEFORE POST INSERTED: `, event.entity);
    }

}
```

`EntitySubscriberInterface`의 모든 메소드를 구현할 수 있습니다. 엔티티를 수신하려면 `listenTo` 메소드를 생략하고 `any`를 사용하면됩니다.

```typescript
@EventSubscriber()
export class PostSubscriber implements EntitySubscriberInterface {

    /**
     * 엔티티가 로드된 후 호출됩니다.
     */
    afterLoad(entity: any) {
        console.log(`AFTER ENTITY LOADED: `, entity);
    }

    /**
     * 포스트 삽입 전에 호출됩니다.
     */
    beforeInsert(event: InsertEvent<any>) {
        console.log(`BEFORE POST INSERTED: `, event.entity);
    }

    /**
     * 엔티티 삽입 후 호출됩니다.
     */
    afterInsert(event: InsertEvent<any>) {
        console.log(`AFTER ENTITY INSERTED: `, event.entity);
    }

    /**
     * 엔티티 업데이트 전에 호출됩니다.
     */
    beforeUpdate(event: UpdateEvent<any>) {
        console.log(`BEFORE ENTITY UPDATED: `, event.entity);
    }

    /**
     * 엔티티 업데이트 후 호출됩니다.
     */
    afterUpdate(event: UpdateEvent<any>) {
        console.log(`AFTER ENTITY UPDATED: `, event.entity);
    }

    /**
     * 엔티티 제거 전에 호출됩니다.
     */
    beforeRemove(event: RemoveEvent<any>) {
        console.log(`BEFORE ENTITY WITH ID ${event.entityId} REMOVED: `, event.entity);
    }

    /**
     * 엔티티 제거 후 호출됩니다.
     */
    afterRemove(event: RemoveEvent<any>) {
        console.log(`AFTER ENTITY WITH ID ${event.entityId} REMOVED: `, event.entity);
    }

    /**
     * 트랜잭션 시작 전에 호출됩니다.
     */
    beforeTransactionStart(event: TransactionStartEvent) {
        console.log(`BEFORE TRANSACTION STARTED: `, event);
    }

    /**
     * 트랜잭션 시작 후 호출됩니다.
     */
    afterTransactionStart(event: TransactionStartEvent) {
        console.log(`AFTER TRANSACTION STARTED: `, event);
    }

    /**
     * 트랜잭션 커밋 전에 호출됩니다.
     */
    beforeTransactionCommit(event: TransactionCommitEvent) {
        console.log(`BEFORE TRANSACTION COMMITTED: `, event);
    }

    /**
     * 트랜잭션 커밋 후에 호출됩니다.
     */
    afterTransactionCommit(event: TransactionCommitEvent) {
        console.log(`AFTER TRANSACTION COMMITTED: `, event);
    }

    /**
     * 트랜잭션 롤백 전에 호출됩니다.
     */
    beforeTransactionRollback(event: TransactionRollbackEvent) {
        console.log(`BEFORE TRANSACTION ROLLBACK: `, event);
    }

    /**
     * 트랜잭션 롤백 후에 호출됩니다.
     */
    afterTransactionRollback(event: TransactionRollbackEvent) {
        console.log(`AFTER TRANSACTION ROLLBACK: `, event);
    }

}
```

`subscribers` 속성이 [Connection 옵션](./connection-options.md#일반적인-연결-옵션)에 설정되어 있는지 확인하여 TypeORM이 가입자를 로드합니다.
