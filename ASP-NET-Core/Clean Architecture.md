## 📊 .NET CQRS 分层架构完整总结

基于我们之前的深入讨论，这里是各个模块的**依赖关系**和**主要功能**的完整总结。

---

## 🏗️ 四层架构总览

```
┌─────────────────────────────────────────────────────────────────┐
│                      YourProject.Api                            │
│                      (Web API 层)                               │
│                  🚀 可执行项目 | ASP.NET Core                    │
├─────────────────────────────────────────────────────────────────┤
│  职责：HTTP 协议处理、参数绑定、响应格式化、路由、认证授权入口    │
│  依赖：Application, Infrastructure                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓ 依赖
┌─────────────────────────────────────────────────────────────────┐
│                   YourProject.Application                       │
│                     (应用层)                                     │
│                  📦 类库 | 业务编排                              │
├─────────────────────────────────────────────────────────────────┤
│  职责：CQRS 实现、用例编排、业务逻辑、数据验证、领域事件发布      │
│  依赖：Domain                                                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓ 依赖
┌─────────────────────────────────────────────────────────────────┐
│                     YourProject.Domain                          │
│                      (领域层)                                    │
│                  📦 类库 | 核心业务                              │
├─────────────────────────────────────────────────────────────────┤
│  职责：实体、值对象、领域规则、业务逻辑、领域事件定义              │
│  依赖：无（纯净层）                                               │
└─────────────────────────────────────────────────────────────────┘
                              ↑ 实现
┌─────────────────────────────────────────────────────────────────┐
│                  YourProject.Infrastructure                     │
│                    (基础设施层)                                  │
│                  📦 类库 | 技术实现                              │
├─────────────────────────────────────────────────────────────────┤
│  职责：数据库访问、外部服务集成、认证授权实现、缓存、文件存储      │
│  依赖：Domain, Application（实现接口）                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📦 各层详细职责与依赖

### 1️⃣ **YourProject.Domain**（领域层）- 纯净核心

| 维度         | 内容                                                                                                                                                             |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **核心职责** | 封装核心业务规则，表达领域概念                                                                                                                                   |
| **主要功能** | • 实体（Entity）定义<br>• 值对象（Value Object）<br>• 聚合根（Aggregate Root）<br>• 领域事件（Domain Event）<br>• 业务异常（Domain Exception）<br>• 仓储接口定义 |
| **依赖关系** | ✅ 无外部依赖（纯净层）                                                                                                                                          |
| **NuGet 包** | 无（或最小依赖）                                                                                                                                                 |
| **典型代码** | `Product.cs`、`Order.cs`、`Money.cs`、`IProductRepository.cs`                                                                                                    |

```csharp
// 示例：领域实体
public class Product : IAggregateRoot
{
    public int Id { get; private set; }
    public string Name { get; private set; }
    public Money Price { get; private set; }

    // 业务规则封装
    public void UpdatePrice(Money newPrice)
    {
        if (newPrice.Amount <= 0)
            throw new DomainException("Price must be positive");

        Price = newPrice;
        AddDomainEvent(new ProductPriceUpdatedEvent(Id, newPrice));
    }
}
```

---

### 2️⃣ **YourProject.Application**（应用层）- 业务编排

| 维度         | 内容                                                                                                                                                              |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **核心职责** | 用例编排，协调领域对象完成业务场景                                                                                                                                |
| **主要功能** | • CQRS 命令/查询定义<br>• MediatR Handler 实现<br>• 数据验证（FluentValidation）<br>• 对象映射（AutoMapper）<br>• 管道行为（Pipeline Behavior）<br>• 领域事件发布 |
| **依赖关系** | ✅ 依赖 Domain<br>❌ 不依赖 Infrastructure                                                                                                                        |
| **NuGet 包** | MediatR、FluentValidation、AutoMapper                                                                                                                             |
| **典型代码** | `CreateProductCommand.cs`、`CreateProductCommandHandler.cs`、`ProductDto.cs`                                                                                      |

```csharp
// 示例：Command Handler
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, CreateProductResponse>
{
    private readonly IProductRepository _repository;  // 接口在 Domain 层
    private readonly IMediator _mediator;

    public async Task<CreateProductResponse> Handle(CreateProductCommand request, ...)
    {
        // 1. 业务逻辑编排
        var product = new Product(request.Name, new Money(request.Price));

        // 2. 持久化
        await _repository.AddAsync(product);

        // 3. 发布领域事件
        await _mediator.Publish(new ProductCreatedNotification(product.Id));

        return new CreateProductResponse(product.Id);
    }
}
```

---

### 3️⃣ **YourProject.Infrastructure**（基础设施层）- 技术实现

| 维度         | 内容                                                                                                                                                 |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **核心职责** | 实现 Domain/Application 定义的接口，处理所有技术细节                                                                                                 |
| **主要功能** | • 数据库访问（EF Core）<br>• 仓储实现<br>• JWT 认证实现<br>• 邮件/短信服务<br>• 缓存服务（Redis）<br>• 文件存储<br>• 消息队列<br>• 第三方 API 客户端 |
| **依赖关系** | ✅ 依赖 Domain（实现接口）<br>✅ 依赖 Application（实现接口）                                                                                        |
| **NuGet 包** | EF Core、Redis、JWT、第三方 SDK                                                                                                                      |
| **典型代码** | `ApplicationDbContext.cs`、`ProductRepository.cs`、`JwtTokenGenerator.cs`                                                                            |

```csharp
// 示例：仓储实现
public class ProductRepository : IProductRepository  // 实现 Domain 接口
{
    private readonly ApplicationDbContext _context;

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }
}
```

---

### 4️⃣ **YourProject.Api**（表示层）- HTTP 适配

| 维度         | 内容                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------- |
| **核心职责** | 处理 HTTP 请求，适配外部世界                                                                                  |
| **主要功能** | • Controllers<br>• 路由配置<br>• 中间件（认证、日志、异常）<br>• Swagger 文档<br>• 配置管理<br>• 依赖注入配置 |
| **依赖关系** | ✅ 依赖 Application<br>✅ 依赖 Infrastructure                                                                 |
| **NuGet 包** | Swashbuckle、Serilog、HealthChecks                                                                            |
| **典型代码** | `ProductsController.cs`、`Program.cs`                                                                         |

```csharp
// 示例：Controller
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;

    [HttpPost]
    public async Task<ActionResult<CreateProductResponse>> Create(CreateProductCommand command)
    {
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetById), new { id = result.Id }, result);
    }
}
```

---

## 🔗 依赖关系图（详细版）

```
                    ┌─────────────────────┐
                    │   YourProject.Api   │
                    │   (Web API 启动)    │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ↓                ↓                ↓
    ┌─────────────────┐  ┌─────────────────┐
    │   Application   │  │ Infrastructure  │
    │   (业务编排)     │  │   (技术实现)     │
    └────────┬────────┘  └────────┬────────┘
             │                    │
             │          ┌─────────┘
             ↓          ↓
    ┌─────────────────────────────────┐
    │           Domain                │
    │        (核心业务)               │
    │   无依赖，纯净层                │
    └─────────────────────────────────┘
```

### 依赖规则（严格遵循）

| 层级               | 可以依赖                    | 不能依赖                         |
| ------------------ | --------------------------- | -------------------------------- |
| **Domain**         | 无                          | Application, Infrastructure, Api |
| **Application**    | Domain                      | Infrastructure, Api              |
| **Infrastructure** | Domain, Application         | Api                              |
| **Api**            | Application, Infrastructure | 无（除了必要的包）               |

---

## 📋 各层主要功能对比表

| 功能                | Domain | Application | Infrastructure | Api |
| ------------------- | ------ | ----------- | -------------- | --- |
| **实体定义**        | ✅     | ❌          | ❌             | ❌  |
| **值对象**          | ✅     | ❌          | ❌             | ❌  |
| **业务规则**        | ✅     | ❌          | ❌             | ❌  |
| **仓储接口**        | ✅     | ❌          | ❌             | ❌  |
| **Command/Query**   | ❌     | ✅          | ❌             | ❌  |
| **MediatR Handler** | ❌     | ✅          | ❌             | ❌  |
| **数据验证**        | ❌     | ✅          | ❌             | ❌  |
| **对象映射**        | ❌     | ✅          | ❌             | ❌  |
| **管道行为**        | ❌     | ✅          | ❌             | ❌  |
| **领域事件发布**    | ❌     | ✅          | ❌             | ❌  |
| **DbContext**       | ❌     | ❌          | ✅             | ❌  |
| **仓储实现**        | ❌     | ❌          | ✅             | ❌  |
| **JWT 实现**        | ❌     | ❌          | ✅             | ❌  |
| **邮件服务**        | ❌     | ❌          | ✅             | ❌  |
| **缓存服务**        | ❌     | ❌          | ✅             | ❌  |
| **HTTP 路由**       | ❌     | ❌          | ❌             | ✅  |
| **认证中间件**      | ❌     | ❌          | ❌             | ✅  |
| **Swagger**         | ❌     | ❌          | ❌             | ✅  |
| **依赖注入配置**    | ❌     | ✅          | ✅             | ✅  |

---

## 🔄 数据流与调用链

### 完整请求流程示例

```
1. HTTP Request
   ↓
2. Api 层：ProductsController
   - 接收 HTTP POST /api/products
   - 绑定参数到 CreateProductCommand
   ↓
3. Api 层调用：_mediator.Send(command)
   ↓
4. Application 层：Pipeline Behavior 链
   - LoggingBehavior（记录日志）
   - ValidationBehavior（验证数据）
   - TransactionBehavior（开启事务）
   ↓
5. Application 层：CreateProductCommandHandler
   - 调用 Domain 层业务逻辑
   - 调用 Infrastructure 层仓储
   ↓
6. Domain 层：Product 实体
   - 执行业务规则验证
   - 生成领域事件
   ↓
7. Infrastructure 层：ProductRepository
   - EF Core 保存到数据库
   - 提交事务
   ↓
8. Application 层：发布领域事件
   - _mediator.Publish(ProductCreatedNotification)
   ↓
9. Infrastructure 层：Notification Handler
   - SendEmailOnProductCreated（发送邮件）
   - LogProductCreation（记录日志）
   ↓
10. 返回响应到 Api 层
    - CreateProductResponse
    ↓
11. HTTP Response 返回客户端
```

---

## 💡 关键设计原则

### 1. **依赖倒置原则（DIP）**

```csharp
// ✅ 正确：Application 依赖 Domain 接口
public class CreateProductCommandHandler : IRequestHandler<...>
{
    private readonly IProductRepository _repository;  // 接口在 Domain
}

// Infrastructure 实现接口
public class ProductRepository : IProductRepository { }
```

### 2. **关注点分离**

- Domain：**What**（业务是什么）
- Application：**How**（业务流程如何编排）
- Infrastructure：**With What**（用什么技术实现）
- Api：**Where**（如何对外暴露）

### 3. **可测试性**

```csharp
// Domain 层：纯单元测试，无依赖
[Test]
public void Product_UpdatePrice_WithNegative_ThrowsException()
{
    var product = new Product("Test", 100);
    Assert.Throws<DomainException>(() => product.UpdatePrice(-10));
}

// Application 层：Mock Domain 接口
var mockRepo = new Mock<IProductRepository>();
var handler = new CreateProductCommandHandler(mockRepo.Object);
```

### 4. **可替换性**

- 更换数据库：只改 Infrastructure 层
- 更换邮件服务：只改 Infrastructure 层
- 更换认证方式：只改 Infrastructure 层

---

## 📊 项目结构最终版

```
YourProject.sln
│
├── src/
│   ├── YourProject.Api/                      # Web API 层
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Program.cs
│   │   └── appsettings.json
│   │
│   ├── YourProject.Application/              # 应用层
│   │   ├── Features/
│   │   │   ├── Products/
│   │   │   │   ├── Commands/
│   │   │   │   │   └── CreateProduct/
│   │   │   │   ├── Queries/
│   │   │   │   │   └── GetProduct/
│   │   │   │   └── Events/
│   │   │   └── Orders/
│   │   ├── Common/
│   │   │   ├── Behaviors/
│   │   │   ├── Exceptions/
│   │   │   └── Interfaces/
│   │   └── DependencyInjection.cs
│   │
│   ├── YourProject.Domain/                   # 领域层
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Events/
│   │   ├── Exceptions/
│   │   └── Interfaces/
│   │
│   └── YourProject.Infrastructure/           # 基础设施层
│       ├── Persistence/
│       │   ├── Configurations/
│       │   ├── Repositories/
│       │   └── ApplicationDbContext.cs
│       ├── Identity/
│       │   └── JwtTokenGenerator.cs
│       ├── Services/
│       │   ├── EmailService.cs
│       │   └── CacheService.cs
│       └── DependencyInjection.cs
│
└── tests/
    ├── YourProject.UnitTests/
    ├── YourProject.IntegrationTests/
    └── YourProject.FunctionalTests/
```

---

## ✅ 总结

| 层次               | 核心职责  | 关键依赖                     | 可替换性             |
| ------------------ | --------- | ---------------------------- | -------------------- |
| **Domain**         | 业务规则  | 无                           | 核心，不可替换       |
| **Application**    | 用例编排  | Domain                       | 业务逻辑，部分可替换 |
| **Infrastructure** | 技术实现  | Domain + Application         | 完全可替换           |
| **Api**            | HTTP 适配 | Application + Infrastructure | 接口层，可替换       |

这种架构实现了：

- ✅ **高内聚低耦合**
- ✅ **清晰的职责边界**
- ✅ **易于单元测试**
- ✅ **技术栈可替换**
- ✅ **团队并行开发**

这是经过业界验证的 Clean Architecture + CQRS 最佳实践，适用于中大型 .NET 项目。
