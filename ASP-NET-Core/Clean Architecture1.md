你整理的分层依赖和功能对照表已经非常清晰！为了更符合 **DDD + Clean Architecture / Onion Architecture** 的最佳实践，我们可以做以下 **优化建议**：

---

## ✅ 优化目标

1. **修正“领域事件发布”的归属**（应在 Infrastructure）
2. **明确 Application 层不应直接依赖 Infrastructure**
3. **细化“依赖注入配置”的归属**
4. **统一术语，增强可读性**

---

## 🔧 优化后的依赖规则表

| 层级                   | 可以依赖                                 | 不能依赖                             |
| ---------------------- | ---------------------------------------- | ------------------------------------ |
| **Domain**             | —（仅 .NET 基础库，如 `System`）         | Application、Infrastructure、Api     |
| **Application**        | Domain                                   | Infrastructure、Api                  |
| **Infrastructure**     | Domain、Application（⚠️ 有争议，见说明） | Api                                  |
| **Api (Presentation)** | Application                              | Infrastructure（应通过接口间接使用） |

> 💡 **关键说明**：
>
> - **理想情况下，Infrastructure 不应依赖 Application**。  
>   更干净的做法是：**Infrastructure 只依赖 Domain**，而 Application 通过 DI 注入 Infrastructure 的实现。
> - 但在实际项目中（尤其使用 MediatR），`Infrastructure` 常需注册 Handler（位于 Application），导致它“引用”了 Application 程序集。  
>   这属于 **“部署依赖”而非“编译依赖”**，可通过 DI 容器解耦。

✅ **推荐做法**：  
让 `Infrastructure` **只依赖 Domain**，在 `Api` 层完成所有服务注册（包括 MediatR 和 Handler）。

---

## 📊 优化后的各层功能对比表

| 功能                                     | Domain | Application | Infrastructure |     Api      |
| ---------------------------------------- | :----: | :---------: | :------------: | :----------: |
| **实体定义**                             |   ✅   |     ❌      |       ❌       |      ❌      |
| **值对象**                               |   ✅   |     ❌      |       ❌       |      ❌      |
| **业务规则（聚合行为）**                 |   ✅   |     ❌      |       ❌       |      ❌      |
| **仓储接口（IRepository）**              |   ✅   |     ❌      |       ❌       |      ❌      |
| **领域事件定义**                         |   ✅   |     ❌      |       ❌       |      ❌      |
| **Command / Query DTOs**                 |   ❌   |     ✅      |       ❌       |      ❌      |
| **MediatR Handlers**                     |   ❌   |     ✅      |       ❌       |      ❌      |
| **应用服务（用例编排）**                 |   ❌   |     ✅      |       ❌       |      ❌      |
| **输入验证（FluentValidation）**         |   ❌   |     ✅      |       ❌       |      ❌      |
| **对象映射（Mapper Profile）**           |   ❌   |     ✅      |       ❌       |      ❌      |
| **管道行为（Pipeline Behaviors）**       |   ❌   |     ✅      |       ❌       |      ❌      |
| **领域事件发布机制**                     |   ❌   |     ❌      |       ✅       |      ❌      |
| **DbContext**                            |   ❌   |     ❌      |       ✅       |      ❌      |
| **仓储实现（Repository）**               |   ❌   |     ❌      |       ✅       |      ❌      |
| **外部服务适配器**（邮件、短信、支付等） |   ❌   |     ❌      |       ✅       |      ❌      |
| **认证/授权实现**（JWT、OAuth）          |   ❌   |     ❌      |       ✅       |      ❌      |
| **缓存实现（Redis, MemoryCache）**       |   ❌   |     ❌      |       ✅       |      ❌      |
| **HTTP 路由 & Controllers**              |   ❌   |     ❌      |       ❌       |      ✅      |
| **认证/授权中间件**                      |   ❌   |     ❌      |       ❌       |      ✅      |
| **Swagger / OpenAPI**                    |   ❌   |     ❌      |       ❌       |      ✅      |
| **依赖注入容器配置**                     |   ❌   | ⚠️（可放）  |   ⚠️（可放）   | ✅（主入口） |

> ✅ **最佳实践建议**：
>
> - **DI 配置集中在 `Api` 层**（`Program.cs`），避免跨层注册混乱。
> - `Application` 和 `Infrastructure` 可提供 **扩展方法** 供 `Api` 调用，例如：
>
>   ```csharp
>   // 在 Application 层
>   public static IServiceCollection AddApplicationServices(this IServiceCollection services)
>   {
>       services.AddMediatR(...);
>       return services;
>   }
>
>   // 在 Infrastructure 层
>   public static IServiceCollection AddInfrastructureServices(this IServiceCollection services, IConfiguration config)
>   {
>       services.AddScoped<IProductRepository, ProductRepository>();
>       services.AddDbContext<ApplicationDbContext>(...);
>       return services;
>   }
>
>   // 在 Api 层 Program.cs
>   builder.Services.AddApplicationServices();
>   builder.Services.AddInfrastructureServices(builder.Configuration);
>   ```

---

## 🎯 总结：关键修正点

| 原内容                          | 优化后               | 理由                                                    |
| ------------------------------- | -------------------- | ------------------------------------------------------- |
| “领域事件发布” → Application    | → **Infrastructure** | 发布逻辑依赖 `DbContext` 和 `IMediator`，属于基础设施   |
| Infrastructure 依赖 Application | **尽量避免**         | 应通过接口和 DI 解耦，保持 Infrastructure 只依赖 Domain |
| DI 配置分散                     | **集中到 Api 层**    | 入口项目负责组装所有模块，符合 Composition Root 原则    |

---

这样优化后，你的架构将更加 **清晰、可测试、可维护**，并严格遵循 **DDD 和 Clean Architecture 的核心原则**：**内层不依赖外层，依赖通过抽象（接口）注入**。
