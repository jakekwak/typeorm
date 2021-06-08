# Supported platforms

- [노드JS](#노드js)
- [브라우저](#브라우저)
- [Cordova / PhoneGap / Ionic 앱](#cordova--phonegap--ionic-앱)
- [React Native](#react-native)
- [Expo](#expo)
- [NativeScript](#nativescript)

## 노드JS

TypeORM은 Node.js 버전 4 이상에서 테스트되었습니다.

## 브라우저

브라우저에서 [sql.js](https://sql.js.org)를 사용할 수 있습니다.

**Webpack 구성**

`browser` 폴더에서 패키지에는 ES2015 모듈로 컴파일된 버전도 포함되어 있습니다. 다른 로더를 사용하고 싶다면 이것이 시작점입니다. TypeORM 0.1.7 이전에는 패키지가 webpack과 같은 로더가 자동으로 `browser` 폴더를 사용하는 방식으로 설정되었습니다. 0.1.7에서는 Node.js 프로젝트에서 Webpack 사용을 지원하기 위해 삭제되었습니다. 즉, 브라우저 프로젝트에 올바른 버전이 로드되었는지 확인하기 위해 `NormalModuleReplacementPlugin`을 사용해야합니다. 이 플러그인에 대한 웹팩 구성 파일의 구성은 다음과 같습니다.

```js
plugins: [
    ..., // any existing plugins that you already have
    new webpack.NormalModuleReplacementPlugin(/typeorm$/, function (result) {
        result.request = result.request.replace(/typeorm/, "typeorm/browser");
    }),
    new webpack.ProvidePlugin({
      'window.SQL': 'sql.js/dist/sql-wasm.js'
    })
]
```

공개 경로에 [sql-wasm.wasm 파일](https://github.com/sql-js/sql.js/blob/master/README.md#downloadingusing)이 있는지 확인합니다.

**구성 예**

```typescript
createConnection({
    type: "sqljs",
    entities: [
        Photo
    ],
    synchronize: true
});
```

**reflect-metadata를 포함하는 것을 잊지 마세요**

기본 HTML 페이지에 reflect-metadata를 포함해야 합니다.

```html
<script src="./node_modules/reflect-metadata/Reflect.js"></script>
```

## Cordova / PhoneGap / Ionic 앱

TypeORM은 [cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage) 플러그인을 사용하여 Cordova, PhoneGap, Ionic 앱에서 실행할 수 있습니다. 브라우저 패키지에서와 같이 모듈 로더 중에서 선택할 수 있는 옵션이 있습니다. Cordova에서 TypeORM을 사용하는 방법의 예는 [typeorm/cordova-example](https://github.com/typeorm/cordova-example)을 참조하고 Ionic은 [typeorm/ionic-example](https://github.com/typeorm/ionic-example)을 참조하십시오. **중요**: Ionic과 함께 사용하려면 사용자 지정 웹팩 구성 파일이 필요합니다! 필요한 변경 사항을 보려면 예제를 확인하십시오.

## React Native

TypeORM은 [react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) 플러그인을 사용하여 React Native 앱에서 실행할 수 있습니다. 예를 보려면 [typeorm/react-native-example](https://github.com/typeorm/react-native-example)을 참조하세요.

## Expo

TypeORM은 [Expo SQLite API](https://docs.expo.io/versions/latest/sdk/sqlite/)를 사용하여 Expo 앱에서 실행할 수 있습니다. Expo에서 TypeORM을 사용하는 방법의 예는 [typeorm/expo-example](https://github.com/typeorm/expo-example)을 참조하세요.

## NativeScript

1. `tns install webpack` (아래에서 webpack이 필요한 이유를 읽어보십시오)
2. `tns plugin add nativescript-sqlite`
3. 앱의 엔트리포인트에서 데이터베이스 연결 만들기
    ```typescript
    import driver from 'nativescript-sqlite'

    const connection = await createConnection({
        database: 'test.db',
        type: 'nativescript',
        driver,
        entities: [
            Todo //... 당신이 가진 모든 엔티티
        ],
        logging: true
    })
    ```

참고: 이것은 NativeScript 4.x 이상에서만 작동합니다.

_NativeScript와 함께 사용하는 경우 **webpack사용은 필수입니다**. `typeorm/browser` 패키지는 `import/export`가 있는 원시 ES7 코드로 그대로 실행되지 **않습니다**. 번들로 제공되어야 합니다. `tns run --bundle` 메소드를 사용하세요_

[여기](https://github.com/championswimmer/nativescript-vue-typeorm-sample)에서 예를 확인하세요!
