# 🎯 结构体（struct）与类（class）核心区别

## 📊 核心区别对比表

| 特性         | **结构体（struct）**       | **类（class）**    |
| ------------ | -------------------------- | ------------------ |
| **类型**     | 值类型                     | 引用类型           |
| **内存分配** | 栈（通常）                 | 堆（托管堆）       |
| **继承**     | 不能继承，隐式密封         | 支持单继承，可派生 |
| **默认值**   | 所有字段初始化为0/null     | 引用为null         |
| **赋值**     | 值拷贝（深拷贝）           | 引用拷贝（浅拷贝） |
| **可为null** | 需要`Nullable<T>`或`?`修饰 | 默认可为null       |
| **性能**     | 栈分配快，无GC压力         | 堆分配，有GC开销   |
| **大小限制** | 建议小于16字节             | 无限制             |

---

## 💾 内存分配与行为差异

### **内存分配图示**

```
结构体（值类型）：
┌─────────────────┐
│     栈内存      │
├─────────────────┤
│ struct Point    │
│   int X = 5     │ ← 值直接存储在栈上
│   int Y = 10    │
└─────────────────┘

类（引用类型）：
┌─────────────────┐    ┌─────────────────┐
│     栈内存      │    │     堆内存      │
├─────────────────┤    ├─────────────────┤
│ classRef →      │───→│ Person对象      │
│ 地址0x1234      │    │   Name="John"   │
│                 │    │   Age=25        │
└─────────────────┘    └─────────────────┘
```

### **赋值行为示例**

```csharp
// 结构体 - 值拷贝
struct Point
{
    public int X, Y;
}

Point p1 = new Point { X = 1, Y = 2 };
Point p2 = p1;  // 创建副本
p2.X = 99;      // 只修改副本
Console.WriteLine(p1.X); // 输出: 1（原值不变）

// 类 - 引用拷贝
class Person
{
    public string Name;
    public int Age;
}

Person person1 = new Person { Name = "Alice", Age = 25 };
Person person2 = person1;  // 复制引用
person2.Name = "Bob";
Console.WriteLine(person1.Name); // 输出: Bob（原对象也被修改）
```

---

## 🎯 何时使用结构体（而不是类）

### ✅ **使用结构体的场景**

#### 1. **小型、简单的数据载体**

```csharp
// ✅ 适合用struct：简单坐标
public struct Point3D
{
    public float X, Y, Z;
    public Point3D(float x, float y, float z) => (X, Y, Z) = (x, y, z);
}

// ❌ 不适合用struct：复杂对象
public struct Employee  // 不好！结构体太大且复杂
{
    public string Name;
    public DateTime BirthDate;
    public decimal Salary;
    public List<string> Skills;  // 引用类型字段！
}
```

#### 2. **不可变的值语义**

```csharp
// ✅ 不可变结构体
public readonly struct Currency
{
    public decimal Amount { get; }
    public string Code { get; }  // "USD", "EUR"等

    public Currency(decimal amount, string code)
    {
        Amount = amount;
        Code = code;
    }
}

// 使用示例
Currency price = new Currency(99.99m, "USD");
// price.Amount = 100; // 编译错误 - 不可变！
```

#### 3. **频繁创建的轻量级对象**

```csharp
// 游戏开发中的典型用例
public struct Vector2
{
    public float X, Y;

    public static Vector2 operator +(Vector2 a, Vector2 b)
        => new Vector2(a.X + b.X, a.Y + b.Y);
}

// 每帧创建成千上万个，无GC压力
void UpdateGame()
{
    Vector2[] positions = new Vector2[10000];
    for (int i = 0; i < positions.Length; i++)
    {
        positions[i] = new Vector2(i, i * 2);  // 栈分配，快速
    }
}
```

#### 4. **逻辑上是值而不是对象**

```csharp
// ✅ 颜色值
public struct RgbaColor
{
    public byte R, G, B, A;

    public static readonly RgbaColor Red = new RgbaColor(255, 0, 0, 255);
    public static readonly RgbaColor Green = new RgbaColor(0, 255, 0, 255);
}

// ✅ 日期范围
public readonly struct DateRange
{
    public DateTime Start { get; }
    public DateTime End { get; }
    public TimeSpan Duration => End - Start;

    public DateRange(DateTime start, DateTime end)
    {
        Start = start;
        End = end;
    }
}
```

### ❌ **避免使用结构体的场景**

1. **需要继承或多态**
2. **大小超过16字节**（性能考虑）
3. **需要频繁装箱拆箱**
4. **包含可变引用类型字段**
5. **需要表示"无"值**（用class更合适）

---

## ⚡ 性能考虑

```csharp
// 性能测试对比
public class PerformanceTest
{
    struct Vector3Struct { public float X, Y, Z; }
    class Vector3Class { public float X, Y, Z; }

    void Test()
    {
        const int ITERATIONS = 1_000_000;

        // 结构体数组 - 内存连续，缓存友好
        Vector3Struct[] structs = new Vector3Struct[ITERATIONS];

        // 类数组 - 内存分散，指针间接访问
        Vector3Class[] classes = new Vector3Class[ITERATIONS];
        for (int i = 0; i < ITERATIONS; i++)
            classes[i] = new Vector3Class();  // 每个都单独分配
    }
}
```

---

## 🔍 实际案例：何时选择struct

### **案例1：数学计算库**

```csharp
// 数学库中的结构体
public readonly struct ComplexNumber
{
    public double Real { get; }
    public double Imaginary { get; }

    public ComplexNumber(double real, double imaginary)
    {
        Real = real;
        Imaginary = imaginary;
    }

    // 操作符重载 - 值语义很自然
    public static ComplexNumber operator +(ComplexNumber a, ComplexNumber b)
        => new ComplexNumber(a.Real + b.Real, a.Imaginary + b.Imaginary);
}
```

### **案例2：高性能游戏引擎**

```csharp
// 游戏引擎中的结构体
public struct Transform
{
    public Vector3 Position;
    public Quaternion Rotation;
    public Vector3 Scale;

    // 大小：3*4 + 4*4 + 3*4 = 40字节（略大，但在可接受范围）
}

// 使用Transform矩阵进行批处理
Transform[] transforms = new Transform[1000];
// 连续内存，SIMD优化友好
```

### **案例3：数据库记录映射**

```csharp
// 小型DTO
public readonly struct OrderItemKey
{
    public int OrderId { get; }
    public int ProductId { get; }

    public OrderItemKey(int orderId, int productId)
    {
        OrderId = orderId;
        ProductId = productId;
    }

    // 作为字典键很高效
    public override int GetHashCode()
        => HashCode.Combine(OrderId, ProductId);
}
```

---

## 📋 决策检查清单

**考虑struct当：**

- ✓ 数据大小 < 16字节
- ✓ 逻辑上是"值"而非"对象"
- ✓ 需要值语义（拷贝行为）
- ✓ 频繁实例化，关注性能
- ✓ 数据是不可变的或很少修改
- ✓ 不需要继承或多态

**考虑class当：**

- ✓ 数据较大或复杂
- ✓ 需要继承或接口实现
- ✓ 需要引用语义
- ✓ 需要表示"无"（null）
- ✓ 包含大量引用类型字段
- ✓ 需要身份标识（而非值相等）

---

## 💡 最佳实践建议

1. **优先使用`readonly struct`**确保不可变性
2. **实现`IEquatable<T>`**避免装箱
3. **重写`Equals()`和`GetHashCode()`**
4. **提供有意义的`ToString()`**
5. **考虑实现操作符重载**（==, !=, +, -等）
6. **如果可能，提供`Deconstruct()`方法**

```csharp
public readonly struct Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public bool Equals(Money other)
        => Amount == other.Amount && Currency == other.Currency;

    public override bool Equals(object obj)
        => obj is Money other && Equals(other);

    public override int GetHashCode()
        => HashCode.Combine(Amount, Currency);

    public void Deconstruct(out decimal amount, out string currency)
        => (amount, currency) = (Amount, Currency);
}
```

**记住：struct是值，class是对象。选择取决于你的数据在概念上是什么，而不仅仅是性能考虑。**
