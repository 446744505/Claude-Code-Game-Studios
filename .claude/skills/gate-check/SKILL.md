---
name: gate-check
description: "评估是否具备进入下一开发阶段的条件。输出 PASS/CONCERNS/FAIL 结论，并列出具体阻塞项与所需产物。"
argument-hint: "[目标阶段: systems-design | technical-setup | pre-production | production | polish | release]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write
---

# 阶段门控校验

本技能用于校验项目是否**可以**进入下一开发阶段，检查必备产物、质量标准和阻塞项。

**与 `/project-stage-detect` 的区别**：该技能是诊断性的（「我们现在在哪？」）。本技能是规范性的（「能否推进？」并给出正式结论）。

## 制作阶段（7 个）

项目按以下阶段推进：

1. **概念（Concept）** — 头脑风暴、游戏概念文档
2. **系统设计（Systems Design）** — 系统拆解、撰写 GDD
3. **技术搭建（Technical Setup）** — 引擎配置、架构决策
4. **预制作（Pre-Production）** — 原型、垂直切片验证
5. **制作（Production）** — 功能开发（Epic/Feature/Task 跟踪启用）
6. **打磨（Polish）** — 性能、试玩、修 Bug
7. **发行（Release）** — 上线准备、认证

**当门控通过时**，将新阶段名写入 `production/stage.txt`（单行，例如 `Production`），可立即更新状态行。

---

## 1. 解析参数

- **带参数**：`/gate-check production` — 校验进入该特定阶段的准备度
- **无参数**：使用与 `/project-stage-detect` 相同的启发式规则自动识别当前阶段，然后校验**下一阶段**的过渡

---

## 2. 阶段门控定义

### 门控：概念 → 系统设计

**必备产物：**
- [ ] `design/gdd/game-concept.md` 存在且有实质内容
- [ ] 游戏支柱已定义（在概念文档或 `design/gdd/game-pillars.md` 中）

**质量检查：**
- [ ] 游戏概念已通过审阅（`/design-review` 结论不是 MAJOR REVISION NEEDED）
- [ ] 核心循环已描述且可被理解
- [ ] 目标受众已明确

---

### 门控：系统设计 → 技术搭建

**必备产物：**
- [ ] 系统索引存在于 `design/gdd/systems-index.md`，且至少列出 MVP 系统
- [ ] `design/gdd/` 中至少有 1 份 GDD（除 game-concept.md 与 systems-index.md 外）

**质量检查：**
- [ ] GDD 通过设计审阅（8 个必填章节齐全）
- [ ] 系统依赖已在系统索引中映射
- [ ] MVP 优先级层级已定义

---

### 门控：技术搭建 → 预制作

**必备产物：**
- [ ] 引擎已选定（CLAUDE.md 技术栈中不再是 `[CHOOSE]`）
- [ ] 技术偏好已配置（`.claude/docs/technical-preferences.md` 已填写）
- [ ] `docs/architecture/` 中至少有 1 条架构决策记录（ADR）
- [ ] `docs/engine-reference/` 中存在引擎参考文档

**质量检查：**
- [ ] 架构决策覆盖核心系统（渲染、输入、状态管理）
- [ ] 技术偏好中已设定命名规范与性能预算

---

### 门控：预制作 → 制作

**必备产物：**
- [ ] `prototypes/` 中至少有 1 个原型且带 README
- [ ] `production/sprints/` 中存在首份冲刺计划
- [ ] 系统索引中所有 MVP 层级的 GDD 均已完成

**质量检查：**
- [ ] 原型验证了核心循环假设
- [ ] 冲刺计划引用 GDD 中的真实工作项
- [ ] 垂直切片范围已定义

---

### 门控：制作 → 打磨

**必备产物：**
- [ ] `src/` 中有按子系统组织的活跃代码
- [ ] GDD 中的核心机制均已实现（交叉对照 `design/gdd/` 与 `src/`）
- [ ] 主玩法路径可从头到尾游玩
- [ ] `tests/` 中存在测试文件
- [ ] 至少有 1 份试玩报告（或已运行 `/playtest-report`）

**质量检查：**
- [ ] 测试通过（通过 Bash 运行测试套件）
- [ ] 缺陷跟踪或已知问题中无严重/阻塞级 Bug
- [ ] 核心循环玩法符合设计（对照 GDD 验收标准）
- [ ] 性能在预算内（对照 technical-preferences.md 中的目标）

---

### 门控：打磨 → 发行

**必备产物：**
- [ ] 里程碑计划中的功能均已实现
- [ ] 内容完整（设计文档中引用的关卡、资产、对白等均存在）
- [ ] 本地化字符串已外置（`src/` 中无硬编码的玩家可见文案）
- [ ] 存在 QA 测试计划
- [ ] 平衡数据已审阅（已运行 `/balance-check`）
- [ ] 发行检查清单已完成（已运行 `/release-checklist` 或 `/launch-checklist`）
- [ ] 商店元数据已准备（如适用）
- [ ] 变更日志 / 补丁说明已起草

**质量检查：**
- [ ] 完整 QA 已由 `qa-lead` 签字通过
- [ ] 全部测试通过
- [ ] 所有目标平台均达到性能目标
- [ ] 无已知的严重、高或中等级 Bug
- [ ] 无障碍基础已覆盖（如适用：重映射、文字缩放等）
- [ ] 所有目标语言的本地化已验证
- [ ] 法律要求已满足（如适用：EULA、隐私政策、年龄分级等）
- [ ] 工程可干净编译与打包

---

## 3. 执行门控检查

对目标门控中的每一项：

### 产物检查
- 使用 `Glob` 与 `Read` 确认文件存在且有实质内容
- 不要只检查存在性 — 确认文件有真实内容（而非仅有模板标题）
- 对代码检查，验证目录结构与文件数量

### 质量检查
- 测试相关：若已配置测试运行器，通过 `Bash` 运行测试套件
- 设计审阅相关：`Read` GDD 并检查 8 个必填章节是否存在
- 性能相关：`Read` technical-preferences.md，并与 `tests/performance/` 中的分析数据或近期 `/perf-profile` 输出对照
- 本地化相关：在 `src/` 中用 `Grep` 查找硬编码字符串

### 交叉引用检查
- 将 `design/gdd/` 文档与 `src/` 实现对照
- 确认架构文档中引用的每个系统都有对应代码
- 确认冲刺计划引用的是真实工作项

---

## 4. 协作式评估

对无法自动核验的项，**询问用户**：

- 「我无法自动确认核心循环是否好玩。是否已经做过试玩？」
- 「未找到试玩报告。是否做过非正式测试？」
- 「没有性能分析数据。是否要运行 `/perf-profile`？」

**不要对无法核验的项默认 PASS。** 标记为 MANUAL CHECK NEEDED。

---

## 5. 输出结论

```
## 门控检查：[当前阶段] → [目标阶段]

**日期**：[date]
**检查方**：gate-check 技能

### 必备产物：[X/Y 已具备]
- [x] design/gdd/game-concept.md — 存在，2.4KB
- [ ] docs/architecture/ — 缺失（未发现 ADR）
- [x] production/sprints/ — 存在，1 份冲刺计划

### 质量检查：[X/Y 通过]
- [x] GDD 8/8 必填章节齐全
- [ ] 测试 — 未通过（tests/unit/ 中 3 个失败）
- [?] 核心循环已试玩 — 需人工确认

### 阻塞项
1. **无架构决策记录（ADR）** — 进入制作前请运行 `/architecture-decision` 创建，
   覆盖核心系统架构。
2. **3 个测试失败** — 推进前请修复 tests/unit/ 中的失败用例。

### 建议
- [解除阻塞的优先动作]
- [非阻塞的可选改进]

### 结论：[PASS / CONCERNS / FAIL]
- **PASS**：必备产物齐全，质量检查全部通过
- **CONCERNS**：存在小缺口，可在下一阶段补齐
- **FAIL**：存在关键阻塞，推进前必须解决
```

---

## 6. PASS 时更新阶段

当结论为 **PASS** 且用户确认要推进时：

1. 将新阶段名写入 `production/stage.txt`（单行，末尾无换行）
2. 此后所有会话的状态行会立即反映新阶段

示例：通过「预制作 → 制作」门控时：
```bash
echo -n "Production" > production/stage.txt
```

**写入前务必询问**：「门控已通过。是否允许将 `production/stage.txt` 更新为 'Production'？」

---

## 7. 后续动作

根据结论建议具体下一步：

- **没有游戏概念？** → `/brainstorm` 起草
- **没有系统索引？** → `/map-systems` 将概念拆成系统
- **缺少设计文档？** → `/reverse-document` 或委派 `game-designer`
- **缺少 ADR？** → `/architecture-decision`
- **测试失败？** → 委派 `lead-programmer` 或 `qa-tester`
- **没有试玩数据？** → `/playtest-report`
- **性能不明？** → `/perf-profile`
- **未本地化？** → `/localize`
- **准备发行？** → `/launch-checklist`

---

## 协作协议

本技能遵循协作设计原则：

1. **先扫描**：检查所有产物与质量门控
2. **未知处询问**：对无法核验的内容不要假设 PASS
3. **呈现结果**：展示完整清单及每项状态
4. **用户决策**：结论为建议 — 最终由用户拍板
5. **取得批准**：「是否允许将本门控检查报告写入 production/gate-checks/？」

**切勿**阻止用户推进 — 结论仅供参考。记录风险，由用户自行决定是否在存在顾虑的情况下继续。
