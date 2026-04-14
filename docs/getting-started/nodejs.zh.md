# Node.js

[Node.js](https://nodejs.org/) 是一个开源、跨平台的 JavaScript 运行时环境。

Hono 最初并不是为 Node.js 设计的，但有了 [Node.js 适配器](https://github.com/honojs/node-server)，它也可以在 Node.js 上运行。

::: info
它适用于大于 18.x 版本的 Node.js。具体需要的 Node.js 版本如下：

- 18.x => 18.14.1+
- 19.x => 19.7.0+
- 20.x => 20.0.0+

基本上，你可以简单地使用每个主要版本的最新版本。
:::

## 1. 设置

Node.js 有入门模板。
使用 "create-hono" 命令开始你的项目。
在这个示例中选择 `nodejs` 模板。

::: code-group

```sh [npm]
npm create hono@latest my-app
```

```sh [yarn]
yarn create hono my-app
```

```sh [pnpm]
pnpm create hono my-app
```

```sh [bun]
bun create hono@latest my-app
```

```sh [deno]
deno init --npm hono my-app
```

:::

移动到 `my-app` 并安装依赖。

::: code-group

```sh [npm]
cd my-app
npm i
```

```sh [yarn]
cd my-app
yarn
```

```sh [pnpm]
cd my-app
pnpm i
```

```sh [bun]
cd my-app
bun i
```

:::

## 2. Hello World

编辑 `src/index.ts`：

```ts
import { serve } from '@hono/node-server'
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Node.js!'))

serve(app)
```

如果你想优雅地关闭服务器，这样写：

```ts
const server = serve(app)

// 优雅关闭
process.on('SIGINT', () => {
  server.close()
  process.exit(0)
})
process.on('SIGTERM', () => {
  server.close((err) => {
    if (err) {
      console.error(err)
      process.exit(1)
    }
    process.exit(0)
  })
})
```

## 3. 运行

在本地运行开发服务器。然后，在浏览器中访问 `http://localhost:3000`。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm dev
```

:::

## 更改端口号

你可以使用 `port` 选项指定端口号。

```ts
serve({
  fetch: app.fetch,
  port: 8787,
})
```

## 访问原始 Node.js API

你可以从 `c.env.incoming` 和 `c.env.outgoing` 访问 Node.js API。

```ts
import { Hono } from 'hono'
import { serve, type HttpBindings } from '@hono/node-server'
// 如果使用 HTTP2，则为 `Http2Bindings`

type Bindings = HttpBindings & {
  /* ... */
}

const app = new Hono<{ Bindings: Bindings }>()

app.get('/', (c) => {
  return c.json({
    remoteAddress: c.env.incoming.socket.remoteAddress,
  })
})

serve(app)
```

## 提供静态文件

你可以使用 `serveStatic` 从本地文件系统提供静态文件。例如，假设目录结构如下：

```sh
./
├── favicon.ico
├── index.ts
└── static
    ├── hello.txt
    └── image.png
```

如果请求路径 `/static/*` 并且你想返回 `./static` 下的文件，可以这样写：

```ts
import { serveStatic } from '@hono/node-server/serve-static'

app.use('/static/*', serveStatic({ root: './' }))
```

::: warning
`root` 选项相对于当前工作目录（`process.cwd()`）解析路径。这意味着行为取决于**你从何处运行 Node.js 进程**，而不是源文件的位置。如果你从不同的目录启动服务器，文件解析可能会失败。

对于始终指向源文件所在目录的可靠路径解析，使用 `import.meta.url`：

```ts
import { fileURLToPath } from 'node:url'
import { serveStatic } from '@hono/node-server/serve-static'

app.use(
  '/static/*',
  serveStatic({ root: fileURLToPath(new URL('./', import.meta.url)) })
)
```

:::

使用 `path` 选项提供目录根目录中的 `favicon.ico`：

```ts
app.use('/favicon.ico', serveStatic({ path: './favicon.ico' }))
```

如果请求路径 `/hello.txt` 或 `/image.png` 并且你想返回名为 `./static/hello.txt` 或 `./static/image.png` 的文件，可以使用以下方法：

```ts
app.use('*', serveStatic({ root: './static' }))
```

### `rewriteRequestPath`

如果你想将 `http://localhost:3000/static/*` 映射到 `./statics`，可以使用 `rewriteRequestPath` 选项：

```ts
app.get(
  '/static/*',
  serveStatic({
    root: './',
    rewriteRequestPath: (path) =>
      path.replace(/^\/static/, '/statics'),
  })
)
```

## http2

你可以在 [Node.js http2 服务器](https://nodejs.org/api/http2.html) 上运行 Hono。

### 未加密的 http2

```ts
import { createServer } from 'node:http2'

const server = serve({
  fetch: app.fetch,
  createServer,
})
```

### 加密的 http2

```ts
import { createSecureServer } from 'node:http2'
import { readFileSync } from 'node:fs'

const server = serve({
  fetch: app.fetch,
  createServer: createSecureServer,
  serverOptions: {
    key: readFileSync('localhost-privkey.pem'),
    cert: readFileSync('localhost-cert.pem'),
  },
})
```

## 构建和部署

::: code-group

```sh [npm]
npm run build
```

```sh [yarn]
yarn run build
```

```sh [pnpm]
pnpm run build
```

```sh [bun]
bun run build
```

::: info
带有前端框架的应用可能需要使用 [Hono 的 Vite 插件](https://github.com/honojs/vite-plugins)。
:::

### Dockerfile

以下是 Node.js Dockerfile 的示例。

```Dockerfile
FROM node:22-alpine AS base

FROM base AS builder

RUN apk add --no-cache gcompat
WORKDIR /app

COPY package*json tsconfig.json src ./

RUN npm ci && \
    npm run build && \
    npm prune --production

FROM base AS runner
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 hono

COPY --from=builder --chown=hono:nodejs /app/node_modules /app/node_modules
COPY --from=builder --chown=hono:nodejs /app/dist /app/dist
COPY --from=builder --chown=hono:nodejs /app/package.json /app/package.json

USER hono
EXPOSE 3000

CMD ["node", "/app/dist/index.js"]
```
