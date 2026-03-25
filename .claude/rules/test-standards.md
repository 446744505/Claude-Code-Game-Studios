---
paths:
  - "tests/**"
---

# 测试规范

- 测试命名：采用 `test_[系统]_[场景]_[预期结果]` 模式
- 每个测试须具备清晰的 arrange（准备）/ act（执行）/ assert（断言）结构
- 单元测试不得依赖外部状态（文件系统、网络、数据库）
- 集成测试须在结束后自行清理环境
- 性能测试须写明可接受阈值，超出则判定失败
- 测试数据须在测试内或专用 fixture 中定义，禁止共享可变状态
- 对外部依赖使用 mock —— 测试应快速且可重复
- 每个缺陷修复须配有回归测试，且该测试应能捕获原始缺陷

## 示例

**正确**（命名规范 + Arrange/Act/Assert）：

```gdscript
func test_health_system_take_damage_reduces_health() -> void:
    # 准备
    var health := HealthComponent.new()
    health.max_health = 100
    health.current_health = 100

    # 执行
    health.take_damage(25)

    # 断言
    assert_eq(health.current_health, 75)
```

**错误**：

```gdscript
func test1() -> void:  # 违规：无描述性名称
    var h := HealthComponent.new()
    h.take_damage(25)  # 违规：无准备步骤，断言不清晰
    assert_true(h.current_health < 100)  # 违规：断言不精确
```
