# Pretty JSON 中间件

Pretty JSON 中间件为 JSON 响应体启用"_JSON 美化打印_"。
将 `?pretty` 添加到 URL 查询参数，JSON 字符串将被美化。

```js
// GET /
{"project":{"name":"Hono","repository":"https://github.com/honojs/hono"}}
```

将变为：

```js
// GET /?pretty
{
  "project": {
    "name": "Hono",
    "repository": "https://github.com/honojs/hono"
  }
}
```

## 导入

```ts
import { Hono } from 'hono'
import { prettyJSON } from 'hono/pretty-json'
```

## 用法

```ts
const app = new Hono()

app.use(prettyJSON()) // 带有选项：prettyJSON({ space: 4 })
app.get('/', (c) => {
  return c.json({ message: 'Hono!' })
})
```

## 选项

### <Badge type="info" text="optional" /> space: `number`

缩进的空格数。默认值为 `2`。

### <Badge type="info" text="optional" /> query: `string`

应用的查询字符串名称。默认值为 `pretty`。

### <Badge type="info" text="optional" /> force: `boolean`

设置为 `true` 时，JSON 响应将始终美化，无论查询参数如何。默认值为 `false`。
