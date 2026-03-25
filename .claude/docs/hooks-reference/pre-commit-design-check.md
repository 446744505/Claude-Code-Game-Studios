# Hook：pre-commit-design-check

## 触发时机

在任意会修改 `design/` 或 `assets/data/` 下文件的提交**之前**运行。

## 目的

在设计文档与游戏数据进入版本控制之前，保证其一致性与完整性。在问题扩散前拦截缺失章节、断裂的交叉引用以及无效数据。

## 实现

```bash
#!/bin/bash
# Pre-commit hook：设计文档与游戏数据校验
# 置于 .git/hooks/pre-commit，或通过你的 hook 管理器配置

DESIGN_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '^design/')
DATA_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '^assets/data/')

EXIT_CODE=0

# 检查设计文档是否包含必填章节
if [ -n "$DESIGN_FILES" ]; then
    for file in $DESIGN_FILES; do
        if [[ "$file" == *.md ]]; then
            # GDD 文档必填章节（英文标题，与正文一致）
            if [[ "$file" == design/gdd/* ]]; then
                for section in "Overview" "Detailed" "Edge Cases" "Dependencies" "Acceptance Criteria"; do
                    if ! grep -qi "$section" "$file"; then
                        echo "错误：$file 缺少必填章节：$section"
                        EXIT_CODE=1
                    fi
                done
            fi
        fi
    done
fi

# 校验 JSON 数据文件
if [ -n "$DATA_FILES" ]; then
    for file in $DATA_FILES; do
        if [[ "$file" == *.json ]]; then
            # 查找可用的 Python 命令
            PYTHON_CMD=""
            for cmd in python python3 py; do
                if command -v "$cmd" >/dev/null 2>&1; then
                    PYTHON_CMD="$cmd"
                    break
                fi
            done
            if [ -n "$PYTHON_CMD" ] && ! "$PYTHON_CMD" -m json.tool "$file" > /dev/null 2>&1; then
                echo "错误：$file 不是合法 JSON"
                EXIT_CODE=1
            fi
        fi
    done
fi

exit $EXIT_CODE
```

## 与 Agent 的衔接

当本 hook 失败时，提交者应：

1. **缺失设计章节**：调用 `game-designer` agent 补全文档。
2. **无效 JSON**：调用 `tools-programmer` agent 修复，或手动修正。
