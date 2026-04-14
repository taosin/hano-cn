# Create-hono

`create-hono` 支持的命令行选项 - 这是运行 `npm create hono@latest`、`npx create-hono@latest` 或 `pnpm create hono@latest` 时运行的项目初始化工具。

> [!NOTE]
> **为什么有这个页面？** 安装/快速入门示例通常显示最小的 `npm create hono@latest my-app` 命令。`create-hono` 支持一些有用的标志，你可以传递这些标志来自动化和自定义项目创建（选择 templates、跳过提示、选择包管理器、使用本地缓存等）。

## Passing arguments:

当你使用 `npm create`（或 `npx`）时，用于初始化脚本的参数必须放在 `--` 之后。`--` 之后的任何内容都会转发给初始化脚本。

::: code-group

```sh [npm]
# 转发参数到 create-hono（npm 需要 `--`）
npm create hono@latest my-app -- --template cloudflare-workers
```

```sh [yarn]
# "--template cloudflare-workers" 选择 Cloudflare Workers template
yarn create hono my-app --template cloudflare-workers
```

```sh [pnpm]
# "--template cloudflare-workers" 选择 Cloudflare Workers template
pnpm create hono@latest my-app --template cloudflare-workers
```

```sh [bun]
# "--template cloudflare-workers" 选择 Cloudflare Workers template
bun create hono@latest my-app --template cloudflare-workers
```

```sh [deno]
# "--template cloudflare-workers" 选择 Cloudflare Workers template
deno init --npm hono@latest my-app --template cloudflare-workers
```

:::

## Commonly used arguments

| Argument                | Description                                                                                                                                      | Example                         |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------ |
| `--template <template>` | 选择 starter template 并跳过交互式 template 提示。Templates 可能包括名称如 `bun`、`cloudflare-workers`、`vercel` 等。 | `--template cloudflare-workers` |
| `--install`             | 在创建 template 后自动安装依赖项。                                                                                | `--install`                     |
| `--pm <packageManager>` | 指定安装依赖项时运行哪个包管理器。常用值：`npm`、`pnpm`、`yarn`。                                         | `--pm pnpm`                     |
| `--offline`             | 使用本地缓存/templates 而不是获取最新的远程 templates。适用于离线环境或确定性本地运行。      | `--offline`                     |

> [!NOTE]
> 确切的 templates 集和可用选项由 `create-hono` 项目维护。此文档页面总结了最常用的标志 - 查看下面链接的仓库获取完整、权威的参考。

## Example flows

### Minimal, interactive

```bash
npm create hono@latest my-app
```

这会提示你选择 template 和选项。

### Non-interactive, pick template and package manager

```bash
npm create hono@latest my-app -- --template vercel --pm npm --install
```

这使用 `vercel` template 创建 `my-app`，使用 `npm` 安装依赖项，并跳过交互式提示。

### Use offline cache (no network)

```bash
pnpm create hono@latest my-app --template deno --offline
```

## Troubleshooting & tips

- 如果某个选项似乎未被识别，请确保在使用 `npm create` / `npx` 时使用 `--` 转发它。
- 要查看最新的 templates 和标志列表，请咨询 `create-hono` 仓库或在本地运行初始化脚本并遵循其帮助输出。

## Links & references

- `create-hono` 仓库：[create-hono](https://github.com/honojs/create-hono)
