# Unreal Engine 5.7 — 当前最佳实践

**最后核对：** 2026-02-13

现代 UE5 模式，可能尚未进入 LLM 训练数据。  
以下为截至 UE 5.7 的生产级建议。

---

## 项目搭建

### 新项目使用 UE 5.7
- 最新能力：Megalights、可用于生产的 Substrate 与 PCG
- 更好的性能与稳定性

### 选择合适的渲染特性
- **Lumen**：实时全局光照（多数项目**推荐**）
- **Nanite**：高模虚拟化几何（细节环境**推荐**）
- **Megalights**：海量动态光源（复杂光照**推荐**）
- **Substrate**：模块化材质系统（新项目**推荐**）

---

## C++ 编码

### 使用现代 C++（UE5.7 为 C++20）

```cpp
// ✅ 使用 TObjectPtr<T>（UE5 类型安全指针）
UPROPERTY()
TObjectPtr<UStaticMeshComponent> MeshComp;

// ✅ 结构化绑定
if (auto [bSuccess, Value] = TryGetValue(); bSuccess) {
    // 使用 Value
}

// ✅ Concepts 与约束（C++20）
template<typename T>
concept Damageable = requires(T t, float damage) {
    { t.TakeDamage(damage) } -> std::same_as<void>;
};
```

### 使用 UPROPERTY() 参与垃圾回收

```cpp
// ✅ UPROPERTY 保证 GC 不会误删此引用
UPROPERTY()
TObjectPtr<AActor> MyActor;

// ❌ 裸指针可能变成悬空指针
AActor* MyActor; // 危险！可能被垃圾回收
```

### 使用 UFUNCTION() 暴露给 Blueprint

```cpp
// ✅ 可从 Blueprint 调用
UFUNCTION(BlueprintCallable, Category="Combat")
void TakeDamage(float Damage);

// ✅ 可在 Blueprint 中实现
UFUNCTION(BlueprintImplementableEvent, Category="Combat")
void OnDeath();
```

---

## Blueprint 最佳实践

### Blueprint 与 C++ 分工

- **C++**：核心玩法系统、性能关键路径、底层引擎交互
- **Blueprint**：快速原型、内容制作、数据驱动逻辑、策划工作流

### Blueprint 性能提示

```cpp
// ✅ 少用 Event Tick（开销大）
// 优先用 Timer 或事件

// ✅ 使用 Blueprint Nativization（Blueprint → C++）
// 项目设置 > 打包 > Blueprint Nativization

// ✅ 缓存频繁访问的组件
// 不要在每帧调用 GetComponent
```

---

## 渲染（UE 5.7）

### 全局光照使用 Lumen

```cpp
// 启用：项目设置 > 引擎 > 渲染 > 动态全局光照方法 = Lumen
// 实时 GI，无需烘焙光照贴图（推荐）
```

### 高模使用 Nanite

```cpp
// 在静态网格体上启用：细节 > Nanite 设置 > 启用 Nanite 支持
// 自动处理数百万三角形的 LOD（高细节网格推荐）
```

### 复杂光照使用 Megalights（UE 5.5+）

```cpp
// 启用：项目设置 > 引擎 > 渲染 > Megalights = 已启用
// 以较低成本支持海量动态光源
```

### 使用 Substrate 材质（5.7 已可用于生产）

```cpp
// 启用：项目设置 > 引擎 > Substrate > 启用 Substrate
// 模块化、物理准确的材质（新项目推荐）
```

---

## Enhanced Input 系统

### 配置 Enhanced Input

```cpp
// 1. 创建 Input Action（IA_Jump）
// 2. 创建 Input Mapping Context（IMC_Default）
// 3. 添加映射：IA_Jump → 空格键

// C++ 配置：
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"

void AMyCharacter::BeginPlay() {
    Super::BeginPlay();

    if (APlayerController* PC = Cast<APlayerController>(GetController())) {
        if (UEnhancedInputLocalPlayerSubsystem* Subsystem =
            ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer())) {
            Subsystem->AddMappingContext(DefaultMappingContext, 0);
        }
    }
}

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
    EIC->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
}

void AMyCharacter::Move(const FInputActionValue& Value) {
    FVector2D MoveVector = Value.Get<FVector2D>();
    AddMovementInput(GetActorForwardVector(), MoveVector.Y);
    AddMovementInput(GetActorRightVector(), MoveVector.X);
}
```

---

## Gameplay Ability System（GAS）

### 复杂玩法使用 GAS

```cpp
// ✅ GAS 适用于：技能、Buff、伤害计算、冷却等
// 模块化、可扩展、适合多人

// 安装：启用「Gameplay Abilities」插件

// 技能示例：
UCLASS()
class UGA_Fireball : public UGameplayAbility {
    GENERATED_BODY()

public:
    virtual void ActivateAbility(...) override {
        // 技能逻辑
        SpawnFireball();
        CommitAbility(); // 提交消耗/冷却
    }
};
```

---

## World Partition（大世界）

### 开放世界使用 World Partition

```cpp
// 启用：世界设置 > 启用 World Partition
// 根据玩家位置自动流送世界单元

// Data Layers：组织内容（例如「Gameplay」「Audio」「Lighting」）
// Runtime Data Layers：运行时加载/卸载
```

---

## Niagara（特效）

### 使用 Niagara（不用 Cascade）

```cpp
// 创建：内容浏览器 > 右键 > FX > Niagara System
// GPU 加速、基于节点的粒子系统（推荐）

// 生成粒子：
UNiagaraComponent* NiagaraComp = UNiagaraFunctionLibrary::SpawnSystemAtLocation(
    GetWorld(),
    ExplosionSystem,
    GetActorLocation()
);
```

---

## MetaSounds（音频）

### 程序化音频使用 MetaSounds

```cpp
// 创建：内容浏览器 > 右键 > 声音 > MetaSound Source
// 基于节点的音频，复杂逻辑下替代 Sound Cue（推荐）

// 播放 MetaSound：
UAudioComponent* AudioComp = UGameplayStatics::SpawnSound2D(
    GetWorld(),
    MetaSoundSource
);
```

---

## Replication（多人）

### 服务器权威模式

```cpp
// ✅ 客户端发输入，服务器校验并复制
UFUNCTION(Server, Reliable)
void Server_Move(FVector Direction);

void AMyCharacter::Server_Move_Implementation(FVector Direction) {
    // 服务器校验并应用移动
    AddMovementInput(Direction);
}

// ✅ 复制重要状态
UPROPERTY(Replicated)
int32 Health;

void AMyCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    DOREPLIFETIME(AMyCharacter, Health);
}
```

---

## 性能优化

### 使用对象池

```cpp
// ✅ 复用对象，避免频繁 Spawn/Destroy
TArray<AActor*> ProjectilePool;

AActor* GetPooledProjectile() {
    for (AActor* Proj : ProjectilePool) {
        if (!Proj->IsActive()) {
            Proj->SetActive(true);
            return Proj;
        }
    }
    // 池耗尽则新建
    return SpawnNewProjectile();
}
```

### 使用实例化静态网格体

```cpp
// ✅ 分层实例化静态网格体组件（HISM）
// 一次 Draw Call 渲染大量相同网格
UHierarchicalInstancedStaticMeshComponent* HISM = CreateDefaultSubobject<UHierarchicalInstancedStaticMeshComponent>(TEXT("Trees"));
for (int i = 0; i < 1000; i++) {
    HISM->AddInstance(FTransform(RandomLocation));
}
```

---

## 调试

### 使用日志

```cpp
// ✅ 结构化日志
UE_LOG(LogTemp, Warning, TEXT("Player health: %d"), Health);

// 自定义日志分类
DECLARE_LOG_CATEGORY_EXTERN(LogMyGame, Log, All);
DEFINE_LOG_CATEGORY(LogMyGame);
UE_LOG(LogMyGame, Error, TEXT("Critical error!"));
```

### 使用 Visual Logger

```cpp
// ✅ 可视化调试
#include "VisualLogger/VisualLogger.h"

UE_VLOG_SEGMENT(this, LogTemp, Log, StartPos, EndPos, FColor::Red, TEXT("Raycast"));
UE_VLOG_LOCATION(this, LogTemp, Log, TargetLocation, 50.f, FColor::Green, TEXT("Target"));
```

---

## 小结：UE 5.7 推荐技术栈

| 特性 | 2026 年选用 | 说明 |
|---------|------------------|-------|
| **光照** | Lumen + Megalights | 实时 GI，海量光源 |
| **几何** | Nanite | 高模，自动 LOD |
| **材质** | Substrate | 模块化，物理准确 |
| **输入** | Enhanced Input | 可重绑、模块化 |
| **特效** | Niagara | GPU 加速 |
| **音频** | MetaSounds | 程序化音频 |
| **世界流送** | World Partition | 大型开放世界 |
| **玩法** | Gameplay Ability System | 复杂技能与 Buff |

---

**参考：**
- https://docs.unrealengine.com/5.7/en-US/
- https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
