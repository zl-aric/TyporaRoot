# 1️⃣ Blazor 托管模型（Hosting Model / Deployment Model）

Blazor 的托管模型指的是 **应用程序运行在哪里、由谁提供执行环境**，核心就是 **组件执行位置**：

| 模型                                | 执行位置 | 特点                                                         | 是否需要服务器持续连接               |
| ----------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------ |
| **Blazor Server**                   | 服务器端 | 组件生命周期、事件、RenderTree diff 全部在服务器执行；客户端通过 SignalR 接收 DOM 更新 | ✔ 需要 SignalR 持续连接              |
| **Blazor WebAssembly (WASM)**       | 浏览器端 | 组件和渲染全部在浏览器 WASM 执行；服务器只提供静态文件       | ❌ 不依赖 SignalR（除非应用额外通信） |
| **Blazor Hybrid (MAUI / Electron)** | 本地应用 | Blazor UI 运行在本地 WebView 中，组件执行在本地              | ❌ 不依赖服务器（除非后端通信）       |

所以：

- **Blazor Server** = 托管在服务器
- **Blazor WebAssembly** = 托管在客户端浏览器

------

# 2️⃣ Blazor 渲染模式（Rendering Mode）

渲染模式指 **页面 HTML 的生成和交互方式**，跟托管模型不同。它决定 **首屏 SSR / SPA / Streaming / Auto**：

| 模式            | 托管模型              | 首屏渲染             | 后续交互              | WASM                  |
| --------------- | --------------------- | -------------------- | --------------------- | --------------------- |
| Static SSR      | 可以是 Server 或 WASM | 服务器 SSR 输出 HTML | ❌ 无交互              | ❌                     |
| Streaming SSR   | 服务器                | 服务器分块输出 HTML  | ❌ 无交互              | ❌                     |
| Interactive SSR | WASM / Server         | 服务器 SSR 首屏      | ✔ 交互在浏览器 WASM   | ✔ 激活时加载轻量 WASM |
| WASM SPA        | WASM                  | 浏览器               | ✔ 全部浏览器执行      | ✔ 全部浏览器 WASM     |
| Auto            | WASM / Server         | SSR 首屏             | ✔ 浏览器按需激活 WASM | ✔ 按需 WASM           |

------

# 3️⃣ 小结

1. **托管模型** = 组件执行位置
   - Blazor Server → 服务器执行
   - Blazor WebAssembly → 浏览器执行
2. **渲染模式** = HTML 首屏渲染 + 后续交互策略
   - SSR / Streaming SSR / Interactive SSR / WASM SPA / Auto

> 换句话说，**托管模型告诉你“代码在哪跑”，渲染模式告诉你“页面怎么生成、交互怎么处理”**