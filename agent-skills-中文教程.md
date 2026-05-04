# agent-skills 中文使用教程（针对你的 9 个问题）

> 来源：`C:\Users\Administer\.claude\plugins\marketplaces\addy-agent-skills`
> 作者整理：2026-05-02，结合本机 StockTrading（006-uniapp-multi-platform）项目实情

---

## 0. 它本质是什么？一句话

**agent-skills 不是一个测试工具，而是 21 份"工程纪律工作流" + 7 个 slash 命令快捷入口 + 3 个 subagent 人格。**

它的目的是让 Claude Code 在做任何事之前，**先按高级工程师的流程走一遍**：写规格 → 拆任务 → 小步实现 → 测试驱动 → 多维评审 → 干净提交 → 安全上线。

它**本身不会自动跑测试**，也**不会自动 commit**。它做的事情是：在你下达指令后，**强迫 Claude 按一套固定流程执行**（比如先写失败用例再写实现代码，比如评审必须过五维），从而提高产出质量。

---

## 1. 触发机制：4 条路径要分清

| 触发方式 | 谁来发起 | 何时用 |
|---|---|---|
| **Slash 命令**（7 个） | 你手动输入 `/spec` `/plan` `/build` `/test` `/review` `/code-simplify` `/ship` | 你想明确进入某个开发阶段时 |
| **Skill 自动激活** | Claude 看到任务匹配某 skill 的 description 时自动调用 | 你不指定阶段，让 Claude 自己挑流程 |
| **Skill 显式调用** | 你说"用 agent-skills:debugging-and-error-recovery 解决这个" | 想精确指定某一份工作流 |
| **Subagent 人格**（3 个） | `/ship` 内部并行 fan-out，或你显式要求 | 需要"换一双独立眼睛"做评审 |

**一个隐藏机制**：插件装了 `SessionStart` hook（`hooks/session-start.sh`），每次启动新会话会把 `using-agent-skills` 这份元 skill（含 flowchart）注入到 Claude 的系统消息里。**前提是本机有 jq**，没有 jq 就只是 fallback 降级，单独 skill 仍可用。

> 检查 jq：终端输入 `jq --version`。Windows 没有的话用 `winget install jqlang.jq` 或 `scoop install jq`。

---

## 2. 7 个 slash 命令详解（何时用 + 产出什么）

### 2.0 决策图：先看这一张

```
你现在手里是什么？
├─ 只有一句模糊念头，连能不能做都没想清 → ideate      产出：docs/ideas/*.md 一页纸（非 slash，是 skill）
├─ 想法已收敛，但缺结构化规格 ──────→ /spec        产出：SPEC.md
├─ 有 SPEC 但没拆任务 ─────────────→ /plan        产出：tasks/plan.md + tasks/todo.md
├─ 有任务列表等实现 ───────────────→ /build       产出：代码 + 测试 + 一次 commit（推进 1 个任务）
├─ 只想给某段逻辑补测试 / 修 bug ──→ /test        产出：失败测试 → 绿测试（可能伴随少量代码改动）
├─ 刚改完代码想体检 ───────────────→ /review      产出：五维评审报告（文本，不落盘）
├─ 代码能跑但又乱又长 ─────────────→ /code-simplify  产出：化简后的代码（行为不变，测试仍过）
└─ 准备发布 / 合主干 ──────────────→ /ship        产出：GO/NO-GO 决策文档 + 回滚方案
```

> 记住一条分水岭：`ideate /spec /plan` 是**写文档**；`/build /test /code-simplify` 是**改代码**；`/review /ship` 是**出报告**。
>
> 注意：**`ideate` 不是 slash 命令**，是 skill `agent-skills:idea-refine` 的触发短语。你对 Claude 说"ideate 这个想法"或"用 idea-refine 帮我打磨"即可唤起。它是 `/spec` 的前置步骤——**想法还没成型就不要进 /spec**，先用 idea-refine 把"这到底值不值得做"过一遍。

---

### 2.0.5 `ideate` / `agent-skills:idea-refine` — 打磨想法（Skill，非 slash）

| 项 | 内容 |
|---|---|
| **什么时候用** | 脑子里只有一句模糊念头（"想做个 X"），但回答不清"给谁用""成功长啥样""最小版本是什么"。在 `/spec` **之前**过一遍，避免规格写歪 |
| **什么时候别用** | 问题和用户已经清晰（直接 /spec）；只是修 bug 或加小功能（根本不需要 ideate）；你要的是技术选型对比（那是 `source-driven-development` 或直接聊） |
| **触发方式** | **不是 slash 命令**。对 Claude 说：①"ideate 这个想法：……" ②"用 idea-refine 帮我打磨" ③"stress-test 我的方案" |
| **Claude 会干什么** | 三阶段对话：<br>①**发散（Understand & Expand）**：把你的念头改写成 "How Might We……" 问句，问 3-5 个锐化问题（给谁 / 成功标准 / 约束 / 前人做过啥 / why now），然后用 6 种视角生成 5-8 个变体（反转 / 去约束 / 换受众 / 组合 / 简化 10x / 放大 10x）<br>②**收敛（Evaluate & Converge）**：把打动你的变体聚成 2-3 个方向，按"用户价值 / 可行性 / 差异化"压力测试，显式列出"在赌什么是真的 / 什么能杀掉这个想法 / 故意忽略了什么"<br>③**定型（Sharpen & Ship）**：产出一页纸 markdown |
| **产出** | 一页纸 markdown（需要你确认后才保存到 `docs/ideas/[idea-name].md`），固定 6 节：Problem Statement / Recommended Direction / Key Assumptions to Validate / MVP Scope / **Not Doing list** / Open Questions。其中 **Not Doing 是最有价值的一节**——显式写下为什么不做哪些"好想法"，后续 SPEC 不会飘 |
| **之后接什么** | 带着这份一页纸去跑 `/spec`，SPEC.md 就有了明确的目标、用户、边界 |
| **重要特性** | ①**不是 yes-machine**：idea 弱它会直接顶回来。你想要有人点头就别用这个 skill<br>②**会扫代码库**：如果在项目里调用，它会用 Glob/Grep/Read 看现有架构，变体会落在真实约束上<br>③**最多 5-8 个变体**，不是 20 个 |
| **最小示例** | 你说："ideate 这个想法：给自由职业者做个记账 App"<br>Claude 会问"给哪类自由职业者？设计师还是开发者？成功是什么——DAU 还是付费率？"，然后给 6 个变体（含"只做发票→入账全自动化，不做手工记账"这种反转版），最后产出一页纸里 Not Doing 明确写"不做预算规划、不做多币种"|

---

### 2.1 `/spec` — 写规格

> **可以跳过 /spec 的场景**：如果你已经有 `docs/ideas/xxx.md`（ideate 一页纸）+ `tasks/plan.md` + `tasks/todo.md` 三件套，**`SPEC.md` 不是必需的**——一页纸已经包含 Problem / Direction / MVP Scope / Not Doing，信息密度对小到中等项目足够，直接 /plan 或 /build 就行。SPEC.md 真正的增量价值在 Commands / Project Structure / Code Style / Testing Strategy / Boundaries 这几节，大型项目或多人协作才值得补。

| 项 | 内容 |
|---|---|
| **什么时候用** | 新项目／新大模块启动；或老项目方向变了（比如 UI 库换了、核心流程重写），需要重新对齐意图 |
| **什么时候别用** | 已经有 `SPEC.md` 或同等规格文件；只是改一个小 bug；探索阶段还没想清楚产品边界（先跟我聊，别急着 /spec） |
| **前置条件** | 你能用三句话说清：目标、用户、核心功能。说不清就先聊天迭代，不要进 /spec |
| **Claude 会干什么** | 问你 4 类澄清题：①目标和用户 ②核心功能和验收标准 ③技术栈偏好和约束 ④边界（always do / ask first / never do） |
| **产出** | **项目根目录新建 `SPEC.md`**，6 个固定小节：objective / commands / project structure / code style / testing strategy / boundaries |
| **之后接什么** | `/plan` |
| **最小示例** | 你说："做一个个人记账 App，iOS 优先，目标用户是自由职业者"，Claude 会开始问细节并最终产出 SPEC.md |

---

### 2.2 `/plan` — 拆任务

| 项 | 内容 |
|---|---|
| **什么时候用** | `SPEC.md` 已定稿，要把它切成"一个任务 = 一个可单独验证的切片" |
| **什么时候别用** | 没有 SPEC（先 /spec）；只是修一个已知 bug（直接 /test 或 /build 即可）；已有 `tasks.md`（比如 speckit 已生成的）——这种情况跳过 /plan 直接 /build |
| **前置条件** | 项目根存在 `SPEC.md` 或等价文件 |
| **Claude 会干什么** | ①进入 plan 只读模式 ②梳理组件依赖图 ③**垂直切片**（每个任务是一条完整链路，不是横着拆"先全部前端再全部后端"）④为每个任务写验收标准和验证步骤 ⑤阶段之间加 checkpoint ⑥提交给你审阅 |
| **产出** | **`tasks/plan.md`**（总体计划 + 依赖图）+ **`tasks/todo.md`**（可勾选的任务清单） |
| **之后接什么** | `/build` |
| **最小示例** | "按 SPEC.md 拆任务"，得到 `tasks/todo.md` 里 20 条带验收标准的任务 |

---

### 2.3 `/build` — 实现下一个任务（最常用）

| 项 | 内容 |
|---|---|
| **什么时候用** | 有任务列表，要把它变成能跑的代码。**这是日常主力命令**。 |
| **什么时候别用** | 还没想清楚要做什么（先 /spec 或 /plan）；只是想跑一下测试看结果（用 /test）；改动跨多个无关模块（拆开分别 /build） |
| **前置条件** | 任一任务源存在：`tasks/todo.md`、speckit 的 `specs/*/tasks.md`、或者你口头指定"做 T070" |
| **Claude 会干什么** | 对**一个**任务走完 8 步：①读验收标准 ②加载相关上下文 ③写**失败**测试（RED）④写最小代码让它通过（GREEN）⑤跑全套回归 ⑥跑 build 验证编译 ⑦写 atomic commit ⑧标记完成 |
| **产出** | ①新增/修改的源码 ②新增的测试文件 ③一次 atomic git commit（message 解释 why）④任务状态更新为完成 |
| **之后接什么** | 循环 /build 下一个任务；阶段性收工前跑 /review；发布前 /ship |
| **注意** | **每次只推一个任务**。如果你喊 /build 而它吃了 3 个任务，打断它让它重来。小步快跑是纪律核心。 |
| **最小示例** | "按 tasks/todo.md 跑 /build"，Claude 拿 T01 → 写失败测试 → 实现 → 提交 → 问要不要继续 T02 |

---

### 2.4 `/test` — 单独跑 TDD

| 项 | 内容 |
|---|---|
| **什么时候用** | 两种场景：①给已有代码补测试 ②修 bug（Prove-It 模式：先复现再修） |
| **什么时候别用** | 你要的是新功能 + 测试 + 提交整套（用 /build）；你只想看现有测试是否通过（直接 `npm test`，不用 skill） |
| **前置条件** | 项目有测试框架已配置（pytest / vitest / jest 等） |
| **Claude 会干什么（新功能）** | 写**应该失败**的测试 → 实现最小代码 → 重构时保持绿 |
| **Claude 会干什么（修 bug / Prove-It）** | ①写复现 bug 的失败测试 ②**确认它真的失败** ③修代码 ④测试转绿 ⑤全套回归防引入新问题 |
| **浏览器场景** | 自动叠加 `browser-testing-with-devtools`，通过 Chrome DevTools MCP 直接看 console / DOM / network，不用你手工点页面 |
| **产出** | 新测试文件 + 最小必要的实现/修复代码；**不自动 commit**（对比 /build） |
| **之后接什么** | 满意了手动 commit，或走 /build 让它帮你 commit |
| **最小示例** | "这个日期解析函数闰年有 bug，用 /test 复现并修" |

---

### 2.5 `/review` — 五维评审

| 项 | 内容 |
|---|---|
| **什么时候用** | 刚完成一段较大改动（3-5 个 /build 之后）想体检；PR 合并前自查；别人写的代码交给你过一遍 |
| **什么时候别用** | 改动只有几行（不值当）；准备发布（用更重的 /ship，它并行跑 3 个 subagent） |
| **前置条件** | 有 staged changes 或最近几个 commit |
| **Claude 会干什么** | 对改动走**五维**：正确性（匹配 spec / 边界 case / 测试覆盖）、可读性（命名 / 逻辑 / 结构）、架构（风格一致 / 边界清晰 / 抽象层级）、安全（输入校验 / 密钥 / 认证）、性能（N+1 / 无界操作） |
| **产出** | **纯文本评审报告**（不落盘），每条带 `file:line` 和分级：Critical / Important / Suggestion |
| **之后接什么** | 按建议修 → 再 /review 或 /ship |
| **注意** | /review 的报告默认只在对话里返回，要存档就让 Claude"把评审存到 `reviews/YYYY-MM-DD.md`" |
| **最小示例** | "对最近 3 个 commit 做 /review" |

---

### 2.6 `/code-simplify` — 化简（不改行为）

| 项 | 内容 |
|---|---|
| **什么时候用** | 代码能跑但又长又嵌套；review 之后决定重构；接手别人/AI 写的脏代码 |
| **什么时候别用** | 行为有 bug（先 /test 修 bug）；没有测试覆盖（先补测试再简化，不然你不知道改坏没）；项目正要改接口（等接口稳定再简化） |
| **前置条件** | **目标代码必须有测试覆盖**（关键！没测试不让简化，否则你不知道行为改没改） |
| **Claude 会干什么** | ①读 CLAUDE.md 对齐项目风格 ②理解代码用途和调用方 ③按套路扫：深嵌套 → guard clauses / 长函数 → 拆分 / 嵌套三元 → if-else / 泛名 → 描述名 / 重复 → 抽公共 / 死代码 → 删 ④**每改一处就跑一次测试** ⑤测试挂了立刻回滚那一处 |
| **产出** | 行为不变的代码 diff；测试全绿；可选再跑 /review 确认 |
| **之后接什么** | commit 这次化简（单独一个 commit，不和功能改动混） |
| **最小示例** | "对 `src/parser.ts` 做 /code-simplify，注意保持测试绿" |

---

### 2.7 `/ship` — 发布前终检（最重）

| 项 | 内容 |
|---|---|
| **什么时候用** | 准备合入主干 / 准备部署 / PR 评审前终检。大改动必跑。 |
| **什么时候别用** | 改动 ≤ 2 个文件且 ≤ 50 行 **且** 不碰认证/支付/数据访问/配置（这种用 /review 够了）；还在开发中途 |
| **前置条件** | 改动已基本完成并通过本地测试 |
| **Claude 会干什么** | **并行**启 3 个独立 subagent（每个独立 context，不污染主对话）：<br>① `code-reviewer` 五维评审<br>② `security-auditor` 安全审计（OWASP / 密钥 / 认证 / CVE）<br>③ `test-engineer` 测试覆盖度分析<br>主 agent 收完三份报告后**合并**成决策文档 |
| **产出** | 一份固定模板的 **Ship Decision 文档**，含：<br>- GO / NO-GO 结论<br>- Blockers（必修）<br>- Recommended fixes（应修）<br>- Acknowledged risks（明知有风险仍发布的事项）<br>- **Rollback plan（触发条件 + 回滚步骤 + RTO）**<br>- 三份子 agent 原始报告 |
| **之后接什么** | 有 Critical → 修完再 /ship；没 Critical 或风险已接受 → 真正 push / deploy |
| **注意** | 只要任一 subagent 报 Critical，默认 NO-GO。Rollback plan 是 GO 前置条件，没 rollback 方案不给发。 |
| **最小示例** | 功能分支准备合并前："对当前分支跑 /ship" |

---

**为什么没有 `/commit` 命令**：commit 是 `/build` 第 7 步内置的（atomic commit，message 解释 why）。如果你想单独按规矩提交一堆手动改动，对 Claude 说"按 `agent-skills:git-workflow-and-versioning` 的规则提交当前改动"即可。

---

## 3. 直接回答你的 9 个问题

### Q1. 如何触发提交？

**有三种方式，任选**：

1. **隐式（推荐）**：用 `/build`，它每完成一个任务的最小切片，会自动按 atomic commit 标准提交。流程是 `Implement → Test → Verify → Commit → Next slice`。
2. **显式（半人工）**：直接对 Claude 说"按 `agent-skills:git-workflow-and-versioning` 的规则提交当前改动"。它会要求 commit 是原子的（一个 commit 干一件事，message 解释 why 不是 what）。
3. **完全人工**：自己 `git add` + `git commit`。

> 你这套项目（StockTrading）的 CLAUDE.md 全局铁律里有"未审阅的新增文件不要 git add -A"，所以用 `/build` 时建议保留一个二次确认环节，不要让它无脑 commit `.env` 之类。

### Q2. 如何触发测试？

**用 `/test`**。它走 TDD 的红绿循环：

- **新功能**：写一个**应该失败**的测试 → 实现最小代码让它通过 → 重构。
- **修 bug**（Prove-It 模式）：先写一个**能复现 bug 的失败用例** → 确认它真的失败 → 修代码 → 用例转绿 → 跑全套回归。

如果是浏览器相关 bug，`/test` 会自动叠加 `browser-testing-with-devtools` skill，让 Claude 通过 Chrome DevTools MCP 直接看 console、DOM、network——**这就是降低人工参与的关键**（见 Q4）。

### Q3. 如何解决测试的 bug？

**走 `agent-skills:debugging-and-error-recovery` 的 5 步**（直接对 Claude 说"用这个 skill"或者它会自动接管）：

```
1. Stop the line   停止加新功能，先修
2. Reproduce       让 bug 可稳定复现（不能复现就不能放心修）
3. Localize        定位到层（UI/API/DB/Build/外部服务/测试本身）
4. Fix             修根因，不修症状
5. Guard           加测试或检查防止它再来一次
```

它有一份"非确定性 bug 排查决策树"专门处理 timing-dependent / state-dependent / 真随机 三种难复现场景。

### Q4. 如何解决"动态自动测试"？

我猜你说的是**"让 Claude 自己跑测试、看结果、改代码，不用你手工点页面验证"**。三件套：

| 需求 | 用什么 |
|---|---|
| 让 Claude 自动跑单测/集成测试看结果 | `/test` 或 `/build`（内置跑测试 + 看输出 + 失败就自动调试） |
| 让 Claude 在浏览器里实时验证 UI | `agent-skills:browser-testing-with-devtools`（需要 Chrome DevTools MCP） |
| 让 push 后自动触发流水线 | `agent-skills:ci-cd-and-automation`（帮你写 GitHub Actions / 流水线 yaml） |

**对你 006 项目的具体建议**：你已经有 H5 等价回归 agent（`tools/equivalence_h5.py` + `h5-regression` subagent），这本身就是"动态自动测试"的本地实现。`agent-skills` 不替代它，而是**补两件事**：

1. 用 `agent-skills:test-driven-development` 写 backend 单测（你已有 backend pytest）和 frontend-uniapp 单测。
2. 用 `agent-skills:browser-testing-with-devtools` 让 Claude 自己开 5174 看 H5 控制台、不用你截图反馈。

### Q5. 它有什么用？是不是全用 slash 命令就行？

**用处**：把 Claude 从"快速堆代码"逼回"工程师式工作"。最大三个收益：

- 失败测试先行，避免 Claude 编出"看起来对其实没跑过"的代码。
- 评审走五维 + 安全 + 性能，避免"功能能跑就 ship"。
- 提交是原子的、有 why 的，便于回滚和 review。

**是不是全用 slash 就行？答：80% 够，但要补两个动作**：

1. 在阶段切换时用 slash（`/spec → /plan → /build → /test → /review → /ship`）。
2. 在阶段内部碰到具体问题，**叫 skill 名**比 slash 更准。例：碰 bug 说"用 debugging-and-error-recovery"；写 API 说"用 api-and-interface-design"；优化前端说"用 frontend-ui-engineering"。

完全不用 slash 也行——只要你提的需求里包含 skill 的"Use when"触发词，Claude 会自动激活。

### Q6. 是否有命令降低人为参与的测试部分？

**有，按降低程度排序**：

| 命令/skill | 你需要做什么 | 降低程度 |
|---|---|---|
| `/test` | 给一句需求，剩下 Claude 写测试+实现+跑 | 高 |
| `/build` | 给任务，Claude 走完 TDD + 提交 | 高 |
| `agent-skills:browser-testing-with-devtools` | 不用你手工点页面看效果 | 高（前提：装 Chrome DevTools MCP） |
| `agent-skills:ci-cd-and-automation` | 帮你写流水线 yaml，push 后自动跑 | 极高（一次性投入） |
| `/ship` | 三个子 agent 并行评审，你只看最后 GO/NO-GO | 极高 |

**真正能让你脱手的是 ci-cd-and-automation + browser-testing-with-devtools 两个**，前者是构建阶段自动化，后者是验证阶段自动化。`/test` 只能做到"自动写测试自动跑"，验证 UI 还得人看截图——除非用 DevTools MCP。

### Q7. 之后如果架构、idea 改变是要重开分支吗？

**多数情况不必**。规则是：

- **小到中等架构调整**（接口变、模块拆分、某层重写）：在当前分支 `/spec` 重新跑一次，把 SPEC.md 改了，然后 `/plan` 重生 tasks/plan.md，继续 `/build`。
- **整个产品方向变了**（你 007 那种 C# 重写）：开新分支，原分支留作历史基线。
- **trunk-based 原则**：分支寿命 1-3 天最佳，超过 1 周风险陡增（合并冲突、漂移）。

**结合你项目情况**：
- 006 → 007 的 C# 重写按你内存记录已决定**串行**（006 先完工再启 007），所以 006 当前分支保持不动直到完工，**不重开分支**。
- 如果 006 内部 idea 变了（比如 wot-design-uni 换成别的 UI 库），改 `specs/006-uniapp-multi-platform/spec.md` + `plan.md` + `tasks.md`，留在当前分支。

### Q8. 是否可以应用到已开发一半的项目上？

**完全可以**。你的 006 就是一个典型"开发到一半"的项目（spec/plan/tasks 已落，113 任务分 6 Phase，等 implement）。

**核心要点**：agent-skills **不要求你从零开始**。它的 7 个 slash 命令对应的是开发阶段，不是项目时间——你处在哪个阶段就从哪个命令切入。

### Q9. 已开发一半的项目用什么命令接上？

按你的 006 现状（spec/plan/tasks 已生成，未实现），**直接 `/build` 就行**。

但你项目已经在用 **speckit**（你看到的 `/speckit-specify /speckit-plan /speckit-tasks /speckit-implement` 那一套），它和 agent-skills 是**两套并行的工作流**：

| | speckit（你已有） | agent-skills（这一套） |
|---|---|---|
| 风格 | 规格驱动开发四阶段（Specify → Plan → Tasks → Implement），强结构化产物 | 21 份工程纪律工作流，可任意组合，更注重过程执行 |
| 产物位置 | `specs/006-.../{spec,plan,tasks}.md` | 项目根 `SPEC.md` + `tasks/plan.md` + `tasks/todo.md` |
| 强项 | 规格阶段的产物完整、可追溯 | 实现阶段的纪律执行（TDD、五维评审、并行 ship） |

**建议混用方案（针对你 006）**：

```
规格 + 计划阶段：继续用 speckit（你已经走完了）
   ↓
实现阶段：用 agent-skills 的 /build /test 配合 speckit 的 /speckit-implement
   ↓
评审 + 上线：用 agent-skills 的 /review /ship（speckit 这部分弱）
```

具体接入命令（按顺序）：

1. **不需要 `/spec`**——你已有 `specs/006-uniapp-multi-platform/spec.md`。
2. **不需要 `/plan`**——你已有 `tasks.md`（113 个任务）。
3. **直接开始 `/build`**：每次让 Claude 拿 tasks.md 里的下一个待办任务，按 incremental-implementation + TDD 推进。**注意**告诉 Claude "任务源是 `specs/006-uniapp-multi-platform/tasks.md`，不是默认的 `tasks/todo.md`"。
4. 阶段性调 `/review` 看代码质量。
5. 上线前调 `/ship`（会触发三个 subagent 并行评审）。

### Q10. 项目没有 SPEC.md，只有 docs/ 文件夹，还能 /build 吗？

**能，取决于 docs/ 里是什么**。先自己对一下：

| docs/ 里是什么 | 能否直接 /build | 接入方式 |
|---|---|---|
| 产品说明 / 需求文档 / 架构图 | ❌ 不能直接 | 让 Claude 读完 docs/ 后**口头生成 tasks**（"基于 docs/ 下的内容帮我拆任务到 tasks/todo.md"），再 /build |
| **ideate 一页纸（`docs/ideas/*.md`）+ `tasks/plan.md` + `tasks/todo.md`** | ✅ 可以 | 这就是 agent-skills 的**默认约定路径**，直接 /build |
| 已有的 tasks 清单 / TODO / 迭代计划（散落在 docs 里） | ✅ 可以 | 告诉 Claude"任务源是 `docs/xxx.md`"，并确保每条有验收标准 |
| API 文档 / 设计稿 / 技术调研 | ❌ 不能直接 | 这些是参考材料不是任务，先 /plan 让 Claude 结合它们拆任务 |

**实例：StockTradingAI（C# 版 AI Auto-Trader）**

项目结构：
```
docs/
├─ ideas/
│  └─ auto-trader-mvp.md        # ideate 一页纸（等同 SPEC）
└─ stage-a-runbook.md           # 运维手册
tasks/
├─ plan.md                      # 实施计划（Stage A-E）
├─ todo.md                      # 带 checkbox 的任务清单
└─ research-refs.md             # 调研清单
```

**完美对齐 agent-skills 默认约定**——`docs/ideas/` + `tasks/plan.md` + `tasks/todo.md` 正是 idea-refine → planning-and-task-breakdown → incremental-implementation 三个 skill 的标准产物路径。

接入命令：
```
# 最简：Claude 会自动在 tasks/todo.md 找下一个未勾选任务
/build

# 或明确指定任务
对 tasks/todo.md 里的 A.3（Sim 首次启动）跑 /build，
验收标准详见 tasks/plan.md 对应小节。
```

**不需要回头补 SPEC.md**——信息密度已经够 /build 干活。

**一个需要注意的纪律点**：你 todo.md 里可能有 Stage A/B 并行推进的设计，和 agent-skills"一次只 /build 一个任务"不冲突——连续 /build 几次推进 B 线再切回 A 线 OK，但**别一次让它干 A.3 + B.1 + B.2 三件事**，会破坏 atomic commit 和可回滚性。

---

## 4. 给你的速记小抄

**按"你手里有什么"定位**：
```
只有一句模糊念头        → ideate         产 docs/ideas/*.md 一页纸（skill，非 slash）
有想法没 SPEC           → /spec         产 SPEC.md
有 SPEC 没任务          → /plan         产 tasks/plan.md + tasks/todo.md
有任务等实现            → /build        产 代码+测试+commit（日常主力）
只想补测试/修 bug       → /test         产 测试+少量代码改动（不自动 commit）
改动完想体检            → /review       产 评审报告（纯文本）
能跑但代码脏            → /code-simplify 产 化简后的 diff（行为不变）
准备合主干/部署         → /ship         产 GO/NO-GO + rollback 方案
```

**按"关键信号词"定位**：
```
"我有个想法 / 这靠谱吗"     → 说 "ideate ……" 或 "用 idea-refine"
"重写 / 换库 / 新方向"      → /spec
"该先做啥"                  → /plan
"做 T070 / 实现这个任务"    → /build
"加个测试 / bug 复现不了"   → /test
"合并前看看"                → /review
"这段代码太乱"              → /code-simplify
"可以发了吗"                → /ship
"bug 排查"（非测试相关）    → 说 "用 debugging-and-error-recovery"
"忘了哪个 skill 合适"       → 说 "看 using-agent-skills 的 flowchart"
```

---

## 5. 21 份 skill 全列表（可叫名字唤起）

按开发阶段分组，每个 skill 在 system prompt 里都注册了名字 `agent-skills:<name>`：

**Define（定义）**
- `idea-refine` 想法发散收敛
- `spec-driven-development` 写规格

**Plan（计划）**
- `planning-and-task-breakdown` 拆任务

**Build（实现）**
- `incremental-implementation` 增量切片
- `test-driven-development` TDD
- `context-engineering` 上下文工程
- `source-driven-development` 官方文档驱动
- `frontend-ui-engineering` 前端
- `api-and-interface-design` API 设计

**Verify（验证）**
- `browser-testing-with-devtools` 浏览器实时验证
- `debugging-and-error-recovery` 调试排错

**Review（评审）**
- `code-review-and-quality` 五维评审
- `code-simplification` 化简
- `security-and-hardening` 安全加固
- `performance-optimization` 性能

**Ship（上线）**
- `git-workflow-and-versioning` git 流程
- `ci-cd-and-automation` CI/CD
- `deprecation-and-migration` 废弃迁移
- `documentation-and-adrs` 文档与 ADR
- `shipping-and-launch` 发布

**Meta**
- `using-agent-skills` 元 skill（每次 session 自动注入）

---

## 6. 3 个 subagent（独立子人格）

通过 Agent 工具 `subagent_type` 调用：

| subagent_type 值 | 职责 | 何时用 |
|---|---|---|
| `agent-skills:code-reviewer` | 五维代码评审 | 大改动合并前 |
| `agent-skills:security-auditor` | 安全审计（OWASP/secrets/auth/CVE） | 涉及认证、外部输入、密钥时 |
| `agent-skills:test-engineer` | 测试覆盖度分析 | 担心测试有漏 |

它们的特点：**独立 context window，不会污染你主对话**。`/ship` 命令会一口气并发起这三个，最后合并报告——这是它最有价值的一面。

---

## 7. 给你 006 项目的一个具体示范

假设你现在要实现 `tasks.md` 里"T070 微信小程序仪表盘只读屏"：

```
你说：
  按 specs/006-uniapp-multi-platform/tasks.md 里的 T070，用 /build 推进。
  注意：H5 已实现是基线，wot-design-uni 重写。

Claude 会自动：
  1. 读 T070 的验收条件
  2. 写一个失败的单测（描述 T070 的预期行为）
  3. 写最小实现让测试通过
  4. 跑全套回归（你的 backend pytest 201 passed 不能掉）
  5. 跑 build 确认编译过
  6. atomic commit："feat(frontend-uniapp): T070 微信小程序仪表盘只读"
  7. 标记 T070 完成，问你要不要继续 T071

中间任何一步失败，自动切到 debugging-and-error-recovery。
```

---

## 8. 最后两件事容易忽略

1. **每次新会话开头花 5 秒确认 jq 是否在 PATH**。没有 jq，session-start hook 不工作，你就拿不到自动注入的 flowchart——skill 还能用，但 Claude 自动选 skill 的能力会下降。

2. **Skills 是工作流，不是知识库**。它们没用来"问问题查资料"——那是 references/ 目录下 4 份 checklist 的事（testing-patterns / security-checklist / performance-checklist / accessibility-checklist）。需要查"测试到底该怎么写"的时候，叫 Claude "看 references/testing-patterns.md"。
