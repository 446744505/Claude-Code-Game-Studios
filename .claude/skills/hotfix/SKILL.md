---
name: hotfix
description: "紧急修复工作流：在完整审计轨迹下绕过常规冲刺流程。创建 hotfix 分支、跟踪审批，并确保修复被正确 backport。"
argument-hint: "[bug-id 或描述]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---
当本技能被调用时：

> **仅限显式调用**：本技能仅应在用户用 `/hotfix` 显式请求时运行。请勿仅因上下文匹配而自动调用。

1. **评估紧急程度** — 阅读缺陷描述或 ID。判定严重级别：
   - **S1（致命）**：游戏无法游玩、数据丢失、安全漏洞 — 立即热修
   - **S2（重大）**：重要功能损坏、存在变通办法 — 24 小时内热修
   - 若为 S3 或更低，建议走常规缺陷修复流程

2. **创建热修记录** 于 `production/hotfixes/hotfix-[date]-[short-name].md`：

   ```markdown
   ## Hotfix: [简短描述]
   日期: [日期]
   严重级别: [S1/S2]
   报告人: [发现者]
   状态: IN PROGRESS

   ### 问题
   [说明损坏内容与对玩家的影响]

   ### 根因
   [调查过程中填写]

   ### 修复
   [实现过程中填写]

   ### 测试
   [测了什么、如何测]

   ### 审批
   - [ ] 修复已由 lead-programmer 审阅
   - [ ] 回归测试通过（qa-tester）
   - [ ] 发布已批准（producer）

   ### 回滚方案
   [若修复引发新问题，如何回退]
   ```

3. **创建 hotfix 分支**（若已初始化 git）：
   ```
   git checkout -b hotfix/[short-name] [release-tag-or-main]
   ```

4. **调查并实现修复** — 聚焦能解决问题的最小改动。热修中请勿重构、整理代码或顺带加功能。

5. **验证修复** — 对受影响系统运行针对性测试；检查相邻系统是否回归。

6. **更新热修记录**，填入根因、修复细节与测试结果。

6b. **收集审批** — 使用 Task 工具请求签核：
   - `subagent_type: lead-programmer` — 审查修复正确性与副作用
   - `subagent_type: qa-tester` — 对受影响系统做针对性回归测试
   - `subagent_type: producer` — 批准上线时间与沟通方案

7. **输出摘要**，包含：严重级别、根因、已做修复、测试状态、部署前仍缺哪些审批。

### 规则
- 热修必须是解决问题的**最小**改动 — 不做清理、不重构、不搞「顺便改一下」
- 每次热修部署前必须文档化回滚方案
- hotfix 分支需合并到**发布分支**与**开发分支**
- 所有热修须在 48 小时内做事后复盘
- 若修复复杂到预计超过 4 小时，上报 `technical-director` 做范围决策
