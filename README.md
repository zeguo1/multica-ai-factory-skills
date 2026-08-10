# Multica AI 软件工厂 Skills

这是一组面向 Multica 的中文 AI 软件工厂治理 Skill，当前基线为 V3.2，覆盖需求编排、规划、设计、开发、质量验证、独立审计和发布。

## V3.2 自动编排与依赖门禁

当前 Multica 不会通过可用 API/UI 自动强制 Issue 依赖。V3.2 以 Issue 正文“依赖与阶段”为权威来源，由编排 Agent 执行软门禁：

- 每条 `hard` 依赖声明前置 Issue key/文档、必需状态/版本和满足证据；
- 依赖图必须无自依赖、循环和缺失节点；
- 未满足时保持 `backlog`，执行中发现失效时设为 `blocked`；
- 只有编排 Agent 或明确授权的人可以在 Ready 检查后提升为 `todo`；
- `cancelled` 默认不满足要求为 `done` 的依赖；人工豁免必须留下完整证据。

V3.2 同时新增 `Coordination` 任务类型和显式自动推进：

- 仅当 Project 或协调父 Issue 声明 `自动推进模式：autonomous` 时启用；未声明时保持 `manual`，不会静默改变既有项目；
- Planning、Design、Development、QA、Audit 和 Release 交付后，在最终评论中提及编排 Agent；
- 编排 Agent 启动下一门禁，按 PASS 关闭已满足节点并推进下游，按 FAIL 退回原责任角色整改；
- `in_review` 表示等待下一门禁，不默认表示需要人审核；
- 业务/范围选择、数据与合规决定、重大风险、生产发布和不可逆操作仍由人批准。

## Skill 列表

| Skill | 作用 | Multica URL 导入地址 |
| --- | --- | --- |
| `multica-ai-factory-orchestrator` | Coordination 父 Issue、任务路由、接棒与门禁闭环 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-orchestrator` |
| `multica-ai-factory-planning` | 仓库调查、PRD、架构与任务图规划 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-planning` |
| `multica-ai-factory-design` | UX、UI、状态与设计基线 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-design` |
| `multica-ai-factory-execution` | 受治理的开发实现与自测 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-execution` |
| `multica-ai-factory-qa` | 固定候选版本的独立质量验证 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-qa` |
| `multica-ai-factory-audit` | 基线、交付物与证据的独立审计 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-audit` |
| `multica-ai-factory-release` | 授权环境中的部署、验证与回滚 | `https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-release` |

## 导入方法

在 Multica 的 **Skills → 新建 Skill → 从 URL 导入** 中粘贴上表对应地址。每个 URL 导入一个 Skill。

也可以使用 CLI：

```bash
multica skill import --url https://github.com/zeguo1/multica-ai-factory-skills/tree/main/skills/multica-ai-factory-planning
```

如果工作区已经存在同名 Skill，请根据意图选择 `overwrite`、`rename` 或 `skip` 冲突策略。

## 语言约定

用户可见说明、报告模板和界面元数据使用中文。Multica 状态值、CLI 命令、Skill 标识及 `Planning`、`Design`、`PASS`、`FAIL` 等机器字段保留英文，避免破坏自动识别和流程约束。

## 目录结构

每个 Skill 目录包含：

```text
SKILL.md
agents/openai.yaml
references/governance-baseline.md
references/multica-issue-contract.md
```
