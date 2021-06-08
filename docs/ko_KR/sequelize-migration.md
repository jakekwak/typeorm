# Sequelize에서 TypeORM으로 마이그레이션

- [연결 설정](#연결-설정)
- [스키마 동기화](#스키마-동기화)
- [모델 생성](#모델-생성)
- [기타 모델 설정](#기타-모델-설정)

## 연결 설정

sequelize에서는 다음과 같이 연결을 만듭니다.

```javascript
const sequelize = new Sequelize("database", "username", "password", {
  host: "localhost",
  dialect: "mysql"
});

sequelize
  .authenticate()
  .then(() => {
    console.log("Connection has been established successfully.");
  })
  .catch(err => {
    console.error("Unable to connect to the database:", err);
  });
```

TypeORM에서 다음과 같은 연결을 만듭니다.

```typescript
import {createConnection} from "typeorm";

createConnection({
    type: "mysql",
    host: "localhost",
    username: "username",
    password: "password"
}).then(connection => {
    console.log("Connection has been established successfully.");
})
.catch(err => {
    console.error("Unable to connect to the database:", err);
});
```

그런 다음 `getConnection`을 사용하여 앱의 어느 곳에서나 연결 인스턴스를 가져올 수 있습니다.

[커넥션](./connection.md)에 대해 자세히 알아보기

## 스키마 동기화

sequelize에서는 다음과 같이 스키마 동기화를 수행합니다.

```javascript
Project.sync({force: true});
Task.sync({force: true});
```

TypeORM에서는 연결 옵션에 `synchronize: true`를 추가하기만 하면됩니다.

```typescript
createConnection({
    type: "mysql",
    host: "localhost",
    username: "username",
    password: "password",
    synchronize: true
});
```

## 모델 생성

sequelize에서 모델을 정의하는 방법은 다음과 같습니다.

```javascript
module.exports = function(sequelize, DataTypes) {

    const Project = sequelize.define("project", {
      title: DataTypes.STRING,
      description: DataTypes.TEXT
    });

    return Project;

};
```

```javascript
module.exports = function(sequelize, DataTypes) {

    const Task = sequelize.define("task", {
      title: DataTypes.STRING,
      description: DataTypes.TEXT,
      deadline: DataTypes.DATE
    });

    return Task;
};
```

TypeORM에서는 이러한 모델을 엔티티라고하며 다음과 같이 정의할 수 있습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class Project {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class Task {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column("text")
    description: string;

    @Column()
    deadline: Date;

}
```

파일당 하나의 엔티티 클래스를 정의하는 것이 좋습니다. TypeORM을 사용하면 클래스를 데이터베이스 모델로 사용할 수 있으며 모델의 어떤 부분이 데이터베이스 테이블의 일부가 될지 정의하는 선언적 방법을 제공합니다. TypeScript의 강력한 기능은 클래스에서 사용할 수 있는 타입힌트 및 기타 유용한 기능을 제공합니다.

[엔티티 및 컬럼](./entities.md)에 대해 자세히 알아보기

## 기타 모델 설정

sequelize에서는 다음과 같습니다.

```javascript
flag: { type: Sequelize.BOOLEAN, allowNull: true, defaultValue: true },
```

다음과 같이 TypeORM에서 얻을 수 있습니다.

```typescript
@Column({ nullable: true, default: true })
flag: boolean;
```

sequelize에서는 다음과 같습니다.

```javascript
flag: { type: Sequelize.DATE, defaultValue: Sequelize.NOW }
```

TypeORM에서 다음과 같이 작성됩니다.

```typescript
@Column({ default: () => "NOW()" })
myDate: Date;
```

sequelize에서는 다음과 같습니다.

```javascript
someUnique: { type: Sequelize.STRING, unique: true },
```

다음과 같이 TypeORM에서 얻을 수 있습니다.

```typescript
@Column({ unique: true })
someUnique: string;
```

sequelize에서는 다음과 같습니다.

```javascript
fieldWithUnderscores: { type: Sequelize.STRING, field: "field_with_underscores" },
```

TypeORM에서 다음과 같이 작성됩니다.

```typescript
@Column({ name: "field_with_underscores" })
fieldWithUnderscores: string;
```

sequelize에서는 다음과 같습니다.

```javascript
incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },
```

TypeORM에서 다음과 같이 작성됩니다.

```typescript
@Column()
@Generated()
incrementMe: number;
```

sequelize에서는 다음과 같습니다.

```javascript
identifier: { type: Sequelize.STRING, primaryKey: true },
```

TypeORM에서 다음과 같이 작성됩니다.

```typescript
@Column({ primary: true })
identifier: string;
```

`createDate` 및 `updateDate`와 유사한 컬럼을 생성하려면 엔티티에 두 개의 컬럼(원하는 이름)을 정의해야합니다.

```typescript
@CreateDateColumn();
createDate: Date;

@UpdateDateColumn();
updateDate: Date;
```

### 모델을 작업하기

sequelize에서 새 모델을 만들고 저장하려면 다음을 작성하십시오.

```javascript
const employee = await Employee.create({ name: "John Doe", title: "senior engineer" });
```

TypeORM에는 새 모델을 만들고 저장하는 몇 가지 방법이 있습니다.

```typescript
const employee = new Employee(); // 생성자 매개변수도 사용할 수 있습니다.
employee.name = "John Doe";
employee.title = "senior engineer";
await getRepository(Employee).save(employee)
```

또는 액티브 레코드 패턴

```typescript
const employee = Employee.create({ name: "John Doe", title: "senior engineer" });
await employee.save();
```

데이터베이스에서 기존 엔티티를 로드하고 일부 속성을 바꾸려면 다음 방법을 사용할 수 있습니다.

```typescript
const employee = await Employee.preload({ id: 1, name: "John Doe" });
```
[액티브 레코드 대 데이터 매퍼](./active-record-data-mapper.md) 및 [Repository API](./repository-api.md)에 대해 자세히 알아보세요.

sequelize에서 속성에 액세스하려면 다음을 수행하십시오.

```typescript
console.log(employee.get("name"));
```

TypeORM에서는 다음을 수행합니다.

```typescript
console.log(employee.name);
```

sequelize에서 인덱스를 생성하려면 다음을 수행하십시오.

```typescript
sequelize.define("user", {}, {
  indexes: [
    {
      unique: true,
      fields: ["firstName", "lastName"]
    }
  ]
});
```

TypeORM에서는 다음을 수행합니다.

```typescript
@Entity()
@Index(["firstName", "lastName"], { unique: true })
export class User {
}
```
[색인](./indices.md)에 대해 자세히 알아보기
