下面这份速记表 = **题目 + 面试官想听的“关键词” + 一句话答案 + 底层/坑点提示**。  
背下来就能直接“对答如流”，每题控制在 15 秒内说完。

---

### C# 新特性 · 面试高频问答速记表（2025 版）

| 特性                    | 面试官想问                        | 一句话答案（必背）                                                                                 | 关键词/坑点提示                                                                     |
| ----------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **init**                | 和 set 有啥区别？                 | `init` 只让属性在“对象初始化期”赋值一次，之后只读；`set` 可反复改。                                | 生成 `IsInitOnly` 标志位；反射可改，但性能差。                                      |
| **record**              | 值类型还是引用类型？比较规则？    | 仍是 `class`（引用类型），但编译器重写 `Equals` 做**逐字段值比较**和 `GetHashCode`。               | 隐藏字段 `EqualityContract` 做类型区分；`with` 表达式做非破坏性修改。               |
| **主构造函数（C# 12）** | 普通类 vs record 主构造函数差异？ | 普通类把参数**注入到整个类体**当“临时常量”，不会自动生成属性；record 会**自动生成 init 属性**。    | 普通类参数作用域=整个类；record 参数只用于生成属性后就不再存在。                    |
| **field（C# 13）**      | 它是字段吗？能在 get 外用吗？     | 是编译器自动生成的**私有匿名字段**，名字 `<P>k__BackingField`；**只能在 get/init/set 体内用**。    | 反射可见；不能手动声明同名字段；性能=手写字段。                                     |
| **required（C# 11）**   | 跟构造函数必填参数比优势？        | 无需自己写重载构造函数，**对象初始化器阶段强制赋值**，还能和 `System.Text.Json` 反序列化直接兼容。 | 编译器生成 `RequiredMemberAttribute`+`SetsRequiredMembers` 构造；不赋值就 CS9035。  |
| **file 类型（C# 11）**  | 解决啥问题？场景？                | 让类/record 只在**当前源文件**可见，避免**一个项目里内部类型重名冲突**，写源代码生成器时特爽。     | 编译器生成 `internal`+`sealed` 并加 `[CompilerGenerated]`；对其它源文件完全不可见。 |

---

### 实战场景“万能模板”——说一遍就能加分

> “我们 DTO 有 60 个字段，之前写 60 行构造，现在用 **record + 主构造函数** 一行搞定，还自带值比较，单元测试断言直接 `dto1 == dto2`。”
>
> “MVVM 原来 50 个属性每套都要 `OnPropertyChanged`，现在用 `field` 关键字 + `CommunityToolkit.Mvvm` 的 `[ObservableProperty]`，编译器替我生成 200 行代码，性能还提升 8%。”

---

### 3 个底层原理“秒答句”

1. `field` 背后  
   “编译器生成 `private readonly T <Prop>k__BackingField`，名字带尖括号，反射能拿到。”

2. `record` 的 `Equals`  
   “先比较 `EqualityContract` 确保同类型，再逐字段比较，最后调 `EqualityComparer<T>.Default`。”

3. `required` 检查时机  
   “编译器在 `new()` 完后插一段 `__Clone` 方法检查所有 `required` 字段是否赋值，未赋值就抛 `MissingMemberException`。”

---

背完这张表，**新特性面试题基本全覆盖**。  
需要我再给你出 5 道“代码挖坑题”做临场演练吗？
除了主构造函数，C# 近几个版本确实引入了许多令人兴奋的新特性，这些特性共同推动着.NET生态向更现代化、更高效的方向发展。让我为你详细介绍这些重要的新特性：

## **C# 10（.NET 6）的关键特性**

### **1. 全局using指令**

```csharp
// 传统方式 - 每个文件都要写
using System;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

// C# 10新方式
// GlobalUsings.cs
global using System;
global using System.Collections.Generic;
global using Microsoft.Extensions.Logging;

// 任何.cs文件都不需要再写这些using了
```

### **2. 文件范围的命名空间**

```csharp
// 以前 - 需要缩进整个文件
namespace MyCompany.MyProject.Services
{
    public class MyService
    {
        // 所有代码都要缩进
    }
}

// C# 10 - 更简洁
namespace MyCompany.MyProject.Services;

public class MyService
{
    // 不需要额外缩进
}
```

### **3. 记录结构体**

```csharp
// 值类型的记录 - 性能更好
public readonly record struct Point(int X, int Y);

// 与类记录对比
var p1 = new Point(1, 2);
var p2 = new Point(1, 2);
Console.WriteLine(p1 == p2); // true - 值相等
Console.WriteLine(p1.Equals(p2)); // true
```

### **4. Lambda表达式改进**

```csharp
// 更简洁的Lambda语法
Func<string, int> parse = (string s) => int.Parse(s);

// C# 10可以推断委托类型
var parse = (string s) => int.Parse(s);

// 支持特性注解
var parse = [NotNull] (string s) => int.Parse(s);
```

## **C# 11（.NET 7）的突破**

### **1. 原始字符串字面量**

```csharp
// 处理JSON、XML、SQL等更简单
string json = """
    {
        "name": "张三",
        "age": 30,
        "address": "北京市"
    }
    """;

// 包含引号也很方便
string code = """
    public class Test
    {
        public void Method()
        {
            Console.WriteLine("Hello World");
        }
    }
    """;
```

### **2. 列表模式匹配**

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// 模式匹配列表
var result = numbers switch
{
    [1, 2, 3, ..] => "以1,2,3开头",
    [.., 4, 5] => "以4,5结尾",
    [] => "空数组",
    [var first, .. var rest] => $"第一个是{first}, 剩余{rest.Length}个",
    _ => "其他"
};
```

### **3. 泛型数学支持**

```csharp
// 编写数学通用算法更容易
static T Add<T>(T left, T right) where T : INumber<T>
{
    return left + right; // 现在可以编译了！
}

// 支持所有数值类型
var intResult = Add(1, 2);        // 3
var doubleResult = Add(1.5, 2.5); // 4.0
var decimalResult = Add(1.5m, 2.5m); // 4.0m
```

### **4. 必需成员**

```csharp
// 确保对象初始化时提供必要属性
public class Person
{
    public required string FirstName { get; init; }
    public required string LastName { get; init; }
    public int Age { get; init; }
}

// 使用 - 必须初始化必需属性
var person = new Person
{
    FirstName = "John",  // 必需
    LastName = "Doe"     // 必需
    // Age是可选的
};

// 这会导致编译错误：
// var person = new Person(); // 缺少FirstName和LastName
```

## **C# 12（.NET 8）的创新**

### **1. 主构造函数**

```csharp
// 之前讨论过的
public class OrderService(IOrderRepository repo, ILogger logger)
{
    public void Process() => repo.Save();
}
```

### **2. 集合表达式**

```csharp
// 统一各种集合的初始化语法
int[] array = [1, 2, 3, 4, 5];
List<int> list = [1, 2, 3, 4, 5];
Span<int> span = [1, 2, 3, 4, 5];

// 支持展开运算符
int[] numbers1 = [1, 2, 3];
int[] numbers2 = [4, 5, 6];
int[] combined = [.. numbers1, .. numbers2, 7, 8, 9];
```

### **3. 内联数组**

```csharp
// 性能关键场景的优化
[System.Runtime.CompilerServices.InlineArray(10)]
public struct Buffer
{
    private int _element0;
    // 编译器生成 _element1 到 _element9
}

// 使用
var buffer = new Buffer();
for (int i = 0; i < 10; i++)
{
    buffer[i] = i; // 没有边界检查，性能更好
}
```

### **4. 默认Lambda参数**

```csharp
// Lambda支持默认参数值
var add = (int x, int y = 1) => x + y;
Console.WriteLine(add(5));    // 6
Console.WriteLine(add(5, 3)); // 8
```

### **5. 别名任意类型**

```csharp
// 为复杂类型创建别名
using Measurement = (string units, int distance);
using Path = int[];

// 使用
Measurement m = ("meters", 5);
Path path = [1, 2, 3, 4, 5];
```

## **C# 13（预计.NET 9）的预览特性**

### **1. 扩展类型（Extension Everything）**

```csharp
// 不仅仅是扩展方法，还能扩展属性、事件等
public class MyExtensions
{
    public extension MyExtension for Person
    {
        public string FullName => $"{FirstName} {LastName}";
        public void CelebrateBirthday() => Age++;
    }
}

// 使用
var person = new Person { FirstName = "John", LastName = "Doe" };
Console.WriteLine(person.FullName); // 扩展属性
person.CelebrateBirthday(); // 扩展方法
```

### **2. 参数空值检查的改进**

```csharp
// 更简洁的空值检查
public void Process(string name!!, IRepository repo!!)
{
    // 自动生成空值检查代码
    // 相当于：
    // if (name is null) throw new ArgumentNullException(nameof(name));
    // if (repo is null) throw new ArgumentNullException(nameof(repo));
}
```

### **3. 字段访问器权限控制**

```csharp
public class BankAccount
{
    private decimal _balance;

    // 允许同程序集内的类读取，但只有本类能修改
    private protected decimal Balance { get; private set; }

    // 更细粒度的访问控制
    private field _secret only BankAccount, Logger;
}
```

## **这些特性如何共同改进开发体验**

### **组合使用示例**

```csharp
// 现代C#代码示例 - 结合多个新特性
namespace ModernCSharpDemo;

// 使用必需属性和主构造函数
public class OrderProcessor(
    required IOrderRepository repository,
    required IPaymentGateway gateway)
{
    // 集合表达式初始化
    private readonly List<string> _supportedCurrencies = ["USD", "EUR", "CNY"];

    // 原始字符串处理JSON
    private const string ConfigTemplate = """
        {
            "timeout": 30,
            "retryCount": 3
        }
        """;

    public Result<Order> ProcessOrder(OrderRequest request)
    {
        // 模式匹配
        return request switch
        {
            { Items: [] } => Result.Fail("订单不能为空"),
            { Total: > 10000 } => ProcessLargeOrder(request),
            [var firstItem, ..] when firstItem.Price > 500 => ProcessExpensiveItem(request),
            _ => ProcessNormalOrder(request)
        };
    }
}
```

## **为什么这些特性重要**

### **1. 开发效率提升**

- **更少的样板代码**：全局using、文件范围命名空间
- **更安全的代码**：必需成员、改进的空值检查
- **更清晰的表达**：原始字符串、集合表达式

### **2. 性能优化**

- **内联数组**：零开销的固定大小集合
- **结构体记录**：减少堆分配
- **泛型数学**：避免装箱

### **3. 现代化趋势**

- **函数式风格**：更好的模式匹配、不可变性
- **声明式编程**：主构造函数、必需成员
- **跨领域统一**：统一的集合语法

## **未来展望**

微软正在沿着几个明确的方向推进C#：

1. **简洁性**：减少样板代码
2. **安全性**：编译时检查更多错误
3. **性能**：提供零成本抽象
4. **表达力**：更丰富的语义
5. **互操作性**：更好地与其他语言和技术栈集成

这些特性中，哪些最让你兴奋？或者你在实际项目中已经用到了哪些新特性？
