# Spec-Kit 使用教程

> 本文档基于 `F:\AIProject\spec-kit` 本地源码整理，重点讲解：
>
> 1. 整体工作流（7 个斜杠命令）
> 2. 如何触发测试
> 3. 如何解决测试的 bug
> 4. 如何解决"动态自动测试"
> 5. **实现完成后发现 bug 该用什么命令解决**
>
> 日期：2026/05/03

---

## 一、整体工作流（7 个 Slash Command）

Spec-Kit 是"规格驱动开发（Spec-Driven Development, SDD）"工具，核心是一组 `/speckit.*` 斜杠命令，驱动 AI coding agent 完成开发。

| 阶段 | 命令 | 作用 |
|------|------|------|
| 1 | `/speckit.constitution` | 建立项目宪章（原则、规范、是否强制 TDD） |
| 2 | `/speckit.specify` | 描述做什么、为什么（不关心技术栈） |
| 3 | `/speckit.clarify` | 消除规格中的歧义 |
| 4 | `/speckit.plan` | 确定技术栈、架构 |
| 5 | `/speckit.tasks` | 生成可执行的任务清单 `tasks.md` |
| 6 | `/speckit.analyze` | （可选）审查 spec/plan/tasks 一致性 |
| 7 | `/speckit.implement` | 按 `tasks.md` 执行实现 |

> Codex CLI 在 skills 模式下使用 `$speckit-*` 前缀，其他 agent 一般是 `/speckit.*`。

典型最小流程：

```bash
/speckit.constitution 本项目采用 TDD，所有功能必须先写失败测试再实现
/speckit.specify      描述你要做什么
/speckit.clarify      消除歧义
/speckit.plan         技术栈/架构
/speckit.tasks        生成任务清单
/speckit.analyze      审一致性（可选）
/speckit.implement    执行实现
```

---

## 二、如何触发测试

Spec-Kit 中的"测试"分两层，务必区分。

### 1. 让 spec-kit 为你的项目生成测试（最常见需求）

**关键点：测试默认是可选的（OPTIONAL）**，来自 `templates/tasks-template.md`：

> "Tests are OPTIONAL - only include them if explicitly requested in the feature specification."

**三种触发方式**：

**方式 A：在 constitution 中强制 TDD（推荐）**

```
/speckit.constitution 本项目严格采用 TDD。所有功能必须先写失败测试再实现。
```

**方式 B：在 specify 或 tasks 调用时显式要求**

```
/speckit.specify ... 请为每个用户故事生成契约测试和集成测试。
/speckit.tasks 请生成测试任务（contract tests + integration tests）。
```

**方式 C：用 `/speckit.checklist` 生成测试检查清单**

```
/speckit.checklist test
```

触发后，`tasks.md` 会在每个 User Story 的 Phase 里出现类似：

```
- [ ] T010 [P] [US1] Contract test for /api/foo in tests/contract/test_foo.py
- [ ] T011 [P] [US1] Integration test for user journey in tests/integration/test_journey.py
```

运行 `/speckit.implement` 时，会按 `templates/commands/implement.md` 第 7 步严格按 TDD 顺序执行：

> "Tests before code: If you need to write tests for contracts, entities, and integration scenarios"
> "Follow TDD approach: Execute test tasks before their corresponding implementation tasks"

### 2. 跑 spec-kit 本身的测试（贡献代码时用）

位于 `F:\AIProject\spec-kit\tests\`，参考 `CONTRIBUTING.md`：

```bash
cd F:\AIProject\spec-kit
uv sync --extra test
.venv\Scripts\Activate.ps1

# 跑全部测试
uv run pytest -q

# 跑关键一致性测试（改过 agent/wiring 后必跑）
uv run python -m pytest tests/test_agent_config_consistency.py -q
```

---

## 三、如何解决测试的 Bug

Spec-Kit 没有内置 debug 命令，通过下列工作流解决测试失败。

### 流程 A：`/speckit.implement` 执行中测试失败

`implement.md` 第 8 条定义了错误处理规则：

- **Halt on failure**：非并行任务失败就停止
- **Parallel [P] tasks**：失败的单独报告，其他继续
- **Clear error messages with context**：提供调试信息
- **Suggest next steps**：给出下一步建议

你应该做的：

1. 查看 agent 输出的具体失败任务 ID 和错误
2. **不要直接让 agent "fix it"**，先问根因（root cause）
3. 如果根因在 spec 或 plan 中有歧义 → 回到 `/speckit.clarify` 或修改 `plan.md`
4. 如果仅是实现错误 → 让 agent 针对失败任务重跑，完成后在 `tasks.md` 把该 task 打 `[X]`

### 流程 B：测试本身有 bug（测试写错了）

1. 用 `/speckit.analyze` 做只读审查，它会标出：
   - 契约与实现不一致（Inconsistency）
   - 覆盖缺口（Coverage Gaps）
   - 歧义（Ambiguity）
2. 根据报告决定改 spec 还是改测试
3. 若是规格缺陷 → `/speckit.clarify` → 重新生成 tasks 中的测试任务

### 流程 C：借助外部 skill（强烈推荐）

本地可用的调试类 skill：

- `agent-skills:debugging-and-error-recovery`：系统性根因调试
- `agent-skills:test-driven-development`：Prove-It 模式（先写暴露 bug 的失败测试）
- `agent-skills:test`：TDD 红-绿-重构

推荐组合：**测试失败 → `debugging-and-error-recovery` 定位 → `test` 固化回归**。

---

## 四、如何解决"动态自动测试"

"动态自动测试" = 每次改动都自动重跑测试、自动验证回归。Spec-Kit 本身不内置 runner/watcher，但提供了下列机制：

### 方案 1：`.specify/extensions.yml` 钩子（原生）

`templates/commands/implement.md` 的 Pre-Execution Checks 和第 10 步，会读取项目根的 `.specify/extensions.yml`，查找：

- `hooks.before_implement`
- `hooks.after_implement`

同样支持 `before_tasks` / `after_tasks` / `before_analyze` / `after_analyze`。

在 `.specify/extensions.yml` 里注册一个自动跑测试的命令，就能在每次 `/speckit.implement` 完成后自动执行（如 `pytest -q` / `npm test`）。这是 spec-kit 原生的"动态自动测试"机制。

最小示例：

```yaml
hooks:
  after_implement:
    - extension: auto-test
      command: run-tests
      description: 每次实现完成后自动跑单元测试
      prompt: 请运行 `pytest -q` 并汇报失败用例
      enabled: true
      optional: false
```

### 方案 2：Claude Code Hooks（harness 级）

在 `settings.json` 注册 `PostToolUse` hook，每次 Edit/Write 后自动跑测试，比 extensions.yml 更通用。相应 skill：`update-config`。

### 方案 3：`/loop` 定时监控

```
/loop 5m pytest -q
```

每 5 分钟跑一次，适合长跑测试或 watch 场景。

### 方案 4：CI/CD 挂载

用 `agent-skills:ci-cd-and-automation` 把测试接入 GitHub Actions，作为上游自动关卡。

---

## 五、**实现完成后发现 Bug，该用什么命令解决？**

**直接答案：Spec-Kit 没有专门的 "bug fix" 命令。**

官方做法是**把 bug 当作一次新的、小型的规格变更**，按 bug 严重程度分三档处理。

### 档位一：小 bug / 逻辑错误（推荐 90% 场景使用）

直接让 AI agent 基于 `/speckit.analyze` 报告或你的描述修复，然后手动跑测试验证。

```bash
# 1. 先让 agent 审查现状（只读）
/speckit.analyze

# 2. 直接描述 bug，让 agent 定位并修复
看看 tasks.md 里 T023 实现的登录接口，在密码错误时返回了 500 而不是 401，
请定位 src/services/auth_service.py 并修复，同时补一个回归测试。

# 3. 修完后手动验证
pytest tests/ -q
```

注意事项：

- 让 agent 修完后，**不要信它的"已修复"汇报**，自己跑测试确认。
- 修复后对应 task（如 T023）在 `tasks.md` 中保持 `[X]`，无需重跑 `/speckit.implement`。

### 档位二：中等 bug / 涉及多任务或规格不清

用 `/speckit.clarify` 澄清 → 重新生成 tasks → 局部 `/speckit.implement`。

```bash
# 1. 回到规格层澄清歧义
/speckit.clarify 登录失败错误码：密码错误→401，账号锁定→423，请补充到 spec.md

# 2. 重新生成任务（增量追加修复任务，如 T045/T046）
/speckit.tasks 请只针对新澄清的错误码规则生成修复任务

# 3. 审一遍一致性
/speckit.analyze

# 4. 只执行新任务
/speckit.implement 只执行 T045-T046
```

### 档位三：严重 bug / 架构级问题

回到 `/speckit.plan` 调整方案，重走后半段流程。

```bash
/speckit.plan 调整：JWT 签名算法从 HS256 改为 RS256，因为密钥泄露风险
/speckit.tasks
/speckit.analyze
/speckit.implement
```

### 黄金路径：结合本地 skill（强烈推荐）

本地已装的几个 skill 比 spec-kit 原生更适合 debug，优先用：

| 场景 | 用哪个 | 做什么 |
|------|--------|--------|
| 定位 bug 根因 | `agent-skills:debugging-and-error-recovery` | 系统性根因分析，避免猜测式修改 |
| 写回归测试 + 修复 | `agent-skills:test-driven-development`（Prove-It 模式）| 先写能复现 bug 的失败测试再修复，保证不再回归 |
| 单纯跑测试流程 | `agent-skills:test` | 红-绿-重构 |

**推荐组合流程**：

```
1. agent-skills:debugging-and-error-recovery  —— 定位根因
2. agent-skills:test-driven-development       —— 写失败测试复现 bug
3. 让 agent 修复，直到测试转绿
4. （可选）/speckit.analyze                    —— 审查是否引入新的不一致
5. 把修复纳入 git commit，tasks.md 中该 task 保持 [X]
```

这样**Prove-It 模式**强制先写出能暴露 bug 的测试，修复后该测试自然成为永久回归测试，避免同一个 bug 反复出现。

### 哪些做法千万别用

- ❌ **不要直接重跑 `/speckit.implement`**：它只会按 tasks.md 的未完成项执行，不会重新审视已完成的任务，对 bug 修复无效。
- ❌ **不要直接跟 agent 说 "fix it"**：没有根因分析和回归测试的修复，大概率引入新 bug。

### 一句话速查表

| Bug 严重度 | 用什么命令 |
|-----------|-----------|
| 小 bug / 逻辑错 | 直接描述让 agent 改 + `pytest` 验证 |
| 中等 bug | `/speckit.clarify` → `/speckit.tasks` → `/speckit.analyze` → `/speckit.implement` |
| 架构级 bug | `/speckit.plan` → `/speckit.tasks` → `/speckit.analyze` → `/speckit.implement` |
| 所有档位 | 搭配 `agent-skills:debugging-and-error-recovery` + `test-driven-development` |

---

## 六、关键文件速查

| 想做什么 | 看这里 |
|---------|-------|
| 整体快速上手 | `docs/quickstart.md` |
| 本地改 CLI | `docs/local-development.md` |
| 提 PR / 跑测试 | `CONTRIBUTING.md` |
| 命令定义 | `templates/commands/*.md`（每个 `/speckit.*` 对应一个） |
| 任务模板 | `templates/tasks-template.md` |
| 脚本 | `scripts/bash/*.sh`、`scripts/powershell/*.ps1` |
| spec-kit 自身测试 | `tests/test_*.py` |
| 扩展参考 | `docs/reference/extensions.md` |
| 工作流参考 | `docs/reference/workflows.md` |

---

## 七、推荐的"含测试"完整流程

```bash
/speckit.constitution 严格 TDD，所有 P1 故事必须有契约测试和集成测试
/speckit.specify <描述需求>
/speckit.clarify
/speckit.plan <技术栈>
/speckit.tasks 请包含测试任务
/speckit.analyze        # 审一致性
/speckit.implement      # 测试先行，自动按 TDD 执行
```

在 `.specify/extensions.yml` 添加 `after_implement` 钩子，自动跑完整测试套件，完成"动态自动测试"闭环。

---

## 八、常见坑 & 提示

1. **tasks.md 里 test 任务没出现** → 说明你没在 constitution / specify / tasks 中明确要求测试，测试默认不生成。
2. **切换功能用 git 分支** → spec-kit 自动按当前分支名（如 `001-feature-name`）识别当前 feature 目录。
3. **复杂项目分阶段实现** → MVP（仅 User Story 1）先做完验证，再加下一阶段，避免 agent 上下文爆掉。
4. **Bug 修复不重跑 `/speckit.implement`** → 它对已完成任务无动作，靠直接让 agent 改或走"澄清-重生成-局部实现"闭环。
5. **密钥安全** → 生成 `.env` 等文件前先确认 `.gitignore` 覆盖；`extensions.yml` 里不要写明文密钥。
