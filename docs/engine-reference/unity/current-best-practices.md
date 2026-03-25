# Unity 6.3 LTS — 当前最佳实践

**最后核对：** 2026-02-13

面向现代 Unity 6 的做法，可能尚未出现在 LLM 训练数据中。
以下为截至 Unity 6.3 LTS 的生产环境建议。

---

## 项目搭建

### 生产环境使用 Unity 6.3 LTS
- **Tech Stream**（6.4+）：最新功能，稳定性较低
- **LTS**（6.3）：适合生产，两年支持期（至 2027 年 12 月）

### 选择合适的渲染管线
- **URP（Universal）**：移动端、跨平台、性能较好 ✅ 多数游戏推荐
- **HDRP（High Definition）**：高端 PC / 主机，偏写实
- **Built-in**：已弃用，新项目应避免

---

## 脚本

### 使用 C# 9+ 特性（Unity 6 支持 C# 9）

```csharp
// ✅ 数据用 record 类型
public record PlayerData(string Name, int Level, float Health);

// ✅ 仅 init 可写的属性
public class Config {
    public string GameMode { get; init; }
}

// ✅ 模式匹配
var result = enemy switch {
    Boss boss => boss.Enrage(),
    Minion minion => minion.Flee(),
    _ => null
};
```

### 用 Async/Await 加载资源

```csharp
// ✅ 现代 async 模式
public async Task<GameObject> LoadEnemyAsync(string key) {
    var handle = Addressables.LoadAssetAsync<GameObject>(key);
    return await handle.Task;
}
```

### 序列化使用 Source Generator（Unity 6+）

```csharp
// ✅ 源码生成序列化（更快，更少反射）
[GenerateSerializer]
public partial struct PlayerStats : IComponentData {
    public int Health;
    public int Mana;
}
```

---

## DOTS / ECS（Unity 6.3 LTS 已可用于生产）

### 使用 ISystem（不要用 ComponentSystem）

```csharp
// ✅ 现代非托管 ISystem（可配合 Burst）
public partial struct MovementSystem : ISystem {
    public void OnCreate(ref SystemState state) { }

    public void OnUpdate(ref SystemState state) {
        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<MoveSpeed>>()) {
            transform.ValueRW.Position += speed.ValueRO.Value * SystemAPI.Time.DeltaTime;
        }
    }
}
```

### 并行 Job 使用 IJobEntity

```csharp
// ✅ IJobEntity（替代 IJobForEach）
[BurstCompile]
public partial struct DamageJob : IJobEntity {
    public float DeltaTime;

    void Execute(ref Health health, in DamageOverTime dot) {
        health.Value -= dot.DamagePerSecond * DeltaTime;
    }
}

// 调度
var job = new DamageJob { DeltaTime = SystemAPI.Time.DeltaTime };
job.ScheduleParallel();
```

---

## 输入

### 使用 Input System 包（不要用旧版 Input）

```csharp
// ✅ Input Actions（可重绑定、跨平台）
using UnityEngine.InputSystem;

public class PlayerInput : MonoBehaviour {
    private PlayerControls controls;

    void Awake() {
        controls = new PlayerControls();
        controls.Gameplay.Jump.performed += ctx => Jump();
    }

    void OnEnable() => controls.Enable();
    void OnDisable() => controls.Disable();
}
```

在编辑器中创建 Input Actions 资源，并通过 Inspector 生成 C# 类。

---

## UI

### 运行时 UI 使用 UI Toolkit（Unity 6 已可用于生产）

```csharp
// ✅ UI Toolkit（新项目替代 UGUI）
using UnityEngine.UIElements;

public class MainMenu : MonoBehaviour {
    void OnEnable() {
        var root = GetComponent<UIDocument>().rootVisualElement;

        var playButton = root.Q<Button>("play-button");
        playButton.clicked += StartGame;

        var scoreLabel = root.Q<Label>("score");
        scoreLabel.text = $"High Score: {PlayerPrefs.GetInt("HighScore")}";
    }
}
```

**UXML**（结构）+ **USS**（样式）≈ HTML / CSS 式工作流。

---

## 资源管理

### 使用 Addressables（不要用 Resources）

```csharp
// ✅ Addressables（异步、省内存）
using UnityEngine.AddressableAssets;

public async Task SpawnEnemyAsync(string enemyKey) {
    var handle = Addressables.InstantiateAsync(enemyKey);
    var enemy = await handle.Task;

    // 清理：销毁时释放
    Addressables.ReleaseInstance(enemy);
}
```

**优点：** 异步加载、远程内容分发、内存更易控。

---

## 渲染

### 自定义 Pass 使用 RenderGraph API（URP / HDRP）

```csharp
// ✅ RenderGraph API（Unity 6+）
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData) {
    using (var builder = renderGraph.AddRasterRenderPass<PassData>("My Pass", out var passData)) {
        // 配置 Pass
        builder.SetRenderFunc((PassData data, RasterGraphContext context) => {
            // 执行命令
        });
    }
}
```

**替代：** 旧的 `CommandBuffer.Execute()` 模式。

---

## 性能

### 使用 Burst Compiler + Jobs System

```csharp
// ✅ Burst 编译的 Job（性能提升显著）
[BurstCompile]
struct ParticleUpdateJob : IJobParallelFor {
    public NativeArray<float3> Positions;
    public NativeArray<float3> Velocities;
    public float DeltaTime;

    public void Execute(int index) {
        Positions[index] += Velocities[index] * DeltaTime;
    }
}

// 调度
var job = new ParticleUpdateJob {
    Positions = positions,
    Velocities = velocities,
    DeltaTime = Time.deltaTime
};
job.Schedule(positions.Length, 64).Complete();
```

相较等价 C# 代码约 **快 20–100 倍**。

---

### 重复物体使用 GPU Instancing

```csharp
// ✅ GPU Instancing（大量物体、Draw Call 少）
Graphics.RenderMeshInstanced(
    new RenderParams(material),
    mesh,
    0,
    matrices // NativeArray<Matrix4x4>
);
```

---

## 内存管理

### Job 中使用 NativeContainers（不要用托管数组）

```csharp
// ✅ NativeArray（无 GC、可配合 Burst）
NativeArray<int> data = new NativeArray<int>(1000, Allocator.TempJob);
// ... 在 Job 中使用
data.Dispose(); // 需手动释放

// ✅ 或使用 using
using var data = new NativeArray<int>(1000, Allocator.TempJob);
// 自动 Dispose
```

---

## 多人游戏

### 使用 Netcode for GameObjects（官方方案）

```csharp
// ✅ Unity 官方联机方案
using Unity.Netcode;

public class Player : NetworkBehaviour {
    private NetworkVariable<int> health = new NetworkVariable<int>(100);

    [ServerRpc]
    public void TakeDamageServerRpc(int damage) {
        health.Value -= damage;
    }
}
```

**替代：** UNet（已弃用）、MLAPI（已更名为 Netcode for GameObjects）。

---

## 测试

### 使用 Unity Test Framework（基于 NUnit）

```csharp
// ✅ Play Mode 测试
[UnityTest]
public IEnumerator Player_TakesDamage_HealthDecreases() {
    var player = new GameObject().AddComponent<Player>();
    player.Health = 100;

    player.TakeDamage(25);
    yield return null; // 等待一帧

    Assert.AreEqual(75, player.Health);
}
```

---

## 调试

### 日志最佳实践

```csharp
// ✅ 结构化日志（Unity 6+）
using UnityEngine;

Debug.Log($"Player {playerName} scored {score} points");

// ✅ 调试代码用条件编译
#if UNITY_EDITOR || DEVELOPMENT_BUILD
    Debug.DrawRay(transform.position, direction, Color.red);
#endif
```

---

## 摘要：Unity 6 技术栈

| 能力 | 建议采用（2026） | 避免（旧方案） |
|------|------------------|----------------|
| **输入** | Input System 包 | `Input` 类 |
| **UI** | UI Toolkit | UGUI（Canvas） |
| **ECS** | ISystem + IJobEntity | ComponentSystem |
| **渲染** | URP + RenderGraph | Built-in 管线 |
| **资源** | Addressables | Resources |
| **Jobs** | Burst + IJobParallelFor | 重负载仍用协程 |
| **多人** | Netcode for GameObjects | UNet |

---

**参考来源：**
- https://docs.unity3d.com/6000.0/Documentation/Manual/BestPracticeGuides.html
- https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html
