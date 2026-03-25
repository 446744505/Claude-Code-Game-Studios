# Agent 名册

下列 agent 可用。每个 agent 在 `.claude/agents/` 中都有专用定义文件。请选用与当前任务最匹配的 agent。当任务跨多个领域时，应由协调用 agent（通常为 `producer` 或对应领域负责人）委派给各 specialist。

## 一级 — 领导层 Agent（Opus）
| Agent | 领域 | 适用场景 |
|-------|--------|-------------|
| `creative-director` | 高层创意愿景 | 重大创意决策、支柱冲突、基调/方向 |
| `technical-director` | 技术愿景 | 架构决策、技术栈选型、性能策略 |
| `producer` | 制作管理 | Sprint 规划、里程碑跟踪、风险管理、协调 |

## 二级 — 部门负责人 Agent（Sonnet）
| Agent | 领域 | 适用场景 |
|-------|--------|-------------|
| `game-designer` | 游戏设计 | 机制、系统、成长、经济、平衡 |
| `lead-programmer` | 代码架构 | 系统设计、代码审阅、API 设计、重构 |
| `art-director` | 视觉方向 | 风格指南、美术圣经、资产标准、UI/UX 方向 |
| `audio-director` | 音频方向 | 音乐方向、声音调色板、音频实现策略 |
| `narrative-director` | 叙事与文案 | 故事弧线、世界观构建、角色设计、对话策略 |
| `qa-lead` | 质量保证 | 测试策略、缺陷分流、发布就绪度、回归规划 |
| `release-manager` | 发布管线 | 构建管理、版本号、changelogs、部署、回滚 |
| `localization-lead` | 国际化 | 字符串外置、翻译管线、locale 测试 |

## 三级 — Specialist Agent（Sonnet 或 Haiku）
| Agent | 领域 | Model | 适用场景 |
|-------|--------|-------|-------------|
| `systems-designer` | 系统设计 | Sonnet | 具体机制落地、公式设计、循环 |
| `level-designer` | 关卡设计 | Sonnet | 关卡布局、节奏、遭遇设计、流程 |
| `economy-designer` | 经济/平衡 | Sonnet | 资源经济、loot 表、成长曲线 |
| `gameplay-programmer` | 玩法代码 | Sonnet | 功能实现、玩法系统代码 |
| `engine-programmer` | 引擎系统 | Sonnet | 核心引擎、渲染、物理、内存管理 |
| `ai-programmer` | AI 系统 | Sonnet | Behavior tree、pathfinding、NPC 逻辑、状态机 |
| `network-programmer` | 网络 | Sonnet | Netcode、replication、lag compensation、matchmaking |
| `tools-programmer` | 开发工具 | Sonnet | 编辑器扩展、管线工具、调试工具 |
| `ui-programmer` | UI 实现 | Sonnet | UI 框架、界面、控件、data binding |
| `technical-artist` | 技术美术 | Sonnet | Shader、VFX、优化、美术管线工具 |
| `sound-designer` | 声音设计 | Haiku | SFX 设计文档、音频事件列表、混音说明 |
| `writer` | 对话/设定 | Sonnet | 对话撰写、设定条目、物品描述 |
| `world-builder` | 世界/设定设计 | Sonnet | 世界规则、派系设计、历史、地理 |
| `qa-tester` | 测试执行 | Haiku | 编写测试用例、缺陷报告、测试检查清单 |
| `performance-analyst` | 性能 | Sonnet | Profiling、优化建议、内存分析 |
| `devops-engineer` | 构建/部署 | Haiku | CI/CD、构建脚本、版本控制工作流 |
| `analytics-engineer` | Telemetry | Sonnet | 事件跟踪、dashboard、A/B 测试设计 |
| `ux-designer` | UX 流程 | Sonnet | 用户流程、线框图、无障碍、输入处理 |
| `prototyper` | 快速原型 | Sonnet | 可丢弃原型、机制验证、可行性验证 |
| `security-engineer` | 安全 | Sonnet | 反作弊、漏洞利用防护、存档加密、网络安全 |
| `accessibility-specialist` | 无障碍 | Haiku | WCAG 合规、色盲模式、键位重映射、文字缩放 |
| `live-ops-designer` | 线上运营 | Sonnet | 赛季、活动、battle passes、留存、线上经济 |
| `community-manager` | 社区 | Haiku | Patch notes、玩家反馈、危机沟通、社区健康 |

## 引擎相关 Agent（使用与所选引擎对应的一套）

### 引擎负责人

| Agent | Engine | Model | 适用场景 |
| ---- | ---- | ---- | ---- |
| `unreal-specialist` | Unreal Engine 5 | Sonnet | Blueprint 与 C++、GAS 概览、UE 子系统、Unreal 优化 |
| `unity-specialist` | Unity | Sonnet | MonoBehaviour 与 DOTS、Addressables、URP/HDRP、Unity 优化 |
| `godot-specialist` | Godot 4 | Sonnet | GDScript 模式、节点/场景架构、signals、Godot 优化 |

### Unreal Engine 子领域 Specialist

| Agent | 子系统 | Model | 适用场景 |
| ---- | ---- | ---- | ---- |
| `ue-gas-specialist` | Gameplay Ability System | Sonnet | Abilities、gameplay effects、attribute sets、tags、prediction |
| `ue-blueprint-specialist` | Blueprint 架构 | Sonnet | BP/C++ 边界、图规范、命名、BP 优化 |
| `ue-replication-specialist` | 网络/Replication | Sonnet | Property replication、RPC、prediction、relevancy、带宽 |
| `ue-umg-specialist` | UMG/CommonUI | Sonnet | Widget 层级、data binding、CommonUI 输入、UI 性能 |

### Unity 子领域 Specialist

| Agent | 子系统 | Model | 适用场景 |
| ---- | ---- | ---- | ---- |
| `unity-dots-specialist` | DOTS/ECS | Sonnet | Entity Component System、Jobs、Burst compiler、hybrid renderer |
| `unity-shader-specialist` | Shader/VFX | Sonnet | Shader Graph、VFX Graph、URP/HDRP 定制、后处理 |
| `unity-addressables-specialist` | 资产管理 | Sonnet | Addressable groups、异步加载、内存、内容分发 |
| `unity-ui-specialist` | UI Toolkit/UGUI | Sonnet | UI Toolkit、UXML/USS、UGUI Canvas、data binding、跨平台输入 |

### Godot 子领域 Specialist

| Agent | 子系统 | Model | 适用场景 |
| ---- | ---- | ---- | ---- |
| `godot-gdscript-specialist` | GDScript | Sonnet | 静态类型、设计模式、signal、协程、GDScript 性能 |
| `godot-shader-specialist` | Shader/渲染 | Sonnet | Godot shading language、visual shader、粒子、后处理 |
| `godot-gdextension-specialist` | GDExtension | Sonnet | C++/Rust 绑定、原生性能、自定义节点、构建系统 |
