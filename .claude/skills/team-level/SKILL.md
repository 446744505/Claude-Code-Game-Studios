---
name: team-level
description: "编排关卡设计团队：`level-designer` + `narrative-director` + `world-builder` + `art-director` + `systems-designer` + `qa-tester`，完成整块区域/关卡的创作。"
argument-hint: "[关卡名或要设计的区域]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
---

当本技能被调用时：

**决策点：** 在每一步切换时，使用 `AskUserQuestion` 将子智能体的方案以可选项形式呈现给用户。在对话中写出该智能体的完整分析，再用简短标签记录决策。进入下一步前必须经用户批准。

1. **阅读参数**，明确目标关卡或区域（例如 `tutorial`、`forest dungeon`、`hub town`、`final boss arena`）。

2. **收集上下文**：
   - 阅读游戏概念：`design/gdd/game-concept.md`
   - 阅读游戏支柱：`design/gdd/game-pillars.md`
   - 阅读已有关卡文档：`design/levels/`
   - 阅读相关叙事文档：`design/narrative/`
   - 阅读该区域所属地域/派系的世界观文档

## 如何委派

使用 Task 工具为每位成员启动子智能体：
- `subagent_type: narrative-director` — 叙事目的、角色、情感弧线
- `subagent_type: world-builder` — 设定背景、环境叙事、世界规则
- `subagent_type: level-designer` — 空间布局、节奏、遭遇、导航
- `subagent_type: systems-designer` — 敌人组合、战利品表、难度平衡
- `subagent_type: art-director` — 视觉主题、色板、光照、资产需求
- `subagent_type: qa-tester` — 测试用例、边界测试、试玩检查清单

在每个智能体的提示词中始终提供完整上下文（游戏概念、支柱、已有关卡文档、叙事文档）。

3. **按顺序编排关卡设计团队**：

### 第一步：叙事语境（`narrative-director` + `world-builder`）
启动 `narrative-director` 智能体以：
- 界定本区域的叙事目的（在此发生哪些剧情节点？）
- 标出关键角色、对话触发器与设定要素
- 说明情感弧线（进入、进行中、离开时应让玩家有何感受？）

启动 `world-builder` 智能体以：
- 提供该区域设定背景（历史、派系存在、生态）
- 界定环境叙事机会
- 说明影响本区域玩法的任何世界规则

### 第二步：布局与遭遇设计（`level-designer`）
启动 `level-designer` 智能体以：
- 设计空间布局（关键路径、可选路径、秘密）
- 定义节奏曲线（张力高峰、休整区、探索区）
- 布置遭遇并安排难度递进
- 设计环境谜题或导航挑战
- 定义兴趣点与地标以利寻路
- 标明出入口及与相邻区域的连接

### 第三步：系统整合（`systems-designer`）
启动 `systems-designer` 智能体以：
- 规定敌人组合与遭遇公式
- 定义战利品表与奖励摆放
- 相对预期玩家等级/装备平衡难度
- 设计区域专属机制或环境危害
- 规定资源分布（生命拾取、存档点、商店等）

### 第四步：视觉方向（`art-director`）
启动 `art-director` 智能体以：
- 定义区域视觉主题与色板
- 说明光照氛围与时段设定
- 列出所需美术资产（环境道具、独特资产）
- 定义视觉地标与视线
- 说明特殊 VFX 需求（天气、粒子、雾等）

### 第五步：QA 规划（`qa-tester`）
启动 `qa-tester` 智能体以：
- 为关键路径编写测试用例
- 识别边界与极端情况（顺序破坏、软锁）
- 为本区域建立试玩检查清单
- 定义关卡完成的验收标准

4. **汇总关卡设计文档**，将各团队产出合并为关卡设计模板格式。

5. **保存至** `design/levels/[level-name].md`。

6. **输出摘要**，包含：区域概览、遭遇数量、预估资产清单、叙事节点，以及跨团队依赖或未决问题。
