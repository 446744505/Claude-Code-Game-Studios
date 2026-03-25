# Unreal Engine 5.7 — 动画模块参考

**最后核验：** 2026-02-13  
**知识缺口：** UE 5.7 动画制作改进、Control Rig 2.0

---

## 概述

UE 5.7 动画相关系统：
- **Animation Blueprint**：基于状态机的动画逻辑
- **Control Rig**：运行时程序化动画（在 UE5 中已可用于生产）
- **IK Rig + Retargeter**：现代重定向系统
- **Sequencer**：过场 / 镜头动画

---

## Animation Blueprint

### 创建 Animation Blueprint

1. 内容浏览器 > 右键 > Animation > Animation Blueprint
2. 选择父类：`AnimInstance`
3. 选择骨架

### 动画状态机

```cpp
// 在 Animation Blueprint 事件图中：
// - 状态机驱动动画状态（Idle、Walk、Run、Jump）
// - 使用 Blend Space 做方向性移动混合

// 在 C++ 中访问：
UAnimInstance* AnimInstance = Mesh->GetAnimInstance();
AnimInstance->Montage_Play(AttackMontage);
```

---

## 播放 Animation Montage

### Animation Montage

```cpp
// 播放 montage
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
AnimInstance->Montage_Play(AttackMontage, 1.0f);

// 停止 montage
AnimInstance->Montage_Stop(0.2f, AttackMontage);

// 检查 montage 是否正在播放
bool bIsPlaying = AnimInstance->Montage_IsPlaying(AttackMontage);
```

### Montage Notify 事件

```cpp
// 在 Animation Montage 中添加 notify（时间轴右键 > Add Notify > New Notify）
// 在 C++ 中实现：

UCLASS()
class UMyAnimInstance : public UAnimInstance {
    GENERATED_BODY()

public:
    UFUNCTION()
    void AnimNotify_AttackHit() {
        // 到达 notify 时调用
        DealDamage();
    }
};
```

---

## Blend Space

### 一维 Blend Space（按速度混合）

```cpp
// 创建：内容浏览器 > Animation > Blend Space 1D
// 横轴：Speed（0 = Idle，1 = Walk，2 = Run）
// 在关键位置添加动画

// 在 Anim Blueprint 中使用：
// - 从角色读取速度
// - 输入到 Blend Space
```

### 二维 Blend Space（方向性移动）

```cpp
// 创建：内容浏览器 > Animation > Blend Space
// 横轴：Direction X（-1 到 1）
// 纵轴：Direction Y（-1 到 1）
// 放置动画（前、后、左、右、斜向）
```

---

## Control Rig（程序化动画）

### 创建 Control Rig

1. 内容浏览器 > Animation > Control Rig
2. 选择骨架
3. 搭建 rig 层级（骨骼、控制器、IK）

### 在 Animation Blueprint 中使用 Control Rig

```cpp
// 在 Anim Blueprint 中添加「Control Rig」节点
// 指定 Control Rig 资源
// 在运行时程序化修改骨骼
```

### 在 C++ 中使用 Control Rig

```cpp
// 获取 Control Rig 组件
UControlRig* ControlRig = /* 从 animation instance 获取 */;

// 设置控制器数值
ControlRig->SetControlValue<FVector>(TEXT("IK_Hand_R"), TargetLocation);
```

---

## IK Rig 与重定向（UE5）

### 创建 IK Rig

1. 内容浏览器 > Animation > IK Rig
2. 选择骨架
3. 添加 IK 目标（手、脚等）
4. 配置 solver 链

### 重定向动画

1. 为源骨架创建 IK Rig
2. 为目标骨架创建 IK Rig
3. 创建 IK Retargeter 资源
4. 指定源与目标 IK Rig
5. 批量重定向动画

### 在 C++ 中重定向

```cpp
// 重定向主要在编辑器中完成
// 动画重定向一次后即可按常规方式使用
```

---

## Animation Notify State

### 自定义 Notify State（基于持续时长的事件）

```cpp
UCLASS()
class UAnimNotifyState_Invulnerable : public UAnimNotifyState {
    GENERATED_BODY()

public:
    virtual void NotifyBegin(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float TotalDuration, const FAnimNotifyEventReference& EventReference) override {
        // 开始无敌
        AMyCharacter* Character = Cast<AMyCharacter>(MeshComp->GetOwner());
        Character->bIsInvulnerable = true;
    }

    virtual void NotifyEnd(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, const FAnimNotifyEventReference& EventReference) override {
        // 结束无敌
        AMyCharacter* Character = Cast<AMyCharacter>(MeshComp->GetOwner());
        Character->bIsInvulnerable = false;
    }
};
```

---

## Skeletal Mesh 与 Socket

### 将物体挂接到 Socket

```cpp
// 在 Skeletal Mesh 编辑器中创建 socket（骨架树 > Add Socket）

// 将组件挂接到 socket
UStaticMeshComponent* Weapon = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Weapon"));
Weapon->SetupAttachment(GetMesh(), TEXT("hand_r_socket"));
```

---

## Animation Curve

### 使用 Animation Curve

```cpp
// 向动画添加曲线：
// 动画编辑器 > Curves > Add Curve

// 在 Anim Blueprint 或 C++ 中读取曲线值：
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
float CurveValue = AnimInstance->GetCurveValue(TEXT("MyCurve"));
```

---

## Root Motion

### 启用 Root Motion

```cpp
// 在 Animation Sequence 中：资源细节 > Root Motion > Enable Root Motion

// 在 Character 类中：
GetCharacterMovement()->bAllowPhysicsRotationDuringAnimRootMotion = true;
```

---

## Animation Layer（Linked Anim Graph）

### 使用 Linked Anim Layer

```cpp
// 为各层创建独立的 Anim Blueprint（例如上半身、下半身）
// 在主 Anim Blueprint 中链接：添加「Linked Anim Graph」节点

// 动态切换层：
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
AnimInstance->LinkAnimClassLayers(NewLayerClass);
```

---

## Sequencer（镜头动画）

### 创建 Sequence

1. 内容浏览器 > Cinematics > Level Sequence
2. 添加轨道：Camera、Character、Animation 等

### 从 C++ 播放 Sequence

```cpp
#include "LevelSequenceActor.h"
#include "LevelSequencePlayer.h"

ALevelSequenceActor* SequenceActor = /* 在关卡中生成或查找 */;
SequenceActor->GetSequencePlayer()->Play();
```

---

## 性能建议

### 动画优化

```cpp
// 骨骼网格体的 LOD（细节层次）
// 远处角色减少骨骼数量

// Anim Blueprint 优化：
// - 使用「Anim Node Relevancy」（不可见时跳过更新）
// - 在屏幕外时禁用更新：
GetMesh()->VisibilityBasedAnimTickOption = EVisibilityBasedAnimTickOption::OnlyTickPoseWhenRendered;
```

---

## 调试

### 动画调试可视化

```cpp
// 控制台命令：
// showdebug animation - 显示动画状态信息
// a.VisualizeSkeletalMeshBones 1 - 显示骨架骨骼

// 绘制调试用骨骼坐标系：
DrawDebugCoordinateSystem(GetWorld(), BoneLocation, BoneRotation, 50.0f, false, -1.0f, 0, 2.0f);
```

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/animation-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/control-rig-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/ik-rig-in-unreal-engine/
