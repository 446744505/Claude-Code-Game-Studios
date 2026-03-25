# Godot 输入 — 速查

上次核对：2026-02-12 | 引擎：Godot 4.6

## 相对约 4.3（LLM 训练截止）以来的变化

### 4.6 变更
- **双焦点系统**：鼠标/触摸焦点与键盘/手柄焦点现已分离
  - 视觉反馈随输入方式不同而不同
  - 自定义焦点实现可能需要更新
- **Select Mode 快捷键变更**：「Select Mode」现为 `v` 键；原模式更名为「Transform Mode」（`q` 键）

### 4.5 变更
- **SDL3 手柄驱动**：手柄处理交由 SDL 库，以改善跨平台支持
- **递归 Control 禁用**：单一属性即可禁用整棵节点树的鼠标/焦点

### 4.3 变更（在训练数据内）
- **InputEventShortcut**：菜单快捷键的专用事件类型（可选）

## 当前 API 用法

### 输入动作（未变）
```gdscript
func _physics_process(delta: float) -> void:
    var input_dir: Vector2 = Input.get_vector(
        &"move_left", &"move_right", &"move_forward", &"move_back"
    )
    if Input.is_action_just_pressed(&"jump"):
        jump()
```

### 输入事件（未变）
```gdscript
func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            handle_click(event.position)
    elif event is InputEventKey:
        if event.keycode == KEY_ESCAPE and event.pressed:
            toggle_pause()
```

### 焦点管理（4.6 — 已变更）
```gdscript
# 鼠标/触摸与键盘/手柄焦点现已分离
# 视觉样式可能随当前输入方式不同而不同
# 若有自定义焦点绘制，请用两种输入方式都测一遍

# 常规写法仍然可用：
func _ready() -> void:
    %StartButton.grab_focus()  # 键盘/手柄焦点

# 注意：在 4.6 中，鼠标悬停焦点 ≠ 键盘焦点
```

### 手柄（4.5+ — SDL3 后端）
```gdscript
# API 未变，但 SDL3 带来：
# - 跨平台设备检测更好
# - 震动支持改进
# - 按键映射更一致

func _input(event: InputEvent) -> void:
    if event is InputEventJoypadButton:
        if event.button_index == JOY_BUTTON_A and event.pressed:
            confirm_selection()
```

## 常见误区
- 未同时测试鼠标与键盘两条焦点路径（4.6 的双焦点）
- 误以为 `grab_focus()` 会影响鼠标焦点（在 4.6 中它只影响键盘/手柄）
- 在热点路径里对动作名使用字符串字面量，而非 `StringName`（`&"action"`）
