# Godot 引擎 — 版本参考

| 字段 | 值 |
|-------|-------|
| **引擎版本** | Godot 4.6 |
| **发布日期** | 2026 年 1 月 |
| **项目固定日期** | 2026-02-12 |
| **文档最后核验** | 2026-02-12 |
| **LLM 知识截止** | 2025 年 5 月 |

## 知识缺口提示

LLM 的训练数据大致覆盖到 Godot ~4.3。4.4、4.5
与 4.6 引入了重大变更，模型并未掌握。
在建议调用 Godot API 之前，务必对照本目录交叉核对。

## 知识截止之后的版本时间线

| 版本 | 发布 | 风险级别 | 主题要点 |
|---------|---------|------------|-----------|
| 4.4 | ~2025 年中 | 中 | Jolt 物理可选、FileAccess 返回类型、shader 纹理类型变更 |
| 4.5 | ~2025 年末 | 高 | 无障碍（AccessKit）、可变参数、@abstract、shader baker、SMAA |
| 4.6 | 2026 年 1 月 | 高 | Jolt 默认、glow 重构、Windows 上 D3D12 默认、IK 恢复 |

## 已核验来源

- 官方文档：https://docs.godotengine.org/en/stable/
- 4.5→4.6 迁移：https://docs.godotengine.org/en/stable/tutorials/migrating/upgrading_to_godot_4.6.html
- 4.4→4.5 迁移：https://docs.godotengine.org/en/stable/tutorials/migrating/upgrading_to_godot_4.5.html
- 变更日志：https://github.com/godotengine/godot/blob/master/CHANGELOG.md
- 发行说明：https://godotengine.org/releases/4.6/
