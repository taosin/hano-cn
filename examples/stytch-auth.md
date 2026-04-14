# 使用 Hono 的 Stytch Auth

本示例展示了如何在 Cloudflare Workers 上使用 Stytch Frontend SDK 和 Hono backend 设置全栈应用，搭配 `vite` 和 `react`。

使用这些原则的完整示例应用可以在 [这里](https://github.com/honojs/examples/tree/main/stytch-auth) 找到。

## 安装

::: code-group

```sh [npm]
# Backend
npm install @hono/stytch-auth stytch

# Frontend
npm install @stytch/react @stytch/vanilla-js
```

```sh [yarn]
# Backend
yarn add @hono/stytch-auth stytch

# Frontend
yarn add @stytch/react @stytch/vanilla-js
```

```sh [pnpm]
# Backend
pnpm add @hono/stytch-auth stytch

# Frontend
pnpm add @stytch/react @stytch/vanilla-js
```

```sh [bun]
# Backend
bun add @hono/stytch-auth stytch

# Frontend
bun add @stytch/react @stytch/vanilla-js
```

:::

## 设置

1. 创建 [Stytch](https://stytch.com/?utm_source=hono&utm_medium=website&utm_campaign=workers) 账户并选择 **Consumer Authentication**。
2. 在 [Configuration](https://stytch.com/dashboard/sdk-configuration) 中启用 **Frontend SDK**。
3. 从 [Project Settings](https://stytch.com/dashboard) 获取你的凭证。

## 环境变量

Backend Workers 环境变量放在 `.dev.vars` 中。Frontend Vite 环境变量放在 `.env.local` 中。

::: code-group

```Plain Text[.dev.vars]
STYTCH_PROJECT_ID=project-live-xxx
STYTCH_PROJECT_SECRET=secret-live-xxx
```

```Plain Text[.env.local]
VITE_STYTCH_PUBLIC_TOKEN=public-token-live-xxx
```

:::

## 前端

1. 使用 `<StytchProvider />` 组件包装你的应用，并传入 Stytch UI Client 的实例。
2. 使用 `<StytchLogin />` 组件让用户登录。查看 [Component Playground](https://stytch.com/docs/sdks/component-playground) 了解不同认证方法和样式自定义的示例。
3. 用户登录后，可以使用 `useStytchUser()` 钩子检索活跃用户数据。
4. 用户的会话信息将自动存储为 cookie 并可供你的 backend 使用。

::: code-group

```tsx[App.tsx]
import React from 'react'
import {StytchUIClient} from '@stytch/vanilla-js';
import {StytchProvider, useStytchUser} from '@stytch/react';
import LoginPage from './LoginPage'
import Dashboard from './Dashboard'

const stytch = new StytchUIClient(import.meta.env.VITE_STYTCH_PUBLIC_TOKEN ?? '');

function AppContent() {
  const { user, isInitialized } = useStytchUser()

  if (!isInitialized) return <div>Loading...</div>
  return user ? <Dashboard /> : <LoginPage />
}

function App() {
  return (
    <StytchProvider stytch={stytch}>
      <AppContent />
    </StytchProvider>
  )
}

export default App
```

```tsx[LoginPage.tsx]
import React from 'react'
import { StytchLogin } from '@stytch/react'
import { Products, OTPMethods } from '@stytch/vanilla-js'

const loginConfig = {
  products: [Products.otp],
  otpOptions: {
    expirationMinutes: 10,
    methods: [OTPMethods.Email],
  },
}

const LoginPage = () => {
  return <StytchLogin config={loginConfig} />
}

export default LoginPage
```

```tsx[Dashboard.tsx]
import React from 'react'
import { useStytchUser, useStytch } from '@stytch/react'

const Dashboard = () => {
  const { user } = useStytchUser()
  const stytchClient = useStytch()

  const handleLogout = () => stytchClient.session.revoke()

  return (
    <div style={{ maxWidth: '600px', margin: '2rem auto' }}>
      <div style={{ display: 'flex', justifyContent: 'space-between' }}>
        <h1>Dashboard</h1>
        <button onClick={handleLogout}>Logout</button>
      </div>
      <p>Welcome, {user.emails[0]?.email}!</p>
    </div>
  )
}

export default Dashboard
```

:::

## Backend

1. 使用 `Consumer.authenticateSessionLocal()` 中间件包装受保护的端点以认证 Stytch 会话 JWT。
2. 在路由中使用 `Consumer.getStytchSession(c)` 方法检索 Stytch 会话信息。
3. 需要完整用户对象的路由可以使用 `Consumer.authenticateSessionRemote()` 方法向 Stytch 服务器执行网络调用。

```ts[src/index.ts]
import { Hono } from 'hono'
import { Consumer } from '@hono/stytch-auth'

const app = new Hono()

// 公共路由
app.get('/health', (c) => c.json({ status: 'ok' }))

// 具有本地认证的受保护路由（非常快）
app.get('/api/local', Consumer.authenticateSessionLocal(), (c) => {
  const session = Consumer.getStytchSession(c)
  return c.json({
    message: 'Protected data',
    sessionId: session.session_id,
  })
})

// 具有远程认证和完整用户数据的受保护路由
app.get('/api/remote', Consumer.authenticateSessionRemote(), (c) => {
  const session = Consumer.getStytchSession(c)
  const user = Consumer.getStytchUser(c)
  return c.json({
    message: 'Protected data',
    sessionId: session.session_id,
    firstName: user.name.first_name,
  })
})

export default app
```

## 下一步

更多文档和资源：

- 查看 [Stytch Auth Hono 示例应用](https://github.com/honojs/examples/tree/main/stytch-auth)。
- [Stytch JS SDK](https://stytch.com/docs/sdks/installation) 入门指南。
- [@hono/stytch-auth 包](https://www.npmjs.com/package/@hono/stytch-auth) 完整文档。

对企业 B2B 功能（如组织管理、RBAC 和 SSO）感兴趣？查看 [Stytch B2B Authentication](https://stytch.com/docs/getting-started/b2b-vs-consumer-auth) 产品线。

加入讨论、提问并在 [Stytch Slack Community](https://stytch.com/docs/resources/support/overview) 中建议新功能。
