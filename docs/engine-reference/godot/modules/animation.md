# Godot 动画 — 速查

最后核对：2026-02-12 | 引擎：Godot 4.6

## 相对约 4.3 以来的变化（LLM 训练截止）

### 4.6 变更
- **IK 系统全面恢复**：面向 3D 骨架的完整逆运动学
  - CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK
  - 通过 `SkeletonModifier3D` 节点应用（非旧版 IK 做法）
- **动画编辑器体验**：Bezier 节点组可 solo/隐藏/锁定/删除；时间轴可拖拽

### 4.5 变更
- **BoneConstraint3D**：将骨骼与其他骨骼绑定并施加修饰器
  - `AimModifier3D`、`CopyTransformModifier3D`、`ConvertTransformModifier3D`

### 4.3 变更（在训练数据内）
- **AnimationMixer**：`AnimationPlayer` 与 `AnimationTree` 的基类
  - `method_call_mode` → `callback_mode_method`
  - `playback_active` → `active`
  - `bone_pose_updated` 信号 → `skeleton_updated`
- **`Skeleton3D.add_bone()`**：现返回 `int32`（原为 `void`）

## 当前 API 用法

### AnimationPlayer（API 未变，基类更新）
```gdscript
@onready var anim_player: AnimationPlayer = %AnimationPlayer

func play_attack() -> void:
    anim_player.play(&"attack")
    await anim_player.animation_finished
```

### IK 配置（4.6 — 新增）
```gdscript
# 将基于 SkeletonModifier3D 的 IK 节点作为 Skeleton3D 的子节点添加
# 可用类型：
# - SkeletonModifier3D（基类）
# - TwoBoneIK（手臂、腿）
# - FABRIK（链、触手）
# - CCDIK（尾巴、脊柱）
# - Jacobian IK（复杂多关节）
# - Spline IK（沿曲线）

# 在编辑器或代码中配置：
# 1. 将 IK 修饰器节点添加为 Skeleton3D 的子节点
# 2. 设置目标骨骼与末端骨骼
# 3. 添加 Marker3D 作为 IK 目标
# 4. IK 求解器每帧自动运行
```

### BoneConstraint3D（4.5 — 新增）
```gdscript
# 作为 Skeleton3D 的子节点添加
# 类型：
# - AimModifier3D：骨骼指向目标
# - CopyTransformModifier3D：复制另一根骨骼的变换
# - ConvertTransformModifier3D：重映射变换数值
```

### AnimationTree（4.3 起基类变更）
```gdscript
# AnimationTree 现继承 AnimationMixer（不再直接继承 Node）
# 使用 AnimationMixer 属性：
@onready var anim_tree: AnimationTree = %AnimationTree

func _ready() -> void:
    anim_tree.active = true  # 勿用 playback_active（4.3 起已弃用）
```

## 常见错误
- 使用 `playback_active` 而非 `active`（4.3 起已弃用）
- 使用 `bone_pose_updated` 信号而非 `skeleton_updated`（4.3 重命名）
- 仍用旧 IK 方案而非 SkeletonModifier3D 体系（4.6 已恢复新体系）
- 对动画节点做类型检查时未考虑 `is AnimationMixer`
