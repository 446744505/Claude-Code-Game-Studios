# Unity 引擎 — 版本参考

| 字段 | 值 |
|-------|-------|
| **引擎版本** | Unity 6.3 LTS |
| **发布日期** | 2025 年 12 月 |
| **项目锁定日期** | 2026-02-13 |
| **文档最后核对** | 2026-02-13 |
| **LLM 知识截止** | 2025 年 5 月 |

## 知识缺口提示

LLM 的训练数据通常覆盖到约 Unity 2022 LTS（2022.3）。整个 Unity 6 系列（原 Unity 2023 Tech Stream）带来了大量模型并不了解的变更。在建议任何 Unity API 调用前，务必对照本目录交叉验证。

## 知识截止之后的版本时间线

| 版本 | 发布 | 风险等级 | 要点 |
|---------|---------|------------|-----------|
| 6.0 | 2024 年 10 月 | 高 | Unity 6 品牌焕新、新渲染能力、Entities 1.3、DOTS 改进 |
| 6.1 | 2024 年 11 月 | 中 | 缺陷修复、稳定性提升 |
| 6.2 | 2024 年 12 月 | 中 | 性能优化、新 Input System 改进 |
| 6.3 LTS | 2025 年 12 月 | 高 | 自 6.0 以来首个 LTS、可用于生产的 DOTS、图形能力增强 |

## 自 2022 LTS 至 Unity 6.3 LTS 的主要变化

### 破坏性变更
- **Entities/DOTS**：Entities 1.0+ 大规模 API 调整，ECS 模式整体重设
- **Input System**：旧版 Input Manager 已弃用，新 Input System 为默认
- **渲染**：URP/HDRP 重大升级，SRP Batcher 改进
- **Addressables**：资源管理工作流变化
- **脚本**：支持 C# 9，新 API 用法

### 新特性（知识截止之后）
- **DOTS**：可用于生产的 Entity Component System（Entities 1.3+）
- **图形**：增强的 URP/HDRP 管线、GPU Resident Drawer
- **多人**：Netcode for GameObjects 改进
- **UI Toolkit**：运行时 UI 可用于生产（新项目推荐替代 UGUI）
- **异步资源加载**：Addressables 性能改进
- **Web**：WebGPU 支持

### 已弃用系统
- **旧版 Input Manager**：请使用新 Input System 包
- **旧版粒子系统**：请使用 Visual Effect Graph
- **UGUI**：仍受支持，但新项目推荐 UI Toolkit
- **旧版 ECS（GameObjectEntity）**：由现代 DOTS/Entities 取代

## 已核对来源

- 官方文档：https://docs.unity3d.com/6000.0/Documentation/Manual/index.html
- Unity 6 发布页：https://unity.com/releases/unity-6
- Unity 6.3 LTS 公告：https://unity.com/blog/unity-6-3-lts-is-now-available
- 升级指南：https://docs.unity3d.com/6000.0/Documentation/Manual/upgrade-guides.html
- Unity 6 支持：https://unity.com/releases/unity-6/support
- C# API 参考：https://docs.unity3d.com/6000.0/Documentation/ScriptReference/index.html
