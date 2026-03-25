---
name: team-combat
description: "编排战斗团队：协调 game-designer、gameplay-programmer、ai-programmer、technical-artist、sound-designer 与 qa-tester，端到端设计、实现并验证一项战斗特性。"
argument-hint: "[战斗特性描述]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
---
本技能被调用时，通过结构化流水线编排战斗团队。

**决策点：** 在每个阶段切换时，使用 `AskUserQuestion` 将子智能体的方案以可选项形式呈现给用户。在对话中写出该智能体的完整分析，再用简短标签记录决策。进入下一阶段前须获得用户批准。

## 团队构成
- **game-designer** — 设计机制，定义公式与边界情况
- **gameplay-programmer** — 实现核心玩法代码
- **ai-programmer** — 为该特性实现 NPC/敌人 AI 行为
- **technical-artist** — 制作 VFX、着色器效果与视觉反馈
- **sound-designer** — 定义音频事件、打击音效与战斗环境音
- **qa-tester** — 编写测试用例并验证实现

## 如何委派

使用 `Task` 工具将每位成员作为子智能体启动：
- `subagent_type: game-designer` — 设计机制，定义公式与边界情况
- `subagent_type: gameplay-programmer` — 实现核心玩法代码
- `subagent_type: ai-programmer` — 实现 NPC/敌人 AI 行为
- `subagent_type: technical-artist` — 制作 VFX、着色器效果、视觉反馈
- `subagent_type: sound-designer` — 定义音频事件、打击音效、环境音
- `subagent_type: qa-tester` — 编写测试用例并验证实现

在每位智能体的提示词中提供完整上下文（设计文档路径、相关代码文件、约束）。在流水线允许时并行启动相互独立的智能体（例如第 3 阶段中的智能体可同时运行）。

## 流水线

### 第 1 阶段：设计
委派给 **game-designer**：
- 在 `design/gdd/` 中创建或更新设计文档，涵盖：机制概览、玩家角色幻想（player fantasy）、详细规则、带变量定义的公式、边界情况、依赖关系、可调参数与安全范围，以及验收标准
- 产出：已完成的设计文档

### 第 2 阶段：架构
委派给 **gameplay-programmer**（若涉及 AI，则同时委派 **ai-programmer**）：
- 审阅设计文档
- 设计代码架构：类结构、接口、数据流
- 标出与现有系统的集成点
- 产出：架构草图、文件清单与接口定义

### 第 3 阶段：实现（在可能时并行）
并行委派：
- **gameplay-programmer**：实现核心战斗机制代码
- **ai-programmer**：实现 AI 行为（若特性涉及 NPC 反应）
- **technical-artist**：制作 VFX 与着色器效果
- **sound-designer**：定义音频事件列表与混音说明

### 第 4 阶段：集成
- 将玩法代码、AI、VFX 与音频串联起来
- 确保所有可调参数已暴露且数据驱动
- 验证该特性与现有战斗系统协同正常

### 第 5 阶段：验证
委派给 **qa-tester**：
- 根据验收标准编写测试用例
- 测试设计中记录的所有边界情况
- 确认性能影响在预算内
- 对发现的问题提交缺陷报告

### 第 6 阶段：签收
- 汇总各团队成员结果
- 报告特性状态：COMPLETE / NEEDS WORK / BLOCKED
- 列出未决问题及其负责人

## 产出
一份摘要报告，涵盖：设计完成度、各成员实现情况、测试结果，以及未关闭问题。
