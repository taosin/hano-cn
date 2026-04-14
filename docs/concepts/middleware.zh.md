# 中间件

我们将返回 `Response` 的原语称为 "Handler"（处理程序）。
"Middleware"（中间件）在处理程序之前和之后执行，并处理 `Request` 和 `Response`。
它像洋葱结构。

![](/images/onion.png)

例如，我们可以编写中间件来添加 "X-Response-Time" 头部，如下所示。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()
// ---cut---
app.use(async (c, next) => {
  const start = performance.now()
  await next()
  const end = performance.now()
  c.res.headers.set('X-Response-Time', `${end - start}`)
})
```

使用这个简单的方法，我们可以编写自己的自定义中间件，也可以使用内置或第三方中间件。
