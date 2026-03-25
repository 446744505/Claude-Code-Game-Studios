# 游戏工作室 Agent 架构 — 快速入门

## 这是什么？

这是一套面向游戏开发的完整 Claude Code Agent 架构。它将 48 个专职 AI Agent 组织成贴近真实游戏工作室的层级：职责清晰、委派规则与协作协议明确。其中包含针对 Godot、Unity、Unreal 的引擎专项 Agent，以及各引擎主要子系统的子专家。所有设计向 Agent 与模板均扎根于成熟游戏设计理论（MDA、自我决定理论、心流、Bartle 玩家类型）。请按项目选用对应引擎的一组 Agent。

## 如何使用

### 1. 理解层级

Agent 分三层：

- **第一层（Opus）**：做高层决策的总监
  - `creative-director` — 愿景与创意冲突裁决
  - `technical-director` — 架构与技术决策
  - `producer` — 排期、协调与风险管理

- **第二层（Sonnet）**：各职能领域的负责人
  - `game-designer`、`lead-programmer`、`art-director`、`audio-director`、
    `narrative-director`、`qa-lead`、`release-manager`、`localization-lead`

- **第三层（Sonnet/Haiku）**：在各自领域内执行的专员
  - 策划、程序、美术、文案、测试、工程等

### 2. 为任务选对 Agent

自问：「在真实工作室里，这件事该哪个部门管？」

| 我需要…… | 使用此 Agent |
|-------------|---------------|
| 设计新机制 | `game-designer` |
| 写战斗代码 | `gameplay-programmer` |
| 做 Shader | `technical-artist` |
| 写对白 | `writer` |
| 规划下一个 Sprint | `producer` |
| 审代码质量 | `lead-programmer` |
| 写测试用例 | `qa-tester` |
| 设计关卡 | `level-designer` |
| 解决性能问题 | `performance-analyst` |
| 搭建 CI/CD | `devops-engineer` |
| 设计掉落/经济表 | `economy-designer` |
| 化解创意冲突 | `creative-director` |
| 做架构决策 | `technical-director` |
| 管理一次发布 | `release-manager` |
| 准备可翻译字符串 | `localization-lead` |
| 快速验证机制想法 | `prototyper` |
| 审代码安全问题 | `security-engineer` |
| 检查无障碍合规 | `accessibility-specialist` |
| Unreal 相关建议 | `unreal-specialist` |
| Unity 相关建议 | `unity-specialist` |
| Godot 相关建议 | `godot-specialist` |
| 设计 GAS 能力/效果 | `ue-gas-specialist` |
| 界定 BP/C++ 边界 | `ue-blueprint-specialist` |
| 实现 UE 复制 | `ue-replication-specialist` |
| 搭建 UMG/CommonUI 控件 | `ue-umg-specialist` |
| 设计 DOTS/ECS 架构 | `unity-dots-specialist` |
| 写 Unity Shader/VFX | `unity-shader-specialist` |
| 管理 Addressable 资源 | `unity-addressables-specialist` |
| 搭建 UI Toolkit/UGUI 界面 | `unity-ui-specialist` |
| 写地道 GDScript | `godot-gdscript-specialist` |
| 制作 Godot Shader | `godot-shader-specialist` |
| 开发 GDExtension 模块 | `godot-gdextension-specialist` |
| 规划线上活动与赛季 | `live-ops-designer` |
| 写面向玩家的补丁说明 | `community-manager` |
| 头脑风暴新游戏创意 | 使用 `/brainstorm` 技能 |

### 3. 用斜杠命令处理常见任务

| 命令 | 作用 |
|---------|-------------|
| `/start` | 首次入门 — 询问你当前进度，引导到合适工作流 |
| `/design-review` | 审阅设计文档 |
| `/code-review` | 审代码质量与架构 |
| `/playtest-report` | 创建或分析试玩反馈 |
| `/balance-check` | 分析游戏平衡数据 |
| `/sprint-plan` | 创建或更新 Sprint 计划 |
| `/architecture-decision` | 创建 ADR |
| `/asset-audit` | 审计资源合规性 |
| `/milestone-review` | 审阅里程碑进度 |
| `/onboard` | 为某角色生成入职文档 |
| `/prototype` | 搭建可丢弃原型脚手架 |
| `/release-checklist` | 校验发布前清单 |
| `/changelog` | 根据 git 历史生成变更日志 |
| `/retrospective` | 运行 Sprint/里程碑复盘 |
| `/estimate` | 产出结构化工作量估算 |
| `/hotfix` | 带审计轨迹的紧急修复 |
| `/tech-debt` | 扫描、跟踪与排定技术债优先级 |
| `/scope-check` | 对照计划检测范围蔓延 |
| `/localize` | 本地化扫描、抽取与校验 |
| `/perf-profile` | 性能剖析与瓶颈识别 |
| `/gate-check` | 校验阶段就绪度（PASS/CONCERNS/FAIL） |
| `/project-stage-detect` | 分析项目状态、判断阶段、识别缺口 |
| `/reverse-document` | 从现有代码生成设计/架构文档 |
| `/setup-engine` | 配置引擎与版本，填充参考文档 |
| `/map-systems` | 将概念拆成系统、映射依赖、指导各系统 GDD |
| `/design-system` | 对单个游戏系统分节引导式撰写 GDD |
| `/team-combat` | 编排完整战斗团队管线 |
| `/team-narrative` | 编排完整叙事团队管线 |
| `/team-ui` | 编排完整 UI 团队管线 |
| `/team-release` | 编排完整发布团队管线 |
| `/team-polish` | 编排完整打磨团队管线 |
| `/team-audio` | 编排完整音频团队管线 |
| `/team-level` | 编排完整关卡制作管线 |
| `/launch-checklist` | 完整上线就绪校验 |
| `/patch-notes` | 生成面向玩家的补丁说明 |
| `/brainstorm` | 从零开始的引导式游戏概念构思 |

### 4. 新文档使用模板

模板位于 `.claude/docs/templates/`：

- `game-design-document.md` — 新机制与系统
- `architecture-decision-record.md` — 技术决策
- `risk-register-entry.md` — 新风险条目
- `narrative-character-sheet.md` — 新角色
- `test-plan.md` — 功能测试计划
- `sprint-plan.md` — Sprint 规划
- `milestone-definition.md` — 新里程碑
- `level-design-document.md` — 新关卡
- `game-pillars.md` — 核心设计支柱
- `art-bible.md` — 视觉风格参考
- `technical-design-document.md` — 各系统技术设计
- `post-mortem.md` — 项目/里程碑复盘
- `sound-bible.md` — 音频风格参考
- `release-checklist-template.md` — 平台发布清单
- `changelog-template.md` — 面向玩家的补丁说明
- `release-notes.md` — 面向玩家的发行说明
- `incident-response.md` — 线上事故响应手册
- `game-concept.md` — 初始游戏概念（MDA、SDT、心流、Bartle）
- `pitch-document.md` — 向干系人推介游戏
- `economy-model.md` — 虚拟经济设计（水龙头/水槽模型）
- `faction-design.md` — 派系身份、设定与玩法角色
- `systems-index.md` — 系统拆解与依赖映射
- `project-stage-report.md` — 项目阶段检测结果输出
- `design-doc-from-implementation.md` — 从实现反推为 GDD
- `architecture-doc-from-code.md` — 从代码反推为架构文档
- `concept-doc-from-prototype.md` — 从原型反推为概念文档

### 5. 遵守协作规则

1. 工作自上而下流动：总监 → 负责人 → 专员
2. 冲突向上升级
3. 跨部门工作由 `producer` 协调
4. 未经委派，Agent 不得修改职责域外文件
5. 重要决策均需留档

## 新项目的第一步

**不知道从哪开始？** 运行 `/start`。它会问清你当前进度，把你导向合适工作流，不对你的游戏、引擎或经验做预设。

若你已明确需求，可直接走对应路径：

### 路径 A：「还不知道做什么」

1. **运行 `/start`**（或 `/brainstorm open`）— 引导式创意探索：
   什么让你兴奋、你玩过的游戏、你的约束
   - 生成 3 个概念、协助选定其一、定义核心循环与支柱
   - 产出游戏概念文档并推荐引擎
2. **配置引擎** — 运行 `/setup-engine`（沿用头脑风暴的推荐）
   - 配置 CLAUDE.md、识别知识盲区、填充参考文档
   - 创建 `.claude/docs/technical-preferences.md`（命名约定、
     性能预算、引擎默认项）
   - 若引擎版本新于 LLM 训练数据，会从网络拉取当前文档，
     以便 Agent 建议正确 API
3. **校验概念** — 运行 `/design-review design/gdd/game-concept.md`
4. **拆解为系统** — 运行 `/map-systems` 映射系统与依赖
5. **逐系统设计** — 运行 `/design-system [system-name]`（或 `/map-systems next`）
   按依赖顺序写 GDD
6. **验证核心循环** — 运行 `/prototype [core-mechanic]`
7. **试玩验证** — 运行 `/playtest-report` 检验假设
8. **规划首个 Sprint** — 运行 `/sprint-plan new`
9. 开始制作

### 路径 B：「已经知道要做什么」

若已有游戏概念与引擎选择：

1. **配置引擎** — 运行 `/setup-engine [engine] [version]`
   （例如 `/setup-engine godot 4.6`）— 同时生成技术偏好
2. **撰写游戏支柱** — 委派给 `creative-director`
3. **拆解为系统** — 运行 `/map-systems` 枚举系统与依赖
4. **逐系统设计** — 运行 `/design-system [system-name]` 按依赖顺序写 GDD
5. **创建初始 ADR** — 运行 `/architecture-decision`
6. **在 `production/milestones/` 创建首个里程碑**
7. **规划首个 Sprint** — 运行 `/sprint-plan new`
8. 开始制作

### 路径 C：「知道游戏，还没定引擎」

若有概念但不确定引擎：

1. **无参运行 `/setup-engine`** — 会询问游戏需求（2D/3D、平台、
   团队规模、语言偏好等）并据此推荐引擎
2. 从路径 B 的第 2 步继续

### 路径 D：「已有项目在跑」

若已有设计文档、原型或代码：

1. **运行 `/start`**（或 `/project-stage-detect`）— 分析现有产物、
   识别缺口并建议下一步
2. **按需配置引擎** — 若尚未配置则运行 `/setup-engine`
3. **校验阶段就绪** — 运行 `/gate-check` 了解当前位置
4. **规划下一 Sprint** — 运行 `/sprint-plan new`

## 文件结构速查

```
CLAUDE.md                          -- 主配置（先读，约 60 行）
.claude/
  settings.json                    -- Claude Code 钩子与项目设置
  agents/                          -- 48 个 Agent 定义（YAML frontmatter）
  skills/                          -- 37 个斜杠命令定义（YAML frontmatter）
  hooks/                           -- 8 个由 settings.json 接线的 .sh 脚本
  rules/                           -- 11 个按路径区分的规则文件
  docs/
    quick-start.md                 -- 本文件
    technical-preferences.md       -- 项目专用标准（由 /setup-engine 生成）
    coding-standards.md            -- 代码与设计文档规范
    coordination-rules.md          -- Agent 协作规则
    context-management.md          -- 上下文预算与压缩说明
    review-workflow.md             -- 审阅与签字流程
    directory-structure.md         -- 项目目录布局
    agent-roster.md                -- 完整 Agent 列表与层级
    skills-reference.md            -- 全部斜杠命令
    rules-reference.md             -- 按路径的规则说明
    hooks-reference.md             -- 已启用的钩子
    agent-coordination-map.md      -- 完整委派与工作流地图
    setup-requirements.md          -- 系统前置（Git Bash、jq、Python）
    settings-local-template.md     -- 个人 settings.local.json 说明
    hooks-reference/               -- 钩子文档与 git hook 示例
    templates/                     -- 28 份文档模板
```
