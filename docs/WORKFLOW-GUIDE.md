# Claude Code Game Studios — 完整工作流指南

> **如何从零开始，借助 Agent 架构把游戏做出来并发售。**
>
> 本指南带你走完使用 48 个 Agent、37 条斜杠命令与自动化 hooks 进行游戏开发的各个阶段。假定你已安装 Claude Code，并在项目根目录下工作。

---

## 目录

1. [阶段 0：搭建与配置](#阶段-0搭建与配置)
2. [阶段 1：创意与概念](#阶段-1创意与概念)
3. [阶段 2：预制作与设计](#阶段-2预制作与设计)
4. [阶段 3：原型与验证](#阶段-3原型与验证)
5. [阶段 4：制作期 Sprint 工作流](#阶段-4制作期-sprint-工作流)
6. [阶段 5：实现深入](#阶段-5实现深入)
7. [阶段 6：测试与质量保证](#阶段-6测试与质量保证)
8. [阶段 7：打磨与优化](#阶段-7打磨与优化)
9. [阶段 8：本地化与无障碍](#阶段-8本地化与无障碍)
10. [阶段 9：发布与上线](#阶段-9发布与上线)
11. [阶段 10：上线后与 Live Ops](#阶段-10上线后与-live-ops)
12. [附录 A：Agent 速查](#附录-aagent-速查)
13. [附录 B：斜杠命令速查](#附录-b斜杠命令速查)
14. [附录 C：常见工作流](#附录-c常见工作流)

---

## 阶段 0：搭建与配置

### 你需要什么

开始之前，请确认具备：

- 已安装并可正常使用的 **Claude Code**
- **Git**，以及 Git Bash（Windows）或系统自带终端（Mac/Linux）
- **jq**（可选但推荐 — 若缺失，hooks 会回退到 `grep`）
- **Python 3**（可选 — 部分 hooks 用它做 JSON 校验）

### 步骤 0.1：克隆与配置

克隆仓库并在编辑器中打开：

```bash
git clone <repo-url> my-game
cd my-game
```

### 步骤 0.2：运行 /start（新用户推荐）

若你是新项目，或还不确定要做什么游戏：

```
/start
```

该引导会询问你当前处于哪一步（没想法、模糊想法、清晰概念、已有工作），并把你导向对应阶段。若你已有游戏概念与引擎决策，可跳过。

### 步骤 0.3：选择引擎

在 Claude Code 中运行 `/setup-engine`。这是**最重要**的配置步骤 — 它告诉每个 Agent 你使用的引擎、语言与工具链：

```bash
/setup-engine godot 4.6
```

或不带参数运行 `/setup-engine`，根据游戏需求（2D/3D、平台、团队规模、语言偏好）获得交互式推荐。

**`/setup-engine` 会做什么：**

- 在 `CLAUDE.md` 中固定引擎与版本
- 用命名约定、性能预算与引擎默认项填充 `.claude/docs/technical-preferences.md`
- 检测知识缺口（引擎版本新于 LLM 训练数据）并从网络拉取当前文档，使 Agent 能建议正确的 API
- 在 `docs/engine-reference/` 中创建固定版本的参考文档

**为何重要：** 设定引擎后，系统才知道该启用哪些引擎专项 Agent。若选 Godot，`godot-specialist`、`godot-gdscript-specialist`、`godot-shader-specialist` 等会成为主力专家。Unity 与 Unreal 的专家仍可用，但不会是首选。

> **手动替代方案：** 你也可以直接编辑 `CLAUDE.md` 中的技术栈小节。

### 步骤 0.3：确认 Hooks 可用

新开一个 Claude Code 会话。你应能看到 `session-start.sh` hook 的输出：

```
=== Claude Code Game Studios -- Session Context ===
Branch: main
Recent commits:
  abc1234 Initial commit
===================================
```

若能看到，说明 hooks 正常。若不能，检查 `.claude/settings.json` 中的 hook 路径是否与本机 OS 一致。

### 步骤 0.4：创建目录结构

`CLAUDE.md` 中列出的目录并非都已存在。按需创建 — 系统期望如下布局：

```
src/                  # 游戏源码
  core/               # 引擎 / framework 代码
  gameplay/           # 玩法系统
  ai/                 # AI 系统
  networking/         # 多人代码
  ui/                 # UI 代码
  tools/              # 开发工具
assets/               # 游戏资源
  art/                # 精灵、模型、贴图
  audio/              # 音乐、SFX
  vfx/                # 粒子特效
  shaders/            # Shader 文件
  data/               # JSON 配置 / 平衡数据
design/               # 设计文档
  gdd/                # 游戏设计文档
  narrative/          # 剧情、设定、对白
  levels/             # 关卡设计文档
  balance/            # 平衡表与数据
docs/                 # 技术文档
  architecture/       # 架构决策记录（ADR）
  api/                # API 文档
  postmortems/        # 复盘
tests/                # 测试套件
prototypes/           # 可丢弃原型
production/           # Sprint 计划、里程碑、发布
  sprints/
  milestones/
  releases/
```

> **提示：** 第一天不必全建齐。做到需要该阶段的目录时再建即可。重要的是**一旦创建就遵循此结构**，因为 **rules 系统**会按文件路径强制执行规范。`src/gameplay/` 下的代码适用 gameplay 规则，`src/ai/` 下的适用 AI 规则，以此类推。

---

## 阶段 1：创意与概念

### 本阶段做什么

从「没想法」或「模糊想法」推进到结构化的游戏概念文档。这里要定的是 **做什么游戏**。

> **提示：** 若在阶段 0 运行过 `/start` 并选了路径 A 或 B，你已经在本阶段。`/start` 会自动把你导向 `/brainstorm`。

### 步骤 1.1：用 `/brainstorm` 头脑风暴

若跳过了 `/start`，从这里开始。运行 brainstorm skill：

```
/brainstorm
```

或带上类型提示：

```
/brainstorm roguelike deckbuilder
```

**会发生什么：** brainstorm skill 用专业工作室技法，带你完成协作式、分 6 阶段的创意过程：

Agent: "我们来探索游戏概念。你对哪种类型或主题感兴趣？（可选，留空则开放探索）"

User: "Roguelike deckbuilder"

Agent: "我生成了 10 个结合 roguelike 与 deckbuilder 机制的概念种子。以下是前 3 个：

       **概念 A：Spell Compiler** — 以代码为主题的 deckbuilder，卡牌即函数
       **概念 B：Memory Palace** — 心理惊悚，卡牌即记忆
       **概念 C：Ecosystem Builder** — 自然主题，卡牌即物种

       希望我对哪些概念做 MDA 分析？"

User: "概念 B 听起来很独特"

Agent: [对概念 B 执行 MDA 分析、玩家动机映射、受众定位、动词优先设计，每步展示并征求反馈]

       "分析完成。是否整理成结构化的概念文档？"

User: "好"

Agent: [创建结构化概念文档，含核心创意、目标受众、核心机制与情感目标]

### 步骤 1.2：固化胜出的概念

把头脑风暴产出正式化。使用 **游戏概念模板**：

```
请 Claude 使用 .claude/docs/templates/game-concept.md 模板创建游戏概念文档
```

模板包含：
- 电梯陈述（一句话）
- 核心幻想（玩家想象自己在做什么）
- MDA 拆解
- 目标受众（Bartle 类型、人口统计）
- 核心循环示意图
- 独特卖点（USP）
- 对标作品与差异化

### 步骤 1.3：定义游戏支柱

在继续之前，先锁定 **游戏支柱** — 3～5 条不可妥协的设计价值观，指导所有决策。使用模板：

```
请 Claude 使用 .claude/docs/templates/game-pillars.md 模板创建游戏支柱文档
```

支柱示例：
- **有意义的选择**：每个决定都会带来某种永久改变
- **涌现式叙事**：由系统产生独特故事，而非纯脚本
- **可及的深度**：易上手，难精通

同时定义 **反支柱** — 游戏有意不追求的方向：
- 我们不追求：照片级写实、竞技平衡、线性叙事

> **当你对概念与支柱满意时，即可进入预制作。**

---

## 阶段 2：预制作与设计

### 本阶段做什么

撰写定义游戏如何运作的全部设计文档。此时尚不写代码 — 纯设计与架构。

### 步骤 2.1：将概念拆成系统

在写各份 GDD 之前，先列出游戏需要的所有系统：

```
/map-systems
```

这会创建 `design/gdd/systems-index.md` — 总览跟踪文档，用于：

- 列出每个需要的系统（战斗、移动、UI 等）
- 映射系统间依赖
- 分配优先级层级（MVP、Vertical Slice、Alpha、完整愿景）
- 确定设计顺序（基础 → 核心 → 功能 → 表现 → 打磨）

再按依赖顺序设计各系统：

```
/map-systems next
```

这会选取优先级最高、尚未设计的系统，交给 `/design-system`，分节引导你完成其 GDD。每完成一份 GDD，在下一份开始前先过 `/design-review`。

也可以直接为某系统写 GDD：

```
/design-system combat-system
```

### 步骤 2.2：撰写游戏设计文档（GDD）

对游戏中的每个主要系统，在 `design/gdd/` 下创建设计文档。使用模板：

```
请 Claude 使用 .claude/docs/templates/game-design-document.md 模板创建 GDD
```

**`design-docs` 规则**（`.claude/rules/design-docs.md`）要求每份 GDD 包含以下 8 节：

| # | 小节 | 内容 |
|---|------|------|
| 1 | **概述** | 系统的一句话级摘要 |
| 2 | **玩家幻想** | 玩家使用该系统时想象/感受到什么 |
| 3 | **详细规则** | 无歧义的机制规则 — 两名设计师读后应能做出同一实现 |
| 4 | **公式** | 每个计算公式，含变量定义与取值范围 |
| 5 | **边界情况** | 怪异情形下发生什么？需明确写出 |
| 6 | **依赖** | 与其他系统的连接（双向） |
| 7 | **调参旋钮** | 策划可安全改动的数值及安全范围 |
| 8 | **验收标准** | 如何验证实现正确？需具体、可度量 |

**可能创建的 GDD 示例：**
- `design/gdd/core-loop.md` — 基础游玩循环
- `design/gdd/combat-system.md` — 战斗如何运作
- `design/gdd/progression-system.md` — 玩家如何成长
- `design/gdd/inventory-system.md` — 物品管理
- `design/gdd/ability-system.md` — 玩家能力与技能

### 步骤 2.2：复杂机制请 `game-designer` Agent

需要深度机制推演时，交给 `game-designer`：

```
请 game-designer agent 设计一套制作系统：支持涌现式配方（玩家发现组合），
技能成长与实验挂钩。系统要有成就感但不要刷。目标受众为探索者与成就型玩家。
```

**会发生什么：** `game-designer` 与你协作完成设计：

Agent: "我来协助设计制作系统。先问几个问题：
       - 配方应通过实验发现，还是明确教学？
       - 制作失败应有多严厉？
       - 目标受众主要是探索者还是成就型？"

User: "通过实验发现。中等惩罚 — 消耗材料但给部分技能经验。主要面向探索者。"

Agent: "基于此，有三种机制取向：

       **方案 A：纯发现** — 无提示，完全自由
       优点：探索回报最大
       缺点：挫败风险高，可能需要 wiki

       **方案 B：提示系统** — 材料相容时有视觉提示
       优点：引导式探索，挫败较低
       缺点：「顿悟」时刻满足感略弱

       **方案 C：混合** — 自由实验 + 可选日记提示（随技能解锁）
       优点：兼顾两种玩法，渐进支架
       缺点：实现更复杂

       我推荐方案 C：尊重探索者又提供安全网，日记提示可挂在技能进度上满足成就型。
       哪种更符合你的愿景？"

User: "方案 C 很合适"

Agent: [分节起草 GDD — 概述、玩家幻想、详细规则、公式、边界情况 — 每节展示并吸收反馈]

       "成功率和技能成长曲线需要更细的公式。我去咨询 systems-designer 做数学部分。"

Agent: [咨询后] "完整草稿就绪，公式已由 systems-designer 校验。
       economy-designer 指出原料成本需平衡 — 我已纳入其建议。

       我可以写入 design/gdd/crafting-system.md 吗？"

User: "可以"

Agent: [写入文件]

### 步骤 2.3：设计评审

任何设计文档交给程序之前，先评审：

```
/design-review design/gdd/crafting-system.md
```

**会发生什么：** 该 skill 按 8 节标准检查文档，验证：
- 各节是否齐全且完整？
- 公式是否无歧义、变量是否定义？
- 边界情况是否明确（不能写「待定」）？
- 依赖是否双向列出？
- 验收标准是否可测试？

在继续前修复所有指出的问题。

### 步骤 2.4：叙事设计（如适用）

若游戏包含故事、设定或对白，在此阶段构建：

1. **世界观** — 用 `world-builder` 定义派系、历史、地理与世界规则
2. **故事结构** — 用 `narrative-director` 设计故事线、角色弧与叙事节拍
3. **角色表** — 对每个主要角色使用 `narrative-character-sheet.md` 模板

`narrative` 规则（`.claude/rules/narrative.md`）强制：
- 文档间设定一致
- 正史层级（核心正史 / 风味 / 外传）
- 角色声线档案
- 世界规则文档化

### 步骤 2.5：技术架构

开写代码前，先敲定关键技术决策：

```
/architecture-decision "NPC AI 用状态机还是行为树"
```

**会发生什么：** 该 skill 引导你撰写架构决策记录（ADR）：

Agent: "我来记录这项架构决策。先了解背景：当前是什么在推动这个决策？"

User: "要在状态机与行为树之间为 NPC AI 做选择。计划里有 50+ 种敌人类型。"

Agent: "在 50+ 类型前提下，分析如下：

       **方案 A：有限状态机（FSM）**
       优点：简单、易调试、开销小
       缺点：行为复杂时难以维护，逻辑难复用

       **方案 B：行为树（Behavior Tree）**
       优点：高度可复用、易扩展、可视觉化调试
       缺点：学习曲线陡，开销略高

       **方案 C：效用 AI（Utility AI）**
       优点：行为涌现、动态决策
       缺点：更难调试，需仔细调参

       对 50+ 类型，我推荐行为树 — 可复用性与扩展性胜过学习成本。

       是否创建 ADR 记录该决策？"

User: "可以，用行为树"

Agent: [在 docs/architecture/ 创建 ADR：背景、驱动因素、各方案利弊、选定方案与理由、后果]

**复杂架构决策可请 `technical-director` agent：**

```
请 technical-director agent 评估：在团队规模（单人）与引擎（Godot 4）前提下，
玩法实体应采用 ECS 还是传统组件架构。
```

Agent: "针对你的情境分析 ECS 与传统组件：

       [结合 Godot 4 场景系统、单人工作流、学习曲线、调试工具、性能需求做详细分析]

       **建议：传统 Godot 基于节点的组件**

       理由：[详细论证]

       是否创建包含完整分析的 ADR？"

User: "好"

Agent: [创建完整 ADR]

### 步骤 2.6：里程碑规划

用模板定义项目里程碑：

```
请 Claude 使用 .claude/docs/templates/milestone-definition.md 模板创建里程碑定义
```

典型独立游戏里程碑：
1. **Vertical Slice** — 一关可玩、品质具代表性
2. **Alpha** — 核心系统齐全，占位美术可接受
3. **Beta** — 内容完整、功能齐套，开始打磨
4. **Release Candidate** — 仅修 Bug，准备上线
5. **Gold** — 发售

> **当所有设计文档已评审且架构已定，即可开始搭建。**

---

## 阶段 3：原型与验证

### 本阶段做什么

在全力投入制作前，用可丢弃原型验证高风险或不确定的机制。可避免数周制作白费。

### 步骤 3.1：识别需要原型的部分

并非一切都要原型。在以下情况做原型：
- 机制新颖，不确定是否好玩
- 技术路线有风险，不确定是否可行
- 两套设计看起来都行，需要上手感受差异

### 步骤 3.2：运行原型 Skill

```
/prototype "带动量传递的抓钩移动"
```

**会发生什么：** 该 skill 与你协作搭建原型：

Agent: "我们来原型化抓钩移动。先澄清要验证的假设：

       - 是验证基于动量的抓钩手感？
       - 还是验证物理可行性？
       - 或对比不同操控方案？

       这个原型要回答的核心问题是什么？"

User: "验证动量传递是否令人满意，能否形成技巧表达"

Agent: "明白。建议设置如下：

       **假设**：带动量传递的抓钩令人满意，且通过技巧性物理操作产生涌现移动选择

       **成功标准**：
       - 玩家可流畅串联 3+ 个抓钩点而不丢动量
       - 动量足够可预判以规划，又足够丰富以产生惊喜
       - 5 分钟内玩家能发现非预期移动技巧

       **原型范围**（最多 1～2 小时）：
       - 基础抓钩点附着
       - 动量计算与传递
       - 简单测试关，多种抓钩布局
       - 无美术、UI、音效 — 纯机制

       是否符合你要验证的内容？"

User: "很好，再加一条：能否甩过拐角"

Agent: "成功标准已加入拐角摆动。是否创建 prototypes/grappling-hook/ 并实现？"

User: "好"

Agent: [创建目录结构、含假设/标准/状态的 README，委托 prototyper agent 做最小实现]

**关键规则：** `prototype-code` 规则（`.claude/rules/prototype-code.md`）有意放宽原型代码标准 — 硬编码可接受、不要求测试，但 **必须** 有含假设与结论的 README。

### 步骤 3.3：评估原型

试玩/测试原型并回答：
- 假设是否成立？是否好玩 / 可行 / 更优？
- 有什么意外发现？
- 上正式制作前要改什么？

在原型 README 中更新发现并给出 **结论**：Ship It、Rework 或 Kill It。

### 步骤 3.4：试玩报告

若请他人试玩原型：

```
/playtest-report
```

会生成结构化报告模板，涵盖：
- 测试者背景与情境
- 任务完成率
- 困惑点与摩擦
- 高光时刻
- 量化评分
- 按优先级排序的 takeaway

> **当原型已验证（或证伪）你最危险的假设时，即可进入制作期。**

---

## 阶段 4：制作期 Sprint 工作流

### 本阶段做什么

这是核心制作循环。以 Sprint（通常 1～2 周）为单位实现功能、修 Bug、跟踪进度。重复直至游戏内容完备。

### 步骤 4.1：创建首个 Sprint

```
/sprint-plan new
```

**会发生什么：** 该 skill 委托 `producer` 协作制定 Sprint：

Agent: "我们来规划 Sprint。本 Sprint 的首要目标是什么？（一句话定义成功）"

User: "实现核心战斗：基础攻击与生命"

Agent: "本 Sprint 你有多少可用时间？（小时或天数）"

User: "两周共 20 小时"

Agent: "基于「核心战斗系统」，我拆任务如下：

       **Must Have**（没有则 Sprint 失败）：
       - 伤害计算系统（design/gdd/combat-system.md 中的公式）
       - 生命组件与受伤处理
       - 基础攻击输入与执行
       - 伤害计算单元测试

       **Should Have**（重要但可顺延）：
       - 受击反馈与 VFX
       - 死亡状态处理
       - 战斗数值数据文件

       **Nice to Have**（有时间再做）：
       - 攻击连段
       - 伤害数字 UI

       **风险**：
       - 尚无战斗设计文档（阻塞 — 需先写）
       - VFX 需要 technical-artist 参与

       任务拆分是否符合你的设想？要增删或调整优先级吗？"

User: "连段移到 Should Have — 对手感很重要"

Agent: "已更新。是否创建 production/sprints/sprint-01.md？"

User: "好"

Agent: [创建 Sprint 计划：目标、分类任务、20 小时容量估算、风险登记与依赖]

### 步骤 4.2：日常工作循环

每次会话可按此模式：

```
1. 启动 Claude Code 会话
   （session-start hook 显示分支、Sprint 状态、最近提交）

2. 从 Sprint 计划中选任务
   （例如：「我在做战斗伤害计算系统」）

3. Claude 识别合适的 Agent
   - 设计需调整时找 game-designer
   - 实现找 gameplay-programmer
   - 公式细节找 systems-designer

4. 实现功能（详见阶段 5）

5. 编写测试（验证驱动开发）

6. 提交工作
   （validate-commit hook 检查设计文档引用、
    JSON 数据合法性、无不当硬编码）

7. 重复直至本会话上下文接近满载（约 65～70%）

8. 使用 /clear 重新开始，或 compact 后继续
```

### 步骤 4.3：Sprint 跟踪

随时查看 Sprint 进度：

```
/sprint-plan status
```

若范围在膨胀：

```
/scope-check production/sprints/sprint-03.md
```

会将当前范围与原始计划对比并标出：
- Sprint 开始后新增的任务
- 范围增长百分比
- 建议删减项

### 步骤 4.4：Sprint 复盘与回顾

Sprint 结束时：

```
/retrospective
```

会分析：
- 计划完成 vs 实际完成
- 速率（完成任务数/点数）
- 做得好的地方
- 不顺的地方
- 遇到的阻塞
- 下一 Sprint 可执行的改进

再规划下一 Sprint：

```
/sprint-plan new
```

新 Sprint 会纳入未完成任务、回顾所得与更新后的优先级。

### 步骤 4.5：里程碑评审

在里程碑检查点：

```
/milestone-review "alpha"
```

会产出全面报告：
- 功能完成度百分比
- 质量指标（Bug 数、测试覆盖率）
- 风险评估
- 通过/不通过建议
- 达成里程碑的行动项

---

## 阶段 5：实现深入

### 本阶段做什么

此处开始写代码。Agent 体系通过路径规则、自动化 hooks 与结构化委托保障代码质量。

### 步骤 5.1：单系统实现

对单个玩法系统（例如「实现伤害计算」）：

```
请 gameplay-programmer agent 按 design/gdd/combat-system.md 实现伤害计算系统。
公式在第 4 节。采用项目的数据驱动方式，配置放在 assets/data/。
```

**会发生什么：** `gameplay-programmer` 协作实现：

Agent: "我已阅读 design/gdd/combat-system.md。实现前需确认：

       第 4 节伤害公式：`base_damage * (1 + attack_stat * 0.1) * type_modifier`

       问题：
       - attack_stat 是否要设上限？
       - type_modifier 按武器还是按伤害类型存放？
       - 伤害取整方式（向下/向上/四舍五入）？

       另外尚未看到生命组件的设计文档。要我写一份极简版，还是你单独设计？"

User: "attack_stat 限制在 0～100。类型修正按伤害类型。向下取整。
      写极简生命文档 — 仅当前/最大 HP 与受伤处理。"

Agent: "实现思路如下：

       **将创建的文件**：
       - src/gameplay/combat/damage_calculator.gd（核心公式）
       - src/gameplay/combat/health_component.gd（HP 跟踪）
       - assets/data/combat_damage.json（基础伤害、类型修正）
       - design/gdd/health-system.md（极简文档）

       **架构**：
       - DamageCalculator 为无状态工具（易测）
       - HealthComponent 为节点组件（伤害/死亡用 signal）
       - 数值均来自配置文件（gameplay-code 规则）

       可以开始实现吗？"

User: "可以"

Agent: [在规则约束下实现：数据驱动、与 delta time 无关、signal 通信、可测试的依赖注入]

       "实现完成。已创建：
       - src/gameplay/combat/damage_calculator.gd
       - src/gameplay/combat/health_component.gd
       - assets/data/combat_damage.json（JSON 结构已校验）
       - design/gdd/health-system.md

       可进入测试。需要我起草单元测试吗？"

User: "需要"

Agent: [编写测试]

### 步骤 5.2：跨系统功能（团队编排）

跨多个领域时，使用 team skill：

```
/team-combat "治疗技能：HoT（持续治疗）与驱散"
```

**会发生什么：** 该 team skill 编排 6 阶段协作开发：

**阶段 1 — 设计**：
Agent (game-designer): "我来设计治疗技能。问题：
       - HoT 叠加还是刷新持续时间？
       - 驱散移除哪些 debuff — 全部还是某几类？
       - 法力消耗是否随治疗量缩放？"

User: [回答]

Agent: [创建设计文档并展示草稿] "设计就绪。批准进入架构？"

User: "批准"

**阶段 2 — 架构**：
Agent (gameplay-programmer): "建议代码结构：
       - HealAbility 继承 BaseAbility
       - HoT 为独立 buff 组件（可复用）
       - Cleanse 为工具函数（别处可能复用）

       这样可以吗？"

User: "批准"

**阶段 3 — 并行实现**：
Agent (coordinator): "开始并行实现：
       - gameplay-programmer：HealAbility + HoT buff 代码
       - ai-programmer：NPC 何时治疗的 AI 逻辑
       - technical-artist：HoT VFX 粒子规格
       - sound-designer：治疗施放与 HoT 跳字 SFX 规格

       全部完成后再同步。"

[各专家在定稿前会提问并展示工作]

**阶段 4 — 集成**：
Agent (gameplay-programmer): "各组件就绪，正在集成…"
       [展示集成结果] "测试场景中可运行。可以验证了吗？"

**阶段 5 — 验证**：
Agent (qa-tester): "按设计文档验收标准测试…"
       [汇报结果] "发现 2 个边界情况：[详情]。修还是延后？"

User: "修严重的，另一个延后"

**阶段 6 — 报告**：
Agent (coordinator): "功能完成。状态：
       ✓ 设计、架构、实现、集成完成
       ✓ 严重 Bug 已修
       - 1 个轻微边界情况记入 backlog
       可合并。"

其他 team skill 类似：
- `/team-narrative` — 从结构到成文对白
- `/team-ui` — 从线框到打磨后的 UI
- `/team-level` — 从布局到完整遭遇与装扮的关卡
- `/team-audio` — 从方向到落地音频事件

### 步骤 5.3：AI 系统

NPC 行为、寻路或决策：

```
请 ai-programmer agent 按 design/gdd/enemy-ai.md 为敌人 NPC 实现
巡逻/调查/追逐行为树。
```

**会发生什么：** `ai-programmer` 在实现前会澄清问题：

Agent: "我已读 design/gdd/enemy-ai.md。搭建行为树前需确认：

       - 巡逻时在路点停留还是连续循环？
       - 调查状态搜索多久后回到巡逻？
       - 追逐：一见即追还是需要「确认时间」？
       - 追逐中玩家丢失视野时如何处理？

       另外性能预算是每帧 2ms 给全部 AI。最坏情况下同时活跃 NPC 有多少个？"

User: "路点停 2 秒。搜索 8 秒。见即追。
      丢视野则回到最后已知位置搜索，再进入调查。
      最多 15 个活跃 NPC。"

Agent: "即每 NPC 约 0.13ms。我会用分层行为树 + 共享寻路查询以控制在预算内。

       方案：
       - 根选择器：Chase > Investigate > Patrol
       - 时间/范围来自 assets/data/enemy_ai.json（数据驱动，符合 ai-code 规则）
       - Debug 可视化当前状态 + 侦测半径
       - 攻击前 0.5s 预备动作（玩家可见）

       批准此方案？"

User: "批准"

Agent: [在规则下实现：2ms 预算、数据驱动参数、debug 可视化、意图可读]

       "实现完成。用 20 个 NPC 测试 — 约 1.8ms/帧。
       Debug 可视化：绿色=巡逻，黄色=调查，红色=追逐。可以评审了吗？"

`ai-code` 规则自动约束性能预算、数据驱动参数、debug 可视化与对玩家公平的行为。

### 步骤 5.4：引擎专项工作

需要引擎深度知识时：

```
# Godot 示例：
请 godot-specialist agent 评审我们的场景树架构，
并就性能与可维护性给出改进建议。

# 子系统示例：
请 godot-gdscript-specialist 重构 signal 架构，
降低战斗与 UI 系统之间的耦合。
```

引擎专家了解：
- 引擎特有模式与反模式
- 该引擎的性能优化手段
- 子系统正确使用方式（Godot signal、Unity Addressables、UE GAS）
- 何时用原生扩展 vs 脚本

### 步骤 5.5：架构决策记录（ADR）

实现过程中遇到重大技术选择时：

```
/architecture-decision "系统通信用事件总线还是直接 Signal"
```

记录下来，以便未来的你（或贡献者）理解 **为何** 如此构建。

### 步骤 5.6：代码评审

实现某功能后：

```
/code-review src/gameplay/combat/damage_calculator.gd
```

**会发生什么：** 该 skill 进行协作式架构评审：

Agent: "正在评审 src/gameplay/combat/damage_calculator.gd…

       **架构合规**：
       ✓ 符合 ADR-003（无状态工具模式）
       ✓ 数据驱动（gameplay-code 规则）
       ✗ 问题：直接引用 HealthComponent — 破坏依赖注入

       **SOLID**：
       ✓ 单一职责 — 仅计算伤害
       ✗ 依赖倒置 — 与 HealthComponent 类型紧耦合

       **可测试性**：
       ✗ HealthComponent 依赖难以 mock

       **性能**：
       ✓ 热路径无分配
       ⚠ 警告：每次调用在字典中做类型查找 — 可考虑缓存

       **建议**：
       1. 高：为 HealthComponent 增加接口以便 mock
       2. 中：缓存类型修正查找
       3. 低：边界情况增加 null 检查

       要展示如何修复问题 1（依赖注入）吗？"

User: "要"

Agent: [展示带接口的重构代码]

       "这样可传入任何带 take_damage() 的对象，便于测试。批准此重构？"

User: "批准"

Agent: [应用修改] "代码评审完成。高优先级问题已解决。"

---

## 阶段 6：测试与质量保证

### 本阶段做什么

测试贯穿制作，而非只在末尾。项目支持测试驱动开发与自动化约束。

### 步骤 6.1：测试先行（验证驱动开发）

项目推行 **验证驱动开发**：测试与实现同步或先于实现，而非事后补写。

`test-standards` 规则（`.claude/rules/test-standards.md`）要求：
- 测试命名：`test_[system]_[scenario]_[expected_result]`
- 结构：Arrange / Act / Assert
- 不依赖外部可变状态
- 测试自行清理
- 测试文件与被测源文件对应

```
请 qa-tester agent 根据 design/gdd/combat-system.md 中的验收标准
为伤害计算系统编写测试用例。
```

**会发生什么：** `qa-tester` 协作编写测试：

Agent: "我已阅读 design/gdd/combat-system.md 的验收标准。写测试前确认范围：

       从验收标准可见：
       - 基础伤害公式须产出预期值
       - 类型修正正确应用
       - 边界：attack_stat 限制在 0～100
       - 边界：负伤害变为 0

       问题：
       - 是否需要与 HealthComponent 的集成测试，还是仅单元测试？
       - 是否需要性能测试（例如 1000 次计算 < Xms）？
       - 文档以外还有要特别覆盖的边界吗？"

User: "暂时只要单元测试。加性能测试 — 目标 10000 次计算 < 1ms。
      还要测无效伤害类型时的行为。"

Agent: "测试计划如下：

       **单元测试**：
       - test_damage_basic_calculation（基础公式、已知值）
       - test_damage_type_modifiers（配置中各修正）
       - test_damage_attack_stat_clamping（低于 0、高于 100）
       - test_damage_negative_becomes_zero
       - test_damage_invalid_type（配置中未定义类型）
       - test_damage_rounding（验证向下取整）

       **性能测试**：
       - test_damage_performance_10k_calculations

       批准此测试套件？"

User: "批准"

Agent: [按 test-standards 规则编写：命名、Arrange/Act/Assert、无外部依赖、清理]

       "测试套件完成。共 7 个用例。要运行吗？"

### 步骤 6.2：缺陷报告

发现 Bug 时：

```
/bug-report "目标带护盾 buff 时治疗法术造成伤害而非治疗"
```

会生成结构化缺陷报告，含：
- 严重度（Critical / Major / Minor / Cosmetic）
- 优先级（P0～P4）
- 复现步骤
- 预期 vs 实际
- 涉及系统/组件
- 相关设计文档

### 步骤 6.3：测试计划

里程碑验证时：

```
请 qa-lead agent 为 Alpha 里程碑创建测试计划，
覆盖战斗、成长与 UI 系统。
```

**会发生什么：** `qa-lead` 协作制定全面测试计划：

Agent: "我来为 Alpha 里程碑写测试计划。先确认范围：

       从里程碑定义看，在范围内的是：
       - 战斗系统
       - 成长系统
       - UI 系统

       问题：
       - 测试哪些平台？（Windows、Linux、Mac、移动端？）
       - 目标性能？（60fps？30fps？）
       - 已知高风险区是否需要额外覆盖？
       - 测试须何时完成？"

User: "Windows 与 Linux。目标 60fps。战斗 AI 风险高 — 边界情况多。
      测试截止下周末。"

Agent: "建议测试计划结构：

       **功能测试**（3 天）：
       - 战斗：伤害计算、状态效果、死亡状态、AI 行为
       - 成长：经验、升级、技能解锁、属性提升
       - UI：菜单、HUD、背包、设置

       **边界测试**（1 天）：
       - 战斗 AI：丢视野、寻路失败、卡住
       - 边界：满级、背包满、负伤害

       **回归套件**（1 天）：
       - 重跑此前里程碑全部测试

       **性能基准**（1 天）：
       - 15 敌人 + 复杂 VFX 下 60fps
       - 内存低于 2GB
       - 加载低于 5s

       **平台测试**（1 天）：
       - Windows 10/11
       - Ubuntu 22.04 LTS

       时间线上可行。批准或调整？"

User: "批准"

Agent: [使用 test-plan.md 模板创建详细测试计划]

### 步骤 6.4：平衡测试

实现经济或成长系统后：

```
/balance-check assets/data/combat_damage.json
```

会分析平衡数据中的：
- 统计异常值
- 断裂的成长曲线
- 退化策略（某一选项碾压其他）
- 经济失衡（通胀、通缩、 exploit）

---

## 阶段 7：打磨与优化

### 本阶段做什么

功能已齐。现在要把它 **做好**。本阶段侧重性能、画面打磨、音频与手感。

### 步骤 7.1：性能剖析

```
/perf-profile
```

**会发生什么：** 该 skill 引导结构化性能剖析：

Agent: "开始性能剖析。先定目标：
       - 目标帧率？
       - 内存上限？
       - 目标平台？"

User: "中端 PC 60fps。内存低于 2GB。Windows/Linux。"

Agent: "对当前版本跑 profiler…

       **当前性能**：
       - 平均 FPS：48
       - 帧时间：20.8ms（60fps 目标：16.6ms）
       - 内存：2.4GB（超出目标 400MB）

       **瓶颈**（按影响排序）：
       1. 战斗 AI：8.2ms/帧（占帧时间 49%）— 高优先级
       2. 粒子 VFX：3.1ms/帧（18%）— 中
       3. UI 渲染：2.8ms/帧（17%）— 中
       4. 物理：1.9ms/帧（11%）— 低

       **内存问题**：
       1. 图集未压缩：+280MB — 高
       2. 音频未流式：+120MB — 高

       **建议**：
       1. 战斗 AI：分层行为树、减少每帧检测
       2. 贴图：启用 GPU 压缩
       3. 音频：长音频流式加载而非全载入

       要为前 3 项生成详细优化任务吗？"

User: "要"

Agent: [生成可执行的优化任务，含代码位置与预期收益]

**更深分析可请 `performance-analyst` agent：**

```
请 performance-analyst agent 分析战斗系统的帧时间尖峰。
10+ 敌人活跃时会出现卡顿。
```

Agent: "我来分析战斗性能。用不同敌人数 profiling…

       **结果**：
       - 5 敌人：稳定 58fps
       - 10 敌人：52fps，偶发跌至 45fps
       - 15 敌人：38fps，尖峰至 22fps

       **根因**：行为树每帧对每个 NPC 整树重算。15 个 NPC = 15 次完整遍历 = 8.2ms。

       **方案**：
       1. 交错更新：每帧只更新 5 个 NPC（轮转）
       2. 缓存树结果：仅在状态变化时重算
       3. 简化树：节点数从 12 减到 6

       建议 2 + 3：缓存并简化树。
       预期：8.2ms → 1.5ms（省 6.7ms ≈ +40fps）

       要展示优化后的行为树吗？"

User: "要"

Agent: [展示优化实现]

### 步骤 7.2：画面打磨

用 team polish skill 做全面打磨：

```
/team-polish "combat system"
```

**会发生什么：** 协调多角色打磨：

Agent (coordinator): "开始对战斗系统打磨。评估当前状态与优先级…

       **现状**：
       - 性能：48fps（需优化）
       - 视觉反馈：有受击效果但冲击力不足
       - 音频：SFX 已有但混音需调整
       - 手感：输入响应尚可，命中缺乏重量感

       将并行协调 4 位专家。请先排序打磨优先级：
       - 性能（达成 60fps）
       - 视觉冲击（VFX、shader、反馈）
       - 音频（混音、空间感、力度）
       - 手感/juice（屏震、hitstop、粒子）"

User: "性能第一，然后视觉，再手感，最后音频"

**阶段 1 — 性能** (performance-analyst)：
Agent: "执行战斗性能优化…"
       [应用此前剖析中的优化]
       "现稳定 62fps。性能目标达成。"

**阶段 2 — 视觉** (technical-artist)：
Agent: "分析视觉反馈。受击 VFX 较泛用。问题：
       - 不同伤害类型是否要有明显不同的视觉反馈？
       - 暴击是否要一眼可辨？
       - 命中屏震 — 轻微还是强烈？"

User: "按伤害类型区分。暴击要抢眼。屏震轻微。"

Agent: [输出打磨后的 VFX 规格与 shader 改进]
       "VFX 打磨完成。要看对比演示吗？"

**阶段 3 — 手感** (technical-artist + gameplay-programmer)：
Agent: "加入命中 hitstop、暴击粒子爆发、轻微镜头震动。测试手感…"
       "普通命中 hitstop 100ms、暴击 200ms 手感不错。批准？"

User: "批准"

**阶段 4 — 验证** (qa-tester)：
Agent: "对战斗打磨变更跑回归…"
       "全部通过。未引入新 Bug。"

Agent (coordinator): "打磨完成：
       ✓ 性能：48fps → 62fps
       ✓ 视觉：强化 VFX、分类型效果、暴击反馈
       ✓ 手感：Hitstop、粒子、镜头震动
       ✓ 测试：全部通过
       可合并。"

### 步骤 7.3：音频落地

```
/team-audio "combat audio"
```

**会发生什么：** 协调完整音频管线：

**阶段 1 — 音频方向** (audio-director)：
Agent: "定义战斗音频方向。情感目标是？
       - 残酷血腥？
       - 战术精准？
       - 魔法奇幻？
       - 快节奏街机？"

User: "战术精准，命中要有一定重量感"

Agent: "战斗音频方向如下：

       **音频支柱**：
       - 清晰度：每个动作声音可辨
       - 空间感：仅凭听感能判断敌人位置
       - 反馈：成功行动有满足感
       - 氛围：保持张力但不疲劳

       **优先级**：
       1. 武器声有力但不刺耳
       2. 脚步对空间感关键
       3. 敌人攻击用音频做预备提示
       4. 命中确认音要有冲击力

       批准此方向？"

User: "批准"

**阶段 2 — SFX 规格** (sound-designer)：
Agent: "基于方向，SFX 事件列表：

       **玩家动作**：
       - weapon_swing_light, weapon_swing_heavy（不同呼啸）
       - footstep_walk, footstep_run（随材质变化）
       - ability_cast_start, ability_cast_release

       **命中**：
       - hit_flesh_light, hit_flesh_heavy（音高变化）
       - hit_armor_deflect（金属弹跳）
       - critical_hit（分层特殊命中）

       **敌人音频**：
       - enemy_attack_windup（0.5s 预告）
       - enemy_footstep（比玩家响以利空间判断）

       每个声音 3～5 条变体防重复。
       还有遗漏的关键项吗？"

User: "加闪避/翻滚声"

Agent: "已加 dodge_roll。SFX 规格完整。"

**阶段 3 — 音频实现** (technical-artist)：
Agent: "实现音频系统：
       - 事件驱动触发（无硬编码音频调用）
       - 3D 空间音频与距离衰减
       - 优先级（关键声不被切）
       - 动态混音（战斗音乐层 + SFX 平衡）

       问题：
       - 同时播放上限？（建议 32）
       - 距离衰减曲线 — 线性还是对数？"

User: "最多 32 个声。对数衰减。"

Agent: [接入音频中间件]

**阶段 4 — 集成** (gameplay-programmer)：
Agent: "将音频事件接到玩法…
       - 武器挥击：接动画事件
       - 脚步：接移动速度与地面类型
       - 命中：接伤害事件
       - 敌人预告：攻击执行前 0.5s

       可进行音频试玩。"

Agent (coordinator): "战斗音频管线完成。请在游戏中试听并从平衡、清晰度、手感反馈。"

### 步骤 7.4：技术债

发布前评估技术债：

```
/tech-debt
```

会扫描代码库中的：
- TODO/FIXME/HACK 注释
- 重复代码
- 过度复杂的函数
- 缺失测试
- 过时依赖

每项分类并排序。由你决定发布前修哪些、哪些延后。

---

## 阶段 8：本地化与无障碍

### 步骤 8.1：本地化扫描

```
/localize src/
```

扫描：
- 应外置的硬编码字符串
- 会破坏翻译的字符串拼接
- 未考虑文本膨胀的 UI（德文约长 30%）
- 缺失的 locale 文件

### 步骤 8.2：无障碍评审

```
请 accessibility-specialist agent 审计我们的 UI 系统：
WCAG 合规、色盲支持与输入重映射。
```

`ui-code` 规则已部分强制无障碍：
- 可本地化字符串（无硬编码文案）
- 键盘与手柄输入
- 文字缩放
- 对色盲友好的设计

`accessibility-specialist` 更深入：
- 屏幕阅读器兼容
- 键位重映射完整性
- 字幕与说明性字幕
- 动态敏感选项
- 难度/辅助选项

---

## 阶段 9：发布与上线

### 本阶段做什么

游戏已打磨、测试完毕。现在发售。

### 步骤 9.1：发布清单

```
/release-checklist v1.0.0
```

生成全面的发布前清单，涵盖：
- 构建验证（各平台可编译运行）
- 认证要求（平台相关）
- 商店元数据（描述、截图、预告片）
- 法务合规（EULA、隐私政策、分级）
- 存档兼容性
- Analytics 与 telemetry 校验

### 步骤 9.2：上线就绪（完整校验）

```
/launch-checklist
```

这是「全部门」级校验：

| 部门 | 检查内容 |
|------|----------|
| **工程** | 构建稳定性、崩溃率、内存泄漏、加载时间 |
| **设计** | 功能完整度、教程流程、难度曲线 |
| **美术** | 资源质量、缺失贴图、LOD |
| **音频** | 缺失音效、混音电平、空间音频 |
| **QA** | 按严重度统计未关 Bug、回归通过率 |
| **叙事** | 对白完整、设定一致、错别字 |
| **本地化** | 字符串全译、无截断、各 locale 测试 |
| **无障碍** | 合规清单、辅助功能测试 |
| **商店** | 元数据完整、截图获批、定价确定 |
| **市场** | 媒体包、上线预告片、社媒排期 |
| **社区** | 补丁说明草稿、FAQ、支持渠道 |
| **基础设施** | 服务器容量、CDN、监控 |
| **法务** | EULA 定稿、隐私政策、COPPA/GDPR |

每项给出 **Go / No-Go**。全部 Go 方可上线。

### 步骤 9.3：面向玩家的内容

```
/patch-notes v1.0.0
```

根据 git 历史与内部数据生成玩家向补丁说明。把开发用语（如「重构状态机」）转成玩家能懂的话（如「敌人反应更灵敏」）。

```
/changelog v1.0.0
```

生成内部变更日志（更偏技术、面向团队）。

### 步骤 9.4：协调发布

```
/team-release
```

**会发生什么：** 协调各部门上线：

**阶段 1 — 发布前校验** (release-manager)：
Agent: "开始协调 v1.0.0 发布。先各部门状态…

       上线清单结果：
       ✓ 工程：全绿
       ✓ QA：2 个轻微 Bug 未关（不阻塞）
       ✓ 美术/音频：完成
       ✓ 本地化：完成
       ⚠ 商店：截图待批
       ✗ 市场：上线预告片未定稿

       两处阻塞。对时间线影响？"

User: "截图今早批了。预告片今天下班前完成。"

Agent: "时间线更新：预告片完成后明日发布。是否开始准备构建？"

User: "好"

**阶段 2 — 构建管理** (release-manager)：
Agent: "创建发布构建 v1.0.0…
       - Windows：✓ 完成、已测、2.1GB
       - Linux：✓ 完成、已测、2.3GB
       - 校验和已生成
       - 版本清单已创建
       - 更新服务器已配置

       构建均可部署。"

**阶段 3 — QA 最终签字** (qa-lead)：
Agent: "对发布构建跑最终回归…
       - 全部关键测试：✓
       - 全部高优先级测试：✓
       - 平台专项测试：✓

       QA 签字放行。无阻塞项。"

**阶段 4 — 部署准备** (devops-engineer)：
Agent: "CI/CD 就绪。部署计划：
       1. 12:00 UTC 上传构建到 CDN
       2. 12:30 UTC 更新商店页
       3. 13:00 UTC 开放下载（上线时刻）
       4. 监控负载与错误率

       若出现严重问题已有回滚方案。
       批准部署时间表？"

User: "批准"

**阶段 5 — Go/No-Go** (producer)：
Agent: "最终 go/no-go：
       ✓ 全部构建已验证
       ✓ QA 签字完成
       ✓ 商店元数据已批
       ✓ 上线预告片完成（已确认）
       ✓ 市场材料就绪
       ✓ 支持团队已简报
       ✓ 部署计划已批

       全部 GO。建议明日 13:00 UTC 上线。
       最终批准？"

User: "GO，上线"

Agent: "上线批准。部署将于明日 12:00 UTC 开始。我会监控并汇报状态。"

### 步骤 9.5：发货

`validate-push` hook 在向 `main` 或 `develop` push 时会警告 — 这是刻意的，发布推送应慎重：

```
git tag v1.0.0
git push origin main --tags
```

---

## 阶段 10：上线后与 Live Ops

### 步骤 10.1：Hotfix 工作流

生产环境出现严重 Bug 时：

```
/hotfix "背包超过 99 件时玩家丢存档"
```

在完整审计轨迹下绕过常规 Sprint：
1. 创建 hotfix 分支
2. 跟踪审批
3. 实现修复
4. 确保 backport 到开发分支
5. 记录事件

### 步骤 10.2：复盘

上线风波平息后：

```
请 Claude 使用 .claude/docs/templates/post-mortem.md 模板创建复盘文档
```

涵盖：
- 做得好的
- 不顺的
- 意外之处
- 关键指标（销量、评价、崩溃率）
- 下一项目的经验

### 步骤 10.3：Live Ops（如适用）

对持续运营内容的游戏：

```
请 live-ops-designer agent 设计赛季内容节奏：战斗通行证、周常、月更内容包。
关注 30/60/90 日队列留存。
```

`live-ops-designer` 负责：
- 赛季/活动日历
- 战斗通行证设计
- 内容投放节奏
- 留存机制
- Live 经济调参
- 参与度 analytics

### 步骤 10.4：社区运营

```
请 community-manager agent 起草 v1.0.1 补丁说明、
回应反馈频道中前 5 条社区抱怨，并准备「已知问题」帖。
```

---

## 附录 A：Agent 速查

### 「我要做 X — 该用哪个 Agent？」

| 我要… | Agent | 层级 |
|------|-------|------|
| 想游戏创意 | `/brainstorm` skill | -- |
| 设计游戏机制 | `game-designer` | 2 |
| 设计具体公式/数值 | `systems-designer` | 3 |
| 设计关卡 | `level-designer` | 3 |
| 设计战利品表/经济 | `economy-designer` | 3 |
| 构建世界观 | `world-builder` | 3 |
| 撰写对白 | `writer` | 3 |
| 规划故事 | `narrative-director` | 2 |
| 规划 Sprint | `producer` | 1 |
| 做创意决策 | `creative-director` | 1 |
| 做技术决策 | `technical-director` | 1 |
| 实现玩法代码 | `gameplay-programmer` | 3 |
| 实现核心引擎系统 | `engine-programmer` | 3 |
| 实现 AI 行为 | `ai-programmer` | 3 |
| 实现多人 | `network-programmer` | 3 |
| 实现 UI | `ui-programmer` | 3 |
| 做开发工具 | `tools-programmer` | 3 |
| 评审代码架构 | `lead-programmer` | 2 |
| 制作 shader / VFX | `technical-artist` | 3 |
| 定义视觉风格 | `art-director` | 2 |
| 定义音频风格 | `audio-director` | 2 |
| 设计 UX 流程 | `ux-designer` | 3 |
| 编写测试用例 | `qa-tester` | 3 |
| 规划测试策略 | `qa-lead` | 2 |
| 性能剖析 | `performance-analyst` | 3 |
| 搭建 CI/CD | `devops-engineer` | 3 |
| 设计 analytics | `analytics-engineer` | 3 |
| 检查无障碍 | `accessibility-specialist` | 3 |
| 规划 Live Ops | `live-ops-designer` | 3 |
| 管理发布 | `release-manager` | 2 |
| 管理本地化 | `localization-lead` | 2 |
| 快速原型 | `prototyper` | 3 |
| 安全审计 | `security-engineer` | 3 |
| 与玩家沟通 | `community-manager` | 3 |
| Godot 专项 | `godot-specialist` | 3 |
| Unity 专项 | `unity-specialist` | 3 |
| Unreal 专项 | `unreal-specialist` | 3 |

### Agent 层级

```
                    creative-director / technical-director / producer
                                         |
          ---------------------------------------------------------------
          |            |           |           |          |        |
    game-designer  lead-prog  art-dir  audio-dir  narr-dir  qa-lead  release-mgr
          |            |           |           |          |        |
     specialists  programmers  tech-art  snd-design  writer   qa-tester  devops
```

**上报规则：** 两名 Agent 意见不一致时向上汇报。设计冲突找 `creative-director`。技术冲突找 `technical-director`。范围冲突找 `producer`。

---

## 附录 B：斜杠命令速查

### 按工作流阶段

| 阶段 | 命令 |
|------|------|
| **入门** | `/start` |
| **创意** | `/brainstorm` |
| **设计** | `/map-systems`, `/design-system`, `/design-review`, `/architecture-decision` |
| **Sprint** | `/sprint-plan`, `/estimate`, `/scope-check`, `/retrospective` |
| **实现** | `/code-review`, `/prototype`, `/tech-debt` |
| **测试** | `/balance-check`, `/playtest-report`, `/perf-profile` |
| **资源** | `/asset-audit`, `/localize` |
| **发布** | `/release-checklist`, `/launch-checklist`, `/changelog`, `/patch-notes`, `/hotfix` |
| **制作管理** | `/milestone-review`, `/onboard` |
| **团队** | `/team-combat`, `/team-narrative`, `/team-ui`, `/team-release`, `/team-polish`, `/team-audio`, `/team-level` |

---

## 附录 C：常见工作流

### 工作流 1：「刚开始，还没有游戏想法」

```
1. /start（询问进度，导向合适工作流）
   — 或直接 /brainstorm 进入创意
2. 从头脑风暴产出中选定最佳概念
3. 创建游戏概念文档（templates/game-concept.md）
4. 定义游戏支柱（templates/game-pillars.md）
5. 对概念文档做 /design-review
6. /map-systems 将概念拆成带依赖与优先级的系统
7. /design-system 分节撰写各系统 GDD
```

### 工作流 2：「已有设计，想开始写代码」

```
1. 对每份 GDD 做 /design-review 确保扎实
2. 首个重大技术选择用 /architecture-decision
3. /sprint-plan new 规划首个 Sprint
4. 用 gameplay-programmer / engine-programmer 开始实现
5. 每个大功能后 /code-review
6. 与代码同步写测试
7. 频繁提交（hooks 自动校验）
```

### 工作流 3：「要加一个复杂功能」

```
1. 在 design/gdd/ 创建或更新该功能的 GDD
2. /design-review 校验设计
3. /estimate 评估工作量与风险
4. 选用合适的 /team-* skill：
   - /team-combat — 战斗向功能
   - /team-narrative — 叙事内容
   - /team-ui — UI 功能
   - /team-level — 新关卡/区域
   - /team-audio — 音频工作
5. 对实现做 /code-review
6. 若影响平衡则 /balance-check
```

### 工作流 4：「线上出了问题」

```
1. /hotfix "问题描述"
2. 在 hotfix 分支实现修复
3. 对修复做 /code-review
4. 运行测试
5. hotfix 构建走 /release-checklist
6. 部署并 backport
```

### 工作流 5：「临近里程碑」

```
1. /milestone-review 查看进度
2. /scope-check 检查范围蔓延
3. /tech-debt 评估里程碑前技术债
4. /perf-profile 核对性能目标
5. /team-polish 最终打磨
6. 准备发布时用 /release-checklist
```

### 工作流 6：「新开一个 Sprint」

```
1. /retrospective 回顾上一 Sprint
2. /sprint-plan new 创建下一 Sprint
3. /scope-check 确保范围可控
4. 按 Sprint 任务推进
5. Sprint 中途用 /sprint-plan status 查看进度
```

### 工作流 7：「准备发售游戏」

```
1. 最终里程碑用 /milestone-review
2. /tech-debt 决定上线可接受的技术债
3. /localize 做最后一轮本地化
4. 通过 accessibility-specialist 做无障碍审计
5. /launch-checklist 做全部门校验
6. /team-release 协调发布
7. /patch-notes 与 /changelog 面向玩家/团队沟通
8. 发货！
9. 上线后若出问题用 /hotfix
10. 上线稳定后做复盘
```

---

## 充分利用本体系的建议

1. **永远先设计再实现。** Agent 体系默认「先有设计文档再写代码」。Agent 会频繁引用 GDD。

2. **跨领域功能用 team skill。** 不要自己手动协调 4 个 Agent — 让 `/team-combat`、`/team-narrative` 等负责编排。

3. **相信 rules 系统。** 规则标出问题就修。规则里凝结的是数据驱动、delta time、无障碍等经验。

4. **主动 compact。** 上下文用到约 65～70% 时就 compact 或 `/clear`。pre-compact hook 会保存进度。不要等到顶满。

5. **用对层级的 Agent。** 不要让 `creative-director` 写 shader，也不要让 `qa-tester` 做设计决策。层级有其用途。

6. **交给程序前跑 `/design-review`。** 尽早发现不完整规格，减少返工。

7. **每个大功能后跑 `/code-review`。** 在架构问题扩散前抓住它们。

8. **高风险机制先做原型。** 一天原型可能省下一周在不可行机制上的制作时间。

9. **Sprint 计划要诚实。** 定期用 `/scope-check`。范围蔓延是独立游戏头号杀手。

10. **用 ADR 记录决策。** 未来的你会感谢现在的你写下 **为什么** 这样实现。
