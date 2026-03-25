# Godot — 破坏性变更

上次核对：2026-02-12

各 Godot 版本之间的变更，侧重 LLM 训练截止之后的改动（4.4+）。

## 4.5 → 4.6（2026 年 1 月 — 截止后，高风险）

| 子系统 | 变更 | 说明 |
|-----------|--------|---------|
| 物理 | Jolt 现为默认 3D 物理引擎 | 新项目默认使用 Jolt。已有项目保留原设置。部分 HingeJoint3D 属性（如 `damp`）仅在 GodotPhysics 下生效。 |
| 渲染 | Glow 在 tonemapping 之前处理 | 此前在 tonemapping 之后。带 glow 的场景观感会不同。请在 WorldEnvironment 中调整强度/混合。 |
| 渲染 | Windows 上默认 D3D12 | 此前为 Vulkan。旨在改善驱动兼容性。 |
| 渲染 | AgX tonemapper 新增控件 | 增加白点与对比度参数。 |
| 核心 | Quaternion 默认初始化为单位四元数 | 此前为零四元数。多数代码不受影响，但属技术性破坏性变更。 |
| UI | 双焦点系统 | 鼠标/触摸焦点与键盘/手柄焦点分离。不同输入方式的视觉反馈不同。 |
| 动画 | IK 系统完整恢复 | 通过 SkeletonModifier3D 节点提供 CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK。 |
| 编辑器 | 新「Modern」主题为默认 | 灰阶取代蓝灰色调。恢复方式：Editor Settings → Interface → Theme → Style: Classic |
| 编辑器 | 「Select Mode」快捷键变更 | 新「Select Mode」（v 键）减少误操作变换。原模式更名为「Transform Mode」（q 键）。 |
| 2D | TileMapLayer 场景图块旋转 | 场景图块现可像 atlas 图块一样旋转。 |
| 本地化 | CSV 复数形式支持 | 复数不再依赖 Gettext。新增 context 列。 |
| C# | 自动字符串抽取 | 从 C# 代码自动抽取翻译字符串。 |
| 插件 | 新增 EditorDock 类 | 用于插件 dock 的专用容器，可控制布局。 |

## 4.4 → 4.5（2025 年末 — 截止后，高风险）

| 子系统 | 变更 | 说明 |
|-----------|--------|---------|
| GDScript | 新增可变参数 | 函数可接受 `...` 任意参数 — 新语言特性 |
| GDScript | `@abstract` 装饰器 | 抽象类与方法现可强制约束 |
| GDScript | 脚本回溯 | Release 构建也可获得详细调用栈 |
| 渲染 | Stencil buffer 支持 | 高级视觉效果的新能力 |
| 渲染 | SMAA 1x 抗锯齿 | 新的后处理 AA 选项 |
| 渲染 | Shader Baker | 预编译 shader — 据称部分 demo 启动快约 20 倍 |
| 渲染 | Bent normal maps、specular occlusion | 新的材质特性 |
| 无障碍 | 屏幕阅读器支持 | Control 节点通过 AccessKit 与无障碍工具协作 |
| 编辑器 | 实时翻译预览 | 在编辑器内用不同语言测试 GUI 布局 |
| 物理 | 3D 插值架构调整 | 从 RenderingServer 迁至 SceneTree。API 未变，内部实现不同。 |
| 动画 | BoneConstraint3D | 新增：AimModifier3D、CopyTransformModifier3D、ConvertTransformModifier3D |
| 资源 | 新增 `duplicate_deep()` | 嵌套资源深拷贝的显式方法 |
| 导航 | 专用 2D 导航服务器 | 不再代理到 3D 导航；2D 游戏导出体积更小 |
| UI | FoldableContainer 节点 | 可折叠 UI 区块的手风琴式新容器 |
| UI | Control 递归行为 | 可禁用整棵节点树的鼠标/焦点交互 |
| 平台 | visionOS 导出支持 | 新平台目标 |
| 平台 | SDL3 手柄驱动 | 手柄处理委托给 SDL |
| 平台 | Android 16KB 页支持 | 面向 Android 15+ 的 Google Play 要求 |

## 4.3 → 4.4（2025 年中 — 接近截止，请核实）

| 子系统 | 变更 | 说明 |
|-----------|--------|---------|
| 核心 | `FileAccess.store_*` 返回 `bool` | 此前为 `void`。涉及方法：`store_8`、`store_16`、`store_32`、`store_64`、`store_buffer`、`store_csv_line`、`store_double`、`store_float`、`store_half`、`store_line`、`store_pascal_string`、`store_real`、`store_string`、`store_var` |
| 核心 | `OS.execute_with_pipe` | 新增可选参数 `blocking` |
| 核心 | `RegEx.compile/create_from_string` | 新增可选参数 `show_error` |
| 渲染 | `RenderingDevice.draw_list_begin` | 移除大量参数；新增 `breadcrumb` 参数 |
| 渲染 | Shader 纹理类型 | 参数/返回类型由 `Texture2D` 改为 `Texture` |
| 粒子 | `.restart()` 方法 | 新增可选参数 `keep_seed`（CPU/GPU 2D/3D） |
| GUI | `RichTextLabel.push_meta` | 新增可选参数 `tooltip` |
| GUI | `GraphEdit.connect_node` | 新增可选参数 `keep_alive` |

## 4.2 → 4.3（训练数据内 — 低风险）

| 子系统 | 变更 | 说明 |
|-----------|--------|---------|
| 动画 | `Skeleton3D.add_bone` 返回 `int32` | 此前为 `void` |
| 动画 | `bone_pose_updated` 信号 | 由 `skeleton_updated` 取代 |
| TileMap | `TileMapLayer` 取代 `TileMap` | 每层一个节点，而非单层多图层节点 |
| 导航 | `NavigationRegion2D` | 移除 `avoidance_layers`、`constrain_avoidance` 属性 |
| 编辑器 | `EditorSceneFormatImporterFBX` | 重命名为 `EditorSceneFormatImporterFBX2GLTF` |
| 动画 | AnimationMixer 基类 | AnimationPlayer 与 AnimationTree 现继承 AnimationMixer |
