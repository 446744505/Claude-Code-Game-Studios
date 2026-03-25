---
name: team-release
description: "编排发布团队：协调 `release-manager`、`qa-lead`、`devops-engineer` 与 `producer`，从候选版本执行到部署。"
argument-hint: "[版本号或 'next']"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
---
当本技能被调用时，通过结构化流水线编排发布团队。

**决策点：** 在每个阶段切换时，使用 `AskUserQuestion` 将子智能体的提案以可选项形式呈现给用户。在对话中写出智能体的完整分析，再用简短标签记录决策。进入下一阶段前必须获得用户批准。

## 团队构成
- **`release-manager`** — 发布分支、版本号、变更日志、部署
- **`qa-lead`** — 测试签字、回归套件、发布质量门禁
- **`devops-engineer`** — 构建流水线、制品、部署自动化
- **`producer`** — 上线/不上线决策、干系人沟通、排期

## 如何委派

使用 Task 工具将每位成员作为子智能体启动：
- `subagent_type: release-manager` — 发布分支、版本号、变更日志、部署
- `subagent_type: qa-lead` — 测试签字、回归套件、发布质量门禁
- `subagent_type: devops-engineer` — 构建流水线、制品、部署自动化
- `subagent_type: producer` — 上线/不上线决策、干系人沟通

在每个智能体的提示中提供完整上下文（版本号、里程碑状态、已知问题）。在流水线允许时并行启动相互独立的智能体（例如阶段 3 中的智能体可同时运行）。

## 流水线

### 阶段 1：发布规划
委派给 **`producer`**：
- 确认所有里程碑验收标准已满足
- 标出本发布中推迟的范围项
- 确定目标发布日期并通知团队
- 产出：带范围确认的发布授权

### 阶段 2：发布候选（RC）
委派给 **`release-manager`**：
- 从约定提交点切出发布分支
- 在所有相关文件中提升版本号
- 使用 `/release-checklist` 生成发布前校验清单
- 冻结分支 — 不再合入功能，仅允许缺陷修复
- 产出：发布分支名称与清单

### 阶段 3：质量门禁（并行）
并行委派：
- **`qa-lead`**：执行完整回归测试套件；覆盖所有关键路径；确认无 S1/S2 缺陷；完成质量签字。
- **`devops-engineer`**：为所有目标平台构建发布制品；确认构建干净且可复现；在 CI 中运行自动化测试。

### 阶段 4：本地化与性能
委派（若资源允许可与阶段 3 并行）：
- 确认所有字符串已翻译（若有 `localization-lead` 可向其委派）
- 对照目标运行性能基准（若有 `performance-analyst` 可向其委派）
- 产出：本地化与性能签字

### 阶段 5：上线/不上线
委派给 **`producer`**：
- 收集来自 `qa-lead`、`release-manager`、`devops-engineer`、`technical-director` 的签字
- 评估未结问题 — 是否阻塞发布或可随版发布？
- 做出上线/不上线决定
- 产出：发布决策与理由

### 阶段 6：部署（若决定上线）
委派给 **`release-manager`** 与 **`devops-engineer`**：
- 在版本控制中打发布标签
- 使用 `/changelog` 生成变更日志
- 部署到预发布环境做最终冒烟测试
- 部署到生产环境
- 发布后监控 48 小时

### 阶段 7：发布后
- **`release-manager`**：生成发布报告（已交付内容、推迟项、指标）
- **`producer`**：更新里程碑跟踪，向干系人同步
- **`qa-lead`**：关注新上报缺陷中的回归
- 若发布中出现问题，安排发布后复盘

## 产出
一份摘要报告，涵盖：发布版本、范围、质量门禁结果、上线/不上线决策、部署状态与监控计划。
