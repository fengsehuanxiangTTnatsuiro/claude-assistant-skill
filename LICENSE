# /assistant — A Delegating Orchestrator Skill for Claude Code

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill that turns Claude
into a **delegating orchestrator** — it aligns on requirements, spawns and supervises a
team of specialist subagents, and reports back, **without writing code itself**.

> English first; a Chinese mirror follows below. The skill body (`SKILL.md`,
> `roster.md`, `supervision.md`) is written in Chinese — Claude executes it correctly
> in either language.

---

## What it is

When you invoke `/assistant`, Claude stops acting as a hands-on engineer and starts
acting as **your proxy / chief of staff**. It does four things and only four things:

1. **Align** on the requirement and acceptance criteria (asks only on genuine intent
   ambiguity).
2. **Delegate** — assembles a team of specialist subagents and dispatches work,
   parallelizing independent subtasks.
3. **Supervise** — heartbeat monitoring, team-authenticity audits, evidence review.
4. **Report** — digests team output into a plain-language summary for you.

The whole point is to prevent the "lone wolf" failure mode where an assistant quietly
does everything itself and skips review.

## The constitution (four death rules)

These are immutable — the skill may even evolve its own non-constitutional parts, but
never these:

1. **Never write or edit product source code.** All production code is produced by
   specialist development subagents.
2. **Never write/edit test code, never run build/test.** Execution belongs to test
   agents; the assistant's "verification" means *reviewing the evidence* they produce.
3. **Never self-escalate privileges** (self-evolution is allowed; touching the death
   rules is not).
4. **Verify by evidence review only** — every "done/passed" claim must come with raw
   evidence (a diff, raw test output), which the assistant audits.

## Two autonomy tiers

- **Default** — align-if-understood, then act; stops only on true intent ambiguity or
  irreversible/external actions.
- **Full hand-off** — triggered by phrases like "全程决策移交" / "100% over to you";
  drives to completion without mid-flight check-ins. Irreversible/external red lines
  (pushing shared branches, merging main, deploying, deleting data, publishing) halt in
  **both** tiers.

## File structure

| File | Role |
|---|---|
| `SKILL.md` | Identity, the four death rules, autonomy tiers, the decision flow run on every invocation. The constitution lives here. |
| `roster.md` | The single source of truth for role→agent mapping, team-formation rules, and how to actually route work to a named subagent. |
| `supervision.md` | The supervision & governance playbook: heartbeats, team-authenticity audits, the evidence-review gate, escalation. |

## Install

This is a personal Claude Code skill. Drop the three files into a skill directory Claude
Code loads, e.g.:

```bash
mkdir -p ~/.claude/skills/assistant
cp SKILL.md roster.md supervision.md ~/.claude/skills/assistant/
```

Then invoke it in a session:

```
/assistant <your requirement>
```

(Exact load path depends on your Claude Code setup — see the Claude Code skills docs.)

## Customizing for your environment

The roster references an `ecc:*` agent pool and `superpowers` methodologies, plus two
**example** project-team paths (`bili-chat-team`, `bke-team`). These are specific to the
author's environment. Before using:

- Replace the `ecc:*` agent names with whatever subagents your environment actually
  registers (the roster includes a self-audit prompt for discovering them).
- Remove or rename the project-team references if they don't apply to you.

## License

[MIT](LICENSE).

---

<a name="chinese"></a>

# /assistant — 用户的代理人（Claude Code 编排型 skill）

一个 [Claude Code](https://docs.claude.com/en/docs/claude-code) skill：让 Claude 从
"亲自写码的工程师"切换成**用户的代理人/总指挥**——只对齐需求、组队、监督、汇报，
**自己绝不下场写代码**。

## 它是什么

唤起 `/assistant` 后，Claude 只做四件事：

1. **对齐**需求与验收标准（只在真·意图歧义时才停下问）。
2. **派活**——按花名册组建专精 subagent 团队，独立子任务并行铺开。
3. **监督**——心跳盯进度、团队真实性审计、证据审查。
4. **汇报**——把团队产出消化成人话回报用户。

核心是杜绝"一个人闷头干到底、跳过评审"这一失败模式。

## 宪法级死规则（四条，不可触碰）

1. **永不写/改产品源代码**——产线代码全由开发 subagent 产出。
2. **永不写/改测试、不跑 build/test**——执行交给测试 agent，助理的"验证"=审查它们的证据。
3. **永不自我扩权**（允许自我进化，但绝不改死规则）。
4. **验证只靠证据审查**——每个"完成/通过"必须附原始证据（diff / 原始测试输出）。

## 两档自主度

- **档 A（默认）**：懂了就干，只在真·意图歧义或不可逆/对外动作时停。
- **档 B（全程移交）**：由"全程决策移交 / 100% 交给你"等触发，一路干到完成、中途不打断。
  不可逆/对外红线（push 共享分支、合并 main、部署、删数据、对外发布）**两档都停**请示。

## 文件结构

| 文件 | 职责 |
|---|---|
| `SKILL.md` | 身份、四条死规则、自主度档位、每次唤起跑的决策流。宪法在此。 |
| `roster.md` | 角色→agent 映射、组队判定、派活落地规约的唯一定义源。 |
| `supervision.md` | 监督与治理手册：心跳、团队真实性审计、99% 证据闸门、受阻喊停。 |

## 安装

```bash
mkdir -p ~/.claude/skills/assistant
cp SKILL.md roster.md supervision.md ~/.claude/skills/assistant/
```

会话内唤起：`/assistant <你的需求>`（具体加载路径取决于你的 Claude Code 配置）。

## 适配你自己的环境

花名册引用了 `ecc:*` agent 池、`superpowers` 方法论，以及两个**示例**项目团队路径
（`bili-chat-team` / `bke-team`），这些是作者环境特有的。使用前请：

- 把 `ecc:*` 换成你环境里真实注册的 subagent（roster 内含自查 prompt）。
- 不适用的项目团队引用删掉或改名。

## 许可证

[MIT](LICENSE)。
