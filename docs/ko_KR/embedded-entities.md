# 임베디드 엔티티

`임베디드 컬럼`을 사용하여 앱의 중복을 줄이는 놀라운 방법이 있습니다(상속보다 구성(Composition) 사용). 임베디드 컬럼은 자체 컬럼이 있는 클래스를 허용하고 해당 컬럼을 현재 엔터티의 데이터베이스 테이블에 병합하는 컬럼입니다.

예:

`User`, `Employee` 및 `Student` 엔티티가 있다고 가정해 보겠습니다. 이러한 모든 항목에는 공통점이 거의 없습니다 - `firstname` 및 `lastname` 속성

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: string;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    isActive: boolean;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class Employee {

    @PrimaryGeneratedColumn()
    id: string;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    salary: string;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class Student {

    @PrimaryGeneratedColumn()
    id: string;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    faculty: string;

}
```

우리가 할 수 있는 것은 해당 컬럼으로 새 클래스를 생성하여 `firstName` 및 `lastName` 중복을 줄이는 것입니다.

```typescript
import {Column} from "typeorm";

export class Name {

    @Column()
    first: string;

    @Column()
    last: string;

}
```

그런 다음 엔티티에서 해당 컬럼을 "연결"할 수 있습니다.

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
import {Name} from "./Name";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: string;

    @Column(() => Name)
    name: Name;

    @Column()
    isActive: boolean;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
import {Name} from "./Name";

@Entity()
export class Employee {

    @PrimaryGeneratedColumn()
    id: string;

    @Column(() => Name)
    name: Name;

    @Column()
    salary: number;

}
```

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
import {Name} from "./Name";

@Entity()
export class Student {

    @PrimaryGeneratedColumn()
    id: string;

    @Column(() => Name)
    name: Name;

    @Column()
    faculty: string;

}
```

`Name` 항목에 정의된 모든 컬럼은 `user`, `employee` 및 `student`로 병합됩니다.

```shell
+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| nameFirst   | varchar(255) |                            |
| nameLast    | varchar(255) |                            |
| isActive    | boolean      |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                        employee                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| nameFirst   | varchar(255) |                            |
| nameLast    | varchar(255) |                            |
| salary      | int(11)      |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                         student                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| nameFirst   | varchar(255) |                            |
| nameLast    | varchar(255) |                            |
| faculty     | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

이렇게 하면 엔티티 클래스의 코드 중복이 줄어 듭니다. 임베디드 클래스에서 필요한 만큼의 컬럼(또는 관계)을 사용할 수 있습니다. 임베디드 클래스내에 중첩된 임베디드 컬럼을 가질 수도 있습니다.
