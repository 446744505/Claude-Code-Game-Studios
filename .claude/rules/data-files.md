---
paths:
  - "assets/data/**"
---

# 数据文件规范

- 所有 JSON 文件必须是合法 JSON —— 格式错误会阻塞整条构建流水线
- 文件命名：仅使用小写与下划线，遵循 `[系统]_[名称].json` 模式
- 每个数据文件都应有已文档化的 schema（JSON Schema，或写在对应设计文档中）
- 数值字段须在注释或配套文档中说明其含义
- 键名风格保持一致：JSON 内使用 camelCase
- 禁止孤立数据条目 —— 每条记录须被代码或其他数据文件引用
- 在发生破坏性 schema 变更时对数据文件做版本管理
- 所有可选字段应提供合理默认值

## 示例

**正确**的命名与结构（`combat_enemies.json`）：

```json
{
  "goblin": {
    "baseHealth": 50,
    "baseDamage": 8,
    "moveSpeed": 3.5,
    "lootTable": "loot_goblin_common"
  },
  "goblin_chief": {
    "baseHealth": 150,
    "baseDamage": 20,
    "moveSpeed": 2.8,
    "lootTable": "loot_goblin_rare"
  }
}
```

**错误**示例（`EnemyData.json`）：

```json
{
  "Goblin": { "hp": 50 }
}
```

违规点：文件名大写、键名大写、未遵循 `[系统]_[名称]` 模式、缺少必填字段。
