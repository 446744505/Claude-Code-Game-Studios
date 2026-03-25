# Unity 6.3 — Cinemachine

**上次核对：** 2026-02-13  
**状态：** 可用于生产  
**包：** `com.unity.cinemachine` v3.0+（Package Manager）

---

## 概述

**Cinemachine** 是 Unity 的虚拟摄像机系统，可在几乎不写脚本的情况下实现专业、动态的摄像机行为，是 Unity 摄像机工作的业界常用方案。

**适合用 Cinemachine 的场景：**
- 第三人称跟随摄像机
- 过场与镜头演出
- 摄像机混合与过渡
- 动态构图与取景
- 屏幕震动与摄像机特效

**⚠️ 知识缺口：** Cinemachine 3.0（Unity 6）相对 2.x 是一次大改版，大量 API 名称与组件已变更。

---

## 安装

### 通过 Package Manager 安装

1. `窗口 > Package Manager`
2. Unity Registry > 搜索 “Cinemachine”
3. 安装 `Cinemachine`（3.0 及以上版本）

---

## 核心概念

### 1. **虚拟摄像机（Virtual Cameras）**
- 定义摄像机行为（位置、旋转、镜头）
- 场景中可存在多台虚拟摄像机，同一时刻只有一台处于「生效」状态

### 2. **Cinemachine Brain**
- 挂在主 Camera 上的组件
- 在虚拟摄像机之间做混合
- 将虚拟摄像机的设置应用到 Unity Camera

### 3. **优先级**
- 虚拟摄像机带有优先级数值
- 优先级最高者生效
- 优先级变化时平滑混合

---

## 基础搭建

### 1. 在主摄像机上添加 Cinemachine Brain

```csharp
// 创建第一台虚拟摄像机时会自动添加
// 或手动：添加组件 > Cinemachine Brain
```

### 2. 创建虚拟摄像机

`GameObject > Cinemachine > Cinemachine Camera`

会生成带默认设置的 **CinemachineCamera** GameObject。

---

## 虚拟摄像机组件

### CinemachineCamera（Unity 6 / Cinemachine 3.0+）

```csharp
using Unity.Cinemachine;

public class CameraController : MonoBehaviour {
    public CinemachineCamera virtualCamera;

    void Start() {
        // 设置优先级（数值越高越优先）
        virtualCamera.Priority = 10;

        // 设置跟随目标
        virtualCamera.Follow = playerTransform;

        // 设置注视目标
        virtualCamera.LookAt = playerTransform;
    }
}
```

---

## 跟随模式（Body 组件）

### 第三人称跟随（Orbital Follow）

```csharp
// 在 Inspector 中：
// CinemachineCamera > Body > 3rd Person Follow

// 可配置项示例：
// - Shoulder Offset: (0.5, 0, 0) 过肩视角
// - Camera Distance: 5.0
// - Vertical Damping: 0.5（上下方向平滑）
```

### Framing Transposer（平滑跟随）

```csharp
// CinemachineCamera > Body > Position Composer

// 可配置项示例：
// - Screen Position: 居中 (0.5, 0.5)
// - Dead Zone: 目标在区域内时不移动摄像机
// - Damping: 跟随阻尼、平滑度
```

### Hard Lock（完全锁定）

```csharp
// CinemachineCamera > Body > Hard Lock to Target
// 摄像机位置与目标完全一致（无偏移、无阻尼）
```

---

## 瞄准模式（Aim 组件）

### Composer（框住目标）

```csharp
// CinemachineCamera > Aim > Composer

// 可配置项示例：
// - Tracked Object Offset: 例如对准头部而非脚底
// - Screen Position: 目标在画面中的位置
// - Dead Zone: 目标在区域内时不旋转
```

### Look At Target

```csharp
// CinemachineCamera > Aim > Rotate With Follow Target
// 摄像机旋转与跟随目标一致（例如第一人称）
```

---

## 摄像机之间混合

### 基于优先级的混合

```csharp
public CinemachineCamera normalCamera; // 优先级：10
public CinemachineCamera aimCamera;    // 优先级：5

void StartAiming() {
    // 提高瞄准摄像机优先级
    aimCamera.Priority = 15; // 变为当前生效
    // Brain 会自动从 normalCamera 混合到 aimCamera
}

void StopAiming() {
    aimCamera.Priority = 5; // 恢复常态
}
```

### 自定义混合时长

```csharp
// 创建 Custom Blends 资源：
// Assets > Create > Cinemachine > Cinemachine Blender Settings

// 在 Cinemachine Brain 中：
// - Custom Blends = 你的资源
// - 按摄像机对配置混合时间
```

---

## 摄像机震动

### Impulse Source（触发震动）

```csharp
using Unity.Cinemachine;

public class ExplosionShake : MonoBehaviour {
    public CinemachineImpulseSource impulseSource;

    void Explode() {
        // 触发摄像机震动
        impulseSource.GenerateImpulse();
    }
}
```

### Impulse Listener（接收震动）

```csharp
// 在 CinemachineCamera 上：
// 添加组件 > CinemachineImpulseListener

// Impulse Listener 会自动接收附近 Impulse Source 产生的震动
```

---

## Freelook 摄像机（第三人称 + 鼠标环视）

### Cinemachine Free Look

```csharp
// GameObject > Cinemachine > Cinemachine Free Look

// 会创建 3 个 Rig（上、中、下），按纵向输入混合
// 可配置：
// - Orbit Radius: 与目标的距离
// - Height Offset: 各 Rig 的高度偏移
// - X/Y Axis: 鼠标或摇杆输入
```

---

## State-Driven Camera（基于 Animator）

### Cinemachine State-Driven Camera

```csharp
// GameObject > Cinemachine > Cinemachine State-Driven Camera

// 可配置：
// - Animated Target: 带 Animator 的角色
// - Layer: 要监听的 Animator 层
// - State: 为各动画状态指定摄像机（Idle、Run、Jump 等）

// 摄像机随动画状态自动切换
```

---

## Dolly 轨道（过场）

### Cinemachine Dolly Track

```csharp
// 1. 创建样条：GameObject > Cinemachine > Cinemachine Spline

// 2. 创建 Dolly 摄像机：
//    GameObject > Cinemachine > Cinemachine Camera
//    Body > Spline Dolly
//    指定 Spline

// 3. 在样条上驱动 Dolly 位置（Timeline 或脚本）
```

---

## 常见模式

### 第三人称跟随摄像机

```csharp
// CinemachineCamera
// - Follow: 玩家 Transform
// - Body: 3rd Person Follow（肩偏移、距离约 5）
// - Aim: Composer（将玩家框在画面中心）
```

---

### 瞄准摄像机（拉近）

```csharp
// 普通摄像机（优先级 10）：
//   - Distance: 5.0

// 瞄准摄像机（优先级 5）：
//   - Distance: 2.0
//   - FOV: 更窄

// 脚本：
void StartAiming() {
    aimCamera.Priority = 15; // 混合到瞄准摄像机
}
```

---

### 过场镜头序列

```csharp
// 使用 Timeline：
// 1. 创建 Timeline（Assets > Create > Timeline）
// 2. 添加 Cinemachine Track
// 3. 将虚拟摄像机作为片段加入
// 4. Timeline 会在摄像机之间自动混合
```

---

## 从 Cinemachine 2.x（Unity 2021）迁移

### API 变更（Unity 6 / Cinemachine 3.0）

```csharp
// ❌ 旧版（Cinemachine 2.x）：
CinemachineVirtualCamera vcam;
vcam.m_Follow = target;

// ✅ 新版（Cinemachine 3.0+）：
CinemachineCamera vcam;
vcam.Follow = target; // API 更简洁
```

**主要变化：**
- `CinemachineVirtualCamera` → `CinemachineCamera`
- `m_Follow`、`m_LookAt` → `Follow`、`LookAt`（去掉 `m_` 前缀）
- 多个组件重命名，语义更清晰
- 性能更好

---

## 性能建议

- 控制同时激活的虚拟摄像机数量（按需启用）
- 用低优先级隐藏/待机，避免频繁销毁与创建
- 远离玩家时禁用不需要的虚拟摄像机

---

## 调试

### Cinemachine Debug

```csharp
// 窗口 > 分析 > Cinemachine Debugger
// 显示当前摄像机、混合信息、镜头质量等
```

---

## 参考来源
- https://docs.unity3d.com/Packages/com.unity.cinemachine@3.0/manual/index.html
- https://learn.unity.com/tutorial/cinemachine
