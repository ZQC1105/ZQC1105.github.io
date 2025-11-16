
DevOps（Development + Operations）是一组 过程、方法与系统的统称，核心目标是：  
> 让开发（Dev）与运维（Ops）高效协作，通过自动化、持续交付与快速反馈，缩短从需求到上线的周期，同时保证质量与稳定性。

---

## 一句话记忆
“代码一提交，几分钟内自动编译、测试、部署到生产，出问题还能快速回滚”——这就是 DevOps 想做到的终极状态。

---

### DevOps 不是工具，而是一套 文化 + 实践 + 工具 的组合拳

1. 文化  
   - 开发、测试、运维、安全 共享责任（You build it, you run it）。  
   - 小步快跑、频繁交付、允许失败但快速恢复。

2. 实践（CALMS 模型）  
   - **C**ulture 文化  
   - **A**utomation 自动化（CI/CD、IaC）  
   - **L**ean 精益（减少浪费、持续改进）  
   - **M**easurement 度量（MTTR、Lead Time、Deployment Frequency）  
   - **S**haring 共享（知识、工具、复盘）

3. 工具链示例（常见组合）  
   - 代码：Git、GitLab、GitHub  
   - 构建：Maven、Gradle、npm、Docker  
   - CI/CD：Jenkins、GitLab CI、GitHub Actions、Azure DevOps、Argo CD  
   - 测试：JUnit、Selenium、Postman、SonarQube  
   - 部署：Kubernetes、Helm、Terraform、Ansible  
   - 监控：Prometheus、Grafana、ELK、Jaeger  
   - 协作：Jira、Trello、Slack、Microsoft Teams

---

### 典型流水线（Pipeline）长什么样
```
开发者 push 代码
      ↓
WebHook 触发 CI 服务器
      ↓
自动编译 → 单元测试 → 静态扫描 → 打 Docker 镜像
      ↓
镜像推入仓库（Harbor/ECR/ACR）
      ↓
CD 触发：自动部署到测试环境 → 接口/UI 自动化测试
      ↓
人工审批（可选）
      ↓
自动部署到生产 → 冒烟 → 监控告警 → 自动回滚（若失败）
```

---

### 衡量 DevOps 成熟度的 4 个关键指标（DORA）

| 指标                 | 优秀值           | 意义     |
| -------------------- | ---------------- | -------- |
| 部署频率             | 按需（每天多次） | 交付能力 |
| 变更前置时间         | <1 小时          | 响应速度 |
| 失败率               | <15 %            | 质量     |
| 故障恢复时间（MTTR） | <1 小时          | 稳定性   |

---

### DevOps 的延伸
- **DevSecOps**：把安全扫描（SAST/DAST/镜像扫描）嵌入流水线，实现“安全左移”。  
- **GitOps**：以 Git 为单一事实来源，Kubernetes 集群自动同步 YAML 变更。  
- **AIOps**：用 AI 分析日志/指标，自动定位故障、弹性扩缩容。

---

### 快速上手的 5 个步骤
1. 代码放 Git，主干开发 + Pull Request。  
2. 用 Jenkins/GitHub Actions 跑通 自动编译 + 单元测试。  
3. 把环境做成 Docker 镜像，同一份镜像逐级晋升（测试→预发→生产）。  
4. 用 Terraform/Ansible 把基础设施代码化（IaC），不再手工装服务器。  
5. 上线 Prometheus + Grafana 做监控告警，失败能自动回滚。

---

### 一句话总结
DevOps = “文化”让团队愿意一起快，“实践”告诉团队怎么快，“工具”帮团队把快变成自动化。
```