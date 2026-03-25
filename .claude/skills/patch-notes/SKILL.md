---
name: patch-notes
description: "根据 git 历史、sprint 数据与内部变更日志生成面向玩家的补丁说明。将开发者用语转化为清晰、有吸引力的玩家沟通文案。"
argument-hint: "[version] [--style brief|detailed|full]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash
---

当本技能被调用时：

1. **解析参数**：
   - `version`：要生成说明的发布版本（例如 `1.2.0`）
   - `--style`：输出风格 — `brief`（要点列表）、`detailed`（含上下文）、
     `full`（含开发者评述）。默认：`detailed`。

2. **从多个来源收集变更数据**：
   - 若存在，读取内部变更日志 `production/releases/[version]/changelog.md`
   - 在上一发布标签与当前标签/HEAD 之间运行 `git log`
   - 读取 `production/sprints/` 中的 sprint 回顾以获取背景
   - 读取 `design/balance/` 中的数值/平衡相关文档
   - 若可用，读取 QA 的缺陷修复记录

3. **将所有变更归类**为面向玩家的类别：
   - **新内容**：新功能、地图、角色、物品、模式
   - **玩法变更**：平衡调整、机制改动、进度/成长相关变化
   - **体验优化**：UI 改进、便利功能、无障碍
   - **缺陷修复**：按系统分组（战斗、UI、联机等）
   - **性能**：玩家可能感知到的优化
   - **已知问题**：对未解决问题的透明说明

4. **将开发者语言转为玩家语言**（示例）：
   - 「重构伤害计算管线」→「提升了命中判定准确度」
   - 「修复背包管理器空引用」→「修复打开背包时偶发崩溃」
   - 「减少战斗循环中的 GC 分配」→「战斗场景性能优化」
   - 删除对玩家无影响的纯内部改动
   - 平衡类改动保留具体数值（例如伤害：50 → 45）

5. **按选定风格生成补丁说明**：

### brief 风格
```markdown
# Patch [Version] — [Title]

**New**
- [Feature 1]
- [Feature 2]

**Changes**
- [Balance/mechanic change with before → after values]

**Fixes**
- [Bug fix 1]
- [Bug fix 2]

**Known Issues**
- [Issue 1]
```

### detailed 风格
```markdown
# Patch [Version] — [Title]
*[Date]*

## Highlights
[用 1–2 句话概括本次最令人兴奋的变更]

## New Content
### [Feature Name]
[2–3 句话说明该功能及玩家为何值得关注]

## Gameplay Changes
### Balance
| Change | Before | After | Reason |
| ---- | ---- | ---- | ---- |
| [Item/ability] | [old value] | [new value] | [brief rationale] |

### Mechanics
- **[Change]**: [说明改了什么、为何改]

## Quality of Life
- [改进点及背景说明]

## Bug Fixes
### Combat
- Fixed [玩家侧现象描述]

### UI
- Fixed [描述]

### Networking
- Fixed [描述]

## Performance
- [玩家可感知的优化]

## Known Issues
- [问题描述；若有变通办法一并写出]
```

### full 风格
包含 detailed 的全部内容，另加：
```markdown
## Developer Commentary
### [Topic]
> [围绕重大变更的开发者视角：为何如此改、曾考虑哪些方案、
> 团队学到了什么。使用第一人称团队口吻撰写。]
```

6. **审阅输出**，确保：
   - 无内部行话（用玩家易懂的说法替代技术术语）
   - 不提及内部系统、工单号或 sprint 编号
   - 平衡改动包含改前/改后数值
   - 缺陷修复描述玩家遇到的现象，而非技术根因
   - 语气符合游戏调性（根据游戏风格调整正式程度）

7. **将补丁说明保存**到 `production/releases/[version]/patch-notes.md`，
   必要时创建目录。

8. **向用户输出**：完整补丁说明、文件路径、按类别统计的变更数量，以及
   被排除供复核的内部改动列表。
