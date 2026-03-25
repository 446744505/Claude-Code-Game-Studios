---
name: team-audio
description: "编排音频团队：`audio-director` + `sound-designer` + `technical-artist` + `gameplay-programmer`，完成从方向到落地的完整音频管线。"
argument-hint: "[要为其设计音频的功能或区域]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Task, AskUserQuestion, TodoWrite
---

调用本技能时，通过结构化管线编排音频团队。

**决策点：** 在每一步切换时，使用 `AskUserQuestion` 将子智能体的方案以可选项形式呈现给用户。在对话中写出智能体的完整分析，再用简短标签记录决策。进入下一步前须获得用户批准。

1. **阅读参数**，明确目标功能或区域（例如 `combat`、`main menu`、`forest biome`、`boss encounter`）。

2. **收集上下文**：
   - 阅读 `design/gdd/` 中与该功能相关的设计文档
   - 若存在，阅读 `design/gdd/sound-bible.md`
   - 阅读 `assets/audio/` 中现有音频资产清单
   - 阅读该区域已有的声音设计文档（如有）

## 如何委派

使用 Task 工具将每位成员作为子智能体启动：
- `subagent_type: audio-director` — 声音识别、情绪基调、音频调色板
- `subagent_type: sound-designer` — 音效规格、音频事件、混音编组
- `subagent_type: technical-artist` — 音频中间件、总线结构、内存预算
- `subagent_type: gameplay-programmer` — 音频管理器、玩法触发、自适应音乐

在每个智能体的提示中始终提供完整上下文（功能描述、现有音频资产、设计文档引用）。

3. **按顺序编排音频团队**：

### 步骤 1：音频方向（`audio-director`）
启动 `audio-director` 智能体以：
- 定义该功能/区域的声音识别
- 明确情绪基调与音频调色板
- 设定音乐方向（自适应层、分轨、过渡）
- 定义音频优先级与混音目标
- 建立自适应音频规则（战斗强度、探索、张力等）

### 步骤 2：声音设计（`sound-designer`）
启动 `sound-designer` 智能体以：
- 为每个音频事件编写详细音效规格
- 定义声音类别（环境、UI、玩法、音乐、对白）
- 规定每项声音的参数（音量范围、音高变化、衰减）
- 规划带触发条件的音频事件列表
- 定义混音编组与闪避（ducking）规则

### 步骤 3：技术实现（`technical-artist`）
启动 `technical-artist` 智能体以：
- 设计音频中间件集成（Wwise/FMOD/引擎原生）
- 定义音频总线路由结构
- 按平台规定音频资产的内存预算
- 规划流式加载与预加载策略
- 设计任何音频驱动的视觉效果

### 步骤 4：代码集成（`gameplay-programmer`）
启动 `gameplay-programmer` 智能体以：
- 实现音频管理器系统或审阅现有实现
- 将音频事件接入玩法触发
- 实现自适应音乐系统（若已规定）
- 设置音频遮挡/混响区域
- 为音频事件触发编写单元测试

4. **汇总音频设计文档**，合并各团队产出。

5. **保存至** `design/gdd/audio-[feature].md`。

6. **输出摘要**，包含：音频事件数量、预估资产数量、实现任务，以及团队成员之间仍待澄清的问题。
