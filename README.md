# /assistant

> A triage-first Claude Code skill that turns one-line engineering goals into the right execution shape: one agent, a reviewed pair, parallel agents, or a real leader-led agent team.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-blue)](https://docs.claude.com/en/docs/claude-code)

`/assistant` is an orchestration skill for Claude Code.

It does not blindly throw every request into a heavy team workflow, and it does not let a complex feature collapse into one unsupervised agent. It first triages the task, then dispatches the right execution shape.

```text
simple one-off task        -> one suitable agent
small verified code change -> writer + reviewer/tester
independent batch work     -> parallel agents
complex feature/module     -> leader-led agent team
explicit user team         -> that team's leader/entrypoint
```

The main session stays a thin orchestrator: it aligns the request, routes work, supervises evidence, and reports back. Product code, tests, builds, reviews, and heavy investigation are delegated.

---

## No more prompt engineering for every subtask

With `/assistant`, the user does not need to hand-craft long specialist prompts for every agent.

You describe the goal in plain language:

```text
/assistant implement password reset, including API, DB, frontend, tests, and PR summary
```

`/assistant` turns that goal into precise task briefs for the right execution shape:

- a solo investigation agent;
- a writer + reviewer/tester pair;
- parallel workers;
- a native agent team;
- or a managed team led by a team leader.

Those task briefs include:

- role and responsibility;
- task scope and boundaries;
- allowed tools and forbidden actions;
- acceptance criteria;
- testing and review expectations;
- evidence requirements;
- stop conditions for ambiguity, blockers, or irreversible actions.

The user writes the goal. `/assistant` writes the working prompts, dispatches the work, supervises evidence, and reports back.

---

## Why this exists

Raw coding-agent workflows tend to fail in predictable ways:

- the main session reads too much code and pollutes its own context;
- a named specialist is mentioned in a prompt, but never actually routed;
- a complex feature is handled by one generic agent with no owner, reviewer, or tester;
- several agents are spawned flatly, but no one owns the final outcome;
- “done” is reported without raw diff, test output, or review evidence;
- users have their own agents or teams, but the assistant ignores them and falls back to a generic roster.

`/assistant` fixes these by making dispatch shape explicit. It is strict where structure matters, but lightweight when the task is simple.

---

## Core principle: triage first

Every request starts with a lightweight classification.

| Task type | Dispatch shape | What happens |
|---|---|---|
| Simple one-off task | Solo agent | One focused agent handles the task. |
| Small code change needing verification | Writer + reviewer/tester | The assistant may lightly coordinate a writer and a reviewer/tester. |
| Independent batch/ops work | Flat parallel agents | The assistant dispatches independent agents in parallel and summarizes results. |
| Complex feature / module / migration | Leader-led team | A team leader plans, coordinates members, collects evidence, and reports back. |
| User explicitly names a team | Explicit team route | The assistant routes to that team’s leader/entrypoint instead of the default roster. |

The rule is:

> **Simple work must stay simple. Complex goal-oriented work must not degrade into one solo agent or assistant-managed flat fan-out.**

---

## Dispatch shapes

### 1. Solo agent

Used for bounded one-off work:

```text
/assistant explain this error
/assistant inspect this config and tell me why it fails
/assistant check where this function is defined
```

Shape:

```text
user -> assistant -> one suitable agent -> assistant -> user
```

---

### 2. Writer + reviewer/tester

Used for small code changes where self-certification would be unsafe, but a full team would be overkill:

```text
/assistant fix this login parameter validation bug and add a regression test
```

Shape:

```text
user
  -> assistant
     -> writer
     -> reviewer/tester
  -> assistant
-> user
```

This is intentionally not a full team. The assistant may lightly coordinate the writer and reviewer, but it should not micromanage their work. It collects verdicts and raw evidence.

---

### 3. Flat parallel agents

Used when the subtasks are independent:

```text
/assistant check these 8 nodes and report which ones have broken Docker networking
/assistant inspect these 12 files and summarize duplicate patterns
```

Shape:

```text
user
  -> assistant
     -> agent for item 1
     -> agent for item 2
     -> agent for item 3
     -> ...
  -> assistant summarizes
-> user
```

Flat fan-out is correct for independent work. It is **not** a replacement for a real team when the work needs shared planning, sequencing, review, and final ownership.

---

### 4. Leader-led team

Used for real engineering goals:

- new feature;
- new module;
- multi-layer implementation;
- migration;
- integration;
- PR-level delivery;
- long-running goal;
- work needing planning + implementation + testing + review under one owner.

Example:

```text
/assistant implement password reset, including API, DB changes, frontend flow, tests, and PR summary
```

Correct shape:

```text
user
  -> assistant
     -> team leader / PM
        -> planner
        -> implementer
        -> reviewer
        -> tester
        -> docs agent
     -> team leader reports evidence
  -> assistant reports to user
```

In this mode, the assistant talks to the **team leader**, not every worker. The leader owns planning, member assignment, coordination, evidence collection, and final reporting.

Incorrect shape:

```text
assistant -> planner
assistant -> implementer
assistant -> reviewer
assistant -> tester
assistant summarizes everything itself
```

That is flat fan-out, not a team.

If native agent teams are available, `/assistant` should use them. If native teams are not available or are awkward in the current harness, `/assistant` may use a **managed-team** fallback: it dispatches a team leader/PM, and that leader coordinates worker agents. The assistant still talks only to the leader.

The leader’s first response should be lightweight, not bureaucratic:

```text
team: <team name or managed-team>
leader: <leader name / entrypoint>
members:
  - role: planner, actual_agent_type: ...
  - role: implementer, actual_agent_type: ...
  - role: reviewer, actual_agent_type: ...
assistant_contact_surface: leader_only
next_plan: <short plan>
```

This is an audit clue, not a heavy manifest protocol.

---

### 5. Explicit user team

Users can name their own project team:

```text
/assistant use backend-team to implement the billing module
/assistant 交给 深度研究 team，分析这个项目架构
```

When a team is explicitly named, user intent wins:

```text
user custom team / agent > project team skill > default ECC roster
```

The assistant should resolve the team from `roster.md`, route to its leader or entrypoint, and stop if the team cannot be found. It must not silently fall back to the default roster or pretend that a prompt mention is real routing.

---

## Custom agents and teams

`roster.md` is the routing source of truth.

It contains:

- the default role -> agent mapping;
- a user custom team registry;
- a user custom agent / role override registry;
- routing rules for real `subagent_type` dispatch;
- the team-vs-flat-fan-out boundary.

### Recommended: ask `/assistant` to register routes for you

The easiest way to extend this skill is to let `/assistant` update `roster.md`.

#### Register a custom agent

```text
/assistant 注册一个自定义 agent 路由：
用户叫法：qa-reviewer / QA评审
实际入口：custom:qa-reviewer
适用范围：测试质量、验收用例、回归风险评审
路由说明：用户点名 QA评审 时使用；找不到则停报，不静默回落
```

Expected behavior:

1. `/assistant` checks whether `custom:qa-reviewer` really exists in the current Claude Code harness.
2. If it exists, it adds an entry to the custom agent registry in `roster.md`.
3. If it does not exist, it stops with `requested_team_or_agent_not_found` instead of writing a fake route.

The resulting `roster.md` entry should look like:

```markdown
| `qa-reviewer` / `QA评审` | `custom:qa-reviewer` | 测试质量、验收用例、回归风险评审 | 用户点名该角色时使用；不存在则停报，不静默回落。 |
```

#### Register a custom team

```text
/assistant 注册一个自定义 team：
team 名：backend-team
leader / 入口：backend-team-leader
适用范围：后端新功能、API、DB、服务端重构
路由说明：用户说“用 backend-team / 交给 backend-team”时使用；assistant 只对接 leader
```

Expected behavior:

1. `/assistant` verifies that `backend-team-leader` is a real team entrypoint or routable leader.
2. If it exists, it adds the route to `roster.md`.
3. If it does not exist, it stops instead of routing to a generic agent.

The resulting `roster.md` entry should look like:

```markdown
| `backend-team` | `backend-team-leader` | 后端新功能、API、DB、服务端重构 | 用户说“用/交给/调用 backend-team”时使用；assistant 只对接 leader。 |
```

After registration:

```text
/assistant 用 backend-team 实现 billing 模块
```

should route to `backend-team-leader`, not the default ECC roster.

### Manual route registration

You can also edit `roster.md` directly.

Custom team table:

```markdown
| 用户说法 / team 名 | leader / 入口 | 适用范围 | 路由说明 |
|---|---|---|---|
| `backend-team` | `backend-team-leader` | 后端新功能、API、DB、服务端重构 | 用户说“用/交给/调用 backend-team”时使用；assistant 只对接 leader。 |
```

Custom agent table:

```markdown
| 用户说法 / 角色 | 实际 `subagent_type` 或入口 | 适用范围 | 路由说明 |
|---|---|---|---|
| `qa-reviewer` / `QA评审` | `custom:qa-reviewer` | 测试质量、验收用例、回归风险评审 | 用户点名该角色时使用；不存在则停报，不静默回落。 |
```

Manual editing is supported, but using `/assistant` is recommended because it can check whether the referenced entrypoint actually exists before writing the route.

---

## Quick start

### 1. Install prerequisites

The default roster is designed around ECC-style agents and superpowers-style methodologies. The orchestration model itself is agent-pool agnostic: if your agent names are different, update `roster.md`.

### 2. Install the skill

```bash
mkdir -p ~/.claude/skills/assistant
cp SKILL.md roster.md supervision.md ~/.claude/skills/assistant/
```

Start a new Claude Code session so the skill can be loaded.

### 3. Use it

Simple task:

```text
/assistant explain this error
```

Small verified fix:

```text
/assistant fix this bug and add a regression test
```

Complex feature:

```text
/assistant implement password reset, including API, DB, frontend, tests, and PR summary
```

Explicit team:

```text
/assistant 用 backend-team 实现 billing 模块
```

Register custom route:

```text
/assistant 注册一个自定义 agent 路由：
用户叫法：qa-reviewer
实际入口：custom:qa-reviewer
适用范围：测试质量、验收用例、回归风险评审
```

Full handoff:

```text
/assistant 全程决策移交 — implement the notification module and prepare the PR summary
```

Full handoff reduces mid-flight interruptions. It does not remove safeguards for irreversible actions such as push, merge, deploy, publish, or permanent deletion.

---

## Evidence and supervision

`/assistant` does not treat “done” as enough.

For coding work, completion should be backed by:

- changed files;
- diff summary or raw diff;
- test/build command and raw output;
- reviewer verdict;
- unresolved risks;
- external integration evidence when mocks are not enough.

For leader-led team work, the assistant checks that the contact surface is really leader-only and that the reported workers are real routed agents, not decorative names.

---

## Repository structure

| File | Purpose |
|---|---|
| `SKILL.md` | Defines assistant identity, boundaries, autonomy levels, task triage, and decision flow. |
| `roster.md` | Defines role routing, custom team / agent registration, dispatch shape, routing laws, and team boundaries. |
| `supervision.md` | Defines team audits, heartbeat monitoring, evidence gates, blocker handling, and failure-mode defenses. |
| `LICENSE` | MIT license. |

---

## What `/assistant` should not do

- It should not write product code in the main session.
- It should not write tests or run build/test commands in the main session.
- It should not treat every request as a team task.
- It should not downgrade complex feature work into one solo agent.
- It should not call flat fan-out a team.
- It should not ignore a user-specified team.
- It should not route by prompt mention when a real `subagent_type` or team entrypoint is required.
- It should not register custom agents or teams that do not actually exist.
- It should not report completion without raw evidence.

---

## FAQ

### Does every request create an agent team?

No. `/assistant` is triage-first. Simple tasks stay lightweight. Only complex feature/module/migration work or explicitly named teams require leader-led team mode.

### What is the difference between flat parallel agents and a team?

Flat parallel agents are independent workers dispatched directly by the assistant. This is useful for batch checks.

A team has a leader. The leader plans, assigns work, coordinates dependencies, collects evidence, and reports back. Complex engineering work needs this ownership layer.

### Can I use my own agents?

Yes. Register them in `roster.md`, preferably by asking `/assistant` to do it so it can verify that the actual entrypoint exists.

### What happens if I name a team or agent that is not registered?

The assistant should stop with `requested_team_or_agent_not_found`. It must not silently fall back to the default roster.

### Does this require ECC?

The default roster is written for ECC-style agents, but the dispatch model is not ECC-specific. Replace the role mappings and custom registries with your own agent pool.

---

# 中文说明

`/assistant` 是一个 Claude Code skill。它的目标不是让一个 agent 从头干到尾，也不是把所有请求都强行拉成一个复杂 team，而是让 Claude Code 主会话变成一个**轻量编排者**：

```text
用户只描述目标
  -> /assistant 先判断任务类型
  -> 选择合适的派活形态
  -> 把实现、测试、评审、调查交给对应 agent / team
  -> assistant 审查证据并汇报结果
```

---

## 用户不用再写复杂提示词

使用 `/assistant` 后，用户不需要再为每个 agent 手写长 prompt。

用户只需要说清楚目标：

```text
/assistant 实现 password reset 模块，包括 API、DB、前端、测试和 PR 说明
```

剩下的由 `/assistant` 负责：

- 判断任务类型；
- 选择单 agent、writer + reviewer/tester、flat fan-out、native team 或 managed-team；
- 给 agent 或 team leader 写专业任务书；
- 写清楚角色、边界、可用工具、禁止事项；
- 写清楚验收标准、测试要求、review 要点；
- 处理不确定、受阻、不可逆动作等停下条件；
- 收集 diff、测试输出、review 结论等证据；
- 最后用人话汇报结果。

也就是说，使用方式从：

```text
用户自己写一堆复杂 agent prompt
```

变成：

```text
用户说目标
  -> assistant 写任务书
  -> agent / team 执行
  -> assistant 验收证据并汇报
```

这也是 `/assistant` 的核心价值之一：把多 agent 协作里的提示词工程成本，从用户身上转移到 assistant 身上。


它解决的是工程协作里的几个常见问题：

- 简单问题被过度流程化，浪费时间；
- 复杂功能被单个 agent 直接干完，没有 owner、没有 review、没有测试证据；
- 表面上派了多个 agents，但其实是 assistant 自己扁平管理，不是真正的 team；
- 用户明明有自己的 agents/team，assistant 却不知道怎么路由；
- 名字只写在 prompt 里，实际没有进入真正的 `subagent_type` 或 team 入口；
- “完成”没有 diff、测试输出、review 结论等证据。

---

## 中文核心逻辑：先分诊，再派活

`/assistant` 每次被调用时，第一件事不是立刻开工，而是先判断任务属于哪一类。

```text
简单一次性任务
  -> 单 agent

小修小补，但需要测试 / review
  -> writer + reviewer/tester

N 个互不依赖对象
  -> 并行 agents

新功能 / 新模块 / 长任务 / 多模块开发 / 迁移集成
  -> leader-led agent team

用户显式指定 team
  -> 指定 team 的 leader / 入口
```

一句话原则：

> **简单任务不要上重流程；复杂目标任务不能退化成单 agent。**

---

## 1. 简单任务：单 agent

适合：

- 看一段报错；
- 查一个配置；
- 解释一个日志；
- 找某个函数在哪里；
- 做一次只读调查；
- 机械性小修改。

示例：

```text
/assistant 看看这个报错是什么原因
```

执行形态：

```text
用户 -> assistant -> 单个合适 agent -> assistant -> 用户
```

这种任务不需要 team，也不需要 leader。

---

## 2. 小写码任务：writer + reviewer/tester

适合：

- 修一个小 bug；
- 改一个小接口；
- 补一个回归测试；
- 修改单个功能点；
- 影响范围较小，但不能让写代码的 agent 自己证明自己。

示例：

```text
/assistant 修复登录参数校验 bug，并补一个回归测试
```

执行形态：

```text
用户
  -> assistant
     -> writer
     -> reviewer/tester
  -> assistant 收证据
-> 用户
```

这里仍然不是完整 team。assistant 可以轻量直管 writer/reviewer，但要遵守三点：

1. 不亲自写代码；
2. 不亲自跑测试；
3. 只收 verdict、diff、测试输出、review 结论，不陷入细节 micromanage。

---

## 3. 批量独立任务：flat fan-out

适合：

- 检查 N 台机器；
- 查看 N 个配置文件；
- 对 N 个目录做只读分析；
- 对多个互不依赖对象做同类检查。

示例：

```text
/assistant 检查这 8 个节点，看看哪些 Docker 网络有问题
```

执行形态：

```text
assistant -> node-1 checker
assistant -> node-2 checker
assistant -> node-3 checker
...
assistant 汇总结果
```

这种叫 **flat fan-out**，也就是扁平并行派发。它适合批量独立任务。

但它不是 team。

---

## 4. 复杂目标任务：leader-led team

复杂任务包括：

- 新功能；
- 新模块；
- 多模块改造；
- 前后端联动；
- DB + API + UI 一起改；
- 迁移；
- 集成；
- PR 级交付；
- 长时间任务；
- 需要计划、实现、测试、评审、文档一起收口的任务。

示例：

```text
/assistant 实现 password reset 模块，包括 API、DB、前端流程、测试和 PR 说明
```

正确形态：

```text
用户
  -> assistant
     -> team leader / PM
        -> planner
        -> implementer
        -> reviewer
        -> tester
        -> docs agent
     -> leader 汇总证据
  -> assistant 汇报用户
```

关键点：

- assistant 只对接 team leader；
- team leader 负责拆任务、派成员、盯进度、收证据；
- planner / coder / reviewer / tester 由 leader 管；
- assistant 不直接逐个指挥 team members；
- 完成时必须有证据：改了什么、测试怎么跑、review 是否通过、还有哪些风险。

错误形态：

```text
assistant -> planner
assistant -> coder
assistant -> reviewer
assistant -> tester
assistant 自己汇总
```

这只是扁平派发，不是真正的 team。

---

## native team 和 managed-team

如果 Claude Code 环境支持原生 agent team，就优先走 native team：

```text
assistant -> native agent team leader -> team members
```

如果当前 harness 不方便直接创建 native team，也可以退到 managed-team：

```text
assistant -> managed-team leader / PM -> worker agents
```

managed-team 的重点不是“少一个功能”，而是仍然保持 leader-led：

- assistant 只对接 leader；
- leader 负责协调 worker；
- worker 不直接由 assistant 扁平管理；
- leader 统一收口证据。

---

## leader 第一次回报应该包含什么

进入 team 模式后，不需要复杂 YAML，也不需要重型 manifest。leader 第一次只需要给一个轻量回执：

```text
team: <team 名或 managed-team>
leader: <leader 名 / 入口>
members:
  - role: planner, actual_agent_type: ...
  - role: implementer, actual_agent_type: ...
  - role: reviewer, actual_agent_type: ...
assistant_contact_surface: leader_only
next_plan: <简短计划>
```

这不是形式主义，而是为了确认两件事：

1. 这次真的有 leader；
2. assistant 的接触面确实是 leader-only，而不是扁平派 worker。

---

## 用户指定 team

用户可以直接点名自己的 team：

```text
/assistant 用 backend-team 实现 billing 模块
```

或者：

```text
/assistant 交给 深度研究 team，分析这个项目架构
```

优先级是：

```text
用户自定义 team / agent > 项目 team skill > 默认 ECC roster
```

也就是说，只要用户显式点名，assistant 就应该优先走用户指定的 team，而不是自己重新用通用 roster 派 `ecc:tdd-guide`、`ecc:code-reviewer` 等。

如果找不到指定 team，assistant 应该停报：

```text
requested_team_or_agent_not_found
```

不能静默回落到普通 agent，也不能把 team 名写进 prompt 假装已经路由。

---

## 推荐：直接让 `/assistant` 注册你的 agents 和 team

用户不需要手动改 `roster.md`。推荐直接让 `/assistant` 自己添加路由。

因为改 `roster.md` 是治理/规则维护，不是产品代码。assistant 可以做，但必须先验证入口真实存在。

---

### 注册自定义 agent

示例：

```text
/assistant 注册一个自定义 agent 路由：
用户叫法：qa-reviewer / QA评审
实际入口：custom:qa-reviewer
适用范围：测试质量、验收用例、回归风险评审
路由说明：用户点名 QA评审 时使用；找不到则停报，不静默回落
```

assistant 应该做：

1. 检查 `custom:qa-reviewer` 是否是当前环境里真实存在的 agent / `subagent_type`；
2. 存在就写入 `roster.md` 的「用户自定义 agent / 角色覆盖」表；
3. 不存在就停报，不写假路由。

写入后的表项类似：

```markdown
| `qa-reviewer` / `QA评审` | `custom:qa-reviewer` | 测试质量、验收用例、回归风险评审 | 用户点名该角色时使用；不存在则停报，不静默回落。 |
```

以后就可以这样用：

```text
/assistant 让 QA评审 检查这次改动的回归风险
```

---

### 注册自定义 team

示例：

```text
/assistant 注册一个自定义 team：
team 名：backend-team
leader / 入口：backend-team-leader
适用范围：后端新功能、API、DB、服务端重构
路由说明：用户说“用 backend-team / 交给 backend-team”时使用；assistant 只对接 leader
```

assistant 应该做：

1. 检查 `backend-team-leader` 是否真实存在；
2. 存在就写入 `roster.md` 的「用户自定义 team」表；
3. 不存在就停报；
4. 后续用户点名 `backend-team` 时，优先路由到该 team。

写入后的表项类似：

```markdown
| `backend-team` | `backend-team-leader` | 后端新功能、API、DB、服务端重构 | 用户说“用/交给/调用 backend-team”时使用；assistant 只对接 leader。 |
```

以后就可以这样用：

```text
/assistant 用 backend-team 实现订单结算模块
```

此时 assistant 应该走：

```text
assistant -> backend-team-leader -> backend-team members
```

而不是：

```text
assistant -> 默认 ecc:tdd-guide
```

---

## 为什么推荐让 assistant 自己注册

手动改 `roster.md` 也可以，但推荐让 `/assistant` 加，原因是：

1. **它可以先自查入口是否存在**  
   避免把不存在的 agent/team 写进路由表。

2. **它能保持表格格式一致**  
   减少 Markdown 表格写坏、行插错位置的问题。

3. **它能遵守 fail-closed**  
   找不到就停报，不会伪装成已路由。

4. **它能把“用户说法”和“真实入口”分开**  
   用户可以叫 `QA评审`，真实入口可以是 `custom:qa-reviewer`。

5. **它符合 assistant 的定位**  
   用户只需要说“我要注册这个 agent/team”，具体路由维护交给 assistant。

---

## 手动注册也支持

如果你想手动改，可以直接编辑 `roster.md`。

自定义 team 表：

```markdown
| 用户说法 / team 名 | leader / 入口 | 适用范围 | 路由说明 |
|---|---|---|---|
| `backend-team` | `backend-team-leader` | 后端新功能、API、DB、服务端重构 | 用户说“用/交给/调用 backend-team”时使用；assistant 只对接 leader。 |
```

自定义 agent 表：

```markdown
| 用户说法 / 角色 | 实际 `subagent_type` 或入口 | 适用范围 | 路由说明 |
|---|---|---|---|
| `qa-reviewer` / `QA评审` | `custom:qa-reviewer` | 测试质量、验收用例、回归风险评审 | 用户点名该角色时使用；不存在则停报，不静默回落。 |
```

注意：手动注册时也要确保真实入口存在。否则 assistant 后续点名时应该停报 `requested_team_or_agent_not_found`。

---

## 安装

```bash
mkdir -p ~/.claude/skills/assistant
cp SKILL.md roster.md supervision.md ~/.claude/skills/assistant/
```

建议重新打开一个 Claude Code 会话，让 skill 重新加载。

---

## 使用示例

简单任务：

```text
/assistant 看看这个 Kubernetes 报错是什么意思
```

小修小补：

```text
/assistant 修复登录接口参数校验 bug，并补测试
```

批量检查：

```text
/assistant 检查这 8 台节点的 Docker 网络状态
```

复杂新功能：

```text
/assistant 实现 password reset 模块，包括 API、DB、前端、测试和 PR 说明
```

指定 team：

```text
/assistant 用 backend-team 实现 billing 模块
```

注册自定义 agent：

```text
/assistant 注册一个自定义 agent 路由：
用户叫法：安全评审
实际入口：custom:security-reviewer
适用范围：鉴权、权限、敏感信息、依赖风险
```

注册自定义 team：

```text
/assistant 注册一个自定义 team：
team 名：infra-team
leader / 入口：infra-team-leader
适用范围：部署、Kubernetes、CI/CD、基础设施改造
```

全程移交：

```text
/assistant 全程决策移交 — 实现通知模块，并准备 PR 说明
```

全程移交只减少中途打断，不取消红线。涉及 push、merge、deploy、发布、永久删除数据等不可逆动作时，仍然需要停下请示。

---

## `/assistant` 不应该做什么

- 不应该在主会话里亲自写产品代码；
- 不应该在主会话里亲自写测试；
- 不应该在主会话里亲自跑 build/test；
- 不应该把所有任务都强制拉 team；
- 不应该把复杂新功能降级成一个 solo agent；
- 不应该把 flat fan-out 冒充成 team；
- 不应该忽略用户显式指定的 team；
- 不应该把 agent/team 名字只写进 prompt 假装路由；
- 不应该注册不存在的 agent/team；
- 不应该没有证据就汇报完成。

---

## License

MIT.
