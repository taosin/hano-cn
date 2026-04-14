# JSX

你可以使用 `hono/jsx` 用 JSX 语法编写 HTML。

虽然 `hono/jsx` 在客户端上工作，但你最常在服务器端渲染内容时使用它。以下是与 JSX 相关的一些对服务器和客户端都通用的事项。

## Settings

要使用 JSX，修改 `tsconfig.json`：

`tsconfig.json`：

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx"
  }
}
```

或者，使用 pragma directives：

```ts
/** @jsx jsx */
/** @jsxImportSource hono/jsx */
```

对于 Deno，你必须修改 `deno.json` 而不是 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "jsx": "precompile",
    "jsxImportSource": "@hono/hono/jsx"
  }
}
```

## Usage

:::info
如果你直接从 [Quick Start](/docs/#quick-start) 来，主文件的扩展名为 `.ts` - 你需要将其更改为 `.tsx` - 否则你将根本无法运行应用程序。你还应该修改 `package.json`（如果你使用 Deno 则为 `deno.json`）以反映该更改（例如，在 dev 脚本中不使用 `bun run --hot src/index.ts`，而应该使用 `bun run --hot src/index.tsx`）。
:::

`index.tsx`：

```tsx
import { Hono } from 'hono'
import type { FC } from 'hono/jsx'

const app = new Hono()

const Layout: FC = (props) => {
  return (
    <html>
      <body>{props.children}</body>
    </html>
  )
}

const Top: FC<{ messages: string[] }> = (props: {
  messages: string[]
}) => {
  return (
    <Layout>
      <h1>Hello Hono!</h1>
      <ul>
        {props.messages.map((message) => {
          return <li>{message}!!</li>
        })}
      </ul>
    </Layout>
  )
}

app.get('/', (c) => {
  const messages = ['Good Morning', 'Good Evening', 'Good Night']
  return c.html(<Top messages={messages} />)
})

export default app
```

## Metadata hoisting

你可以直接在组件内编写文档元数据标签，如 `<title>`、`<link>` 和 `<meta>`。这些标签将自动提升到文档的 `<head>` 部分。当 `<head>` 元素远离确定适当元数据的组件渲染时，这特别有用。

```tsx
import { Hono } from 'hono'

const app = new Hono()

app.use('*', async (c, next) => {
  c.setRenderer((content) => {
    return c.html(
      <html>
        <head></head>
        <body>{content}</body>
      </html>
    )
  })
  await next()
})

app.get('/about', (c) => {
  return c.render(
    <>
      <title>About Page</title>
      <meta name='description' content='This is the about page.' />
      about page content
    </>
  )
})

export default app
```

:::info
当发生提升时，现有元素不会被移除。后面出现的元素会添加到末尾。例如，如果你在 `<head>` 中有 `<title>Default</title>` 并且组件渲染 `<title>Page Title</title>`，两个标题都会出现在 head 中。
:::

## Fragment

使用 Fragment 将多个元素分组而不添加额外的节点：

```tsx
import { Fragment } from 'hono/jsx'

const List = () => (
  <Fragment>
    <p>first child</p>
    <p>second child</p>
    <p>third child</p>
  </Fragment>
)
```

或者，如果设置正确，你可以用 `<></>` 编写。

```tsx
const List = () => (
  <>
    <p>first child</p>
    <p>second child</p>
    <p>third child</p>
  </>
)
```

## `PropsWithChildren`

你可以使用 `PropsWithChildren` 在函数组件中正确推断子元素。

```tsx
import { PropsWithChildren } from 'hono/jsx'

type Post = {
  id: number
  title: string
}

function Component({ title, children }: PropsWithChildren<Post>) {
  return (
    <div>
      <h1>{title}</h1>
      {children}
    </div>
  )
}
```

## Inserting Raw HTML

要直接插入 HTML，使用 `dangerouslySetInnerHTML`：

```tsx
app.get('/foo', (c) => {
  const inner = { __html: 'JSX &middot; SSR' }
  const Div = <div dangerouslySetInnerHTML={inner} />
})
```

## Memoization

使用 `memo` 通过记忆计算的字符串来优化你的组件：

```tsx
import { memo } from 'hono/jsx'

const Header = memo(() => <header>Welcome to Hono</header>)
const Footer = memo(() => <footer>Powered by Hono</footer>)
const Layout = (
  <div>
    <Header />
    <p>Hono is cool!</p>
    <Footer />
  </div>
)
```

## Context

通过使用 `useContext`，你可以在 Component 树的任何级别全局共享数据，而无需通过 props 传递值。

```tsx
import type { FC } from 'hono/jsx'
import { createContext, useContext } from 'hono/jsx'

const themes = {
  light: {
    color: '#000000',
    background: '#eeeeee',
  },
  dark: {
    color: '#ffffff',
    background: '#222222',
  },
}

const ThemeContext = createContext(themes.light)

const Button: FC = () => {
  const theme = useContext(ThemeContext)
  return <button style={theme}>Push!</button>
}

const Toolbar: FC = () => {
  return (
    <div>
      <Button />
    </div>
  )
}

// ...

app.get('/', (c) => {
  return c.html(
    <div>
      <ThemeContext.Provider value={themes.dark}>
        <Toolbar />
      </ThemeContext.Provider>
    </div>
  )
})
```

## Async Component

`hono/jsx` 支持 Async Component，因此你可以在组件中使用 `async`/`await`。如果你使用 `c.html()` 渲染它，它将自动等待。

```tsx
const AsyncComponent = async () => {
  await new Promise((r) => setTimeout(r, 1000)) // sleep 1s
  return <div>Done!</div>
}

app.get('/', (c) => {
  return c.html(
    <html>
      <body>
        <AsyncComponent />
      </body>
    </html>
  )
})
```

## Suspense <Badge style="vertical-align: middle;" type="warning" text="Experimental" />

可以使用 React-like 的 `Suspense` 功能。如果你用 `Suspense` 包装 async component，fallback 中的内容将首先渲染，一旦 Promise 解析，将显示等待的内容。你可以将其与 `renderToReadableStream()` 一起使用。

```tsx
import { renderToReadableStream, Suspense } from 'hono/jsx/streaming'

//...

app.get('/', (c) => {
  const stream = renderToReadableStream(
    <html>
      <body>
        <Suspense fallback={<div>loading...</div>}>
          <Component />
        </Suspense>
      </body>
    </html>
  )
  return c.body(stream, {
    headers: {
      'Content-Type': 'text/html; charset=UTF-8',
      'Transfer-Encoding': 'chunked',
    },
  })
})
```

## ErrorBoundary <Badge style="vertical-align: middle;" type="warning" text="Experimental" />

你可以使用 `ErrorBoundary` 捕获子组件中的错误。

在下面的示例中，如果发生错误，它将显示在 `fallback` 中指定的内容。

```tsx
function SyncComponent() {
  throw new Error('Error')
  return <div>Hello</div>
}

app.get('/sync', async (c) => {
  return c.html(
    <html>
      <body>
        <ErrorBoundary fallback={<div>Out of Service</div>}>
          <SyncComponent />
        </ErrorBoundary>
      </body>
    </html>
  )
})
```

`ErrorBoundary` 也可以与 async components 和 `Suspense` 一起使用。

```tsx
async function AsyncComponent() {
  await new Promise((resolve) => setTimeout(resolve, 2000))
  throw new Error('Error')
  return <div>Hello</div>
}

app.get('/with-suspense', async (c) => {
  return c.html(
    <html>
      <body>
        <ErrorBoundary fallback={<div>Out of Service</div>}>
          <Suspense fallback={<div>Loading...</div>}>
            <AsyncComponent />
          </Suspense>
        </ErrorBoundary>
      </body>
    </html>
  )
})
```

## StreamingContext <Badge style="vertical-align: middle;" type="warning" text="Experimental" />

你可以使用 `StreamingContext` 为 `Suspense` 和 `ErrorBoundary` 等流式组件提供配置。这对于为这些组件生成的脚本标签添加 nonce 值以用于 Content Security Policy (CSP) 很有用。

```tsx
import { Suspense, StreamingContext } from 'hono/jsx/streaming'

// ...

app.get('/', (c) => {
  const stream = renderToReadableStream(
    <html>
      <body>
        <StreamingContext
          value={{ scriptNonce: 'random-nonce-value' }}
        >
          <Suspense fallback={<div>Loading...</div>}>
            <AsyncComponent />
          </Suspense>
        </StreamingContext>
      </body>
    </html>
  )

  return c.body(stream, {
    headers: {
      'Content-Type': 'text/html; charset=UTF-8',
      'Transfer-Encoding': 'chunked',
      'Content-Security-Policy':
        "script-src 'nonce-random-nonce-value'",
    },
  })
})
```

`scriptNonce` 值将自动添加到 `Suspense` 和 `ErrorBoundary` 组件生成的任何 `<script>` 标签中。

## Integration with html Middleware

结合 JSX 和 HTML middlewares 以实现强大的模板。有关详细信息，请咨询 [HTML middleware documentation](/docs/helpers/html)。

```tsx
import { Hono } from 'hono'
import { html } from 'hono/html'

const app = new Hono()

interface SiteData {
  title: string
  children?: any
}

const Layout = (props: SiteData) =>
  html`<!doctype html>
    <html>
      <head>
        <title>${props.title}</title>
      </head>
      <body>
        ${props.children}
      </body>
    </html>`

const Content = (props: { siteData: SiteData; name: string }) => (
  <Layout {...props.siteData}>
    <h1>Hello {props.name}</h1>
  </Layout>
)

app.get('/:name', (c) => {
  const { name } = c.req.param()
  const props = {
    name: name,
    siteData: {
      title: 'JSX with html sample',
    },
  }
  return c.html(<Content {...props} />)
})

export default app
```

## With JSX Renderer Middleware

[JSX Renderer Middleware](/docs/middleware/builtin/jsx-renderer) 允许你更轻松地使用 JSX 创建 HTML 页面。

## Override type definitions

你可以覆盖类型定义以添加自定义元素和属性。

```ts
declare module 'hono/jsx' {
  namespace JSX {
    interface IntrinsicElements {
      'my-custom-element': HTMLAttributes & {
        'x-event'?: 'click' | 'scroll'
      }
    }
  }
}
```
