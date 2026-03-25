# Godot 渲染 — 速查

最后核对：2026-02-12 | 引擎：Godot 4.6

## 相对约 4.3 以来的变化（LLM 训练数据截止点）

### 4.6 变更
- **Windows 上默认渲染后端为 D3D12**（此前为 Vulkan）
- **Glow 在色调映射之前处理**（此前在之后）— 使用 screen 混合模式
- **AgX 色调映射器**：新增白点与对比度控制
- **SSR 大幅重做**：真实感、画面稳定性与性能均有提升

### 4.5 变更
- **Shader Baker**：预编译着色器以缩短启动时间
- **SMAA 1x**：新的抗锯齿选项（比 FXAA 更清晰，比 TAA 更省）
- **Stencil buffer 支持**：可实现选择性几何遮罩、传送门类效果
- **Bent normal maps**：在法线贴图纹理中编码的方向性遮挡
- **Specular occlusion**：环境光遮挡现在能正确影响反射

### 4.4 变更
- **`RenderingDevice.draw_list_begin`**：大量参数已移除；新增可选 `breadcrumb`
- **着色器纹理类型**：由以 `Texture2D` 为基改为以 `Texture` 为基
- **粒子 `.restart()`**：新增可选参数 `keep_seed`

### 4.3 变更（在训练数据范围内）
- **Compositor 节点**：`Compositor` + `CompositorEffect` 用于后处理链

## 当前 API 用法

### 后处理（4.3+）
```gdscript
# 使用 Compositor 节点 — 不要用手动 viewport 着色器链
# 将 Compositor 作为 WorldEnvironment 或 Camera3D 的子节点添加
# 为每个后处理步骤创建 CompositorEffect 资源
```

### 抗锯齿选项（4.6）
```
Project Settings → Rendering → Anti Aliasing:
- MSAA 2D/3D：硬件 MSAA（质量好但开销大）
- Screen Space AA：FXAA（快、偏糊）或 SMAA（清晰、成本适中）  # SMAA 自 4.5 起新增
- TAA：时间性抗锯齿（质量最好，快速运动时易拖影）
```

### 渲染后端选择（4.6）
```
Project Settings → Rendering → Renderer:
- Forward+（默认）：功能完整，面向桌面
- Mobile：面向移动/低端设备优化，功能受限
- Compatibility：OpenGL 3.3 / WebGL 2，硬件支持面最广

Windows 默认后端：D3D12（4.6 之前为 Vulkan）
```

## 常见误区
- 以为 Windows 上默认后端仍是 Vulkan（自 4.6 起为 D3D12）
- 用后处理时用手动 viewport 链，而不用 Compositor
- 在着色器 uniform 类型里仍写 `Texture2D`（自 4.4 起应使用 `Texture`）
- 着色器变体很多的项目未使用 Shader Baker
