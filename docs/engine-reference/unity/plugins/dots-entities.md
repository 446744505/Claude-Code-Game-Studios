# Unity 6.3 — DOTS / Entities（ECS）

**上次核对：** 2026-02-13  
**状态：** 可用于生产（Entities 1.3+，Unity 6.3 LTS）  
**包：** `com.unity.entities`（Package Manager）

---

## 概述

**DOTS（Data-Oriented Technology Stack，数据导向技术栈）** 是 Unity 的高性能 **ECS（Entity Component System，实体组件系统）** 框架，面向大规模场景（数千到上万个实体）。

**适合用 DOTS：**
- RTS（大量单位）
- 模拟（人群、交通、物理）
- 程序化内容生成
- 性能敏感系统

**不适合用 DOTS：**
- 体量很小的游戏（开销不划算）
- 需要频繁做结构性变更的玩法
- 大量依赖 UnityEngine API（用 MonoBehaviour 更简单）

**⚠️ 知识断层：** Entities 1.0+（Unity 6）相对 0.x 为完全重写，许多针对 Entities 0.x 的教程已过时。

---

## 安装

### 通过 Package Manager 安装

1. `Window > Package Manager`
2. Unity Registry > 搜索 “Entities”
3. 安装：
   - `Entities`（ECS 核心）
   - `Burst`（LLVM 编译器）
   - `Jobs`（随依赖自动安装）
   - `Mathematics`（SIMD 数学）

---

## 核心概念

### 1. **Entity（实体）**
- 轻量 ID（int）
- 无行为，仅作标识

### 2. **Component（组件）**
- 仅数据（无方法）
- 实现 `IComponentData` 的结构体

### 3. **System（系统）**
- 对组件执行逻辑
- 实现 `ISystem` 的结构体

### 4. **Archetype（原型）**
- 组件类型的唯一组合
- 组件组合相同的实体共享同一原型

---

## 基础 ECS 模式

### 定义组件

```csharp
using Unity.Entities;
using Unity.Mathematics;

// ✅ 组件：仅数据，无方法
public struct Position : IComponentData {
    public float3 Value;
}

public struct Velocity : IComponentData {
    public float3 Value;
}
```

---

### 定义系统

```csharp
using Unity.Entities;
using Unity.Burst;

// ✅ 系统：处理实体的逻辑
[BurstCompile]
public partial struct MovementSystem : ISystem {
    [BurstCompile]
    public void OnUpdate(ref SystemState state) {
        float deltaTime = SystemAPI.Time.DeltaTime;

        // 查询所有带 Position + Velocity 的实体
        foreach (var (transform, velocity) in
            SystemAPI.Query<RefRW<Position>, RefRO<Velocity>>()) {

            transform.ValueRW.Value += velocity.ValueRO.Value * deltaTime;
        }
    }
}
```

---

### 创建实体

```csharp
using Unity.Entities;
using Unity.Mathematics;

public partial class EntitySpawner : SystemBase {
    protected override void OnUpdate() {
        var em = EntityManager;

        // 创建实体
        Entity entity = em.CreateEntity();

        // 添加组件
        em.AddComponentData(entity, new Position { Value = float3.zero });
        em.AddComponentData(entity, new Velocity { Value = new float3(1, 0, 0) });
    }
}
```

---

## 混合 ECS（MonoBehaviour + ECS）

### Baker（将 GameObject 转为 Entity）

```csharp
using Unity.Entities;
using UnityEngine;

public class PlayerAuthoring : MonoBehaviour {
    public float speed;
}

public class PlayerBaker : Baker<PlayerAuthoring> {
    public override void Bake(PlayerAuthoring authoring) {
        var entity = GetEntity(TransformUsageFlags.Dynamic);

        AddComponent(entity, new Position { Value = authoring.transform.position });
        AddComponent(entity, new Velocity { Value = new float3(authoring.speed, 0, 0) });
    }
}
```

**工作流程：**
1. 在编辑器中把 `PlayerAuthoring` 挂到 GameObject 上
2. Baker 在运行时自动转换为 Entity
3. Entity 带有 Position + Velocity 组件

---

## 查询（Queries）

### 按组件查询所有实体

```csharp
foreach (var (position, velocity) in
    SystemAPI.Query<RefRW<Position>, RefRO<Velocity>>()) {

    position.ValueRW.Value += velocity.ValueRO.Value * deltaTime;
}
```

---

### 带 Entity 的查询

```csharp
foreach (var (position, velocity, entity) in
    SystemAPI.Query<RefRW<Position>, RefRO<Velocity>>().WithEntityAccess()) {

    // 访问实体 ID
    Debug.Log($"Entity: {entity}");
}
```

---

### 带过滤条件的查询

```csharp
// 仅带 "Enemy" 标签的实体
foreach (var position in
    SystemAPI.Query<RefRW<Position>>().WithAll<EnemyTag>()) {
    // 只处理敌人
}
```

---

## Jobs（并行执行）

### IJobEntity（并行 foreach）

```csharp
using Unity.Entities;
using Unity.Burst;

[BurstCompile]
public partial struct MovementJob : IJobEntity {
    public float DeltaTime;

    // Execute 对每个实体并行执行
    void Execute(ref Position position, in Velocity velocity) {
        position.Value += velocity.Value * DeltaTime;
    }
}

[BurstCompile]
public partial struct MovementSystem : ISystem {
    public void OnUpdate(ref SystemState state) {
        var job = new MovementJob {
            DeltaTime = SystemAPI.Time.DeltaTime
        };
        job.ScheduleParallel(); // 并行执行
    }
}
```

---

## Burst 编译器（性能）

### 启用 Burst

```csharp
using Unity.Burst;

[BurstCompile] // 比普通 C# 快约 10–100 倍
public partial struct MySystem : ISystem {
    [BurstCompile]
    public void OnUpdate(ref SystemState state) {
        // 经 Burst 编译的代码
    }
}
```

**Burst 限制：**
- 不能有托管引用（class、string 等）
- 仅 blittable 类型（struct、基元类型、Unity.Mathematics 类型）
- 不能使用异常

---

## Entity Command Buffers（结构性变更）

### 延迟结构性变更

```csharp
using Unity.Entities;

public partial struct SpawnSystem : ISystem {
    public void OnUpdate(ref SystemState state) {
        var ecb = new EntityCommandBuffer(Allocator.Temp);

        // 延迟创建实体（迭代中不要直接改结构）
        foreach (var spawner in SystemAPI.Query<Spawner>()) {
            Entity newEntity = ecb.CreateEntity();
            ecb.AddComponent(newEntity, new Position { Value = spawner.SpawnPos });
        }

        ecb.Playback(state.EntityManager); // 应用变更
        ecb.Dispose();
    }
}
```

---

## Dynamic Buffers（类数组组件）

### 定义 Dynamic Buffer

```csharp
public struct PathWaypoint : IBufferElementData {
    public float3 Position;
}
```

### 使用 Dynamic Buffer

```csharp
// 向实体添加 buffer
var buffer = EntityManager.AddBuffer<PathWaypoint>(entity);
buffer.Add(new PathWaypoint { Position = new float3(0, 0, 0) });
buffer.Add(new PathWaypoint { Position = new float3(10, 0, 0) });

// 查询 buffer
foreach (var buffer in SystemAPI.Query<DynamicBuffer<PathWaypoint>>()) {
    foreach (var waypoint in buffer) {
        Debug.Log(waypoint.Position);
    }
}
```

---

## Tags（零大小组件）

### 定义 Tag

```csharp
public struct EnemyTag : IComponentData { } // 空组件 = 标签
```

### 用 Tag 做过滤

```csharp
// 只处理带 EnemyTag 的实体
foreach (var position in
    SystemAPI.Query<RefRW<Position>>().WithAll<EnemyTag>()) {
    // 敌人相关逻辑
}
```

---

## 系统排序

### 显式排序

```csharp
[UpdateBefore(typeof(PhysicsSystem))]
public partial struct InputSystem : ISystem { }

[UpdateAfter(typeof(PhysicsSystem))]
public partial struct RenderSystem : ISystem { }
```

---

## 性能模式

### Chunk 迭代（最高性能）

```csharp
public void OnUpdate(ref SystemState state) {
    var query = SystemAPI.QueryBuilder().WithAll<Position, Velocity>().Build();

    var chunks = query.ToArchetypeChunkArray(Allocator.Temp);
    var positionType = state.GetComponentTypeHandle<Position>();
    var velocityType = state.GetComponentTypeHandle<Velocity>(true); // 只读

    foreach (var chunk in chunks) {
        var positions = chunk.GetNativeArray(ref positionType);
        var velocities = chunk.GetNativeArray(ref velocityType);

        for (int i = 0; i < chunk.Count; i++) {
            positions[i] = new Position {
                Value = positions[i].Value + velocities[i].Value * deltaTime
            };
        }
    }

    chunks.Dispose();
}
```

---

## 从 MonoBehaviour 迁移

```csharp
// ❌ 旧：MonoBehaviour（OOP）
public class Enemy : MonoBehaviour {
    public float speed;
    void Update() {
        transform.position += Vector3.forward * speed * Time.deltaTime;
    }
}

// ✅ 新：DOTS（ECS）
public struct EnemyData : IComponentData {
    public float Speed;
}

[BurstCompile]
public partial struct EnemyMovementSystem : ISystem {
    public void OnUpdate(ref SystemState state) {
        float dt = SystemAPI.Time.DeltaTime;
        foreach (var (transform, enemy) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<EnemyData>>()) {
            transform.ValueRW.Position += new float3(0, 0, enemy.ValueRO.Speed * dt);
        }
    }
}
```

---

## 调试

### Entities Hierarchy 窗口

`Window > Entities > Hierarchy`

- 显示所有实体及其组件
- 可按原型、组件类型筛选

### Entities Profiler

`Window > Analysis > Profiler > Entities`

- 各系统执行时间
- 按原型的内存占用

---

## 来源
- https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/index.html
- https://learn.unity.com/tutorial/entity-component-system
