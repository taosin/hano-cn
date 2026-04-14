# 路由器

路由器是 Hono 最重要的功能。

Hono 有五个路由器。

## RegExpRouter

**RegExpRouter** 是 JavaScript 世界中最快的路由器。

虽然这被称为 "RegExp"，但它不是使用 [path-to-regexp](https://github.com/pillarjs/path-to-regexp) 的类 Express 实现。
它们使用线性循环。
因此，将对所有路由执行正则表达式匹配，随着路由增多，性能会下降。

![](/images/router-linear.jpg)

Hono 的 RegExpRouter 将路由模式转换为"一个大型正则表达式"。
然后它可以通过一次性匹配获得结果。

![](/images/router-regexp.jpg)

这在大多数情况下比使用基于树的算法（如基数树）的方法更快。

然而，RegExpRouter 不支持所有路由模式，所以它通常与其他支持所有路由模式的路由器之一结合使用。

## TrieRouter

**TrieRouter** 是使用 Trie 树算法的路由器。
与 RegExpRouter 一样，它不使用线性循环。

![](/images/router-tree.jpg)

这个路由器没有 RegExpRouter 快，但比 Express 路由器快得多。
TrieRouter 支持所有模式。

## SmartRouter

**SmartRouter** 在使用多个路由器时很有用。它通过从注册的路由器推断来选择最佳路由器。
Hono 默认使用 SmartRouter、RegExpRouter 和 TrieRouter：

```ts
// 在 Hono 的核心内部。
readonly defaultRouter: Router = new SmartRouter({
  routers: [new RegExpRouter(), new TrieRouter()],
})
```

当应用程序启动时，SmartRouter 根据路由检测最快的路由器并继续使用它。

## LinearRouter

RegExpRouter 很快，但路由注册阶段可能稍慢。
因此，它不适合每次请求都初始化的环境。

**LinearRouter** 针对"一次性"情况进行了优化。
路由注册比 RegExpRouter 快得多，因为它使用线性方法添加路由而不编译字符串。

以下是基准测试结果之一，包括路由注册阶段。

```console
• GET /user/lookup/username/hey
----------------------------------------------------- -----------------------------
LinearRouter     1.82 µs/iter      (1.7 µs … 2.04 µs)   1.84 µs   2.04 µs   2.04 µs
MedleyRouter     4.44 µs/iter     (4.34 µs … 4.54 µs)   4.48 µs   4.54 µs   4.54 µs
FindMyWay       60.36 µs/iter      (45.5 µs … 1.9 ms)  59.88 µs  78.13 µs  82.92 µs
KoaTreeRouter    3.81 µs/iter     (3.73 µs … 3.87 µs)   3.84 µs   3.87 µs   3.87 µs
TrekRouter       5.84 µs/iter     (5.75 µs … 6.04 µs)   5.86 µs   6.04 µs   6.04 µs

summary for GET /user/lookup/username/hey
  LinearRouter
   2.1x faster than KoaTreeRouter
   2.45x faster than MedleyRouter
   3.21x faster than TrekRouter
   33.24x faster than FindMyWay
```

## PatternRouter

**PatternRouter** 是 Hono 路由器中最小的。

虽然 Hono 已经很紧凑，但如果你需要在资源有限的环境中使其更小，使用 PatternRouter。

仅使用 PatternRouter 的应用程序大小小于 15KB。

```console
$ npx wrangler deploy --minify ./src/index.ts
 ⛅️ wrangler 3.20.0
-------------------
Total Upload: 14.68 KiB / gzip: 5.38 KiB
```
