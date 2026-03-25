# Unreal Engine 5.7 — UI 模块参考

**最后核实：** 2026-02-13  
**知识缺口：** UE 5.7 中 UMG 与 CommonUI 的改进

---

## 概述

UE 5.7 的 UI 体系：
- **UMG（Unreal Motion Graphics）**：可视化控件式 UI（推荐）
- **CommonUI**：跨平台、感知输入的 UI 框架（主机/PC）
- **Slate**：底层 C++ UI（引擎/编辑器 UI）

---

## UMG（Unreal Motion Graphics）

### 创建 Widget Blueprint

1. Content Browser > User Interface > Widget Blueprint  
2. 打开 Widget Designer  
3. 从 Palette 拖入控件：Button、Text、Image、ProgressBar 等  

---

## C++ 中的基础 UMG 配置

### 创建并显示控件

```cpp
#include "Blueprint/UserWidget.h"

UPROPERTY(EditAnywhere, Category = "UI")
TSubclassOf<UUserWidget> HealthBarWidgetClass;

void AMyCharacter::BeginPlay() {
    Super::BeginPlay();

    // 创建控件
    UUserWidget* HealthBarWidget = CreateWidget<UUserWidget>(GetWorld(), HealthBarWidgetClass);

    // 添加到视口
    HealthBarWidget->AddToViewport();
}
```

### 移除控件

```cpp
HealthBarWidget->RemoveFromParent();
```

---

## 从 C++ 访问控件元素

### 绑定到控件元素

```cpp
UCLASS()
class UMyHealthWidget : public UUserWidget {
    GENERATED_BODY()

public:
    // ✅ 绑定到控件元素（须与 Widget Blueprint 中的名称一致）
    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UTextBlock> HealthText;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UProgressBar> HealthBar;

    void UpdateHealth(int32 CurrentHealth, int32 MaxHealth) {
        HealthText->SetText(FText::FromString(FString::Printf(TEXT("%d / %d"), CurrentHealth, MaxHealth)));
        HealthBar->SetPercent((float)CurrentHealth / MaxHealth);
    }
};
```

---

## 常用 UMG 控件

### 文本块（Text Block）

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UTextBlock> ScoreText;

ScoreText->SetText(FText::FromString(TEXT("Score: 100")));
ScoreText->SetColorAndOpacity(FLinearColor::Green);
```

### 按钮（Button）

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UButton> PlayButton;

void NativeConstruct() override {
    Super::NativeConstruct();

    // 绑定按钮点击
    PlayButton->OnClicked.AddDynamic(this, &UMyMenuWidget::OnPlayClicked);
}

UFUNCTION()
void OnPlayClicked() {
    UE_LOG(LogTemp, Warning, TEXT("Play clicked"));
}
```

### 图像（Image）

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UImage> PlayerAvatar;

PlayerAvatar->SetBrushFromTexture(AvatarTexture);
PlayerAvatar->SetColorAndOpacity(FLinearColor::White);
```

### 进度条（Progress Bar）

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UProgressBar> HealthBar;

HealthBar->SetPercent(0.75f); // 75%
HealthBar->SetFillColorAndOpacity(FLinearColor::Red);
```

### 滑块（Slider）

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<USlider> VolumeSlider;

void NativeConstruct() override {
    Super::NativeConstruct();
    VolumeSlider->OnValueChanged.AddDynamic(this, &UMyWidget::OnVolumeChanged);
}

UFUNCTION()
void OnVolumeChanged(float Value) {
    // Value 范围为 0.0 - 1.0
    UE_LOG(LogTemp, Warning, TEXT("Volume: %f"), Value);
}
```

### EditableTextBox（输入框）

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UEditableTextBox> PlayerNameInput;

void NativeConstruct() override {
    Super::NativeConstruct();
    PlayerNameInput->OnTextChanged.AddDynamic(this, &UMyWidget::OnNameChanged);
}

UFUNCTION()
void OnNameChanged(const FText& Text) {
    FString PlayerName = Text.ToString();
}
```

---

## UMG 动画

### 播放动画

```cpp
UPROPERTY(Transient, meta = (BindWidgetAnim))
TObjectPtr<UWidgetAnimation> FadeInAnimation;

void ShowUI() {
    PlayAnimation(FadeInAnimation);
}
```

### 停止动画

```cpp
StopAnimation(FadeInAnimation);
```

---

## Canvas Panel（布局）

### Canvas Panel（绝对定位）

```cpp
// 在 Widget Blueprint 中用于绝对定位
// 将控件锚定到角/边以实现响应式 UI
```

### Vertical Box（垂直堆叠）

```cpp
// 子项自动垂直堆叠
```

### Horizontal Box（水平堆叠）

```cpp
// 子项自动水平堆叠
```

### Grid Panel（网格布局）

```cpp
// 将子项排列成网格
```

---

## 世界空间 UI（3D UI）

### Widget Component（世界中的 3D UI）

```cpp
#include "Components/WidgetComponent.h"

UWidgetComponent* HealthBarWidget = CreateDefaultSubobject<UWidgetComponent>(TEXT("HealthBar"));
HealthBarWidget->SetupAttachment(RootComponent);
HealthBarWidget->SetWidgetClass(HealthBarWidgetClass);
HealthBarWidget->SetWidgetSpace(EWidgetSpace::World); // 3D 世界空间
HealthBarWidget->SetDrawSize(FVector2D(200, 50));
```

---

## UMG 中的输入处理

### 覆盖键盘输入

```cpp
UCLASS()
class UMyWidget : public UUserWidget {
    GENERATED_BODY()

public:
    virtual FReply NativeOnKeyDown(const FGeometry& InGeometry, const FKeyEvent& InKeyEvent) override {
        if (InKeyEvent.GetKey() == EKeys::Escape) {
            // 处理 Esc 键
            CloseMenu();
            return FReply::Handled();
        }
        return Super::NativeOnKeyDown(InGeometry, InKeyEvent);
    }
};
```

---

## CommonUI（跨平台输入）

### 启用 CommonUI 插件

```cpp
// 启用：Edit > Plugins > CommonUI
// 重启编辑器
```

### 使用 CommonUI 控件

```cpp
// CommonUI 控件：
// - CommonActivatableWidget：界面/菜单基类
// - CommonButtonBase：感知输入的按钮（手柄 + 鼠标）
// - CommonTextBlock：带样式的文本
```

### CommonActivatableWidget 示例

```cpp
UCLASS()
class UMyMenuWidget : public UCommonActivatableWidget {
    GENERATED_BODY()

public:
    virtual void NativeOnActivated() override {
        Super::NativeOnActivated();
        // 菜单已激活（显示）
    }

    virtual void NativeOnDeactivated() override {
        Super::NativeOnDeactivated();
        // 菜单已停用（隐藏）
    }
};
```

---

## HUD 类（UMG 的替代方案）

### 创建 HUD

```cpp
UCLASS()
class AMyHUD : public AHUD {
    GENERATED_BODY()

public:
    virtual void DrawHUD() override {
        Super::DrawHUD();

        // 绘制文本
        DrawText(TEXT("Score: 100"), FLinearColor::White, 50, 50);

        // 绘制纹理
        DrawTexture(CrosshairTexture, Canvas->SizeX / 2, Canvas->SizeY / 2, 32, 32);
    }
};
```

---

## 性能建议

### 优化 UMG

```cpp
// Invalidation Box：仅在内容变化时重绘
// 在 Widget Blueprint 中添加 “Invalidation Box” 控件

// 若不需要则关闭 Tick
bIsFocusable = false;
SetVisibility(ESlateVisibility::Collapsed); // Collapsed = 不参与渲染
```

---

## 调试

### UI 调试命令

```cpp
// 控制台命令：
// widget.debug - 显示控件层级
// Slate.ShowDebugOutlines 1 - 显示控件边界
// stat slate - 显示 Slate 性能
```

---

## 来源
- https://docs.unrealengine.com/5.7/en-US/umg-ui-designer-for-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/commonui-plugin-for-advanced-user-interfaces-in-unreal-engine/
