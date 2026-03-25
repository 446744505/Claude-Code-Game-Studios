---
paths:
  - "assets/shaders/**"
---

# 着色器代码规范

`assets/shaders/` 下的所有着色器文件须遵循本规范，以维持画面质量、性能与跨平台兼容性。

## 命名约定
- 文件命名：`[类型]_[类别]_[名称].[扩展名]`
  - `spatial_env_water.gdshader`（Godot）
  - `SG_Env_Water`（Unity Shader Graph）
  - `M_Env_Water`（Unreal 材质）
- 使用能说明材质用途的清晰名称
- 以着色器类型为前缀：`spatial_`、`canvas_`、`particles_`、`post_`

## 代码质量
- 所有 uniform/参数须有清晰命名与合适的 hint（提示）
- 将相关参数分组（Godot：`group_uniforms`，Unity：`[Header]`，Unreal：Category）
- 对不直观的计算加注释（尤其是数学密集段落）
- 禁止魔法数字——使用具名常量或文档化的 uniform 值
- 每个着色器文件顶部须包含作者与用途说明注释

## 性能要求
- 为每个着色器注明目标平台与复杂度预算
- 选用合适精度：在移动端若无需全精度，使用 `half` / `mediump`
- 尽量减少片元着色器中的纹理采样次数
- 避免在片元着色器中使用动态分支——优先用 `step()`、`mix()`、`smoothstep()`
- 禁止在循环内读取纹理
- 模糊等效果采用双 pass（先横后竖）

## 跨平台
- 在最低规格目标硬件上测试着色器
- 为较低画质档位提供降级/简化版本
- 文档中注明着色器面向的渲染管线（Forward/Deferred、URP/HDRP、Forward+/Mobile/Compatibility 等）
- 同一目录内不要混放不同渲染管线的着色器

## 变体管理
- 尽量减少 shader variant——每个变体都会单独编译成一份着色器
- 文档化所有 keyword/变体及其用途
- 在可行处使用 feature stripping 以减小构建体积
- 记录并监控每个着色器的总变体数量
