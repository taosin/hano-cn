# Client Components

`hono/jsx` 不仅支持服务器端，也支持客户端。这意味着可以创建在浏览器中运行的交互式 UI。我们称之为 Client Components 或 `hono/jsx/dom`。

它非常快且非常小。`hono/jsx/dom` 中的计数器程序使用 Brotli 压缩后仅为 2.8KB，而 React 为 47.8KB。

本节介绍 Client Components 特定的功能。

## Counter example

这是一个简单计数器的示例，与 React 中的代码相同。

```tsx
import { useState } from 'hono/jsx'
import { render } from 'hono/jsx/dom'

function Counter() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  )
}

function App() {
  return (
    <html>
      <body>
        <Counter />
      </body>
    </html>
  )
}

const root = document.getElementById('root')
render(<App />, root)
```

## `render()`

你可以使用 `render()` 在指定的 HTML 元素内插入 JSX 组件。

```tsx
render(<Component />, container)
```

你可以在此处查看完整示例代码：[Counter example](https://github.com/honojs/examples/tree/main/hono-vite-jsx)。

## Hooks compatible with React

hono/jsx/dom 具有与 React 兼容或部分兼容的 Hooks。你可以通过查看 [React documentation](https://react.dev/reference/react/hooks) 了解这些 APIs。

- `useState()`
- `useEffect()`
- `useRef()`
- `useCallback()`
- `use()`
- `startTransition()`
- `useTransition()`
- `useDeferredValue()`
- `useMemo()`
- `useLayoutEffect()`
- `useReducer()`
- `useDebugValue()`
- `createElement()`
- `memo()`
- `isValidElement()`
- `useId()`
- `createRef()`
- `forwardRef()`
- `useImperativeHandle()`
- `useSyncExternalStore()`
- `useInsertionEffect()`
- `useFormStatus()`
- `useActionState()`
- `useOptimistic()`

## `startViewTransition()` family

`startViewTransition()` 系列包含原始的 hooks 和函数，用于轻松处理 [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API)。以下是如何使用它们的示例。

### 1. An easiest example

你可以使用 `startViewTransition()` 简短地编写使用 `document.startViewTransition` 的过渡。

```tsx
import { useState, startViewTransition } from 'hono/jsx'
import { css, Style } from 'hono/css'

export default function App() {
  const [showLargeImage, setShowLargeImage] = useState(false)
  return (
    <>
      <Style />
      <button
        onClick={() =>
          startViewTransition(() =>
            setShowLargeImage((state) => !state)
          )
        }
      >
        Click!
      </button>
      <div>
        {!showLargeImage ? (
          <img src='https://hono.dev/images/logo.png' />
        ) : (
          <div
            class={css`
              background: url('https://hono.dev/images/logo-large.png');
              background-size: contain;
              background-repeat: no-repeat;
              background-position: center;
              width: 600px;
              height: 600px;
            `}
          ></div>
        )}
      </div>
    </>
  )
}
```

### 2. Using `viewTransition()` with `keyframes()`

`viewTransition()` 函数允许你获取唯一的 `view-transition-name`。

你可以将其与 `keyframes()` 一起使用，`::view-transition-old()` 转换为 `::view-transition-old(${uniqueName))`。

```tsx
import { useState, startViewTransition } from 'hono/jsx'
import { viewTransition } from 'hono/jsx/dom/css'
import { css, keyframes, Style } from 'hono/css'

const rotate = keyframes`
  from {
    rotate: 0deg;
  }
  to {
    rotate: 360deg;
  }
`

export default function App() {
  const [showLargeImage, setShowLargeImage] = useState(false)
  const [transitionNameClass] = useState(() =>
    viewTransition(css`
      ::view-transition-old() {
        animation-name: ${rotate};
      }
      ::view-transition-new() {
        animation-name: ${rotate};
      }
    `)
  )
  return (
    <>
      <Style />
      <button
        onClick={() =>
          startViewTransition(() =>
            setShowLargeImage((state) => !state)
          )
        }
      >
        Click!
      </button>
      <div>
        {!showLargeImage ? (
          <img src='https://hono.dev/images/logo.png' />
        ) : (
          <div
            class={css`
              ${transitionNameClass}
              background: url('https://hono.dev/images/logo-large.png');
              background-size: contain;
              background-repeat: no-repeat;
              background-position: center;
              width: 600px;
              height: 600px;
            `}
          ></div>
        )}
      </div>
    </>
  )
}
```

### 3. Using `useViewTransition`

如果你想在动画期间仅更改样式。你可以使用 `useViewTransition()`。这个 hook 返回 `[boolean, (callback: () => void) => void]`，它们是 `isUpdating` 标志和 `startViewTransition()` 函数。

当使用这个 hook 时，Component 会在以下两次进行评估。

- 在调用 `startViewTransition()` 的回调内。
- 当 [the `finish` promise becomes fulfilled](https://developer.mozilla.org/en-US/docs/Web/API/ViewTransition/finished)

```tsx
import { useState, useViewTransition } from 'hono/jsx'
import { viewTransition } from 'hono/jsx/dom/css'
import { css, keyframes, Style } from 'hono/css'

const rotate = keyframes`
  from {
    rotate: 0deg;
  }
  to {
    rotate: 360deg;
  }
`

export default function App() {
  const [isUpdating, startViewTransition] = useViewTransition()
  const [showLargeImage, setShowLargeImage] = useState(false)
  const [transitionNameClass] = useState(() =>
    viewTransition(css`
      ::view-transition-old() {
        animation-name: ${rotate};
      }
      ::view-transition-new() {
        animation-name: ${rotate};
      }
    `)
  )
  return (
    <>
      <Style />
      <button
        onClick={() =>
          startViewTransition(() =>
            setShowLargeImage((state) => !state)
          )
        }
      >
        Click!
      </button>
      <div>
        {!showLargeImage ? (
          <img src='https://hono.dev/images/logo.png' />
        ) : (
          <div
            class={css`
              ${transitionNameClass}
              background: url('https://hono.dev/images/logo-large.png');
              background-size: contain;
              background-repeat: no-repeat;
              background-position: center;
              width: 600px;
              height: 600px;
              position: relative;
              ${isUpdating &&
              css`
                &:before {
                  content: 'Loading...';
                  position: absolute;
                  top: 50%;
                  left: 50%;
                }
              `}
            `}
          ></div>
        )}
      </div>
    </>
  )
}
```

## The `hono/jsx/dom` runtime

有一个用于 Client Components 的小型 JSX Runtime。使用这将比使用 `hono/jsx` 产生更小的捆绑结果。在 `tsconfig.json` 中指定 `hono/jsx/dom`。对于 Deno，修改 deno.json。

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx/dom"
  }
}
```

或者，你可以在 `vite.config.ts` 中的 esbuild 转换选项中指定 `hono/jsx/dom`。

```ts
import { defineConfig } from 'vite'

export default defineConfig({
  esbuild: {
    jsxImportSource: 'hono/jsx/dom',
  },
})
```
