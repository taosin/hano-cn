# Combine 中间件

Combine 中间件将多个中间件函数组合为单个中间件。它提供三个函数：

- `some` - 仅运行给定中间件中的一个。
- `every` - 运行所有给定中间件。
- `except` - 仅当不满足条件时运行所有给定中间件。

## 导入

```ts
import { Hono } from 'hono'
import { some, every, except } from 'hono/combine'
```

## 用法

以下是使用 Combine 中间件的复杂访问控制规则示例。

```ts
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'
import { getConnInfo } from 'hono/cloudflare-workers'
import { every, some } from 'hono/combine'
import { ipRestriction } from 'hono/ip-restriction'
import { rateLimit } from '@/my-rate-limit'

const app = new Hono()

app.use(
  '*',
  some(
    every(
      ipRestriction(getConnInfo, { allowList: ['192.168.0.2'] }),
      bearerAuth({ token })
    ),
    // 如果两个条件都满足，rateLimit 将不会执行。
    rateLimit()
  )
)

app.get('/', (c) => c.text('Hello Hono!'))
```

### some

运行返回 true 的第一个中间件。中间件按顺序应用，如果任何中间件成功退出，后续中间件将不会运行。

```ts
import { some } from 'hono/combine'
import { bearerAuth } from 'hono/bearer-auth'
import { myRateLimit } from '@/rate-limit'

// 如果客户端有有效令牌，则跳过速率限制。
// 否则，应用速率限制。
app.use(
  '/api/*',
  some(bearerAuth({ token }), myRateLimit({ limit: 100 }))
)
```

### every

运行所有中间件，如果任何中间件失败则停止。中间件按顺序应用，如果任何中间件抛出错误，后续中间件将不会运行。

```ts
import { some, every } from 'hono/combine'
import { bearerAuth } from 'hono/bearer-auth'
import { myCheckLocalNetwork } from '@/check-local-network'
import { myRateLimit } from '@/rate-limit'

// 如果客户端在本地网络中，则跳过认证和速率限制。
// 否则，应用认证和速率限制。
app.use(
  '/api/*',
  some(
    myCheckLocalNetwork(),
    every(bearerAuth({ token }), myRateLimit({ limit: 100 }))
  )
)
```

### except

运行所有中间件，除非满足条件。你可以传递字符串或函数作为条件。如果需要匹配多个目标，请将它们作为数组传递。

```ts
import { except } from 'hono/combine'
import { bearerAuth } from 'hono/bearer-auth'

// 如果客户端正在访问公共 API，则跳过认证。
// 否则，需要有效令牌。
app.use('/api/*', except('/api/public/*', bearerAuth({ token })))
```
