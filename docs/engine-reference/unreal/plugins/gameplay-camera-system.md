# Unreal Engine 5.7 — Gameplay 摄像机系统

**最后核对：** 2026-02-13  
**状态：** ⚠️ 实验性（随 UE 5.5 引入）  
**插件：** `GameplayCameras`（内置，在插件中启用）

---

## 概述

**Gameplay 摄像机系统**是 UE 5.5 引入的模块化摄像机管理框架。  
它用灵活、基于节点的系统替代传统摄像机搭建方式，用于处理摄像机模式、混合以及随情境变化的摄像机行为。

**适合使用 Gameplay 摄像机的场景：**
- 动态摄像机行为（第三人称、瞄准、载具、过场）
- 随情境切换摄像机（战斗、探索、对话）
- 模式之间平滑混合
- 程序化摄像机运动（摄像机抖动、滞后、偏移）

**⚠️ 注意：** 该插件在 UE 5.5–5.7 中为实验性质，后续版本 API 可能变更。

---

## 核心概念

### 1. **Camera Rig（摄像机装备）**
- 定义摄像机配置（位置、旋转、FOV 等）
- 模块化节点图（类似材质编辑器）

### 2. **Camera Director（摄像机导演）**
- 管理当前激活的摄像机装备
- 处理不同摄像机装备之间的混合

### 3. **摄像机节点**
- 构成摄像机行为的积木：
  - **位置节点**：环绕（Orbit）、跟随（Follow）、固定位置
  - **旋转节点**：朝向目标（Look At）、匹配 Actor 旋转
  - **修饰器**：摄像机抖动、滞后、偏移

---

## 配置步骤

### 1. 启用插件

`编辑 > 插件 > Gameplay Cameras > 启用 > 重启`

### 2. 添加摄像机组件

```cpp
#include "GameplayCameras/Public/GameplayCameraComponent.h"

UCLASS()
class AMyCharacter : public ACharacter {
    GENERATED_BODY()

public:
    AMyCharacter() {
        // Create camera component
        CameraComponent = CreateDefaultSubobject<UGameplayCameraComponent>(TEXT("GameplayCamera"));
        CameraComponent->SetupAttachment(RootComponent);
    }

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera")
    TObjectPtr<UGameplayCameraComponent> CameraComponent;
};
```

---

## 创建 Camera Rig

### 1. 创建摄像机装备资源

1. 内容浏览器 > Gameplay > Gameplay Camera Rig  
2. 打开 Camera Rig 编辑器（基于节点的图）

### 2. 搭建摄像机装备（示例：第三人称）

**节点结构：**
```
Actor Position (Character)
  ↓
Orbit Node (Orbit around character)
  ↓
Offset Node (Shoulder offset)
  ↓
Look At Node (Look at character)
  ↓
Camera Output
```

---

## 摄像机节点

### 位置节点

#### Orbit 节点（第三人称）
- 环绕目标 Actor 运动  
- 可配置项：
  - **Orbit Distance**：与目标的距离（例如 300 单位）
  - **Pitch Range**：俯仰角最小/最大
  - **Yaw Range**：偏航角最小/最大

#### Follow 节点（平滑跟随）
- 带滞后地跟随目标  
- 可配置项：
  - **Lag Speed**：摄像机追上目标的速度
  - **Offset**：相对目标的固定偏移

#### Fixed Position 节点
- 世界空间中的静态摄像机位置

---

### 旋转节点

#### Look At 节点
- 将摄像机对准目标  
- 可配置项：
  - **Target**：要看的 Actor 或组件
  - **Offset**：注视偏移（例如对准头部而非脚部）

#### Match Actor Rotation
- 与目标 Actor 的旋转一致  
- 适用于第一人称或载具摄像机

---

### 修饰器节点

#### Camera Shake
- 增加程序化抖动（例如脚步、爆炸）  
- 可配置项：
  - **Shake Pattern**：Perlin 噪声、正弦波、自定义
  - **Amplitude**：抖动强度

#### Camera Lag
- 平滑抑制摄像机运动  
- 可配置项：
  - **Lag Speed**：阻尼系数（0 为即时，越大滞后越明显）

#### Offset 节点
- 在计算得到的位置基础上施加静态偏移  
- 常用于肩射视角偏移

---

## Camera Director（在不同 Rig 间切换）

### 指定摄像机装备

```cpp
#include "GameplayCameras/Public/GameplayCameraComponent.h"

void AMyCharacter::SetCameraMode(UGameplayCameraRig* NewRig) {
    if (CameraComponent) {
        CameraComponent->SetCameraRig(NewRig);
    }
}
```

### 在摄像机装备之间混合

```cpp
// Blend to aiming camera over 0.5 seconds
CameraComponent->BlendToCameraRig(AimingCameraRig, 0.5f);
```

---

## 示例：第三人称 + 瞄准

### 1. 创建两个摄像机装备

**第三人称 Rig：**
```
Actor Position → Orbit (distance: 300) → Look At → Output
```

**瞄准 Rig：**
```
Actor Position → Orbit (distance: 150) → Offset (shoulder) → Look At → Output
```

### 2. 瞄准时切换

```cpp
UPROPERTY(EditAnywhere, Category = "Camera")
TObjectPtr<UGameplayCameraRig> ThirdPersonRig;

UPROPERTY(EditAnywhere, Category = "Camera")
TObjectPtr<UGameplayCameraRig> AimingRig;

void StartAiming() {
    CameraComponent->BlendToCameraRig(AimingRig, 0.3f); // Blend over 0.3s
}

void StopAiming() {
    CameraComponent->BlendToCameraRig(ThirdPersonRig, 0.3f);
}
```

---

## 常见模式

### 越肩摄像机

```
Actor Position
  ↓
Orbit Node (distance: 250, yaw offset: 30°)
  ↓
Offset Node (X: 0, Y: 50, Z: 50) // Shoulder offset
  ↓
Look At Node (target: Character head)
  ↓
Output
```

---

### 载具摄像机

```
Vehicle Position
  ↓
Follow Node (lag: 0.2)
  ↓
Offset Node (behind vehicle: X: -400, Z: 150)
  ↓
Look At Node (target: Vehicle)
  ↓
Output
```

---

### 第一人称摄像机

```
Character Head Socket
  ↓
Match Actor Rotation
  ↓
Output
```

---

## 摄像机抖动

### 触发摄像机抖动

```cpp
#include "GameplayCameras/Public/GameplayCameraShake.h"

void TriggerExplosionShake() {
    if (APlayerController* PC = GetWorld()->GetFirstPlayerController()) {
        if (UGameplayCameraComponent* CameraComp = PC->FindComponentByClass<UGameplayCameraComponent>()) {
            CameraComp->PlayCameraShake(ExplosionShakeClass, 1.0f);
        }
    }
}
```

---

## 性能建议

- 限制摄像机抖动触发频率（不要每帧都触发）
- 谨慎使用摄像机滞后（高滞后值开销较大）
- 缓存摄像机装备引用（不要每帧查找）

---

## 调试

### 摄像机调试可视化

```cpp
// Console commands:
// GameplayCameras.Debug 1 - Show active camera rig info
// showdebug camera - Show camera debug info
```

---

## 从旧版摄像机迁移

### 旧的 Spring Arm + Camera 组件

```cpp
// ❌ OLD: Spring Arm Component
USpringArmComponent* SpringArm;
UCameraComponent* Camera;

// ✅ NEW: Gameplay Camera Component
UGameplayCameraComponent* CameraComponent;
// Build orbit + look-at rig in Camera Rig asset
```

---

## 限制（实验性状态）

- **API 不稳定**：UE 5.8+ 可能出现破坏性变更  
- **文档有限**：官方文档仍在完善  
- **Blueprint 支持**：以 C++ 为主（Blueprint 支持在改进中）  
- **上线风险**：发布前务必充分测试  

---

## 参考来源
- https://docs.unrealengine.com/5.7/en-US/gameplay-cameras-in-unreal-engine/
- UE 5.5+ 发行说明  
- **说明：** 本系统为实验性质，API 变更请以最新官方文档为准。
