# Frequently Asked Questions

本指南是关于 Hono 的常见问题解答（FAQ）以及如何解决它们。

## Is there an official Renovate config for Hono?

Hono 团队目前不维护 [Renovate](https://github.com/renovatebot/renovate) 配置。因此，请使用如下第三方 renovate-config。

在你的 `renovate.json` 中：

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>shinGangan/renovate-config-hono" // [!code ++]
  ]
}
```

查看 [renovate-config-hono](https://github.com/shinGangan/renovate-config-hono) 仓库了解更多详情。
