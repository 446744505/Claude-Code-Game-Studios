# Unity 6.3 LTS — 可选包与系统

**最近核对：** 2026-02-13

本文档索引 Unity 6.3 LTS 中可用的**可选包与系统**。
它们不属于核心引擎，但常用于特定类型的游戏。

---

## 如何使用本指南

**✅ 有详细文档** — 见 `plugins/` 目录中的完整指南  
**🟡 仅简要概述** — 链接至官方文档，细节请用 WebSearch  
**⚠️ 预览版** — 未来版本可能有破坏性变更  
**📦 需安装包** — 通过 Package Manager 安装  

---

## 生产就绪包（有详细文档）

### ✅ Cinemachine
- **用途：** 虚拟相机系统（动态镜头、过场、镜头混合）
- **适用场景：** 第三人称游戏、过场动画、复杂相机行为
- **知识缺口：** Cinemachine 3.0+（Unity 6）相对 2.x 有重大 API 变更
- **状态：** 生产就绪
- **包：** `com.unity.cinemachine`（Package Manager）
- **详细文档：** [plugins/cinemachine.md](plugins/cinemachine.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.cinemachine@3.0/manual/index.html

---

### ✅ Addressables
- **用途：** 高级资源管理（异步加载、远程内容、内存控制）
- **适用场景：** 大型项目、DLC、远程内容分发
- **知识缺口：** Unity 6 的改进与更好性能
- **状态：** 生产就绪
- **包：** `com.unity.addressables`（Package Manager）
- **详细文档：** [plugins/addressables.md](plugins/addressables.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.addressables@2.0/manual/index.html

---

### ✅ DOTS / Entities（ECS）
- **用途：** Data-Oriented Technology Stack（面向海量规模的高性能 ECS）
- **适用场景：** 数千实体、RTS、模拟类游戏
- **知识缺口：** Entities 1.3+（Unity 6）已生产就绪，相对 0.x 系重大重写
- **状态：** 生产就绪（截至 Unity 6.3 LTS）
- **包：** `com.unity.entities`（Package Manager）
- **详细文档：** [plugins/dots-entities.md](plugins/dots-entities.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html

---

## 其他生产就绪包（仅简要概述）

### 🟡 Input System（文中另有说明）
- **用途：** 现代输入处理（可重绑定、跨平台）
- **状态：** 生产就绪（Unity 6 默认）
- **包：** `com.unity.inputsystem`
- **文档：** 见 [modules/input.md](../modules/input.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html

---

### 🟡 UI Toolkit（文中另有说明）
- **用途：** 现代运行时 UI（类 HTML/CSS、高性能）
- **状态：** 生产就绪（Unity 6）
- **包：** 内置
- **文档：** 见 [modules/ui.md](../modules/ui.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.ui@2.0/manual/index.html

---

### 🟡 Visual Effect Graph（VFX Graph）
- **用途：** GPU 加速粒子系统（百万级粒子）
- **适用场景：** 大规模 VFX、火焰、烟雾、魔法、爆炸
- **状态：** 生产就绪
- **包：** `com.unity.visualeffectgraph`（仅 URP/HDRP）
- **官方：** https://docs.unity3d.com/Packages/com.unity.visualeffectgraph@17.0/manual/index.html

---

### 🟡 Shader Graph
- **用途：** 可视化着色器编辑器（节点式着色器创作）
- **适用场景：** 不写 HLSL 的自定义着色器
- **状态：** 生产就绪
- **包：** `com.unity.shadergraph`（URP/HDRP）
- **官方：** https://docs.unity3d.com/Packages/com.unity.shadergraph@17.0/manual/index.html

---

### 🟡 Timeline
- **用途：** 过场与序列编排（过场动画、脚本化事件）
- **适用场景：** 叙事驱动游戏、过场、脚本化序列
- **状态：** 生产就绪
- **包：** `com.unity.timeline`（内置）
- **官方：** https://docs.unity3d.com/Packages/com.unity.timeline@1.8/manual/index.html

---

### 🟡 Animation Rigging
- **用途：** 运行时 IK、程序化动画
- **适用场景：** 足部 IK、瞄准偏移、程序化肢体摆放
- **状态：** 生产就绪（Unity 6）
- **包：** `com.unity.animation.rigging`
- **官方：** https://docs.unity3d.com/Packages/com.unity.animation.rigging@1.3/manual/index.html

---

### 🟡 ProBuilder
- **用途：** 编辑器内 3D 建模（关卡原型、灰盒）
- **适用场景：** 快速原型、关卡体块搭建
- **状态：** 生产就绪
- **包：** `com.unity.probuilder`
- **官方：** https://docs.unity3d.com/Packages/com.unity.probuilder@6.0/manual/index.html

---

### 🟡 Netcode for GameObjects
- **用途：** Unity 官方多人联机网络
- **适用场景：** 多人游戏（客户端-服务器架构）
- **状态：** 生产就绪
- **包：** `com.unity.netcode.gameobjects`
- **官方：** https://docs-multiplayer.unity3d.com/netcode/current/about/

---

### 🟡 Burst Compiler
- **用途：** 面向 C# Jobs 的基于 LLVM 的编译器（大幅提升性能）
- **适用场景：** 性能关键代码、DOTS、Jobs System
- **状态：** 生产就绪
- **包：** `com.unity.burst`（随 DOTS 自动安装）
- **官方：** https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/index.html

---

### 🟡 Jobs System
- **用途：** 多线程 Job 调度（CPU 并行）
- **适用场景：** 性能优化、并行处理
- **状态：** 生产就绪
- **包：** 内置
- **官方：** https://docs.unity3d.com/Manual/JobSystem.html

---

### 🟡 Mathematics
- **用途：** SIMD 数学库（为 Burst 优化）
- **适用场景：** DOTS、高性能数学运算
- **状态：** 生产就绪
- **包：** `com.unity.mathematics`
- **官方：** https://docs.unity3d.com/Packages/com.unity.mathematics@1.3/manual/index.html

---

### 🟡 ML-Agents（Machine Learning）
- **用途：** 用强化学习训练 AI
- **适用场景：** 高级 AI 训练、程序化行为
- **状态：** 生产就绪
- **包：** `com.unity.ml-agents`
- **官方：** https://github.com/Unity-Technologies/ml-agents

---

### 🟡 Recorder
- **用途：** 录制实机画面、截图、动画片段
- **适用场景：** 宣传片、回放、调试录制
- **状态：** 生产就绪
- **包：** `com.unity.recorder`
- **官方：** https://docs.unity3d.com/Packages/com.unity.recorder@5.0/manual/index.html

---

## 预览 / 实验性包（谨慎使用）

### ⚠️ Splines
- **用途：** 运行时样条创建与编辑
- **适用场景：** 道路、路径、程序化内容
- **状态：** 生产就绪（Unity 6）
- **包：** `com.unity.splines`
- **官方：** https://docs.unity3d.com/Packages/com.unity.splines@2.6/manual/index.html

---

### ⚠️ Muse（AI Assistant）
- **用途：** AI 驱动的资源创作（贴图、精灵、动画等）
- **状态：** 预览（Unity 6）
- **包：** `com.unity.muse.*`
- **官方：** https://unity.com/products/muse

---

### ⚠️ Sentis（Neural Network Inference）
- **用途：** 在 Unity 中运行神经网络（AI 推理）
- **状态：** 预览
- **包：** `com.unity.sentis`
- **官方：** https://docs.unity3d.com/Packages/com.unity.sentis@2.0/manual/index.html

---

## 已弃用包（新项目应避免）

### ❌ UGUI（Canvas UI）
- **已弃用：** 仍受支持，但推荐 UI Toolkit
- **改用：** UI Toolkit

---

### ❌ Legacy Particle System
- **已弃用：** 请使用 Visual Effect Graph（VFX Graph）
- **改用：** VFX Graph

---

### ❌ Legacy Animation
- **已弃用：** 请使用 Animator（Mecanim）
- **改用：** Animator Controller

---

## 按需 WebSearch 策略

若用户询问的包未列于上文，可采用以下方式：

1. **WebSearch** 最新文档：`"Unity 6.3 [包名]"`
2. 核实该包是否：
   - 晚于训练截止（2025 年 5 月之后）
   - 预览版还是生产就绪
   - 在 Unity 6.3 LTS 中仍受支持
3. 可选：将结论缓存到 `plugins/[包名].md` 供日后查阅

---

## 快速决策指引

**需要虚拟相机** → **Cinemachine**  
**需要异步资源加载 / DLC** → **Addressables**  
**需要数千实体（RTS、模拟）** → **DOTS/Entities**  
**需要现代输入** → **Input System**（见 modules/input.md）  
**需要 GPU 粒子** → **Visual Effect Graph**  
**需要可视化着色器** → **Shader Graph**  
**需要过场编排** → **Timeline**  
**需要运行时 IK** → **Animation Rigging**  
**需要关卡原型** → **ProBuilder**  
**需要多人联机** → **Netcode for GameObjects**  

---

**最后更新：** 2026-02-13  
**引擎版本：** Unity 6.3 LTS  
**LLM 知识截止：** 2025 年 5 月
