# Unity 6.3 — 动画模块参考

**最后核对：** 2026-02-13  
**知识缺口：** Unity 6 动画改进、Timeline 增强

---

## 概述

Unity 6.3 动画体系：
- **Animator Controller（Mecanim）**：基于状态机（**推荐**）
- **Timeline**：过场、镜头序列
- **Animation Rigging**：运行时程序化动画
- **Legacy Animation**：已弃用，避免使用

---

## 相对 2022 LTS 的主要变化

### Animation Rigging 包（Unity 6 中已可用于生产）

```csharp
// 安装：包管理器（Package Manager）> Animation Rigging
// 运行时 IK、瞄准约束、程序化动画
```

### Timeline 改进
- 性能更好
- 轨道类型更多
- 信号（Signal）系统改进

---

## Animator Controller（Mecanim）

### 基本设置

```csharp
// 创建：资源 > 创建 > Animator Controller
// 挂到 GameObject：添加组件 > Animator
// 指定控制器：Animator > Controller = 你的 AnimatorController
```

### 状态过渡

```csharp
Animator animator = GetComponent<Animator>();

// ✅ 触发过渡
animator.SetTrigger("Jump");

// ✅ 布尔参数
animator.SetBool("IsRunning", true);

// ✅ 浮点参数（混合树）
animator.SetFloat("Speed", currentSpeed);

// ✅ 整型参数
animator.SetInteger("WeaponType", 2);
```

### 动画层
- **Base Layer**：默认动画（位移等）
- **Override Layer**：覆盖基础层（例如换武器）
- **Additive Layer**：叠加在基础层之上（例如呼吸、瞄准偏移）

```csharp
// 设置层权重（0–1）
animator.SetLayerWeight(1, 0.5f); // 50% 混合
```

---

## 混合树（Blend Trees）

### 一维混合树（按速度混合）

```csharp
// 待机（Speed = 0）→ 行走（Speed = 0.5）→ 奔跑（Speed = 1.0）
animator.SetFloat("Speed", moveSpeed);
```

### 二维混合树（八向/方向移动）

```csharp
// X 轴：横移（-1 到 1）
// Y 轴：前后（-1 到 1）
animator.SetFloat("MoveX", input.x);
animator.SetFloat("MoveY", input.y);
```

---

## 动画事件（Animation Events）

### 从动画片段触发事件

```csharp
// 在 Animation 窗口：时间轴右键 > Add Animation Event
// GameObject 上需有同名方法：

public void OnFootstep() {
    // 播放脚步音效
    AudioSource.PlayClipAtPoint(footstepClip, transform.position);
}

public void OnAttackHit() {
    // 造成伤害
    DealDamageInFrontOfPlayer();
}
```

---

## 根运动（Root Motion）

### 由动画驱动角色位移

```csharp
Animator animator = GetComponent<Animator>();
animator.applyRootMotion = true; // 根据动画移动角色

void OnAnimatorMove() {
    // 自定义根运动处理
    transform.position += animator.deltaPosition;
    transform.rotation *= animator.deltaRotation;
}
```

---

## Animation Rigging（Unity 6+）

### IK（逆向运动学）

```csharp
// 安装：包管理器 > Animation Rigging
// 添加：Rig Builder 组件 + Rig 子物体

// Two Bone IK（手臂/腿部）
// - 添加 Two Bone IK Constraint
// - 指定 Tip（手/脚）、Mid（肘/膝）、Root（肩/髋）
// - 设置 Target（手/脚应到达的位置）

// 运行时控制：
TwoBoneIKConstraint ikConstraint = rig.GetComponentInChildren<TwoBoneIKConstraint>();
ikConstraint.data.target = targetTransform;
ikConstraint.weight = 1f; // 0–1 混合
```

### Aim 约束（注视目标）

```csharp
// 角色看向目标
MultiAimConstraint aimConstraint = rig.GetComponentInChildren<MultiAimConstraint>();
aimConstraint.data.sourceObjects[0] = new WeightedTransform(targetTransform, 1f);
```

---

## Timeline（过场）

### 基本 Timeline 设置

```csharp
// 创建：资源 > 创建 > Timeline
// 挂到 GameObject：添加组件 > Playable Director
// 指定 Timeline：Playable Director > Playable = 你的 Timeline

// 脚本中播放：
PlayableDirector director = GetComponent<PlayableDirector>();
director.Play();
```

### Timeline 轨道
- **Activation Track**：启用/禁用 GameObject
- **Animation Track**：在 Animator 上播放动画
- **Audio Track**：同步播放音频
- **Cinemachine Track**：镜头运动
- **Signal Track**：在指定时刻触发事件

### 信号系统（事件）

```csharp
// 创建 Signal 资源：资源 > 创建 > Signals > Signal
// 在 Timeline 轨道上添加 Signal Emitter
// 在 GameObject 上添加 Signal Receiver 组件

public class CutsceneEvents : MonoBehaviour {
    public void OnDialogueStart() {
        // 由信号触发
    }
}
```

---

## 动画播放控制

### 直接播放动画（不经状态机）

```csharp
// ✅ CrossFade（平滑过渡）
animator.CrossFade("Attack", 0.2f); // 0.2 秒过渡

// ✅ Play（立即切换）
animator.Play("Idle");

// ❌ 避免：旧版 Animation 组件
Animation anim = GetComponent<Animation>(); // 已弃用
```

---

## 动画曲线（Animation Curves）

### 自定义属性动画

```csharp
// 在 Animation 窗口：Add Property > 自定义组件 > 你的脚本 > 你的 float 字段

public class WeaponTrail : MonoBehaviour {
    public float trailIntensity; // 由片段中的曲线驱动

    void Update() {
        // 强度由动画曲线控制
        trailRenderer.startWidth = trailIntensity;
    }
}
```

---

## 性能优化

### 剔除（Culling）
- `Animator > Culling Mode`：
  - **Always Animate**：始终更新（开销大）
  - **Cull Update Transforms**：离屏时不再更新骨骼（**推荐**）
  - **Cull Completely**：离屏时完全停止动画

### LOD（细节层次）
- 远处角色使用更简动画
- LOD 网格减少骨骼数量

---

## 常见用法

### 判断动画是否播完

```csharp
AnimatorStateInfo stateInfo = animator.GetCurrentAnimatorStateInfo(0);
if (stateInfo.IsName("Attack") && stateInfo.normalizedTime >= 1.0f) {
    // 攻击动画已结束
}
```

### 覆盖动画播放速度

```csharp
animator.speed = 1.5f; // 150% 速度
```

### 获取当前动画片段名

```csharp
AnimatorClipInfo[] clipInfo = animator.GetCurrentAnimatorClipInfo(0);
string currentClip = clipInfo[0].clip.name;
```

---

## 调试

### Animator 窗口
- `窗口 > 动画 > Animator`
- 可视化状态机、查看当前状态

### Animation 窗口
- `窗口 > 动画 > Animation`
- 编辑动画片段、添加事件

---

## 来源
- https://docs.unity3d.com/6000.0/Documentation/Manual/AnimationOverview.html
- https://docs.unity3d.com/Packages/com.unity.animation.rigging@1.3/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.timeline@1.8/manual/index.html
