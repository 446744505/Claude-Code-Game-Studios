# 目录结构

```text
/
├── CLAUDE.md                    # 主配置
├── .claude/                     # Agent 定义、技能、钩子、规则与文档
├── src/                         # 游戏源码（core、gameplay、ai、networking、ui、tools）
├── assets/                      # 游戏资源（art、audio、vfx、shaders、data）
├── design/                      # 游戏设计文档（gdd、叙事、关卡、平衡）
├── docs/                        # 技术文档（架构、API、复盘）
│   └── engine-reference/        # 精选引擎 API 快照（按版本固定）
├── tests/                       # 测试套件（单元、集成、性能、试玩）
├── tools/                       # 构建与管线工具（ci、build、asset-pipeline）
├── prototypes/                  # 可丢弃原型（与 src/ 隔离）
└── production/                  # 生产管理（冲刺、里程碑、发布）
    ├── session-state/           # 临时会话状态（active.md — 已加入 gitignore）
    └── session-logs/            # 会话审计轨迹（已加入 gitignore）
```
