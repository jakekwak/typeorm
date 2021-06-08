# 로깅

- [로깅 활성화](#로깅-활성화)
- [로깅 옵션](#로깅-옵션)
- [장기 실행 쿼리 기록](#장기-실행-쿼리-기록)
- [기본 로거 변경](#기본-로거-변경)
- [사용자 정의 로거 사용](#사용자-정의-로거-사용)

## 로깅 활성화

연결 옵션에서 `logging: true`를 설정하기 만하면 모든 쿼리 및 오류의 로깅을 활성화할 수 있습니다.

```typescript
{
    name: "mysql",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    ...
    logging: true
}
```

## 로깅 옵션

연결 옵션에서 다양한 유형의 로그인을 활성화 할 수 있습니다.

```typescript
{
    host: "localhost",
    ...
    logging: ["query", "error"]
}
```

실패한 쿼리의 로깅만 활성화하려면`error` 만 추가하십시오.

```typescript
{
    host: "localhost",
    ...
    logging: ["error"]
}
```

사용할 수 있는 다른 옵션이 있습니다.

* `query` - 모든 쿼리를 기록합니다.
* `error` - 실패한 모든 쿼리 및 오류를 기록합니다.
* `schema` - 스키마 빌드 프로세스를 기록합니다.
* `warn` - 내부 orm 경고를 기록합니다.
* `info` - 내부 orm 정보 메시지를 기록합니다.
* `log` - 내부 orm 로그 메시지를 기록합니다.

필요한만큼 옵션을 지정할 수 있습니다. 모든 로깅을 활성화하려면 간단히 `logging: "all"`을 지정하면됩니다.

```typescript
{
    host: "localhost",
    ...
    logging: "all"
}
```

## 장기 실행 쿼리 기록

성능 문제가 있는 경우 연결 옵션에서 `maxQueryExecutionTime`을 설정하여 실행하는데 너무 많은 시간이 걸리는 쿼리를 기록 할 수 있습니다.

```typescript
{
    host: "localhost",
    ...
    maxQueryExecutionTime: 1000
}
```

이 코드는 `1 초`이상 실행되는 모든 쿼리를 기록합니다.

## 기본 로거 변경

TypeORM은 4가지 유형의 로거와 함께 제공됩니다.

* `advanced-console` - 이것은 색상 및 SQL 구문 강조([chalk](https://github.com/chalk/chalk) 사용)를 사용하여 모든 메시지를 콘솔에 기록하는 기본 로거입니다.
* `simple-console` - 이것은 고급 로거와 똑같은 간단한 콘솔 로거이지만 색상 강조 표시를 사용하지 않습니다. 이 로거는 문제가 있거나 색이 지정된 로그가 마음에 들지 않을 때 사용할 수 있습니다.
* `file` - 이 로거는 모든 로그를 프로젝트의 루트 폴더(`package.json` 및 `ormconfig.json` 근처)에 있는 `ormlogs.log`에 기록합니다.
* `debug` - 이 로거는 [debug package](https://github.com/visionmedia/debug)를 사용하여 로깅을 설정하고 env 변수 `DEBUG=typeorm:*`을 설정합니다 (로거 로깅 옵션은 이 로거에 영향을 주지 않음).

연결 옵션에서 이들 중 하나를 활성화할 수 있습니다.

```typescript
{
    host: "localhost",
    ...
    logging: true,
    logger: "file"
}
```

## 사용자 정의 로거 사용

`Logger` 인터페이스를 구현하여 고유한 로거 클래스를 만들 수 있습니다.

```typescript
import {Logger} from "typeorm";

export class MyCustomLogger implements Logger {

    // 로거 클래스의 모든 메소드 구현

}
```

연결 옵션에서 지정하십시오.

```typescript
import {createConnection} from "typeorm";
import {MyCustomLogger} from "./logger/MyCustomLogger";

createConnection({
    name: "mysql",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    logger: new MyCustomLogger()
});
```

`ormconfig` 파일에서 연결 옵션을 정의한 경우 이를 사용하고 다음과 같은 방법으로 재정의할 수 있습니다.

```typescript
import {createConnection, getConnectionOptions} from "typeorm";
import {MyCustomLogger} from "./logger/MyCustomLogger";

// getConnectionOptions는 ormconfig 파일에서 옵션을 읽고
// connectionOptions 객체에 반환한 다음 추가 속성을 추가할 수 있습니다.
getConnectionOptions().then(connectionOptions => {
    return createConnection(Object.assign(connectionOptions, {
        logger: new MyCustomLogger()
    }))
});
```

로거 메서드는 사용 가능한 경우 `QueryRunner`를 허용할 수 있습니다. 추가 데이터를 기록하려는 경우 유용합니다. 또한 쿼리 실행기를 통해 지속 / 제거중에 전달된 추가 데이터에 액세스할 수 있습니다. 예를 들면:

```typescript
// 사용자가 엔티티 저장 중에 요청을 보냅니다.
postRepository.save(post, { data: { request: request } });

// 로거에서 다음과 같이 액세스 할 수 있습니다.
logQuery(query: string, parameters?: any[], queryRunner?: QueryRunner) {
    const requestUrl = queryRunner && queryRunner.data["request"] ? "(" + queryRunner.data["request"].url + ") " : "";
    console.log(requestUrl + "executing query: " + query);
}
```
