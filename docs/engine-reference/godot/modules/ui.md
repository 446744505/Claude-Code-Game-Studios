# Godot UI — 速查

上次核对：2026-02-12 | 引擎：Godot 4.6

## 相对约 4.3（LLM 训练截止）以来的变化

### 4.6 变更
- **双焦点系统**：鼠标/触摸焦点现与键盘/手柄焦点**分离**
  - 视觉反馈随输入方式不同
  - 自定义焦点实现可能需要更新
- **TabContainer**：可在 Inspector 中直接编辑标签属性
- **TileMapLayer 场景图块旋转**：场景图块可像图集图块一样旋转

### 4.5 变更
- **FoldableContainer**：新的手风琴式 UI 节点，用于可折叠区块
- **Recursive Control 行为**：用单个属性即可为整棵节点树禁用鼠标/焦点
- **屏幕阅读器支持**：Control 节点可与 AccessKit 配合工作
- **实时翻译预览**：在编辑器内测试不同语言区域
- **`RichTextLabel.push_meta`**：增加可选 `tooltip` 参数（自 4.4 起）

### 4.4 变更
- **`GraphEdit.connect_node`**：增加可选 `keep_alive` 参数

## 当前 API 用法

### 主题与样式（4.6）
```gdscript
# 编辑器默认使用新的「Modern」主题
# 游戏 UI 仍像以前一样使用自定义主题：
var theme := Theme.new()
theme.set_color(&"font_color", &"Label", Color.WHITE)
theme.set_font_size(&"font_size", &"Label", 24)
```

### 焦点管理（4.6 — 有变更）
```gdscript
# 键盘/手柄焦点（grab_focus 仍可用）
func _ready() -> void:
    %StartButton.grab_focus()

# 重要：在 4.6 中，鼠标悬停与键盘焦点分离
# 二者可同时作用于不同控件
# 请同时用鼠标与键盘/手柄测试你的 UI

# 焦点邻居（未变）
%Button1.focus_neighbor_bottom = %Button2.get_path()
%Button1.focus_neighbor_right = %Button3.get_path()
```

### FoldableContainer（4.5 — 新增）
```gdscript
# 手风琴式可折叠容器
# 作为要折叠内容的父节点添加
# 点击标题时子节点显示/隐藏
# 通过编辑器属性或代码配置
```

### 递归禁用（4.5 — 新增）
```gdscript
# 为整棵层级禁用所有鼠标/焦点交互
# 适合整块禁用菜单等区域
%SettingsPanel.mouse_filter = Control.MOUSE_FILTER_IGNORE
# 在 4.5+ 可递归传播到子节点
```

### 本地化就绪的 UI（最佳实践）
```gdscript
# 所有可见字符串使用 tr()
label.text = tr("MENU_START_GAME")

# 标签使用自动换行（不同语言文本长度不同）
label.autowrap_mode = TextServer.AUTOWRAP_WORD_SMART

# 在编辑器中用实时翻译预览测试（4.5+）
```

## 常见误区
- 以为 `grab_focus()` 会影响鼠标焦点（在 4.6 中仅键盘/手柄）
- 升级到 4.6 后未同时用鼠标与手柄测试 UI
- 硬编码字符串而不用 `tr()` 做本地化
- 可折叠 UI 不用 `FoldableContainer`（4.5 新增，比自定义实现更干净）
