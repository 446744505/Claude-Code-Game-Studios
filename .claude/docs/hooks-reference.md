# 启用的 Hooks

Hooks 在 `.claude/settings.json` 中配置并自动触发：

| Hook | 事件 | 触发条件 | 动作 |
| ---- | ----- | ------- | ------ |
| `validate-commit.sh` | PreToolUse (Bash) | `git commit` 命令 | 校验设计文档章节、JSON 数据文件、硬编码值、TODO 格式 |
| `validate-push.sh` | PreToolUse (Bash) | `git push` 命令 | 在推送到受保护分支（develop/main）时发出警告 |
| `validate-assets.sh` | PostToolUse (Write/Edit) | 资源文件变更 | 检查 `assets/` 中文件的命名约定与 JSON 有效性 |
| `session-start.sh` | SessionStart | 会话开始 | 加载 sprint 上下文、milestone、git 活动；检测并预览用于恢复的 active session state 文件 |
| `detect-gaps.sh` | SessionStart | 会话开始 | 检测全新项目（建议 /start），以及存在 code/prototypes 时文档缺失情况，建议 /reverse-document 或 /project-stage-detect |
| `pre-compact.sh` | PreCompact | 上下文压缩 | 在 compaction 前将会话状态（active.md、已修改文件、WIP design docs）写入对话，以便经 summarization 后仍能保留 |
| `session-stop.sh` | Stop | 会话结束 | 总结已完成工作并更新 session log |
| `log-agent.sh` | SubagentStart | 子 Agent 启动 | 带时间戳的所有 subagent 调用的 audit trail |

Hook 参考文档：`.claude/docs/hooks-reference/`
Hook 输入 schema 文档：`.claude/docs/hooks-reference/hook-input-schemas.md`
