# Cloudflare 测试

你可以使用 `@cloudflare/vitest-pool-workers` 轻松实现 Cloudflare 测试，为此需要预先进行一些配置，更多信息请参阅 [Cloudflare 关于测试的文档](https://developers.cloudflare.com/workers/testing/vitest-integration/get-started/write-your-first-test/)。

Cloudflare Testing 与 vitest pool workers 在运行时提供了一个 `cloudflare:test` 模块，该模块公开了在测试期间作为第二个参数传入的 env，更多信息请参阅 [Cloudflare Test APIs 部分](https://developers.cloudflare.com/workers/testing/vitest-integration/test-apis/)。

下面是一个配置示例：

:::code-group

```ts [vitest.config.ts]
import { defineWorkersProject } from '@cloudflare/vitest-pool-workers/config'

export default defineWorkersProject(() => {
  return {
    test: {
      globals: true,
      poolOptions: {
        workers: { wrangler: { configPath: './wrangler.toml' } },
      },
    },
  }
})
```

```toml [wrangler.toml]
compatibility_date = "2024-09-09"
compatibility_flags = [ "nodejs_compat" ]

[vars]
MY_VAR = "my variable"
```

:::

假设应用如下所示：

```ts
// src/index.ts
import { Hono } from 'hono'

type Bindings = {
  MY_VAR: string
}

const app = new Hono<{ Bindings: Bindings }>()

app.get('/hello', (c) => {
  return c.json({ hello: 'world', var: c.env.MY_VAR })
})

export default app
```

你可以通过将从模块 `cloudflare:test` 公开的 `env` 传递给 `app.request()` 来测试带有 Cloudflare Bindings 的应用：

```ts
// src/index.test.ts
import { env } from 'cloudflare:test'
import app from './index'

describe('Example', () => {
  it('Should return 200 response', async () => {
    const res = await app.request('/hello', {}, env)

    expect(res.status).toBe(200)
    expect(await res.json()).toEqual({
      hello: 'world',
      var: 'my variable',
    })
  })
})
```

## 另请参阅

`@cloudflare/vitest-pool-workers` [GitHub 仓库示例](https://github.com/cloudflare/workers-sdk/tree/main/fixtures/vitest-pool-workers-examples)\
[从旧测试系统迁移](https://developers.cloudflare.com/workers/testing/vitest-integration/get-started/migrate-from-miniflare-2/)
