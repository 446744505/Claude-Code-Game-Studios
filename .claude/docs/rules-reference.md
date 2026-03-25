# 路径专属规则

`.claude/rules/` 下的规则在编辑匹配路径下的文件时会自动生效：

| 规则文件 | 路径模式 | 约束内容 |
| ---- | ---- | ---- |
| `gameplay-code.md` | `src/gameplay/**` | 数据驱动数值、delta 时间、不引用 UI |
| `engine-code.md` | `src/core/**` | 热路径零分配、线程安全、API 稳定 |
| `ai-code.md` | `src/ai/**` | 性能预算、可调试性、数据驱动参数 |
| `network-code.md` | `src/networking/**` | 服务端权威、带版本消息、安全 |
| `ui-code.md` | `src/ui/**` | 不持有游戏状态、本地化就绪、无障碍 |
| `design-docs.md` | `design/gdd/**` | 必填 8 节、公式格式、边界情况 |
| `narrative.md` | `design/narrative/**` | 设定一致、角色口吻、正史层级 |
| `data-files.md` | `assets/data/**` | JSON 合法、命名约定、schema 规则 |
| `test-standards.md` | `tests/**` | 测试命名、覆盖率要求、fixture 模式 |
| `prototype-code.md` | `prototypes/**` | 放宽标准、需 README、记录假设 |
| `shader-code.md` | `assets/shaders/**` | 命名约定、性能目标、跨平台规则 |
