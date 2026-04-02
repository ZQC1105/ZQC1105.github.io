这两个方法在**大多数情况下效果相同**，但存在一些关键区别，主要体现在**类型安全**和**适用场景**上。

## 1. 主要区别

| 对比维度       | `GetConnectionString()`                 | 索引器 `["..."]`         |
| :------------- | :-------------------------------------- | :----------------------- |
| **用途**       | **专门**用于读取连接字符串              | **通用**读取任意配置值   |
| **返回值**     | `string?` (可能为 null)                 | `string?` (可能为 null)  |
| **命名约束**   | 键名必须位于 `ConnectionStrings` 节点下 | 可以是任意配置路径       |
| **语义清晰度** | 高，一看就知道是读连接字符串            | 一般，需要看具体键名才懂 |

## 2. 核心差异详解

### `GetConnectionString("DefaultConnection")`

这是一个**专用方法**，它**隐式**地在 `ConnectionStrings` 节点下查找：

```csharp
// 实际等价于
configuration["ConnectionStrings:DefaultConnection"]
```

它做了两件事：

1. 自动添加 `ConnectionStrings:` 前缀
2. 语义明确，告诉其他开发者这是数据库连接字符串

### 索引器 `configuration["..."]`

这是一个**通用访问方式**，可以读取任何配置节点：

```csharp
// 读取连接字符串（需要完整路径）
var conn = configuration["ConnectionStrings:DefaultConnection"];

// 读取其他任意配置
var appName = configuration["AppSettings:Name"];
var logLevel = configuration["Logging:LogLevel:Default"];
```

## 3. 使用建议

### ✅ 推荐：`GetConnectionString()`

```csharp
var connString = builder.Configuration.GetConnectionString("DefaultConnection");
```

- **更清晰**：意图一目了然
- **更简洁**：不需要写完整路径
- **更规范**：符合 ASP.NET Core 的设计惯例

### ⚠️ 特殊场景：索引器

只在以下情况使用索引器：

- 读取**非连接字符串**的配置
- 需要**动态拼接**配置路径时

```csharp
var provider = configuration[$"ConnectionStrings:{tenantId}"];
```

## 4. 底层实现

查看 ASP.NET Core 源码可以发现，`GetConnectionString()` 本质就是调用索引器：

```csharp
public static string GetConnectionString(this IConfiguration configuration, string name)
{
    return configuration?[ConnectionStringKey + ":" + name];
    // ConnectionStringKey = "ConnectionStrings"
}
```

---

**总结**：两者功能等价，但 **`GetConnectionString()` 是专门为读取连接字符串设计的 API，语义更清晰、代码更易维护**。只有在读取非连接字符串配置或需要动态路径时才使用索引器。
