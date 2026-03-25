# CLAUDE.local.md 模板

将此文件复制到项目根目录并重命名为 `CLAUDE.local.md`，用于个人覆盖配置。
该文件会被 git 忽略，不会提交到仓库。

```markdown
# 个人偏好

## 模型偏好
- 复杂设计任务优先使用 Opus
- 快速查找与简单编辑使用 Haiku

## 工作流偏好
- 代码改动后始终运行测试
- 上下文用量达到 60% 时主动压缩
- 无关任务之间使用 /clear

## 本地环境
- Python 命令：python（或 py / python3）
- Shell：Windows 上使用 Git Bash
- IDE：VS Code 与 Claude Code 扩展

## 沟通风格
- 回复保持精简
- 所有代码引用中展示文件路径
- 简要说明架构决策

## 个人快捷指令
- 我说「review」时，对最近改动的文件执行 /code-review
- 我说「status」时，显示 git status 与 sprint 进度
```

## 设置步骤

1. 将本模板复制到项目根目录：`cp .claude/docs/CLAUDE-local-template.md CLAUDE.local.md`
2. 按需编辑以匹配你的偏好
3. 确认 `CLAUDE.local.md` 已列入 `.gitignore`（Claude Code 从项目根目录读取该文件）
