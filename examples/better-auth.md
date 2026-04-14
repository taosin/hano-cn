# Better Auth

使用 Hono 搭配 [Better Auth](http://better-auth.com/) 进行认证。

Better Auth 是一个与框架无关的 TypeScript 认证和授权框架。它提供了全面的开箱即用功能，并包含一个插件生态系统，可简化高级功能的添加。

## 配置

1. 安装框架：

```sh
# npm
npm install better-auth

# bun
bun add better-auth

# pnpm
pnpm add better-auth

# yarn
yarn add better-auth
```

2. 在 `.env` 文件中添加所需的环境变量：

```sh
BETTER_AUTH_SECRET=<生成一个密钥> (例如 D27gijdvth3Ul3DjGcexjcFfgCHc8jWd)
BETTER_AUTH_URL=<你的服务器 URL> (例如 http://localhost:1234)
```

3. 创建 Better Auth 实例

```ts
import { betterAuth } from 'better-auth'
import { prismaAdapter } from 'better-auth/adapters/prisma'

import prisma from '@/db/index'
import env from '@/env'

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: 'postgresql',
  }),
  // 允许来自前端开发服务器的请求
  trustedOrigins: ['http://localhost:5173'],
  emailAndPassword: {
    enabled: true,
  },
  socialProviders: {
    github: {
      clientId: env.GITHUB_CLIENT_ID,
      clientSecret: env.GITHUB_CLIENT_SECRET,
    },
    google: {
      clientId: env.GOOGLE_CLIENT_ID,
      clientSecret: env.GOOGLE_CLIENT_SECRET,
    },
  },
})

export type AuthType = {
  user: typeof auth.$Infer.Session.user | null
  session: typeof auth.$Infer.Session.session | null
}
```

上述代码：

- 设置数据库使用 Prisma ORM 和 PostgreSQL
- 指定受信任的源
  - 受信任的源是允许向认证 API 发出请求的应用。通常，那是你的客户端（前端）
  - 所有其他源会被自动阻止
- 启用邮箱/密码认证并配置社交登录提供商。

4. 生成所有必需的模型、字段和关系到 Prisma schema 文件：

```sh
bunx @better-auth/cli generate
```

5. 在 `routes/auth.ts` 中为认证 API 请求创建 API 处理器

此路由使用 Better Auth 提供的处理器来服务所有对 `/api/auth` 端点的 `POST` 和 `GET` 请求。

```ts
import { Hono } from 'hono'
import { auth } from '../lib/auth'
import type { AuthType } from '../lib/auth'

const router = new Hono<{ Bindings: AuthType }>({
  strict: false,
})

router.on(['POST', 'GET'], '/auth/*', (c) => {
  return auth.handler(c.req.raw)
})

export default router
```

6. 挂载路由

以下代码挂载路由。

```ts
import { Hono } from "hono";
import type { AuthType } from "../lib/auth"
import auth from "@/routes/auth";

const app = new Hono<{ Variables: AuthType }>({
  strict: false,
});

const routes = [auth, ...其他路由] as const;

routes.forEach((route) => {
  app.basePath("/api").route("/", route);
});

export default app;
```

## 另请参阅

- [包含完整代码的仓库](https://github.com/catalinpit/example-app/)
- [使用 Hono、Bun、TypeScript、React 和 Vite 的 Better Auth](https://catalins.tech/better-auth-with-hono-bun-typescript-react-vite/)
