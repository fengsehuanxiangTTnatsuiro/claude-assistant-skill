# 通用路径花名册（roster）

本文件是 `/assistant` **通用路径**（无专属项目团队的工作）的角色→agent 映射，并且是**组队判定与收敛上限的唯一定义源**（SKILL.md 与 supervision.md 只引用、不重述）。身份与死规则在 [SKILL.md](SKILL.md)，监督操作在 [supervision.md](supervision.md)。

## 角色 → agent 映射

| 角色 | 主力 agent | 备选 / 语言专精 | 注入方法论（superpowers / skill，非 agent） |
|---|---|---|---|
| 需求 → 计划 | `ecc:planner` | `ecc:code-explorer`（先摸代码） | brainstorming → writing-plans |
| 架构 | `ecc:architect` | `ecc:code-architect`（贴合现有代码出蓝图） | — |
| 功能写码（主力） | `ecc:tdd-guide` | `claude`（琐碎改动）/ `ecc:gan-generator`（spec 驱动带评测对抗） | tdd-workflow |
| 代码评审 | `ecc:code-reviewer` + 语言专精 reviewer（`ecc:typescript-reviewer` 等） | `ecc:security-reviewer`（敏感场景） | code-review |
| 构建修复 | `ecc:build-error-resolver` | `ecc:<lang>-build-resolver` | systematic-debugging |
| 测试 / E2E | `ecc:e2e-runner` / `ecc:tdd-guide` | `webapp-testing`（浏览器 UI） | verify |
| 重构 / 清理 | `ecc:refactor-cleaner` + `ecc:code-simplifier` | — | — |
| 安全审查 | `ecc:security-reviewer` | — | security-review |
| 文档同步 | `ecc:doc-updater` | — | — |
| 兜底（无匹配） | `claude`（通用） | — | — |

## 用户 team / 自定义 agent 注册表（开源扩展入口）

本节只做**路由登记**，不重定义 SKILL.md 的分诊规则。用户显式点名以下 team / agent 时，优先走这里；未点名时才走上方通用角色→agent 映射。

### A. 用户自定义 team

| 用户说法 / team 名 | leader / 入口 | 适用范围 | 路由说明 |
|---|---|---|---|
| `<team-name>` | `<team-leader-or-skill-entry>` | `<场景>` | 用户说“用/交给/调用 <team-name>”时使用；assistant 只对接 leader。 |

### B. 用户自定义 agent / 角色覆盖

| 用户说法 / 角色 | 实际 `subagent_type` 或入口 | 适用范围 | 路由说明 |
|---|---|---|---|
| `<role-or-agent-name>` | `<actual-agent-type>` | `<场景>` | 用户点名该角色时使用；不存在则停报,不静默回落。 |

**显式点名优先级:** 用户自定义 team / agent > 项目 team skill > 通用 ecc roster。点名对象解析失败 → 停下报告 `requested_team_or_agent_not_found`，不许把名字写进普通 prompt 假装已路由。

## 兵器库与场景映射（"用尽一切工具"的可执行版——别只当口号）

SKILL.md 说"调度一切环境资产武装自己",但**空泛的鼓励产生不了行为**。助理之所以总退回"裸 subagent 串行",正是因为没有"什么场景→点名哪个工具"的硬触发。本节把它写实。**先开工前自检一次:这活有没有现成工具/skill 比手搓更快?有→点名用。**

### A. Claude Code 原生并行/编排手段（官方已验证,优先用）

| 手段 | 怎么调 | 什么场景点名用它 |
|---|---|---|
| **subagents** | 连续发多个 spawn(点名渠道,见下) | **通用并行主力**。N 个独立子任务→并行派 N 个(克隆 N 个 VM、配 N 个节点)。这是最常用、最该先想到的。 |
| **dynamic workflows** (`/workflows`) | `/workflows` 或让 Claude 写一个 | 活大到一回合协调不过来:全代码库审计、几百文件迁移、需交叉核验的批量操作。比手动派一堆 subagent 更稳。 |
| **agent teams** | 需 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`,默认关 | worker 之间需互相通信/共享任务列表的复杂写码项目。本 skill 写码项目档用的就是它。 |
| **`/batch`** | `/batch <大改动>` | **仅写码批量改**:拆成 5–30 个 worktree 隔离 subagent 各开 PR。**运维活不产 PR,别硬套**(此前误推荐,纠正)。 |
| **agent view** (`claude agents`) | 终端 dashboard,研究预览 | 几个独立后台会话,想一屏盯状态、按需介入。 |
| **worktrees** | `--worktree` | 多个并行会话改同一 repo 时隔离文件,防写冲突。并行写码必备。 |
| **background bash** | 后台跑一条 shell | 跑一条不阻塞会话的长命令(不 spawn agent)。长 virt-clone 之类可后台挂。 |

**检查在跑的并行活:** subagents→`/agents` 的 Running 页;后台→`/tasks`;workflows→`/workflows`;后台会话→`claude agents`。

### B. 官方内置能力（60+ 命令 + 5 捆绑 skill）

输入 `/` 即列全部。**助理开工前若不确定有没有现成命令覆盖本活,先 `/` 扫一眼再决定手搓还是调现成。** 内置 skill 与同名 command 冲突时 skill 优先。常用面向本助理:`/plan`(大改动前规划)、`/code-review`(评审/找 bug)、`/simplify`(清理)、`/init`(项目记忆)、`/permissions`(信任边界)。

### C. 环境特有兵器（你装的,我无法预先列——助理实机自查后填）

ecc/superpowers 插件里的 skill、你接的 MCP server、CodeGraph 等,**只有实机环境知道确切清单**。预先硬列必造幻觉条目(参见 `/batch` 教训)。所以:**助理首次唤起(或清单可能变动时),先跑一次自查,把真实可用兵器填进下方占位区**,之后按填好的清单点名用。

**自查指令(派 Explore agent 跑,回结构化清单,别自己 cat):**
> 扫以下来源,列出当前实际可用的非内置兵器,按"名称 | 类型(skill/MCP/subagent/command) | 一句话能干啥 | 适配场景"汇总:① `~/.claude/skills/` 和各插件 `*/skills/` 下的 SKILL.md;② `~/.claude/plugins/` 已装插件(ecc/superpowers 等)的 agents/ 与 commands/;③ 已连接的 MCP server(`/mcp` 或 config);④ CodeGraph 等其他已装工具的入口。只列真实存在的,不臆测。

**占位区(自查后填,这是活清单,随环境更新):**
```
# 我的环境特有兵器(YYYY-MM-DD 自查)
# ecc skill:  <填>
# superpowers: tdd-workflow / systematic-debugging / brainstorming / ...(roster 已引用部分)
# MCP server:  <填,如 github / 数据库 / 监控>
# CodeGraph:   <填入口与用途>
# 其他:        <填>
```

> 铁律:**点名用具体工具前,该工具必须在 A/B(官方已验证)或 C(实机自查确认)里真实存在。** 不确定存不存在→先 `/` 或自查,不凭记忆硬调,更不凭记忆写进汇报。

## 派活落地规约（spawn 怎么真正派到 ecc agent —— 最易漏的一环）

**问题背景（机制现实，已勘误）**：上表选了"该用谁"，但**怎么把这个 ecc agent 真正 spawn 出来**才是关键。本节曾在错误前提上立规并导致大面积退化，现已按当前 harness 实情纠正——**先说清机制，再说规则**。

**当前 harness 的真实机制（实测，非推测）**：
- 本 harness 的 `Agent` 工具 `subagent_type` 参数**直接登记了全部 ecc 插件 agent**（`ecc:go-reviewer` / `ecc:typescript-reviewer` / `ecc:tdd-guide` / `ecc:planner` / `ecc:security-reviewer` …，见工具 schema 的 enum）。`Agent(subagent_type="ecc:tdd-guide", …)` 是**合法且首选**的派法，会**正确路由**到那个 ecc agent，不会报 not found、不会退回 general-purpose。
- ecc agent 的存在性也已核对：`~/.claude/plugins/marketplaces/ecc/agents/*.md` 真实存在，每个带 `name` / `tools` / `model` frontmatter，是注册过的插件 subagent。

**勘误（防再次踩坑——本节曾错在哪）**：旧版断言"`Task()` 只认 3 个内置 subagent，传 `ecc:*` 会报 not found 并悄悄退回裸 general-purpose"。这条**在当前 harness 下不成立**——它把某个旧版/别的 harness 的限制当成了跨环境长期现实。更糟的是它的"纠正动作"反了：禁止走 `subagent_type` → 助理只能填 `general-purpose` → 再把 `@ecc:X` 写进 prompt 正文当"点名" → **但 @-mention 写在 Agent 工具的 prompt 参数里没有路由力**，它只是给那个已经 spawn 成 general-purpose 的 agent 看的一段普通文字。结果：100% 退化成裸 general-purpose。这正是实践中易出现任务全部退化为 general-purpose 的根因——不是偶发手滑，是错规则导致的必然。

**铁律 0（落地铁律，凌驾于"选谁"之上）：派 ecc 具名 agent，必须把它的具名 `ecc:*` 写进 `subagent_type`，由 `subagent_type` 决定"谁来跑"。**

正确派法（按确定性排序）：
1. **`subagent_type: "ecc:<name>"`（首选，唯一真正决定路由的方式）**：`Agent(subagent_type="ecc:go-reviewer", prompt=…)`。这会把活路由到那个 ecc reviewer，带它自己的 tools / model / 方法论。
2. 自然语言点名（prompt 里写「Use the ecc:X subagent to …」）**仅在不直接指定 `subagent_type` 的入口**（如纯聊天让某 agent 接手）才有意义；**不要靠在 prompt 正文里写 `@ecc:X` 来路由**——Agent 工具 prompt 字符串里的 @ 没有路由力。

**自检（每次派写码/评审类活，spawn 前过一遍）**：
- ✅ `subagent_type` 是不是填了某个具名 `ecc:*`？ → 是 → 放行。
- ❌ `subagent_type` 填的是 `general-purpose`，而 prompt 正文里却写 `@ecc:X` / "use the ecc:X subagent"？ → **这正是退化信号**：名字被写在没路由力的地方，实际跑的是裸 general-purpose。停手，把 `ecc:X` 挪进 `subagent_type`。违规信号词：spawn 时 `subagent_type` 出现 `general-purpose`、而名字只出现在 prompt 正文里。
- 🔒 **派前声明闸（防"对用户说 ecc、spawn 却 general-purpose"——具名 agent 路由失效的典型根因）**：凡对用户承诺过"用 ecc:X 写码/评审"，spawn 前必须点出**具体哪个 `ecc:*` 进 `subagent_type`**，且与承诺一致（承诺 `ecc:tdd-guide` → `subagent_type` 必须是 `ecc:tdd-guide`）。该 ecc:X 点名不出 → 停下报告（见下"拿不到真 ecc agent 时的处置"），**不许静默降级 general-purpose**。对用户说了 ecc、spawn 却填 general-purpose = 承诺与执行不符的硬违规。

**拿不到真 ecc agent 时的处置（按性质分流，默认从严）：**
- **写码/改功能/评审类** → 若 `subagent_type="ecc:X"` 真的报 not found（插件未装/未启用）→ **停下报告**（用 SKILL.md 受阻醒目块：「ecc:X 不可用，需你确认插件状态或换路线」）。**不许凑合裸 `general-purpose` 顶替** —— 裸 general-purpose 没有 ecc 的 test-first / 评审方法论，顶替 = 假装组了队，正是要防的失败模式①④。
- **只读调查类**（查故障/翻代码/调研）→ 允许用内置 `explore` / `general-purpose`（这类本就不要求 ecc 专精），但仍优先 `ecc:code-explorer` / `ecc:planner` 若 `subagent_type` 可填。
- **降级注入（仅在你明确允许时启用，默认关）**：若某 ecc agent 确实点名不出、又必须推进，可退而用裸 subagent + 把该 ecc agent 的定义文件（`插件 agents/ 目录下的 .md`）内容当 system prompt 注入。**这是有损降级**（丢工具隔离、丢 type safety），必须在汇报里单列标注「降级注入了 ecc:X」，不可默默做。

**层 2 · 委派 PM / 团队 leader（仅限 assistant 自组的 team / 通用路径独立 PM）** — 铁律0 升级为**全链路路由优先**：层1=直接派（上方"正确派法""自检"），层2=本节。助理只对接 PM/team-leader，**移交时必须显式带一条绑定指令**传给对方：

> "组队优先走路由（[roster.md](roster.md)）：每个角色先看花名册有没有对应 `ecc:*`——有就把 `subagent_type` 填那个 ecc agent（写码→`ecc:tdd-guide`、评审→`ecc:code-reviewer`+语言专精、构建修复→`ecc:build-error-resolver`/`ecc:<lang>-build-resolver`、架构→`ecc:architect`…）；**只有** roster.md 无对应角色的项目专有岗（无 roster.md 对应的领域专有岗）才用兜底 agent。不许在 roster.md 有对应角色处绕开 ecc、用裸 `general-purpose` 顶替。"

PM/team-leader 若仍 spawn 裸 `general-purpose` 做写码/评审 = 层2违规，等同于助理自己违规。**验真（防 PM 挂名 ecc、实填 general-purpose）**：PM 首次回报必须附"每个角色实际 spawn 的 subagent_type 原文"（role→`ecc:X`），助理比对绑定指令、不符即驳回；团队真实性审计（supervision §1）核对每个 member 的 `agentType`（实际路由），**不信 `name`**（display 名可随意起）。违例驳回。**这是常见的双重失败模式的完整闭环**：层1=助理口头"派 ecc"却 spawn general-purpose（具名 agent 路由失效）；层2=对自组 team 的 PM 说"用 ecc"却不带绑定指令、PM 默认 general-purpose。两层都要堵。

> 注（路由订正）：**用户显式点名的项目团队不受本铁律约束**——用户点名 = 显式选择该 team 的自有 agent 体系（该 team 自有的项目专有 role card），按各 team skill 规定组队，**助理不对其强加 ecc 路由**。本铁律只管：① 助理直接派（层1）；② 通用路径下助理自组的 team / 独立 PM（层2）。

## 派活要点

- **写码主力**：`ecc:tdd-guide`（test-first 实现）。写码类的组队/评审闸门见下方「组队判定」；**琐碎一次性改动**（不改变既定行为的机械修复，详见组队判定边界条）可派单 code agent（通用 `claude`）。
- **架构**：`ecc:architect`（通用系统设计，无 K8s/云原生绑定）。
- **评审**：`ecc:code-reviewer` + 对应语言 reviewer（语言专精，比泛型强）—— 这是 ecc 最该用上的地方。
- **构建挂了**：`ecc:build-error-resolver` 或 `ecc:<lang>-build-resolver`（专为最小 diff 修构建）。
- **UX/创意/集成**（ecc 无完全对应）：UX 走 `ecc:frontend-design-direction` skill + `webapp-testing`；创意走 brainstorming；集成走 `ecc:tdd-guide` + `ecc:refactor-cleaner`。
- **superpowers 是方法论，不是 agent**：在对应阶段注入（bug→systematic-debugging，新功能→tdd-workflow，创意前→brainstorming），**不占角色**（组队判据是任务性质不是角色计数，方法论注入不改变写码/只读的定性）。

### 任务简报质量准则（派活高频病灶，与"选谁"同等重要）

默认契约（背景/交付物/验收线/边界，实测已 76-83% 齐备）不再赘述；**只钉四项真实短板**。派任何活，spawn 前 prompt 过一遍这 4 条 checklist：

1. **开头一句身份红线（防越界跑偏）**：prompt **第一句**钉"你是谁 + 你不做什么"。模板：`你是一名 X 专家，本任务是 Y；你【只读不改 / 不写测试 / 只调查不实现】。` 给最易踩的禁区用 `NEVER`/`MUST` 加权（如"只读调查 NEVER 改码"）。
   - **1b. ecc:* 具名派活别复述身份、补本次禁区**（审计发现：派 `ecc:go-reviewer` / `ecc:tdd-guide` 等 ecc 具名 agent 时，prompt 普遍只写"你是 ecc:X，审 Y"——复述了 `subagent_type` 已自带、frontmatter 已定义的常驻身份，却漏了**本次任务特有红线**）。ecc agent 的 frontmatter 已自带身份/工具/通用行为，prompt **不要再复述身份句**；要补的是 frontmatter 没有的**本次专属禁区**——如"本次只看回归安全，不看风格/性能""本次只验证修复不扩散、不重构"。把身份补充位用在刀刃上，不浪费在重复通用身份。
2. **防误判显式化（别只靠只读红线兜底）**：除只读禁令外，**显式写**——"不确定就停下来问/上报，不要猜；删除/覆盖前先核对目标与描述一致。" 调查类另加配对禁令：**禁止把助理的砍/留/对错判断预嵌进 prompt**（写"判断该不该砍 X""给模块打 KEEP/CUT"= 让 agent 在助理画的框里成像）→ 强制两步：① agent 只做客观描述、禁判断词；② 判断由助理基于事实自己做。
3. **非写码任务按场景选 agent（补痛点7/诊断4，别用裸 general-purpose 顶替）**：外部仓库/方案调研 → `general-purpose`；本地代码摸底 → `ecc:code-explorer`/`Explore`；外部库文档 → `ecc:docs-lookup`；运维批量操作 → 数据并行铺开（见下"并行铺开"）。**写码/评审类必须 ecc:\* 具名**（铁律0），不许裸 general-purpose 顶替。**四类场景都没专用 agent 时（ecc 空白，见下第 5 条）**：动态定制一个 agent 级 prompt 装进载体，不要简单交代甩 general-purpose。
4. **凭据卫生（硬规则，安全问题）**：**收到用户提供的明文凭据（密码/token/key），NEVER 直接写进派给子 agent 的 prompt 正文。** 转为引用：export 进会话环境变量、或写权限 600 的 secret 文件、或优先建议用户配 SSH key + ssh-agent 免密；prompt 里只引用 `$VAR`/`--password-file`/`ssh -i`。检测到明文凭据时，主动提醒用户该凭据已暴露、建议 rotate。
5. **动态定制 agent（ecc 无对应时的兜底，适用任意场景）**：第 3 条四类场景里若**没有专用 agent**（运维深层、特殊领域调研等 ecc 空白场景），派活前先查有没有现成专用 agent（`ecc:*`）/ skill / MCP：
   - **有** → 点名用（走铁律0）。
   - **没有** → **不要简单交代甩给裸 general-purpose**。用 `general-purpose` / `Explore` 当**载体**，把 prompt 写成 **ecc agent `.md` 级密度**：身份红线 + 可用工具约束 + 行为规范 + 输出契约 + 防误判 + 验收（复用一流 prompt 范式）。这等价于为该任务临时造一个专业 agent，**当前会话即时生效、不碰「不新建 agent」红线**（没新文件、没新 subagent_type）。
   - **技术约束（必读，否则走弯路）**：Claude Code 的 agent 是 `.md` 文件，**会话启动时**扫描注册成 `subagent_type`；运行时新写 `.md` 当前会话**不保证热加载**，调不动。所以当前会话一律走"载体 + agent 级 prompt"，**不要**尝试当场写 `.md` + 立即 `subagent_type` 调用。仅当某定制 prompt **高频复用**时，才沉淀成持久 `.md` 到 `~/.claude/agents/`（**下次会话**生效——这是"新建 agent"的合理例外，用下次会话规避热加载）。
   - **成本边界**：简单活（查状态、跑现成命令）别上 agent 级 prompt（overkill）；复杂活 / 无专用 agent 的活才上。分诊决定。

## 组队判定（一张表 + 五条铁律）

**判定轴：先性质（改不改产线代码），后规模（角色数/并行度）。不数 agent。**

| 任务性质 | 规模 | 派活方式 | team-lead | 评审闸门 |
|---|---|---|---|---|
| **只读调查**（查故障/翻代码/读config/调研/审计/spike/摸实现） | 任意 | 单 agent 或多只读 fan-out（`Explore` 原生搜索 / `ecc:code-explorer` 摸自己代码 / `ecc:docs-lookup` 查外部库文档），主会话收子 agent 的完整分析交付物（综合类；简单事实仍可一句结论，见 SKILL 第-1步 交付物完整性） | 无 team | 无 |
| **写码/改功能**（改产线代码行为、需测试） | 小（≤2-3角色、单/双文件、依赖清晰） | writer + reviewer/tester；助理可轻量直管收口 | 助理 | 写+审不可减 |
| **复杂目标写码**（新功能/新模块/长任务/多模块/迁移集成/PR 级交付/需频繁协调） | 大（≥3角色或跨多模块/多阶段） | **leader-led team**：优先 native team；不可用则 managed-team leader | team leader / PM | leader 内组，写+审不可减 |
| **用户显式指定 team** | 任意 | 路由到用户指定 team 的 leader/入口；不走通用 roster | 指定 team leader | 按该 team 自有规则 + 证据闸门 |
| **琐碎机械修复**（typo/错常量/单行bug/import/删死码/格式；不改行为、≤数行、无需新测试） | — | 派单 code agent（`claude`） | 无 | 无 |
| **治理修正**（改 .md 协议/规则/skill；非产线代码） | 改法已明确（审计给出行+措辞） | 助理**直接改+自检**（grep/Read） | 无 | 自检 |
| **治理修正** | 改动大/拿不准 | 走 team（写+审） | 助理或PM | 写+审 |

**五条铁律**（表的脚注，不可违背）：
1. **评审闸门不可削减**：写码类最小安全形态 = 写手(tdd-guide) + 审(code-reviewer/语言专精)。小任务可由助理轻量直管这对 writer/reviewer；复杂目标任务必须升级为 leader-led team,不能只靠一对 agent 扛完整交付。
2. **team ≠ 全并行；flat fan-out ≠ team**：team 价值是 leader 统一拆解、分工、闸门和收口，非无脑全开。并行只在依赖图允许处（独立模块并行写、多reviewer并行审）；有依赖按时序串行（评审/build-fix/测试依赖代码先存在）。助理直接派 planner/coder/reviewer/tester 并自己汇总 = flat fan-out,不是 team。
3. **只读调查不许顺手改码**：调查 agent 撞到"得改X文件"必须停手、回结构化结论给助理（根因/要改哪些/几个模块），自身永不越界写码。助理重判走哪行表。严禁孤狼改码。
4. **助理直管 writer/reviewer 时三纪律**（堵 micromanage 后门）：① 靠心跳盯，不亲自催；② 忽略 idle/重复方案噪声，不当裁判；③ 只收 verdict + 原始证据，不亲自判细节。**升级信号**：消息淹没主会话/任务跨多阶段 → 立即派 team leader/PM 收口。
5. **收敛上限**：`fix_round ≥ 3` 不收敛 → 升级（换人/换方案/请示用户），不空转。

**复用花名册**：不新建 agent，全用表中 ecc:* / claude。

**并行铺开（与组队判定同等重要,现有最缺的一维）：** 选完"用谁"后,必须判**子任务能否并行**。判据是依赖图,不是任务大小:
- **互不依赖的子任务 → 必须并行铺开**(一次性派出全部,不串行等待)。两类并行都要会:①**数据并行**——同种操作作用于 N 个对象(克隆 N 个 VM/配 N 个节点/改 N 个同类文件),并行派 N 个 agent 各管一个;②**角色分工并行**——独立模块并行写、多 reviewer 并行审。
- **有依赖的 → 按依赖时序**,但同一层内无依赖的部分仍并行。
- **机制:** Claude Code 异步执行——连续发多个 spawn、不等返回即并行;或用 `/batch` 让其自动拆分并行。独立任务并行省 60–80% 墙钟。
- **结构性强制（堵串行伪并行，不只靠自觉）**：N 个互不依赖的子任务必须写在**同一个 spawn 批次**（一条消息里多个 Agent tool_use block 同时发），不许拆成多轮串行；汇报附 spawn 时序证据（派活后 X 秒内几个 running），可被核对。
- **反面:** N 个独立子任务串行排队派 = 严重低效,违背助理职责。运维批量活尤其要警惕(它们天然高度可并行)。

**harness 注记（native team 与 managed-team）：** 若 Claude Code 原生 agent team 可用,复杂目标写码优先创建/调用 native team。若原生 team 不可用或有"一个 lead 只管一个 team+team 删不掉"等摩擦,允许退到 **managed-team**：助理派一个 team leader/PM,由 leader 再协调 worker agents,助理只和 leader 对接。子 agent 级并行(同时派多个独立 Task)仍可用于只读/批量独立对象,但它不是复杂写码 team 的替代品。

**禁止因 harness 摩擦把大活降级成裸并行 subagent（已知反模式："从来不创建 PM、不拉 agent team"）：** 真属复杂目标写码（新功能/新模块/长任务/多模块/迁移集成/PR 级交付/需频繁协调）→ **必须上 native team 或 managed-team leader**，不得降级成一堆由助理扁平直管的并行 subagent 了事——裸并行**缺统一 owner、缺协调、缺收口**，正是用户诟病的"不拉 team"。摩擦是已知成本，不是降级理由。**反之**：轻活/只读调查/单文件机械修复仍按轻重自适应不组队（别过度纠正成"什么都组队"，那违背任务分诊总纲）。判据始终是组队判定表：按任务性质 + 规模定，不按"组队麻烦不麻烦"定。

**leader 首次回执（轻量，不上重型 handshake）：** 进入 leader-led team 后,leader 第一次回报只需给最小事实清单：`team/leader/members(role + actual agent_type)/assistant_contact_surface=leader_only/下一步计划`。这是审计线索,不是十字段 YAML 仪式；后续以 supervision.md 的真实 team 审计和原始证据为准。
