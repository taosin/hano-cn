# Deno

[Deno](https://deno.com/) 是一个基于 V8 构建的 JavaScript 运行时。它不是 Node.js。
Hono 也可以在 Deno 上运行。

你可以使用 Hono，用 TypeScript 编写代码，使用 `deno` 命令运行应用程序，并将其部署到 "Deno Deploy"。

## 1. 安装 Deno

首先，安装 `deno` 命令。
请参考 [官方文档](https://docs.deno.com/runtime/getting_started/installation/)。

## 2. 设置

Deno 有入门模板。
使用 [`deno init`](https://docs.deno.com/runtime/reference/cli/init/) 命令开始你的项目。

```sh
deno init --npm hono --template=deno my-app
```

进入 `my-app`。对于 Deno，你不需要显式安装 Hono。

```sh
cd my-app
```

## 3. Hello World

编辑 `main.ts`：

```ts [main.ts]
import { Hono } from 'hono'

const app = new Hono()

app.get('/', (c) => c.text('Hello Deno!'))

Deno.serve(app.fetch)
```

## 4. 运行

在本地运行开发服务器。然后，在浏览器中访问 `http://localhost:8000`。

```sh
deno task start
```

## 更改端口号

你可以通过更新 `main.ts` 中 `Deno.serve` 的参数来指定端口号：

```ts
Deno.serve(app.fetch) // [!code --]
Deno.serve({ port: 8787 }, app.fetch) // [!code ++]
```

## 提供静态文件

要提供静态文件，使用从 `hono/deno` 导入的 `serveStatic`。

```ts
import { Hono } from 'hono'
import { serveStatic } from 'hono/deno'

const app = new Hono()

app.use('/static/*', serveStatic({ root: './' }))
app.use('/favicon.ico', serveStatic({ path: './favicon.ico' }))
app.get('/', (c) => c.text('You can access: /static/hello.txt'))
app.get('*', serveStatic({ path: './static/fallback.txt' }))

Deno.serve(app.fetch)
```

对于上面的代码，它将适用于以下目录结构。

```
./
├── favicon.ico
├── index.ts
└── static
    ├── demo
    │   └── index.html
    ├── fallback.txt
    ├── hello.txt
    └── images
        └── dinotocat.png
```

### `rewriteRequestPath`

如果你想将 `http://localhost:8000/static/*` 映射到 `./statics`，可以使用 `rewriteRequestPath` 选项：

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

### `mimes`

你可以使用 `mimes` 添加 MIME 类型：

```ts
app.get(
  '/static/*',
  serveStatic({
    mimes: {
      m3u8: 'application/vnd.apple.mpegurl',
      ts: 'video/mp2t',
    },
  })
)
```

### `onFound`

你可以使用 `onFound` 指定找到请求文件时的处理：

```ts
app.get(
  '/static/*',
  serveStatic({
    // ...
    onFound: (_path, c) => {
      c.header('Cache-Control', `public, immutable, max-age=31536000`)
    },
  })
)
```

### `onNotFound`

你可以使用 `onNotFound` 指定未找到请求文件时的处理：

```ts
app.get(
  '/static/*',
  serveStatic({
    onNotFound: (path, c) => {
      console.log(`${path} is not found, you access ${c.req.path}`)
    },
  })
)
```

### `precompressed`

`precompressed` 选项检查具有 `.br` 或 `.gz` 等扩展名的文件是否可用，并根据 `Accept-Encoding` 头部提供它们。它优先使用 Brotli，然后是 Zstd 和 Gzip。如果都不可用，则提供原始文件。

```ts
app.get(
  '/static/*',
  serveStatic({
    precompressed: true,
  })
)
```

## Deno Deploy

Deno Deploy 是一个用于在云中运行 JavaScript 和 TypeScript 应用程序的无服务器平台。
它通过 GitHub 部署等集成提供管理和运行应用程序的管理平面。

Hono 也可以在 Deno Deploy 上运行。请参考 [官方文档](https://docs.deno.com/deploy/manual/)。

## 测试

在 Deno 上测试应用程序很简单。
你可以使用 `Deno.test` 编写测试，并使用 [@std/assert](https://jsr.io/@std/assert) 中的 `assert` 或 `assertEquals`。

```sh
deno add jsr:@std/assert
```

```ts [hello.ts]
import { Hono } from 'hono'
import { assertEquals } from '@std/assert'

Deno.test('Hello World', async () => {
  const app = new Hono()
  app.get('/', (c) => c.text('Please test me'))

  const res = await app.request('http://localhost/')
  assertEquals(res.status, 200)
})
```

然后运行命令：

```sh
deno test hello.ts
```

## npm 和 JSR

Hono 在 [npm](https://www.npmjs.com/package/hono) 和 [JSR](https://jsr.io/@hono/hono)（JavaScript Registry）上都可用。你可以在 `deno.json` 中使用 `npm:hono` 或 `jsr:@hono/hono`：

```json
{
  "imports": {
    "hono": "jsr:@hono/hono" // [!code --]
    "hono": "npm:hono" // [!code ++]
  }
}
```

要使用中间件，你需要在导入中使用 [Deno 目录](https://docs.deno.com/runtime/fundamentals/configuration/#custom-path-mappings) 语法。

```json
{
  "imports": {
    "hono/": "npm:/hono/"
  }
}
```

使用第三方中间件时，为了正确的 TypeScript 类型推断，你可能需要使用与中间件相同注册表的 Hono。例如，如果使用 npm 中的中间件，你也应该使用 npm 中的 Hono：

```json
{
  "imports": {
    "hono": "npm:hono",
    "zod": "npm:zod",
    "@hono/zod-validator": "npm:@hono/zod-validator"
  }
}
```

我们还在 [JSR](https://jsr.io/@hono) 上提供许多第三方中间件包。使用 JSR 上的中间件时，使用 JSR 中的 Hono：

```json
{
  "imports": {
    "hono": "jsr:@hono/hono",
    "zod": "npm:zod",
    "@hono/zod-validator": "jsr:@hono/zod-validator"
  }
}
```
