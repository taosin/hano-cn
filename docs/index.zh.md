---
title: Hono - 基于 Web Standards 构建的 Web 框架
titleTemplate: ':title'
---

# Hono

Hono - _**在日语中意为火焰🔥**_ - 是一个小型、简单且超快的 Web 框架，基于 Web Standards 构建。
它可以在任何 JavaScript 运行时上运行：Cloudflare Workers、Fastly Compute、Deno、Bun、Vercel、Netlify、AWS Lambda、Lambda@Edge 和 Node.js。

快速，但不止于快速。

```ts twoslash
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => c.text('Hono!'))

export default app
```

## 快速开始

只需运行：

::: code-group

```sh [npm]
npm create hono@latest
```

```sh [yarn]
yarn create hono
```

```sh [pnpm]
pnpm create hono@latest
```

```sh [bun]
bun create hono@latest
```

```sh [deno]
deno init --npm hono@latest
```

:::

## 特性

- **超快** 🚀 - `RegExpRouter` 路由器非常快。不使用线性循环。快速。
- **轻量** 🪶 - `hono/tiny` 预设小于 14kB。Hono 零依赖，仅使用 Web Standards。
- **多运行时** 🌍 - 可在 Cloudflare Workers、Fastly Compute、Deno、Bun、AWS Lambda 或 Node.js 上运行。相同的代码在所有平台上都能运行。
- **功能齐全** 🔋 - Hono 内置了中间件、自定义中间件、第三方中间件和辅助函数。功能齐全。
- **愉悦的开发体验** 😃 - 超干净的 API。一流的 TypeScript 支持。现在，我们有了"类型"。

## 使用场景

Hono 是一个简单的 Web 应用程序框架，类似于 Express，不包含前端。
但它可以运行在 CDN 边缘，结合中间件可以构建更大的应用程序。
以下是一些使用场景示例。

- 构建 Web API
- 后端服务器代理
- CDN 前端
- 边缘应用程序
- 库的基础服务器
- 全栈应用程序

## 谁在使用 Hono？

| 项目                                                                               | 平台               | 用途                                                                                                         |
| ---------------------------------------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------ |
| [cdnjs](https://cdnjs.com)                                                         | Cloudflare Workers | 免费开源的 CDN 服务。_Hono 用于 API 服务器_。                                                                 |
| [Cloudflare D1](https://www.cloudflare.com/developer-platform/d1/)                 | Cloudflare Workers | 无服务器 SQL 数据库。_Hono 用于内部 API 服务器_。                                                             |
| [Cloudflare Workers KV](https://www.cloudflare.com/developer-platform/workers-kv/) | Cloudflare Workers | 无服务器键值数据库。_Hono 用于内部 API 服务器_。                                                              |
| [BaseAI](https://baseai.dev)                                                       | 本地 AI 服务器     | 带记忆的无服务器 AI 代理管道。用于 Web 的开源代理 AI 框架。_使用 Hono 的 API 服务器_。                        |
| [Unkey](https://unkey.dev)                                                         | Cloudflare Workers | 开源 API 认证和授权。_Hono 用于 API 服务器_。                                                                 |
| [OpenStatus](https://openstatus.dev)                                               | Bun                | 开源网站和 API 监控平台。_Hono 用于 API 服务器_。                                                             |
| [Deno Benchmarks](https://deno.com/benchmarks)                                     | Deno               | 基于 V8 构建的安全 TypeScript 运行时。_Hono 用于基准测试_。                                                   |
| [Clerk](https://clerk.com)                                                         | Cloudflare Workers | 开源用户管理平台。_Hono 用于 API 服务器_。                                                                    |

以及以下项目：

- [Drivly](https://driv.ly/) - Cloudflare Workers
- [repeat.dev](https://repeat.dev/) - Cloudflare Workers

想看更多？查看 [谁在生产环境中使用 Hono？](https://github.com/orgs/honojs/discussions/1510)。

## 1 分钟了解 Hono

使用 Hono 创建 Cloudflare Workers 应用程序的演示。

![一个 gif，展示使用 Hono 快速创建应用程序并快速迭代。](/images/sc.gif)

## 超快

**Hono 是最快的**，与其他 Cloudflare Workers 路由器相比。

```
Hono x 402,820 ops/sec ±4.78% (80 runs sampled)
itty-router x 212,598 ops/sec ±3.11% (87 runs sampled)
sunder x 297,036 ops/sec ±4.76% (77 runs sampled)
worktop x 197,345 ops/sec ±2.40% (88 runs sampled)
Fastest is Hono
✨  Done in 28.06s.
```

查看 [更多基准测试](/docs/concepts/benchmarks)。

## 轻量

**Hono 非常小**。使用 `hono/tiny` 预设，压缩后大小**小于 14KB**。有许多中间件和适配器，但仅在使用时才会打包。作为参考，Express 的大小为 572KB。

```
$ npx wrangler dev --minify ./src/index.ts
 ⛅️ wrangler 2.20.0
--------------------
⬣ Listening at http://0.0.0.0:8787
- http://127.0.0.1:8787
- http://192.168.128.165:8787
Total Upload: 11.47 KiB / gzip: 4.34 KiB
```

## 多个路由器

**Hono 有多个路由器**。

**RegExpRouter** 是 JavaScript 世界中最快的路由器。它使用调度前创建的单个大型 Regex 匹配路由。使用 **SmartRouter**，它支持所有路由模式。

**LinearRouter** 注册路由非常快，因此适合每次初始化应用程序的环境。**PatternRouter** 简单地添加和匹配模式，使其变小。

查看 [有关路由的更多信息](/docs/concepts/routers)。

## Web Standards

由于使用 **Web Standards**，Hono 可以在许多平台上运行。

- Cloudflare Workers
- Cloudflare Pages
- Fastly Compute
- Deno
- Bun
- Vercel
- AWS Lambda
- Lambda@Edge
- 其他

通过使用 [Node.js 适配器](https://github.com/honojs/node-server)，Hono 可以在 Node.js 上运行。

查看 [有关 Web Standards 的更多信息](/docs/concepts/web-standard)。

## 中间件和辅助函数

**Hono 有许多中间件和辅助函数**。这使得"少写代码，多做事情"成为现实。

开箱即用，Hono 提供以下中间件和辅助函数：

- [基本认证](/docs/middleware/builtin/basic-auth)
- [Bearer 认证](/docs/middleware/builtin/bearer-auth)
- [请求体限制](/docs/middleware/builtin/body-limit)
- [缓存](/docs/middleware/builtin/cache)
- [压缩](/docs/middleware/builtin/compress)
- [上下文存储](/docs/middleware/builtin/context-storage)
- [Cookie](/docs/helpers/cookie)
- [CORS](/docs/middleware/builtin/cors)
- [ETag](/docs/middleware/builtin/etag)
- [html](/docs/helpers/html)
- [JSX](/docs/guides/jsx)
- [JWT 认证](/docs/middleware/builtin/jwt)
- [日志记录器](/docs/middleware/builtin/logger)
- [语言](/docs/middleware/builtin/language)
- [Pretty JSON](/docs/middleware/builtin/pretty-json)
- [安全头](/docs/middleware/builtin/secure-headers)
- [SSG](/docs/helpers/ssg)
- [流式传输](/docs/helpers/streaming)
- [GraphQL 服务器](https://github.com/honojs/middleware/tree/main/packages/graphql-server)
- [Firebase 认证](https://github.com/honojs/middleware/tree/main/packages/firebase-auth)
- [Sentry](https://github.com/honojs/middleware/tree/main/packages/sentry)
- 其他！

例如，使用 Hono 只需几行代码即可添加 ETag 和请求日志记录：

```ts
import { Hono } from 'hono'
import { etag } from 'hono/etag'
import { logger } from 'hono/logger'

const app = new Hono()
app.use(etag(), logger())
```

查看 [有关中间件的更多信息](/docs/concepts/middleware)。

## 开发体验

Hono 提供愉悦的"**开发体验**"。

得益于 `Context` 对象，可以轻松访问请求/响应。
此外，Hono 使用 TypeScript 编写。Hono 拥有"**类型**"。

例如，路径参数将是字面量类型。

![截图显示 Hono 在使用 URL 参数时具有适当的字面量类型。URL "/entry/:date/:id" 允许请求参数为 "date" 或 "id"](/images/ss.png)

此外，Validator 和 Hono Client `hc` 启用 RPC 模式。在 RPC 模式中，
你可以使用喜欢的验证器（如 Zod），轻松与客户端共享服务器端 API 规范，并构建类型安全的应用程序。

查看 [Hono 技术栈](/docs/concepts/stacks)。
