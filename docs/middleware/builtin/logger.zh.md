# Logger 中间件

它是一个简单的记录器。

## 导入

```ts
import { Hono } from 'hono'
import { logger } from 'hono/logger'
```

## 用法

```ts
const app = new Hono()

app.use(logger())
app.get('/', (c) => c.text('Hello Hono!'))
```

## 记录详情

Logger 中间件为每个请求记录以下详情：

- **传入请求**：记录 HTTP 方法、请求路径和传入请求。
- **传出响应**：记录 HTTP 方法、请求路径、响应状态码和请求/响应时间。
- **状态码着色**：响应状态码进行颜色编码，以提高可见性和快速识别状态类别。不同的状态码类别由不同的颜色表示。
- **经过时间**：请求/响应周期所花费的时间以人类可读的格式记录，以毫秒 (ms) 或秒 (s) 为单位。

通过使用 Logger 中间件，你可以轻松监控 Hono 应用程序中的请求和响应流，并快速识别任何问题或性能瓶颈。

你还可以通过提供自己的 `PrintFunc` 函数来进一步扩展中间件，以实现定制的记录行为。

::: tip

要禁用_状态码着色_，你可以设置 `NO_COLOR` 环境变量。这是在记录库中禁用 ANSI 颜色转义代码的常见方法，并在 <https://no-color.org/> 中描述。请注意，Cloudflare Workers 没有 `process.env` 对象，因此将默认为纯文本日志输出。
:::

## PrintFunc

Logger 中间件接受可选的 `PrintFunc` 函数作为参数。此函数允许你自定义记录器并添加额外的日志。

## 选项

### <Badge type="info" text="optional" /> fn: `PrintFunc(str: string, ...rest: string[])`

- `str`：由记录器传递。
- `...rest`：要打印到控制台的其他字符串属性。

### 示例

将自定义 `PrintFunc` 函数设置到 Logger 中间件：

```ts
export const customLogger = (message: string, ...rest: string[]) => {
  console.log(message, ...rest)
}

app.use(logger(customLogger))
```

在路由中设置自定义记录器：

```ts
app.post('/blog', (c) => {
  // 路由逻辑

  customLogger('Blog saved:', `Path: ${blog.url},`, `ID: ${blog.id}`)
  // 输出
  // <-- POST /blog
  // Blog saved: Path: /blog/example, ID: 1
  // --> POST /blog 201 93ms

  // 返回 Context
})
```
