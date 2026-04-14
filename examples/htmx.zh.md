# htmx

使用 Hono 与 [htmx](https://htmx.org/)。

## typed-htmx

通过使用 [typed-htmx](https://github.com/Desdaemon/typed-htmx)，你可以使用 htmx 属性的 TypeScript 定义编写 JSX。
我们可以遵循 [typed-htmx Example Project](https://github.com/Desdaemon/typed-htmx/blob/main/example/src/types.d.ts) 中找到的相同模式来与 `hono/jsx` 一起使用。

安装包：

```sh
npm i -D typed-htmx
```

在 `src/global.d.ts`（或如果你使用 HonoX 则为 `app/global.d.ts`）中，导入 `typed-htmx` 类型：

```ts
import 'typed-htmx'
```

使用 typed-htmx 定义扩展 Hono 的 JSX 类型：

```ts
// 演示如何使用 htmx 属性扩充外部类型。
// 在这种情况下，Hono 从其自己的命名空间获取类型，所以我们做同样的事情
// 并直接扩展其命名空间。
declare module 'hono/jsx' {
  namespace JSX {
    interface HTMLAttributes extends HtmxAttributes {}
  }
}
```

## 另请参阅

- [htmx](https://htmx.org/)
- [typed-htmx](https://github.com/Desdaemon/typed-htmx)
