# Unreal Engine 5.7 — 已弃用 API

**最后核对：** 2026-02-13

已弃用 API 及其替代方案的速查表。  
格式：**不要用 X** → **改用 Y**

---

## 输入（Input）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `InputComponent->BindAction()` | Enhanced Input `BindAction()` | 新输入系统 |
| `InputComponent->BindAxis()` | Enhanced Input `BindAxis()` | 新输入系统 |
| `PlayerController->GetInputAxisValue()` | Enhanced Input Action Values | 新输入系统 |

**迁移：** 启用 Enhanced Input 插件，创建 Input Actions 与 Input Mapping Contexts。

---

## 渲染（Rendering）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| 旧版材质节点 | Substrate 材质节点 | Substrate 在 5.7 中已可用于生产 |
| Forward shading（默认） | Deferred + Lumen | UE5 默认使用 Lumen |
| 旧光照工作流 | Lumen Global Illumination | 实时光线追踪 GI |

---

## 世界构建（World Building）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| UE4 World Composition | World Partition（UE5） | 大世界流送 |
| Level Streaming Volumes | World Partition Data Layers | 更好的关卡流送 |

---

## 动画（Animation）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| 旧动画重定向 | IK Rig + IK Retargeter | UE5 重定向系统 |
| Legacy control rig | Control Rig 2.0 | 可用于生产的绑定 |

---

## 玩法（Gameplay）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `UGameplayStatics::LoadStreamLevel()` | World Partition streaming | 使用 Data Layers |
| 硬编码输入绑定 | Enhanced Input system | 可重绑定、模块化输入 |

---

## Niagara（VFX）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| Cascade 粒子系统 | Niagara | Cascade 已完全弃用 |

---

## 音频（Audio）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| 旧版 audio mixer | MetaSounds | 程序化音频系统 |
| Sound Cue（复杂逻辑） | MetaSounds | 更强大，基于节点 |

---

## 网络（Networking）

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `DOREPLIFETIME()`（基础用法） | `DOREPLIFETIME_CONDITION()` | 条件复制以优化性能 |

---

## C++ 脚本

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| 对 UObject 使用 `TSharedPtr<T>` | `TObjectPtr<T>` | UE5 类型安全指针 |
| 手动 RTTI 检查 | `Cast<T>()` / `IsA<T>()` | 类型安全转换 |

---

## 快速迁移示例

### 输入示例
```cpp
// ❌ 已弃用
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);
}

// ✅ Enhanced Input
#include "EnhancedInputComponent.h"

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    if (EIC) {
        EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
    }
}
```

### 材质示例
```cpp
// ❌ 已弃用：Legacy material
// 仍可使用标准材质图（但不推荐）

// ✅ Substrate Material
// 启用：Project Settings > Engine > Substrate > Enable Substrate
// 在材质编辑器中使用 Substrate 节点
```

### World Partition 示例
```cpp
// ❌ 已弃用：Level streaming volumes
// 手动加载/卸载关卡

// ✅ World Partition
// 启用：World Settings > Enable World Partition
// 使用 Data Layers 进行流送
```

### 粒子系统示例
```cpp
// ❌ 已弃用：Cascade
UParticleSystemComponent* PSC = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("Particles"));

// ✅ Niagara
UNiagaraComponent* NiagaraComp = CreateDefaultSubobject<UNiagaraComponent>(TEXT("Niagara"));
```

### 音频示例
```cpp
// ❌ 已弃用：复杂逻辑用 Sound Cue
// 使用 Sound Cue 编辑器节点

// ✅ MetaSounds
// 创建 MetaSound Source 资源，使用基于节点的音频
```

---

## 摘要：UE 5.7 技术栈

| 功能 | 建议采用（2026） | 避免使用（旧版） |
|------|------------------|------------------|
| **输入** | Enhanced Input | Legacy Input Bindings |
| **材质** | Substrate | Legacy Material System |
| **光照** | Lumen + Megalights | Lightmaps + 有限灯光数 |
| **粒子** | Niagara | Cascade |
| **音频** | MetaSounds | Sound Cue（用于逻辑） |
| **世界流送** | World Partition | World Composition |
| **动画重定向** | IK Rig + Retargeter | 旧重定向 |
| **几何体** | Nanite（高面数） | 标准静态网格 LOD |

---

**参考来源：**
- https://docs.unrealengine.com/5.7/en-US/deprecated-and-removed-features/
- https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
