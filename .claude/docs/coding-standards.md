# 编码规范

- 所有游戏代码必须为 public API 编写 doc comments
- 每个系统必须在 `docs/architecture/` 中有对应的 architecture decision record
- gameplay 数值必须 data-driven（外部 config），不得 hardcode
- 所有 public method 必须可 unit test（优先 dependency injection，而非 singleton）
- commits 必须引用相关 design document 或 task ID
- **Verification-driven development**：新增 gameplay system 时先写 tests。UI 变更须用 screenshots 验证。在标记工作完成前，将预期输出与 actual output 对比。每项实现都应有可证明其可用的方式。

# 设计文档规范

- 所有 design docs 使用 Markdown
- 每个 mechanic 在 `design/gdd/` 中有独立文档
- 文档必须包含以下 8 个必填章节：
  1. **Overview** — 单段摘要
  2. **Player Fantasy** — 预期感受与体验
  3. **Detailed Rules** — 无歧义的机制说明
  4. **Formulas** — 全部数学关系以变量定义
  5. **Edge Cases** — 异常情形处理
  6. **Dependencies** — 列出其他相关系统
  7. **Tuning Knobs** — 标出可配置项
  8. **Acceptance Criteria** — 可验证的成功条件
- balance 数值必须链接到其来源 formula 或 rationale
