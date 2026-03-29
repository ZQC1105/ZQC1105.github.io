## `app.UseAuthentication()` 的核心作用

**一句话概括：**
> 它读取请求中的凭证（如 JWT Token、Cookie），验证后把用户身份信息设置到 `HttpContext.User` 上，供后续的授权中间件和业务代码使用。

## 具体做了什么

```csharp
app.UseAuthentication();
```

这个中间件执行以下步骤：

### 1. **读取凭证**
从请求中提取身份凭证：
- JWT：从 `Authorization: Bearer <token>` 请求头读取
- Cookie：从请求的 Cookie 中读取
- API Key：从请求头或查询参数读取

### 2. **验证凭证**
调用注册的认证处理器（Authentication Handler）验证凭证：
- 检查 JWT 签名是否有效
- 验证 Token 是否过期
- 检查颁发者、受众等配置

### 3. **创建用户身份**
验证成功后，构建 `ClaimsPrincipal` 对象：
```csharp
// 伪代码示例
var identity = new ClaimsIdentity(new[]
{
    new Claim(ClaimTypes.NameIdentifier, "123"),
    new Claim(ClaimTypes.Name, "zhangsan"),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim("Permission", "CanEdit"),
    new Claim("Department", "IT")
}, "jwt");

var user = new ClaimsPrincipal(identity);
```

### 4. **赋值给 HttpContext.User**
```csharp
context.User = user;  // 设置当前请求的用户身份
```

### 5. **如果没有凭证或验证失败**
- 不会报错
- `HttpContext.User` 为空（或匿名用户）
- 继续执行下一个中间件

## 实际效果演示

### 带 Token 的请求
```csharp
// 请求头：Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

app.UseAuthentication();  // 执行后，解析并验证 Token

app.Use(async (context, next) =>
{
    // 现在可以访问用户信息了
    var userName = context.User.Identity.Name;  // "zhangsan"
    var isAdmin = context.User.IsInRole("Admin");  // true
    
    await next();
});
```

### 不带 Token 的请求
```csharp
app.UseAuthentication();  // 发现没有凭证，什么都不做

app.Use(async (context, next) =>
{
    var isAuthenticated = context.User.Identity.IsAuthenticated;  // false
    var userName = context.User.Identity.Name;  // null
    
    await next();
});
```

## 为什么必须调用它？

### ❌ 不调用 UseAuthentication()
```csharp
app.UseRouting();
app.UseAuthorization();  // 没有认证中间件，User 始终为空
app.MapControllers();

[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    var user = User;  // 永远为空，Authorize 过滤器会直接返回 401
    return Ok(user?.Identity?.Name);
}
```
- `[Authorize]` 特性看到 `User` 为空 → 返回 401
- 即使带了正确的 Token，也不会被解析

### ✅ 调用 UseAuthentication()
```csharp
app.UseRouting();
app.UseAuthentication();  // 解析 Token，设置 User
app.UseAuthorization();   // 基于 User 做授权
app.MapControllers();
```

## 与授权中间件的配合

```csharp
app.UseAuthentication();  // 职责：设置 User
app.UseAuthorization();   // 职责：基于 User 决定是否允许访问
```

**授权中间件的工作：**
```csharp
// Authorization 中间件伪代码
if (需要授权 && !context.User.Identity.IsAuthenticated)
{
    context.Response.StatusCode = 401;  // Unauthorized
    return;  // 短路，不执行后续 Action
}

if (需要特定角色/策略 && !满足条件)
{
    context.Response.StatusCode = 403;  // Forbidden
    return;
}

await _next();  // 继续执行
```

## 配置多个认证方案

```csharp
builder.Services.AddAuthentication()
    .AddJwtBearer()      // JWT 认证
    .AddCookie();        // Cookie 认证

app.UseAuthentication();  // 尝试所有注册的方案
```

认证中间件会按顺序尝试每个认证方案，直到其中一个成功：
1. 尝试 JWT：有 Bearer Token → 验证成功 → 使用 JWT 身份
2. 如果 JWT 失败，尝试 Cookie：有 Cookie → 验证成功 → 使用 Cookie 身份
3. 都失败 → User 为空

## 在 Controller 中访问

```csharp
[HttpGet("me")]
[Authorize]  // 确保已认证
public IActionResult GetMe()
{
    // 方式1：通过 User 属性
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var userName = User.Identity.Name;
    
    // 方式2：通过 HttpContext
    var user = HttpContext.User;
    
    // 方式3：注入（需要配置）
    var principal = User;
    
    return Ok(new
    {
        Id = userId,
        Name = userName,
        Roles = User.Claims.Where(c => c.Type == ClaimTypes.Role).Select(c => c.Value)
    });
}
```

## 总结

`app.UseAuthentication()` 的本质是：

> **将 HTTP 请求中的无状态凭证（Token/Cookie），转换为有状态的身份对象（ClaimsPrincipal），并附加到当前请求上下文中。**

没有它：
- `[Authorize]` 无法工作
- `User` 属性永远为空
- 业务代码无法获取登录用户信息

有了它：
- 认证和授权分离，各司其职
- 业务代码可以通过 `User` 获取身份信息
- 支持多种认证方案灵活组合

这就是为什么它必须在 `UseAuthorization()` 之前调用的根本原因。