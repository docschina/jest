---
id: mongodb
title: 结合 MongoDB 使用
---

通过[全局配置/卸载](Configuration.md#globalsetup-string)和[异步测试环境](Configuration.md#testenvironment-string)的 API, Jest 就可以与 [MongoDB](https://www.mongodb.com/) 无缝衔接了。

## 使用 jest-mongodb 预设

[Jest MongoDB](https://github.com/shelfio/jest-mongodb) 提供了使用 MongoDB 运行测试代码所需要的所有配置。

1. 首先，安装 `@shelf/jest-mongodb`

```
yarn add @shelf/jest-mongodb --dev
```

2. 在 Jest 的配置项中指定预设:

```json
{
  "preset": "@shelf/jest-mongodb"
}
```

3. 编写测试代码

```js
const {MongoClient} = require('mongodb');

describe('insert', () => {
  let connection;
  let db;

  beforeAll(async () => {
    connection = await MongoClient.connect(global.__MONGO_URI__, {
      useNewUrlParser: true,
    });
    db = await connection.db(global.__MONGO_DB_NAME__);
  });

  afterAll(async () => {
    await connection.close();
    await db.close();
  });

  it('should insert a doc into collection', async () => {
    const users = db.collection('users');

    const mockUser = {_id: 'some-user-id', name: 'John'};
    await users.insertOne(mockUser);

    const insertedUser = await users.findOne({_id: 'some-user-id'});
    expect(insertedUser).toEqual(mockUser);
  });
});
```

这里是不需要载入任何依赖的。

更多细节请查阅[相关文档](https://github.com/shelfio/jest-mongodb) (包含 MongoDB 版本配置等)。
