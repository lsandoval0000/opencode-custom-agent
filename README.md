# OpenCode Custom Agents

A multi-agent orchestration system for [OpenCode V2](https://opencode.ai/v2/docs/) — seven specialized agents that collaborate to handle complex tasks from planning through implementation and verification.

## How It Works

An orchestrator (**A.L.L.I.C.E.**) coordinates the work. She breaks goals into a **global plan** — an index of one or more **sub-plans** — delegates each unit to the right specialist, and tracks progress end-to-end. The user never interacts with subagents directly — A.L.L.I.C.E. handles all routing.

```
User → A.L.L.I.C.E. → Planner ──→ writes .aiw/plans/<slug>.md (one or more sub-plans)
                        ↓ manifest
      A.L.L.I.C.E. ──→ .aiw/global-plan.md (index) + .aiw/worklogs/<slug>.md (one per sub-plan)
                        ↓ routes GLOBAL PLAN / SUB-PLAN / WORKLOG in every brief
      ┌────────────┬────────────┬────────────┬──────────────────┐
      ↓            ↓            ↓            ↓                  ↓
  Explorer      Builder      Tester      Summarizer        Documenter
      └────────────┴────────────┴────────────┴──────────────────┘
                        ↓
              write the routed worklog
```

## Agents

| Agent | Role | Access |
|-------|------|--------|
| **A.L.L.I.C.E.** (Lead orchestrator) | Lead coordinator. Checks the global plan on every session, routes sub-plans and worklogs, delegates, and reports progress. **All tasks must go through ALL phases (explorer → planner → builder → tester). No exceptions, regardless of task size.** Asks the user before each delegation whether to change the model for the upcoming subagent. All output in English. | Read-only on code; full read+write on `.aiw/**` (owns `.aiw/global-plan.md` — index, live rollup, append-only `## Global Rulings` — and worklog headers; never authors sub-plan content); sub-plan/global-plan corrections require the user's prior approval; no shell, no web tools. |
| **Planner** | Senior-level strategic planner. Researches best practices, analyzes the codebase, and produces a set of self-contained sub-plans with maximum detail. Evidence-based planning using available tools (Context7, websearch). | Read-only (code + web); writes `.aiw/plans/<slug>.md` for every sub-plan and the briefed planning worklog (`.aiw/worklogs/planning.md`); never writes the global plan. |
| **Explorer** | Senior-level read-only investigator. Never assumes — verifies with evidence. Leverages available tools (Context7, websearch) to gather facts from the project and returns cited findings. | Read-only (code, webfetch, websearch); writes to the worklog named in its brief (journal — visible corrections). |
| **Builder** | Senior-level implementation specialist. Never assumes — searches and applies design patterns. Produces code, configs, docs, or any artifact at production quality. Best possible change with maximum quality (not minimal diff). Avoids overengineering. | Full edit + shell scoped to the brief, except `.aiw/**` (only the briefed worklog is writable); writes to the routed worklog (journal — visible corrections). |
| **Tester** | Senior QA engineer. Never assumes — tests edge cases thoroughly. Researches unfamiliar tools/libraries before testing. Proves whether acceptance criteria hold via tests, builds, and validation checklists. | Shell (code, webfetch, websearch); writes to the worklog named in its brief (journal — visible corrections). Never edits project files. |
| **Summarizer** | Distills completed work into a fact-dense summary absorbable in under a minute. | Read-only for project content, no shell, no web tools; writes to the briefed worklog (journal — visible corrections). |
| **Documenter** | Writes human-readable documentation grounded in evidence from the routed sub-plan and worklog. | Writes `docs/**` and `README*`; writes to the briefed worklog (journal — visible corrections). |

**Note:** The **Access** column describes each agent's own instructions rather than a declarative configuration; OpenCode's base policy applies underneath, and each agent's scope is enforced through its prompt.

## Installation

These are native OpenCode **V2** agent definitions. Each file carries `mode` (primary or subagent) and an optional `request.body` sampling block. Copy the `agents/` folder into your OpenCode configuration directory:

```bash
# Linux / macOS
cp -r agents/ ~/.config/opencode/agents/

# Windows
xcopy agents\ %USERPROFILE%\.config\opencode\agents\ /E /I
# Agent definitions load at startup — restart OpenCode after copying.
```

Restart OpenCode after copying — agent definitions are loaded at startup, so an already-running session keeps the old definitions.

Or symlink it:

```bash
ln -s /path/to/opencode-custom-agent/agents ~/.config/opencode/agents
```

For a single project, place them under `.opencode/agents/` instead; files in that directory are project-local.

## Usage

Once installed, the agents are available in OpenCode. **A.L.L.I.C.E.** (agent ID `allice`) is the primary entry point — just describe your goal and she will coordinate the rest.

### Key Features

- **Model selection on delegation (enforced)** — Before every delegation, A.L.L.I.C.E. must ask the user whether to change the model. The chosen model is recorded in each delegation brief's MODEL field. This is a hard constraint, not optional.
- **English only** — All output, logs, briefs, and communication are enforced in English.
- **Global plan gate** — Every session starts by checking `.aiw/global-plan.md`. If it is missing, A.L.L.I.C.E. asks the user what to do instead of inventing one; if it exists, she verifies every referenced sub-plan and worklog before routing work.
- **Multiple sub-plans, one index** — The planner emits one or more self-contained sub-plans to `.aiw/plans/<slug>.md`; the orchestrator indexes them in `.aiw/global-plan.md` (root → branches → leaf actions) and assigns `SP-N` IDs.
- **Routed worklogs** — Every sub-plan has its own worklog at `.aiw/worklogs/<slug>.md`; planning/re-planning uses `.aiw/worklogs/planning.md`. A.L.L.I.C.E. chooses the sub-plan and worklog for every delegation and puts both paths in the brief.
- **Self-sufficient subagents** — Each subagent writes its own outputs directly to files (sub-plans, worklogs) and returns only a short summary to the orchestrator. The orchestrator never handles full deliverable content.
- **Journal worklogs (visible corrections)** — Each worklog shows its sub-plan's work end to end. Entries may be corrected in place, but nothing is ever silently rewritten: a correction must strike the wrong value through, put the correct value beside it, and state the reason (e.g. `~~wrong~~ **correct** — corrected <YYYY-MM-DD HH:mm>: <reason>`). Deleting or pruning is forbidden.
- **Append-only rulings, split by scope** — Each sub-plan carries a `## Plan Rulings` section (`R-00N`); `.aiw/global-plan.md` carries `## Global Rulings` (`GR-00N`) for set-level decisions. Rulings are appended and never edited or deleted; a later ruling supersedes an earlier one by reference.
- **User-approved plan corrections** — a generated sub-plan or global plan may be validated and corrected later, but only with the user's approval: A.L.L.I.C.E. first notifies the user (what is wrong, the fix, the impact); each change is recorded as a ruling in the correct scope.

### Example

> "Add user authentication with JWT to my Express app"

A.L.L.I.C.E. will:
1. Delegate to the **Planner** — writes one or more sub-plans to `.aiw/plans/<slug>.md`, each with an append-only `## Plan Rulings` section, and returns a manifest
2. Create `.aiw/global-plan.md` (index + execution order) and one worklog per sub-plan under `.aiw/worklogs/`
3. Send the **Explorer** — investigates existing auth patterns within a routed sub-plan, writes findings to its worklog
4. Have the **Builder** — implements that sub-plan's nodes, writes to the routed worklog
5. Run the **Tester** — verifies acceptance criteria, writes to the routed worklog
6. Ask the **Summarizer** — distills what was done, writes to the routed worklog
7. Have the **Documenter** — writes the final docs, writes to the routed worklog

### Mandatory Phase Workflow

**All tasks must go through ALL phases (explorer → planner → builder → tester). No exceptions, regardless of task size.** This is a hard constraint enforced by the orchestrator. Every task, whether a one-line fix or a complex feature, follows the complete workflow to ensure thorough investigation, planning, implementation, and verification.

## Project Structure

```
opencode-custom-agent/
├── agents/
│   ├── ALLICE.md          # Lead coordinator (A.L.L.I.C.E.); agent ID: allice
│   ├── planner.md         # Strategic planner (sub-plan set)
│   ├── explorer.md        # Read-only investigator
│   ├── builder.md         # Implementation specialist
│   ├── tester.md          # Verification specialist
│   ├── summarizer.md      # Summary specialist
│   └── documenter.md      # Documentation writer
├── docs/
│   └── CHANGELOG.md       # User-facing change notes
├── .gitignore
└── README.md
```

At runtime, work is tracked under `.aiw/` (gitignored):

```
.aiw/                          # runtime root
├── global-plan.md             # root index: sub-plan registry, rollup, global rulings
├── plans/
│   └── <slug>.md              # one self-contained sub-plan per file
├── worklogs/
│   ├── planning.md            # planner's journal (planning / re-planning)
│   └── <slug>.md              # one worklog per sub-plan
└── ...                        # other agent working files/subfolders
```

## License

MIT
