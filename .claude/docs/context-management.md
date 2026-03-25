# 上下文管理

Context 是 Claude Code 会话中最关键的资源。要主动管理它。

## 文件承载状态（主要策略）

**文件才是记忆，对话不是。** 对话是临时的，会被 compact 或丢失。磁盘上的文件在 compaction 与会话崩溃后仍然存在。

### Session 状态文件

将 `production/session-state/active.md` 作为持续更新的 checkpoint。在每次重要里程碑后更新：

- 设计章节已批准并写入文件
- 架构决策已定
- 实现里程碑达成
- 已取得测试结果

状态文件应包含：当前任务、进度 checklist、已做关键决策、正在编辑的文件、以及未决问题。

### Status line 区块（仅 Production+）

当项目处于 Production、Polish 或 Release 阶段时，在 `active.md` 中加入可被 status line 脚本解析的结构化状态块：

```markdown
<!-- STATUS -->
Epic: Combat System
Feature: Melee Combat
Task: Implement hitbox detection
<!-- /STATUS -->
```

- 三个字段（Epic、Feature、Task）均为可选——只写适用的
- 切换工作焦点时更新此块
- Status line 以 breadcrumb 形式显示：`Combat System > Melee Combat > Hitboxes`
- 无活跃工作焦点时删除或清空该块

在任何中断之后（compaction、崩溃、`/clear`），先读状态文件。

### 增量写入文件

在撰写多章节文档（设计文档、架构文档、设定条目）时：

1. 立即创建文件并写好骨架（所有章节标题，正文留空）
2. 在对话中一次只讨论、起草一个章节
3. 章节一经批准就写入文件
4. 每写完一个章节就更新 session 状态文件
5. 写入某章节后，关于该章节的先前讨论可被安全 compact——决策已在文件中

这样 context window 里主要保留*当前*章节的讨论（约 3–5k tokens），而不是整份文档的对话历史（约 30–50k tokens）。

## 主动 Compaction

- 在 context 用量约 **60–70%** 时 **主动 compact**，不要等到顶格才反应
- 在无关任务之间，或连续 **2+** 次纠错失败后使用 **`/clear`**
- **自然的 compaction 时机：** 写完一节到文件后、commit 后、完成一项任务后、开始新话题前
- **聚焦 compaction：** `/compact Focus on [current task] — sections 1-3 are written to file, working on section 4`

## 按任务类型的上下文预算

- 轻量（读/审阅）：启动约 3k tokens
- 中等（实现功能）：约 8k tokens
- 重（多系统 refactor）：约 15k tokens

## Subagent 委派

用 subagent 做调研与探索，保持主 session 干净。Subagent 在各自的 context window 中运行，只返回摘要：

- **使用 subagent** 当需要跨多文件排查、探索陌生代码，或调研会消耗 **>5k tokens** 的文件读取时
- **直接 Read** 当你明确知道要查哪 1–2 个文件时
- Subagent 不继承对话历史——在 prompt 中提供完整 context

## Compaction 说明

当 context 被 compact 时，在摘要中保留：

- 指向 `production/session-state/active.md` 的引用（读它以恢复状态）
- 本会话修改过的文件列表及其用途
- 已做的架构决策及理由
- 活跃的 sprint 任务及当前状态
- Agent 调用及其结果（success/failure/blocked）
- 测试结果（pass/fail 数量、具体失败项）
- 未解决的阻塞或等待用户输入的问题
- 当前任务及进行到哪一步
- 当前文档哪些章节已写入文件、哪些仍在进行中

**Compaction 之后：** 读 `production/session-state/active.md` 以及正在积极编辑的文件以恢复完整 context。文件里才是决策；对话历史是次要的。

## Session 崩溃后恢复

若 session 中断（「prompt too long」）或你开新 session 继续工作：

1. `session-start.sh` hook 会自动检测并预览 `active.md`
2. 读完整状态文件以获取 context
3. 读状态中列出的未完成文件
4. 从下一个未完成的章节或任务继续
