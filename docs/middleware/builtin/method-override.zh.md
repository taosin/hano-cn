# Method Override 中间件

此中间件根据 form、header 或 query 的值执行指定方法的 handler（与请求的实际方法不同），并返回其响应。

## 导入

```ts
import { Hono } from 'hono'
import { methodOverride } from 'hono/method-override'
```

## 用法

```ts
const app = new Hono()

// 如果未指定选项，则使用 form 中的 `_method` 的值，
// 例如 DELETE，作为方法。
app.use('/posts', methodOverride({ app }))

app.delete('/posts', (c) => {
  // ....
})
```

## 例如

由于 HTML 表单无法发送 DELETE 方法，你可以将值 `DELETE` 放入名为 `_method` 的属性中并发送。然后 `app.delete()` 的 handler 将执行。

HTML 表单：

```html
<form action="/posts" method="POST">
  <input type="hidden" name="_method" value="DELETE" />
  <input type="text" name="id" />
</form>
```

应用程序：

```ts
import { methodOverride } from 'hono/method-override'

const app = new Hono()
app.use('/posts', methodOverride({ app }))

app.delete('/posts', () => {
  // ...
})
```

你可以更改默认值或使用 header 值和 query 值：

```ts
app.use('/posts', methodOverride({ app, form: '_custom_name' }))
app.use(
  '/posts',
  methodOverride({ app, header: 'X-METHOD-OVERRIDE' })
)
app.use('/posts', methodOverride({ app, query: '_method' }))
```

## 选项

### <Badge type="danger" text="required" /> app: `Hono`

你的应用程序中使用的 `Hono` 实例。

### <Badge type="info" text="optional" /> form: `string`

包含方法名的值的 Form 键。
默认值为 `_method`。

### <Badge type="info" text="optional" /> header: `boolean`

包含方法名的值的 Header 名称。

### <Badge type="info" text="optional" /> query: `boolean`

包含方法名的值的 Query 参数键。
