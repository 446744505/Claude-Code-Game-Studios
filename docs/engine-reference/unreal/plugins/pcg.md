# Unreal Engine 5.7 — PCG（程序化内容生成）

**最近校验：** 2026-02-13  
**状态：** 可用于生产（截至 UE 5.7）  
**插件：** `PCG`（内置，在插件中启用）

---

## 概述

**程序化内容生成（PCG）** 是虚幻引擎中基于节点的框架，用于大规模生成程序化内容。其设计目标是向大型开放世界中填充植被、岩石、道具、建筑及其他环境细节。

**适合用 PCG 的场景：**
- 程序化放置植被（树、草、岩石）
- 基于生物群系的环境生成
- 道路/路径生成
- 建筑/结构放置
- 世界细节填充（道具、杂物）

**不要用 PCG 的场景：**
- 玩法逻辑（请用 Blueprint / C++）
- 一次性手工摆放（请用编辑器工具）

**⚠️ 说明：** PCG 在 UE 5.0–5.6 为实验性功能，在 UE 5.7 起可用于生产。

---

## 核心概念

### 1. **PCG 图表**
- 基于节点的图表（与材质编辑器类似）
- 定义生成规则

### 2. **PCG 组件**
- 置于关卡中，执行 PCG 图表
- 在指定体积内生成内容

### 3. **PCG 数据**
- 点数据（位置、旋转、缩放）
- 样条数据（路径、道路、河流）
- 体积数据（密度、生物群系遮罩）

### 4. **节点**
- **采样器（Samplers）**：生成点（网格、泊松盘、表面）
- **过滤器（Filters）**：按规则剔除点（密度、标签、边界）
- **修改器（Modifiers）**：变换点（偏移、旋转、缩放）
- **生成器（Spawners）**：在点处实例化网格体/Actor

---

## 设置

### 1. 启用插件

`编辑 > 插件 > PCG > 启用 > 重启`

### 2. 创建 PCG 体积

1. 放置 Actor > 体积 > PCG Volume  
2. 将体积缩放到期望的生成区域

### 3. 创建 PCG 图表

1. 内容浏览器 > PCG > PCG Graph  
2. 打开 PCG 图表编辑器

---

## 基本工作流

### 示例：森林生成

#### 1. 创建 PCG 图表

**节点布置：**
```
Input (Volume)
  ↓
Surface Sampler（在体积表面采样，每平方米点数：0.5）
  ↓
Density Filter（使用纹理遮罩或噪声）
  ↓
Static Mesh Spawner（树木网格体）
  ↓
Output
```

#### 2. 将图表指定给体积

1. 选中 PCG Volume  
2. 细节面板 > PCG 组件 > Graph = 你的 PCG 图表  
3. 点击「Generate」按钮

---

## 主要节点类型

### 采样器（点生成）

#### Grid Sampler（网格采样器）
- 规则网格上的点  
- 可配置项：  
  - **Grid Size**：点间距  
  - **Offset**：每个点的随机偏移  

#### Poisson Disk Sampler（泊松盘采样器）
- 满足最小距离的随机点  
- 可配置项：  
  - **Points Per m²**：密度  
  - **Min Distance**：点之间的间距  

#### Surface Sampler（表面采样器）
- 位于网格表面或地形上的点  
- 可配置项：  
  - **Points Per m²**：密度  
  - **Surface Only**：仅表面，非体积内部  

---

### 过滤器（剔除点）

#### Density Filter（密度过滤器）
- 根据密度值剔除点  
- 输入：纹理或噪声  
- 用途：生物群系遮罩、空地、路径  

#### Tag Filter（标签过滤器）
- 按标签过滤点  
- 用途：条件化生成  

#### Bounds Filter（边界过滤器）
- 仅保留边界内的点  
- 用途：将生成限制在特定区域  

---

### 修改器（点变换）

#### Rotate（旋转）
- 随机化点的旋转  
- 可配置项：  
  - **Min/Max Rotation**：各轴旋转范围  

#### Scale（缩放）
- 随机化点的缩放  
- 可配置项：  
  - **Min/Max Scale**：缩放范围  

#### Project to Ground（投影到地面）
- 将点对齐到地形表面  

---

### 生成器（网格体/Actor 实例化）

#### Static Mesh Spawner（静态网格体生成器）
- 在点处生成静态网格体  
- 可配置项：  
  - **Mesh List**：网格体数组（随机选取）  
  - **Culling Distance**：LOD / 剔除设置  

#### Actor Spawner（Actor 生成器）
- 在点处生成 Blueprint Actor  
- 用途：玩法 Actor、可交互物体  

---

## 数据源

### Landscape（地形）
- 将地形作为采样输入  
- 自动投影到地形高度  

### Splines（样条）
- 沿样条生成内容（道路、河流、路径）  
- 示例：沿路植树  

### Textures（纹理）
- 将纹理用作密度遮罩  
- 绘制生物群系、空地、区域  

---

## 生物群系示例（混交林）

### 图表结构

```
Input (Landscape)
  ↓
Surface Sampler（密度：1.0）
  ↓
┌─────────────────┬─────────────────┐
│ Tree Biome      │ Rock Biome      │
│（density > 0.5）│（density < 0.5）│
├─────────────────┼─────────────────┤
│ Tree Spawner    │ Rock Spawner    │
└─────────────────┴─────────────────┘
  ↓
Merge
  ↓
Output
```

---

## 基于样条的生成（带树的道路）

### 1. 创建 PCG 图表

```
Spline Input
  ↓
Spline Sampler（沿样条采样）
  ↓
Offset（相对路径偏移）
  ↓
Tree Spawner
  ↓
Output
```

### 2. 向 PCG Volume 添加样条组件

1. PCG Volume > 添加组件 > Spline  
2. 绘制样条路径  
3. PCG 图表读取样条数据  

---

## 运行时生成

### 从 C++ 触发生成

```cpp
#include "PCGComponent.h"

UPCGComponent* PCGComp = /* 获取 PCG 组件 */;
PCGComp->Generate(); // 执行 PCG 图表
```

### 流式生成（大型世界）

- PCG 会随 World Partition 自动流送  
- 仅为已加载的单元格生成内容  

---

## 性能

### 优化建议

- 对生成的网格体使用 **culling distance**（LOD）  
- 控制 **density**（点越少性能越好）  
- 对重复网格体使用 **Hierarchical Instanced Static Meshes（HISM）**  
- 大型世界启用 **streaming**  

### 调试性能

```cpp
// 控制台命令：
// pcg.graph.debug 1 - 显示 PCG 调试信息
// stat pcg - 显示 PCG 性能统计
```

---

## 常见模式

### 带空地的森林

```
Surface Sampler
  ↓
Density Filter（带空地的噪声纹理）
  ↓
Tree Spawner（松、橡、桦）
```

---

### 陡坡上的岩石

```
Landscape Input
  ↓
Surface Sampler
  ↓
Slope Filter（倾角 > 30°）
  ↓
Rock Spawner
```

---

### 沿路道具

```
Spline Input（道路样条）
  ↓
Spline Sampler
  ↓
Offset（道路一侧）
  ↓
Street Light Spawner
```

---

## 调试

### PCG 调试可视化

```cpp
// 控制台命令：
// pcg.debug.display 1 - 显示点与生成边界
// pcg.debug.colormode points - 按颜色区分点
```

### 图表调试

- PCG 图表编辑器 > Debug > Show Debug Points  
- 在图中每个节点处可视化点  

---

## 从 UE 5.6（实验性）迁移到 5.7（生产）

### API 变更

```cpp
// ❌ 旧版（5.6 实验性 API）：
// 部分节点重命名，API 不稳定

// ✅ 新版（5.7 生产 API）：
// 稳定的节点类型，已文档化的 API
```

**迁移：** 使用稳定的 5.7 节点重建 PCG 图表，并充分测试。

---

## 局限

- **不用于玩法逻辑**：游戏规则请用 Blueprint / C++  
- **大型图表可能较慢**：通过过滤器与降低密度优化  
- **运行时生成有开销**：在可能的情况下预生成  

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/procedural-content-generation-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/pcg-quick-start-in-unreal-engine/
- UE 5.7 发行说明（PCG 可用于生产的公告）
