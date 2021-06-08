# 네이밍 전략

* 사용자 정의 테이블 이름 지정
* 사용자 지정 컬럼 이름 지정
* 사용자 지정 외부 컬럼 이름 지정
* 사용자 지정 다대다 정션 테이블 이름 지정
* 나만의 `NamingStrategy` 만들기

## 나만의 'NamingStrategy' 만들기

`ormconfig` 파일에서 연결 옵션을 정의한 경우
그런 다음 간단히 사용하고 다음과 같이 재정의할 수 있습니다.

```typescript
import {createConnection, getConnectionOptions} from "typeorm";
import {MyNamingStrategy} from "./logger/MyNamingStrategy";

// getConnectionOptions는 ormconfig 파일에서 옵션을 읽고
// connectionOptions 객체에 반환한 다음 추가 속성을 추가할 수 있습니다.
getConnectionOptions().then(connectionOptions => {
    return createConnection(Object.assign(connectionOptions, {
        namingStrategy: new MyNamingStrategy()
    }))
});
```

네이밍 전략은 변경 될 수 있습니다.
API가 안정화되면 자세한 문서를 기대하십시오.