# FAQ

- [데이터베이스 스키마를 어떻게 업데이트합니까?](#데이터베이스-스키마를-어떻게-업데이트합니까)
- [데이터베이스에서 컬럼 이름을 어떻게 변경합니까?](#데이터베이스에서-컬럼-이름을-어떻게-변경합니까)
- [예를 들어 `NOW()`와 같은 일부 함수에 기본값을 어떻게 설정할 수 있습니까?](#예를-들어-now와-같은-일부-함수에-기본값을-어떻게-설정할-수-있습니까)
- [유효성 검사는 어떻게하나요?](#유효성-검사는-어떻게하나요)
- [관계에서 "소유자 측"은 무엇을 의미하거나 `@JoinColumn` 및 `@JoinTable`을 사용해야하는 이유는 무엇입니까?](#관계에서-소유자-측은-무엇을-의미하거나-joincolumn-및-jointable을-사용해야하는-이유는-무엇입니까)
- [다대다(정션) 테이블에 추가 컬럼을 어떻게 추가합니까?](#다대다정션-테이블에-추가-컬럼을-어떻게-추가합니까)
- [종속성 주입 도구로 TypeORM을 사용하는 방법은 무엇입니까?](#종속성-주입-도구로-typeorm을-사용하는-방법은-무엇입니까)
- [outDir TypeScript 컴파일러 옵션을 처리하는 방법은 무엇입니까?](#outdir-typescript-컴파일러-옵션을-처리하는-방법은-무엇입니까)
- [ts-node에서 TypeORM을 사용하는 방법은 무엇입니까?](#ts-node에서-typeorm을-사용하는-방법은-무엇입니까)
- [백엔드에 Webpack을 사용하는 방법은 무엇입니까?](#백엔드에-webpack을-사용하는-방법은-무엇입니까)
  - [마이그레이션 파일 번들링](#마이그레이션-파일-번들링)


## 데이터베이스 스키마를 어떻게 업데이트합니까?

TypeORM의 주요 책임중 하나는 데이터베이스 테이블을 엔티티와 동기화 상태로 유지하는 것입니다. 이를 달성하는 데 도움이되는 두 가지 방법이 있습니다.

* 연결 옵션에서 `synchronize: true`를 사용하십시오.

    ```typescript
    import {createConnection} from "typeorm";

    createConnection({
        synchronize: true
    });
    ```

    이 옵션은 이 코드를 실행할 때마다 데이터베이스 테이블을 지정된 엔터티와 자동으로 동기화합니다. 이 옵션은 개발중에는 완벽하지만 프로덕션에서는 이 옵션을 활성화하지 않을 수 있습니다.

* 명령줄 도구를 사용하고 명령줄에서 수동으로 스키마 동기화를 실행합니다.

    ```
    typeorm schema:sync
    ```

    이 명령은 스키마 동기화를 실행합니다. 명령줄 도구가 작동하도록 하려면 ormconfig.json 파일을 만들어야 합니다.

스키마 동기화는 매우 빠릅니다. 성능 문제로 인해 개발 중에 동기화 비활성화 옵션을 고려중인 경우 먼저 속도를 확인하십시오.

## 데이터베이스에서 컬럼 이름을 어떻게 변경합니까?

기본적으로 컬럼 이름은 속성 이름에서 생성됩니다. `name` 컬럼 옵션을 지정하여 간단히 변경할 수 있습니다.

```typescript
@Column({ name: "is_active" })
isActive: boolean;
```

## 예를 들어 `NOW()`와 같은 일부 함수에 기본값을 어떻게 설정할 수 있습니까?

`default`컬럼 옵션은 함수를 지원합니다. 문자열을 반환하는 함수를 전달하는 경우 해당 문자열을 이스케이프하지 않고 기본값으로 사용합니다.

예를 들면:

```typescript
@Column({ default: () => "NOW()" })
date: Date;
```

## 유효성 검사는 어떻게하나요?

유효성 검사는 TypeORM이 수행하는 작업과 실제로 관련이 없는 별도의 프로세스이기 때문에 TypeORM의 일부가 아닙니다. 유효성 검사를 사용하려면 [class-validator](https://github.com/pleerock/class-validator)를 사용하세요. TypeORM과 완벽하게 작동합니다.

## 관계에서 "소유자 측"은 무엇을 의미하거나 `@JoinColumn` 및 `@JoinTable`을 사용해야하는 이유는 무엇입니까?

`일대일` 관계부터 시작하겠습니다. `User`와 `Photo`라는 두개의 엔티티가 있다고 가정해 보겠습니다.

```typescript
@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToOne()
    photo: Photo;

}
```

```typescript
@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

    @OneToOne()
    user: User;

}
```

이 예에는 잘못된 `@JoinColumn`이 없습니다. 왜? 실제 관계를 만들려면 데이터베이스에 컬럼을 만들어야합니다. `photo`에 `userId` 컬럼을, `user`에 `photoId` 컬럼을 생성해야합니다. 그러나 `userId` 또는 `photoId` 중 어떤 컬럼을 만들어야합니까? TypeORM은 당신을 위해 결정할 수 없습니다. 결정을 내리려면 한쪽에 `@JoinColumn`을 사용해야합니다. `photo`에 `@JoinColumn`을 입력하면 `photo` 테이블에 `userId`라는 컬럼이 생성됩니다. `User`에 `@JoinColumn`을 입력하면 `user` 테이블에 `photoId`라는 컬럼이 생성됩니다. `@JoinColumn`이 있는 쪽을 **"관계의 소유자 쪽"**이라고 합니다. `@JoinColumn`이 없는 관계의 다른 쪽을 **"관계의 역(비 소유자)쪽"**이라고 합니다.

`@ ManyToMany` 관계에서도 동일합니다. 관계의 소유자 측을 표시하려면 `@JoinTable`을 사용합니다.

`@ManyToOne` 또는 `@OneToMany` 관계에서는 `@JoinColumn`이 필요하지 않습니다. 두 데코레이터는 모두 다르며 `@ManyToOne` 데코레이터를 넣은 테이블에는 관계형 컬럼이 있습니다.

`@JoinColumn` 및 `@JoinTable` 데코레이터를 사용하여 조인 컬럼 이름 또는 접합 테이블 이름과 같은 추가 조인 컬럼/정션 테이블 설정을 지정할 수도 있습니다.

## 다대다(정션) 테이블에 추가 컬럼을 어떻게 추가합니까?

다대다 관계에 의해 생성된 테이블에 추가 컴럼을 추가할 수 없습니다. 별도의 엔터티를 만들고 대상 엔터티와 두개의 다대일 관계를 사용하여 바인딩하고(효과는 다대다 테이블을 만드는 것과 동일함) 거기에 추가 컬럼을 추가해야 합니다. 이에 대한 자세한 내용은 [다대다 관계](./many-to-many-relations.md#many-to-many-relations-with-custom-properties)에서 확인할 수 있습니다.

## 종속성 주입 도구로 TypeORM을 사용하는 방법은 무엇입니까?

TypeORM에서는 서비스 컨테이너를 사용할 수 있습니다. 서비스 컨테이너를 사용하면 구독자 또는 사용자 지정 이름 지정 전략과 같은 일부 위치에 사용자 지정 서비스를 삽입할 수 있습니다. 예를 들어 서비스 컨테이너를 사용하여 모든 위치에서 ConnectionManager에 액세스할 수 있습니다.

다음은 TypeORM으로 typedi 서비스 컨테이너를 설정하는 방법에 대한 예입니다. 참고: TypeORM으로 모든 서비스 컨테이너를 설정할 수 있습니다.

```typescript
import {useContainer, createConnection} from "typeorm";
import {Container} from "typedi";

// its important to setup container before you start to work with TypeORM
useContainer(Container);
createConnection({/* ... */});
```

## outDir TypeScript 컴파일러 옵션을 처리하는 방법은 무엇입니까?

`outDir` 컴파일러 옵션을 사용하는 경우 앱에서 사용중인 자산과 리소스를 출력 디렉터리에 복사하는 것을 잊지 마십시오. 그렇지 않으면 해당 자산에 대한 올바른 경로를 설정해야 합니다.

알아야 할 한가지 중요한 점은 엔티티를 제거하거나 이동할 때 이전 엔티티가 출력 디렉토리 내부에 그대로 남아 있다는 것입니다. 예를 들어 `Post` 항목을 만들고 이름을 `Blog`로 변경하면 더 이상 프로젝트에 `Post.ts`가 없습니다. 그러나 `Post.js`는 출력 디렉토리 안에 남아 있습니다. 이제 TypeORM이 출력 디렉토리에서 항목을 읽을 때 `Post`와 `Blog`라는 두 항목이 표시됩니다. 이것은 버그의 원인일 수 있습니다. 그렇기 때문에 `outDir`이 활성화된 상태에서 엔티티를 제거하고 이동할 때 출력 디렉토리를 제거하고 프로젝트를 다시 컴파일하는 것이 좋습니다.

## ts-node에서 TypeORM을 사용하는 방법은 무엇입니까?

[ts-node](https://github.com/TypeStrong/ts-node)를 사용하여 매번 파일 컴파일을 방지할 수 있습니다. ts-node를 사용하는 경우 연결 옵션 내에서 `ts` 항목을 지정할 수 있습니다.

```
{
    entities: ["src/entity/*.ts"],
    subscribers: ["src/subscriber/*.ts"]
}
```

또한 typescript 파일이있는 폴더에 js 파일을 컴파일하는 경우 `outDir` 컴파일러 옵션을 사용하여 [이 문제](https://github.com/TypeStrong/ts-node/issues/432)를 방지하십시오.

또한 ts-node CLI를 사용하려는 경우 다음 방법으로 TypeORM을 실행할 수 있습니다.

```
ts-node ./node_modules/.bin/typeorm schema:sync
```

## 백엔드에 Webpack을 사용하는 방법은 무엇입니까?

Webpack은 require 문이 누락된 것으로 간주하여 경고를 생성합니다. TypeORM에서 지원하는 모든 드라이버에 대한 문이 필요합니다. 사용하지 않는 드라이버에 대한 이러한 경고를 표시하지 않으려면 웹팩 구성 파일을 편집해야 합니다.

```js
const FilterWarningsPlugin = require('webpack-filter-warnings-plugin');

module.exports = {
    ...
    plugins: [
        //원하지 않는 드라이버는 무시하십시오. 이것은 모든 드라이버의 전체 목록입니다. 사용하려는 드라이버에 대한 억제를 제거하십시오.
        new FilterWarningsPlugin({
            exclude: [/mongodb/, /mssql/, /mysql/, /mysql2/, /oracledb/, /pg/, /pg-native/, /pg-query-stream/, /react-native-sqlite-storage/, /redis/, /sqlite3/, /sql.js/, /typeorm-aurora-data-api-driver/]
        })
    ]
};
```

### 마이그레이션 파일 번들링

기본적으로 Webpack은 모든 것을 하나의 파일로 묶으려고 합니다. 프로젝트에 번들 코드가 프로덕션에 배포된 후 실행될 마이그레이션 파일이 있는 경우 문제가 될 수 있습니다. 모든 마이그레이션이 TypeORM에서 인식되고 실행될 수 있도록 하려면 마이그레이션 파일에 대해서만 `entry` 구성에 "Object Syntax"를 사용해야 할 수 있습니다.

```js
const glob = require('glob');
const path = require('path');

module.exports = {
  // ... 여기에 웹팩 구성 ...
  // `entry` 옵션에 대한 `{ [name]: sourceFileName }` 맵을 동적으로 생성합니다.
  // `src/db/migrations`를 마이그레이션 폴더의 상대 경로로 변경합니다.
  entry: glob.sync(path.resolve('src/db/migrations/*.ts')).reduce((entries, filename) => {
    const migrationName = path.basename(filename, '.ts');
    return Object.assign({}, entries, {
      [migrationName]: filename,
    });
  }, {}),
  resolve: {
    // 모든 마이그레이션 파일이 TypeScript로 작성되었다고 가정합니다.
    extensions: ['.ts']
   },
  output: {
    // 트랜스파일된 마이그레이션 파일을 저장할 위치로 `path`를 변경합니다.
    path: __dirname + '/dist/db/migrations',
    // 이것은 중요합니다. 마이그레이션 파일에 UMD(Universal Module Definition)가 필요합니다.
    libraryTarget: 'umd',
    filename: '[name].js',
  },
};
```

또한 Webpack 4 이후 `mode: 'production'`을 사용할 때 파일 크기를 최소화하기 위해 코드를 변경하는 등 기본적으로 파일이 최적화됩니다. 이것은 TypeORM이 이미 실행된 것을 결정하기 위해 이름에 의존하기 때문에 마이그레이션을 중단합니다. 다음을 추가하여 최소화를 완전히 비활성화 할 수 있습니다.

```js
module.exports = {
  // ... other Webpack configurations here
  optimization: {
    minimize: false,
  },
};
```

또는 `UglifyJsPlugin`을 사용하는 경우 다음과 같이 클래스 또는 함수 이름을 변경하지 않도록 지시할 수 있습니다.

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
  // ... 여기에 다른 Webpack 구성
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        uglifyOptions: {
          keep_classnames: true,
          keep_fnames: true
        }
      })
    ],
  },
};
```

마지막으로 `ormconfig` 파일에 트랜스파일된 마이그레이션 파일이 포함되어 있는지 확인하십시오.

```js
// TypeORM 구성
module.exports = {
  // ...
  migrations: [
    // 프로덕션에서 트랜스파일된 마이그레이션 파일의 상대 경로입니다.
    'db/migrations/**/*.js',
    // 개발 모드에서 사용되는 소스 마이그레이션 파일
    'src/db/migrations/**/*.ts',
  ],
};
```
