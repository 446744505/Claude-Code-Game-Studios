# Claude Code Game Studios -- 游戏工作室 Agent 架构

通过 48 个协同工作的 Claude Code 子代理管理独立游戏开发。
每个 Agent 负责特定领域，落实关注点分离与质量把控。

## 技术栈

- **引擎**：[选择：Godot 4 / Unity / Unreal Engine 5]
- **语言**：[选择：GDScript / C# / C++ / Blueprint]
- **版本控制**：Git，采用主干式（trunk-based）开发
- **构建系统**：[选定引擎后填写]
- **资源管线**：[选定引擎后填写]

> **说明**：Godot、Unity、Unreal 均有引擎专项 Agent，并配有更细分的子专家。请使用与所选引擎对应的一组。

## 项目结构

@.claude/docs/directory-structure.md

## 引擎版本参考

@docs/engine-reference/godot/VERSION.md

## 技术偏好

@.claude/docs/technical-preferences.md

## 协作规则

@.claude/docs/coordination-rules.md

## 协作协议

**由用户主导协作，而非自主执行。**
每项任务遵循：**提问 → 选项 → 决策 → 草案 → 批准**

- Agent 在使用 Write/Edit 工具前必须询问「是否可以将此写入 [filepath]？」
- Agent 在请求批准前必须展示草案或摘要
- 多文件变更须就完整变更集获得明确批准
- 未经用户指示不得执行 commit

完整协议与示例见 `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md`。

> **首次会话？** 若项目尚未配置引擎且尚无游戏概念，
> 请运行 `/start` 以开始引导式入门流程。

## 编码规范

@.claude/docs/coding-standards.md

## 上下文管理

@.claude/docs/context-management.md
