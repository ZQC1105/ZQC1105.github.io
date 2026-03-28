# Cline 中 Hooks：事件驱动的自动化脚本

Hooks 是 Cline 中**最特殊**的机制——它不像 Rules/Skills 那样影响 AI 的"思考"，而是让 Cline 在**特定事件发生时自动执行脚本**，实现**无人值守的自动化**。

---

## 🎯 Hooks 一句话定位

> **"当 X 发生时，自动做 Y"**

- **Rules** = 告诉 AI "怎么想"
- **Skills** = 告诉 AI "怎么做"
- **Workflows** = 定义 "做什么流程"
- **Hooks** = **不经过 AI，直接执行脚本**

---

## 📊 Hooks vs 其他机制

| 维度         | Rules       | Skills      | Workflows   | **Hooks**      |
| :----------- | :---------- | :---------- | :---------- | :------------- |
| **激活方式** | 始终激活    | 按需触发    | 手动调用    | **事件触发**   |
| **AI 参与**  | AI 遵守规则 | AI 学习知识 | AI 执行步骤 | **无 AI 参与** |
| **执行内容** | 约束文本    | 指导文本    | 步骤序列    | **可执行脚本** |
| **响应速度** | 即时        | 即时        | 需 AI 推理  | **毫秒级**     |
| **典型用途** | 规范约束    | 专业指导    | 重复流程    | **自动化操作** |

---

## 🔌 Hooks 支持的事件

Cline 目前支持以下事件（3.71.0+）：

| 事件                   | 触发时机       | 典型用途               |
| :--------------------- | :------------- | :--------------------- |
| **`preCompact`**       | 上下文压缩前   | 保存重要信息、备份对话 |
| **`postCompact`**      | 上下文压缩后   | 恢复状态、记录日志     |
| **`preToolUse`**       | AI 调用工具前  | 权限检查、参数验证     |
| **`postToolUse`**      | AI 调用工具后  | 结果处理、通知         |
| **`userPromptSubmit`** | 用户提交提示词 | 内容过滤、添加上下文   |
| **`stop`**             | 对话停止时     | 清理资源、保存进度     |
| **`notification`**     | 需要通知用户时 | 发送 Slack、邮件提醒   |

---

## 📝 实际示例

### 示例1：任务完成通知

在 `.cline/hooks/` 目录创建 `notification.sh`：

```bash
#!/bin/bash
# 当任务完成时发送通知

echo "✅ Cline 任务已完成！"
osascript -e 'display notification "任务完成" with title "Cline"'
```

配置到 `settings.json`：

```json
{
  "hooks": {
    "notification": [
      {
        "command": "sh",
        "args": [".cline/hooks/notification.sh"]
      }
    ]
  }
}
```

### 示例2：工具调用审计

记录 AI 执行的所有命令：

```python
#!/usr/bin/env python3
# .cline/hooks/audit.py

import json
import sys
from datetime import datetime

# 从 stdin 读取事件数据
data = json.load(sys.stdin)

tool_name = data.get('tool_name')
arguments = data.get('arguments')

with open('cline-audit.log', 'a') as f:
    f.write(f"[{datetime.now()}] {tool_name}: {arguments}\n")

print(json.dumps({"continue": True}))
```

### 示例3：敏感操作拦截

防止 AI 删除重要文件：

```javascript
// .cline/hooks/pre-tool-use.js
const data = JSON.parse(process.stdin.read());

if (data.tool_name === "execute_command") {
  const cmd = data.arguments.command;
  if (cmd.includes("rm -rf /") || cmd.includes("DROP DATABASE")) {
    console.log(
      JSON.stringify({
        continue: false,
        reason: "危险操作被阻止",
      }),
    );
    process.exit(0);
  }
}

console.log(JSON.stringify({ continue: true }));
```

---

## 🏗️ Hooks 架构

```
┌─────────────────────────────────────────────────┐
│                    Cline                         │
│              (事件发射器)                         │
└─────────────────────────────────────────────────┘
                        ↓ 触发事件
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
   preToolUse      postToolUse    notification
        ↓               ↓               ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Hook Script  │ │ Hook Script  │ │ Hook Script  │
│ (bash/py/js) │ │ (bash/py/js) │ │ (bash/py/js) │
└──────────────┘ └──────────────┘ └──────────────┘
        ↓               ↓               ↓
   权限检查        日志记录         发送通知
```

### 关键特性

| 特性         | 说明                                              |
| :----------- | :------------------------------------------------ |
| **语言无关** | 支持 bash、Python、Node.js、Ruby 等任何可执行脚本 |
| **数据传递** | 事件数据通过 **stdin** 传入 JSON 格式             |
| **返回值**   | 通过 **stdout** 返回 JSON 控制 Cline 行为         |
| **异步执行** | 不阻塞主流程（除非返回 `continue: false`）        |

---

## 💡 实战场景

### 场景1：自动格式化保存

```bash
#!/bin/bash
# .cline/hooks/post-tool-use.sh

# 当 AI 修改文件后自动格式化
if [ "$TOOL_NAME" = "write_to_file" ] || [ "$TOOL_NAME" = "replace_in_file" ]; then
    npx prettier --write "$FILE_PATH"
fi
```

### 场景2：Slack 部署通知

```python
# .cline/hooks/post-deploy.py
import requests
import json
import sys

data = json.load(sys.stdin)
if data['tool_name'] == 'execute_command' and 'deploy' in data['arguments']['command']:
    requests.post('https://hooks.slack.com/services/xxx', json={
        'text': f"✅ 部署完成！时间: {data['timestamp']}"
    })
```

### 场景3：上下文压缩前备份

```bash
#!/bin/bash
# .cline/hooks/pre-compact.sh

# 压缩前保存对话历史
cp .cline/conversation.json .cline/backups/conv_$(date +%Y%m%d_%H%M%S).json
```

---

## 🆚 完整对比：五机制全景

| 机制          | 本质           | 触发方式     | AI 参与 | 执行内容       | 典型用途          |
| :------------ | :------------- | :----------- | :------ | :------------- | :---------------- |
| **Rules**     | 行为约束       | 始终激活     | ✅ 遵守 | 文本规则       | 编码规范          |
| **Skills**    | 专家知识       | 按需触发     | ✅ 学习 | 指导文档       | 最佳实践          |
| **Workflows** | 流程编排       | 手动调用     | ✅ 执行 | 步骤序列       | 发布流程          |
| **Hooks**     | **自动化钩子** | **事件触发** | ❌ 无   | **可执行脚本** | **自动通知/检查** |
| **MCP**       | 能力扩展       | 按需调用     | ✅ 调用 | 外部工具       | 数据库查询        |

---

## 🎯 选择指南

### 用 Hooks 当...

- ✅ 需要**自动化**（无人值守）
- ✅ 响应速度要求**毫秒级**
- ✅ 不依赖 AI 判断（确定性逻辑）
- ✅ 需要**系统级操作**（通知、日志、备份）

### 不用 Hooks 当...

- ❌ 需要 AI 理解和判断
- ❌ 逻辑复杂，需要多步推理
- ❌ 需要用户交互确认

---

## 📌 记忆口诀

| 机制          | 口诀                         |
| :------------ | :--------------------------- |
| **Rules**     | "规矩时时在，条条都得改"     |
| **Skills**    | "专家随叫来，用到才打开"     |
| **Workflows** | "流程手动开，步骤跟着来"     |
| **Hooks**     | **"事件自动抓，脚本自己发"** |

---

**总结**：Hooks 是 Cline 的**自动化引擎**，让工具从"需要人盯着"变成"自己会干活"。与 Rules/Skills/Workflows 配合，形成完整的自动化闭环。
