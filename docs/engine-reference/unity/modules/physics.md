# Unity 6.3 — 物理模块参考

**最后核对：** 2026-02-13  
**知识缺口：** Unity 6 物理改进、求解器变更

---

## 概述

Unity 6.3 使用 **PhysX 5.1**（相较 2022 LTS 中的 PhysX 4.x 有所升级）：
- 求解器稳定性更好
- 性能提升
- 碰撞检测增强

---

## 相对 2022 LTS 的主要变化

### 默认求解器迭代次数提高

Unity 6 为提高稳定性，提高了默认求解器迭代次数：

```csharp
// 默认值由 6 次改为 8 次
Physics.defaultSolverIterations = 8; // 若依赖旧行为请核对
```

### 碰撞检测增强

```csharp
// ✅ Unity 6：连续碰撞检测（CCD）改进
rigidbody.collisionDetectionMode = CollisionDetectionMode.ContinuousDynamic;
// 对高速运动物体处理更好
```

---

## 核心物理组件

### Rigidbody

```csharp
// ✅ 建议：用 AddForce，不要直接改 velocity
Rigidbody rb = GetComponent<Rigidbody>();
rb.AddForce(Vector3.forward * 10f, ForceMode.Impulse);

// ❌ 避免：直接赋值 velocity（可能导致不稳定）
rb.velocity = new Vector3(0, 10, 0); // 仅在必要时使用
```

### Colliders

```csharp
// 基础碰撞体：Box、Sphere、Capsule（开销最低）
// Mesh 碰撞体：开销大，仅用于静态几何

// ✅ 组合碰撞体（多个基础体）优于单个 mesh 碰撞体
```

---

## 射线检测

### 高效射线检测（避免分配）

```csharp
// ✅ 无分配射线检测
if (Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance)) {
    Debug.Log($"Hit: {hit.collider.name}");
}

// ✅ 多次命中（无分配）
RaycastHit[] results = new RaycastHit[10];
int hitCount = Physics.RaycastNonAlloc(origin, direction, results, maxDistance);
for (int i = 0; i < hitCount; i++) {
    Debug.Log($"Hit {i}: {results[i].collider.name}");
}

// ❌ 避免：RaycastAll（每次调用都分配数组）
RaycastHit[] hits = Physics.RaycastAll(origin, direction); // GC 分配！
```

### 用 LayerMask 做选择性射线检测

```csharp
// ✅ 使用 LayerMask 过滤碰撞
int layerMask = 1 << LayerMask.NameToLayer("Enemy");
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask);
```

---

## 物理查询

### OverlapSphere（检测附近物体）

```csharp
// ✅ 无分配版本
Collider[] results = new Collider[10];
int count = Physics.OverlapSphereNonAlloc(center, radius, results);
for (int i = 0; i < count; i++) {
    // 处理 results[i]
}
```

### SphereCast（粗射线）

```csharp
// 常用于角色控制器
if (Physics.SphereCast(origin, radius, direction, out RaycastHit hit, maxDistance)) {
    // 用球形射线命中物体
}
```

---

## 碰撞事件

### OnCollisionEnter / Stay / Exit

```csharp
void OnCollisionEnter(Collision collision) {
    // 碰撞开始时触发
    Debug.Log($"Collided with {collision.gameObject.name}");

    // 访问接触点
    foreach (ContactPoint contact in collision.contacts) {
        Debug.DrawRay(contact.point, contact.normal, Color.red, 2f);
    }
}
```

### OnTriggerEnter / Stay / Exit

```csharp
void OnTriggerEnter(Collider other) {
    // 触发器碰撞体（Is Trigger = true）
    if (other.CompareTag("Pickup")) {
        Destroy(other.gameObject);
    }
}
```

---

## 角色控制器

### CharacterController 组件

```csharp
CharacterController controller = GetComponent<CharacterController>();

// ✅ 带碰撞检测的移动
Vector3 move = transform.forward * speed * Time.deltaTime;
controller.Move(move);

// 重力需手动施加
if (!controller.isGrounded) {
    velocity.y += Physics.gravity.y * Time.deltaTime;
}
controller.Move(velocity * Time.deltaTime);
```

---

## 物理材质

### 摩擦与弹性

```csharp
// 创建：Assets > Create > Physic Material
// 赋给碰撞体：Collider > Material

// PhysicMaterial 设置：
// - Dynamic Friction: 0.6（滑动摩擦）
// - Static Friction: 0.6（静摩擦）
// - Bounciness: 0.0 - 1.0
// - Friction Combine: Average, Minimum, Maximum, Multiply
// - Bounce Combine: Average, Minimum, Maximum, Multiply
```

---

## 关节

### Fixed Joint（连接两个刚体）

```csharp
FixedJoint joint = gameObject.AddComponent<FixedJoint>();
joint.connectedBody = otherRigidbody;
```

### Hinge Joint（门、车轮）

```csharp
HingeJoint hinge = gameObject.AddComponent<HingeJoint>();
hinge.axis = Vector3.up; // 旋转轴
hinge.useLimits = true;
hinge.limits = new JointLimits { min = -90, max = 90 };
```

---

## 性能优化

### 物理层碰撞矩阵
`Edit > Project Settings > Physics > Layer Collision Matrix`
- 关闭层之间不必要的碰撞检测
- 可带来显著性能收益

### Fixed Timestep
`Edit > Project Settings > Time > Fixed Timestep`
- 默认：0.02（物理 50 FPS）
- 数值越小越精确，CPU 开销越高
- 尽量与游戏目标帧率匹配

### 简化碰撞几何
- 优先用基础碰撞体（box、sphere、capsule），少用 mesh 碰撞体
- mesh 碰撞体在构建时烘焙，而非运行时生成

---

## 常见模式

### 地面检测（角色控制器）

```csharp
bool IsGrounded() {
    float rayLength = 0.1f;
    return Physics.Raycast(transform.position, Vector3.down, rayLength);
}
```

### 施加爆炸力

```csharp
void ApplyExplosion(Vector3 explosionPos, float radius, float force) {
    Collider[] colliders = Physics.OverlapSphere(explosionPos, radius);
    foreach (Collider hit in colliders) {
        Rigidbody rb = hit.GetComponent<Rigidbody>();
        if (rb != null) {
            rb.AddExplosionForce(force, explosionPos, radius);
        }
    }
}
```

---

## 调试

### Physics Debugger（Unity 6+）
- `Window > Analysis > Physics Debugger`
- 可视化碰撞体、接触、查询

### Gizmos

```csharp
void OnDrawGizmos() {
    Gizmos.color = Color.red;
    Gizmos.DrawWireSphere(transform.position, detectionRadius);
}
```

---

## 来源
- https://docs.unity3d.com/6000.0/Documentation/Manual/PhysicsOverview.html
- https://docs.unity3d.com/ScriptReference/Physics.html
