# 자바스크립트화 함께 사용하기

TypeORM은 TypeScript뿐만 아니라 JavaScript에서도 사용할 수 있습니다.
타입을 생략하고 플랫폼이 ES6 클래스를 지원하지 않는 경우 필요한 모든 메타 데이터로 객체를 정의해야 한다는 점을 제외하면 모든 것이 동일합니다.

##### app.js

```typescript
var typeorm = require("typeorm");

typeorm.createConnection({
    type: "postgres",
    host: "localhost",
    port: 5432,
    username: "test",
    password: "admin",
    database: "test",
    synchronize: true,
    entities: [
        require("./entity/Post"),
        require("./entity/Category")
    ]
}).then(function (connection) {

    var category1 = {
        name: "TypeScript"
    };
    var category2 = {
        name: "Programming"
    };

    var post = {
        title: "Control flow based type analysis",
        text: "TypeScript 2.0 implements a control flow-based type analysis for local variables and parameters.",
        categories: [
            category1, category2
        ]
    };

    var postRepository = connection.getRepository("Post");
    postRepository.save(post)
        .then(function(savedPost) {
            console.log("Post has been saved: ", savedPost);
            console.log("Now lets load all posts: ");

            return postRepository.find();
        })
        .then(function(allPosts) {
            console.log("All posts: ", allPosts);
        });


}).catch(function(error) {
    console.log("Error: ", error);
});
```

##### entity/Category.js

```typescript
var EntitySchema = require("typeorm").EntitySchema;

module.exports = new EntitySchema({
    name: "Category", // 테이블 이름` category`를 기본 동작으로 사용합니다.
    tableName: "categories", // 선택 사항: 테이블 이름의 기본 동작을 재정의하려면 `tableName` 속성을 제공합니다.
    columns: {
        id: {
            primary: true,
            type: "int",
            generated: true
        },
        name: {
            type: "varchar"
        }
    }
});
```

##### entity/Post.js

```typescript
var EntitySchema = require("typeorm").EntitySchema;

module.exports = new EntitySchema({
    name: "Post", // 테이블 이름 'post'를 기본 동작으로 사용합니다.
    tableName: "posts", // 선택 사항: '테이블 이름'속성을 제공하여 테이블 이름의 기본 동작을 재정의합니다.
    columns: {
        id: {
            primary: true,
            type: "int",
            generated: true
        },
        title: {
            type: "varchar"
        },
        text: {
            type: "text"
        }
    },
    relations: {
        categories: {
            target: "Category",
            type: "many-to-many",
            joinTable: true,
            cascade: true
        }
    }
});
```

자세한 내용은 이 예제 [typeorm/javascript-example](https://github.com/typeorm/javascript-example)를 확인하세요.
