---
title: " 仓储模式"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - 设计模式
  - ASP.NET CORE
---

**`Repository`（仓储模式）是一种经典的设计模式（Design Pattern）**，它属于 **架构设计模式** 中的一种，广泛应用于企业级应用开发中，特别是在 .NET 和 ASP.NET Core 项目中非常常见。

---

### ✅ 正确说法：
> **“Repository 是一种设计模式”**，更准确地说，它是 **企业应用架构模式（Enterprise Application Architecture Pattern）** 的一种。

---

### 📚 一、什么是设计模式？

**设计模式（Design Pattern）** 是解决某一类常见软件设计问题的可复用、经过验证的解决方案模板。它不是具体的代码库，而是一种“套路”或“最佳实践”。

常见的设计模式分类：
1. **创建型模式**：如 `Singleton`、`Factory`、`Builder`
2. **结构型模式**：如 `Adapter`、`Decorator`、`Proxy`
3. **行为型模式**：如 `Observer`、`Command`、`Strategy`
4. **架构型模式**：如 `Repository`、`Unit of Work`、`Service Layer`、`MVC`

> 🔹 `Repository Pattern` 属于 **架构型模式**，用于组织代码结构和分层。

---

### 🏗️ 二、仓储模式（Repository Pattern）详解

#### 🎯 目的：
在**领域实体（Domain Entities）** 和 **数据存储（如数据库）** 之间建立一个**隔离层**，使得业务逻辑不需要直接了解数据是如何存储或检索的。

#### 🧱 核心思想：
- 把数据访问逻辑封装起来。
- 提供类似“集合（Collection）”的 API，例如：`GetByIdAsync()`、`ListAsync()`、`AddAsync()`。
- 上层代码（如 Service 或 Controller）只依赖接口 `IRepository`，而不依赖 EF Core 或数据库。

#### 📐 结构示意：

```csharp
// 接口抽象
public interface IBrainstormSessionRepository
{
    Task<List<BrainstormSession>> ListAsync();
    Task<BrainstormSession> GetByIdAsync(int id);
    Task AddAsync(BrainstormSession session);
}

// 实现（依赖 EF Core）
public class BrainstormSessionRepository : IBrainstormSessionRepository
{
    private readonly AppDbContext _context;
    public BrainstormSessionRepository(AppDbContext context) => _context = context;

    public async Task<List<BrainstormSession>> ListAsync()
    {
        return await _context.Sessions.ToListAsync();
    }

    // 其他实现...
}
```

上层代码只依赖 `IBrainstormSessionRepository`，完全不知道底层是 SQL Server、SQLite 还是内存列表。

---

### 🌟 三、为什么说它是“模式”？

因为它满足设计模式的几个关键特征：

| 特征 | 说明 |
|------|------|
| ✅ 可复用 | 可以在多个实体（User、Order、Product）上重复使用 |
| ✅ 解决通用问题 | “如何解耦业务逻辑与数据访问？” |
| ✅ 经过验证 | 被 Martin Fowler、微软、DDD 社区广泛推荐 |
| ✅ 有标准结构 | 接口 + 实现 + 注入使用 |

---

### 📖 四、相关模式（常搭配使用）

| 模式 | 说明 |
|------|------|
| **Unit of Work** | 管理多个 Repository 的事务一致性，通常封装 `DbContext.SaveChanges()` |
| **Service Layer** | 在 Repository 之上封装业务逻辑 |
| **Dependency Injection** | 将 Repository 注入到 Controller 或 Service 中 |

例如：

```csharp
public class BrainstormService
{
    private readonly IBrainstormSessionRepository _sessionRepo;
    private readonly IUserRepository _userRepo;

    public BrainstormService(
        IBrainstormSessionRepository sessionRepo,
        IUserRepository userRepo)
    {
        _sessionRepo = sessionRepo;
        _userRepo = userRepo;
    }

    public async Task StartSessionAsync(...)
    {
        // 使用多个 Repository 完成一个业务操作
    }
}
```

---

### 📚 五、官方与权威来源支持

1. **Martin Fowler 的《企业应用架构模式》**（Patterns of Enterprise Application Architecture）  
   👉 首次系统性定义了 Repository 模式  
   🔗 [https://www.martinfowler.com/eaaCatalog/repository.html](https://www.martinfowler.com/eaaCatalog/repository.html)

2. **微软文档：在 ASP.NET Core 中实现 Repository 和 Unit of Work 模式**  
   🔗 [https://learn.microsoft.com/en-us/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns](https://learn.microsoft.com/en-us/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application)

3. **领域驱动设计（DDD）** 中也大力推崇 Repository 模式，用于聚合根（Aggregate Root）的持久化。

---

### ✅ 总结

| 问题 | 回答 |
|------|------|
| `Repository` 是一种设计模式吗？ | ✅ **是的！** 它是企业级应用中广泛使用的**架构设计模式** |
| 它的作用是什么？ | 解耦业务逻辑与数据访问，提升可测试性、可维护性 |
| 是否推荐在 ASP.NET Core 中使用？ | ✅ **推荐**，尤其是在中大型项目或需要单元测试的场景 |
| 命名带 `Repository` 是否合理？ | ✅ 非常合理，符合行业惯例和语义清晰原则 |

---

💡 所以你完全可以说：
> “我们在项目中使用了 **Repository 设计模式**，通过 `IBrainstormSessionRepository` 接口抽象数据访问，实现了解耦和可测试性。”

这是一种**专业、规范、可维护**的开发方式。