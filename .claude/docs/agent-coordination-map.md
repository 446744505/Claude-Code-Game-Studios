# Agent 协作与委派关系图

## 组织层级

```
                           [人类开发者]
                                 |
                 +---------------+---------------+
                 |               |               |
         creative-director  technical-director  producer
                 |               |               |
        +--------+--------+     |        （协调全局）
        |        |        |     |
  game-designer art-dir  narr-dir  lead-programmer  qa-lead  audio-dir
        |        |        |         |                |        |
     +--+--+     |     +--+--+  +--+--+--+--+--+   |        |
     |  |  |     |     |     |  |  |  |  |  |  |   |        |
     sys lvl eco  ta   wrt  wrld gp ep  ai net tl ui qa-t    snd
                                 |
                             +---+---+
                             |       |
                          perf-a   devops   analytics

  其他 Lead（向 producer / 总监汇报）:
    release-manager         -- 发布管线、版本号、部署
    localization-lead       -- i18n、字符串表、翻译管线
    prototyper              -- 快速可丢弃原型、概念验证
    security-engineer       -- 反作弊、漏洞利用、数据隐私、网络安全
    accessibility-specialist -- WCAG、色觉辅助、键位重映射、文字缩放
    live-ops-designer       -- 赛季、活动、战斗通行证、留存、线上经济
    community-manager       -- patch notes、玩家反馈、危机沟通

  引擎 Specialist（使用与所选引擎匹配的一套）:
    unreal-specialist  -- UE5 总览：Blueprint/C++、GAS 概览、UE 子系统
      ue-gas-specialist         -- GAS：能力、效果、属性、Gameplay Tags、prediction
      ue-blueprint-specialist   -- Blueprint：BP/C++ 边界、图表规范、优化
      ue-replication-specialist -- 网络：replication、RPC、prediction、带宽
      ue-umg-specialist         -- UI：UMG、CommonUI、控件层级、数据绑定

    unity-specialist   -- Unity 总览：MonoBehaviour/DOTS、Addressables、URP/HDRP
      unity-dots-specialist         -- DOTS/ECS：Jobs、Burst、hybrid renderer
      unity-shader-specialist       -- Shader：Shader Graph、VFX Graph、SRP 定制
      unity-addressables-specialist -- 资源：异步加载、bundle、内存、CDN
      unity-ui-specialist           -- UI：UI Toolkit、UGUI、UXML/USS、数据绑定

    godot-specialist   -- Godot 4 总览：GDScript、node/scene、signal、Resource
      godot-gdscript-specialist    -- GDScript：静态类型、模式、signal、性能
      godot-shader-specialist      -- Shader：Godot shading language、visual shader、VFX
      godot-gdextension-specialist -- Native：C++/Rust 绑定、GDExtension、构建系统
```

### 图例
```
sys  = systems-designer       gp  = gameplay-programmer
lvl  = level-designer         ep  = engine-programmer
eco  = economy-designer       ai  = ai-programmer
ta   = technical-artist       net = network-programmer
wrt  = writer                 tl  = tools-programmer
wrld = world-builder          ui  = ui-programmer
snd  = sound-designer         qa-t = qa-tester
narr-dir = narrative-director perf-a = performance-analyst
art-dir = art-director
```

## 委派规则

### 谁可以委派给谁

| 委派方 | 可委派至 |
|------|----------------|
| creative-director | game-designer, art-director, audio-director, narrative-director |
| technical-director | lead-programmer, devops-engineer, performance-analyst, technical-artist（技术决策） |
| producer | 任意 agent（仅在其职责范围内分配任务） |
| game-designer | systems-designer, level-designer, economy-designer |
| lead-programmer | gameplay-programmer, engine-programmer, ai-programmer, network-programmer, tools-programmer, ui-programmer |
| art-director | technical-artist, ux-designer |
| audio-director | sound-designer |
| narrative-director | writer, world-builder |
| qa-lead | qa-tester |
| release-manager | devops-engineer（发布构建）, qa-lead（发布测试） |
| localization-lead | writer（字符串审阅）, ui-programmer（文本适配） |
| prototyper | （独立工作，向 producer 与相关 lead 汇报结论） |
| security-engineer | network-programmer（安全审阅）, lead-programmer（安全编码模式） |
| accessibility-specialist | ux-designer（无障碍模式）, ui-programmer（实现）, qa-tester（a11y 测试） |
| [engine]-specialist | engine sub-specialists（委派子系统专项工作） |
| [engine] sub-specialists | （就引擎子系统模式与优化向所有程序员提供建议） |
| live-ops-designer | economy-designer（线上经济）, community-manager（活动沟通）, analytics-engineer（参与度指标） |
| community-manager | （与 producer 协调审批，与 release-manager 协调 patch notes 时机） |

### 升级路径

| 情况 | 升级至 |
|-----------|------------|
| 两名策划对某一机制意见不一 | game-designer |
| 玩法设计 vs 叙事冲突 | creative-director |
| 玩法设计 vs 技术可行性 | producer（主持协调），再 creative-director + technical-director |
| 美术 vs 音频调性冲突 | creative-director |
| 代码架构分歧 | technical-director |
| 跨系统代码冲突 | lead-programmer，再 technical-director |
| 部门间排期冲突 | producer |
| 范围超出产能 | producer，再 creative-director 决定是否删减 |
| 质量门禁分歧 | qa-lead，再 technical-director |
| 违反性能预算 | performance-analyst 标出，technical-director 裁决 |

## 常见工作流模式

### 模式 1：新功能（完整管线）

```
1. creative-director  -- 批准功能概念与愿景一致
2. game-designer      -- 撰写含完整规格的设计文档
3. producer           -- 排期并识别依赖
4. lead-programmer    -- 设计代码架构，勾勒接口草图
5. [specialist-programmer] -- 实现该功能
6. technical-artist   -- 实现视觉效果（如需要）
7. writer             -- 撰写文本内容（如需要）
8. sound-designer     -- 制作音频事件列表（如需要）
9. qa-tester          -- 编写测试用例
10. qa-lead           -- 审阅并批准测试覆盖
11. lead-programmer   -- 代码审阅（code review）
12. qa-tester         -- 执行测试
13. producer          -- 标记任务完成
```

### 模式 2：Bug 修复

```
1. qa-tester          -- 使用 /bug-report 提交缺陷报告
2. qa-lead            -- 分级严重度与优先级
3. producer           -- 排入 sprint（若非 S1）
4. lead-programmer    -- 定位根因，指派给程序员
5. [specialist-programmer] -- 修复 Bug
6. lead-programmer    -- 代码审阅（code review）
7. qa-tester          -- 验证修复并做回归
8. qa-lead            -- 关闭缺陷
```

### 模式 3：数值平衡调整

```
1. analytics-engineer -- 从数据（或玩家反馈）识别失衡
2. game-designer      -- 对照设计意图评估问题
3. economy-designer   -- 建模调整方案
4. game-designer      -- 批准新数值
5. [data file update] -- 修改配置数值
6. qa-tester          -- 对受影响系统做回归测试
7. analytics-engineer -- 监控变更后指标
```

### 模式 4：新区域/关卡

```
1. narrative-director -- 定义区域的叙事目的与节拍
2. world-builder      -- 构建 lore 与环境语境
3. level-designer     -- 设计布局、遭遇与节奏
4. game-designer      -- 审阅遭遇的玩法设计
5. art-director       -- 定义区域视觉方向
6. audio-director     -- 定义区域音频方向
7. [由相关程序员与美术实现]
8. writer             -- 撰写区域专属文本
9. qa-tester          -- 测试完整区域
```

### 模式 5：Sprint 周期

```
1. producer           -- 使用 /sprint-plan new 规划 sprint
2. [所有 agent]       -- 执行分配的任务
3. producer           -- 使用 /sprint-plan status 做每日状态
4. qa-lead            -- Sprint 期间持续测试
5. lead-programmer    -- Sprint 期间持续代码审阅（code review）
6. producer           -- 使用 post-sprint hook 做 Sprint 复盘
7. producer           -- 结合复盘规划下一 Sprint
```

### 模式 6：里程碑检查点

```
1. producer           -- 运行 /milestone-review
2. creative-director  -- 审阅创意进度
3. technical-director -- 审阅技术健康度
4. qa-lead            -- 审阅质量指标
5. producer           -- 主持上线/不上线讨论
6. [所有总监]    -- 如需则就范围调整达成一致
7. producer           -- 记录决策并更新计划
```

### 模式 7：发布管线

```text
1. producer             -- 宣布 release candidate，确认里程碑条件已满足
2. release-manager      -- 切发布分支，生成 /release-checklist
3. qa-lead              -- 跑完整回归，就质量签字
4. localization-lead    -- 确认所有字符串已翻译，文本适配通过
5. performance-analyst  -- 确认性能基准在目标内
6. devops-engineer      -- 构建发布产物，运行部署 pipeline
7. release-manager      -- 生成 /changelog，打 tag，撰写 release notes
8. technical-director   -- 重大发布最终签字
9. release-manager      -- 部署并监控 48 小时
10. producer            -- 标记发布完成
```

### 模式 8：快速原型

```text
1. game-designer        -- 定义假设与成功标准
2. prototyper           -- 使用 /prototype 搭建原型骨架
3. prototyper           -- 最小实现（以小时计，非以天计）
4. game-designer        -- 对照标准评估原型
5. prototyper           -- 撰写发现报告
6. creative-director    -- 是否进入正式制作的 go/no-go 决策
7. producer             -- 若批准则排产
```

### 模式 9：Live 活动 / 赛季上线

```text
1. live-ops-designer     -- 设计活动/赛季内容、奖励与时间表
2. game-designer         -- 校验活动相关玩法机制
3. economy-designer      -- 平衡活动经济与奖励数值
4. narrative-director    -- 提供赛季叙事主题
5. writer                -- 撰写活动描述与 lore
6. producer              -- 排实现工作
7. [由相关程序员实现]
8. qa-lead               -- 端到端测试活动流程
9. community-manager     -- 起草活动公告与 patch notes
10. release-manager      -- 部署活动内容
11. analytics-engineer   -- 监控活动参与与指标
12. live-ops-designer    -- 活动后分析与复盘
```

## 跨领域沟通协议

### 设计变更通知

设计文档变更时，game-designer 必须通知：
- lead-programmer（实现影响）
- qa-lead（需更新测试计划）
- producer（评估排期影响）
- 视变更而定的相关专业 agent

### 架构变更通知

创建或修改 ADR 时，technical-director 必须通知：
- lead-programmer（所需代码变更）
- 所有受影响的 specialist programmer
- qa-lead（测试策略可能变化）
- producer（排期影响）

### 资源标准变更通知

美术圣经或资源标准变更时，art-director 必须通知：
- technical-artist（管线变更）
- 所有使用受影响资源的内容创作者
- devops-engineer（若构建管线受影响）

## 应避免的反模式

1. **绕过层级**：专业 agent 未经沟通不得做出属于其 lead 的决策。
2. **跨领域实现**：未经相关负责人（owner）明确委派，agent 不得修改其职责范围外的文件。
3. **影子决策**：所有决策须留档。无书面记录的口头约定会导致矛盾。
4. **巨石任务**：分配给 agent 的任务应可在 1–3 天内完成；更大须先拆分。
5. **基于假设的实现**：规格模糊时，实现者须询问制定者而非猜测；猜错的代价比多问一句更高。
