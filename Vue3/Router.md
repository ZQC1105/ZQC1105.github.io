下面用一句话先给出**核心定义**，再拆成三部分**逐条解释** + **对应代码片段**，最后给一张**关系图**，让你一眼看懂它们如何协同工作。

--------------------------------------------------
一、一句话速记
--------------------------------------------------
「**动态路由**」解决“同一路由组件展示不同数据”的问题；  
「**路由守卫**」解决“能不能进/离开页面”的问题；  
「**编程式导航**」解决“代码里主动跳转到哪里”的问题；  
三者叠加，就能实现：  
**带参数跳转 → 权限校验 → 自动放行/拦截 → 页面渲染** 的完整链路。

--------------------------------------------------
二、逐条拆解
--------------------------------------------------
1. 动态路由（Dynamic Route Matching）  
   场景：/posts/1、/posts/2、/posts/999 都映射到同一个组件 `PostDetail.vue`。  
   写法：在 `path` 里加“冒号占位符”。  
   ```js
   // router/index.js
   {
     path: '/posts/:id',     // :id 就是动态段
     name: 'post-detail',
     component: () => import('@/views/PostDetail.vue')
   }
   ```
   取值：组件里用 `route.params.id`（或 `props: true` 直接当 prop 用）。  
   好处：不用为每篇文章手写一条路由。

2. 路由守卫（Navigation Guards）  
   场景：跳转到“后台管理页”前检查是否登录；离开编辑页前提示“保存草稿？”  
   分类：  
   - 全局前置守卫 `beforeEach`（最常用）  
   - 路由独享守卫 `beforeEnter`  
   - 组件内守卫 `onBeforeRouteLeave` / `onBeforeRouteUpdate`  
   示例：简单鉴权  
   ```js
   router.beforeEach((to, from, next) => {
     const isLogin = !!localStorage.getItem('token')
     if (to.name === 'admin' && !isLogin) next({ name: 'login' })
     else next()          // 必须调用 next() 放行
   })
   ```

3. 编程式导航（Programmatic Navigation）  
   场景：点击“提交”按钮后，根据返回结果再决定跳转到成功页或失败页；点返回图标。  
   写法：在 `<script setup>` 里拿到 `useRouter()` 实例，调用 `.push()`、`.replace()`、`.go()`。  
   ```js
   import { useRouter } from 'vue-router'
   const router = useRouter()

   function submit() {
     api.login(form).then(() => {
       router.push('/dashboard')     // 跳转并留下历史记录
       // router.replace('/dashboard')  // 替换当前历史记录，不能点后退返回
     })
   }
   ```

--------------------------------------------------
三、三者协同流程图（伪代码版）
--------------------------------------------------
```
┌-------------┐      ┌-------------┐      ┌-------------┐
|  列表页     | ---> | 路由守卫     | ---> |  详情页     |
| /posts      |      | beforeEach  |      | /posts/:id  |
| click 事件  |      | 检查权限    |      | 组件渲染    |
└-------------┘      └-----┬-------┘      └-------------┘
        |                  ▼
        |            next() / next(false)
        |           放行     拦截
        |
        ▼ 编程式导航
   router.push(`/posts/${id}`)
```

--------------------------------------------------
四、30 秒回顾
--------------------------------------------------
- **动态路由** = 路由模板化，`:id` 占位。  
- **路由守卫** = 门卫，拦截或放行，必须 `next()`。  
- **编程式导航** = 代码里手动 `router.push/replace`，想跳就跳。  

把这三招串起来，就能做“带参数跳转 + 登录拦截 + 跳转后回退”的完整 SPA 导航逻辑。

其实已经“体现”了，只是目前只做了**最简的日志打印**，看起来不像“守卫”。

```js
router.beforeEach((to, from, next) => {
  console.log(`[Guard] ${from.path} → ${to.path}`)
  next()
})
```

这段代码就是**全局前置守卫**，只是没加任何**拦截逻辑**，所以你觉得“像不存在”。

--------------------------------------------------
让守卫真正“干活”的 2 个常见示例

1. 登录拦截  
```js
router.beforeEach((to, from, next) => {
  const isLogin = !!localStorage.getItem('token')   // 简易令牌
  if (to.name === 'post-detail' && !isLogin) {
    alert('请先登录')
    next({ name: 'home' })          // 跳到首页
  } else {
    next()                          // 放行
  }
})
```

2. 离开编辑页时给出二次确认（组件内守卫）  
在 `PostDetail.vue` 再加一段：
```ts
import { onBeforeRouteLeave } from 'vue-router'

onBeforeRouteLeave((to, from, next) => {
  const answer = window.confirm('确定要离开当前文章吗？')
  answer ? next() : next(false)   // 取消导航
})
```

--------------------------------------------------
小结
- 你已经写了守卫，只是目前**只打日志不拦截**。  
- 把 `next()` 换成条件跳转或 `next(false)`，就能肉眼看到“守卫”生效。