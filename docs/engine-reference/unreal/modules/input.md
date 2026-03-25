# Unreal Engine 5.7 — 输入模块参考

**上次核对：** 2026-02-13  
**知识缺口：** UE 5.7 默认使用 Enhanced Input（旧版输入已弃用）

---

## 概述

UE 5.7 输入体系：
- **Enhanced Input**（推荐，UE5 默认）：模块化、可重绑、基于上下文
- **Legacy Input**：已弃用，新项目应避免使用

---

## Enhanced Input 系统

### 启用 Enhanced Input

1. **启用插件**：`编辑 > 插件 > Enhanced Input`（UE5 中默认已启用）
2. **项目设置**：`引擎 > 输入 > 默认类 > 默认玩家输入类 = EnhancedPlayerInput`

---

### 创建 Input Action

1. 内容浏览器 > Input > Input Action
2. 命名（例如 `IA_Jump`、`IA_Move`）
3. 配置：
   - **Value Type**：Digital（bool）、Axis1D（float）、Axis2D（Vector2D）、Axis3D（Vector）

示例 Input Action：
- `IA_Jump`：Digital（bool）
- `IA_Move`：Axis2D（Vector2D）
- `IA_Look`：Axis2D（Vector2D）
- `IA_Fire`：Digital（bool）

---

### 创建 Input Mapping Context

1. 内容浏览器 > Input > Input Mapping Context
2. 命名（例如 `IMC_Default`）
3. 添加映射：
   - `IA_Jump` → 空格键
   - `IA_Move` → W/A/S/D（组合 X/Y）
   - `IA_Look` → 鼠标 XY
   - `IA_Fire` → 鼠标左键

---

### 在 C++ 中绑定输入

```cpp
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"
#include "InputActionValue.h"

class AMyCharacter : public ACharacter {
public:
    // 输入操作（在蓝图中指定资源）
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Input")
    TObjectPtr<UInputAction> MoveAction;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Input")
    TObjectPtr<UInputAction> LookAction;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Input")
    TObjectPtr<UInputAction> JumpAction;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Input")
    TObjectPtr<UInputMappingContext> DefaultMappingContext;

protected:
    virtual void BeginPlay() override {
        Super::BeginPlay();

        // 添加 Input Mapping Context
        if (APlayerController* PC = Cast<APlayerController>(Controller)) {
            if (UEnhancedInputLocalPlayerSubsystem* Subsystem =
                ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer())) {
                Subsystem->AddMappingContext(DefaultMappingContext, 0);
            }
        }
    }

    virtual void SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) override {
        Super::SetupPlayerInputComponent(PlayerInputComponent);

        UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
        if (EIC) {
            // 绑定操作
            EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
            EIC->BindAction(JumpAction, ETriggerEvent::Completed, this, &ACharacter::StopJumping);

            EIC->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
            EIC->BindAction(LookAction, ETriggerEvent::Triggered, this, &AMyCharacter::Look);
        }
    }

    void Move(const FInputActionValue& Value) {
        FVector2D MoveVector = Value.Get<FVector2D>();

        if (Controller) {
            AddMovementInput(GetActorForwardVector(), MoveVector.Y);
            AddMovementInput(GetActorRightVector(), MoveVector.X);
        }
    }

    void Look(const FInputActionValue& Value) {
        FVector2D LookVector = Value.Get<FVector2D>();

        if (Controller) {
            AddControllerYawInput(LookVector.X);
            AddControllerPitchInput(LookVector.Y);
        }
    }
};
```

---

## Input Trigger

### 触发类型

Input Action 可配置 Trigger，控制何时触发：
- **Pressed**：按下瞬间
- **Released**：松开瞬间
- **Hold**：按住达到时长
- **Tap**：短按
- **Pulse**：按住期间重复触发

### 在编辑器中添加 Trigger

1. 打开 Input Action 资源
2. Triggers > Add > 选择类型（例如 `Hold`）
3. 配置参数（例如 Hold Time = 0.5s）

---

## Input Modifier

### 修饰类型

Modifier 会变换输入数值：
- **Negate**：取反（-1 ↔ 1）
- **Dead Zone**：忽略小幅输入
- **Scalar**：乘以系数
- **Smooth**：随时间平滑

### 在编辑器中添加 Modifier

1. 打开 Input Action 资源
2. Modifiers > Add > 选择修饰（例如 `Negate`）
3. 配置参数

---

## Input Mapping Context（上下文切换）

### 多个 Context

```cpp
// 定义多个上下文
UPROPERTY(EditAnywhere, Category = "Input")
TObjectPtr<UInputMappingContext> DefaultContext;

UPROPERTY(EditAnywhere, Category = "Input")
TObjectPtr<UInputMappingContext> VehicleContext;

// 切换上下文
void EnterVehicle() {
    if (APlayerController* PC = Cast<APlayerController>(Controller)) {
        if (UEnhancedInputLocalPlayerSubsystem* Subsystem =
            ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer())) {
            Subsystem->RemoveMappingContext(DefaultContext);
            Subsystem->AddMappingContext(VehicleContext, 0);
        }
    }
}
```

---

## Legacy Input（已弃用）

### 旧版绑定方式

```cpp
// ❌ 已弃用：新项目请勿使用

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    // 旧版 Action 绑定
    PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);

    // 旧版轴绑定
    PlayerInputComponent->BindAxis("MoveForward", this, &AMyCharacter::MoveForward);
}

void MoveForward(float Value) {
    AddMovementInput(GetActorForwardVector(), Value);
}
```

**迁移：** 请改用 Enhanced Input。

---

## 手柄输入

### 配合 Enhanced Input 使用手柄

```cpp
// Input Mapping Context 中可配置：
// - IA_Move → 手柄左摇杆
// - IA_Look → 手柄右摇杆
// - IA_Jump → 手柄下面功能键（A / Cross）

// 无需改代码，只需在 Input Mapping Context 里添加手柄映射
```

---

## 触摸输入（移动端）

### 配合 Enhanced Input 使用触摸

```cpp
// Input Mapping Context 中可配置：
// - IA_Move → 触摸（虚拟摇杆）
// - IA_Look → 触摸（滑动）

// 虚拟控件可使用 Touch Interface 资源
```

---

## 运行时重绑输入

### 更换按键映射

```cpp
#include "PlayerMappableInputConfig.h"

// 获取子系统
UEnhancedInputLocalPlayerSubsystem* Subsystem = /* 获取子系统 */;

// 获取玩家可映射键位
FPlayerMappableKeySlot KeySlot = FPlayerMappableKeySlot(/*..*/);
FKey NewKey = EKeys::F; // 重绑到 F 键

// 应用新映射
Subsystem->AddPlayerMappedKey(/*..*/);
```

---

## 输入调试

### 调试输入

```cpp
// 控制台命令：
// showdebug input - 显示输入调试信息

// 记录输入值：
UE_LOG(LogTemp, Warning, TEXT("Move Input: %s"), *MoveVector.ToString());
```

---

## 常见写法

### 判断是否按下某键（临时调试用）

```cpp
// 仅建议用于调试（玩法逻辑不推荐）
if (GetWorld()->GetFirstPlayerController()->IsInputKeyDown(EKeys::SpaceBar)) {
    // 空格键当前为按下状态
}
```

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/enhanced-input-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/enhanced-input-action-and-input-mapping-context-in-unreal-engine/
