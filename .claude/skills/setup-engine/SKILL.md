---
name: setup-engine
description: "配置项目的游戏引擎与版本。在 CLAUDE.md 中固定引擎；识别知识盲区；当版本超出 LLM 训练数据时通过 WebSearch 填充引擎参考文档。"
argument-hint: "[引擎 版本] 或不带参数以进入引导选择"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, WebSearch, WebFetch, Task
---

调用本技能时：

## 1. 解析参数

三种模式：

- **完整规格**：`/setup-engine godot 4.6` — 已提供引擎与版本
- **仅引擎**：`/setup-engine unity` — 已提供引擎，版本将另行查询
- **无参数**：`/setup-engine` — 完全引导模式（引擎建议 + 版本）

---

## 2. 引导模式（无参数）

若未指定引擎，执行交互式引擎选择流程：

### 检查是否已有游戏概念
- 若存在 `design/gdd/game-concept.md`，则阅读 — 提取类型、规模、目标平台、美术风格、团队规模，以及来自 `/brainstorm` 的引擎建议（若有）
- 若不存在概念，告知用户：
  > "未找到游戏概念。建议先运行 `/brainstorm` 理清想做什么 — 它也会推荐引擎。或者直接描述你的游戏，我可以帮你选型。"

### 若用户想在没有概念的情况下选型，询问：
1. **做什么类型的游戏？**（2D、3D，或两者？）
2. **目标平台？**（PC、移动端、主机、Web？）
3. **团队规模与经验？**（单人新手、单人熟手、小团队？）
4. **是否有强烈的语言偏好？**（GDScript、C#、C++、可视化脚本？）
5. **引擎授权预算？**（仅免费，或可接受商业授权？）

### 给出建议

使用下列决策矩阵：

| 因素 | Godot 4 | Unity | Unreal Engine 5 |
|--------|---------|-------|-----------------|
| **最适合** | 2D、小规模 3D、单人/小团队 | 移动端、中等规模 3D、跨平台 | AAA 3D、照片级画面、大团队 |
| **语言** | GDScript（+ 扩展用 C#、C++） | C# | C++ / Blueprint |
| **成本** | 免费，MIT 许可 | 收入门槛以下免费 | 收入门槛以下免费，5% 分成 |
| **学习曲线** | 平缓 | 中等 | 陡峭 |
| **2D 支持** | 极佳（原生） | 良好（但以 3D 为先） | 可用但非理想 |
| **3D 上限** | 良好（快速迭代中） | 很好 | 业界顶尖 |
| **Web 导出** | 是（原生） | 是（有限） | 否 |
| **主机导出** | 经第三方 | 是（需授权） | 是 |
| **开源** | 是 | 否 | 源码可用 |

根据用户回答给出 1–2 个首选方案及理由。
由用户最终决定 — 不要强加推荐。

---

## 3. 查询当前版本

选定引擎后：

- 若已提供版本，直接使用
- 若未提供版本，用 WebSearch 查找最新稳定版：
  - 搜索：`"[engine] latest stable version [current year]"`
  - 与用户确认："当前 [engine] 最新稳定版为 [version]。是否采用？"

---

## 4. 更新 CLAUDE.md 技术栈

阅读 `CLAUDE.md` 并更新「Technology Stack」小节。将
`[CHOOSE]` 占位符替换为实际值：

**Godot：**
```markdown
- **Engine**: Godot [version]
- **Language**: GDScript (primary), C++ via GDExtension (performance-critical)
- **Build System**: SCons (engine), Godot Export Templates
- **Asset Pipeline**: Godot Import System + custom resource pipeline
```

**Unity：**
```markdown
- **Engine**: Unity [version]
- **Language**: C#
- **Build System**: Unity Build Pipeline
- **Asset Pipeline**: Unity Asset Import Pipeline + Addressables
```

**Unreal：**
```markdown
- **Engine**: Unreal Engine [version]
- **Language**: C++ (primary), Blueprint (gameplay prototyping)
- **Build System**: Unreal Build Tool (UBT)
- **Asset Pipeline**: Unreal Content Pipeline
```

---

## 5. 填充技术偏好

更新 `CLAUDE.md` 后，按引擎默认值创建或更新 `.claude/docs/technical-preferences.md`。先阅读现有模板，再填写：

### 引擎与语言小节
- 根据第 4 步的引擎选择填写

### 命名约定（引擎默认）

**Godot（GDScript）：**
- 类：PascalCase（如 `PlayerController`）
- 变量/函数：snake_case（如 `move_speed`）
- 信号：snake_case、过去时（如 `health_changed`）
- 文件：snake_case，与类名一致（如 `player_controller.gd`）
- 场景：PascalCase，与根节点一致（如 `PlayerController.tscn`）
- 常量：UPPER_SNAKE_CASE（如 `MAX_HEALTH`）

**Unity（C#）：**
- 类：PascalCase（如 `PlayerController`）
- 公共字段/属性：PascalCase（如 `MoveSpeed`）
- 私有字段：_camelCase（如 `_moveSpeed`）
- 方法：PascalCase（如 `TakeDamage()`）
- 文件：PascalCase，与类名一致（如 `PlayerController.cs`）
- 常量：PascalCase 或 UPPER_SNAKE_CASE

**Unreal（C++）：**
- 类：带前缀的 PascalCase（`A` Actor、`U` UObject、`F` struct）
- 变量：PascalCase（如 `MoveSpeed`）
- 函数：PascalCase（如 `TakeDamage()`）
- 布尔：`b` 前缀（如 `bIsAlive`）
- 文件：与类名一致但不带前缀（如 `PlayerController.h`）

### 其余小节
- Performance Budgets：保留为 `[TO BE CONFIGURED]`，并附建议：
  > "常见目标：60fps / 16.6ms 帧预算。现在要设定吗？"
- Testing：建议适合引擎的框架（Godot 用 GUT，Unity 用 NUnit，等）
- Forbidden Patterns / Allowed Libraries：保留占位

### 协作步骤
将填好的偏好展示给用户：
> "以下为 [engine] 的默认技术偏好。需要自定义哪些项，还是直接保存默认？"

写入文件前等待用户批准。

---

## 6. 判定知识盲区

判断引擎版本是否可能超出 LLM 训练数据。

**已知大致覆盖范围**（随模型更新请修订）：
- LLM 知识截止：**May 2025**
- Godot：训练数据大致覆盖至 ~4.3
- Unity：训练数据大致覆盖至 ~2023.x / 早期 6000.x
- Unreal：训练数据大致覆盖至 ~5.3 / 早期 5.4

将用户选定版本与上述基线对比：

- **在训练数据内** → `LOW RISK` — 参考文档可选但建议有
- **接近边界** → `MEDIUM RISK` — 建议准备参考文档
- **超出训练数据** → `HIGH RISK` — 必须有参考文档

告知用户所属类别及原因。

---

## 7. 填充引擎参考文档

### 若在训练数据内（LOW RISK）：

创建精简版 `docs/engine-reference/<engine>/VERSION.md`：

```markdown
# [Engine] — Version Reference

| Field | Value |
|-------|-------|
| **Engine Version** | [version] |
| **Project Pinned** | [today's date] |
| **LLM Knowledge Cutoff** | May 2025 |
| **Risk Level** | LOW — version is within LLM training data |

## Note

This engine version is within the LLM's training data. Engine reference
docs are optional but can be added later if agents suggest incorrect APIs.

Run `/setup-engine refresh` to populate full reference docs at any time.
```

勿创建 breaking-changes.md、deprecated-apis.md 等 — 性价比低且占用上下文。

### 若超出训练数据（MEDIUM 或 HIGH RISK）：

通过联网搜索创建完整参考文档集：

1. **搜索官方迁移/升级指南**：
   - `"[engine] [old version] to [new version] migration guide"`
   - `"[engine] [version] breaking changes"`
   - `"[engine] [version] changelog"`
   - `"[engine] [version] deprecated API"`

2. **从官方文档抓取并提取**：
   - 自训练截止版本至当前各版本间的 breaking changes
   - 已弃用 API 及替代方案
   - 新特性与最佳实践

3. **创建完整参考目录**：
   ```
   docs/engine-reference/<engine>/
   ├── VERSION.md              # Version pin + knowledge gap analysis
   ├── breaking-changes.md     # Version-by-version breaking changes
   ├── deprecated-apis.md      # "Don't use X → Use Y" tables
   ├── current-best-practices.md  # New practices since training cutoff
   └── modules/                # Per-subsystem references (create as needed)
   ```

4. **按联网结果填充各文件**，格式与既有参考文档一致。每个文件须含
   "Last verified: [date]" 页眉。

5. **模块文件**：仅在有显著变更的子系统下创建模块。勿创建空壳或极简模块文件。

---

## 8. 更新 CLAUDE.md 中的引用

更新「Engine Version Reference」下的 `@` 引用，指向正确引擎：

```markdown
## Engine Version Reference

@docs/engine-reference/<engine>/VERSION.md
```

若先前引用指向其他引擎（例如从 Godot 换到 Unity），须一并更新。

---

## 9. 更新 Agent 说明

针对所选引擎的专项 agent，确认是否包含「Version Awareness」小节。若无，按现有 Godot 专项 agent 中的模式补充。

该小节应要求 agent：
1. 阅读 `docs/engine-reference/<engine>/VERSION.md`
2. 在建议代码前核对已弃用 API
3. 核对相关版本迁移中的 breaking changes
4. 对不确定的 API 使用 WebSearch 核实

---

## 10. refresh 子命令

若以 `/setup-engine refresh` 调用：

1. 阅读现有 `docs/engine-reference/<engine>/VERSION.md`，获取当前引擎与版本
2. 使用 WebSearch 检查：
   - 自上次核验以来是否有新引擎发布
   - 迁移指南是否有更新
   - 是否有新弃用 API
3. 根据新发现更新所有参考文档
4. 更新所有已修改文件的 "Last verified" 日期
5. 汇报变更内容

---

## 11. 输出摘要

配置完成后输出：

```
引擎配置完成
=====================
引擎:            [name] [version]
知识风险:        [LOW/MEDIUM/HIGH]
参考文档:        [已创建/已跳过]
CLAUDE.md:       [已更新]
技术偏好:        [已创建/已更新]
Agent 配置:      [已核对]

后续步骤:
1. 查看 docs/engine-reference/<engine>/VERSION.md
2. [若来自 /brainstorm] 运行 /map-systems，将概念拆成各系统
3. [若来自 /brainstorm] 运行 /design-system，按小节协作撰写各系统 GDD
4. [若来自 /brainstorm] 运行 /prototype [core-mechanic] 验证核心循环
5. [若全新开始] 运行 /brainstorm 探索游戏概念
6. 创建首个里程碑：/sprint-plan new
```

---

## 约束

- 切勿臆测引擎版本 — 务必经 WebSearch 或用户确认
- 切勿在未询问的情况下覆盖已有参考文档 — 应追加或更新
- 若已存在其他引擎的参考文档，替换前须询问用户
- 修改 CLAUDE.md 前务必向用户展示将变更的内容
- 若 WebSearch 结果含糊，展示给用户并由其决定
