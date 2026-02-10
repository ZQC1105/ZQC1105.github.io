## 🎯 **核心区别总结**

| 特性         | **const**                     | **readonly**               |
| ------------ | ----------------------------- | -------------------------- |
| **本质**     | 编译时常量                    | 运行时常量                 |
| **赋值时机** | 声明时必须赋值                | 声明时**或**构造函数中赋值 |
| **内存**     | 编译时值替换（硬编码）        | 运行时分配内存             |
| **类型**     | 仅限简单类型（int、string等） | 支持任何类型               |
| **访问**     | 通过类名访问（隐式静态）      | 实例/静态均可              |

---

## 📌 **const适用场景**

**✅ 用const当：**

- **数学/物理常量** → `const double PI = 3.14159;`
- **不会变的配置值** → `const int MaxRetry = 3;`
- **简单枚举替代** → `const int AdminRole = 1;`
- **性能要求高**（编译时优化）

**❌ 不用const当：**

- 需要运行时计算的值
- 引用类型（类、数组、集合）
- 可能需要修改的"常量"
- 跨程序集引用（版本控制问题）

---

## 📌 **readonly适用场景**

**✅ 用readonly当：**

- **对象级不可变数据** → `readonly int OrderId;`
- **运行时确定的值** → `readonly DateTime Created = DateTime.Now;`
- **引用类型** → `readonly List<string> items = new();`
- **依赖注入字段** → `readonly ILogger _logger;`
- **每个实例不同值**

**🚨 注意陷阱：**

```csharp
readonly List<int> numbers = new();
numbers.Add(1);     // ✅ 允许！（修改内容）
numbers = new List(); // ❌ 不允许！（重新赋值）
```

---

## 🎯 **一句话选择指南**

> **const = "永远不变"的简单值**
> **readonly = "构造后不变"的任何值**

---

## 💡 **实际选择建议**

```csharp
// 场景1：真正的数学常量 → const
public const double PI = 3.1415926;

// 场景2：应用启动时间 → readonly
public static readonly DateTime StartupTime = DateTime.Now;

// 场景3：对象ID（每个对象不同）→ readonly
public class Order
{
    public readonly int Id;
    public Order(int id) => Id = id;
}

// 场景4：配置列表 → readonly（因为List是引用类型）
public readonly List<string> ValidFiles = new() { ".txt", ".pdf" };
```

---

## 🔄 **现代替代方案**

C# 9.0+ 推荐：

```csharp
// init-only属性（更灵活）
public class Config
{
    public string Name { get; init; }  // 只能在初始化时设值
}

// 记录类型（完全不可变）
public record Person(string FirstName, string LastName);
```

---

## 📋 **决策流程图**

```
需要不可变值吗？
    ├─ 值在编译时已知且是简单类型？
    │    ├─ Yes → 用 const
    │    └─ No → ↓
    ├─ 值在运行时确定或复杂类型？
    │    ├─ Yes → 用 readonly
    │    └─ No → 考虑普通变量
    └─ 需要完全不可变对象？
         └─ 用 record 或 init-only属性
```

**简单记法：能用 const 就用 const，否则用 readonly。**
