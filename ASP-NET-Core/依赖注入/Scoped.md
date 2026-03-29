## "Scoped 服务在每个 HTTP 请求内共享同一个实例" 的含义

简单说就是：**同一个请求中，无论你在哪里使用这个服务，拿到的都是同一个对象；不同请求之间，对象是独立的。**

## 图解说明

```
应用启动
    │
    ├─ 请求1 到达（用户A）
    │   ├─ 创建 Scoped 服务实例 A
    │   ├─ Controller 使用实例 A
    │   ├─ Service 使用实例 A
    │   ├─ Repository 使用实例 A
    │   └─ 请求结束 → 销毁实例 A
    │
    ├─ 请求2 到达（用户B）
    │   ├─ 创建 Scoped 服务实例 B（新的）
    │   ├─ Controller 使用实例 B
    │   ├─ Service 使用实例 B
    │   ├─ Repository 使用实例 B
    │   └─ 请求结束 → 销毁实例 B
    │
    └─ 请求3 到达（用户C）
        ├─ 创建 Scoped 服务实例 C（新的）
        ├─ Controller 使用实例 C
        └─ 请求结束 → 销毁实例 C
```

## 代码示例说明

### 1. **Scoped 服务定义**

```csharp
// 一个简单的 Scoped 服务
public interface IUserContext
{
    string CurrentUserId { get; set; }
    void SetUser(string userId);
}

public class UserContext : IUserContext
{
    public string CurrentUserId { get; set; }

    public void SetUser(string userId)
    {
        CurrentUserId = userId;
    }
}

// Program.cs
builder.Services.AddScoped<IUserContext, UserContext>();
```

### 2. **在多个地方使用同一个 Scoped 服务**

```csharp
// Controller
public class UserController : Controller
{
    private readonly IUserContext _userContext;
    private readonly IUserService _userService;

    public UserController(IUserContext userContext, IUserService userService)
    {
        _userContext = userContext;  // 实例 A
        _userService = userService;
    }

    public async Task<IActionResult> GetProfile()
    {
        // Controller 中设置用户ID
        _userContext.SetUser("User123");

        // 调用 Service
        var profile = await _userService.GetProfile();

        return Ok(profile);
    }
}

// Service
public class UserService : IUserService
{
    private readonly IUserContext _userContext;
    private readonly IRepository _repository;

    public UserService(IUserContext userContext, IRepository repository)
    {
        _userContext = userContext;  // 同一个实例 A
        _repository = repository;
    }

    public async Task<UserProfile> GetProfile()
    {
        // Service 中能获取到 Controller 设置的同一个用户ID
        var userId = _userContext.CurrentUserId;  // "User123"

        return await _repository.GetUser(userId);
    }
}

// Repository
public class UserRepository : IRepository
{
    private readonly IUserContext _userContext;
    private readonly ApplicationDbContext _db;

    public UserRepository(IUserContext userContext, ApplicationDbContext db)
    {
        _userContext = userContext;  // 还是同一个实例 A
        _db = db;
    }

    public async Task<UserProfile> GetUser(string userId)
    {
        // 如果这里不传 userId，也可以从 _userContext 获取
        return await _db.Users.FindAsync(_userContext.CurrentUserId);
    }
}
```

**关键点**：在整个请求处理过程中，`Controller`、`Service`、`Repository` 中的 `_userContext` 都是**同一个对象**。

## 实际运行示例

```csharp
// 请求1：用户A访问
GET /api/user/profile
    ├─ UserController 构造函数：创建 UserContext 实例 #001
    ├─ UserService 构造函数：注入同一个 UserContext 实例 #001
    ├─ UserRepository 构造函数：注入同一个 UserContext 实例 #001
    ├─ Controller 执行：_userContext.SetUser("UserA")
    ├─ Service 执行：_userContext.CurrentUserId = "UserA"  ✅
    └─ 请求结束：实例 #001 被销毁

// 请求2：用户B访问（并发或稍后）
GET /api/user/profile
    ├─ UserController 构造函数：创建新的 UserContext 实例 #002（新的！）
    ├─ UserService 构造函数：注入新的 UserContext 实例 #002
    ├─ UserRepository 构造函数：注入新的 UserContext 实例 #002
    ├─ Controller 执行：_userContext.SetUser("UserB")
    ├─ Service 执行：_userContext.CurrentUserId = "UserB"  ✅
    └─ 请求结束：实例 #002 被销毁
```

## 对比不同生命周期

### 1. **Singleton（单例）**

```csharp
// 注册为单例
builder.Services.AddSingleton<IUserContext, UserContext>();

// 请求1：用户A访问
Controller 拿到实例 #001，设置 CurrentUserId = "UserA"
Service 拿到同一个实例 #001，CurrentUserId = "UserA" ✅

// 请求2：用户B访问（同时发生）
Controller 拿到实例 #001（同一个！），设置 CurrentUserId = "UserB"
Service 拿到同一个实例 #001，CurrentUserId 变成了 "UserB"
// ❌ 问题：请求1的 Service 可能还在处理，但 CurrentUserId 已经被覆盖了！
```

### 2. **Transient（瞬时）**

```csharp
// 注册为瞬时
builder.Services.AddTransient<IUserContext, UserContext>();

// 请求1：用户A访问
Controller 创建实例 #001，设置 CurrentUserId = "UserA"
Service 创建新实例 #002（不同！），CurrentUserId 是 null
Repository 创建新实例 #003（不同！），CurrentUserId 是 null
// ❌ 问题：Service 和 Repository 拿不到 Controller 设置的值
```

## 实际应用场景

### 场景1：数据库上下文（DbContext）

```csharp
// DbContext 是 Scoped
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

public class OrderService
{
    private readonly ApplicationDbContext _db1;  // 实例 A
    private readonly IRepository _repo;

    public OrderService(ApplicationDbContext db, IRepository repo)
    {
        _db1 = db;  // 实例 A
        _repo = repo;
    }

    public async Task CreateOrder()
    {
        // 使用 _db1 添加订单
        _db1.Orders.Add(new Order());

        // Repository 中也是同一个 DbContext
        await _repo.UpdateProductStock();

        // 同一个 DbContext.SaveChanges() 会同时保存订单和库存更新
        await _db1.SaveChangesAsync();  // ✅ 原子操作
    }
}

public class Repository
{
    private readonly ApplicationDbContext _db2;  // 同一个实例 A

    public Repository(ApplicationDbContext db)
    {
        _db2 = db;  // 和 OrderService 中的是同一个对象
    }

    public async Task UpdateProductStock()
    {
        _db2.Products.Update(product);  // 同一个 DbContext
    }
}
```

### 场景2：请求级缓存

```csharp
public class RequestCache : IRequestCache
{
    private readonly Dictionary<string, object> _cache = new();

    public void Set(string key, object value) => _cache[key] = value;
    public object Get(string key) => _cache.GetValueOrDefault(key);
}

// 注册为 Scoped
builder.Services.AddScoped<IRequestCache, RequestCache>();

public class ProductService
{
    private readonly IRequestCache _cache;

    public async Task<Product> GetProduct(int id)
    {
        var key = $"product_{id}";
        var product = _cache.Get(key);

        if (product == null)
        {
            product = await _db.Products.FindAsync(id);
            _cache.Set(key, product);  // 缓存到当前请求
        }

        return product;
    }
}

// 同一个请求中多次调用
public async Task<IActionResult> GetProductDetails(int id)
{
    var product = await _productService.GetProduct(id);     // 查询数据库
    var related = await _productService.GetProduct(id);     // 从缓存获取，不查数据库
    // 因为用的是同一个 RequestCache 实例
}
```

## 总结

**"Scoped 服务在每个 HTTP 请求内共享同一个实例"** 意味着：

1. ✅ **同一个请求**：无论你在这个请求的哪个位置（Controller、Service、Repository等）注入这个服务，拿到的都是**同一个对象**
2. ✅ **不同请求**：每个请求都有**自己独立的对象**，互不干扰
3. ✅ **请求结束**：对象被自动销毁，释放资源

**记忆口诀**：

- **Singleton**：一个对象，所有人共用（像公司的打印机）
- **Scoped**：一个请求一个对象（像餐厅的每桌一个服务员）
- **Transient**：每次用都新建对象（像一次性餐具）

## Scoped 服务使用场景总结

### 核心特征

- **生命周期**：每个 HTTP 请求创建一次，请求结束销毁
- **共享范围**：同一请求内的所有组件共享同一个实例
- **适用场景**：需要维护请求级别状态或资源的场景

---

## 一、数据访问层

### 1. **Entity Framework DbContext**

```csharp
// 最经典的 Scoped 使用场景
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

// 同一请求内所有操作共享同一个 DbContext
public class OrderService
{
    public OrderService(ApplicationDbContext db) { }  // 实例 A
}

public class ProductService
{
    public ProductService(ApplicationDbContext db) { }  // 同一个实例 A
}
```

**为什么用 Scoped**：

- 维护对象状态跟踪
- 共享数据库连接
- 支持事务（多个操作一起提交）

### 2. **工作单元（Unit of Work）**

```csharp
public interface IUnitOfWork
{
    Task BeginTransactionAsync();
    Task CommitAsync();
    Task RollbackAsync();
}

builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

**为什么用 Scoped**：

- 同一请求内的多个操作共享同一个事务
- 确保数据一致性

---

## 二、用户上下文

### 3. **当前登录用户信息**

```csharp
public interface ICurrentUser
{
    string UserId { get; }
    string UserName { get; }
    bool IsAuthenticated { get; }
}

builder.Services.AddScoped<ICurrentUser, CurrentUser>();
```

**为什么用 Scoped**：

- 每个请求的用户不同
- 整个请求过程需要知道当前用户
- 避免跨请求的用户信息混乱

### 4. **租户信息（多租户系统）**

```csharp
public interface ITenantContext
{
    string TenantId { get; }
    string ConnectionString { get; }
}

builder.Services.AddScoped<ITenantContext, TenantContext>();
```

**为什么用 Scoped**：

- 每个租户有独立的数据库或 schema
- 请求过程中需要保持租户上下文

---

## 三、请求级缓存

### 5. **请求缓存（Request Cache）**

```csharp
public interface IRequestCache
{
    T Get<T>(string key);
    void Set<T>(string key, T value);
}

builder.Services.AddScoped<IRequestCache, RequestCache>();
```

**为什么用 Scoped**：

- 同一请求内避免重复计算/查询
- 不同请求的缓存自动隔离
- 请求结束自动清理

### 6. **性能监控（Per-Request Metrics）**

```csharp
public interface IRequestMetrics
{
    void Record(string metric, long value);
    Dictionary<string, long> GetMetrics();
}

builder.Services.AddScoped<IRequestMetrics, RequestMetrics>();
```

**为什么用 Scoped**：

- 每个请求独立统计
- 请求结束可以记录日志

---

## 四、业务逻辑层

### 7. **请求级验证器**

```csharp
public interface IValidationContext
{
    List<string> Errors { get; }
    bool HasErrors { get; }
    void AddError(string error);
}

builder.Services.AddScoped<IValidationContext, ValidationContext>();
```

**为什么用 Scoped**：

- 收集整个请求过程中的验证错误
- 统一返回给客户端

### 8. **业务流程上下文**

```csharp
public interface IOrderContext
{
    Order CurrentOrder { get; set; }
    List<OrderItem> Items { get; }
    decimal TotalAmount { get; }
}

builder.Services.AddScoped<IOrderContext, OrderContext>();
```

**为什么用 Scoped**：

- 跨多个服务共享订单处理状态
- 避免方法间传递大量参数

---

## 五、安全与审计

### 9. **操作审计**

```csharp
public interface IAuditContext
{
    void Log(string action, object data);
    List<AuditLog> GetLogs();
}

builder.Services.AddScoped<IAuditContext, AuditContext>();
```

**为什么用 Scoped**：

- 收集请求过程中的所有操作记录
- 请求结束统一保存到数据库

### 10. **权限检查缓存**

```csharp
public interface IPermissionCache
{
    bool HasPermission(string permission);
    void CachePermission(string permission, bool has);
}

builder.Services.AddScoped<IPermissionCache, PermissionCache>();
```

**为什么用 Scoped**：

- 同一请求内多次权限检查只查一次数据库
- 避免缓存污染

---

## 六、跨服务数据共享

### 11. **请求级配置**

```csharp
public interface IRequestSettings
{
    string Language { get; }
    string Timezone { get; }
    int PageSize { get; }
}

builder.Services.AddScoped<IRequestSettings, RequestSettings>();
```

**为什么用 Scoped**：

- 从请求头或 JWT 中提取一次
- 整个请求过程使用相同配置

### 12. **API 请求追踪**

```csharp
public interface IRequestTrace
{
    string TraceId { get; }
    DateTime StartTime { get; }
    void AddStep(string step);
}

builder.Services.AddScoped<IRequestTrace, RequestTrace>();
```

**为什么用 Scoped**：

- 记录请求的完整调用链
- 便于问题排查和性能分析

---

## 七、特殊场景

### 13. **多语言/本地化**

```csharp
public interface ILocalizationContext
{
    string CurrentCulture { get; }
    string GetString(string key);
}

builder.Services.AddScoped<ILocalizationContext, LocalizationContext>();
```

**为什么用 Scoped**：

- 每个请求可能使用不同语言
- 语言设置从请求头获取

### 14. **文件上传处理**

```csharp
public interface IUploadContext
{
    List<IFormFile> Files { get; }
    string UploadPath { get; }
    void AddFile(IFormFile file);
}

builder.Services.AddScoped<IUploadContext, UploadContext>();
```

**为什么用 Scoped**：

- 收集上传的文件
- 请求结束统一处理

---

## 八、与 Singleton 和 Transient 的对比

| 场景                     | 推荐生命周期        | 原因                           |
| ------------------------ | ------------------- | ------------------------------ |
| **DbContext**            | Scoped              | 需要事务支持和状态跟踪         |
| **配置读取**             | Singleton           | 配置全局唯一，无状态           |
| **工具类（加密、哈希）** | Singleton           | 无状态，线程安全               |
| **当前用户信息**         | Scoped              | 每个请求用户不同               |
| **临时数据缓存**         | Scoped              | 请求级缓存，自动清理           |
| **日志记录器**           | Singleton           | 全局统一，线程安全             |
| **邮件发送**             | Singleton           | 无状态，可共享                 |
| **业务服务（无状态）**   | Scoped 或 Transient | 看是否需要依赖其他 Scoped 服务 |
| **轻量级工具**           | Transient           | 每次使用新建，简单对象         |

---

## 选择 Scoped 的判断标准

### ✅ 应该使用 Scoped 的情况：

1. **有状态**：需要存储请求相关的数据
2. **需要共享**：同一请求的多个组件需要共享数据
3. **生命周期绑定**：数据生命周期 = 请求生命周期
4. **资源管理**：需要请求结束时自动释放资源

### ❌ 不应该使用 Scoped 的情况：

1. **无状态**：所有请求都能共享（用 Singleton）
2. **非常轻量**：每次使用都新建也无妨（用 Transient）
3. **需要跨请求共享**：用 Singleton + 锁机制

---

## 最佳实践示例

```csharp
// 一个完整的 Scoped 服务示例
public interface IOrderProcessingContext
{
    Order Order { get; set; }
    List<ValidationError> Errors { get; }
    Dictionary<string, object> Cache { get; }
    ICurrentUser CurrentUser { get; }

    void AddError(string code, string message);
    void AddToCache(string key, object value);
    T GetFromCache<T>(string key);
}

public class OrderProcessingContext : IOrderProcessingContext
{
    public Order Order { get; set; }
    public List<ValidationError> Errors { get; } = new();
    public Dictionary<string, object> Cache { get; } = new();
    public ICurrentUser CurrentUser { get; }

    public OrderProcessingContext(ICurrentUser currentUser)
    {
        CurrentUser = currentUser;
    }

    public void AddError(string code, string message)
    {
        Errors.Add(new ValidationError(code, message));
    }

    public void AddToCache(string key, object value) => Cache[key] = value;
    public T GetFromCache<T>(string key) => Cache.TryGetValue(key, out var v) ? (T)v : default;
}

// 注册
builder.Services.AddScoped<IOrderProcessingContext, OrderProcessingContext>();

// 使用 - 多个服务共享同一上下文
public class OrderValidator
{
    public OrderValidator(IOrderProcessingContext context) { }
}

public class PriceCalculator
{
    public PriceCalculator(IOrderProcessingContext context) { }
}

public class InventoryService
{
    public InventoryService(IOrderProcessingContext context) { }
}
```

## 总结

Scoped 服务的核心价值在于：

- 🔄 **请求隔离**：不同请求互不影响
- 📦 **数据共享**：同一请求内无缝共享
- 🧹 **自动清理**：请求结束自动释放资源
- 🎯 **事务支持**：配合 DbContext 实现工作单元模式

**简单记忆**：如果某个数据或资源需要在整个请求过程中保持一致性，就用 Scoped！
