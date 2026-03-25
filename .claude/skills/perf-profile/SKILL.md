---
name: perf-profile
description: "结构化的性能剖析工作流。识别瓶颈、对照预算测量，并生成带优先级排序的优化建议。"
argument-hint: "[系统名称或 'full']"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash
---
当本技能被调用时：

1. **根据参数确定范围**：
   - 若为系统名：聚焦该系统的剖析
   - 若为 `full`：对所有系统做全面剖析

2. **读取性能预算** — 在设计文档或 CLAUDE.md 中查找既有性能目标：
   - 目标 FPS（例如 60fps = 16.67ms 帧预算）
   - 内存预算（总量与分系统）
   - 加载时间目标
   - Draw call 预算
   - 网络带宽上限（若有多人联机）

3. **分析代码库** 中的常见性能问题：

   **CPU 剖析重点**：
   - `_process()` / `Update()` / `Tick()` 等函数 — 列出并估算开销
   - 对大集合的嵌套循环
   - 热路径上的字符串操作
   - 每帧代码中的分配模式
   - 对游戏实体的未优化搜索/排序
   - 每帧执行的高开销物理查询（射线、重叠检测等）

   **内存剖析重点**：
   - 大型数据结构及其增长模式
   - 贴图/资源的内存占用估算
   - 对象池 vs 实例化/销毁模式
   - 泄漏引用（本应释放却未释放的对象）
   - 缓存大小与淘汰策略

   **渲染重点**（如适用）：
   - Draw call 估算
   - 透明物体重叠导致的 overdraw
   - Shader 复杂度
   - 未优化的粒子系统
   - 缺少 LOD 或遮挡剔除

   **I/O 重点**：
   - 存档/读档性能
   - 资源加载模式（同步 vs 异步）
   - 网络消息频率与体积

4. **生成剖析报告**：

   ```markdown
   ## 性能剖析：[系统名或 Full]
   生成时间：[Date]

   ### 性能预算
   | 指标 | 预算 | 估算当前 | 状态 |
   |------|------|----------|------|
   | 帧时间 | [16.67ms] | [estimate] | [OK/WARNING/OVER] |
   | 内存 | [target] | [estimate] | [OK/WARNING/OVER] |
   | 加载时间 | [target] | [estimate] | [OK/WARNING/OVER] |
   | Draw calls | [target] | [estimate] | [OK/WARNING/OVER] |

   ### 已识别热点
   | # | 位置 | 问题 | 估算影响 | 修复成本 |
   |---|------|------|----------|----------|
   | 1 | [file:line] | [description] | [High/Med/Low] | [S/M/L] |
   | 2 | [file:line] | [description] | [High/Med/Low] | [S/M/L] |

   ### 优化建议（按优先级）
   1. **[Title]** — [Description of the optimization]
      - 位置：[file:line]
      - 预期收益：[estimate]
      - 风险：[Low/Med/High]
      - 做法：[How to implement]

   ### 快赢项（每项 < 1 小时）
   - [Simple optimization 1]
   - [Simple optimization 2]

   ### 需进一步调查
   - [Area that needs actual runtime profiling to determine impact]
   ```

5. **输出报告** 并附摘要：前三热点、相对预算的估算余量、建议的下一步行动。

### 规则
- 未经测量不要优化 — 凭感觉判断性能不可靠
- 建议必须包含估算影响 — 仅说「变快」无法执行
- 在目标硬件上剖析，不要只在开发机上测
- 区分 CPU 受限、GPU 受限与 I/O 受限瓶颈
- 考虑最坏情况（实体数最多、最低规格硬件、最差网络条件）
- 静态分析（本技能）找出候选；运行时剖析再确认
