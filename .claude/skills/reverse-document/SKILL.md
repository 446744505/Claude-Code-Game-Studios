---
name: reverse-document
description: "从现有实现生成设计或架构文档。从代码/原型倒推，补全缺失的规划文档。"
argument-hint: "<类型> <路径>（例如 'design src/gameplay/combat' 或 'architecture src/core'）"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---

# 逆向文档（Reverse Documentation）

本技能分析现有实现（代码、原型、系统），并生成相应的设计或架构文档。适用于：
- 先做了功能、尚未写设计文档
- 接手了无文档的代码库
- 已原型验证某机制，需要成文固化
- 需要记录现有代码背后的「为什么」

---

## 工作流

### 1. 解析参数

**格式**：`/reverse-document <类型> <路径>`

**类型选项**：
- `design` → 生成游戏设计文档（GDD 章节）
- `architecture` → 生成架构决策记录（ADR）
- `concept` → 从原型生成概念文档

**路径**：要分析的目录或文件
- `src/gameplay/combat/` → 与战斗相关的全部代码
- `src/core/event-system.cpp` → 单个文件
- `prototypes/stealth-mech/` → 原型目录

**示例**：
```bash
/reverse-document design src/gameplay/magic-system
/reverse-document architecture src/core/entity-component
/reverse-document concept prototypes/vehicle-combat
```

### 2. 分析实现

**阅读并理解代码/原型**：

**设计文档（GDD）**：
- 识别机制、规则、公式
- 提取玩法数值（伤害、冷却、距离等）
- 查找状态机、能力系统、成长线
- 发现代码中已处理的边界情况
- 梳理依赖（哪些系统在交互？）

**架构文档（ADR）**：
- 识别模式（ECS、单例、观察者等）
- 理解技术决策（线程、序列化等）
- 映射依赖与耦合
- 评估性能特征
- 找出约束与权衡

**概念文档（原型分析）**：
- 识别核心机制
- 提炼涌现式玩法模式
- 记录有效与无效之处
- 记录技术可行性洞见
- 记录玩家幻想 / 手感

### 3. 澄清问题（协作协议）

**不要**只复述代码。**要问**设计意图：

**设计向问题**：
- 「我看到战斗中有体力消耗系统。这是为了：
  - 节奏（防止连按）？
  - 资源管理（策略深度）？
  - 还是其他？」
- 「硬直机制看起来很核心。这是支柱玩法，还是辅助功能？」
- 「伤害随等级指数增长。是有意做强度幻想，还是需要再平衡？」

**架构向问题**：
- 「你们用了服务定位器模式。选型原因是：
  - 可测性（模拟依赖）？
  - 解耦（减少硬引用）？
  - 还是从既有代码继承来的？」
- 「我看到是手动内存管理而不是智能指针。是性能要求，还是历史遗留？」

**概念向问题**：
- 「原型强调潜行多于战斗。这是既定支柱吗？」
- 「玩家似乎用抓钩赶路。算特性还是 bug？」

### 4. 呈现发现

在起草前，展示分析结论：

```
已分析 src/gameplay/combat/。发现如下：

已实现机制（MECHANICS IMPLEMENTED）：
- 3-hit combo system with timing windows
- Guard-break mechanic (heavy attack vs blocking enemy)
- Stamina system (depletes on attack/dodge, regens when idle)
- Stagger system (builds up, triggers vulnerable state)

已发现公式（FORMULAS DISCOVERED）：
- Damage = Base * (1 + StrengthScaling * Level)
- Stagger buildup = AttackStaggerValue / (Enemy.Poise * 0.5)
- Stamina cost = BaseStaminaCost * (1 - EfficiencyBonus)

意图不明处（UNCLEAR INTENT AREAS）：
1. 体力系统 — 节奏向还是资源管理向？
2. 硬直 — 支柱玩法还是辅助功能？
3. 伤害成长 — 有意做强度幻想还是需要再调？

起草设计文档前，能否澄清以上几点？
```

等待用户澄清意图后再起草。

### 5. 按模板起草文档

按类型选用对应模板：

| 类型 | 模板 | 输出路径 |
|------|----------|-------------|
| `design` | `templates/design-doc-from-implementation.md` | `design/gdd/[system-name].md` |
| `architecture` | `templates/architecture-doc-from-code.md` | `docs/architecture/[decision-name].md` |
| `concept` | `templates/concept-doc-from-prototype.md` | `prototypes/[name]/CONCEPT.md` 或 `design/concepts/[name].md` |

**草稿结构**：
- 记录**已有内容**（机制、模式、实现）
- 记录**为何如此**（与用户澄清后的意图）
- 标出**缺口**（未处理的边界、设计空白）
- 标注**后续工作**（数值调优、缺失功能）

### 6. 展示草稿并请求批准

**协作协议**：
```
已根据你的代码与澄清起草战斗系统设计文档。

[展示关键章节：Overview, Mechanics, Formulas, Design Intent]

本次增补（ADDITIONS I MADE）：
- 按你的说明将体力记为「节奏机制」
- 补充了代码中未写的边界情况（连段中途体力归零怎么办？）
- 标出平衡风险：高等级下指数伤害成长

标为未完成的章节（SECTIONS MARKED AS INCOMPLETE）：
- "Enemy AI interaction with stagger"（尚未完全实现）
- "Weapon variety and scaling"（目前仅实现剑）

是否可以将此文写入 design/gdd/combat-system.md？
```

等待批准。用户可能在写入前要求修改。

### 7. 写入文档并带元数据

获准后，写入文件并附带标记：

```markdown
---
status: reverse-documented
source: src/gameplay/combat/
date: 2026-02-13
verified-by: [User name]
---

# 战斗系统设计

> **说明**：本文档由现有实现逆向整理而成。
> 记录当前行为与已澄清的设计意图。在实现不完整或意图不清处，部分章节可能不完整。

[文档其余部分...]
```

### 8. 标注后续工作

写入后，建议下一步：

```
✅ 已写入 design/gdd/combat-system.md

建议后续（FOLLOW-UP RECOMMENDED）：
1. 对战斗公式运行 /balance-check（指数成长方面的顾虑）
2. 为体力系统架构决策撰写 ADR
3. 补全缺失边界情况：
   - Stamina depletion mid-combo behavior
   - Stagger state during dodge
4. 武器种类实现后再扩展设计文档

是否现在处理其中某一项？
```

---

## 模板选择逻辑

| 若分析… | 使用模板… | 原因… |
|----------------|-----------------|------------|
| `src/gameplay/*` | design-doc-from-implementation.md | 玩法机制 → GDD |
| `src/core/*`, `src/ai/*` | architecture-doc-from-code.md | 核心系统 → ADR |
| `prototypes/*` | concept-doc-from-prototype.md | 实验 → 概念文档 |
| `src/networking/*` | architecture-doc-from-code.md | 技术系统 → ADR |
| `src/ui/*` | design-doc-from-implementation.md | UI/UX → 设计规格 |

---

## 示例会话：对战斗代码做逆向文档

```
User: /reverse-document design src/gameplay/combat

Agent: 将分析你的战斗实现以撰写设计文档。
       [读代码，归纳机制与公式]

Agent: 已分析 src/gameplay/combat/。发现如下：
       [展示机制、公式、模糊点]

       起草前请澄清：
       1. 体力系统的意图？
       2. 硬直是否为支柱玩法？
       3. 伤害成长 — 有意为之还是需要再调？

User: 1. 体力用于节奏，防止连按
      2. 硬直是核心 — 我要的是有意识的战斗
      3. 指数太强了，应改成线性

Agent: 好的，愿景已清楚。正在起草设计文档。
       [展示含澄清意图的草稿]

       是否可以将此文写入 design/gdd/combat-system.md？

User: 可以，写吧。

Agent: ✅ 已写入 design/gdd/combat-system.md
       ✅ 已标为 [REVERSE-DOCUMENTED]
       ✅ 已标出伤害成长需再平衡

       下一步：
       - 将伤害公式改为线性成长
       - 运行 /balance-check 校验成长曲线
       - 在 game-pillars.md 中将硬直记为支柱玩法
```

---

## 协作协议

本技能遵循协作式设计原则：

1. **先分析**：读代码，理解实现
2. **追问意图**：问「为什么」，不只问「是什么」
3. **呈现发现**：展示结论，标出模糊处
4. **用户澄清**：区分意图与偶然实现
5. **起草文档**：基于现实 + 澄清后的意图
6. **展示草稿**：展示关键章节，说明增补内容
7. **取得批准**：「是否可以将此文写入 [filepath]？」
8. **标注后续**：建议相关工作，不自动执行

**不要臆测意图。在把「为什么」写进文档前，务必先问。**
