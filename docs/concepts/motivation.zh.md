# 理念

在本节中，我们讨论 Hono 的概念或理念。

## 动机

起初，我只是想在 Cloudflare Workers 上创建一个 Web 应用程序。
但是，没有能在 Cloudflare Workers 上运行的好框架。
所以，我开始构建 Hono。

我认为这是一个学习如何使用 Trie 树构建路由器的好机会。
然后一个朋友出现了，带来了超快的路由器 "RegExpRouter"。
我还有一个朋友创建了基本认证中间件。

仅使用 Web Standard API，我们让它在 Deno 和 Bun 上运行。当人们问"Bun 有 Express 吗？"时，我们可以回答："没有，但有 Hono"。
（虽然 Express 现在可以在 Bun 上运行。）

我们还有朋友制作 GraphQL 服务器、Firebase 认证和 Sentry 中间件。
此外，我们还有一个 Node.js 适配器。
一个生态系统应运而生。

换句话说，Hono 非常快，使很多事情成为可能，并且可以在任何地方运行。
我们可以想象 Hono 可能成为 **Web Standards 的标准**。
