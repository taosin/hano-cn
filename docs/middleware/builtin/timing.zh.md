# Server-Timing 中间件

[Server-Timing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing) 中间件在响应 headers 中提供性能指标。

::: info
注意：在 Cloudflare Workers 上，计时器指标可能不准确，因为 [计时器仅显示上次 I/O 的时间](https://developers.cloudflare.com/workers/learning/security-model/#step-1-disallow-timers-and-multi-threading)。
:::

## 导入

```ts [npm]
import { Hono } from 'hono'
import {
  timing,
  setMetric,
  startTime,
  endTime,
  wrapTime,
} from 'hono/timing'
import type { TimingVariables } from 'hono/timing'
```

## 用法

```js
// 指定变量类型以推断 `c.get('metric')`：
type Variables = TimingVariables

const app = new Hono<{ Variables: Variables }>()

// 将中间件添加到你的路由
app.use(timing());

app.get('/', async (c) => {

  // 添加自定义指标
  setMetric(c, 'region', 'europe-west3')

  // 添加带有计时的自定义指标，必须以毫秒为单位
  setMetric(c, 'custom', 23.8, 'My custom Metric')

  // 启动新计时器
  startTime(c, 'db');
  const data = await db.findMany(...);

  // 结束计时器
  endTime(c, 'db');

  // ...或者你也可以使用此函数包装 Promise：
  const data = await wrapTime(c, 'db', db.findMany(...));

  return c.json({ response: data });
});
```

### 有条件启用

```ts
const app = new Hono()

app.use(
  '*',
  timing({
    // c: 请求的 Context
    enabled: (c) => c.req.method === 'POST',
  })
)
```

## 结果

![](/images/timing-example.png)

## 选项

### <Badge type="info" text="optional" /> total: `boolean`

显示总响应时间。默认值为 `true`。

### <Badge type="info" text="optional" /> enabled: `boolean` | `(c: Context) => boolean`

是否应将计时添加到 headers。默认值为 `true`。

### <Badge type="info" text="optional" /> totalDescription: `boolean`

总响应时间的描述。默认值为 `Total Response Time`。

### <Badge type="info" text="optional" /> autoEnd: `boolean`

`startTime()` 是否应在请求结束时自动结束。
如果禁用，未手动结束的计时器将不会显示。

### <Badge type="info" text="optional" /> crossOrigin: `boolean` | `string` | `(c: Context) => boolean | string`

此计时 header 应该可读的来源。

- 如果为 false，仅来自当前来源。
- 如果为 true，来自所有来源。
- 如果为字符串，来自此域。多个域必须用逗号分隔。

默认值为 `false`。请参阅更多 [文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)。
