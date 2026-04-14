# Request ID 中间件

Request ID 中间件为每个请求生成唯一的 ID，你可以在 handlers 中使用它。

::: info
**Node.js**：此中间件使用 `crypto.randomUUID()` 生成 ID。全局 `crypto` 在 Node.js 20 或更高版本中引入。因此，在更早的版本中可能会出现错误。在这种情况下，请指定 `generator`。但是，如果你使用 [Node.js 适配器](https://github.com/honojs/node-server)，它会自动全局设置 `crypto`，因此不需要这样做。
:::

## 导入

```ts
import { Hono } from 'hono'
import { requestId } from 'hono/request-id'
```

## 用法

你可以通过应用了 Request ID 中间件的 handlers 和中间件中的 `requestId` 变量访问 Request ID。

```ts
const app = new Hono()

app.use('*', requestId())

app.get('/', (c) => {
  return c.text(`Your request id is ${c.get('requestId')}`)
})
```

如果你想明确指定类型，请导入 `RequestIdVariables` 并在 `new Hono()` 的泛型中传递它。

```ts
import type { RequestIdVariables } from 'hono/request-id'

const app = new Hono<{
  Variables: RequestIdVariables
}>()
```

### 设置 Request ID

如果你在 header（默认：`X-Request-Id`）中设置自定义 request ID，中间件将使用该值而不是生成新值：

```ts
const app = new Hono()

app.use('*', requestId())

app.get('/', (c) => {
  return c.text(`${c.get('requestId')}`)
})

const res = await app.request('/', {
  headers: {
    'X-Request-Id': 'your-custom-id',
  },
})
console.log(await res.text()) // your-custom-id
```

如果你想禁用此功能，请将 [`headerName` 选项](#headername-string) 设置为空字符串。

## 选项

### <Badge type="info" text="optional" /> limitLength: `number`

Request ID 的最大长度。默认值为 `255`。

### <Badge type="info" text="optional" /> headerName: `string`

用于 Request ID 的 header 名称。默认值为 `X-Request-Id`。

### <Badge type="info" text="optional" /> generator: `(c: Context) => string`

Request ID 生成函数。默认情况下，它使用 `crypto.randomUUID()`。

## 平台特定的 Request IDs

一些平台（如 AWS Lambda）已经为每个请求生成自己的 Request IDs。
无需任何额外配置，此中间件不知道这些特定的 Request IDs
并生成新的 Request ID。这可能在查看应用程序日志时导致混淆。

要统一这些 ID，请使用 `generator` 函数捕获平台特定的 Request ID 并在此中间件中使用它。

### 平台特定链接

- AWS Lambda
  - [AWS 文档：Context 对象](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-context.html)
  - [Hono：访问 AWS Lambda 对象](/docs/getting-started/aws-lambda#access-aws-lambda-object)
- Cloudflare
  - [Cloudflare Ray ID](https://developers.cloudflare.com/fundamentals/reference/cloudflare-ray-id/)
- Deno
  - [Deno 博客上的 Request ID](https://deno.com/blog/zero-config-debugging-deno-opentelemetry#:~:text=s%20automatically%20have-,unique%20request%20IDs,-associated%20with%20them)
- Fastly
  - [Fastly 文档：req.xid](https://www.fastly.com/documentation/reference/vcl/variables/client-request/req-xid/)
