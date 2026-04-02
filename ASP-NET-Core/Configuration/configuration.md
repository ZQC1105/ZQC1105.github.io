## [Configuration](https://learn.microsoft.com/zh-cn/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)
## 🌟 `builder.Configuration` 的类型与默认配置源

在 ASP.NET Core（.NET 6+）中：

```csharp
var builder = WebApplication.CreateBuilder(args);
var config = builder.Configuration;
```

- **编译时类型**：`IConfiguration`  
- **运行时实际类型**：`IConfigurationRoot`（继承自 `IConfiguration`）

> `IConfigurationRoot` 表示“根配置”，管理多个配置提供程序（Providers），支持重新加载（如文件变更时）。

---

## ✅ 默认自动加载的配置源（按优先级从低到高）

`WebApplication.CreateBuilder(args)` 内部自动创建 `ConfigurationBuilder` 并依次添加以下配置源：

| 顺序 | 配置源                           | 说明                                                     |
| ---- | -------------------------------- | -------------------------------------------------------- |
| 1️⃣    | `appsettings.json`               | 项目根目录下的通用配置文件，所有环境共享。               |
| 2️⃣    | `appsettings.{Environment}.json` | 根据 `ASPNETCORE_ENVIRONMENT` 环境变量加载，覆盖同名键。 |
| 3️⃣    | 用户机密（仅 Development）       | 存储敏感信息，如 API 密钥，不会提交到 Git。              |
| 4️⃣    | 环境变量                         | 以 `DOTNET_` 或 `ASPNETCORE_` 开头，支持 `__` 表示层级。 |
| 5️⃣    | 命令行参数                       | 最高优先级，可覆盖前面所有配置。                         |

---

## 🔁 配置覆盖规则（“后胜”原则）

后添加的提供程序如果包含相同的键，会覆盖前面的值。

**最终优先级从低到高**：

1. `appsettings.json`
2. `appsettings.{Environment}.json`
3. 用户机密（仅 Development）
4. 环境变量
5. 命令行参数

> ✅ 生产环境中可通过环境变量轻松覆盖开发时的 JSON 配置。

---

## 🔧 内部实现简析（简化版）

```csharp
var configBuilder = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{env}.json", optional: true, reloadOnChange: true);

if (env == "Development")
{
    configBuilder.AddUserSecrets(assembly);
}

configBuilder
    .AddEnvironmentVariables() // 加载 ASPNETCORE_ 和 DOTNET_ 前缀变量
    .AddCommandLine(args);     // 添加命令行参数

var configuration = configBuilder.Build(); // 返回 IConfigurationRoot
```

> `builder.Configuration` 就是这个 `configuration` 对象。

---

## ✅ 总结

当你写：

```csharp
var builder = WebApplication.CreateBuilder(args);
var config = builder.Configuration;
```

你得到的是一个已经聚合了多种来源配置的根配置对象，它智能地融合了：

- 静态 JSON 文件（通用 + 环境特定）
- 安全的用户机密（开发时）
- 灵活的环境变量（部署时）
- 动态的命令行参数（调试/测试时）

这种设计让 ASP.NET Core 应用既能本地开发方便，又能无缝部署到各种环境。

> 如有需要，还可通过 `builder.Configuration.Add...()` 添加自定义配置源（如 Azure Key Vault、Consul、自定义 JSON 文件等）。
>
> Windows 系统环境变量就是**全局“键-值”仓库**，操作系统、驱动、应用程序、脚本、容器……谁都可以来读，用来**解耦“可变参数”与“硬编码”**。在桌面、服务、开发、部署整条链路里，它们的核心作用可以浓缩成一句话：

> **“一次写入，处处可用；不改程序，只改环境，就能让软件行为大变。”**

## 🌐 什么是系统环境变量？

| 层级        | 典型变量名                      | 作用域     | 实战用途（举例）                                                                                                     |
| ----------- | ------------------------------- | ---------- | -------------------------------------------------------------------------------------------------------------------- |
| 系统启动    | `PATH`                          | 全局       | 命令行任意目录都能直接敲 `dotnet` `code` `python`，无需写全路径。                                                    |
| 系统启动    | `TEMP` / `TMP`                  | 全局       | 告诉所有程序“临时文件放哪”；浏览器、安装器、WinRAR 都会尊重它。把 `TEMP` 改到 RAM-Disk，机械硬盘立刻少 10 万次写入。 |
| 系统启动    | `OS`  `PROCESSOR_ARCHITECTURE`  | 全局       | MSI 安装包判断 32/64 位、脚本 `.bat` 里 `if "%OS%"=="Windows_NT"` 做分支。                                           |
| 用户登录    | `USERPROFILE`                   | 当前账户   | 微信、Chrome 默认把缓存、配置丢到 `%USERPROFILE%\AppData`；重装系统只要保住这个目录，配置全在。                      |
| 用户登录    | `APPDATA`  `LOCALAPPDATA`       | 当前账户   | 漫游与非漫游配置分离：公司域账号换电脑，桌面软件设置跟着走。                                                         |
| 应用运行    | `ASPNETCORE_ENVIRONMENT`        | 进程继承   | ASP.NET Core / WPF 都认它：`appsettings.Production.json` 是否加载就靠它等于 `Production` 还是 `Development`。        |
| 应用运行    | `DOTNET_ENVIRONMENT`            | 进程继承   | .NET 6+ 通用主机统一识别，用来切换日志级别、连接串、功能开关。                                                       |
| 应用运行    | `JAVA_HOME`                     | 全局/用户  | Maven、Gradle、IDEA、Eclipse 全去找它；升级 JDK 只需改一处，所有构建脚本 0 改动。                                    |
| 脚本/自动化 | `DATE`  `TIME`  `RANDOM`        | 批处理即时 | `.bat` 里直接 `%DATE%` 拼接备份文件名，不用 `for /f` 解析。                                                          |
| 容器/云     | `DOCKER_BUILDKIT`  `KUBECONFIG` | 进程继承   | Windows 版 Docker Desktop 读取后决定是否启用 BuildKit；kubectl 找集群证书路径。                                      |
| 调试排错    | `COMPlus_DbgEnableMiniDump=1`   | 进程继承   | .NET 程序崩溃时自动生成 `*.dmp`，windbg 打开即可看堆栈；无需改代码加 try/catch。                                     |

## 🛠️ 快速实验：一分钟体会“环境变量即配置”

1. Win + S → 输入 `env` → 打开“编辑系统环境变量”。  
2. 在“用户变量”里新建 `MYAPP_COLOR=Red`。  
3. 打开 PowerShell 运行：

```powershell
$env:MYAPP_COLOR          # 输出 Red
```

4. 任意 WPF/控制台程序里：

```csharp
var color = Environment.GetEnvironmentVariable("MYAPP_COLOR");
Console.WriteLine($"主题色：{color}");
```

5. 把 `MYAPP_COLOR` 改成 `Blue`，重启程序——**一行代码不改，界面主题色立刻变**。

## 📌 结论

Windows 系统环境变量就是**操作系统级别的“配置文件”**：

- **对 OS**：决定启动行为、驱动加载、目录结构。  
- **对应用**：零成本实现“多环境、敏感信息、热切换”。  
- **对人**：脚本、CI/CD、容器、排障时，用 `setx` / `set` 就能“隔空”改程序行为，无需重新编译、重新打包、重新安装。