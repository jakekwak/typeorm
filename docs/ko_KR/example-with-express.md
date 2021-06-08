# Express와 함께 TypeORM을 사용하는 예

- [초기 설정](#초기-설정)
- [애플리케이션에 Express 추가](#애플리케이션에-express-추가)
- [Adding TypeORM to the application](#adding-typeorm-to-the-application)

## 초기 설정

데이터베이스에 사용자를 저장하고 웹 API 내에서 ID별로 단일 사용자 목록을 생성, 업데이트, 제거 및 가져올 수 있는 `user`라는 간단한 애플리케이션을 만들어 보겠습니다.

먼저 `user`라는 디렉토리를 만듭니다.

```
mkdir user
```

그런 다음 디렉터리로 전환하고 새 프로젝트를 만듭니다.

```
cd user
npm init
```

모든 필수 애플리케이션 정보를 입력하여 초기화 프로세스를 완료하십시오.

이제 TypeScript 컴파일러를 설치하고 설정해야합니다. 먼저 설치하겠습니다.

```
npm i typescript --save-dev
```

그런 다음 애플리케이션을 컴파일하고 실행하는 데 필요한 구성이 포함된 `tsconfig.json` 파일을 만들어 보겠습니다. 좋아하는 편집기를 사용하여 만들고 다음 구성을 넣으십시오.

```json
{
  "compilerOptions": {
    "lib": ["es5", "es6", "dom"],
    "target": "es5",
    "module": "commonjs",
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true
  }
}
```

이제 `src` 디렉토리내에 메인 애플리케이션 엔드포인트인 `app.ts`를 생성해 보겠습니다.

```
mkdir src
cd src
touch app.ts
```

내부에 간단한 `console.log`를 추가해 보겠습니다.

```typescript
console.log("Application is up and running");
```

이제 애플리케이션을 실행할 차례입니다. 실행하려면 먼저 typescript 프로젝트를 컴파일해야합니다.

```
tsc
```

컴파일하면 `src/app.js` 파일이 생성됩니다. 다음을 사용하여 실행할 수 있습니다.

```
node src/app.js
```

애플리케이션을 실행한 직후 콘솔에 "Application is up and running"라는 메시지가 표시되어야 합니다.

변경할 때마다 파일을 컴파일해야합니다. 또는 감시자를 설정하거나 [ts-node](https://github.com/TypeStrong/ts-node)를 설치하여 매번 수동 컴파일을 피할 수 있습니다.

## 애플리케이션에 Express 추가

애플리케이션에 Express를 추가해 보겠습니다. 먼저 필요한 패키지를 설치하겠습니다.

```
npm i express  @types/express --save
```

* `express` 는 익스프레스 엔진 자체입니다. 웹 API를 만들 수 있습니다.
* `@types/express`는 express를 사용할 때 타입 정보를 갖기 위해 사용됩니다.

`src/app.ts` 파일을 편집하고 익스프레스 관련 로직을 추가해 보겠습니다.

```typescript
import * as express from "express";
import {Request, Response} from "express";

// Express 앱 생성 및 설정
const app = express();
app.use(express.json());

// 라우트 등록

app.get("/users", function(req: Request, res: Response) {
    // 여기에 모든 사용자를 반환하는 논리가 있습니다.
});

app.get("/users/:id", function(req: Request, res: Response) {
    // 여기에 id로 사용자를 반환하는 논리가 있습니다.
});

app.post("/users", function(req: Request, res: Response) {
    // 여기에 사용자를 저장하는 로직이 있습니다.
});

app.put("/users/:id", function(req: Request, res: Response) {
    // 여기에 주어진 사용자 ID로 사용자를 업데이트하는 로직이 있습니다.
});

app.delete("/users/:id", function(req: Request, res: Response) {
    // 여기에 주어진 사용자 ID로 사용자를 삭제하는 로직이 있습니다.
});

// 익스프레스 서버 시작
app.listen(3000);
```

이제 프로젝트를 컴파일하고 실행할 수 있습니다.
작업 라우트를 사용하여 지금 Express 서버를 실행해야합니다.
그러나 이러한 라우트는 아직 콘텐츠를 반환하지 않습니다.

## Adding TypeORM to the application

마지막으로 TypeORM을 애플리케이션에 추가해 보겠습니다.
이 예에서는 `mysql` 드라이버를 사용합니다.
다른 드라이버의 설치 프로세스도 비슷합니다.

먼저 필요한 패키지를 설치하겠습니다.

```
npm i typeorm mysql reflect-metadata --save
```

* `typeorm`은 typeorm 패키지 자체입니다.
* `mysql`은 기본 데이터베이스 드라이버입니다. 다른 데이터베이스 시스템을 사용하는 경우 적절한 패키지를 설치해야합니다.
* 데코레이터가 제대로 작동하려면 `reflect-metadata`가 필요합니다.

이제 우리가 사용할 데이터베이스 연결구성으로 `ormconfig.json`을 생성해 보겠습니다.

```json
  {
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "username": "test",
    "password": "test",
    "database": "test",
    "entities": ["src/entity/*.js"],
    "logging": true,
    "synchronize": true
  }
```

필요에 따라 각 옵션을 구성하십시오.
[연결 옵션](./connection-options.md)에 대해 자세히 알아보세요.

`src/entity` 내에 `User` 엔티티를 생성해 보겠습니다.

```typescript
import {Entity, Column, PrimaryGeneratedColumn} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

}
```

`src/app.ts`를 변경해 보겠습니다.

```typescript
import * as express from "express";
import {Request, Response} from "express";
import {createConnection} from "typeorm";
import {User} from "./entity/User";

// typeorm 연결 생성
createConnection().then(connection => {
    const userRepository = connection.getRepository(User);

    // Express 앱 생성 및 설정
    const app = express();
    app.use(express.json());

    // 라우트 등록

    app.get("/users", async function(req: Request, res: Response) {
        const users = await userRepository.find();
        res.json(users);
    });

    app.get("/users/:id", async function(req: Request, res: Response) {
        const results = await userRepository.findOne(req.params.id);
        return res.send(results);
    });

    app.post("/users", async function(req: Request, res: Response) {
        const user = await userRepository.create(req.body);
        const results = await userRepository.save(user);
        return res.send(results);
    });

    app.put("/users/:id", async function(req: Request, res: Response) {
        const user = await userRepository.findOne(req.params.id);
        userRepository.merge(user, req.body);
        const results = await userRepository.save(user);
        return res.send(results);
    });

    app.delete("/users/:id", async function(req: Request, res: Response) {
        const results = await userRepository.delete(req.params.id);
        return res.send(results);
    });

    // 익스프레스 서버 시작
    app.listen(3000);
});
```

액션 콜백을 별도의 파일로 추출하고 `connection` 인스턴스가 필요한 경우, 간단히 `getConnection`을 사용할 수 있습니다.

```typescript
import {getConnection} from "typeorm";
import {User} from "./entity/User";

export function UsersListAction(req: Request, res: Response) {
    return getConnection().getRepository(User).find();
}
```

이 예에서는 `getConnection`이 필요하지 않습니다. `getRepository` 함수를 직접 사용할 수 있습니다.

```typescript
import {getRepository} from "typeorm";
import {User} from "./entity/User";

export function UsersListAction(req: Request, res: Response) {
    return getRepository(User).find();
}
```
