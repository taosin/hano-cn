# Web Standards

Hono 仅使用 **Web Standards**，如 Fetch。
它们最初用于 `fetch` 函数，由处理 HTTP 请求和响应的基本对象组成。
除了 `Requests` 和 `Responses` 之外，还有 `URL`、`URLSearchParam`、`Headers` 等。

Cloudflare Workers、Deno 和 Bun 也基于 Web Standards 构建。
例如，返回 "Hello World" 的服务器可以如下编写。这可以在 Cloudflare Workers 和 Bun 上运行。

```ts twoslash
export default {
  async fetch() {
    return new Response('Hello World')
  },
}
```

Hono 仅使用 Web Standards，这意味着 Hono 可以在支持它们的任何运行时上运行。
此外，我们有一个 Node.js 适配器。Hono 可以在这些运行时上运行：

- Cloudflare Workers (`workerd`)
- Deno
- Bun
- Fastly Compute
- AWS Lambda
- Node.js
- Vercel (edge-light)
- WebAssembly（通过 [`wasi:http`][wasi-http] 使用 [WebAssembly System Interface (WASI)][wasi]）

它也可以在 Netlify 和其他平台上运行。
相同的代码在所有平台上都能运行。

Cloudflare Workers、Deno、Shopify 等发起了 [WinterCG](https://wintercg.org) 来讨论使用 Web Standards 实现 "web-interoperability" 的可能性。
Hono 将跟随他们的步伐，致力于成为 **Web Standards 的标准**。

[wasi]: https://github.com/WebAssembly/wasi
[wasi-http]: https://github.com/WebAssembly/wasi-http
