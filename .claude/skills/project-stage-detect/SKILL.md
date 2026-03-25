---
name: project-stage-detect
description: "根据现有产物自动分析项目状态、判断阶段、识别缺口并推荐下一步。"
argument-hint: "[可选：角色过滤，如 'programmer' 或 'designer']"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash
---

# 项目阶段检测

本 skill 会扫描项目，判断当前开发阶段、产物完整度以及需要关注的缺口。特别适合：
- 接手已有项目
- 熟悉代码库
- 在里程碑前检查缺什么
- 弄清「我们现在在哪」

---

## 工作流

### 1. 扫描关键目录

分析项目结构与内容：

**设计文档**（`design/`）：
- 统计 `design/gdd/*.md` 中的 GDD 文件数量
- 检查是否存在 game-concept.md、game-pillars.md、systems-index.md
- 若存在 systems-index.md，统计系统总数与已设计系统数
- 分析完整度（概述、详细设计、边界情况等）
- 统计 `design/narrative/` 中的叙事文档
- 统计 `design/levels/` 中的关卡设计

**源代码**（`src/`）：
- 统计源文件数量（与语言无关）
- 识别主要系统（含 5+ 文件的目录）
- 检查是否存在 core/、gameplay/、ai/、networking/、ui/ 等目录
- 粗略估算代码行数规模

**生产产物**（`production/`）：
- 检查是否有进行中的 sprint 计划
- 查找里程碑定义
- 查找路线图文档

**原型**（`prototypes/`）：
- 统计原型目录数量
- 检查是否有 README（有文档 vs 无文档）
- 判断原型是归档还是仍在使用

**架构文档**（`docs/architecture/`）：
- 统计 ADR（Architecture Decision Record）数量
- 检查是否有总览/索引类文档

**测试**（`tests/`）：
- 统计测试文件数量
- 粗略估计测试覆盖（启发式）

### 2. 划分项目阶段

根据扫描结果判断阶段。先读 `production/stage.txt` ——
若存在则以其为准（来自 `/gate-check` 的显式覆盖）。否则
按下列启发式自动判断（从最高阶段往回核对）：

| 阶段 | 指标 |
|-------|-----------|
| **Concept（概念）** | 无游戏概念文档，处于头脑风暴阶段 |
| **Systems Design（系统设计）** | 已有游戏概念，systems index 缺失或不完整 |
| **Technical Setup（技术搭建）** | 已有 systems index，引擎未配置 |
| **Pre-Production（预生产）** | 引擎已配置，`src/` 中源文件少于 10 个 |
| **Production（生产）** | `src/` 中源文件 10+，活跃开发中 |
| **Polish（打磨）** | 仅显式设定（由 `/gate-check` 的 Production → Polish 门禁） |
| **Release（发布）** | 仅显式设定（由 `/gate-check` 的 Polish → Release 门禁） |

### 3. 协作式缺口识别

**不要**只列缺失文件。应**提出澄清问题**，例如：

- 「我看到战斗相关代码（`src/gameplay/combat/`）但没有 `design/gdd/combat-system.md`。是先做的原型，还是需要反向补文档？」
- 「已有 15 份 ADR 但没有架构总览。要不要我建一份，方便新成员上手？」
- 「`production/` 里没有 sprint 计划。你们是在别处（Jira、Trello 等）跟踪吗？」
- 「有游戏概念但没有 systems index。你们已经把概念拆成各系统了吗，还是要跑 `/map-systems`？」
- 「`prototypes/` 里有 3 个项目没有 README。这些是实验性尝试，还是需要补文档？」

### 4. 生成阶段报告

使用模板：`.claude/docs/templates/project-stage-report.md`

**报告结构**：
```markdown
# 项目阶段分析

**日期**：[date]
**阶段**：[Concept/Systems Design/Technical Setup/Pre-Production/Production/Polish/Release]

## 完整度总览
- 设计：[X%]（[N] 份文档，[gaps]）
- 代码：[X%]（[N] 个文件，[systems]）
- 架构：[X%]（[N] 份 ADR，[gaps]）
- 生产管理：[X%]（[status]）
- 测试：[X%]（[coverage estimate]）

## 已识别缺口
1. [缺口描述 + 澄清问题]
2. [缺口描述 + 澄清问题]

## 建议的下一步
[按阶段与角色排序的优先级列表]
```

### 5. 按角色过滤的建议（可选）

若用户提供了角色参数（例如 `/project-stage-detect programmer`）：

**Programmer（程序员）**：
- 侧重架构文档、测试覆盖、缺失的 ADR
- 代码与文档不一致的缺口

**Designer（设计师）**：
- 侧重 GDD 完整度、缺失的设计章节
- 原型文档

**Producer（制作人）**：
- 侧重 sprint 计划、里程碑跟踪、路线图
- 跨团队协调类文档

**General（无角色）**：
- 各域缺口的整体视图
- 跨领域优先级最高的事项

### 6. 写入前征得同意

**协作协议**：
```
我已分析你的项目，结论如下：

[展示摘要]

已识别缺口：
1. [缺口 1 + 问题]
2. [缺口 2 + 问题]

建议的下一步：
- [优先级 1]
- [优先级 2]
- [优先级 3]

是否可以将完整阶段分析写入 production/project-stage-report.md？
```

创建文件前等待用户确认。

---

## 使用示例

```bash
# 通用项目分析
/project-stage-detect

# 面向程序员的分析
/project-stage-detect programmer

# 面向设计师的分析
/project-stage-detect designer
```

---

## 后续动作

生成报告后，可建议相关下一步：

- **有概念但没有 systems index？** → 用 `/map-systems` 拆成系统
- **缺设计文档？** → `/reverse-document design src/[system]`
- **缺架构文档？** → `/architecture-decision` 或 `/reverse-document architecture`
- **原型需要文档？** → `/reverse-document concept prototypes/[name]`
- **没有 sprint 计划？** → `/sprint-plan`
- **临近里程碑？** → `/milestone-review`

---

## 协作原则

本 skill 遵循协作式设计原则：

1. **先问**：对缺口提问，不要臆断
2. **给选项**：「要我创建 X，还是你们在别处跟踪？」
3. **由你决定**：等待你的方向
4. **展示草稿**：先给出报告摘要
5. **取得同意**：「是否写入 production/project-stage-report.md？」

**切勿**悄悄写文件。**务必**先展示发现并征得同意再创建产物。
