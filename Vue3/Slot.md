# 🎯 **Slot（插槽）的作用详解**

## 📍 **一句话总结**

**Slot = 父组件向子组件传递内容（不仅仅是数据，是HTML/组件）**

## 🔄 **与 Props 的区别**

| 特性           | Props                        | Slot                         |
| -------------- | ---------------------------- | ---------------------------- |
| **传递什么**   | 数据（字符串、数字、对象等） | **内容**（HTML、组件、文本） |
| **在哪里使用** | 子组件内部逻辑处理           | 子组件指定位置渲染           |
| **类比**       | 函数参数                     | 函数占位符                   |

## 🎯 **核心作用**

### **1. 内容分发**

```vue
<!-- ChildComponent.vue 子组件 -->
<template>
  <div class="card">
    <div class="header">
      <!-- 这里是插槽位置 -->
      <slot></slot>
    </div>
  </div>
</template>

<!-- ParentComponent.vue 父组件 -->
<template>
  <ChildComponent>
    <!-- 这里的内容会出现在子组件的 <slot> 位置 -->
    <h2>我是标题</h2>
    <p>我是内容</p>
  </ChildComponent>
</template>
```

### **2. 组件复用和定制**

```vue
<!-- Button.vue -->
<template>
  <button class="btn">
    <!-- 用户可以自定义按钮内容 -->
    <slot>默认按钮</slot>
  </button>
</template>

<!-- 使用 -->
<Button>登录</Button>
<Button>注册</Button>
<Button>
  <Icon name="save" /> 保存
</Button>
```

## 📦 **Slot 的类型**

### **1. 默认插槽**

```vue
<!-- Modal.vue -->
<template>
  <div class="modal">
    <div class="modal-content">
      <!-- 默认插槽 -->
      <slot></slot>
    </div>
    <div class="modal-footer">
      <slot name="footer">
        <!-- 默认内容 -->
        <button>确定</button>
      </slot>
    </div>
  </div>
</template>

<!-- 使用 -->
<Modal>
  <!-- 默认插槽内容 -->
  <p>这是模态框的内容</p>
  
  <!-- 具名插槽内容 -->
  <template #footer>
    <button>取消</button>
    <button>确认</button>
  </template>
</Modal>
```

### **2. 具名插槽**

```vue
<!-- Layout.vue -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
      <!-- 默认插槽 -->
    </main>
    <aside>
      <slot name="sidebar"></slot>
    </aside>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<!-- 使用 -->
<Layout>
  <template #header>
    <h1>网站标题</h1>
    <NavBar />
  </template>
  
  <!-- 默认插槽 -->
  <ArticleContent />
  
  <template #sidebar>
    <SidebarMenu />
  </template>
  
  <template #footer>
    <CopyrightInfo />
  </template>
</Layout>
```

### **3. 作用域插槽（最强大）**

```vue
<!-- DataTable.vue 子组件 -->
<template>
  <table>
    <thead>
      <tr>
        <th v-for="col in columns" :key="col.key">
          {{ col.title }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, index) in data" :key="row.id">
        <!-- 作用域插槽：向父组件暴露数据 -->
        <slot name="row" :row="row" :index="index" :is-even="index % 2 === 0">
          <!-- 默认内容 -->
          <td v-for="col in columns" :key="col.key">
            {{ row[col.key] }}
          </td>
        </slot>
      </tr>
    </tbody>
  </table>
</template>

<!-- 使用 -->
<DataTable :data="users" :columns="columns">
  <template #row="{ row, index, isEven }">
    <!-- 可以访问子组件的数据 -->
    <td :class="{ 'even-row': isEven }">{{ index + 1 }}</td>
    <td>{{ row.name }}</td>
    <td>
      <button @click="editUser(row)">编辑</button>
    </td>
  </template>
</DataTable>
```

## 🚀 **实际应用场景**

### **场景1：UI 组件库的灵活性**

```vue
<!-- Card.vue -->
<template>
  <div
    class="card"
    :class="[shadow ? 'shadow' : '', bordered ? 'bordered' : '']"
  >
    <!-- 标题插槽 -->
    <div v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </div>

    <!-- 内容插槽 -->
    <div class="card-body">
      <slot></slot>
    </div>

    <!-- 底部插槽 -->
    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<!-- 使用 -->
<Card shadow bordered>
  <template #header>
    <h3>用户信息</h3>
    <Badge type="success">VIP</Badge>
  </template>
  
  <!-- 默认插槽 -->
  <UserProfile :user="currentUser" />
  
  <template #footer>
    <Button>保存</Button>
    <Button type="secondary">取消</Button>
  </template>
</Card>
```

### **场景2：列表渲染定制**

```vue
<!-- VirtualList.vue -->
<template>
  <div class="virtual-list" ref="container">
    <div v-for="item in visibleItems" :key="item.id" class="list-item">
      <!-- 作用域插槽让父组件决定如何渲染 -->
      <slot :item="item" :index="item.index">
        {{ item.content }}
      </slot>
    </div>
  </div>
</template>

<!-- 使用 -->
<VirtualList :items="largeDataset">
  <template #default="{ item }">
    <!-- 自定义复杂渲染 -->
    <div class="custom-item">
      <Avatar :src="item.avatar" />
      <div>
        <h4>{{ item.name }}</h4>
        <p>{{ item.description }}</p>
        <StarRating :rating="item.rating" />
      </div>
    </div>
  </template>
</VirtualList>
```

### **场景3：复合组件模式**

```vue
<!-- Tabs.vue -->
<template>
  <div class="tabs">
    <div class="tabs-header">
      <div
        v-for="tab in tabs"
        :key="tab.name"
        :class="['tab', { active: activeTab === tab.name }]"
        @click="activeTab = tab.name"
      >
        {{ tab.label }}
      </div>
    </div>

    <div class="tabs-content">
      <slot></slot>
    </div>
  </div>
</template>

<!-- Tab.vue -->
<template>
  <div v-if="name === parent?.activeTab">
    <slot></slot>
  </div>
</template>

<!-- 使用 -->
<Tabs>
  <Tab name="info" label="基本信息">
    <UserInfoForm />
  </Tab>
  
  <Tab name="settings" label="设置">
    <UserSettings />
  </Tab>
  
  <Tab name="history" label="历史记录">
    <UserHistory />
  </Tab>
</Tabs>
```

## 🔧 **高级特性**

### **1. 动态插槽名**

```vue
<!-- Layout.vue -->
<template>
  <div class="layout">
    <component
      v-for="(content, name) in $slots"
      :key="name"
      :is="`slot-${name}`"
    >
      <slot :name="name"></slot>
    </component>
  </div>
</template>

<!-- 使用 -->
<Layout>
  <template #[dynamicSlotName]>
    动态插槽内容
  </template>
</Layout>
```

### **2. 插槽作用域访问**

```vue
<script setup>
// 在脚本中访问插槽
import { useSlots } from "vue";

const slots = useSlots();

// 检查是否有特定插槽
const hasHeader = !!slots.header;

// 访问作用域插槽数据
// 需要在渲染时通过作用域插槽传递
</script>
```

### **3. 插槽默认内容**

```vue
<!-- Button.vue -->
<template>
  <button class="btn">
    <!-- 如果没有提供内容，显示默认 -->
    <slot>
      <span class="default-text">点击我</span>
      <Icon name="arrow-right" />
    </slot>
  </button>
</template>
```

## 💡 **最佳实践**

### **实践1：提供合理的默认内容**

```vue
<!-- Badge.vue -->
<template>
  <span class="badge">
    <slot>
      <!-- 有用的默认值 -->
      <span class="badge-content">
        <Icon v-if="icon" :name="icon" />
        {{ label || "新消息" }}
      </span>
    </slot>
  </span>
</template>
```

### **实践2：组合使用 Props 和 Slot**

```vue
<!-- Alert.vue -->
<template>
  <div :class="['alert', type]">
    <!-- 简单情况用 props -->
    <div v-if="title" class="alert-title">{{ title }}</div>

    <!-- 复杂内容用 slot -->
    <div class="alert-content">
      <slot>{{ message }}</slot>
    </div>

    <!-- 操作按钮用具名插槽 -->
    <div v-if="$slots.actions" class="alert-actions">
      <slot name="actions"></slot>
    </div>
  </div>
</template>

<!-- 使用 -->
<Alert type="warning" title="注意" message="这是一条警告信息">
  <!-- 覆盖默认 message -->
  <p>自定义的详细警告内容...</p>
  
  <template #actions>
    <Button>知道了</Button>
  </template>
</Alert>
```

### **实践3：渲染作用域控制**

```vue
<!-- ConditionalSlot.vue -->
<template>
  <div>
    <!-- 条件渲染插槽 -->
    <slot v-if="condition" name="content"></slot>

    <!-- 循环渲染插槽 -->
    <div v-for="item in items" :key="item.id">
      <slot name="item" :item="item"></slot>
    </div>
  </div>
</template>
```

## 🎯 **什么时候用 Slot？**

### **✅ 适合用 Slot 的场景**

1. **内容结构不确定** - 父组件决定内容
2. **需要插入组件** - 不仅仅是数据
3. **需要定制布局** - 不同的UI组合
4. **渲染逻辑复杂** - 需要作用域数据
5. **提供扩展点** - 让组件更灵活

### **❌ 不适合用 Slot 的场景**

1. **简单数据传递** → 用 Props
2. **组件配置参数** → 用 Props
3. **事件通知** → 用 Emits
4. **全局状态** → 用 Pinia

## 📝 **简单记忆法则**

```
需要传递 HTML/组件 → 用 Slot
需要传递数据/配置 → 用 Props
需要通知事件 → 用 Emits
需要灵活定制 → 用 Slot + 作用域
```

**Slot 的本质是"占位符"，让父组件决定在子组件的特定位置渲染什么内容。** 这是 Vue 组件化中实现灵活性和复用性的关键特性！
