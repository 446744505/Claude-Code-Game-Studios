# Unity 6.3 LTS — 破坏性变更

**最后核验：** 2026-02-13

本文档记录 Unity 2022 LTS（模型训练数据中较可能覆盖的版本）与 Unity 6.3 LTS（当前版本）之间的破坏性 API 变更与行为差异，并按风险等级组织。

## 高风险 — 会导致现有代码无法编译或运行

### Entities/DOTS API 全面重构
**版本：** Entities 1.0+（Unity 6.0+）

```csharp
// ❌ 旧写法（Unity 6 之前，GameObjectEntity 模式）
public class HealthComponent : ComponentData {
    public float Value;
}

// ✅ 新写法（Unity 6+，IComponentData）
public struct HealthComponent : IComponentData {
    public float Value;
}

// ❌ 旧：ComponentSystem
public class DamageSystem : ComponentSystem { }

// ✅ 新：ISystem（非托管，可配合 Burst）
public partial struct DamageSystem : ISystem {
    public void OnCreate(ref SystemState state) { }
    public void OnUpdate(ref SystemState state) { }
}
```

**迁移：** 遵循 Unity 的 ECS 迁移指南，通常需要较大规模架构调整。

---

### Input System — 旧版 Input 已弃用
**版本：** Unity 6.0+

```csharp
// ❌ 旧：Input 类（已弃用）
if (Input.GetKeyDown(KeyCode.Space)) { }

// ✅ 新：Input System 包
using UnityEngine.InputSystem;
if (Keyboard.current.spaceKey.wasPressedThisFrame) { }
```

**迁移：** 安装 Input System 包，将所有 `Input.*` 调用替换为新 API。

---

### URP/HDRP 渲染器特性 API 变更
**版本：** Unity 6.0+

```csharp
// ❌ 旧：ScriptableRenderPass.Execute 签名
public override void Execute(ScriptableRenderContext context, ref RenderingData data)

// ✅ 新：使用 RenderGraph API
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
```

**迁移：** 将自定义渲染通道更新为使用 RenderGraph API。

---

## 中风险 — 行为变化

### Addressables — 资源加载返回值
**版本：** Unity 6.2+

资源加载失败时，默认改为抛出异常，而不再返回 null。
请增加适当的异常处理，或使用 `TryLoad` 等变体。

```csharp
// ❌ 旧：失败时静默为 null
var handle = Addressables.LoadAssetAsync<Sprite>("key");
var sprite = handle.Result; // 失败时为 null

// ✅ 新：失败会抛异常，使用 try/catch 或 TryLoad
try {
    var handle = Addressables.LoadAssetAsync<Sprite>("key");
    var sprite = await handle.Task;
} catch (Exception e) {
    Debug.LogError($"Failed to load: {e}");
}
```

---

### Physics — 默认求解器迭代次数变更
**版本：** Unity 6.0+

默认求解器迭代次数已提高，以改善稳定性。
若依赖旧行为，请检查 `Physics.defaultSolverIterations`。

---

## 低风险 — 弃用（仍可用）

### UGUI（旧版 UI）
**状态：** 已弃用但仍受支持  
**替代：** UI Toolkit

UGUI 仍可使用，但新项目建议采用 UI Toolkit。

---

### 旧版粒子系统
**状态：** 已弃用  
**替代：** Visual Effect Graph（VFX Graph）

---

### 旧动画系统
**状态：** 已弃用  
**替代：** Animator Controller（Mecanim）

---

## 平台相关破坏性变更

### WebGL
- **Unity 6.0+**：WebGPU 现为默认（仍可选用 WebGL 2.0 回退）
- 请更新着色器以兼容 WebGPU

### Android
- **Unity 6.0+**：最低 API 级别提升至 24（Android 7.0）

### iOS
- **Unity 6.0+**：最低部署目标提升至 iOS 13

---

## 迁移检查清单

从 2022 LTS 升级到 Unity 6.3 LTS 时：

- [ ] 审计全部 DOTS/ECS 代码（很可能需要整体重写）
- [ ] 用 Input System 包替换 `Input` 类
- [ ] 将自定义渲染通道更新为 RenderGraph API
- [ ] 为 Addressables 调用增加异常处理
- [ ] 测试物理表现（求解器迭代次数已变）
- [ ] 新 UI 可考虑从 UGUI 迁往 UI Toolkit
- [ ] 更新 WebGL 着色器以适配 WebGPU
- [ ] 核对各平台最低版本要求（Android/iOS）

---

**参考来源：**
- https://docs.unity3d.com/6000.0/Documentation/Manual/upgrade-guides.html
- https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/upgrade-guide.html
