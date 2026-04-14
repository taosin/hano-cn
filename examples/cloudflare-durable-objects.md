# Cloudflare Durable Objects

Cloudflare Durable Objects 无法直接处理 HTTP 请求。相反，它们通过一个两步流程工作：

1. Worker 从客户端接收 HTTP fetch 请求
2. Worker 向 Durable Object 发出 RPC（远程过程调用）调用
3. Durable Object 处理 RPC 并将结果返回给 Worker
4. Worker 将 HTTP 响应发送回客户端

你可以使用 Hono 作为 Cloudflare Worker 中的路由器，调用 RPC（远程过程调用）来与 [Durable Objects](https://developers.cloudflare.com/durable-objects/) 交互。这是自 Cloudflare Workers 兼容性日期 `2024-04-03` 以来的推荐方法。

## 示例：计数器 Durable Object

```ts
import { DurableObject } from 'cloudflare:workers'
import { Hono } from 'hono'

export class Counter extends DurableObject {
  // 内存状态
  value = 0

  constructor(ctx: DurableObjectState, env: unknown) {
    super(ctx, env)

    // `blockConcurrencyWhile()` 确保在初始化完成之前不会处理任何请求。
    ctx.blockConcurrencyWhile(async () => {
      // 初始化完成后，未来的读取无需访问存储。
      this.value = (await ctx.storage.get('value')) || 0
    })
  }

  async getCounterValue() {
    return this.value
  }

  async increment(amount = 1): Promise<number> {
    this.value += amount
    await this.ctx.storage.put('value', this.value)
    return this.value
  }

  async decrement(amount = 1): Promise<number> {
    this.value -= amount
    await this.ctx.storage.put('value', this.value)
    return this.value
  }
}

// 创建一个新的 Hono 应用来处理传入的 HTTP 请求
type Bindings = {
  COUNTER: DurableObjectNamespace<Counter>
}

const app = new Hono<{ Bindings: Bindings }>()

// 添加与 Durable Object 交互的路由
app.get('/counter', async (c) => {
  const env = c.env
  const id = env.COUNTER.idFromName('counter')
  const stub = env.COUNTER.get(id)
  const counterValue = await stub.getCounterValue()
  return c.text(counterValue.toString())
})

app.post('/counter/increment', async (c) => {
  const env = c.env
  const id = env.COUNTER.idFromName('counter')
  const stub = env.COUNTER.get(id)
  const value = await stub.increment()
  return c.text(value.toString())
})

app.post('/counter/decrement', async (c) => {
  const env = c.env
  const id = env.COUNTER.idFromName('counter')
  const stub = env.COUNTER.get(id)
  const value = await stub.decrement()
  return c.text(value.toString())
})

// 将 Hono 应用导出为 Worker 的 fetch 处理器
export default app
```

`wrangler.jsonc`：

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "durable",
  "main": "src/index.ts",
  "compatibility_date": "2025-04-14",
  "migrations": [
    {
      "new_sqlite_classes": ["Counter"],
      "tag": "v1",
    },
  ],
  "durable_objects": {
    "bindings": [
      {
        "class_name": "Counter",
        "name": "COUNTER",
      },
    ],
  },
  "observability": {
    "enabled": true,
  },
}
```

现在你拥有了一个功能齐全的 Hono 应用，可以与你的 Durable Object 交互！Hono 路由器提供了一个清晰的 API 接口来与你的 Durable Object 的方法交互并将其暴露出来。
