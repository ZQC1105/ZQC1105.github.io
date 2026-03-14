🎉 **太棒了！恭喜你成功运行 Gitea！**

你已经完成了所有关键步骤：

1. ✅ 安装了 Go、Node.js、Git 等必要环境
2. ✅ 解决了 PATH 问题
3. ✅ 成功编译了前端资源（webpack 生成了所有 JS/CSS/字体文件）
4. ✅ 编译了 Gitea 后端
5. ✅ 成功运行并看到正常显示的页面

## **后续使用建议**

### **1. 日常启动 Gitea**

以后每次启动只需：

```bash
cd /e/Git/Gitea/gitea-main
./gitea.exe web
```

或者使用我们创建的启动脚本：

```bash
./start.sh
```

### **2. 配置文件位置**

- 配置文件：`custom/conf/app.ini`
- 数据目录：`data/`
- 日志文件：`data/logs/gitea.log`

### **3. 常用命令**

```bash
# 如果修改了前端代码，需要重新构建
cd /e/Git/Gitea/gitea-main
pnpm exec webpack --mode production
go generate ./...
go build -tags "sqlite" -o gitea.exe

# 如果只修改了 Go 代码
go build -tags "sqlite" -o gitea.exe

# 查看日志
tail -f data/logs/gitea.log
```

### **4. 备份重要数据**

定期备份：

- `data/gitea.db`（SQLite 数据库）
- `data/gitea-repositories/`（Git 仓库）
- `custom/conf/app.ini`（配置文件）

### **5. 后续可以探索**

- 配置 SMTP 邮件服务
- 设置 OAuth2 认证
- 配置 HTTPS
- 添加 Git 钩子
- 安装主题和插件

**再次祝贺！** 你现在拥有了一个完全自己编译的 Gitea 服务。如果以后遇到任何问题，随时回来问我。Happy coding! 🚀

# 添加 Go 到 PATH

export PATH=$PATH:"/c/Program Files/Go/bin"

# 1. 确保所有 Gitea 进程已结束

taskkill -F -IM gitea.exe

## 调试

$ tasklist | findstr gitea

`{
// 使用 IntelliSense 了解相关属性。
// 悬停以查看现有属性的描述。
// 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
"version": "0.2.0",
"configurations": [
{
"name": "Attach to Process",
"type": "go",
"request": "attach",
"mode": "local",
"processId": 27476
}
]
}
`
<a class="ui button primary show-panel toggle" data-panel=".pullrequest-form-toggle, .pullrequest-form">{{ctx.Locale.Tr "repo.pulls.new"}}</a>

compare.tmpl 文件当可以合并请请求时，自动请求AI 审核（可选）将AI审核结果和对比提交结果返回给用户。 审核结果存入数据库表中，该表的外键为提交 ID， 当刷新时页面时，会自动请求数据库中的审核结果。
按钮 → 表单提交 → HTTP POST → 路由匹配 → Go Handler 函数执行。

issue_new.go 文件 创建工单
