---
name: design-system
description: "针对单个游戏系统的分节引导式 GDD 撰写。从现有文档收集上下文，协作完成各必填章节，交叉引用依赖关系，并增量写入文件。"
argument-hint: "<system-name>（例如 'combat-system'、'inventory'、'dialogue'）"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion, TodoWrite
---

当本技能被调用时：

## 1. 解析参数并校验

**必须**提供系统名称参数。若缺失，报错并说明：
> "用法：`/design-system <system-name>` — 例如 `/design-system combat-system`
> 请先运行 `/map-systems` 创建系统索引，再使用本技能
> 撰写各系统的独立 GDD。"

将系统名规范为 kebab-case 作为文件名（例如 "combat system"
变为 `combat-system`）。

---

## 2. 收集上下文（阅读阶段）

在向用户提问**之前**，先读完所有相关上下文。这是本技能相对临时设计的
主要优势——带着信息进场。

### 2a：必读

- **游戏概念**：阅读 `design/gdd/game-concept.md` — 若缺失则失败：
  > "未找到游戏概念。请先运行 `/brainstorm`。"
- **系统索引**：阅读 `design/gdd/systems-index.md` — 若缺失则失败：
  > "未找到系统索引。请先运行 `/map-systems` 以映射你的系统。"
- **目标系统**：在索引中查找该系统。若未列出，警告：
  > "[system-name] 不在系统索引中。你希望将其加入索引，
  > 还是作为索引外系统设计？"

### 2b：依赖阅读

从系统索引中识别：
- **上游依赖**：本系统所依赖的系统。若已有 GDD，则阅读（其中包含本系统须遵守的决策）。
- **下游依赖方**：依赖本系统的系统。若已有 GDD，则阅读（其中包含本系统须满足的期望）。

对每个已存在的依赖 GDD，提取并保留在上下文中：
- 关键接口（系统间流动的数据）
- 引用本系统输出的公式
- 假定本系统行为的边界情况
- 汇入本系统的调参旋钮

### 2c：选读

- **游戏支柱**：若存在则阅读 `design/gdd/game-pillars.md`
- **已有 GDD**：若存在则阅读 `design/gdd/[system-name].md`（接续撰写，勿从头重来）
- **相关 GDD**：对 `design/gdd/*.md` 做 Glob，并阅读主题相关的文档
  （例如设计 "status-effects" 时，即使无直接依赖也阅读 "combat-system"）

### 2d：呈现上下文摘要

在开始设计工作前，向用户提供简短摘要：

> **正在设计：[System Name]**
> - 优先级：[来自索引] | 层级：[来自索引]
> - 依赖于：[列表，注明哪些已有 GDD、哪些尚未设计]
> - 被以下系统依赖：[列表，注明哪些已有 GDD、哪些尚未设计]
> - 须遵守的既有决策：[来自依赖 GDD 的关键约束]
> - 支柱对齐：本系统主要服务于哪条（哪些）支柱

若有上游依赖尚未设计，警告：
> "[dependency] 尚无 GDD。我们需要对其接口做假设。可考虑先设计该依赖，
> 或我们可定义预期契约并标为暂定。"

使用 `AskUserQuestion`：
- "准备好开始设计 [system-name] 了吗？"
  - 选项："是，开始吧"、"先给我看更多上下文"、"先设计某个依赖"

---

## 3. 创建文件骨架

用户确认后，**立即**用空章节标题创建 GDD 文件。这样增量写入才有落点。

骨架结构来自 `.claude/docs/templates/game-design-document.md`：

```markdown
# [System Name]

> **Status**: In Design
> **Author**: [user + agents]
> **Last Updated**: [today's date]
> **Implements Pillar**: [from context]

## Overview

[To be designed]

## Player Fantasy

[To be designed]

## Detailed Design

### Core Rules

[To be designed]

### States and Transitions

[To be designed]

### Interactions with Other Systems

[To be designed]

## Formulas

[To be designed]

## Edge Cases

[To be designed]

## Dependencies

[To be designed]

## Tuning Knobs

[To be designed]

## Visual/Audio Requirements

[To be designed]

## UI Requirements

[To be designed]

## Acceptance Criteria

[To be designed]

## Open Questions

[To be designed]
```

询问："我可以在 `design/gdd/[system-name].md` 创建骨架文件吗？"

写入后，更新 `production/session-state/active.md`：
- Task：正在设计 [system-name] GDD
- Current section：Starting（骨架已创建）
- File：design/gdd/[system-name].md

---

## 4. 分节设计

按顺序逐节推进。对**每一节**遵循以下循环：

### 单节循环

```
上下文  ->  提问  ->  选项  ->  决策  ->  草案  ->  批准  ->  写入
```

1. **上下文**：说明本节须包含什么，并列出依赖 GDD 中约束本节的有关决策。

2. **提问**：针对本节提出澄清问题。有固定选项时用 `AskUserQuestion`；
   开放式探索用对话文本。

3. **选项**：若本节涉及设计选择（不仅是文档），给出 2–4 种方案及利弊。
   理由用对话说明，再用 `AskUserQuestion` 记录决策。

4. **决策**：用户选定方案或给出自定义方向。

5. **草案**：在对话中写出本节内容供审阅。对未设计依赖的暂定假设须标明。

6. **批准**：询问「批准本节，还是需要修改？」

7. **写入**：用 Edit 工具将 `[To be designed]` 占位符替换为已批准内容。确认写入完成。

每写完一节，更新 `production/session-state/active.md` 中的已完成节名称。

### 各节专项指引

每一节都有独特的设计考量，可酌情引入专家型 agent：

---

### 节 A：Overview（概览）

**目标**：陌生人读一段就能懂。

**可问问题**：
- 一句话说明本系统是什么？
- 玩家如何与之交互？（主动 / 被动 / 自动）
- 本系统为何存在——没有它游戏会失去什么？

**交叉检查**：描述须与系统索引中的表述一致。若有出入须标出。

---

### 节 B：Player Fantasy（玩家幻想）

**目标**：情感目标——玩家应*感受到*什么。

**可问问题**：
- 服务于何种情绪或力量幻想？
- 哪些参考游戏把这种感受做对了？具体是什么造就了这种感受？
- 这是「玩家乐于与之互动」的系统，还是「最好不被注意」的基础设施？

**交叉检查**：须与游戏支柱一致。若本系统服务于某支柱，引用相应支柱原文。

---

### 节 C：Detailed Design（详细设计：核心规则、状态、交互）

**目标**：程序员可无歧义实现、无需再追问的规格。

通常是最长一节。拆成子节：

1. **Core Rules**：基础机制。顺序流程用编号规则，属性用列表。
2. **States and Transitions**：若系统有状态，列出每个状态及每个合法转移。用表格。
3. **Interactions with Other Systems**：对每个依赖（上游与下游），说明流入数据、
   流出数据及接口归属。

**可问问题**：
- 带我走一遍本系统的典型使用流程，逐步说明
- 玩家面临的决策点有哪些？
- 玩家**不能**做什么？（约束与能力同等重要）

**Agent 委派**：机制复杂时，用 Task 工具委派给 `game-designer` 做高层设计评审，
或 `systems-designer` 做详细机制建模。附上第 2 阶段收集的完整上下文。

**交叉检查**：所列每项交互须与依赖 GDD 一致。若依赖写「伤害按 X 计算」而
本系统预期不同，须标出冲突。

---

### 节 D：Formulas（公式）

**目标**：每条数学公式均须给出变量定义、取值范围及边界说明。

**可问问题**：
- 本系统核心计算有哪些？
- 缩放应为线性、对数还是分段？
- 前/中/后期输出的合理区间是什么？

**Agent 委派**：公式繁重的系统（战斗、经济、成长等），通过 Task 委派给
`systems-designer`。提供：
- 节 C 中已写入文件的 Core Rules
- 用户的调参目标
- 依赖 GDD 中的平衡上下文

Agent 应返回建议公式、变量表与预期输出区间。在用户批准前先呈现供审阅。

**交叉检查**：若依赖 GDD 定义了输出进入本系统的公式，须显式引用。勿重复发明——要衔接。

---

### 节 E：Edge Cases（边界情况）

**目标**：明确处理异常情形，避免日后变成 Bug。

**可问问题**：
- 为零时？为最大值时？为负值时？
- 两个效果同时触发时？
- 玩家试图利用规则套利时？（识别退化策略）

**Agent 委派**：交互复杂的系统，委派 `systems-designer` 从公式空间识别边界情况。
叙事类系统可咨询 `narrative-director` 关于破坏叙事的边界情况。

**交叉检查**：对照依赖 GDD 的边界约定。若战斗写「伤害不低于 1」而本系统可把伤害降到 0，
即为待解决的冲突。

---

### 节 F：Dependencies（依赖）

**目标**：映射每条系统连接的方向与性质。

本节部分内容已在上下文收集阶段预备。呈现索引中的已知依赖并询问：
- 是否还有我遗漏的依赖？
- 对每个依赖，具体的数据接口是什么？
- 哪些是硬依赖（没有则系统无法运作）vs. 软依赖（有则增强，没有也能跑）？

**交叉检查**：须双向一致。若本系统写「依赖 Combat」，则 Combat GDD 应写「被 [本系统] 依赖」。
单向-only 的条目须标出待修正。

---

### 节 G：Tuning Knobs（调参旋钮）

**目标**：每个策划可调的数值，附安全区间与极端行为说明。

**可问问题**：
- 哪些值应允许策划在不改代码的情况下调整？
- 每个旋钮过高会怎样？过低会怎样？
- 哪些旋钮互相牵连？（改 A 会让 B 失效）

**Agent 委派**：若公式复杂，委派 `systems-designer` 从公式变量推导调参旋钮。

**交叉检查**：若依赖 GDD 列出影响本系统的旋钮，在此引用。勿重复定义旋钮——指向单一事实来源。

---

### 节 H：Acceptance Criteria（验收标准）

**目标**：可测试的条件，证明系统按设计工作。

**可问问题**：
- 证明本系统可用的最小测试集是什么？
- 本系统的性能预算是多少？（帧时间、内存）
- QA 会先查什么？

**交叉检查**：验收项须覆盖跨系统交互，而非仅本系统孤立行为。

---

### 可选节：Visual/Audio、UI Requirements、Open Questions

模板中包含这些节，但不属于 8 个必填节。在必填节完成后提供：

使用 `AskUserQuestion`：
- "8 个必填节已完成。是否还要定义 Visual/Audio 需求、UI 需求，或记录 Open Questions？"
  - 选项："三个都要"、"只要 Open Questions"、"跳过——我稍后再补"

**Visual/Audio**：若需细节，与 `art-director`、`audio-director` 协同。
GDD 阶段往往简短注记即可。

**UI Requirements**：复杂 UI 系统与 `ux-designer` 协同。

**Open Questions**：记录设计过程中未完全拍板的事项。每条应有负责人与目标解决日期。

---

## 5. 设计后校验

所有节写完后：

### 5a：自检

从文件读回完整 GDD（勿仅靠对话记忆——文件为准）。核验：
- 8 个必填节均有实质内容（非占位）
- 公式引用的变量均已定义
- 边界情况均有处理结论
- 依赖均列出并附接口说明
- 验收标准可测试

### 5b：提供设计评审

呈现完成摘要：

> **GDD 完成：[System Name]**
> - 已写章节：[列表]
> - 暂定假设：[关于未设计依赖的假设列表]
> - 发现的跨系统冲突：[列表或 "无"]

使用 `AskUserQuestion`：
- "现在运行 `/design-review` 校验 GDD 吗？"
  - 选项："是，现在评审"、"我先自己看一遍"、"跳过评审"

若选是，对已完成文件调用 design-review 技能。

### 5c：更新系统索引

GDD 完成（并可选地完成评审）后：

- 阅读系统索引
- 更新目标系统所在行：
  - 若已跑 design-review 且结论为 APPROVED：Status → "Approved"
  - 若已跑 design-review 且结论为 NEEDS REVISION：Status → "In Review"
  - 若跳过 design-review：Status → "Designed"（待评审）
  - 若用户选「我先自己看一遍」：Status → "Designed"
  - Design Doc：链接到 `design/gdd/[system-name].md`
- 更新 Progress Tracker 计数

询问："我可以更新 `design/gdd/systems-index.md` 吗？"

### 5d：更新会话状态

更新 `production/session-state/active.md`：
- Task：[system-name] GDD
- Status：Complete（若跑了 design-review 则为 In Review）
- File：design/gdd/[system-name].md
- Sections：8 节全部写完
- Next：[按设计顺序建议下一系统]

### 5e：建议后续步骤

使用 `AskUserQuestion`：
- "接下来做什么？"
  - 选项：
    - "设计下一系统（[next-in-order]）" — 若仍有未设计系统
    - "修复评审问题" — 若 design-review 标出问题
    - "本会话到此为止"
    - "运行 `/gate-check`" — 若已有足够 MVP 系统设计完成

---

## 6. 专家 Agent 路由

本技能将领域问题委派给专家 agent；主会话编排整体流程，agent 提供专业内容。

| 系统类别 | 主责 Agent | 支持 Agent |
|----------------|---------------|---------------------|
| 战斗、伤害、生命 | `game-designer` | `systems-designer`（公式）、`ai-programmer`（敌人 AI） |
| 经济、战利品、制作 | `economy-designer` | `systems-designer`（曲线）、`game-designer`（循环） |
| 成长、XP、技能 | `game-designer` | `systems-designer`（曲线）、`economy-designer`（消耗） |
| 对话、任务、设定 | `game-designer` | `narrative-director`（故事）、`writer`（内容） |
| UI 系统（HUD、菜单） | `game-designer` | `ux-designer`（流程）、`ui-programmer`（可行性） |
| 音频系统 | `game-designer` | `audio-director`（方向）、`sound-designer`（规格） |
| AI、寻路、行为 | `game-designer` | `ai-programmer`（实现）、`systems-designer`（评分） |
| 关卡/世界系统 | `game-designer` | `level-designer`（空间）、`world-builder`（设定） |
| 镜头、输入、操作 | `game-designer` | `ux-designer`（手感）、`gameplay-programmer`（可行性） |

**通过 Task 工具委派时**：
- 提供：系统名、游戏概念摘要、依赖 GDD 摘录、当前正在写的具体节、需要专家回答的问题
- Agent 将分析/方案返回主会话
- 主会话通过 `AskUserQuestion` 向用户呈现 agent 输出
- 用户决策；主会话负责写入文件
- Agent **不得**直接写文件——主会话拥有全部文件写入权

---

## 7. 恢复与续写

若会话中断（压缩、崩溃、新会话）：

1. 阅读 `production/session-state/active.md` — 其中记录当前系统及各节完成度
2. 阅读 `design/gdd/[system-name].md` — 已有实质内容的节视为完成；
   仍为 `[To be designed]` 的节须继续写
3. 从下一未完成节续写——无需重议已完成部分

这正是增量写入的意义：每个已批准节都能在任意中断后保留。

---

## 协作协议

本技能在每一步遵循协作设计原则：

1. 每一节均遵循 **提问 -> 选项 -> 决策 -> 草案 -> 批准**
2. 每个决策点使用 **AskUserQuestion**（说明 -> 收集 模式）：
   - 第 2 阶段："准备好开始，还是需要更多上下文？"
   - 第 3 阶段："我可以创建骨架吗？"
   - 第 4 阶段（每节）：设计问题、方案选项、草案批准
   - 第 5 阶段："运行设计评审？更新系统索引？接下来做什么？"
3. 创建骨架及每次写入节前询问 **「我可以写入 [filepath] 吗？」**
4. **增量写入**：每节批准后立即写入文件
5. **会话状态**：每写完一节更新
6. **交叉引用**：每节对照已有 GDD 查冲突
7. **专家路由**：复杂节引入专家 agent，将结论呈现给用户决策——绝不静默写入

**绝不**自动生成整份 GDD 并以既成事实呈现。
**绝不**未经用户批准就写入某一节。
**绝不**在未标出冲突的情况下与已批准 GDD 矛盾。
**始终**标明决策来源（依赖 GDD、支柱、用户选择）。
