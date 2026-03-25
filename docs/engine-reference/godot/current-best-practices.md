# Godot — 当前最佳实践

最后验证：2026-02-12 | 引擎：Godot 4.6

自模型训练数据（约 4.3）以来**新增或变更**的实践。
本文补充（而非替代）Agent 内置知识。

## GDScript（4.5+）

- **可变参数**：函数可接受任意数量的参数
  ```gdscript
  func log_values(prefix: String, values: Variant...) -> void:
      for v in values:
          print(prefix, ": ", v)
  ```

- **抽象类与方法**：使用 `@abstract` 强制子类继承并实现
  ```gdscript
  @abstract
  class_name BaseEnemy extends CharacterBody3D

  @abstract
  func get_attack_pattern() -> Array[Attack]:
      pass  # 子类必须重写
  ```

- **脚本回溯**：即使在 Release 构建中也可获得详细调用栈

## 物理（4.6）

- **Jolt Physics 为新项目的默认 3D 物理引擎**
  - 相较 GodotPhysics3D，确定性与稳定性更好
  - 部分 HingeJoint3D 属性（`damp`）仅在 GodotPhysics 下生效
  - 切换方式：项目设置 → Physics → 3D → Physics Engine
  - 2D 物理未变（仍为 Godot Physics 2D）

## 渲染（4.6）

- **Windows 上默认图形后端为 D3D12**（原为 Vulkan）——以改善驱动兼容性
- **Glow 现于色调映射之前处理**，且为屏幕混合模式——既有 Glow 配置观感可能不同
- **SSR 大幅重做**——真实感、稳定性与性能均有明显提升
- **AgX 色调映射器**——新增白点与对比度控制

## 渲染（4.5）

- **Shader Baker**：预编译着色器，消除启动卡顿
- **SMAA 1x**：新抗锯齿选项——比 FXAA 更锐利，比 TAA 更省
- **模板缓冲（Stencil buffer）**：可用于高级遮罩 / 传送门类效果
- **弯曲法线贴图（Bent normal maps）**：法线贴图纹理中的方向性遮蔽
- **镜面遮蔽（Specular occlusion）**：环境光遮蔽现可影响反射

## 无障碍（4.5+）

- **屏幕阅读器支持**：Control 节点通过 AccessKit 与无障碍工具集成
- **实时翻译预览**：在编辑器内直接以不同语言测试 GUI 布局
- **FoldableContainer**：新的手风琴式 UI 节点，用于可折叠区块
- **递归禁用 Control**：单一属性即可禁用整棵节点树的鼠标 / 焦点交互

## 动画（4.5+）

- **BoneConstraint3D**：用修饰器将骨骼绑定到其他骨骼
  - AimModifier3D、CopyTransformModifier3D、ConvertTransformModifier3D

## 动画（4.6）

- **IK 系统全面恢复**：3D 完整逆运动学重新引入
  - 可用修饰器：CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK
  - 通过 `SkeletonModifier3D` 节点应用

## 资源（4.5+）

- **`duplicate_deep()`**：对嵌套资源树进行显式深拷贝
  - 保留旧版 `duplicate()` 行为以兼容
  - 需要嵌套资源的每实例副本时使用 `duplicate_deep()`

## 导航（4.5+）

- **专用 2D 导航服务器**：不再经由 3D NavigationServer 代理
  - 减小纯 2D 游戏的导出体积

## UI（4.6）

- **双焦点系统**：鼠标 / 触控焦点与键盘 / 手柄焦点现已分离
  - 视觉反馈随输入方式不同
  - 设计自定义焦点行为时需考虑这一点

## 编辑器工作流（4.6）

- 可拖拽停靠栏并显示蓝色轮廓预览（含底部面板）
- 多数面板支持浮动窗口（Debugger 除外）
- 新快捷键：Alt+O（输出）、Alt+S（着色器）
- 导出变量自动生成：从文件系统拖资源到脚本编辑器
- 启用「实时预览」时，快速打开对话框内可实时预览
- 新增「选择模式」（v 键）避免误操作变换；原模式更名为「变换模式」（q 键）

## 平台（4.5+）

- **visionOS 导出**：开源后首个新平台（窗口化应用模式）
- **SDL3 手柄驱动**：跨平台手柄支持更好
- **Android**：边到边显示、摄像头画面访问、16KB 页面支持（Android 15+）
- **Linux**：Wayland 子窗口支持，实现多窗口能力
