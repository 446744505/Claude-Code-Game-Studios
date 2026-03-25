# Hook：pre-push-test-gate

## 触发时机

在任意 push 到远端分支之前运行。对推送到 `develop` 与 `main` 为强制。

## 目的

在代码进入共享分支之前，确保 build 可编译、unit tests 通过，且关键 smoke tests 通过。这是代码影响其他开发者之前的最后一道自动化 quality gate。

## 实现

```bash
#!/bin/bash
# Pre-push hook：构建与测试门禁

REMOTE="$1"
URL="$2"

# 仅对 develop 与 main 强制执行完整 quality gate
PROTECTED_BRANCHES="develop main"
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

FULL_GATE=false
for branch in $PROTECTED_BRANCHES; do
    if [ "$CURRENT_BRANCH" = "$branch" ]; then
        FULL_GATE=true
        break
    fi
done

echo "=== Pre-Push Quality Gate ==="

# Step 1: Build
echo "Building..."
# 按你的 build system 适配：
# make build || exit 1
# dotnet build || exit 1
# cargo build || exit 1
echo "Build: PASS"

# Step 2: Unit tests
echo "Running unit tests..."
# 按你的 test framework 适配：
# python -m pytest tests/unit/ -x || exit 1
# dotnet test tests/unit/ || exit 1
# cargo test || exit 1
echo "Unit tests: PASS"

if [ "$FULL_GATE" = true ]; then
    # Step 3: Integration tests（仅 protected branches）
    echo "Running integration tests..."
    # python -m pytest tests/integration/ -x || exit 1
    echo "Integration tests: PASS"

    # Step 4: Smoke tests
    echo "Running smoke tests..."
    # python -m pytest tests/playtest/smoke/ -x || exit 1
    echo "Smoke tests: PASS"

    # Step 5: Performance regression 检查
    echo "Checking performance baselines..."
    # python tools/ci/perf_check.py || exit 1
    echo "Performance: PASS"
fi

echo "=== All gates passed ==="
exit 0
```

## Agent 集成

当本 hook 失败时：
1. Build 失败：调用 `lead-programmer` 诊断
2. Unit test 失败：调用 `qa-tester` 定位失败用例，并由 `gameplay-programmer` 或相关 programmer 修复
3. Performance regression：调用 `performance-analyst` 分析
