---
title: Hono - 基于 Web Standards 构建的 Web 框架
titleTemplate: ':title'
head:
  - [
      'meta',
      {
        property: 'og:description',
        content: 'Hono 是一个小型、简单且超快的 Web 框架，基于 Web Standards 构建。它可以在 Cloudflare Workers、Fastly Compute、Deno、Bun、Vercel、Netlify、AWS Lambda、Lambda@Edge 和 Node.js 上运行。快速，但不止于快速。',
      },
    ]
layout: home
hero:
  name: Hono
  text: Web 应用程序框架
  tagline: 快速、轻量、基于 Web Standards 构建。支持任何 JavaScript 运行时。
  image:
    src: /images/code.webp
    alt: "Hono 代码示例。\
      import { Hono } from 'hono' \
      const app = new Hono() \
      app.get('/', (c) => c.text('Hello Hono!')) \

      export default app"
  actions:
    - theme: brand
      text: 开始使用
      link: /docs/
    - theme: alt
      text: 在 GitHub 上查看
      link: https://github.com/honojs/hono
features:
  - icon: 🚀
    title: 超快且轻量
    details: RegExpRouter 路由器非常快。hono/tiny 预设小于 14kB。仅使用 Web Standard APIs。
  - icon: 🌍
    title: 多运行时
    details: 可在 Cloudflare、Fastly、Deno、Bun、AWS 或 Node.js 上运行。相同的代码在所有平台上都能运行。
  - icon: 🔋
    title: 功能齐全
    details: Hono 内置了中间件、自定义中间件、第三方中间件和辅助函数。功能齐全。
  - icon: 😃
    title: 愉悦的开发体验
    details: 超干净的 API。一流的 TypeScript 支持。现在，我们有了"类型"。
---

<script setup>
// 灵感 heavily 来自 React
// https://github.com/reactjs/react.dev/pull/6817
import { onMounted } from 'vue'
onMounted(() => {
  var preferredKawaii
  try {
    preferredKawaii = localStorage.getItem('kawaii')
  } catch (err) {}
  const urlParams = new URLSearchParams(window.location.search)
  const kawaii = urlParams.get('kawaii')
  const setKawaii = () => {
    const images = document.querySelectorAll('.VPImage.image-src')
    images.forEach((img) => {
      img.src = '/images/hono-kawaii.png'
      img.alt = 'Hono 标志的可爱版本。第一个"o"被火焰替换，右下角有日文字符，火焰上方有 JSX 片段闭合标签。'
      img.classList.add("kawaii")
    })
  }
  if (kawaii === 'true') {
    try {
      localStorage.setItem('kawaii', true)
    } catch (err) {}
    console.log('kawaii 模式已启用。标志感谢 @sawaratsuki1004 via https://github.com/SAWARATSUKI/KawaiiLogos');
    setKawaii()
  } else if (kawaii === 'false') {
    try {
      localStorage.removeItem('kawaii', false)
    } catch (err) {}
    const images = document.querySelectorAll('.VPImage.image-src')
    images.forEach((img) => {
      img.src = '/images/code.webp'
      img.classList.remove("kawaii")
    })
  } else if (preferredKawaii) {
    setKawaii()
  }
})
</script>
