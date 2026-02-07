# Vue 3 面试题与答案（50题）

## 核心概念与基础

### 1. Vue 3 的主要新特性有哪些？

**答案：**

- Composition API
- 响应式系统重构（使用Proxy）
- 性能优化（更好的Tree-shaking、更小的包体积）
- Fragments（支持多根节点组件）
- Teleport（瞬移组件）
- Suspense（异步组件）
- 自定义渲染器API
- 更好的TypeScript支持

### 2. Composition API 与 Options API 的区别？

**答案：**
Composition API 是 Vue 3 引入的新 API 风格，主要区别：

- **组织方式**：Composition API 按逻辑功能组织代码，Options API 按选项类型组织
- **复用性**：Composition API 更容易提取和复用逻辑（composables）
- **TypeScript支持**：Composition API 有更好的类型推断
- **灵活性**：Composition API 更灵活，适合复杂组件

### 3. 什么是响应式系统？Vue 3 如何实现响应式？

**答案：**
Vue 3 使用 Proxy 替代 Object.defineProperty 实现响应式：

```javascript
// Vue 3 响应式原理简例
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      track(target, key); // 收集依赖
      return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
      const result = Reflect.set(target, key, value, receiver);
      trigger(target, key); // 触发更新
      return result;
    },
  });
}
```

### 4. ref 和 reactive 的区别？

**答案：**

- **ref**：包装基本类型值，通过 `.value` 访问，模板中自动解包
- **reactive**：创建对象响应式代理，直接访问属性
- 使用场景：
  ```javascript
  const count = ref(0); // 基本类型
  const state = reactive({ count: 0, name: "Vue" }); // 对象
  ```

### 5. computed 和 watch 的区别？

**答案：**

- **computed**：计算属性，基于依赖缓存，只有依赖变化才重新计算
- **watch**：观察数据变化，执行副作用（异步操作、复杂逻辑）

```javascript
// computed
const fullName = computed(() => `${firstName.value} ${lastName.value}`);

// watch
watch(count, (newVal, oldVal) => {
  console.log(`count changed: ${oldVal} -> ${newVal}`);
});
```

## Composition API

### 6. setup 函数的作用是什么？

**答案：**
`setup` 是 Composition API 的入口函数：

- 在组件创建之前执行（beforeCreate、created之前）
- 接收 props 和 context 参数
- 返回对象或渲染函数
- **无法访问this**

### 7. 如何在 setup 中访问组件实例？

**答案：**
使用 `getCurrentInstance`，但不推荐在业务代码中使用：

```javascript
import { getCurrentInstance } from 'vue'

setup() {
  const instance = getCurrentInstance()
  console.log(instance.proxy) // 组件实例
}
```

### 8. 什么是 provide/inject？如何使用？

**答案：**
跨组件层级传递数据的 API：

```javascript
// 父组件
import { provide } from 'vue'
setup() {
  provide('theme', 'dark')
}

// 子组件
import { inject } from 'vue'
setup() {
  const theme = inject('theme', 'light') // 默认值'light'
}
```

### 9. 如何创建自定义 Composition 函数？

**答案：**
创建可复用的逻辑函数（composable）：

```javascript
// useCounter.js
import { ref } from "vue";

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);

  const increment = () => count.value++;
  const decrement = () => count.value--;
  const reset = () => (count.value = initialValue);

  return { count, increment, decrement, reset };
}

// 组件中使用
const { count, increment } = useCounter(10);
```

### 10. watch 和 watchEffect 的区别？

**答案：**

- **watch**：明确指定观察的数据源，提供新旧值
- **watchEffect**：自动追踪依赖，立即执行副作用函数

```javascript
// watch - 明确数据源
watch(count, (newVal) => console.log(newVal));

// watchEffect - 自动追踪
watchEffect(() => console.log(count.value + state.name));
```

## 响应式系统

### 11. Vue 3 响应式系统的原理？

**答案：**
Vue 3 使用 Proxy 实现响应式追踪：

1. **响应式追踪**：通过 Proxy getter 收集依赖（effect）
2. **依赖收集**：创建副作用函数与数据的关联
3. **触发更新**：通过 Proxy setter 通知依赖更新
4. **批量更新**：通过调度器批量执行更新

### 12. 什么是 shallowRef 和 shallowReactive？

**答案：**
浅层响应式，只追踪第一层属性：

```javascript
import { shallowRef, shallowReactive } from "vue";

// shallowRef - 不追踪 .value 内部变化
const shallowObj = shallowRef({ nested: { value: 1 } });
shallowObj.value.nested.value = 2; // 不会触发更新

// shallowReactive - 只追踪第一层属性
const shallowState = shallowReactive({
  nested: { value: 1 },
});
shallowState.nested.value = 2; // 不会触发更新
```

### 13. toRef 和 toRefs 的作用？

**答案：**
将 reactive 对象的属性转换为 ref：

```javascript
import { reactive, toRef, toRefs } from "vue";

const state = reactive({ x: 0, y: 0 });

// toRef - 单个属性转换
const xRef = toRef(state, "x");

// toRefs - 所有属性转换
const { x, y } = toRefs(state);
// x 和 y 现在是 ref，保持响应式连接
```

### 14. readonly 的作用是什么？

**答案：**
创建只读的响应式代理，防止意外修改：

```javascript
import { reactive, readonly } from "vue";

const original = reactive({ count: 0 });
const copy = readonly(original);

copy.count++; // 警告：无法修改只读属性
```

### 15. 自定义 ref 的实现？

**答案：**
使用 `customRef` 创建自定义 ref：

```javascript
import { customRef } from "vue";

function useDebouncedRef(value, delay = 200) {
  let timeout;
  return customRef((track, trigger) => {
    return {
      get() {
        track();
        return value;
      },
      set(newValue) {
        clearTimeout(timeout);
        timeout = setTimeout(() => {
          value = newValue;
          trigger();
        }, delay);
      },
    };
  });
}
```

## 组件系统

### 16. Vue 3 组件有哪些变化？

**答案：**

- 支持多根节点组件（Fragments）
- 异步组件定义方式改变
- 函数式组件变化
- emits 选项明确声明
- v-model 支持多个绑定
- 自定义指令生命周期钩子重命名

### 17. defineComponent 的作用？

**答案：**
提供 TypeScript 类型推断的组件定义助手：

```typescript
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    message: String,
  },
  setup(props) {
    // props 有正确的类型推断
    return () => h("div", props.message);
  },
});
```

### 18. Teleport 组件的作用？

**答案：**
将组件内容渲染到 DOM 树的其他位置：

```vue
<template>
  <button @click="show = true">打开模态框</button>

  <Teleport to="body">
    <div v-if="show" class="modal">
      <p>模态框内容</p>
    </div>
  </Teleport>
</template>
```

### 19. Suspense 组件的作用？

**答案：**
处理异步组件加载状态：

```vue
<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
</template>
```

### 20. 函数式组件的变化？

**答案：**
Vue 3 中函数式组件：

- 必须显式定义为函数
- 不再接收 `context` 参数，而是 `props` 和 `context`
- 性能差异减小，建议使用有状态组件

```javascript
// Vue 3 函数式组件
import { h } from "vue";

const FunctionalComponent = (props, context) => {
  return h("div", props.message);
};

FunctionalComponent.props = ["message"];
```

## Props & Emits

### 21. Vue 3 中 props 的变化？

**答案：**

- TypeScript 支持更好
- 可以配合 withDefaults 提供默认值
- 移除了 .sync 修饰符，用 v-model:propName 替代

```typescript
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    title: String,
    count: {
      type: Number,
      required: true,
    },
  },
});
```

### 22. emits 选项的作用？

**答案：**
明确声明组件触发的事件，提高可读性和类型安全：

```vue
<script>
export default {
  emits: ["update:modelValue", "custom-event"],
  setup(props, { emit }) {
    const handleClick = () => {
      emit("update:modelValue", "new value");
      emit("custom-event", "data");
    };
  },
};
</script>
```

### 23. v-model 在 Vue 3 中的变化？

**答案：**

- 支持多个 v-model 绑定
- 默认使用 `modelValue` 和 `update:modelValue`
- 移除 `.sync` 修饰符

```vue
<!-- 父组件 -->
<ChildComponent v-model="pageTitle" v-model:content="pageContent" />

<!-- 子组件 -->
<script>
export default {
  props: ["modelValue", "content"],
  emits: ["update:modelValue", "update:content"],
};
</script>
```

### 24. 如何实现组件双向绑定？

**答案：**
使用 `v-model` 或自定义事件：

```vue
<!-- 子组件 MyInput.vue -->
<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>

<script>
export default {
  props: ["modelValue"],
  emits: ["update:modelValue"],
};
</script>

<!-- 父组件使用 -->
<MyInput v-model="text" />
```

### 25. 如何传递属性到子组件？

**答案：**
使用 `$attrs` 或 `useAttrs`：

```vue
<!-- 子组件 -->
<template>
  <div v-bind="attrs">自定义组件</div>
</template>

<script setup>
import { useAttrs } from "vue";

const attrs = useAttrs();
</script>
```

## 生命周期

### 26. Vue 3 生命周期钩子的变化？

**答案：**

- beforeCreate、created → 被 setup 替代
- 生命周期前缀改为 "on"（如 onMounted）
- 需要在 setup 中显式导入
- 新增 onRenderTracked、onRenderTriggered（调试用）

### 27. 生命周期执行顺序？

**答案：**

1. **setup()** - 组件初始化
2. **onBeforeMount()** - 挂载前
3. **onMounted()** - 挂载完成
4. **onBeforeUpdate()** - 更新前
5. **onUpdated()** - 更新完成
6. **onBeforeUnmount()** - 卸载前
7. **onUnmounted()** - 卸载完成
8. **onErrorCaptured()** - 错误捕获

### 28. 如何在 setup 中使用生命周期？

**答案：**
导入并在 setup 中调用：

```javascript
import { onMounted, onUpdated, onUnmounted } from 'vue'

setup() {
  onMounted(() => {
    console.log('组件已挂载')
  })

  onUpdated(() => {
    console.log('组件已更新')
  })

  onUnmounted(() => {
    console.log('组件已卸载')
  })
}
```

### 29. onRenderTracked 和 onRenderTriggered 的作用？

**答案：**
调试钩子，用于追踪响应式依赖：

- **onRenderTracked**：跟踪虚拟 DOM 重新渲染时调用的响应式依赖
- **onRenderTriggered**：跟踪触发虚拟 DOM 重新渲染的依赖

```javascript
import { onRenderTracked, onRenderTriggered } from "vue";

onRenderTracked((event) => {
  console.log("跟踪依赖:", event);
});

onRenderTriggered((event) => {
  console.log("触发更新:", event);
});
```

## 指令系统

### 30. 自定义指令的变化？

**答案：**
Vue 3 自定义指令：

- 生命周期钩子重命名（bind → mounted）
- 添加 `beforeMount` 和 `beforeUpdate` 钩子
- 指令参数通过 `binding.value` 访问

```javascript
const vFocus = {
  mounted(el) {
    el.focus();
  },
  updated(el, binding) {
    if (binding.value) {
      el.focus();
    }
  },
};
```

### 31. 如何创建自定义指令？

**答案：**

```javascript
// 全局指令
app.directive("highlight", {
  beforeMount(el, binding) {
    el.style.backgroundColor = binding.value;
  },
  updated(el, binding) {
    el.style.backgroundColor = binding.value;
  },
});

// 局部指令
const directives = {
  focus: {
    mounted(el) {
      el.focus();
    },
  },
};
```

### 32. v-model 自定义指令？

**答案：**
自定义组件使用 v-model：

```javascript
app.component("CustomInput", {
  props: ["modelValue"],
  emits: ["update:modelValue"],
  template: `
    <input
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    >
  `,
});
```

## 路由与状态管理

### 33. Vue Router 4 的主要变化？

**答案：**

- 创建方式改变：`createRouter()`
- 路由模式：`createWebHistory()` 替代 `mode: 'history'`
- 移除 `*` 通配符路由，改用 `/path/:pathMatch(.*)*`
- 导航守卫 API 变化
- Route 对象属性变化

### 34. 如何在 setup 中使用路由？

**答案：**
使用 `useRouter` 和 `useRoute`：

```javascript
import { useRouter, useRoute } from 'vue-router'

setup() {
  const router = useRouter()
  const route = useRoute()

  const goHome = () => {
    router.push('/')
  }

  // 监听路由参数变化
  watch(() => route.params.id, (newId) => {
    fetchData(newId)
  })
}
```

### 35. Pinia 与 Vuex 的区别？

**答案：**
Pinia 是 Vue 3 推荐的状态管理库：

- 更简单的 API（没有 mutations，只有 actions）
- 更好的 TypeScript 支持
- 支持 Composition API
- 模块化设计（多个 store）
- 轻量级，代码更简洁

### 36. 如何创建和使用 Pinia store？

**答案：**

```javascript
// stores/counter.js
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
  actions: {
    increment() {
      this.count++;
    },
  },
  getters: {
    doubleCount: (state) => state.count * 2,
  },
});

// 组件中使用
import { useCounterStore } from "@/stores/counter";

const counter = useCounterStore();
counter.increment();
console.log(counter.doubleCount);
```

## 性能优化

### 37. Vue 3 的性能优化措施？

**答案：**

- **响应式系统优化**：Proxy 比 defineProperty 更快
- **编译优化**：
  - 静态节点提升（Hoist Static）
  - Patch Flags（更新类型标记）
  - 树结构拍平（Tree Flattening）
  - 缓存事件处理程序（Cache Handlers）
- **Tree-shaking**：更好的按需引入
- **更小的包体积**：核心库约 10KB gzipped

### 38. 什么是静态节点提升？

**答案：**
编译时将静态节点提取到渲染函数外部，避免重复创建：

```javascript
// 编译前
<div>
  <span>静态内容</span>
  <span>{{ dynamic }}</span>
</div>;

// 编译后
const _hoisted = createVNode("span", null, "静态内容");

function render() {
  return (
    _openBlock(),
    _createBlock("div", null, [
      _hoisted, // 复用静态节点
      createVNode("span", null, _toDisplayString(dynamic), 1 /* TEXT */),
    ])
  );
}
```

### 39. 什么是 Patch Flags？

**答案：**
虚拟 DOM 更新时的优化标记，标识节点需要更新的类型：

```javascript
// Patch Flags 示例
const patchFlags = {
  TEXT: 1, // 动态文本
  CLASS: 2, // 动态 class
  STYLE: 4, // 动态 style
  PROPS: 8, // 动态属性
  FULL_PROPS: 16, // 动态 key
  HYDRATE_EVENTS: 32,
  STABLE_FRAGMENT: 64,
  KEYED_FRAGMENT: 128,
  UNKEYED_FRAGMENT: 256,
  NEED_PATCH: 512,
  DYNAMIC_SLOTS: 1024,
  HOISTED: -1, // 静态节点
  BAIL: -2, // 退出优化模式
};
```

### 40. 如何优化大型列表渲染？

**答案：**

- 使用虚拟滚动（vue-virtual-scroller）
- 使用 `v-for` 的 `key`
- 使用 `v-memo` 缓存组件
- 分页或懒加载
- 避免在模板中使用复杂表达式

```vue
<template>
  <!-- 虚拟滚动 -->
  <RecycleScroller :items="largeList" :item-size="50">
    <template #default="{ item }">
      <ListItem :item="item" />
    </template>
  </RecycleScroller>

  <!-- v-memo 缓存 -->
  <div v-memo="[valueA, valueB]">复杂组件内容</div>
</template>
```

## TypeScript 支持

### 41. Vue 3 对 TypeScript 的支持改进？

**答案：**

- Composition API 更好的类型推断
- defineComponent 提供完整的类型支持
- Props、Emits 类型声明
- 模板中的 TypeScript 支持（Volar）
- 更好的泛型支持

### 42. 如何为组件 props 定义类型？

**答案：**

```typescript
import { defineComponent, PropType } from "vue";

interface User {
  id: number;
  name: string;
}

export default defineComponent({
  props: {
    // 简单类型
    title: String,

    // 多种类型
    value: [String, Number],

    // 必填
    count: {
      type: Number,
      required: true,
    },

    // 默认值
    enabled: {
      type: Boolean,
      default: false,
    },

    // 复杂类型
    user: {
      type: Object as PropType<User>,
      required: true,
    },

    // 数组类型
    items: {
      type: Array as PropType<string[]>,
      default: () => [],
    },
  },
});
```

### 43. 如何为 emits 定义类型？

**答案：**

```typescript
import { defineComponent } from "vue";

export default defineComponent({
  emits: {
    // 无参数
    change: null,

    // 带参数
    "update:value": (value: string) => {
      // 验证参数
      return typeof value === "string";
    },

    // 带 payload 类型
    submit: (payload: { email: string; password: string }) => {
      return payload.email.includes("@") && payload.password.length >= 6;
    },
  },
});
```

## 进阶主题

### 44. 什么是渲染函数？如何使用？

**答案：**
使用 JavaScript 编写模板的底层方式：

```javascript
import { h } from "vue";

export default {
  render() {
    return h(
      "div",
      {
        class: "container",
        onClick: this.handleClick,
      },
      [h("h1", "标题"), h("p", this.message)],
    );
  },
};
```

### 45. 什么是 JSX 在 Vue 3 中的使用？

**答案：**
Vue 3 支持 JSX，需要配置 Babel 插件：

```jsx
import { defineComponent } from "vue";

export default defineComponent({
  setup() {
    const count = ref(0);

    return () => (
      <div class="container">
        <h1>Count: {count.value}</h1>
        <button onClick={() => count.value++}>增加</button>
      </div>
    );
  },
});
```

### 46. 如何实现插件开发？

**答案：**

```javascript
// my-plugin.js
export default {
  install(app, options) {
    // 全局组件
    app.component("MyComponent", MyComponent);

    // 全局指令
    app.directive("focus", FocusDirective);

    // 全局属性
    app.config.globalProperties.$myMethod = () => {
      console.log("全局方法");
    };

    // 注入 provide
    app.provide("pluginOptions", options);
  },
};

// 使用
import { createApp } from "vue";
import MyPlugin from "./my-plugin";

const app = createApp(App);
app.use(MyPlugin, { someOption: true });
```

### 47. 什么是自定义渲染器？

**答案：**
创建针对不同平台的渲染器（如 Canvas、终端）：

```javascript
import { createRenderer } from "@vue/runtime-core";

const { createApp } = createRenderer({
  createElement(type) {
    // 创建元素逻辑
  },
  insert(el, parent) {
    // 插入元素逻辑
  },
  // ... 其他节点操作方法
});
```

### 48. 如何实现服务端渲染（SSR）？

**答案：**
Vue 3 SSR 使用 `@vue/server-renderer`：

```javascript
import { createSSRApp } from "vue";
import { renderToString } from "@vue/server-renderer";

const app = createSSRApp({
  data() {
    return { message: "Hello SSR" };
  },
  template: `<div>{{ message }}</div>`,
});

const html = await renderToString(app);
```

### 49. Vue 3 的编译时优化有哪些？

**答案：**

- **静态提升**：将静态节点提取到渲染函数外部
- **Patch Flags**：标记动态节点类型
- **树结构拍平**：减少嵌套组件更新开销
- **缓存事件处理程序**：避免重复创建事件函数
- **SSR 优化**：更高效的服务器端渲染

### 50. 如何从 Vue 2 迁移到 Vue 3？

**答案：**
迁移步骤：

1. **准备工作**：升级 Vue 2 到最新版本，安装 @vue/compat
2. **使用迁移构建**：启用兼容模式
3. **逐步迁移**：
   - 替换废弃的 API
   - 更新生命周期钩子
   - 迁移全局 API
   - 更新路由和状态管理
4. **测试验证**：确保功能正常
5. **优化**：使用 Composition API 重构复杂组件
6. **工具支持**：使用 Vue CLI 或 Vite

**常见变化：**

- `Vue.extend` → `defineComponent`
- `$children` 移除，使用 refs 或 provide/inject
- `$listeners` 合并到 `$attrs`
- `filter` 移除，使用方法或计算属性替代
- 事件总线使用 mitt 或全局事件

---

这些题目涵盖了 Vue 3 的核心概念和实际应用，适合不同层次的面试准备。建议根据具体岗位要求选择重点题目进行深入准备。
