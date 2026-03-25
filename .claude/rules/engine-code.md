---
paths:
  - "src/core/**"
---

# 引擎代码规则

- 热路径（更新循环、渲染、物理）中零分配 —— 预分配、对象池、复用
- 所有引擎 API 必须线程安全，或明确文档标注为仅单线程使用
- 每次优化前后都要做性能剖析 —— 记录实测数据
- 引擎代码不得依赖玩法代码（依赖方向严格为：引擎 ← 玩法）
- 每个公开 API 须在文档注释中附带用法示例
- 对公开接口的修改需要弃用期与迁移指南
- 对所有资源使用 RAII / 确定性清理
- 所有引擎系统须支持优雅降级
- 编写引擎 API 代码前，查阅 `docs/engine-reference/` 以确认当前引擎版本，并对照参考文档核实 API

## 示例

**正确**（热路径零分配）：

```gdscript
# 预分配数组，每帧复用
var _nearby_cache: Array[Node3D] = []

func _physics_process(delta: float) -> void:
    _nearby_cache.clear()  # 复用，勿每帧重新分配
    _spatial_grid.query_radius(position, radius, _nearby_cache)
```

**错误**（在热路径中分配）：

```gdscript
func _physics_process(delta: float) -> void:
    var nearby: Array[Node3D] = []  # 违规：每帧分配
    nearby = get_tree().get_nodes_in_group("enemies")  # 违规：每帧整树查询
```
