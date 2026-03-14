## VueUse 功能完整总结

VueUse 是基于 Composition API 的 300+ 实用工具函数集合，堪称 Vue 3 的"瑞士军刀"！以下是完整的功能分类总结：

## 一、状态管理（State）

| 功能         | 函数                | 作用                           |
| ------------ | ------------------- | ------------------------------ |
| **本地存储** | `useLocalStorage`   | 响应式 localStorage            |
| **会话存储** | `useSessionStorage` | 响应式 sessionStorage          |
| **历史记录** | `useRefHistory`     | 追踪 ref 变化历史（撤销/重做） |
| **防抖 ref** | `debouncedRef`      | 防抖的响应式数据               |
| **节流 ref** | `throttledRef`      | 节流的响应式数据               |
| **异步状态** | `useAsyncState`     | 管理异步请求状态               |

```typescript
// 示例：带历史记录的文本编辑器
const text = ref("");
const history = useRefHistory(text);
const undo = () => history.undo();
const redo = () => history.redo();
```

## 二、浏览器 API（Browser）

| 功能         | 函数            | 作用             |
| ------------ | --------------- | ---------------- |
| **标题**     | `useTitle`      | 修改页面标题     |
| **图标**     | `useFavicon`    | 修改网站图标     |
| **剪贴板**   | `useClipboard`  | 复制文本到剪贴板 |
| **网络状态** | `useNetwork`    | 网络连接状态     |
| **在线状态** | `useOnline`     | 是否在线         |
| **窗口大小** | `useWindowSize` | 响应式窗口尺寸   |
| **滚动**     | `useScroll`     | 滚动位置         |
| **暗色模式** | `useDark`       | 暗色主题切换     |
| **媒体查询** | `useMediaQuery` | CSS 媒体查询     |
| **全屏**     | `useFullscreen` | 全屏切换         |
| **打印**     | `usePrint`      | 打印控制         |

```typescript
// 示例：暗色模式 + 全屏
const isDark = useDark();
const { isFullscreen, toggle } = useFullscreen();
const { copy } = useClipboard();
```

## 三、传感器（Sensors）

| 功能         | 函数                      | 作用           |
| ------------ | ------------------------- | -------------- |
| **鼠标**     | `useMouse`                | 鼠标位置跟踪   |
| **键盘**     | `useKeyboard`             | 键盘事件       |
| **组合键**   | `useMagicKeys`            | 监听组合键     |
| **滚动**     | `useScroll`               | 滚动位置       |
| **交叉观察** | `useIntersectionObserver` | 元素可见性     |
| **大小观察** | `useResizeObserver`       | 元素大小变化   |
| **元素可见** | `useElementVisibility`    | 元素是否在视口 |
| **活动元素** | `useActiveElement`        | 当前焦点元素   |
| **拖拽**     | `useDraggable`            | 拖拽功能       |

```typescript
// 示例：组合键监听
const { Ctrl_C, Ctrl_V } = useMagicKeys();
watch(Ctrl_C, (v) => v && copy());
```

## 四、动画（Animation）

| 功能       | 函数            | 作用         |
| ---------- | --------------- | ------------ |
| **过渡**   | `useTransition` | 数值过渡动画 |
| **弹簧**   | `useSpring`     | 弹簧物理动画 |
| **动画帧** | `useRafFn`      | 请求动画帧   |
| **定时器** | `useInterval`   | 间隔执行     |
| **延时器** | `useTimeout`    | 延时执行     |
| **倒计时** | `useCountdown`  | 倒计时       |
| **时间戳** | `useTimestamp`  | 实时时间戳   |

```typescript
// 示例：数字滚动动画
const source = ref(0);
const output = useTransition(source, {
  duration: 1000,
  transition: [0.75, 0, 0.25, 1],
});
```

## 五、工具函数（Utilities）

| 功能         | 函数            | 作用         |
| ------------ | --------------- | ------------ |
| **防抖**     | `useDebounceFn` | 函数防抖     |
| **节流**     | `useThrottleFn` | 函数节流     |
| **延时执行** | `useTimeoutFn`  | 延时执行函数 |
| **间隔执行** | `useIntervalFn` | 间隔执行函数 |
| **记忆化**   | `useMemoize`    | 缓存函数结果 |
| **取反**     | `useToggle`     | 布尔值切换   |
| **计数器**   | `useCounter`    | 数字增减     |
| **循环**     | `useCycleList`  | 循环列表     |

```typescript
// 示例：常用工具组合
const [value, toggle] = useToggle();
const { count, inc, dec } = useCounter(0);
const search = useDebounceFn(fetchData, 500);
```

## 六、事件（Events）

| 功能         | 函数               | 作用         |
| ------------ | ------------------ | ------------ |
| **事件监听** | `useEventListener` | 添加事件监听 |
| **按键修饰** | `useKeyModifier`   | 按键状态     |
| **组合键**   | `useMagicKeys`     | 组合键监听   |
| **滑动**     | `useSwipe`         | 触摸滑动     |
| **点击外部** | `onClickOutside`   | 点击元素外部 |
| **长按**     | `onLongPress`      | 长按事件     |

```typescript
// 示例：点击外部关闭弹窗
const modal = ref(null);
onClickOutside(modal, () => {
  showModal.value = false;
});
```

## 七、组件（Component）

| 功能         | 函数          | 作用               |
| ------------ | ------------- | ------------------ |
| **v-model**  | `useVModel`   | 简化 v-model 绑定  |
| **传值**     | `useVModels`  | 多个 v-model       |
| **模板 ref** | `templateRef` | 类型安全的模板 ref |

```typescript
// 示例：自定义组件 v-model
const props = defineProps<{ modelValue: string }>();
const emit = defineEmits(["update:modelValue"]);
const value = useVModel(props, "modelValue", emit);
```

## 八、格式化（Format）

| 功能     | 函数                | 作用                |
| -------- | ------------------- | ------------------- |
| **时间** | `useTimeAgo`        | 相对时间（3分钟前） |
| **日期** | `useDateFormat`     | 日期格式化          |
| **数字** | `useNumberFormat`   | 数字格式化          |
| **货币** | `useCurrencyFormat` | 货币格式化          |

```typescript
// 示例：时间显示
const timeAgo = useTimeAgo(new Date("2024-01-01"));
// "2个月后"
```

## 九、网络（Network）

| 功能          | 函数           | 作用           |
| ------------- | -------------- | -------------- |
| **Fetch**     | `useFetch`     | 响应式 fetch   |
| **WebSocket** | `useWebSocket` | WebSocket 连接 |
| **网络状态**  | `useNetwork`   | 网络信息       |
| **在线状态**  | `useOnline`    | 在线检测       |

```typescript
// 示例：自动重试的请求
const { data, error, isLoading } = useFetch("/api/users", {
  retry: 3,
  timeout: 5000,
});
```

## 十、集成（Integrations）

| 功能        | 函数         | 作用         |
| ----------- | ------------ | ------------ |
| **Axios**   | `useAxios`   | Axios 集成   |
| **Cookies** | `useCookies` | Cookies 操作 |
| **焦点**    | `useFocus`   | 焦点管理     |
| **空闲**    | `useIdle`    | 用户空闲检测 |

## 十一、最佳实践组合

### **1. 表单处理**

```typescript
const form = useLocalStorage("form", { name: "", age: "" });
const debouncedSave = useDebounceFn(() => {
  // 自动保存
}, 1000);
watch(form, debouncedSave, { deep: true });
```

### **2. 无限滚动**

```typescript
const loadMoreRef = ref(null);
const { y } = useWindowScroll();
const { stop } = useIntersectionObserver(
  loadMoreRef,
  ([{ isIntersecting }]) => {
    if (isIntersecting) loadMore();
  },
);
```

### **3. 性能优化**

```typescript
// 防抖搜索 + 缓存结果
const search = useDebounceFn(async (kw) => {
  const cached = useMemoize(fetchSearch);
  return cached(kw);
}, 500);
```

## 十二、学习路线图

1. **初级阶段**（最常用）：
   - `useLocalStorage`、`useDark`、`useClipboard`
   - `useDebounceFn`、`useThrottleFn`
   - `useWindowSize`、`useMouse`

2. **中级阶段**：
   - `useRefHistory`、`useFetch`
   - `useIntersectionObserver`
   - `useMagicKeys`、`useVModel`

3. **高级阶段**：
   - `useTransition`、`useSpring`
   - `useWebSocket`
   - 自定义组合函数

## 总结

**VueUse 核心价值**：

- ✅ **300+ 工具函数** - 覆盖绝大多数开发场景
- ✅ **响应式** - 完美融入 Vue 生态
- ✅ **TypeScript** - 类型安全
- ✅ **Tree-shaking** - 按需引入
- ✅ **SSR 友好** - 服务端渲染支持

**一句话总结**：有了 VueUse，你基本不用自己写常用的逻辑了，直接拿来用就行！
