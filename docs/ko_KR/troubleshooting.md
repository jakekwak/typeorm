# 문제 해결
- [Glob 패턴](#glob-패턴)

## Glob 패턴

Glob 패턴은 TypeOrm에서 엔티티, 마이그레이션, 구독자 및 기타 정보의 위치를 지정하는 데 사용됩니다. 패턴의 오류는 일반적인 `RepositoryNotFoundError` 및 익숙한 오류로 이어질 수 있습니다. glob 패턴을 사용하여 TypeOrm에 의해 파일이 로드되었는지 확인하려면 문서의 [Logging](./logging.md) 섹션에 설명된대로 로깅 수준을 `info`로 설정하기만하면 됩니다. 이렇게하면 콘솔에 다음과 같은 로그가있을 수 있습니다.

```bash
# 오류가 발생한 경우
 INFO: No classes were found using the provided glob pattern:  "dist/**/*.entity{.ts}"
```
```bash
# 파일이 발견되면
INFO: All classes found using provided glob pattern "dist/**/*.entity{.js,.ts}" : "dist/app/user/user.entity.js | dist/app/common/common.entity.js"
```