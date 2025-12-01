`<meta>` 是 HTML 的“幕后工作者”：它**不显示在页面上**，却告诉浏览器、搜索引擎、社交平台、移动设备等“这份文档到底是什么脾气”。  
也是**唯一一个必须放在 `<head>` 里的空元素**（void element）。

---

### 1. 语法速记
```html
<meta charset="utf-8">          <!-- 必须放第一梯队，防乱码 -->
<meta name="keywords" content="HTML, meta, 教程">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta property="og:title" content="Meta 标签完全指南">
```

- 没有闭合标签。  
- 全部放在 `<head> … </head>` 内。  
- HTML5 不再要求 `/>` 自封，但 XHTML 仍需。

---

### 2. 四大分类（按用途）

| 类别           | 典型属性               | 作用                                               |
| -------------- | ---------------------- | -------------------------------------------------- |
| **声明类**     | `charset`              | 文档编码，防乱码                                   |
| **HTTP 头类**  | `http-equiv`           | 模拟 HTTP 响应头（刷新、兼容模式、缓存）           |
| **SEO/内容类** | `name="…" content="…"` | 给搜索引擎、爬虫、浏览器看的元数据                 |
| **社交分享类** | `property="og:…"`      | Open Graph / Twitter Card，让链接被分享时带图+标题 |

---

### 3. 日常必写“七件套”
```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>页面标题</title>
  <meta name="description" content="150 字以内精准描述，搜索结果显示">
  <meta name="keywords" content="关键词, 用英文逗号分隔">
  <meta name="author" content="你的名字或公司">
</head>
```

---

### 4. 常见字段拆解

#### 4.1 响应式视口
```html
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">
```
- `width=device-width`：宽度=设备逻辑像素  
- `initial-scale=1`：初始不缩放  
- `user-scalable=no` 可选，禁止双指缩放（移动端 PWA 常用）

#### 4.2 禁止缓存 / 强刷新
```html
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Expires" content="0">
```

#### 4.3 自动跳转/刷新
```html
<meta http-equiv="refresh" content="5; url=https://example.com">
```
5 秒后跳到新地址；SEO 慎用，301 重定向更好。

#### 4.4 浏览器兼容
```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```
让 IE 使用最高内核，防止“兼容模式”降级。

#### 4.5 搜索引擎索引规则
```html
<meta name="robots" content="index,follow">   <!-- 默认，可省略 -->
<meta name="robots" content="noindex,nofollow"> <!-- 不收录、不传递权重 -->
```

#### 4.6 版权/生成器
```html
<meta name="copyright" content="© 2025 YourCompany">
<meta name="generator" content="WordPress 6.5"> <!-- 自动生成 -->
```

---

### 5. 社交分享：Open Graph & Twitter Card
```html
<!-- Open Graph (Facebook, LinkedIn, 钉钉, 飞书) -->
<meta property="og:type" content="article">
<meta property="og:title" content="Meta 标签完全指南">
<meta property="og:description" content="一篇搞懂所有 meta 用法">
<meta property="og:image" content="https://demo.com/og.jpg">
<meta property="og:url" content="https://demo.com/meta-guide">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@yourname">
```
只要这些标签存在，粘贴链接就会“带图+标题+描述”。

---

### 6. 移动应用/快捷方式
```html
<!-- iOS 添加到主屏 -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="短标题">

<!-- 主屏图标 -->
<link rel="apple-touch-icon" sizes="180x180" href="/icon.png">

<!-- 主题色（Chrome/Android） -->
<meta name="theme-color" content="#ff6600">
```

---

### 7. 安全与新兴提案
```html
<!-- 强制 HTTPS (仅当本身已支持 HTTPS) -->
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">

<!-- 控制 Referer 发送 -->
<meta name="referrer" content="no-referrer-when-downgrade">
```

---

### 8. 一句话总结
`<meta>` 就是**网页的身份证+说明书+社交名片+通行证**；  
charset、viewport、description 是三大“保命签”，任何页面都应该先写它们，再谈别的。