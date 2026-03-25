# [机制/系统名称]

> **Status**: Draft | In Review | Approved | Implemented
> **Author**: [Agent 或个人]
> **Last Updated**: [日期]
> **Implements Pillar**: [本机制所支撑的游戏支柱]

## 概述

[一段话，向完全不了解本项目的人说明本机制：它是什么、玩家做什么、为何存在？]

## 玩家幻想

[玩家参与本机制时应**感受**到什么？服务于何种情感或力量幻想？本节指导下文所有细节决策。]

## 详细设计

### 核心规则

[精确、无歧义的规则。程序员应能仅根据本节实现而无需追问。顺序流程用编号规则，属性用项目符号。]

### 状态与转移

[若本系统有状态（例如武器状态、状态效果、阶段），记录每个状态及状态之间的所有合法转移。]

| State | Entry Condition | Exit Condition | Behavior |
|-------|----------------|----------------|----------|

### 与其他系统的交互

[本系统如何与 combat、inventory、progression、UI 交互？
对每次交互说明接口：哪些数据流入、哪些流出，以及各方职责。]

## 公式

[本系统使用的每一个数学公式。对每个公式：]

### [公式名称]

```
result = base_value * (1 + modifier_sum) * scaling_factor
```

| Variable | Type | Range | Source | Description |
|----------|------|-------|--------|-------------|
| base_value | float | 1-100 | data file | 修正前的基础量 |
| modifier_sum | float | -0.9 to 5.0 | calculated | 所有生效修正之和 |
| scaling_factor | float | 0.5-2.0 | data file | 随等级变化的缩放 |

**Expected output range**: [min] to [max]
**Edge case**: 当 modifier_sum < -0.9 时，钳制到 -0.9，以避免结果为负。

## 边界情况

[明确记录异常情形下的行为。每个 edge case 都应有清晰处理方式。]

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| [若 X 为零？] | [会发生此事] | [原因] |
| [若两种效果同时触发？] | [优先级规则] | [设计理由] |

## 依赖

[列出本机制依赖的每一个系统，或依赖本机制的系统。]

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| [Combat] | 本机制依赖 Combat | 需要伤害计算结果 |
| [Inventory] | Inventory 依赖本机制 | 提供物品效果数据 |

## 调参项

[所有应可调整以便平衡的数值。写明当前值、安全范围，以及取极值时的表现。]

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|

## 视觉/音频需求

[本机制需要哪些视觉与音频反馈？]

| Event | Visual Feedback | Audio Feedback | Priority |
|-------|----------------|---------------|----------|

## UI 需求

[需要向玩家展示哪些信息、在何时展示？]

| Information | Display Location | Update Frequency | Condition |
|-------------|-----------------|-----------------|-----------|

## 验收标准

[可测试的标准，用于确认本机制按设计工作。]

- [ ] [标准 1：具体、可度量、可测试]
- [ ] [标准 2]
- [ ] [标准 3]
- [ ] Performance: 系统更新在 [X]ms 内完成
- [ ] 实现中无 hardcoded values

## 待决问题

[尚未拍板的事项。每个问题应有负责人与截止日。]

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
