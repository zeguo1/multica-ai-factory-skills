# Multica Issue 执行契约 V3.1

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
- 依赖执行模式：Issue 正文软门禁（当前平台不自动强制）
- 前置依赖：
  - `<Issue key 或文档>` | 必需状态/版本：`<done / Approved / 固定版本>` | 类型：`hard` | 满足证据：`<链接或路径>`
- 人工豁免：无；如有，填写批准人、理由、批准时间、影响范围和证据链接
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
- 所有直接 `hard` 依赖达到声明状态/版本，满足证据可读取；或存在合规人工豁免；
- 依赖节点真实存在，无自依赖、循环、缺失节点和未说明的跨 Project/工作区依赖；
- 验收、测试、证据和停止条件可执行；
- Project 资源、仓库和环境可用；
- 适用门禁已经 PASS。

任一项不满足：未开始的 Issue 保持 `backlog`；执行中发现依赖缺失、失效或候选漂移时，先写结构化阻塞评论，再设为 `blocked`。不要让 Agent 猜测或自行豁免。

## 4. 依赖强制规则

当前 Multica 没有可由本契约依赖的 Issue 硬依赖 API/UI。依赖采用治理软门禁：

1. Issue 正文“依赖与阶段”是依赖关系的权威来源；Stage 仅用于分组和阶段完成唤醒。
2. 依赖类型使用 `hard`。仅作信息参考的关系写入“相关工作”，不得混入前置依赖列表。
3. 要求状态为 `done` 时，`cancelled` 默认不满足；只有合规人工豁免可以替代。
4. 豁免必须记录批准人、理由、批准时间、影响范围、替代控制和证据链接；Agent 不得批准自身工作的豁免。
5. 规划 Agent 负责生成并检查依赖图；审计 Agent 独立检查缺失节点、循环、状态矛盾、版本漂移和豁免证据。
6. 只有编排 Agent 或明确授权的人能把依赖已满足的 Issue 从 `backlog` 提升为 `todo`。执行 Agent 不得自行提升、解除依赖或把阶段完成当作依赖完成。
7. 依赖新增、删除、必需状态/版本变化或豁免撤销后，必须重新检查全部下游 Issue；重新通过前不得继续推进。

## 5. 执行状态

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

## 6. 子 Issue 和阶段

当前阶段可以创建为 `todo`；未来阶段必须停放为 `backlog`：

```bash
multica issue create --title "..." --parent <parent-id> --stage 1 --status todo
multica issue create --title "..." --parent <parent-id> --stage 2 --status backlog
```

若创建时同时分配 Agent，`todo` 会立即触发；需要先审核 Issue 时不要提前设为 `todo`。

阶段关闭后父负责人被唤醒，但依赖不会被平台自动判定或解除。先读取父 Issue、全部相关子 Issue 正文、属性、评论和证据，形成“下游 Issue → 前置依赖 → 必需状态/版本 → 当前证据 → 结论”的检查矩阵；确认无环且逐项满足后再提升：

```bash
multica issue status <child-id> todo
```

## 7. 属性

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

## 8. 评论与交付证据

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

## 9. PR 关联

使用 Issue key 建立链接但不提前关闭：

```text
MUL-123：实现某项功能
```

只有全部完成门禁允许自动完成时才使用：

```text
Closes MUL-123
```

不能通过分支名、记忆或 metadata 推断 PR 状态；读取 `multica issue pull-requests` 的实际结果。
