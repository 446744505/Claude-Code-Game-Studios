# 引擎参考文档

本目录包含为本项目所用游戏引擎整理的、固定版本的文档快照。这些文件存在是因为 **LLM 知识有截止日期**，而游戏引擎更新频繁。

## 为何需要

Claude 的训练数据有知识截止日期（目前为 2025 年 5 月）。Godot、Unity、Unreal 等引擎会持续发布更新，带来破坏性 API 变更、新特性与弃用模式。没有这些参考文件，智能体会给出过时的代码建议。

## 结构

每个引擎有独立目录：

```
<engine>/
├── VERSION.md              # 固定版本、校验日期、知识缺口区间
├── breaking-changes.md     # 版本间 API 变更，按风险等级组织
├── deprecated-apis.md      # 「不要用 X → 用 Y」对照表
├── current-best-practices.md  # 模型训练数据中尚未覆盖的新实践
└── modules/                # 各子系统速查（每个约 150 行以内）
    ├── rendering.md
    ├── physics.md
    └── ...
```

## 智能体如何使用这些文件

引擎专项智能体应按以下方式使用：

1. 阅读 `VERSION.md` 以确认当前引擎版本
2. 在建议任何引擎 API 前先查 `deprecated-apis.md`
3. 针对版本相关问题查阅 `breaking-changes.md`
4. 针对子系统工作阅读相应的 `modules/*.md`

## 维护

### 何时更新

- 升级引擎版本之后
- LLM 模型更新（新的知识截止日期）之后
- 运行 `/refresh-docs` 之后（若可用）
- 发现模型经常搞错的 API 时

### 如何更新

1. 在 `VERSION.md` 中更新引擎版本与日期
2. 在 `breaking-changes.md` 中为版本过渡补充新条目
3. 将新弃用的 API 移入 `deprecated-apis.md`
4. 在 `current-best-practices.md` 中更新新模式
5. 在相关 `modules/*.md` 中更新 API 变更
6. 为所有修改过的文件设置「最后校验」日期

### 质量规则

- 每个文件必须包含 “Last verified: YYYY-MM-DD” 日期
- 模块文件保持在 150 行以内（上下文预算）
- 包含展示正确/错误模式的代码示例
- 提供官方文档链接以便核对
- 只记录与模型训练数据不一致的内容
