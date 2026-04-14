# Bun

[Bun](https://bun.com) 是另一个 JavaScript 运行时。它不是 Node.js 或 Deno。Bun 包含转译器，我们可以用 TypeScript 编写代码。
Hono 也可以在 Bun 上运行。

## 1. 安装 Bun

要安装 `bun` 命令，请遵循 [官方网站](https://bun.com) 中的说明。

## 2. 设置

### 2.1. 设置新项目

Bun 有入门模板。使用 "bun create" 命令开始你的项目。
在这个示例中选择 `bun` 模板。

```sh
bun create hono@latest my-app
```

进入 my-app 并安装依赖。

```sh
cd my-app
bun install
```

### 2.2. 设置现有项目

在现有 Bun 项目中，我们只需要在项目根目录通过以下命令安装 `hono` 依赖：

```sh
bun add hono
```

然后将 `dev` 命令添加到现有的 `package.json` 中。

```json
{
  "scripts": {
    "dev": "bun run --hot src/index.ts"
  }
}
```

查看 [Bun 入门模板](https://github.com/honojs/starter/tree/main/templates/bun) 了解最小示例设置。这是运行 `bun create hono@latest` 的输出。

## 3. Hello World

"Hello World" 脚本如下。几乎与在其他平台上编写相同。

```ts
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Bun!'))

export default app
```

如果你在现有项目上设置 Hono，`bun run dev` 命令期望 "Hello World" 脚本放在 `src/index.ts` 中。

## 4. 运行

运行命令。

```sh
bun run dev
```

然后，在浏览器中访问 `http://localhost:3000`。

## 更改端口号

你可以通过导出 `port` 来指定端口号。

<!-- prettier-ignore -->
```ts
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Bun!'))

export default app // [!code --]
export default { // [!code ++]
  port: 3000, // [!code ++]
  fetch: app.fetch, // [!code ++]
} // [!code ++]
```

## 提供静态文件

要提供静态文件，使用从 `hono/bun` 导入的 `serveStatic`。

```ts
import { serveStatic } from 'hono/bun'

const app = new Hono()

app.use('/static/*', serveStatic({ root: './' }))
app.use('/favicon.ico', serveStatic({ path: './favicon.ico' }))
app.get('/', (c) => c.text('You can access: /static/hello.txt'))
app.get('*', serveStatic({ path: './static/fallback.txt' }))
```

对于上面的代码，它将适用于以下目录结构。

```
./
├── favicon.ico
├── src
└── static
    ├── demo
    │   └── index.html
    ├── fallback.txt
    ├── hello.txt
    └── images
        └── dinotocat.png
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

## 测试

你可以在 Bun 上使用 `bun:test` 进行测试。

```ts
import { describe, expect, it } from 'bun:test'
import app from '.'

describe('我的第一个测试', () => {
  it('应该返回 200 响应', async () => {
    const req = new Request('http://localhost/')
    const res = await app.fetch(req)
    expect(res.status).toBe(200)
  })
})
```

然后，运行命令。

```sh
bun test index.test.ts
```
