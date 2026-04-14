# Timeout 中间件

Timeout 中间件使你能够轻松管理应用程序中的请求超时。它允许你设置请求的最大持续时间，并可选地定义如果超过指定超时的自定义错误响应。

## 导入

```ts
import { Hono } from 'hono'
import { timeout } from 'hono/timeout'
```

## 用法

以下是如何使用具有默认和自定义设置的 Timeout 中间件：

默认设置：

```ts
const app = new Hono()

// 应用 5 秒超时
app.use('/api', timeout(5000))

// 处理路由
app.get('/api/data', async (c) => {
  // 你的路由 handler 逻辑
  return c.json({ data: 'Your data here' })
})
```

自定义设置：

```ts
import { HTTPException } from 'hono/http-exception'

// 自定义异常工厂函数
const customTimeoutException = (context) =>
  new HTTPException(408, {
    message: `Request timeout after waiting ${context.req.headers.get(
      'Duration'
    )} seconds. Please try again later.`,
  })

// 对于静态异常消息
// const customTimeoutException = new HTTPException(408, {
//   message: 'Operation timed out. Please try again later.'
// });

// 应用 1 分钟超时并带有自定义异常
app.use('/api/long-process', timeout(60000, customTimeoutException))

app.get('/api/long-process', async (c) => {
  // 模拟长时间过程
  await new Promise((resolve) => setTimeout(resolve, 61000))
  return c.json({ data: 'This usually takes longer' })
})
```

## 注意事项

- 超时的持续时间可以以毫秒为单位指定。如果超过指定的持续时间，中间件将自动拒绝 promise 并可能抛出错误。

- timeout 中间件不能与 stream 一起使用因此，请一起使用 `stream.close` 和 `setTimeout`。

```ts
app.get('/sse', async (c) => {
  let id = 0
  let running = true
  let timer: number | undefined

  return streamSSE(c, async (stream) => {
    timer = setTimeout(() => {
      console.log('Stream timeout reached, closing stream')
      stream.close()
    }, 3000) as unknown as number

    stream.onAbort(async () => {
      console.log('Client closed connection')
      running = false
      clearTimeout(timer)
    })

    while (running) {
      const message = `It is ${new Date().toISOString()}`
      await stream.writeSSE({
        data: message,
        event: 'time-update',
        id: String(id++),
      })
      await stream.sleep(1000)
    }
  })
})
```

## 中间件冲突

注意中间件的顺序，特别是使用错误处理或其他与时间相关的中间件时，因为它可能会影响此 timeout 中间件的行为。
