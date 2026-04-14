# Cloudflare 上的 Better Auth

一个专为 Cloudflare Workers 优化的 TypeScript 轻量级认证服务

## 技术栈概览

**🔥 [Hono](https://hono.dev)**  
一个基于 Web 标准构建的快速、轻量级 Web 框架。

**🔒 [Better Auth](https://www.better-auth.com)**  
一个全面的 TypeScript 认证框架。

**🧩 [Drizzle ORM](https://orm.drizzle.team)**  
一个轻量级、高性能的 TypeScript ORM，专为开发者体验而设计。

**🐘 [Postgres with Neon](https://neon.tech)**  
专为云环境优化的无服务器 Postgres。

## 准备工作

### 1. 安装

::: code-group

```sh [npm]
# Hono
# > 选择 cloudflare-workers 模板
npm create hono

# Better Auth
npm install better-auth

# Drizzle ORM
npm install drizzle-orm
npm install --save-dev drizzle-kit

# Neon
npm install @neondatabase/serverless
```

```sh [pnpm]
# Hono
# > 选择 cloudflare-workers 模板
pnpm create hono

# Better Auth
pnpm add better-auth

# Drizzle ORM
pnpm add drizzle-orm
pnpm add -D drizzle-kit

# Neon
pnpm add @neondatabase/serverless
```

```sh [yarn]
# Hono
# > 选择 cloudflare-workers 模板
yarn create hono

# Better Auth
yarn add better-auth

# Drizzle ORM
yarn add drizzle-orm
yarn add --dev drizzle-kit

# Neon
yarn add @neondatabase/serverless
```

```sh [bun]
# Hono
# > 选择 cloudflare-workers 模板
bun create hono

# Better Auth
bun add better-auth

# Drizzle ORM
bun add drizzle-orm
bun add -d drizzle-kit

# Neon
bun add @neondatabase/serverless
```

:::

### 2. 环境变量

设置以下环境变量，将你的应用连接到 Better Auth 和 Neon。

参考官方指南：

- [Better Auth – 指南](https://www.better-auth.com/docs/installation#set-environment-variables)
- [Neon – 指南](https://neon.tech/docs/connect/connect-from-any-app)

**必需文件：**

::: code-group

```Plain Text[.dev.vars]
# 由 Wrangler 在本地开发时使用
# 在生产环境中，这些应设置为 Cloudflare Worker Secrets。

BETTER_AUTH_URL=
BETTER_AUTH_SECRET=
DATABASE_URL=
```

```Plain Text[.env]
# 用于本地开发和 CLI 工具，例如：
#
# - Drizzle CLI
# - Better Auth CLI

BETTER_AUTH_URL=
BETTER_AUTH_SECRET=
DATABASE_URL=
```

:::

### 3. Wrangler

设置好环境变量后，运行以下脚本为 Cloudflare Workers 配置生成类型：

::: code-group

```sh[npm]
npx wrangler types --env-interface CloudflareBindings
# 或
npm run cf-typegen
```

```sh[pnpm]
pnpm wrangler types --env-interface CloudflareBindings
# 或
pnpm cf-typegen

```

```sh[yarn]
yarn wrangler types --env-interface CloudflareBindings
# 或
yarn cf-typegen
```

```sh[bun]
bunx wrangler types --env-interface CloudflareBindings
# 或
bun run cf-typegen
```

:::

然后，确保你的 tsconfig.json 包含生成的类型。

```json[tsconfig.json]
{
  "compilerOptions": {
    "types": ["worker-configuration.d.ts"]
  }
}
```

### 4. Drizzle

要使用 Drizzle Kit CLI，请在项目根目录添加以下 Drizzle 配置文件。

```ts[drizzle.config.ts]
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

## 应用

### 1. Better Auth 实例

使用 Cloudflare Workers bindings 创建 Better Auth 实例。

有许多可用的配置选项，远超本示例所能涵盖的范围。
请参考官方文档，根据项目需求进行配置：

(文档：[Better Auth - 选项](https://www.better-auth.com/docs/reference/options))

::: code-group

```ts[src/lib/better-auth/index.ts]
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { betterAuth } from 'better-auth';
import { betterAuthOptions } from './options';

import * as schema from "../db/schema"; // 确保导入 schema

/**
 * Better Auth 实例
 */
export const auth = (env: CloudflareBindings): ReturnType<typeof betterAuth> => {
  const sql = neon(env.DATABASE_URL);
  const db = drizzle(sql);

  return betterAuth({
    ...betterAuthOptions,
    database: drizzleAdapter(db, { provider: 'pg' }),
    baseURL: env.BETTER_AUTH_URL,
    secret: env.BETTER_AUTH_SECRET,

    // 其他依赖于 env 的选项 ...
  });
};
```

```ts[src/lib/better-auth/options.ts]
import { BetterAuthOptions } from 'better-auth';

/**
 * Better Auth 自定义选项
 *
 * 文档：https://www.better-auth.com/docs/reference/options
 */
export const betterAuthOptions: BetterAuthOptions = {
  /**
   * 应用名称。
   */
  appName: 'YOUR_APP_NAME',
  /**
   * Better Auth 的基础路径。
   * @default "/api/auth"
   */
  basePath: '/api',

  // .... 更多选项
};
```

:::

### 2. Better Auth Schema

要为 Better Auth 创建所需的表，首先在根目录添加以下文件：

```ts[better-auth.config.ts]
/**
 * Better Auth CLI 配置文件
 *
 * 文档：https://www.better-auth.com/docs/concepts/cli
 */
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { betterAuth } from 'better-auth';
import { betterAuthOptions } from './src/lib/better-auth/options';

const { DATABASE_URL, BETTER_AUTH_URL, BETTER_AUTH_SECRET } = process.env;

const sql = neon(DATABASE_URL!);
const db = drizzle(sql);

export const auth: ReturnType<typeof betterAuth> = betterAuth({
  ...betterAuthOptions,
  database: drizzleAdapter(db, { provider: 'pg', schema }),  // 需要 schema 以便 better-auth 识别
  baseURL: BETTER_AUTH_URL,
  secret: BETTER_AUTH_SECRET,
});
```

然后，执行以下脚本：

::: code-group

```sh[npm]
npx @better-auth/cli@latest generate --config ./better-auth.config.ts --output ./src/db/schema.ts
```

```sh[pnpm]
pnpm dlx @better-auth/cli@latest generate --config ./better-auth.config.ts --output ./src/db/schema.ts
```

```sh[yarn]
yarn dlx @better-auth/cli@latest generate --config ./better-auth.config.ts --output ./src/db/schema.ts
```

```sh[bun]
bunx @better-auth/cli@latest generate --config ./better-auth.config.ts --output ./src/db/schema.ts
```

:::

### 3. 将 Schema 应用到数据库

生成 schema 文件后，运行以下命令来创建并应用数据库迁移：
检查你的 wrangler 配置以正确读取 `process.env`，确保 `wrangler dev` 之后能正常工作。你需要 [`node_compatibility`](https://developers.cloudflare.com/workers/wrangler/configuration/#hyperdrive) 设置。

::: code-group

```sh[npm]
npx drizzle-kit generate
npx drizzle-kit migrate
```

```sh[pnpm]
pnpm drizzle-kit generate
pnpm drizzle-kit migrate
```

```sh[yarn]
yarn drizzle-kit generate
yarn drizzle-kit migrate
```

```sh[bun]
bunx drizzle-kit generate
bunx drizzle-kit migrate
```

:::

### 4. 挂载处理器

将 Better Auth 处理器挂载到 Hono 端点，确保挂载路径与 Better Auth 实例中的 `basePath` 设置匹配。

```ts[src/index.ts]
import { Hono } from 'hono';
import { auth } from './lib/better-auth';

const app = new Hono<{ Bindings: CloudflareBindings }>();

app.on(['GET', 'POST'], '/api/*', (c) => {
  return auth(c.env).handler(c.req.raw);
});

export default app;
```

## 进阶

本示例基于 Hono、Better Auth 和 Drizzle 的官方文档组装而成。然而，它不仅仅是简单的集成，还提供了以下优势：

- 通过整合 Cloudflare CLI、Better Auth CLI 和 Drizzle CLI 实现高效开发。
- 开发和生产环境之间的无缝过渡。
- 使用脚本一致地应用更改。

你可以扩展此设置，添加适合你工作流的自定义脚本。例如：

```json[package.json]
{
  "scripts": {
    "dev": "wrangler dev",
    "deploy": "pnpm run cf-gen-types && wrangler secret bulk .dev.vars.production && wrangler deploy --minify",
    "cf-gen-types": "wrangler types --env-interface CloudflareBindings",
    "better-auth-gen-schema": "pnpm dlx @better-auth/cli@latest generate --config ./better-auth.config.ts --output ./src/db/schema.ts"
  },
}
```

> 注意：  
> 请参考每个工具的官方 CLI 文档，了解高级用法和最新选项：
>
> - [Cloudflare CLI](https://developers.cloudflare.com/workers/wrangler/)
> - [Better Auth CLI](https://www.better-auth.com/docs/concepts/cli)
> - [Drizzle ORM CLI](https://orm.drizzle.team/docs/kit-overview)

## 结语

现在你已经拥有一个运行在 Cloudflare Workers 上的轻量级、快速且全面的认证服务。通过利用 Service Bindings，此设置允许你以最小延迟构建基于微服务的架构。

本指南仅展示了一个**基础示例**，因此对于 OAuth 或速率限制等高级用例，请参考官方文档并根据你的服务需求定制配置。

你可以在这里找到完整的示例源代码：  
[GitHub 仓库](https://github.com/bytaesu/cloudflare-auth-worker)
