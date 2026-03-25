# Unreal Engine 5.7 — 可选插件与系统

**最近核验：** 2026-02-13

本文档索引 Unreal Engine 5.7 中可用的**可选插件与系统**。
它们不属于核心引擎，但在特定类型游戏中经常使用。

---

## 如何使用本指南

**✅ 具备详细文档** — 请参阅 `plugins/` 目录中的完整指南  
**🟡 仅简要概述** — 链接至官方文档，细节请使用 WebSearch  
**⚠️ 实验性** — 未来版本可能发生破坏性变更  
**📦 需要插件** — 须在「编辑 > 插件」中启用  

---

## 生产就绪系统（有详细文档）

### ✅ Gameplay Ability System（GAS）
- **用途：** 模块化能力系统（技能、属性、效果、冷却、消耗）
- **适用场景：** RPG、MOBA、带技能机制的射击游戏，以及一切以能力驱动的玩法
- **知识缺口：** GAS 自 UE4 起已较稳定，UE5 的改进在训练截止点之后
- **状态：** 生产就绪
- **插件：** `GameplayAbilities`（内置，在插件中启用）
- **详细文档：** [plugins/gameplay-ability-system.md](plugins/gameplay-ability-system.md)
- **官方：** https://docs.unrealengine.com/5.7/en-US/gameplay-ability-system-for-unreal-engine/

---

### ✅ CommonUI
- **用途：** 跨平台 UI 框架（自动处理手柄/鼠标/触摸输入路由）
- **适用场景：** 多平台游戏（主机 + PC）、与输入方式无关的 UI
- **知识缺口：** UE5+ 已可用于生产，主要改进在训练截止点之后
- **状态：** 生产就绪
- **插件：** `CommonUI`（内置，在插件中启用）
- **详细文档：** [plugins/common-ui.md](plugins/common-ui.md)
- **官方：** https://docs.unrealengine.com/5.7/en-US/commonui-plugin-for-advanced-user-interfaces-in-unreal-engine/

---

### ✅ Gameplay Camera System
- **用途：** 模块化镜头管理（镜头模式、混合、情境感知镜头）
- **适用场景：** 需要动态镜头行为的游戏（第三人称、瞄准、载具等）
- **知识缺口：** UE 5.5 新增，完全在训练截止点之后
- **状态：** ⚠️ 实验性（UE 5.5–5.7）
- **插件：** `GameplayCameras`（内置，在插件中启用）
- **详细文档：** [plugins/gameplay-camera-system.md](plugins/gameplay-camera-system.md)
- **官方：** https://docs.unrealengine.com/5.7/en-US/gameplay-cameras-in-unreal-engine/

---

### ✅ PCG（程序化内容生成）
- **用途：** 基于节点的程序化世界生成（植被、摆件、地形细节等）
- **适用场景：** 开放世界、程序化关卡、大规模环境填充
- **知识缺口：** UE 5.0–5.6 为实验性，5.7 起生产就绪
- **状态：** 生产就绪（自 UE 5.7 起）
- **插件：** `PCG`（内置，在插件中启用）
- **详细文档：** [plugins/pcg.md](plugins/pcg.md)
- **官方：** https://docs.unrealengine.com/5.7/en-US/procedural-content-generation-in-unreal-engine/

---

## 其他生产就绪插件（仅概述）

### 🟡 Mass Entity
- **用途：** 面向大规模 AI/人群的高性能 ECS（1 万+ 实体）
- **适用场景：** RTS、城市模拟、海量人群、大规模 AI
- **状态：** 生产就绪（UE 5.1+）
- **插件：** `MassEntity`、`MassGameplay`、`MassCrowd`
- **官方：** https://docs.unrealengine.com/5.7/en-US/mass-entity-in-unreal-engine/

---

### 🟡 Niagara Fluids
- **用途：** GPU 流体模拟（烟、火、液体）
- **适用场景：** 写实火焰/烟雾效果、水体模拟
- **状态：** 实验性 → 生产就绪（UE 5.4+）
- **插件：** `NiagaraFluids`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/niagara-fluids-in-unreal-engine/

---

### 🟡 Water 插件
- **用途：** 海洋、河流、湖泊渲染及浮力
- **适用场景：** 含水体、船只、游泳的游戏
- **状态：** 生产就绪（UE 5.0+）
- **插件：** `Water`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/water-system-in-unreal-engine/

---

### 🟡 Landmass 插件
- **用途：** 地形雕刻与地貌编辑
- **适用场景：** 大规模地形修改、程序化地貌
- **状态：** 生产就绪
- **插件：** `Landmass`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/landmass-plugin-in-unreal-engine/

---

### 🟡 Chaos Destruction
- **用途：** 实时破碎与破坏
- **适用场景：** 可破坏环境（墙体、建筑、物体）
- **状态：** 生产就绪（UE 5.0+）
- **插件：** `ChaosDestruction`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/destruction-in-unreal-engine/

---

### 🟡 Chaos Vehicle
- **用途：** 高级载具物理（轮式载具、悬挂等）
- **适用场景：** 竞速游戏、重度载具玩法
- **状态：** 生产就绪（替代 PhysX Vehicles）
- **插件：** `ChaosVehicles`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/chaos-vehicles-overview-in-unreal-engine/

---

### 🟡 Geometry Scripting
- **用途：** 运行时程序化网格生成与编辑
- **适用场景：** 动态网格创建、程序化建模
- **状态：** 生产就绪（UE 5.1+）
- **插件：** `GeometryScripting`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/geometry-scripting-in-unreal-engine/

---

### 🟡 Motion Design Tools
- **用途：** 动态图形、程序化动画、关键帧动画
- **适用场景：** UI 动画、程序化运动、关键帧序列
- **状态：** 实验性 → 生产就绪（UE 5.4+）
- **插件：** `MotionDesign`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/motion-design-mode-in-unreal-engine/

---

## 实验性插件（谨慎使用）

### ⚠️ AI Assistant（UE 5.7+）
- **用途：** 编辑器内 AI 引导与帮助
- **状态：** 实验性
- **插件：** 在 UE 5.7 设置中启用
- **官方：** 于 UE 5.7 发布公告中说明

---

### ⚠️ OpenXR（VR/AR）
- **用途：** 跨平台 VR/AR 支持
- **适用场景：** VR/AR 游戏
- **状态：** VR 生产就绪，AR 仍为实验性
- **插件：** `OpenXR`（内置）
- **官方：** https://docs.unrealengine.com/5.7/en-US/openxr-in-unreal-engine/

---

### ⚠️ Online Subsystem（EOS、Steam 等）
- **用途：** 与平台无关的在线服务（匹配、好友、成就等）
- **适用场景：** 带在线功能的多人游戏
- **状态：** 生产就绪
- **插件：** `OnlineSubsystem`、`OnlineSubsystemEOS`、`OnlineSubsystemSteam`
- **官方：** https://docs.unrealengine.com/5.7/en-US/online-subsystem-in-unreal-engine/

---

## 已弃用插件（新项目应避免）

### ❌ PhysX Vehicles
- **弃用说明：** 请改用 Chaos Vehicles
- **状态：** 遗留，不推荐

---

### ❌ Old Replication Graph
- **弃用说明：** 已由 Iris 取代（UE 5.1+）
- **状态：** 现代联网请使用 Iris

---

## 按需 WebSearch 策略

对于上文未列出的插件，当用户询问时可采用以下做法：

1. 使用 **WebSearch** 检索最新文档：`"Unreal Engine 5.7 [插件名]"`
2. 核实该插件是否：
   - 属于训练截止点之后（2025 年 5 月之后）的内容
   - 实验性还是生产就绪
   - 在 UE 5.7 中仍受支持
3. 可选：将结论缓存到 `plugins/[plugin-name].md` 供日后查阅

---

## 快速决策指南

**需要技能/能力/Buff** → **Gameplay Ability System（GAS）**  
**需要跨平台 UI（主机 + PC）** → **CommonUI**  
**需要动态镜头** → **Gameplay Camera System**  
**需要程序化世界** → **PCG**  
**需要大规模人群（数千 AI）** → **Mass Entity**  
**需要可破坏环境** → **Chaos Destruction**  
**需要载具** → **Chaos Vehicles**  
**需要水体/海洋** → **Water 插件**  
**需要 VR/AR** → **OpenXR**  

---

**最近更新：** 2026-02-13  
**引擎版本：** Unreal Engine 5.7  
**LLM 知识截止：** 2025 年 5 月  
