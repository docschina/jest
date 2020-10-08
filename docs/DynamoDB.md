---
id: dynamodb
title: 结合 DynamoDB 使用
---

通过[全局配置/卸载](Configuration.md#globalsetup-string)和[异步测试环境](Configuration.md#testenvironment-string)的 API, Jest 就可以与 [DynamoDB](https://aws.amazon.com/dynamodb/) 无缝衔接了。

## 使用 jest-dynamodb 预设

[Jest DynamoDB](https://github.com/shelfio/jest-dynamodb) 提供了使用 DynamoDB 运行测试代码所需要的所有配置。

1. 首先，安装 `@shelf/jest-dynamodb`

```
yarn add @shelf/jest-dynamodb --dev
```

2. 在 Jest 的配置项中指定预设:

```json
{
  "preset": "@shelf/jest-dynamodb"
}
```

3. 创建 `jest-dynamodb-config.js` 并定义 DynamoDB 的数据表

请查阅[创建数据表 API](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#createTable-property)。

```js
module.exports = {
  tables: [
    {
      TableName: `files`,
      KeySchema: [{AttributeName: 'id', KeyType: 'HASH'}],
      AttributeDefinitions: [{AttributeName: 'id', AttributeType: 'S'}],
      ProvisionedThroughput: {ReadCapacityUnits: 1, WriteCapacityUnits: 1},
    },
    // etc
  ],
};
```

4. 配置 DynamoDB Client 端

```js
const {DocumentClient} = require('aws-sdk/clients/dynamodb');

const isTest = process.env.JEST_WORKER_ID;
const config = {
  convertEmptyValues: true,
  ...(isTest && {
    endpoint: 'localhost:8000',
    sslEnabled: false,
    region: 'local-env',
  }),
};

const ddb = new DocumentClient(config);
```

5. 编写测试代码

```js
it('should insert item into table', async () => {
  await ddb
    .put({TableName: 'files', Item: {id: '1', hello: 'world'}})
    .promise();

  const {Item} = await ddb.get({TableName: 'files', Key: {id: '1'}}).promise();

  expect(Item).toEqual({
    id: '1',
    hello: 'world',
  });
});
```

这里是不需要载入任何依赖的。

更多细节请查阅[相关文档](https://github.com/shelfio/jest-dynamodb)。
