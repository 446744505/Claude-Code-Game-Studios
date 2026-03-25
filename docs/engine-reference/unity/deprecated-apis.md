# Unity 6.3 LTS — 已弃用 API

**最后核验：** 2026-02-13

已弃用 API 及其替代方案的快速查阅表。  
格式：**勿用 X** → **请改用 Y**

---

## 输入

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `Input.GetKey()` | `Keyboard.current[Key.X].isPressed` | 新输入系统 |
| `Input.GetKeyDown()` | `Keyboard.current[Key.X].wasPressedThisFrame` | 新输入系统 |
| `Input.GetMouseButton()` | `Mouse.current.leftButton.isPressed` | 新输入系统 |
| `Input.GetAxis()` | `InputAction` 回调 | 新输入系统 |
| `Input.mousePosition` | `Mouse.current.position.ReadValue()` | 新输入系统 |

**迁移：** 安装 `com.unity.inputsystem` 包。

---

## UI

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `Canvas`（UGUI） | `UIDocument`（UI Toolkit） | UI Toolkit 已可用于生产 |
| `Text` 组件 | `TextMeshPro` 或 UI Toolkit `Label` | 渲染更好，Draw Call 更少 |
| `Image` 组件 | 带背景的 UI Toolkit `VisualElement` | 样式更灵活 |

**迁移：** UGUI 仍可用，但新项目建议采用 UI Toolkit。

---

## DOTS/Entities

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `ComponentSystem` | `ISystem`（非托管） | Entities 1.0+ 全面重写 |
| `JobComponentSystem` | 搭配 `IJobEntity` 的 `ISystem` | 兼容 Burst |
| `GameObjectEntity` | 纯 ECS 工作流 | 不再从 GameObject 转换 |
| `EntityManager.CreateEntity()`（旧签名） | `EntityManager.CreateEntity(EntityArchetype)` | 显式 Archetype |
| `ComponentDataFromEntity<T>` | `ComponentLookup<T>` | Entities 1.0+ 更名 |

**迁移：** 参见 Entities 包迁移指南，需大规模重构。

---

## 渲染

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `CommandBuffer.DrawMesh()` | RenderGraph API | URP/HDRP 渲染 Pass |
| `OnPreRender()` / `OnPostRender()` | `RenderPipelineManager` 回调 | 兼容 SRP |
| `Camera.SetReplacementShader()` | 自定义渲染 Pass | SRP 中不支持 |

---

## 物理

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `Physics.RaycastAll()` | `Physics.RaycastNonAlloc()` | 避免 GC 分配 |
| `Rigidbody.velocity`（直接赋值） | `Rigidbody.AddForce()` | 物理更稳定 |

---

## 资源加载

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `Resources.Load()` | Addressables | 内存控制更好，异步加载 |
| 同步加载资源 | `Addressables.LoadAssetAsync()` | 非阻塞 |

---

## 动画

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| 旧版 Animation 组件 | Animator Controller | Mecanim 系统 |
| `Animation.Play()` | `Animator.Play()` | 状态机控制 |

---

## 粒子

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| 旧版粒子系统 | Visual Effect Graph | GPU 加速，性能更好 |

---

## 脚本

| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| `WWW` 类 | `UnityWebRequest` | 现代异步网络 |
| `Application.LoadLevel()` | `SceneManager.LoadScene()` | 场景管理 |

---

## 平台相关

### WebGL
| 已弃用 | 替代方案 | 说明 |
|--------|----------|------|
| WebGL 1.0 | WebGL 2.0 或 WebGPU | Unity 6+ 默认 WebGPU |

---

## 快速迁移示例

### 输入示例
```csharp
// ❌ 已弃用
if (Input.GetKeyDown(KeyCode.Space)) {
    Jump();
}

// ✅ 新输入系统
using UnityEngine.InputSystem;
if (Keyboard.current.spaceKey.wasPressedThisFrame) {
    Jump();
}
```

### 资源加载示例
```csharp
// ❌ 已弃用
var prefab = Resources.Load<GameObject>("Enemies/Goblin");

// ✅ Addressables
var handle = Addressables.LoadAssetAsync<GameObject>("Enemies/Goblin");
await handle.Task;
var prefab = handle.Result;
```

### UI 示例
```csharp
// ❌ 已弃用（UGUI）
GetComponent<Text>().text = "Score: 100";

// ✅ TextMeshPro
GetComponent<TextMeshProUGUI>().text = "Score: 100";

// ✅ UI Toolkit
rootVisualElement.Q<Label>("score-label").text = "Score: 100";
```

---

**资料来源：**
- https://docs.unity3d.com/6000.0/Documentation/Manual/deprecated-features.html
- https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/Migration.html
