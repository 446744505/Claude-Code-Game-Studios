# Hook：post-merge-asset-validation

## 触发时机

在任意合并到 `develop` 或 `main` 分支、且变更涉及 `assets/` 时运行。

## 目的

校验合并分支中的全部资源是否符合命名约定、体积预算与格式要求。避免不合规资源在集成分支上持续堆积。

## 实现

```bash
#!/bin/bash
# Post-merge hook：资源校验
# 按项目标准检查已合并的资源

MERGED_ASSETS=$(git diff --name-only HEAD@{1} HEAD | grep -E '^assets/')

if [ -z "$MERGED_ASSETS" ]; then
    exit 0
fi

EXIT_CODE=0
WARNINGS=""

for file in $MERGED_ASSETS; do
    filename=$(basename "$file")

    # 检查命名约定（小写 + 下划线）
    if echo "$filename" | grep -qE '[A-Z[:space:]-]'; then
        WARNINGS="$WARNINGS\nNAMING: $file -- 必须为小写并使用下划线"
        EXIT_CODE=1
    fi

    # 检查贴图尺寸（须为 power of 2）
    if [[ "$file" == *.png || "$file" == *.jpg ]]; then
        # 需要 ImageMagick
        if command -v identify &> /dev/null; then
            dims=$(identify -format "%w %h" "$file" 2>/dev/null)
            if [ -n "$dims" ]; then
                w=$(echo "$dims" | cut -d' ' -f1)
                h=$(echo "$dims" | cut -d' ' -f2)
                if (( (w & (w-1)) != 0 || (h & (h-1)) != 0 )); then
                    WARNINGS="$WARNINGS\nSIZE: $file -- 尺寸 ${w}x${h} 非 power-of-2"
                fi
            fi
        fi
    fi

    # 检查文件体积预算
    size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
    if [ -n "$size" ]; then
        # Textures：最大 4MB
        if [[ "$file" == assets/art/* ]] && [ "$size" -gt 4194304 ]; then
            WARNINGS="$WARNINGS\nBUDGET: $file -- ${size} bytes 超过 4MB texture 预算"
            EXIT_CODE=1
        fi
        # Audio：music 最大 10MB，SFX 最大 512KB
        if [[ "$file" == assets/audio/sfx* ]] && [ "$size" -gt 524288 ]; then
            WARNINGS="$WARNINGS\nBUDGET: $file -- ${size} bytes 超过 512KB SFX 预算"
        fi
    fi
done

if [ -n "$WARNINGS" ]; then
    echo "=== 资源校验报告 ==="
    echo -e "$WARNINGS"
    echo "================================"
    echo "运行 /asset-audit 获取完整报告。"
fi

exit $EXIT_CODE
```

## Agent 集成

当本 hook 报告问题时：

1. 命名违规：手动修复或调用 `art-director` 获取指导
2. 体积违规：调用 `technical-artist` 获取优化建议
3. 完整审计：运行 `/asset-audit` skill
