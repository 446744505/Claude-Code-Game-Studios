# Hook：pre-commit-code-quality

## 触发时机

在任意会修改 `src/` 下文件的提交之前运行。

## 目的

在代码进入版本控制之前强制执行编码规范。可发现风格违规、缺失文档、方法过于复杂，以及本应由数据驱动的硬编码值。

## 实现

```bash
#!/bin/bash
# Pre-commit hook：代码质量检查
# 按语言与工具链调整具体检查项

CODE_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '^src/')

EXIT_CODE=0

if [ -n "$CODE_FILES" ]; then
    for file in $CODE_FILES; do
        # 检查玩法代码中的硬编码魔法数字
        if [[ "$file" == src/gameplay/* ]]; then
            # 查找可能是平衡数值的字面量
            # 按语言调整正则
            if grep -nE '(damage|health|speed|rate|chance|cost|duration)[[:space:]]*[:=][[:space:]]*[0-9]+' "$file"; then
                echo "警告：$file 可能含有硬编码玩法数值，请改用数据文件。"
                # 仅警告，不阻断提交
            fi
        fi

        # 检查无负责人的 TODO/FIXME
        if grep -nE '(TODO|FIXME|HACK)[^(]' "$file"; then
            echo "警告：$file 中的 TODO/FIXME 缺少负责人标签，请使用 TODO(name) 格式。"
        fi

        # 运行语言专用 linter（取消注释对应行）
        # GDScript：gdlint "$file" || EXIT_CODE=1
        # C#：dotnet format --check "$file" || EXIT_CODE=1
        # C++：clang-format --dry-run -Werror "$file" || EXIT_CODE=1
    done

    # 对变更过的系统运行单元测试
    # 取消注释并按测试框架调整
    # python -m pytest tests/unit/ -x --quiet || EXIT_CODE=1
fi

exit $EXIT_CODE
```

## Agent 集成

当本 hook 失败时：

1. **风格违规**：用格式化工具自动修复，或调用 `lead-programmer`
2. **硬编码数值**：调用 `gameplay-programmer` 将数值外置到数据文件
3. **测试失败**：调用 `qa-tester` 定位问题，并由 `gameplay-programmer` 修复
