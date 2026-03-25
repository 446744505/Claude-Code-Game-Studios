# settings.local.json 模板

创建 `.claude/settings.local.json`，用于**不应**纳入版本控制的个人覆盖配置。请将其加入 `.gitignore`。

## settings.local.json 示例

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)",
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)"
    ]
  }
}
```

## 权限模式

Claude Code 支持多种权限模式。游戏开发场景下的建议如下：

### 日常开发（默认）

使用**普通模式**——Claude 在运行多数命令前会先询问。对生产代码而言最为稳妥。

### 原型阶段

使用**有限范围内的自动接受模式**——在可丢弃代码上迭代更快。仅在 `prototypes/` 目录下工作时使用。

### 代码审阅

使用**只读**权限——Claude 可读、可搜索，但不可修改文件。

## 本地自定义 Hooks

可在 `settings.local.json` 中添加个人 hooks，用于**扩展**（而非替换）项目级 hooks。例如在会话结束时打一条通知：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'echo Session ended at $(date)'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```
