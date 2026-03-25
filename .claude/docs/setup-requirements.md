# 环境要求

本模板需安装若干工具才能发挥全部能力。若缺少工具，所有 hook 会**优雅降级**——不会崩溃，但会失去校验类能力。

## 必需

| 工具 | 用途 | 安装 |
| ---- | ---- | ---- |
| **Git** | 版本控制、分支管理 | [git-scm.com](https://git-scm.com/) |
| **Claude Code** | AI 代理 CLI | `npm install -g @anthropic-ai/claude-code` |

## 推荐

| 工具 | 使用方 | 用途 | 安装 |
| ---- | ---- | ---- | ---- |
| **jq** | Hook（8 个中的 4 个） | 在 commit/push/资源/agent 等 hook 中解析 JSON | 见下文 |
| **Python 3** | Hook（8 个中的 2 个） | 数据文件的 JSON 校验 | [python.org](https://www.python.org/) |
| **Bash** | 全部 hook | 执行 shell 脚本 | Windows 上随 Git for Windows 附带 |

### 安装 jq

**Windows**（任选其一）：
```
winget install jqlang.jq
choco install jq
scoop install jq
```

**macOS**：
```
brew install jq
```

**Linux**：
```
sudo apt install jq     # Debian/Ubuntu
sudo dnf install jq     # Fedora
sudo pacman -S jq       # Arch
```

## 平台说明

### Windows
- Git for Windows 自带 **Git Bash**，提供 `settings.json` 里所有 hook 使用的 `bash` 命令
- 请确认 Git Bash 在 PATH 中（用 Git 安装程序默认安装时一般已配置）
- Hook 使用 `bash .claude/hooks/[name].sh` —— 在 Windows 上可用，因为 Claude Code 会通过能找到 `bash.exe` 的 shell 执行命令

### macOS / Linux
- 系统自带 Bash
- 通过包管理器安装 `jq` 可获得完整 hook 支持

## 校验环境

运行以下命令检查前置条件：

```bash
git --version          # 应显示 git 版本
bash --version         # 应显示 bash 版本
jq --version           # 应显示 jq 版本（可选）
python3 --version      # 应显示 python 版本（可选）
```

## 缺少可选工具时会发生什么

| 缺失工具 | 影响 |
| ---- | ---- |
| **jq** | Commit 校验、push 保护、资源校验与 agent 审计等 hook 会静默跳过检查。仍可正常 commit 与 push。 |
| **Python 3** | Commit 与资源 hook 中的 JSON 数据文件校验被跳过。无效 JSON 可能在无提示下被提交。 |
| **两者皆无** | 所有 hook 仍会正常结束（退出码 0），但不做校验。相当于没有安全网。 |

## 推荐 IDE

Claude Code 可与任意编辑器配合，但本模板针对以下环境做了优化：
- 安装 Claude Code 扩展的 **VS Code**
- **Cursor**（兼容 Claude Code）
- 终端中的 Claude Code CLI
