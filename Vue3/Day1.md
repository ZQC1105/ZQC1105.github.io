
# 从 ASP.NET Core MVC 到 Vue 3：核心概念对照速查

> 用你熟悉的后端思维，10 分钟看懂 Vue 基础：实例、模板、插值、指令。

---

## 1. Vue 实例（Vue Instance）

| Vue 3（组合式 API）            | ASP.NET Core 类比           |
| ------------------------------ | --------------------------- |
| `createApp(App).mount('#app')` | `builder.Build().Run()`     |
| `App.vue`（根组件）            | `_Layout.cshtml` + 首页内容 |
| `#app`（挂载点）               | `<body>` 容器               |

---

## 2. 模板语法（Template Syntax）

| 场景     | Razor                            | Vue 模板                   |
| -------- | -------------------------------- | -------------------------- |
| 输出变量 | `@Model.Title`                   | `{{ title }}`              |
| 条件     | `@if (Model.Show)`               | `v-if="isVisible"`         |
| 循环     | `@foreach(var x in Model.Items)` | `v-for="item in items"`    |
| 表单     | `<input value="@Model.Name" />`  | `<input v-model="name" />` |

> Razor 服务端一次性渲染；Vue 浏览器内响应式更新。

---

## 3. 插值（Interpolation）

```vue
<p>计数：{{ count }}</p>
<p>双倍：{{ count * 2 }}</p>
<p>大写：{{ user?.name.toUpperCase() }}</p>
```

- 仅支持**表达式**，不支持 `if` / `for` 语句。
- 数据变 → 界面自动变（Razor 静态输出后不再变）。

---

## 4. 指令（Directives）

| 指令                   | 功能     | 类似 Razor           | 备注                      |
| ---------------------- | -------- | -------------------- | ------------------------- |
| `v-if` / `v-else`      | 条件渲染 | `@if`                | 真假直接决定 DOM 是否存在 |
| `v-for="item in list"` | 列表渲染 | `@foreach`           | 必须加 `:key="item.id"`   |
| `v-model="message"`    | 双向绑定 | `asp-for` + 手动提交 | 无刷新实时同步            |

---

## 5. 核心对照表

| 功能     | ASP.NET Core MVC (Razor)     | Vue 3                       |
| -------- | ---------------------------- | --------------------------- |
| 页面结构 | `.cshtml` + `_Layout.cshtml` | `.vue` 单文件组件           |
| 数据输出 | `@Model.Xxx`                 | `{{ xxx }}`                 |
| 条件渲染 | `@if (...) { ... }`          | `v-if="..."`                |
| 循环渲染 | `@foreach (...) { ... }`     | `v-for="item in list"`      |
| 表单绑定 | `asp-for="Name"` + 后端接收  | `v-model="name"`            |
| 状态更新 | 提交 → 刷新页面              | 修改 `ref()` → 视图自动更新 |
| 运行位置 | 服务端                       | 浏览器（客户端）            |

---

## ✅ 下一步学习路线

1. 后端：`dotnet new webapi` 创建 Todo API。
2. 前端：`npm create vue@latest` 创建 Vue 3 项目（启用组合式 API）。
3. 联调：
   - Vue 中用 `fetch` / `axios` 调用 API；
   - `v-for` 渲染列表；
   - `v-model` 绑定输入框，POST 新增数据。

> 把 Vue 想成“浏览器里的前端 MVC”，ASP.NET Core 专心做 API，两者通过 JSON 通信，就是现代 SPA 的标准姿势。
```