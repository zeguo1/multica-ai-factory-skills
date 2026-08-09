# Multica Issue 执行契约 V3.0

> 机器标识约定：Multica 状态值、CLI 命令、Skill 名和任务类型枚举保留英文；所有业务字段与填写说明使用中文。

## 1. 读取与写入原则

读取 Issue、Project、子 Issue、属性和评论是准备工作的组成部分。状态、评论、属性、分配、Issue 创建、Task 取消、PR、部署和发布都是写操作，只在当前 Issue 和角色明确授权时执行。

命令形状不明确时先运行 `multica <command> --help`。脚本化读取使用 `--output json`。

常用读取：

```bash
multica issue get <issue-id> --output json
multica issue children <issue-id> --output json
multica issue property list <issue-id> --output json
multica issue pull-requests <issue-id> --output json
multica issue comment list <issue-id> --roots-only --summary --output json
multica project get <project-id> --output json
multica project resource list <project-id> --output json
```

先扫描根评论，再按需读取相关讨论串，避免无边界加载全部讨论。

## 2. Issue 最低正文

```markdown
## 任务类型
Planning（规划）/ Design（设计）/ Development（开发）/ QA（质量验证）/ Audit（审计）/ Release（发布）

## 子类型
Feature（新功能）/ Change（变更）/ Bug Diagnosis（缺陷诊断）/ Bugfix（缺陷修复）/ Debt（技术债）/ Test（测试）/ Audit（审计）/ Deploy（部署）/ Release（发布）/ Remediation（整改）

## 业务目标
- 目标用户：
- 预期业务、用户或运营结果：
- 成功指标：

## 范围
- 范围内：
- 范围外：

## 已批准输入
- PRD：
- UX / Figma：
- 架构 / ADR：
- 父 Issue / 所属 Project：
- 仓库 / 候选版本：

## 依赖与阶段
- 阶段：
- 前置依赖及必需状态：
- 本 Issue 阻塞的后续工作：

## 预期输出
- 交付物：

## 测试要求
- 必须执行的测试与环境：

## 验收标准
1.
2.

## 必需证据
- 命令与结果、PR / Commit、截图或日志、报告：

## 风险与回滚
- 风险：
- 回滚或不适用原因：

## 状态权限
- 当前角色允许设置的最高状态：

## 停止条件
- 缺少输入、存在冲突、权限不足、安全风险或外部阻塞时的处理：
```

不适用字段写“不适用”并说明原因。

## 3. 就绪检查

`todo` 表示已就绪。进入前必须确认：

- 任务类型、负责人、范围和输出明确；
- 输入基线已经批准且版本固定；
- 直接依赖完成或有人工豁免；
- 验收、测试、证据和停止条件可执行；
- Project 资源、仓库和环境可用；
- 适用门禁已经 PASS。

任一项不满足，将 Issue 留在 `backlog` 或设为 `blocked`。不要让 Agent 猜测。

## 4. 执行状态

开始执行自身负责的已就绪 Issue：

```bash
multica issue status <issue-id> in_progress
```

交付产物后：

```bash
multica issue status <issue-id> in_review
```

阻塞时先写结构化评论，再设状态：

```text
## 阻塞说明
- 事实：
- 影响：
- 责任人：
- 解除条件：
- 下次检查条件：
```

```bash
multica issue status <issue-id> blocked
```

状态改变不会停止已经运行的 Task；需要停止时必须取消具体 Task，且当前 Issue 必须授权该动作。

## 5. 子 Issue 和阶段

当前阶段可以创建为 `todo`；未来阶段必须停放为 `backlog`：

```bash
multica issue create --title "..." --parent <parent-id> --stage 1 --status todo
multica issue create --title "..." --parent <parent-id> --stage 2 --status backlog
```

若创建时同时分配 Agent，`todo` 会立即触发；需要先审核 Issue 时不要提前设为 `todo`。

阶段关闭后父负责人被唤醒。先运行 `multica issue children <parent-id> --output json`，核对下一阶段每个 Issue 的显式依赖，再逐项提升：

```bash
multica issue status <child-id> todo
```

## 6. 属性

推荐使用中文工作区属性名：

- `任务类型`
- `子类型`
- `门禁`
- `QA 结论`
- `审计结论`
- `人工批准`
- `基线版本`
- `候选版本`

任务类型选项仍建议使用稳定机器值：`Planning`、`Design`、`Development`、`QA`、`Audit`、`Release`，并在属性说明中提供中文释义。

Agent 只能使用已存在的属性定义。写入前读取合法选项：

```bash
multica property list --output json
multica issue property list <issue-id> --output json
```

缺少属性时在评论中请求 owner/admin 创建，不用隐蔽 metadata 代替人应看到的门禁状态。

## 7. 评论与交付证据

结果评论至少包含：

```markdown
## 执行结果
- 结果：
- 文件或产物：
- 候选版本：

## 验证结果
- 命令或检查 → 结果

## 验收证据
- 验收标准 → 证据

## 风险与回滚
- 风险：
- 回滚：

## 下一门禁
- 必需评审人、QA、Audit 或人工批准：
```

如果没有代码变更或 Issue 明确要求本地交付，说明为何没有 PR。存在 PR 时提供 URL 和候选 Commit。

## 8. PR 关联

使用 Issue key 建立链接但不提前关闭：

```text
MUL-123：实现某项功能
```

只有全部完成门禁允许自动完成时才使用：

```text
Closes MUL-123
```

不能通过分支名、记忆或 metadata 推断 PR 状态；读取 `multica issue pull-requests` 的实际结果。
