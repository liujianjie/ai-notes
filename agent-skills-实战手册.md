# agent-skills 实战手册

> 入门篇见 `./agent-skills-中文教程.md`(讲清"7 个命令各做什么")。
> 本手册是**进阶篇**,专门答以下盲区:多周期管理、方向修正、bug 处理路径、长期维护、Unity 三种子场景、`skills/` 下 23 个 skill 全部用途。

---

## 目录

- [0. 导读](#0-导读)
- [1. 心智模型](#1-心智模型)
- [2. 23 个 skill 全说明](#2-23-个-skill-全说明)
- [3. 7 个 slash 命令再读](#3-7-个-slash-命令再读)
- [4. 多周期与文件管理](#4-多周期与文件管理)
- [5. 功能模块修改](#5-功能模块修改)
- [6. 方向走错了怎么修正](#6-方向走错了怎么修正)
- [7. bug 处理决策树](#7-bug-处理决策树)
- [8. 维护期工作流](#8-维护期工作流)
- [9. Unity 客户端开发场景](#9-unity-客户端开发场景)
- [10. 个人项目 vs 实际工程差异速查](#10-个人项目-vs-实际工程差异速查)
- [11. 常见疑难 Q&A](#11-常见疑难-qa)
- [12. 一页纸速查表](#12-一页纸速查表)

---

## 0. 导读

### 这本手册解决什么

入门篇回答的是**"每个命令做什么"**。这本手册回答的是**"多个周期串起来怎么用、走错了怎么退、长期维护怎么不变累赘"**——也就是一个项目活两个月之后会冒出来的问题。

具体覆盖你提的所有问题:

| 你的问题 | 在哪一章 |
|---|---|
| 命令完整解释 + 使用场景(Unity/工程/个人) | §3 + §9 + §10 |
| 功能模块修改 | §5 |
| 功能方向错了用什么命令修正 | §6 |
| 重新开 spec 时,todo plan 是否依旧只有一份 | §4.1, §4.2 |
| 已执行的记录,新开 spec 怎么处理 | §4.3 |
| 开发完有 bug 是原地修还是重新规划 | §7 |
| 修改 / 后期维护 / 新 bug 全面考虑 | §5, §6, §7, §8 |
| 23 个 skill 的全部用途 | §2 |

### 适用读者

主要给你自己用:Unity 客户端开发为主,场景包括 ①个人 Unity 工具 ②公司项目主导某模块(同事备资源) ③个人 Unity 独游 ④杂项(脚本、笔记工具、ai-notes 这种文档仓库)。每一节都假设读者直接翻进来,不依赖前文记忆。

### 怎么读

- **第一次读**:0 → 1 → 2(扫一眼速查) → 3 → 12(末尾速查表)
- **后面遇到具体问题**:直接跳到 4-9 对应的情境章
- **不知道哪个 skill 适用**:看 §2 的速查表
- **不记得命令产物**:看 §12 的速查表

---

## 1. 心智模型

### 三层结构:命令 / skill / agent

```
┌────────────────────────────────────────────────────────┐
│   slash 命令 (7 个)        ← 你手动输入的入口           │
│       ↓ 调用                                            │
│   skill (23 份)            ← 工程纪律工作流             │
│       ↓ 部分场景调用                                    │
│   agent (3 个 subagent)    ← 独立 context 的"另一双眼" │
└────────────────────────────────────────────────────────┘
```

- **命令**是**入口快捷方式**(`/spec`、`/plan`、`/build` 这些)。命令本身只有 30-80 行薄壳,真正的指令在 skill 里。
- **skill**是**工作流脚本**(SKILL.md),告诉 Claude 这个阶段的步骤、闸门、反例、验收。23 份覆盖完整开发周期。
- **subagent**是**独立人格**,有自己的 context window,不会污染你主对话。`/ship` 会并行召唤 3 个 subagent 做评审。

### 三种产物形态

每个命令/skill 输出的东西分三类,**记住这点能避免你"以为它会改代码,结果只产了报告"的尴尬**:

| 产物形态 | 哪些产 | 落盘吗? |
|---|---|---|
| **文档**(SPEC / plan / todo / ADR / GDD) | `/spec`、`/plan`、idea-refine、documentation-and-adrs | 是 |
| **代码 + 测试 + commit** | `/build`、`/test` | 是,改源码 |
| **报告**(评审 / 审计 / 决策) | `/review`、`/ship`、code-review-and-quality | **否,只输出在对话里**(除非你让它存) |

### 单数文件的根本约束

**这是最容易踩的坑**:

- `/spec` → 项目根 `SPEC.md`,**单数,会覆盖**
- `/plan` → `tasks/plan.md` + `tasks/todo.md`,**单数,会覆盖**

**默认设计假设你的项目同时只活一份 spec**。多 feature 并行 / 历史档案 / 团队场景都需要你自己改约定(见 §4)。

### 流程不是单向的

agent-skills 的官方图是 `SPEC → PLAN → TASKS → IMPLEMENT`,看起来单向,但**实战是闭环**:

```
       ┌─────────────────────────────────────┐
       │                                     │
       ▼                                     │
   /spec ──→ /plan ──→ /build ──→ /test ──→ /review ──→ /ship
                ↑          ↑                    │
                │          │                    │
                │          └─── bug 修复 ───────┤
                │                               │
                └────── 任务设计错 ─────────────┤
                                                │
       ┌────────────────────────────────────────┘
       │
       └── 方向走错了 → 回 /spec
```

§5、§6、§7 就是讲这些"回头"的箭头。

---

## 2. 23 个 skill 全说明

### 速查图

```
Meta(1)
└── using-agent-skills          每次 session 自动注入,告诉 Claude 怎么挑 skill

Define(3)                       想清楚要做什么
├── interview-me                逼问你究竟想要什么(95% 信心)
├── idea-refine                 想法发散+收敛,产一页纸
└── spec-driven-development     写 SPEC.md(6 节)

Plan(1)                         拆任务
└── planning-and-task-breakdown 垂直切片+依赖图+验收标准

Build(7)                        写代码
├── incremental-implementation  小步切片+atomic commit
├── test-driven-development     RED→GREEN→REFACTOR / Prove-It
├── context-engineering         给 Claude 喂对的上下文
├── source-driven-development   每个决定都引官方文档
├── doubt-driven-development    高风险决策走对抗式自查
├── frontend-ui-engineering     UI 组件/状态/响应式/无障碍
└── api-and-interface-design    契约优先+边界校验

Verify(2)                       验
├── browser-testing-with-devtools  用 Chrome DevTools MCP 看真实运行
└── debugging-and-error-recovery   Reproduce→Localize→Reduce→Fix→Guard

Review(4)                       质量闸门
├── code-review-and-quality     五轴评审+严重度分级
├── code-simplification         保持行为前提下化简
├── security-and-hardening      OWASP+边界三层
└── performance-optimization    Core Web Vitals+先测再改

Ship(5)                         上线
├── git-workflow-and-versioning  trunk-based+atomic commit
├── ci-cd-and-automation         Shift Left+feature flag
├── deprecation-and-migration    废弃清理+迁移
├── documentation-and-adrs       ADR 记决策
└── shipping-and-launch          预发布清单+回滚方案
```

### Meta

#### using-agent-skills

| 字段 | 内容 |
|---|---|
| 一句话 | 元 skill,告诉 Claude 怎么从一句话需求挑出该用哪个 skill |
| 触发方式 | **每次新 session 自动注入**(前提是本机有 `jq`) |
| 何时用 | 你不确定哪个 skill 适用时,对 Claude 说"看 using-agent-skills 的 flowchart" |
| 何时不用 | 你已经知道目标 skill 名字,直接说"用 agent-skills:xxx" 更快 |
| 产物 | 一段决策树式分支说明 |
| 和命令关系 | **不被任何命令直接调**,但所有命令的最初分流都依赖它 |

> 检查 jq:`jq --version`。Windows 没有就 `winget install jqlang.jq` 或 `scoop install jq`。

### Define(定义)

#### interview-me

| 字段 | 内容 |
|---|---|
| 一句话 | 一次问一个问题逼问到 95% 信心,挖出你"真正想要的"而非"以为应该要的" |
| 触发方式 | 你说"interview me"、"grill me"、"are we sure"、"stress-test my thinking",或 Claude 发现需求暧昧时主动启动 |
| 何时用 | ①需求只有一句话 ②你自己也没想清楚 ③要进 /spec 但回答不上"给谁用"、"成功长啥样" |
| 何时不用 | 需求已经清晰可执行;只是修一个具体 bug |
| 产物 | 对话过程 + 最终一段总结 |
| 和命令关系 | **/spec 的前置预热**,不被命令自动调 |

#### idea-refine

| 字段 | 内容 |
|---|---|
| 一句话 | 想法发散到 5-8 个变体 → 收敛到 2-3 个方向 → 产一页纸 |
| 触发方式 | 你说"ideate ……"、"refine this idea"、"stress-test my plan" |
| 何时用 | 脑子里只有一句模糊念头,问"给谁用 / 成功标准 / 最小版本"答不上来 |
| 何时不用 | 需求已经清晰直接 /spec;修 bug;技术选型(那是 source-driven) |
| 产物 | `docs/ideas/<idea-name>.md` 一页纸,固定 6 节(Problem / Recommended Direction / Key Assumptions / MVP Scope / **Not Doing** / Open Questions) |
| 和命令关系 | /spec 的前置(可选);**不是 slash 命令**,只能口头唤起 |

> Not Doing 一节最有价值:显式写下"为什么不做哪些好想法",后续 SPEC 不会飘。

#### spec-driven-development

| 字段 | 内容 |
|---|---|
| 一句话 | 写一份覆盖目标/命令/结构/风格/测试/边界 6 节的规格文档 |
| 触发方式 | 你跑 `/spec`,或对 Claude 说"用 agent-skills:spec-driven-development" |
| 何时用 | 新项目 / 新大模块 / 老项目方向变了 / 改动跨多文件且超 30 分钟 |
| 何时不用 | 单行修复 / 改字 / 已有等价 SPEC |
| 产物 | 项目根 `SPEC.md`(单数,覆盖) |
| 和命令关系 | `/spec` 命令的核心 skill |

> 4 阶段闸门:SPECIFY → PLAN → TASKS → IMPLEMENT,每阶段都要 human review 才能进下一阶段。

### Plan(计划)

#### planning-and-task-breakdown

| 字段 | 内容 |
|---|---|
| 一句话 | 把 SPEC 切成"垂直切片"任务,每个任务带验收标准+验证步骤+依赖关系 |
| 触发方式 | 你跑 `/plan`,或说"用 agent-skills:planning-and-task-breakdown" |
| 何时用 | 有 SPEC 等拆任务 / 任务大到不知从哪下手 / 要并行 / 要给人看清范围 |
| 何时不用 | 单文件改动 / SPEC 已经包含明确 todo |
| 产物 | `tasks/plan.md`(总体计划+依赖图)+ `tasks/todo.md`(可勾选清单) |
| 和命令关系 | `/plan` 命令的核心 skill |

> 任务大小:XS(1 文件)、S(1-2)、M(3-5)、L(5-8)、XL(必须拆)。L 以上要再切。

### Build(实现)

#### incremental-implementation

| 字段 | 内容 |
|---|---|
| 一句话 | 一次只推一个垂直切片,每片都能编译/测试/提交,失败可回滚 |
| 触发方式 | 你跑 `/build`,或说"用 agent-skills:incremental-implementation" |
| 何时用 | 任何跨多文件的改动 / 要写很多代码时先逼自己分片 |
| 何时不用 | 单行 / 单函数 |
| 产物 | 一系列 atomic commit,每个 commit 含代码+测试 |
| 和命令关系 | `/build` 命令的核心 skill,内部嵌入 TDD |

#### test-driven-development

| 字段 | 内容 |
|---|---|
| 一句话 | 新功能走 RED→GREEN→REFACTOR;bug 走 Prove-It(先写失败测试复现,再修) |
| 触发方式 | 你跑 `/test`,或说"用 agent-skills:test-driven-development" |
| 何时用 | 实现任何逻辑、修任何 bug、改任何行为 |
| 何时不用 | 改字 / 改 README / 改注释 |
| 产物 | 测试文件 + 通过的代码改动(可能不自动 commit) |
| 和命令关系 | `/test` 命令的核心 skill;`/build` 内部也用它 |

> 测试金字塔默认 80/15/5(单元/集成/E2E)、DAMP > DRY、Beyonce Rule(if you liked it, you should have put a test on it)。**Unity 项目要打折用,见 §9.1**。

#### context-engineering

| 字段 | 内容 |
|---|---|
| 一句话 | 在合适时机给 Claude 喂对的上下文(rules 文件、context 打包、MCP 集成) |
| 触发方式 | 自动激活(Claude 觉得 context 不够时);或你说"用 agent-skills:context-engineering" |
| 何时用 | 新会话开始时配 CLAUDE.md / 任务切换 / 输出质量明显下降 |
| 何时不用 | 单纯继续上一个任务 |
| 产物 | CLAUDE.md / .cursorrules / GEMINI.md 等 rules 文件,或一次性 context dump |
| 和命令关系 | 不被命令直接调,**所有命令的隐性前置** |

#### source-driven-development

| 字段 | 内容 |
|---|---|
| 一句话 | 用任何框架/库都引官方文档,标注哪些是已验证、哪些是猜的 |
| 触发方式 | 你说"用 source-driven-development" 或"基于官方文档实现 X" |
| 何时用 | 用不熟的框架 / 库版本可能更新 / 决策可能影响后期 |
| 何时不用 | 用了 5 年的语言/库写一个简单脚本 |
| 产物 | 代码 + 引用注释(链接到官方文档) |
| 和命令关系 | 不自动调,需要你显式触发 |

#### doubt-driven-development

| 字段 | 内容 |
|---|---|
| 一句话 | 对每个非平凡决定走 5 步对抗式自查:CLAIM→EXTRACT→DOUBT→RECONCILE→STOP |
| 触发方式 | 你说"用 doubt-driven-development" 或在高风险阶段主动启用 |
| 何时用 | 生产环境 / 安全相关 / 不可逆操作 / 不熟悉的代码区 / 验证比 debug 便宜时 |
| 何时不用 | 玩具项目 / prototype / 只想跑通 |
| 产物 | 推理过程 + 自我反驳记录 + 修正后结论 |
| 和命令关系 | 不自动调,可在 `/build` 或 `/review` 里嵌入 |

> 这是 agent-skills 里**最贵**的 skill(token 消耗最大,因为反复自查)。别滥用。

#### frontend-ui-engineering

| 字段 | 内容 |
|---|---|
| 一句话 | 组件架构、设计系统、状态管理、响应式、WCAG 2.1 AA 无障碍 |
| 触发方式 | 自动(看到 React/Vue/Svelte 等)或你说"用 frontend-ui-engineering" |
| 何时用 | 写或改前端 UI |
| 何时不用 | 后端 / Unity / CLI |
| 产物 | 组件代码 + 测试 + 无障碍标注 |
| 和命令关系 | `/build` 在前端项目中自动 escalate 到这个 |

> **Unity 不走这个**(Unity UI 是 UGUI/UI Toolkit,不是 web)。

#### api-and-interface-design

| 字段 | 内容 |
|---|---|
| 一句话 | 契约优先、Hyrum's Law、One-Version Rule、错误语义、边界校验 |
| 触发方式 | 自动(看到 REST/GraphQL/类库公共接口)或你说"用 api-and-interface-design" |
| 何时用 | 设计 API、模块边界、公共接口(TS 类型契约也算) |
| 何时不用 | 内部私有函数 / 一次性脚本 |
| 产物 | 接口定义 + 错误码表 + 边界校验逻辑 |
| 和命令关系 | `/build` 在涉及接口设计时自动调 |

> Unity 项目里"模块间 ScriptableObject 契约"、"Manager 公共方法"也适用。

### Verify(验证)

#### browser-testing-with-devtools

| 字段 | 内容 |
|---|---|
| 一句话 | 用 Chrome DevTools MCP 看真实 DOM/console/network/性能 |
| 触发方式 | 自动(浏览器场景);或你说"用 browser-testing-with-devtools" |
| 何时用 | 调浏览器里跑的任何东西、看 console 错误、抓网络请求、跑性能 profiling |
| 何时不用 | Unity / Node CLI / 后端 |
| 产物 | 运行时数据 + 截图 + 报告 |
| 和命令关系 | `/test` 在前端场景自动 escalate |
| 前置 | 配好 chrome-devtools MCP server |

#### debugging-and-error-recovery

| 字段 | 内容 |
|---|---|
| 一句话 | 系统化 5 步:Reproduce→Localize→Reduce→Fix→Guard,Stop-the-line 原则 |
| 触发方式 | 测试挂了 / build 坏了 / 行为不对 / 报错时自动激活;或你说"用 debugging-and-error-recovery" |
| 何时用 | **任何意外故障**——不要往下加代码,先查 |
| 何时不用 | 你已经知道根因,直接改就行 |
| 产物 | 复现步骤 + 根因分析 + 修复 + 防回归测试 |
| 和命令关系 | `/build` 任何步骤失败都会切到这个 |

> "Stop-the-line":出错就停,不要带着错往下加新功能。错会复合传染。

### Review(评审)

#### code-review-and-quality

| 字段 | 内容 |
|---|---|
| 一句话 | 五轴评审:正确性 / 可读性 / 架构 / 安全 / 性能,严重度 Nit/Optional/Important/Critical |
| 触发方式 | 你跑 `/review`,或说"用 agent-skills:code-review-and-quality" |
| 何时用 | 任何 PR / 大改完成 / 合主干前 / 给同事 review 之前自查 |
| 何时不用 | 改一行字 |
| 产物 | 评审报告(纯文本,带 file:line 引用),**不落盘** |
| 和命令关系 | `/review` 命令核心;`/ship` 通过 code-reviewer subagent 调它 |

> 要看一段代码的完整健康度,这是最直接的命令。

#### code-simplification

| 字段 | 内容 |
|---|---|
| 一句话 | Chesterton's Fence + Rule of 500,**保持行为**前提下化简 |
| 触发方式 | 你跑 `/code-simplify`,或说"化简这段代码" |
| 何时用 | 代码能跑但乱 / 长函数 / 深嵌套 / 名字差 |
| 何时不用 | 代码已经够清晰 / 你打算同时改行为(那是重构) |
| 产物 | 化简后的代码,测试仍过 |
| 和命令关系 | `/code-simplify` 命令核心 |

> 化简和重构的区别:化简**不动行为**,重构**可能动行为**。混用风险大。

#### security-and-hardening

| 字段 | 内容 |
|---|---|
| 一句话 | OWASP Top 10 + 认证模式 + 密钥管理 + 依赖审计 + 三层边界 |
| 触发方式 | 自动(碰到用户输入/认证/外部调用);或你说"用 security-and-hardening" |
| 何时用 | 处理用户输入、认证、数据存储、外部集成 |
| 何时不用 | 纯本地工具、不联网、不存数据 |
| 产物 | 加固后的代码 + 审计报告 |
| 和命令关系 | `/review` 安全轴会自动调;`/ship` 通过 security-auditor subagent 调它 |

> 对齐你全局 CLAUDE.md 的 P0 密钥铁律:任何明文 API Key/Token 都不进代码、对话、memory。这个 skill 默认就遵守。

#### performance-optimization

| 字段 | 内容 |
|---|---|
| 一句话 | 先测后改,Core Web Vitals 目标,bundle 分析,常见反模式 |
| 触发方式 | 自动(性能要求/性能回归);或你说"用 performance-optimization" |
| 何时用 | 有明确性能要求 / 怀疑回归 / Core Web Vitals 不达标 |
| 何时不用 | 还没测就想优化(过早优化) |
| 产物 | profiling 数据 + 优化代码 + 前后对比 |
| 和命令关系 | `/review` 性能轴自动调 |

### Ship(上线)

#### git-workflow-and-versioning

| 字段 | 内容 |
|---|---|
| 一句话 | trunk-based、atomic commit、change sizing(~100 行)、commit 即存档点 |
| 触发方式 | 自动(每次 commit/branch);或你说"用 git-workflow-and-versioning" |
| 何时用 | **每次代码改动**(总是隐性激活) |
| 何时不用 | / |
| 产物 | 干净的 commit history |
| 和命令关系 | 所有命令产 commit 时都遵守这个 |

> 分支寿命 1-3 天最佳,超过 1 周风险陡增。

#### ci-cd-and-automation

| 字段 | 内容 |
|---|---|
| 一句话 | Shift Left、Faster is Safer、feature flag、质量闸门、失败反馈循环 |
| 触发方式 | 你说"用 ci-cd-and-automation" 或动 `.github/workflows/`、`.gitlab-ci.yml` |
| 何时用 | 配 / 改 build / deploy / test pipeline |
| 何时不用 | 个人独游 / 本地工具(往往不需要 CI) |
| 产物 | CI 配置文件 + pipeline 设计 |
| 和命令关系 | 不自动调 |

#### deprecation-and-migration

| 字段 | 内容 |
|---|---|
| 一句话 | 代码即负债,强制 vs 建议性废弃,僵尸代码清理 |
| 触发方式 | 你说"用 deprecation-and-migration" 或要废弃旧 API/系统 |
| 何时用 | 删旧系统 / 迁用户 / 决定保留还是退役某模块 |
| 何时不用 | 还在加新功能 / 旧代码还有人用 |
| 产物 | 迁移计划 + 弃用通告 + 删除 PR |
| 和命令关系 | 不自动调,通常配合 `/spec` 在大版本规划时启用 |

#### documentation-and-adrs

| 字段 | 内容 |
|---|---|
| 一句话 | ADR(Architecture Decision Record)、API 文档、内联文档,**记 why 而不是 what** |
| 触发方式 | 自动(架构决策/API 变更/发功能);或你说"用 documentation-and-adrs" |
| 何时用 | 做架构决定 / 改公共 API / 上一个新功能 / 未来同事/agent 需要懂 |
| 何时不用 | 改一个内部函数实现 |
| 产物 | `docs/adr/<NNNN>-<title>.md` 或 README 段落 |
| 和命令关系 | `/ship` 阶段会要求至少有 ADR 摘要 |

> **Unity / 团队项目里这个非常关键**——同事会读 ADR 才知道你为什么这么做。

#### shipping-and-launch

| 字段 | 内容 |
|---|---|
| 一句话 | 预发布清单、feature flag 生命周期、灰度、回滚流程、监控 |
| 触发方式 | 你跑 `/ship`,或说"用 agent-skills:shipping-and-launch" |
| 何时用 | 准备上线 / 合主干 / 发版 |
| 何时不用 | 本地玩 / prototype / 还在 dev 阶段 |
| 产物 | GO/NO-GO 决策 + 回滚方案,合并三个 subagent 报告 |
| 和命令关系 | `/ship` 命令核心;并行召唤 code-reviewer + security-auditor + test-engineer |

---

## 3. 7 个 slash 命令再读

入门篇已经讲过每个命令做什么。这里只补**容易踩的坑**和**和 skill 的真实关系**。

### `/spec` — 写 SPEC.md

- **真实调用**:`spec-driven-development` skill。
- **失败模式**:你跑 `/spec` 时项目根**已经有 SPEC.md**,Claude 会问要不要覆盖——**没问就直接覆盖了是 bug**。手动备份再跑。
- **个人项目能不能跳过?** 能。如果有 idea-refine 的一页纸 + 自己脑子里清楚,直接 `/plan` 也行。SPEC.md 的真正增量在 commands / project structure / code style / testing / boundaries 几节,小项目这几节往往不需要。
- **Unity 写 SPEC 的注意**:不要把 GDD/美术/关卡塞进来(见 §9)。

### `/plan` — 写 plan.md + todo.md

- **真实调用**:`planning-and-task-breakdown` skill,期间进入只读 plan mode。
- **失败模式 1**:Claude 看到没 SPEC.md,可能擅自从 README 推断需求。**让它先 /spec 或明确告诉它任务源**。
- **失败模式 2**:横着拆而不是垂直切片。如果你看到任务列表是"先做所有 schema → 再做所有 API → 再做所有 UI",打断它,要求按 feature 切。
- **关键产物**:`tasks/todo.md` 是给 `/build` 吃的,**每条必须带验收标准**。没有验收标准的 todo 行,/build 会瞎做。

### `/build` — 推一个任务

- **真实调用**:`incremental-implementation` + `test-driven-development`。
- **失败模式 1**:Claude 一次吃多个任务。**强制要求"一次一个,做完先 commit 再问下一个"**。
- **失败模式 2**:遇到测试失败往下硬走。Stop-the-line:任何失败都切 `debugging-and-error-recovery`。
- **Unity 适配**:见 §9.1,把"build/test 通过"的判定改成 Unity Console + UnityMCP。

### `/test` — TDD 或 Prove-It

- **真实调用**:`test-driven-development`。
- **两种模式**:
  - 新功能 = RED → GREEN → REFACTOR
  - **bug = Prove-It**:先写一个**失败的测试复现 bug** → 跑测试确认确实失败 → 修代码 → 测试通过 → 全套回归
- **失败模式**:Claude 跳过"先写失败测试"直接修。打断,**测试必须先失败再绿,这是 Prove-It 的核心**。
- **不会自动 commit**(和 /build 不同),你自己 commit。

### `/review` — 五轴评审

- **真实调用**:`code-review-and-quality`,可能拉 `security-and-hardening` 和 `performance-optimization`。
- **产物只是报告,不改代码**。要根据报告改,自己再跑 `/build` 或手改。
- **失败模式**:对一个 1000 行 PR 跑 /review,容易输出表面意见。**先 split PR 到 ~100 行级再 /review,质量更高**。

### `/code-simplify` — 化简

- **真实调用**:`code-simplification`。
- **保持行为**——这是和重构的分水岭。你想顺手改逻辑,**别走这个命令**,会和原意冲突。
- **测试要绿才能动**——如果当前测试本身就挂着,先 `/test` 修绿再化简。

### `/ship` — 上线决策

- **真实调用**:并行 fan-out 三个 subagent(code-reviewer / security-auditor / test-engineer)+ 主 agent 合并 + `shipping-and-launch` skill 给回滚方案。
- **可跳过 fan-out 的唯一条件**:改动 ≤2 文件 且 diff <50 行 且 不动 auth/payments/data/config。
- **个人项目精简版**:跳过 security-auditor(自己玩没敏感数据),保留 code-reviewer + test-engineer。**手动跟 Claude 说**:"/ship 但只跑 code-reviewer 和 test-engineer"。

---

## 4. 多周期与文件管理

> 这一章答你最直接的问题:**重新开 spec 时 todo plan 是不是依旧只有一份?已完成的记录怎么办?**

### 4.1 默认约定:三件套都是单数

```
项目根/
├── SPEC.md              ← /spec 总是写这里
└── tasks/
    ├── plan.md          ← /plan 总是写这里
    └── todo.md          ← /plan 同时写这里
```

**这是 agent-skills 的默认假设**:同时只有一份活跃 spec。

`/spec` 二刷会**直接覆盖** `SPEC.md`(老的进 git history)。
`/plan` 二刷会**直接覆盖** `tasks/plan.md` 和 `tasks/todo.md`。

如果你直接二刷不做处理,**已经勾完的 todo 也会被覆盖**——但代码已经落地、commit 已存在,所以"丢"的只是清单,不是工作成果。

### 4.2 重新开 spec 的 3 种应对模式

| 模式 | 怎么做 | 适用 |
|---|---|---|
| **A. 直接覆盖** | 啥也不动,直接二刷 `/spec`,老的进 git log | 上一个 feature 真完工 + 代码已合并 + 你不打算回看老 todo 清单 |
| **B. 归档+刷新** | `mv tasks/{plan,todo}.md tasks/archive/2026-05-15-feature-A/`,然后 `/spec` | 上个 feature 还没完全收尾,或可能要回看哪些任务做了 |
| **C. 多 spec 子目录** | 改约定为 `specs/<feature>/{spec,plan,todo}.md`,跟 Claude 明说"任务源是 specs/<feature>/todo.md" | 长期多 feature 并行 / 团队 / 大型项目 |

#### 模式 C 的目录长这样

```
项目根/
├── specs/
│   ├── 001-character-controller/
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── todo.md
│   ├── 002-inventory/
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── todo.md
│   └── archive/
│       └── 000-prototype/      ← 已完工归档
│           └── ...
```

> 这正是 `spec-kit` 的目录结构(用户已熟)。**agent-skills 不强制,但你可以用同样模式**。跟 Claude 提交工作时明确说"任务源是 `specs/002-inventory/todo.md`,SPEC 是同目录 `spec.md`",它就会按那个走。

### 4.3 已完成的 todo / plan 怎么处理

#### 三种归档策略

| 策略 | 怎么做 | 优缺点 |
|---|---|---|
| **依赖 git history** | 直接覆盖,老的查 git log | 简单;但回看不直观,要跑 `git log -- tasks/todo.md` |
| **archive 子目录** | `mv` 到 `tasks/archive/<date>-<feature>/` | 简单可见;仓库会膨胀 |
| **删除+commit message 说明** | 在覆盖的那次 commit 里写"replaces tasks/* for feature-A,archived in commit XXX" | 干净;但要自律 |

**推荐策略**(独游/团队):**archive 子目录 + 在 ADR 里留指针**。

```
tasks/
├── plan.md          ← 当前活跃
├── todo.md
├── archive/
│   ├── 2026-04-12-character-base/
│   │   ├── plan.md
│   │   └── todo.md
│   └── 2026-04-28-inventory-mvp/
│       ├── plan.md
│       └── todo.md
└── README.md        ← 简短说明 archive 命名约定
```

ADR(`docs/adr/0007-inventory-mvp-completion.md`)里记录"该 feature 的 plan/todo 归档在 `tasks/archive/2026-04-28-inventory-mvp/`,关键决策见..."

#### 别犯的错

- ❌ 把已完成的 todo.md 一直留在 `tasks/todo.md`,新任务追加在末尾——**会让 /build 选错任务**
- ❌ 删除 todo 但不归档也不写 ADR——**3 个月后想回看完全找不到**
- ❌ 直接修改已勾选的 todo 行——**改写历史,git diff 会很乱**

### 4.4 何时合并 SPEC、何时分裂 SPEC

| 情境 | 选择 |
|---|---|
| 一个项目目标稳定,只是分阶段做 | **一份 SPEC.md** + 多份 plan/todo(模式 C 子目录) |
| 一个项目里两个 feature 目标差别很大(比如"战斗系统"和"商店系统") | **每个 feature 单独 spec.md**(模式 C) |
| 一个 monorepo 里多个独立产品(如客户端 + 服务器 + 工具) | **每个产品单独项目根 SPEC.md**(或者根本不放一个仓库) |
| 老项目接入 agent-skills | 不要写"覆盖整个项目"的 SPEC.md;**只为新模块写 spec**,见 §8 |

判据:**如果两份工作的 boundaries(always do / ask first / never do)不同,就要分裂 SPEC**。

---

## 5. 功能模块修改

> 这一章答"功能模块修改这种问题怎么办"。

### 5.1 三档规模决策

| 规模 | 改了什么 | 用什么命令 | 要不要动 SPEC/plan |
|---|---|---|---|
| **小修** | 改一两个函数实现 / 调一个参数 / 加一个 if 分支 | 直接 `/build` 或手改 + commit | 不动 |
| **中修** | 加一个新功能但在已规划范围内 / 改某模块对外接口 | 给 `tasks/todo.md` 追加 1-3 个新任务 → `/build` | 改 plan,不动 SPEC |
| **大修** | 改某模块的核心目标 / 替换技术栈一部分 / 调整模块边界 | 改 SPEC.md 对应小节 → 重跑 `/plan` → `/build` | 都动 |

### 5.2 决策树

```
你要改一个已开发模块。

  改的是函数实现,行为/接口不变?
  ├── YES → /build(对单个任务) 或手改 + atomic commit
  └── NO
        │
  改了对外接口,但模块整体目标没变?
  ├── YES → 在 tasks/todo.md 新增"接口迁移"任务 + 调用方更新任务
  │         然后 /build 一个一个推
  └── NO
        │
  模块目标变了 / 技术选型变了?
  ├── YES → 改 SPEC.md 对应小节 → /plan 重新拆这个模块的任务
  │         旧任务里"已完成的代码"挑能复用的留下,其余删
  └── NO
        │
  → 你其实只是 bug 或代码乱,看 §6 和 §7
```

### 5.3 真实例子

#### 例 1(小修):Unity 个人独游里"主角跳跃高度调高 20%"

```
直接改 PlayerController.cs 的 jumpForce 常量
  → atomic commit "tweak: jump force 600 → 720 for better feel"
不需要 /build,不需要测试(数值 tuning)
```

#### 例 2(中修):公司项目里"背包要加排序功能"

```
1. 在 specs/002-inventory/todo.md 新增:
   - [ ] T15: ItemSortStrategy SO + 实现 4 种排序
   - [ ] T16: BackpackView 加排序按钮 + 调用 strategy
2. /build T15
3. /build T16
不动 SPEC(背包目标没变,只是加能力)
```

#### 例 3(大修):个人独游里"战斗系统从回合制改为即时制"

```
1. /spec(改 SPEC.md 的 objective + commands + boundaries 三节)
2. 老 tasks/<battle>/todo.md 整体归档到 tasks/archive/2026-05-15-turn-based-battle/
3. /plan 基于新 SPEC 重新拆任务,新 todo 写到 specs/<battle-realtime>/todo.md
4. 旧战斗代码:能复用的(比如伤害计算公式、动画状态机)保留,其余删
5. /build 推进
6. 写 ADR docs/adr/0012-battle-system-pivot.md 记录"为何从回合制改即时"
```

---

## 6. 方向走错了怎么修正

> **核心一章**。用户问"功能方向错了用什么命令修正"。

### 6.1 三层错误模型

```
   [方案层]      ← SPEC 错:目标/用户/技术栈选错
       ↑
   [任务层]      ← plan 错:任务切错粒度/顺序错/拆错维度
       ↑
   [实现层]      ← 代码错:实现细节有问题
```

修正成本:实现层 < 任务层 < 方案层。**先问自己错在哪一层**,不要上来就推倒重来。

### 6.2 判断哪一层的判据

| 判据 | 错在哪 |
|---|---|
| "测试能补回来 / 函数能改 / 类能重写" | **实现层** |
| "我要重新决定先做 A 还是先做 B" | **任务层** |
| "这个任务粒度太大,要拆成 3 个" | **任务层** |
| "我要换前端框架 / 数据库 / 渲染管线" | **方案层** |
| "这个功能根本不该做 / 应该给另一类用户" | **方案层** |
| "用户故事整个错了" | **方案层** |

### 6.3 实现层错:不动 SPEC 不动 plan

```
1. git log --oneline 找到出错前的最后一个干净 commit
2. 选一个:
   a) git revert <bad-commit>           ← 已 push 用这个
   b) git reset --hard <good-commit>    ← 还在本地分支可以用,小心
3. 重新 /build 这个任务,这次给 Claude 更明确的约束
   "对 T07 重做,**这次注意**:接口签名必须是 Foo(int, string),
    不要用 dynamic,因为前一次写歪了"
4. 或 /code-simplify 化简后重写局部
```

> **atomic commit 在这里救命**。如果你一次 commit 干 5 件事,回滚就要砍干净的部分。所以 §3 才反复强调 /build 一次一个任务。

### 6.4 任务层错:重写 plan + todo,SPEC 不动

```
1. 在 tasks/todo.md 把"还没做的"任务标 ❌或注释掉(暂时,别删)
2. 已做的任务保留(代码已落地)
3. 跟 Claude 说:
   "我要重新规划任务。当前 T01-T05 已完成不要动,
    T06-T20 全部废弃。基于 SPEC.md 和现状,从 T06 重新拆任务。
    /plan"
4. Claude 会基于 SPEC + 现有代码重新生成 plan/todo
5. 把废弃的 T06-T20 历史保留在 archive/(可选)
不动 SPEC.md
```

### 6.5 方案层错:改 SPEC,几乎重启

```
1. 改 SPEC.md(可以手改,也可以 /spec 重写)
   - 改 objective 节(如果用户/目标变)
   - 改 boundaries 节(如果什么能做不能做的边界变)
2. 现有 tasks/<feature>/* 整个归档到 tasks/archive/<date>-<old-feature>/
3. 已写代码:
   - 能复用(数据结构、工具函数、稳定的子模块)→ 留下
   - 强绑老方向(比如老 UI、老协议)→ 删干净
4. /plan 基于新 SPEC 重新拆任务
5. /build 推进
6. **写 ADR** docs/adr/<NNNN>-pivot-from-X-to-Y.md
   记录"为何 pivot,旧方案的代价",防未来重复同样的错
```

### 6.6 决策图(总图)

```
代码已经写出来,但发现"不对"。

[1] 先停下加新代码 ───────→ Stop-the-line(出问题就停)

[2] 问自己: "我要改的是什么?"

  ├─ 实现细节 / 函数体 / 算法选择
  │   → 实现层
  │   → git revert 或 reset → /build 重做该任务 (§6.3)
  │
  ├─ 任务顺序 / 任务粒度 / 漏的依赖
  │   → 任务层
  │   → 重跑 /plan,保留已完成代码 (§6.4)
  │
  └─ 目标 / 用户 / 技术栈 / 模块边界
      → 方案层
      → 改 SPEC.md → /plan → /build,写 ADR (§6.5)

[3] 不管哪一层:
   - atomic commit 救命,所以不要一次 commit 干多件事
   - 走完后 /review 一次,确认没遗留垃圾
```

### 6.7 怎么对 Claude 说(口令)

| 场景 | 口令模板 |
|---|---|
| 实现层修正 | "T07 写歪了,先 git reset --hard 到 commit XXX,然后用 /build 重做 T07,这次注意 [具体约束]" |
| 任务层修正 | "重新规划。T01-T05 不动,T06-T20 作废,/plan 基于 SPEC.md 和现状从 T06 重拆" |
| 方案层修正 | "方向变了:从 X 改为 Y。改 SPEC.md 的 objective 和 boundaries,现有 tasks/<old>/ 全部归档到 archive/2026-05-15-old/,然后 /plan 重拆,写 ADR 记录 pivot 原因" |

---

## 7. bug 处理决策树

> 答"开发完有 bug 是原地解决还是重新规划 plan/spec"。

### 7.1 三档 bug 路径表

| bug 性质 | 命令 / skill | 流程 |
|---|---|---|
| **A. 单点可复现 + 单一函数错** | `/test` (Prove-It) | 写失败测试 → 修 → 测试通过 → 全套回归 |
| **B. 多模块串联 / 偶发 / 不可复现** | 手动调 `debugging-and-error-recovery` | Reproduce → Localize → Reduce → Fix → Guard 5 步 |
| **C. bug 揭示 SPEC 写错(根本设计漏了)** | `/spec` 二刷 | 改 SPEC → /plan → /build,bug 修复进新 todo |
| **D. 已上线 bug** | git revert / hotfix → ADR → /spec 校正 | shipping-and-launch + documentation-and-adrs |

### 7.2 决策树

```
报 bug。

[Step 0] STOP-THE-LINE
   不要继续加新功能。复合错误最难查。

[Step 1] 你能在自己机器上稳定复现吗?

  ├── YES → 进 Step 2
  └── NO  → 进 Step 4(B 路径)

[Step 2] bug 在一个模块还是跨模块?

  ├── 单模块,知道大致函数
  │   → A 路径:/test(Prove-It)
  │     "用 /test 修这个 bug:[复现步骤]
  │      Prove-It 模式:先写失败测试"
  │
  └── 跨多个模块,不确定根因
      → B 路径:debugging-and-error-recovery

[Step 3] 修完后问自己:
  这 bug 是因为 SPEC 漏了什么吗?
  比如"用户能通过路径 X 触发"——但 SPEC 根本没考虑过路径 X?

  ├── 不是,只是实现疏忽 → 修完结束,补一条防回归测试
  └── 是,SPEC 漏了    → C 路径:升级到 /spec 二刷

[Step 4] 不能复现:

  → debugging-and-error-recovery
    系统化排查:加 log / 缩减输入 / 跑 CI 干净环境 / 检查 race condition
    复现不出来时,documented(写明观察到的条件)+ 设监控,先继续别的工作
```

### 7.3 A 路径详解(Prove-It,最常用)

```
你说:
   "/test 修 bug:登录后 5 秒自动登出。
    复现:用 admin@x.com 登录,什么都不点,5 秒后回到登录页。"

Claude 会:
1. 写一个测试:
   it('should not logout user within 30 seconds of inactivity', () => {
     login(admin)
     advance time by 5000ms
     expect(currentRoute).not.toBe('/login')
   })
2. 跑测试 — **必须先红**(不红说明没复现到 bug)
3. 定位 bug(可能调 debugging-and-error-recovery)
4. 改代码
5. 跑测试 — 现在绿
6. 跑全套回归
7. (你来 commit)
```

> Prove-It 的意义:**测试就是 bug 的存档**。下次同样代码再出这种 bug,测试会自动抓。

### 7.4 B 路径详解(系统化排查)

```
你说: "用 debugging-and-error-recovery 查这个 bug:[现象]"

Claude 会按 5 步走:
1. Reproduce(复现):
   - 能复现 → 直接进 2
   - 不能 → 收集环境信息、看是不是时序/状态/环境依赖
2. Localize(定位):
   - 加 log、二分注释代码、看 stack trace
   - 找到出问题的最小代码段
3. Reduce(缩减):
   - 把触发条件简化到最小复现案例(MRE)
   - 这一步常常自己就把 bug 看出来了
4. Fix(修):
   - 改代码,**修根因不修症状**
5. Guard(防):
   - 加测试 / 加断言 / 加日志告警
   - 防止下次同样问题再出现
```

### 7.5 C 路径:bug 揭示 SPEC 漏

判别问句:**这个 bug 是不是因为 SPEC 根本没考虑过这种使用路径?**

例子:

- ❌ "用户输入空字符串崩了" → 不是 SPEC 错,是实现漏校验。A 路径。
- ✅ "支持离线模式后,登录态在 reconnect 时丢了" → SPEC 没写离线/reconnect 的处理策略。C 路径,补 SPEC。

C 路径走法:

```
1. 改 SPEC.md,在 objective 或 boundaries 里加进遗漏的路径
   例: "Always do: 在 reconnect 后保留登录态 30 秒"
2. 跑 /plan,把"修复这个 bug"作为一个或多个新任务加到 todo
3. /build 推进
4. 写 ADR 记录"为何 SPEC 当时没考虑到这种路径"——避免下次再漏
```

### 7.6 D 路径:已上线 bug

```
1. **先止血**:
   - git revert 出问题的 commit,或部署前一版本
   - 如果不能回退,配 feature flag 关掉新功能
2. **复盘**:
   - 写 ADR docs/adr/incident-2026-05-15-XXX.md
   - 记录:何时出问题、影响多少用户、怎么发现、怎么止血、根因
3. **修正**:
   - 选 A/B/C 路径根据 bug 性质
4. **加监控**:
   - 这种 bug 怎么提前抓到?加测试 / 加监控 / 加 alert
5. /ship 时把"这次 bug 的防回归"列入 verification
```

> Unity 客户端独游/工具一般没有"线上"概念,**D 路径主要给公司项目**。但发布到 Steam 后也会有上线 bug,届时一样套用。

---

## 8. 维护期工作流

> 答"后期维护怎么不让流程变累赘"。

### 8.1 长项目的三档节律

```
                   日常迭代              季度重构              大版本
  频率              每周                  3 个月              半年-1 年
  触发              新需求 / 小 bug       代码体检 / 性能      战略 pivot
  主用命令          /build /test          /code-simplify       /spec /plan
                    /review               /review              /build
  写 SPEC?         不写                  不写                 改 SPEC
  写 ADR?          关键决策才写          每次必写             必写
```

### 8.2 何时跑健康检查

| 信号 | 该跑什么 |
|---|---|
| 改一个小功能要看 5 个文件才懂 | `/code-simplify` 局部 |
| commit history 里反复出现"fix 上次 fix 的 bug" | `/review` 整段 + 写 ADR 找根因 |
| 新成员/未来的你看不懂某模块 | `documentation-and-adrs` 补文档 |
| 性能比上版差 | `performance-optimization` 先 profile 后改 |
| 依赖库一直没升级 | `deprecation-and-migration` 列迁移清单 |
| 整个模块没人用了 | `deprecation-and-migration` 决定保留还是删 |

### 8.3 老代码遗留的接入策略

**最大原则:不要给老项目"补一个 SPEC.md 覆盖整个项目"**——会膨胀成一份没人维护的废文档。

正确做法:**只为当前要做的新模块单独建 spec**。

```
老项目/                                  ← 已经存在,代码 50K 行
├── (一堆老代码,没 SPEC.md)
├── docs/
│   └── adr/
│       └── 0001-existing-decisions.md   ← 只记关键决策,不补全
└── specs/
    └── 015-new-recommendation-engine/   ← 新模块单独 spec
        ├── spec.md
        ├── plan.md
        └── todo.md
```

跟 Claude 说:"我们不写覆盖全项目的 SPEC,只为当前新模块 specs/015-recommendation-engine/ 写 spec 和 plan。已存在代码作为约束,不动。"

### 8.4 deprecation-and-migration 的实际用法

老代码积累一定会到要删的一天。三类场景:

| 场景 | 怎么删 |
|---|---|
| 完全没人用的死代码 | 直接删,commit 写明"remove unused module X — git history retains" |
| 还有少数地方用 | 标 deprecated → 加迁移指南 → 一段时间后删 |
| 整个模块被新模块取代 | feature flag 切流量 → 灰度 → 监控 → 下线 |

口令:"用 deprecation-and-migration 帮我列出 X 模块的迁移清单,标出每个调用点,生成迁移 PR 顺序"。

### 8.5 维护期的最小流程

```
每周:
  - 新需求 → /spec(小)或直接 /plan 加 1-3 个任务 → /build
  - bug → /test (Prove-It) 或 debugging skill
  - PR 合并前 → /review

每季度:
  - 跑 /code-simplify 在最复杂的 3 个模块
  - 跑 /review 在最近 50 个 commit
  - 写 ADR 总结这季度关键决定
  - 评估废弃 / 迁移清单(deprecation-and-migration)

每大版本(0.x → 1.0,或 1.0 → 2.0):
  - 改 SPEC.md(可能拆成多份子 spec)
  - /plan 重拆活跃任务
  - 归档老 todo 到 archive/
  - /ship 走完整流程
```

---

## 9. Unity 客户端开发场景

> 这是你的核心场景。三个子场景差异很大,先讲通用差异,再分别讲 A/B/C。

### 9.1 通用差异:Unity 项目和 Web SPA 不一样

agent-skills 默认是 Web/SaaS 视角(npm test、CI、前端组件)。Unity 项目要做以下映射:

| agent-skills 假设 | Unity 实际 | 怎么对齐 |
|---|---|---|
| `npm test` | Unity Test Runner(EditMode + PlayMode) | 多数游戏代码 EditMode 跑不动(依赖 MonoBehaviour),PlayMode 又慢又不稳 → **多数 Unity 项目"测试通过"靠 manual Play + UnityMCP** |
| `npm run build` | Unity 编译 + 平台 Build | **替代标准**:①Console 0 错误 ②编辑器/玩家场景能进入 ③可选的平台 Build(慢,不每次都跑) |
| 测试金字塔 80/15/5 | EditMode 占少数 / PlayMode 关键路径 / 手测大量 | 不要照搬 80/15/5,**核心系统(战斗、存档、经济)**写 EditMode 测试,关卡/UI/动画走手测 |
| 前端组件复用 | Prefab + ScriptableObject | api-and-interface-design 的"契约"在 Unity 里就是 SO 字段约定和 Manager 公共方法 |
| 前后端契约 | 客户端 ↔ 服务器协议 | 一样适用 api-and-interface-design |
| Chrome DevTools MCP | UnityMCP | browser-testing-with-devtools 完全用不上,改用 UnityMCP |
| CI/CD pipeline | Unity Cloud Build / GitHub Actions + ulauncher | ci-cd-and-automation 的原则适用,具体配置不同 |

### 9.1.1 UnityMCP / rescue-unity-dialog 的对接位置

你已有的工具链(全局 CLAUDE.md 写明):

- **UnityMCP**:让 Claude 能调 Unity Editor 命令(编译、跑测试、读 Console、调试场景)
- **`fishinggameplay/.claude/tools/rescue-unity-dialog.ps1`**:Unity 弹"Prefab Has Been Modified"对话框时,从 Win32 API 强关
- **`MCPPrefabStageBridge.cs`**:Editor menu,主动 SaveAsPrefabAsset + 反射清 dirty,避免对话框弹出

`/build` 8 步循环里"跑 build"和"跑测试"在 Unity 项目要替换成:

```
原 8 步                Unity 项目实际做什么
───────────────────────────────────────────────────────
1. 读验收标准          照旧
2. 加载相关上下文      照旧 + UnityMCP 读相关脚本/prefab
3. 写失败测试 (RED)    EditMode 能写就写;不能就跳过(非核心系统)
4. 写实现 (GREEN)      写 C# 代码
5. 全套回归            UnityMCP 触发 EditMode 测试 + 检查 Console 0 错误
6. 跑 build 验证编译   UnityMCP 触发编译 → 检查 Console 0 错误
                       (如果弹 Prefab dialog → 自动调 rescue-unity-dialog.ps1)
7. atomic commit       照旧
8. 标记完成            照旧
```

跟 Claude 说:"在 Unity 项目里跑 /build,build 验证用 UnityMCP 触发编译 + 检查 Console;如果遇到 Prefab Has Been Modified 对话框,调 .claude/tools/rescue-unity-dialog.ps1。"

### 9.1.2 GDD vs SPEC vs ADR 分工

对齐你全局 CLAUDE.md 的"协作级决策走 docs/design"原则:

| 文档 | 内容 | 位置 |
|---|---|---|
| **GDD**(Game Design Document) | 玩法、关卡、数值、美术风格、剧情 | `docs/design/gdd.md` 或细分多份 |
| **SPEC.md** | 工程契约:目标、命令、目录结构、代码风格、测试策略、boundaries | 项目根 SPEC.md(或 specs/<feature>/spec.md) |
| **ADR**(Architecture Decision Record) | 单个架构决策的"为什么这么选" | `docs/adr/<NNNN>-<title>.md` |
| **art-bible** | 美术风格、配色、字体、UI 规范 | `docs/design/art-bible.md`(同事/美术维护) |

> **不要把 GDD 内容塞进 SPEC.md**——GDD 会大改,SPEC 应该稳定。SPEC.md 引用 GDD 即可("玩法目标见 docs/design/gdd.md §3.2")。

> 同样**不要把 GDD 写进 auto memory**——GDD 是协作级决策,要进 git、要 review,memory 里最多留一条指针("项目 GDD 在 docs/design/gdd.md")。

### 9.1.3 Unity 任务的"垂直切片"长啥样

Web 项目垂直切片 = `schema + API + UI + 测试` 一条链。
Unity 项目垂直切片 = `数据 SO + Manager + View + Prefab + 接入主场景` 一条链。

例(背包加排序功能):

```markdown
## Task T15: 背包按物品类型排序

**验收标准:**
- [ ] 玩家点击"按类型排序"按钮,背包重新排列
- [ ] 同类型内按名字字母序
- [ ] 切换排序后选中状态不丢

**Verification:**
- [ ] EditMode 测试: SortStrategy.SortByType(testItems) 返回顺序正确
- [ ] Manual Play: 打开背包 → 加 10 个不同类型物品 → 点排序 → 视觉验证
- [ ] UnityMCP 截图保存到 docs/verify/T15-sort-by-type.png

**Files likely touched:**
- Assets/Scripts/Backpack/SortStrategies/SortByType.cs (新)
- Assets/Scripts/Backpack/BackpackView.cs(改:加排序按钮逻辑)
- Assets/ScriptableObjects/SortStrategies/SortByType.asset(新 SO 实例)
- Assets/Prefabs/UI/BackpackPanel.prefab(改:加按钮)

**Estimated scope:** M
```

### 9.2 子场景 A:个人 Unity 工具

代表:EditorWindow 工具(资源批量重命名、批量替换 SerializedField、自定义 Inspector、批量生成 prefab)、构建脚本、自动化任务。

#### 特征

- 代码量小(50-500 行)
- 用户只有你自己
- 没有发布、没有版本、没有用户数据
- 改坏了不影响游戏运行,只影响编辑效率
- 可能写完就不动几个月,然后某天突然要改

#### 推荐流程

| 阶段 | 做什么 |
|---|---|
| 起步 | **跳过 /spec**。对 Claude 说"我要做个 X 工具,目标是 Y,输入是 Z,输出是 W",让 idea-refine 一页纸或直接进 /build |
| 拆任务 | **跳过 /plan**。一两个任务直接做完 |
| 实现 | `/build`(但不强制 TDD)+ 手动验:打开窗口、跑一遍、关掉 |
| 测试 | 不强制 EditMode 测试,但每次改完手验 1-2 个用例 |
| 评审 | 工具长到 300+ 行时跑一次 `/code-simplify` |
| 上线 | 不需要 `/ship`,commit 完结束 |

#### 起步口令

```
"我要做一个 Unity 编辑器工具:批量替换所有 prefab 里指定字段的值。
输入:字段名 + 新值。输出:被替换的 prefab 列表。
先用 idea-refine 帮我打磨一下需求(尤其要回答:遇到嵌套 prefab 怎么处理?
要不要 dry-run 模式?),然后直接 /build。"
```

#### 对话框拦截

工具操作 prefab 时容易触发 "Prefab Has Been Modified" 对话框。配合:

- 在工具代码里调用 `MCPPrefabStageBridge` 的逻辑(SaveAsPrefabAsset + 反射清 dirty)
- 如果 Claude 在 /build 过程中卡住,告诉它"调 rescue-unity-dialog.ps1 -Action save"

#### 个人工具的反例(别这么做)

- ❌ 写一份 SPEC.md(50 行的工具不需要 6 节规格)
- ❌ 跑 /ship(没人发布)
- ❌ 强制写 EditMode 测试(投入产出比太差)
- ❌ 写 ADR(除非真有架构决策)

### 9.3 子场景 B:公司项目主导某模块(同事备资源)

代表:你在公司里负责某个模块(比如战斗系统、UI 系统、关卡加载),同事(美术、关卡设计、策划)给你提供资源(prefab、SO、关卡数据、贴图),但**主导工程实现的是你**。

#### 特征

- 代码主权在你
- 资源主权在同事
- 多人 git 仓库,要 review 要合主干
- 有 PM、可能有版本计划、可能有上线时间点
- bug 会被同事/PM 看到,质量要求高

#### 推荐流程

| 阶段 | 做什么 |
|---|---|
| 起步 | `/spec` 写 SPEC.md,**但只覆盖你的工程边界**(C# 代码契约、prefab 接入约定、与同事产出物的输入接口)。不写美术/关卡/数值。 |
| 拆任务 | `/plan`,**依赖图把"等同事 X 提交资源"标成阻塞依赖**。先做能做的,不要等。 |
| 实现 | `/build` 完整流程,EditMode 测试覆盖核心逻辑,prefab 集成走 PlayMode 或手测 |
| 评审 | `/review` 用得多,每个 PR 合主干前都跑 |
| 上线 | `/ship` 完整版(三个 subagent 都跑),合主干前必走 |

#### SPEC.md 的边界三层(你写,同事看)

```markdown
## Boundaries

### Always do
- 资源命名遵循约定 `Assets/<同事约定的命名空间>/`
- 公共 Manager 公共方法保持向后兼容(同事的代码会调)
- 改动 prefab 要在 PR 描述里截图前后对比

### Ask first
- 改 prefab 嵌套结构(可能影响同事的引用)
- 改公共 Manager 接口(同事代码可能直接调)
- 加新依赖库
- 改 .gitignore / 改 ProjectSettings

### Never do
- 动同事提交目录(`Assets/Art/`、`Assets/LevelData/`)
- 删 .meta 文件
- 改场景(.unity)文件而不写说明
- 提交未编译通过的代码到主干
```

#### 同事产出物的处理

同事的 GDD / 美术规范 / 关卡数据怎么进你的工作流?

```
项目根/
├── Assets/                       ← Unity 资源(同事和你都提交)
├── docs/
│   ├── design/
│   │   ├── gdd.md                ← 同事维护(策划)
│   │   ├── art-bible.md          ← 同事维护(美术)
│   │   └── external/             ← 你只读的同事产出物索引
│   │       └── battle-data-spec.md  ← 策划给的战斗数值规格
│   ├── adr/                      ← 你写的架构决策
│   └── verify/                   ← 验收用截图(UnityMCP 截图)
├── specs/
│   └── 008-battle-system/
│       ├── spec.md               ← 你写,只覆盖工程契约
│       ├── plan.md
│       └── todo.md
└── SPEC.md                       ← (可选)项目级 SPEC,通常没有
```

#### 资源未 ready 时的占位策略

跟 Claude 说:"T08 需要美术 prefab Assets/Art/Boss/Boss01.prefab,但还没提交。先用 Resources/Default/Capsule.prefab 占位,但接入逻辑要写好,等真正 prefab 来了直接替换。在 plan 里把 T08 标'阻塞:等同事 X 提交 Boss01.prefab',继续做 T09。"

#### 起步口令

```
"我在公司项目里负责战斗系统模块。
- 同事会提供:Boss prefab、技能 SO、伤害数值表
- 我负责:战斗管理器、技能释放逻辑、伤害计算、动画状态机调度
- 时间:3 周内交付 MVP,两周后要给 PM demo
- 协作约定:不动 Assets/Art/*,公共接口改要先 PR review

跑 /spec,SPEC 写在 specs/008-battle-system/spec.md,
只覆盖工程契约(我负责那部分),不写美术/数值。
boundaries 三层照协作约定写。然后 /plan 拆任务。"
```

#### 对话框 / UnityMCP 注意

公司项目里多人提交 prefab 是高频操作,每次跑 /build 都可能触发 dialog。让 Claude 在每次操作 prefab 前主动调 `MCPPrefabStageBridge.SaveAndCloseStage`(或 menu 里手动点),减少 rescue 频率。

### 9.4 子场景 C:个人 Unity 独游

代表:Steam/itch.io 发布的独立游戏项目,长周期(6 个月-2 年),你一人或小团队,有发布、有用户、有版本迭代。

#### 特征

- 长周期(超过 3 个月)
- 多版本迭代(0.1 → 0.2 → 1.0 → 1.x)
- 有真实用户(beta player / 玩家社区)
- 美术/关卡/音乐/剧情会大量返工
- 上线后会有 bug 反馈

#### 推荐流程

| 阶段 | 做什么 |
|---|---|
| 起步 | idea-refine 一页纸 → 写 GDD(`docs/design/gdd.md`)→ /spec 写工程 SPEC |
| 多 feature 管理 | **从一开始用模式 C(specs/<feature>/ 子目录)**,避免半年后才迁移 |
| 拆任务 | `/plan`,但**关卡/UI 任务很难写细致验收**——验收多用"GIF 录屏 / UnityMCP 截图 / Play 跑 5 分钟" |
| 实现 | `/build`,**核心系统**(战斗、存档、经济)写 EditMode 测试,**关卡/UI/动画**走手测 |
| 评审 | 每隔 5-10 个 commit 跑一次 `/review` |
| 上线 | `/ship` 个人精简版:跳过 security-auditor(独游一般无敏感数据),保留 code-reviewer + test-engineer |
| 版本收尾 | 每个版本(0.1 → 0.2)归档 `tasks/<feature>/todo.md` 到 `tasks/archive/v0.1-<feature>/`,写一份 ADR |

#### 关卡迭代怎么走

关卡迭代是 Unity 独游最反 TDD 的部分(关卡好不好玩没法用 unit test 测)。务实做法:

| 阶段 | 做什么 |
|---|---|
| 设计 | 在 GDD 里写关卡目标 + 大致流程 |
| 灰盒(white-box) | 用占位资源做出关卡几何 → /build 一个关卡加载任务 → manual play → 录 GIF 给自己看 |
| 测试 | 自己跑 3 遍 + 给 1-2 个 beta player → 收集反馈写到 `docs/playtest/<level>-<date>.md` |
| 调整 | 根据 playtest 改关卡 → 改 GDD 里关卡描述 → 不一定改 SPEC |
| 美化 | 同事或自己换正式资源 → 再 manual play 验视觉 |

`/test` 在这里没什么用(没法写"这关好玩"的测试)。但**关卡加载、状态保存、checkpoint** 这些核心系统**必须**有 EditMode 测试。

#### 起步口令(独游)

```
"我做个人独游,类型是横版动作 roguelike。
我已经有 GDD 草稿在 docs/design/gdd.md。
现在要正式开始工程实现,目标 6 个月做 MVP demo。
计划用模式 C(specs/<feature>/ 子目录)管理多 feature。

帮我:
1. 读 docs/design/gdd.md,识别 MVP 必要的工程模块(估计 4-6 个)
2. 在 specs/ 下为每个模块占位创建子目录,只写 spec.md
3. 优先级最高的模块跑 /plan 拆 todo
4. 写 docs/adr/0001-architecture-overview.md 记录初版架构选型"
```

#### 上线 /ship 个人精简版

```
跟 Claude 说:
"/ship 但只跑 code-reviewer 和 test-engineer 两个 subagent。
独游不存敏感数据,跳过 security-auditor。
另外加一项手测:在 macOS 和 Windows 各跑 build 一次,确认能启动。"
```

#### 版本收尾仪式

每个发布版本(包括 alpha / beta / 正式)做一次:

```
1. 把 specs/*/todo.md 中已勾完的任务整体归档到 tasks/archive/v0.1-<feature>/
2. 写一份版本 ADR docs/adr/release-v0.1.md 记录:
   - 这版做了什么
   - 哪些 spec 改了
   - 哪些 feature 推迟到下一版
   - 已知 bug
3. tag git: v0.1.0
4. 写发布说明 docs/releases/v0.1.md(给玩家看的版本)
```

### 9.5 三种场景的命令落地清单

| 场景 | /spec | /plan | /build | /test | /review | /code-simplify | /ship |
|---|---|---|---|---|---|---|---|
| **A. 个人工具** | 跳 | 跳 | ✅(轻量) | 不强制 | 长到 300 行跑一次 | 长到 300 行跑一次 | 跳 |
| **B. 公司模块** | ✅(只覆盖工程边界) | ✅(标资源依赖) | ✅(完整) | ✅(核心) | ✅(每 PR) | 季度跑 | ✅(完整 fan-out) |
| **C. 个人独游** | ✅(SPEC 引用 GDD) | ✅(模式 C 子目录) | ✅(核心写测试) | ✅(核心) | ✅(每 5-10 commit) | 季度跑 | ✅(精简版,跳 security) |

---

## 10. 个人项目 vs 实际工程差异速查

每个情境章里都散落了差异点,这里汇总一张表。

| 维度 | 个人项目 | 实际工程(团队) |
|---|---|---|
| /spec | 可省略(idea-refine 一页纸够用) | **必写**,boundaries 三层是协作契约 |
| /plan 子目录约定 | 单数文件够用(模式 A) | **必用模式 C**(specs/<feature>/) |
| /build 是否一定 TDD | 可以妥协(关卡/UI 难写测试) | 核心代码必走 TDD;但 UI/动画可妥协 |
| /test Prove-It | bug 修复几乎必走 | 必走 |
| /review 频率 | 每 5-10 commit 或大改前 | 每 PR |
| /code-simplify | 看心情 | 季度安排 + 每个长函数发现时 |
| /ship | 跳过或精简版 | 完整 fan-out |
| ADR | 关键决策才写 | 每个非平凡决策都写 |
| GDD/外部资源 | 简单写在 docs/design/ | 严格分工(GDD/art-bible/数值表都独立维护) |
| memory 用法 | 个人偏好/习惯/实验失败 | 团队约定不入 memory,入 CLAUDE.md/.claude/rules/ |
| 密钥管理 | 仍要严守(任何明文 Key 都不入对话/代码/memory) | 同左,加密钥管理器 |

---

## 11. 常见疑难 Q&A

### Q1. 跑 /plan 会覆盖我现有 tasks/todo.md 吗?

会。**默认行为是覆盖**。三种保护方式:

1. 跑 `/plan` 前手动 `cp tasks/todo.md tasks/todo.bak.md`
2. 用模式 C(specs/<feature>/),每次新 feature 写到自己子目录
3. 跑前明确告诉 Claude:"`/plan`,但产物写到 `specs/008-foo/todo.md`,不要动 `tasks/todo.md`"

### Q2. 多个 feature 同时进行怎么管?

模式 C(`specs/<feature>/` 子目录)+ 每次跟 Claude 说"任务源是 specs/<feature>/todo.md"。

并行 feature 之间互不影响,但**注意 git 分支策略**:同一时间在同一分支并行多 feature 会让 commit 历史混乱。建议每个 feature 单独短分支(1-3 天合)。

### Q3. /build 中途打断了怎么续?

直接说"继续 /build"。Claude 会从 todo.md 找下一个未勾选项目。如果你不确定上次到哪一步,看:

- `git log --oneline -5` 看最近 commit
- `tasks/todo.md` 看勾选状态
- 把上次的对话上下文(如果 Compact 过)简单复述一下

如果 commit 半截(比如代码改了但没 commit),要先决定:`git stash` 暂存还是 `git reset` 丢弃。

### Q4. /spec 写完发现需求又变了一次,白写了?

不算白写。SPEC.md 进了 git,本身就是历史。变化时:

- 小变化 → 直接编辑 SPEC.md 对应小节,commit "spec: update boundaries for X reason"
- 大变化 → 跑 `/spec` 二刷,旧 SPEC.md 在 git history(写一份 ADR 记 pivot 原因)

### Q5. SKILL.md 我没装插件,只读 markdown 行不行?

**完全可以**。SKILL.md 是纯 markdown,可以:

1. 直接 `cat skills/test-driven-development/SKILL.md` 复制到 prompt 里
2. 或在 CLAUDE.md / 项目 rules 里 `@import` 引用
3. 或贴到 ChatGPT / 别的工具

agent-skills 的"插件"价值主要在自动注入 + slash 命令快捷方式。**SKILL 内容本身和插件无关**。

### Q6. 同一仓库混用 agent-skills 和 spec-kit / Roo / Cursor 怎么办?

可以混。原则:

- **目录约定取交集**:模式 C(`specs/<feature>/`)既是 agent-skills 的合法约定,也是 spec-kit 的默认约定
- **不重复跑**:如果 spec-kit 已经生成了 specs/008-foo/spec.md 和 plan.md 和 tasks.md,**不要再跑 agent-skills 的 /spec /plan**,直接说"任务源是 specs/008-foo/tasks.md,跑 /build"
- **互补使用**:spec-kit 强在规格阶段产物,agent-skills 强在执行纪律(TDD、五维评审、并行 ship)。混用就是用 spec-kit 写规格 + agent-skills 跑实现和评审

### Q7. UnityMCP 调用 timeout 时是 agent-skills 的问题吗?

不是。UnityMCP timeout 99% 是 Unity Editor 弹了 modal 对话框阻塞主线程(全局 CLAUDE.md 写得很清楚)。

应对:

- 预防:Editor menu 主动 `SaveAsPrefabAsset` + 反射清 dirty(`MCPPrefabStageBridge`)
- 救援:`rescue-unity-dialog.ps1 -Action save|dont_save|cancel`

跟 agent-skills 流程无关。

### Q8. 老项目改造,从哪个命令切入?

参见 §8.3。**不要补一份覆盖全项目的 SPEC.md**。

- 老代码当作约束(用 context-engineering 把关键路径喂给 Claude)
- 当前要做的新功能 → /spec(只覆盖新模块)→ /plan → /build
- 老代码遇到要改的 → /code-simplify(化简)或 /test(补防回归测试)
- 要删老模块 → deprecation-and-migration

### Q9. 个人项目要不要 /ship?

**要,但精简版**。/ship 的核心价值不是"召唤 3 个 subagent",而是**强制你过一遍 pre-launch checklist**(回滚方案、文档、监控、feature flag)。

个人项目的精简版:

- 跳过 security-auditor(没敏感数据)
- 保留 code-reviewer(代码自己也会读不懂)
- 保留 test-engineer(独游 bug 修复成本高)
- 必有:回滚方案(就算只是"git revert + 告知朋友圈玩家")

### Q10. 多个 skill 描述都匹配,Claude 怎么选?

按优先级:

1. 你显式指定的 skill(`agent-skills:xxx`)永远赢
2. 否则 using-agent-skills 的 flowchart 走分支(主要看任务阶段:Define/Plan/Build/Verify/Review/Ship)
3. 同阶段多个 skill 匹配 → Claude 通常会都用(比如 /build 里同时调 incremental-implementation + test-driven-development)

如果发现 Claude 选错 skill,**显式说**:"用 agent-skills:source-driven-development,不要走 incremental-implementation"。

### Q11. /build 一直推不动同一个任务怎么办?

可能性:

| 症状 | 原因 | 修法 |
|---|---|---|
| 测试反复挂 | 任务设计有漏洞 / 实现策略错 | 切 debugging-and-error-recovery 排根因 |
| 编译反复错 | 上下文不够 / Claude 误解 API | 用 context-engineering 喂关键文件 |
| 改完一个测试又挂另一个 | 任务粒度太大,牵一发动全身 | 任务层错(§6.4),拆任务 |
| Unity 编辑器卡死 | 对话框阻塞 | rescue-unity-dialog.ps1 |
| 跑了 3 次还在试错 | 方向可能错 | 暂停,/review 看现状,可能是方案层错(§6.5) |

### Q12. agent-skills 和 Claude Code 内置的 plan mode / EnterPlanMode 冲突吗?

不冲突。

- Claude Code 的 plan mode 是**通用的"先规划再执行"**机制,任何任务都能用
- agent-skills 的 /plan 是**特定的"基于 SPEC 拆任务"**工作流,会写 plan.md + todo.md

`/plan` 内部本来就会用 plan mode(只读阶段)。你也可以在普通任务里手动进 plan mode,和 agent-skills 无关。

---

## 12. 一页纸速查表

> 打印出来贴屏幕边。

### "我现在手里是 X" → 命令 / skill

```
只有一句模糊念头           → ideate (skill, 非 slash)        产 docs/ideas/*.md 一页纸
有想法没 SPEC              → /spec                           产 SPEC.md
有 SPEC 没任务             → /plan                           产 tasks/plan.md + tasks/todo.md
有任务等实现               → /build                          产 代码+测试+commit (一次一个任务)
只想补测试 / 修 bug        → /test (Prove-It)                产 测试+少量代码改动
改完想体检                 → /review                         产 评审报告(纯文本)
能跑但代码乱               → /code-simplify                  产 化简后的 diff (行为不变)
准备合主干 / 部署          → /ship                           产 GO/NO-GO + rollback 方案

不知道用哪个               → "看 using-agent-skills 的 flowchart"
要权威文档支撑             → "用 source-driven-development"
高风险决策                 → "用 doubt-driven-development"
要废弃旧代码               → "用 deprecation-and-migration"
要写 ADR                   → "用 documentation-and-adrs"
```

### "我要修 / 改 / 退回" → 修正路径

```
代码写歪 / 实现细节错       → 实现层
                            → git revert 或 reset → /build 重做该任务
                            → 不动 SPEC 不动 plan

任务列表设计错 / 粒度错     → 任务层
                            → 重跑 /plan,保留已完成代码
                            → 不动 SPEC

目标 / 用户 / 技术栈错      → 方案层
                            → 改 SPEC.md → /plan 重拆 → /build
                            → 写 ADR 记 pivot 原因
```

### "bug 来了" → 处理路径

```
单点可复现                  → /test (Prove-It)
不可复现 / 跨多模块          → debugging-and-error-recovery 五步
bug 揭示 SPEC 漏           → /spec 二刷,补遗漏路径
已上线 bug                 → git revert / hotfix → ADR → 选 A/B/C 路径
```

### Unity 三场景命令落地

```
A. 个人工具       /spec ✗  /plan ✗  /build ✓ 轻量  /test 不强制  /ship ✗
B. 公司模块       /spec ✓ 工程边界  /plan ✓ 标依赖  /build ✓     /test ✓ 核心  /ship ✓ 完整
C. 个人独游       /spec ✓ 引 GDD    /plan ✓ 模式 C  /build ✓     /test ✓ 核心  /ship ✓ 精简(跳 security)
```

### 多周期文件管理

```
模式 A (单数,默认)      项目根/SPEC.md + tasks/{plan,todo}.md           ← 单 feature
模式 B (归档+刷新)       tasks/archive/<date>-<feature>/                  ← 收尾后保留
模式 C (子目录,推荐多)   specs/<feature>/{spec,plan,todo}.md              ← 长期多 feature
```

### 23 个 skill 速查

```
Meta:    using-agent-skills

Define:  interview-me / idea-refine / spec-driven-development

Plan:    planning-and-task-breakdown

Build:   incremental-implementation / test-driven-development /
         context-engineering / source-driven-development /
         doubt-driven-development / frontend-ui-engineering /
         api-and-interface-design

Verify:  browser-testing-with-devtools / debugging-and-error-recovery

Review:  code-review-and-quality / code-simplification /
         security-and-hardening / performance-optimization

Ship:    git-workflow-and-versioning / ci-cd-and-automation /
         deprecation-and-migration / documentation-and-adrs /
         shipping-and-launch
```

### 3 个 subagent

```
agent-skills:code-reviewer        五维评审(/ship 自动召)
agent-skills:security-auditor     OWASP 安全审计(独游可跳)
agent-skills:test-engineer        测试覆盖度分析
```

### 关键纪律(背下来)

```
1. STOP-THE-LINE     —— 出错就停,不要带错往下加新功能
2. atomic commit     —— 一个任务一个 commit,救命用
3. Prove-It          —— bug 修复必先写失败测试再修
4. 垂直切片          —— 任务按"一条完整链路"切,不横着拆
5. 单数文件警觉      —— /spec /plan 默认覆盖,多 feature 用模式 C
6. SPEC 不写 GDD     —— 协作级决策走 docs/design,memory 留指针
7. 密钥铁律          —— 任何明文 Key 不进对话/代码/memory(全局 P0)
```

---

## 附录:关键路径

### 源头(只读,别动)

```
G:\workspace\AIProject\agent-skills\
├── README.md                          总览
├── .claude\commands\                  7 个命令薄壳
│   ├── spec.md / plan.md / build.md
│   ├── test.md / review.md
│   ├── code-simplify.md / ship.md
├── skills\                            23 个 SKILL.md
├── agents\                            3 个 subagent persona
│   ├── code-reviewer.md
│   ├── security-auditor.md
│   └── test-engineer.md
├── references\                        5 份清单
│   ├── accessibility-checklist.md
│   ├── orchestration-patterns.md
│   ├── performance-checklist.md
│   ├── security-checklist.md
│   └── testing-patterns.md
└── docs\                              setup 指南(各工具)
```

### 你的工作仓库(本手册所在)

```
F:\0.学习\Note\ai-notes\
├── agent-skills-中文教程.md            入门篇(已存在)
└── agent-skills-实战手册.md            本手册
```

### Unity 工具链(全局可复用)

```
fishinggameplay\
├── Assets\Editor\MCPBridge\
│   └── MCPPrefabStageBridge.cs        预防对话框 (主动 SaveAsPrefab + 清 dirty)
└── .claude\tools\
    └── rescue-unity-dialog.ps1         救援对话框 (Win32 SendMessage BM_CLICK)
```

新 Unity 项目沿用:把这两个文件拷过去即可。

