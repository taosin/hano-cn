# 验证器中的错误处理

通过使用验证器，你可以更轻松地处理无效输入。本示例展示了你可以利用回调结果实现自定义错误处理。

虽然此代码片段使用了 [Zod Validator](https://github.com/honojs/middleware/blob/main/packages/zod-validator)，但你可以对任何支持的验证器库应用类似的方法。

```ts
import * as z from 'zod'
import { zValidator } from '@hono/zod-validator'

const app = new Hono()

const userSchema = z.object({
  name: z.string(),
  age: z.number(),
})

app.post(
  '/users/new',
  zValidator('json', userSchema, (result, c) => {
    if (!result.success) {
      return c.text('Invalid!', 400)
    }
  }),
  async (c) => {
    const user = c.req.valid('json')
    console.log(user.name) // string
    console.log(user.age) // number
  }
)
```

## 另请参阅

- [Zod Validator](https://github.com/honojs/middleware/blob/main/packages/zod-validator)
- [Valibot Validator](https://github.com/honojs/middleware/tree/main/packages/valibot-validator)
- [Typebox Validator](https://github.com/honojs/middleware/tree/main/packages/typebox-validator)
- [Typia Validator](https://github.com/honojs/middleware/tree/main/packages/typia-validator)
