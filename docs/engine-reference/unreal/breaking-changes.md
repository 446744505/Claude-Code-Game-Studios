# Unreal Engine 5.7 — 破坏性变更

**最后核对：** 2026-02-13

本文档记录 Unreal Engine 5.3（常见于模型训练数据）与 Unreal Engine 5.7（当前版本）之间的破坏性 API 变更与行为差异，按风险等级组织。

## 高风险 — 会破坏既有代码

### Substrate 材质系统（5.7 生产就绪）
**版本：** UE 5.5+（实验性），5.7（生产就绪）

Substrate 以模块化、物理准确的框架取代旧版材质系统。

```cpp
// ❌ 旧：旧版材质节点（仍可用但已弃用）
// 标准材质图：Base Color、Metallic、Roughness 等

// ✅ 新：Substrate 材质层
// 使用 Substrate 节点：Substrate Slab、Substrate Blend 等
// 模块化材质制作，具备真实物理精度
```

**迁移：** 在 `Project Settings > Engine > Substrate` 中启用 Substrate，并用 Substrate 节点重建材质。

---

### PCG（程序化内容生成）API 大改
**版本：** UE 5.7（生产就绪）

PCG 框架达到生产就绪状态，伴随重大 API 变更。

```cpp
// ❌ 旧：实验性 PCG API（5.7 之前）
// 旧节点类型，API 不稳定

// ✅ 新：生产级 PCG API（5.7+）
// 使用 FPCGContext、IPCGElement、新节点类型
// API 稳定，生产级工作流
```

**迁移：** 遵循 5.7 文档中的 PCG 迁移指南。若使用实验性 PCG，预计需要大量重构。

---

### Megalights 渲染系统
**版本：** UE 5.5+

新光照系统支持海量动态光源。

```cpp
// ❌ 旧：动态光源数量有限（clustered forward shading）
// 约 100–200 盏动态光源后性能明显下降

// ✅ 新：Megalights（5.5+）
// 数百万盏动态光源，性能开销很小
// 启用：Project Settings > Engine > Rendering > Megalights
```

**迁移：** 无需改代码，但光照表现可能不同。启用后请测试场景。

---

## 中风险 — 行为变化

### Enhanced Input（现为默认）
**版本：** UE 5.1+（推荐），5.7（默认）

Enhanced Input 现为默认输入系统。

```cpp
// ❌ 旧：旧版输入绑定（已弃用）
InputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);

// ✅ 新：Enhanced Input
SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
}
```

**迁移：** 将旧版输入绑定替换为 Enhanced Input Action。

---

### Nanite 默认启用倾向
**版本：** UE 5.0+（可选），5.7（鼓励）

Nanite 虚拟化几何现已成为静态网格体的推荐工作流。

```cpp
// 在静态网格体上启用 Nanite：
// Static Mesh Editor > Details > Nanite Settings > Enable Nanite Support
```

**迁移：** 将高面数网格转为 Nanite。在目标平台上测试性能。

---

## 低风险 — 弃用（仍可用）

### 旧版材质系统
**状态：** 已弃用但仍支持  
**替代：** Substrate 材质系统

旧材质仍可用，但新项目建议使用 Substrate。

---

### 旧 World Partition（UE4 风格）
**状态：** 已弃用  
**替代：** World Partition（UE5+）

大世界请使用 UE5 的 World Partition 系统。

---

## 平台相关破坏性变更

### Windows
- **UE 5.7**：DirectX 12 现为默认（旧版本多为 DX11）
- 为 DX12 兼容性更新着色器

### macOS
- **UE 5.5+**：需要 Metal 3（最低 macOS 13）

### 移动端
- **UE 5.7**：最低 Android API 提升至 26（Android 8.0）
- 最低 iOS 部署目标提升至 iOS 14

---

## 迁移检查清单

从 UE 5.3 升级到 UE 5.7 时：

- [ ] 审查 Substrate 材质（若准备采用新系统则转换）
- [ ] 审计 PCG 使用（若使用实验版则更新到生产 API）
- [ ] 测试 Megalights 性能（启用并基准测试）
- [ ] 将旧输入迁移到 Enhanced Input
- [ ] 将高面数网格转为 Nanite
- [ ] 为 DX12（Windows）或 Metal 3（macOS）更新着色器
- [ ] 核对最低平台版本（Android 8.0、iOS 14）
- [ ] 在目标硬件上测试 Lumen 与 Nanite 性能

---

**参考来源：**
- https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
- https://dev.epicgames.com/documentation/en-us/unreal-engine/upgrading-projects-to-newer-versions-of-unreal-engine
