# Godot 导航 — 速查

上次核对：2026-02-12 | 引擎：Godot 4.6

## 相对约 4.3 版（LLM 训练截止）以来的变化

### 4.5 变更
- **独立 2D 导航服务器**：不再作为 3D NavigationServer 的代理
  - 纯 2D 游戏可减小导出包体
  - 2D 与 3D 的 API 保持一致

### 4.3 变更（训练数据中已有）
- **`NavigationRegion2D`**：已移除 `avoidance_layers` 与 `constrain_avoidance` 属性

## 当前 API 用法

### NavigationAgent3D（多数情况首选）
```gdscript
@onready var nav_agent: NavigationAgent3D = %NavigationAgent3D

func _ready() -> void:
    nav_agent.path_desired_distance = 0.5
    nav_agent.target_desired_distance = 1.0
    nav_agent.velocity_computed.connect(_on_velocity_computed)

func navigate_to(target: Vector3) -> void:
    nav_agent.target_position = target

func _physics_process(delta: float) -> void:
    if nav_agent.is_navigation_finished():
        return
    var next_pos: Vector3 = nav_agent.get_next_path_position()
    var direction: Vector3 = global_position.direction_to(next_pos)
    nav_agent.velocity = direction * move_speed

func _on_velocity_computed(safe_velocity: Vector3) -> void:
    velocity = safe_velocity
    move_and_slide()
```

### NavigationAgent2D
```gdscript
@onready var nav_agent: NavigationAgent2D = %NavigationAgent2D

func navigate_to(target: Vector2) -> void:
    nav_agent.target_position = target

func _physics_process(delta: float) -> void:
    if nav_agent.is_navigation_finished():
        return
    var next_pos: Vector2 = nav_agent.get_next_path_position()
    var direction: Vector2 = global_position.direction_to(next_pos)
    velocity = direction * move_speed
    move_and_slide()
```

### 底层路径查询（3D）
```gdscript
# 直接向服务器查询，用于自定义寻路逻辑
var query := NavigationPathQueryParameters3D.new()
query.map = get_world_3d().navigation_map
query.start_position = global_position
query.target_position = target_pos
query.navigation_layers = navigation_layers

var result := NavigationPathQueryResult3D.new()
NavigationServer3D.query_path(query, result)
var path: PackedVector3Array = result.path
```

### 避让
```gdscript
# 启用基于 RVO2 的局部避让
nav_agent.avoidance_enabled = true
nav_agent.radius = 0.5
nav_agent.max_speed = move_speed
nav_agent.neighbor_distance = 10.0

# 通过 velocity_computed 信号使用避让安全移动
nav_agent.velocity_computed.connect(_on_velocity_computed)

# 每帧设置 velocity（避让需要）
nav_agent.velocity = desired_velocity
```

### 导航层
```gdscript
# 用层按单位类型划分可行走区域
# 第 1 层：地面单位
# 第 2 层：飞行单位
# 第 3 层：游泳单位
nav_agent.navigation_layers = 1  # 仅地面
nav_agent.navigation_layers = 1 | 2  # 地面 + 飞行
```

## 常见错误
- 未先检查 `is_navigation_finished()` 就调用 `get_next_path_position()`
- 开启避让后未在 agent 上设置 `velocity`（RVO2 需要）
- 使用已移除的 `NavigationRegion2D.avoidance_layers`（4.3 已删）
- 修改几何后忘记烘焙导航网格
- 未设置 `navigation_layers`（默认为所有层）
