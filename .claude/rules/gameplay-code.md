---
paths:
  - "src/gameplay/**"
---

# 玩法代码规则

- 所有玩法数值必须来自外部配置或数据文件，禁止硬编码
- 所有与时间相关的计算必须使用增量时间（delta），保证与帧率无关
- 禁止直接引用 UI 代码——跨系统通信应使用事件或信号
- 每个玩法系统必须实现清晰、明确的接口
- 状态机必须有显式转移表，并对各状态作文档说明
- 为所有玩法逻辑编写单元测试——将逻辑与表现分离
- 在代码注释中注明每项功能所对应的设计文档
- 不要用静态单例承载游戏状态——应使用依赖注入

## 示例

**正确**（数据驱动）：

```gdscript
var damage: float = config.get_value("combat", "base_damage", 10.0)
var speed: float = stats_resource.movement_speed * delta
```

**错误**（硬编码）：

```gdscript
var damage: float = 25.0   # 违反：玩法数值硬编码
var speed: float = 5.0      # 违反：未来自配置，且未使用 delta
```
