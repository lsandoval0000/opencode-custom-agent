# OpenCode Custom Agents

A multi-agent orchestration system for [OpenCode V2](https://opencode.ai/v2/docs/) — seven specialized agents that collaborate to handle complex tasks from planning through implementation and verification.

## How It Works

An orchestrator (**A.L.L.I.C.E.**) coordinates the work. She breaks goals into a task tree, delegates each unit to the right specialist, and tracks progress end-to-end. The user never interacts with subagents directly — A.L.L.I.C.E. handles all routing.

```
User → Orchestrator → Planner ──→ writes .aiw/plan.md
                       ↓
              ┌────────┼────────┐
              ↓        ↓        ↓
          Explorer  Builder  Tester
              ↓        ↓        ↓
              └────────┼────────┘
                       ↓
              ┌────────┼────────┐
              ↓                 ↓
          Summarizer      Documenter
              ↓                 ↓
         writes .aiw/worklog.md (all subagents)
```

## Agents

| Agent | Role | Access |
|-------|------|--------|
| **A.L.L.I.C.E.** (Lead orchestrator) | Lead coordinator. Bootstraps tracking, creates the task tree, delegates, and reports progress. **All tasks must go through ALL phases (explorer → planner → builder → tester). No exceptions, regardless of task size.** Asks the user before each delegation whether to change the model for the upcoming subagent. All output in English. | Read-only on code; full read+write on `.aiw/**` (bootstraps `.aiw/worklog.md`, maintains `.aiw/plan.md`); no shell, no web tools. |
| **Planner** | Senior-level strategic planner. Researches best practices, analyzes the codebase, and produces a complete execution plan with maximum detail. Evidence-based planning using available tools (Context7, websearch). | Read-only (code + web); writes `.aiw/plan.md` and `.aiw/worklog.md`. |
| **Explorer** | Senior-level read-only investigator. Never assumes — verifies with evidence. Leverages available tools (Context7, websearch) to gather facts from the project and returns cited findings. | Read-only (code, webfetch, websearch); appends to `.aiw/worklog.md`. |
| **Builder** | Senior-level implementation specialist. Never assumes — searches and applies design patterns. Produces code, configs, docs, or any artifact at production quality. Best possible change with maximum quality (not minimal diff). Avoids overengineering. | Full edit + shell scoped to the brief, except `.aiw/**` (only `.aiw/worklog.md` is writable, where the `edit` action grants edit/write/patch); appends to `.aiw/worklog.md`. |
| **Tester** | Senior QA engineer. Never assumes — tests edge cases thoroughly. Researches unfamiliar tools/libraries before testing. Proves whether acceptance criteria hold via tests, builds, and validation checklists. | Shell (code, webfetch, websearch); appends to `.aiw/worklog.md`. Never edits project files. |
| **Summarizer** | Distills completed work into a fact-dense summary absorbable in under a minute. | Read-only, no shell, no web tools; appends to `.aiw/worklog.md`. |
| **Documenter** | Writes human-readable documentation grounded in evidence from the worklog. | Writes `docs/**` and `README*`; appends to `.aiw/worklog.md`. |

## Installation

These are native OpenCode **V2** agent definitions. Each file carries `mode` (primary or subagent), an optional `request.body` sampling block, and an ordered `permissions` rule list. Copy the `agents/` folder into your OpenCode configuration directory:

```bash
# Linux / macOS
cp -r agents/ ~/.config/opencode/agents/

# Windows
xcopy agents\ %USERPROFILE%\.config\opencode\agents\ /E /I
# Agent definitions (including their permission rules) load at startup — restart OpenCode after copying.
```

Restart OpenCode after copying — agent definitions (including their permission rules) are loaded at startup, so an already-running session keeps the old rules.

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
- **Task tree tracking** — Progress is tracked in `.aiw/plan.md` with live status updates.
- **Self-sufficient subagents** — Each subagent writes its own outputs directly to files (plan, worklog) and returns only a short summary to the orchestrator. The orchestrator never handles full deliverable content.
- **Append-only worklog** — A.L.L.I.C.E. bootstraps `.aiw/worklog.md` (session header) and maintains `.aiw/plan.md`; every subagent appends its own entry to `.aiw/worklog.md` for full auditability. A.L.L.I.C.E. does not duplicate subagent entries.

### Example

> "Add user authentication with JWT to my Express app"

A.L.L.I.C.E. will:
1. Delegate to the **Planner** — writes the plan directly to `.aiw/plan.md`, returns a summary
2. Send the **Explorer** — investigates existing auth patterns, writes findings to worklog
3. Have the **Builder** — implements each task node, writes to worklog
4. Run the **Tester** — verifies acceptance criteria, writes to worklog
5. Ask the **Summarizer** — distills what was done, writes to worklog
6. Have the **Documenter** — writes the final docs, writes to worklog

### Mandatory Phase Workflow

**All tasks must go through ALL phases (explorer → planner → builder → tester). No exceptions, regardless of task size.** This is a hard constraint enforced by the orchestrator. Every task, whether a one-line fix or a complex feature, follows the complete workflow to ensure thorough investigation, planning, implementation, and verification.

### Agent Permissions

Agents use the native V2 `permissions` field: an ordered list of `action` / `resource` / `effect` rules evaluated **last-match-wins**. If no rule matches, OpenCode asks. Every agent — custom agents included — starts from the same base policy, whose first rule is `{"action":"*","resource":"*","effect":"allow"}`, followed by `ask` rules for `external_directory` and `.env` reads. Each agent here therefore opens with the explicit denies that matter, then re-allows only what it needs.

**Resources are matched as whole values against the Location-relative path.** A resource such as `.aiw/worklog.md` only matches when the active Location is the project directory. When the Location is a volume root — which happens when the session's `projectID` is `global` — the very same file is addressed as `Desarrollo/Repos/<project>/.aiw/worklog.md`, the literal rule never matches, and the earlier `edit: "*" deny` wins. Note also that `**` is **not** a special operator in V2: it is simply two `*` characters, so `**/.aiw/**` still requires a literal `/` immediately before `.aiw/` and therefore fails on project-relative paths in exactly the way `.aiw/**` fails on volume-root paths.

To stay correct under **both** forms, every path rule is declared as a precise pair — the project-relative form plus its `**/`-prefixed twin:

```yaml
- action: edit
  resource: ".aiw/worklog.md"      # Location = project directory
  effect: allow
- action: edit
  resource: "**/.aiw/worklog.md"   # Location = volume root (or any nested path)
  effect: allow
```

A single `*`-prefixed rule (`*.aiw/worklog.md`) also covers both forms in one line, because `*` matches `/` as well. The pair is used instead because it cannot over-match a directory that merely ends in `.aiw` and because each half is explicit about the form it covers. **Never add a path rule that covers only one form** — that is precisely the bug these rules were rewritten to fix.

Because agent rules are appended after global rules, an agent-level `allow` overrides a later global restriction on the same action and resource. That is why `read`, `glob`, and `grep` are deliberately left undeclared here: an agent-level `read: "*" allow` would also neutralize the base policy's `.env` `ask` rules.

Each agent explicitly declares the actions its role needs:

- **Orchestrator (A.L.L.I.C.E.)** — full read+write on `.aiw/**` in both Location forms; no `edit` anywhere else, no `shell`, no web tools, no `skill`. May launch the six specialists (`subagent` allow per ID, after a `*` deny) and may ask the user (`question` allow). Bootstraps `.aiw/worklog.md`, maintains `.aiw/plan.md`, and does not duplicate subagent worklog entries.
- **Planner** — `edit` deny-all, then `.aiw/plan.md` and `.aiw/worklog.md`, each in both Location forms. `webfetch` / `websearch` / `skill` / `question` allow; `shell` / `subagent` deny.
- **Explorer** — `edit` deny-all, then `.aiw/worklog.md` in both forms. `webfetch` / `websearch` / `skill` / `question` allow; `shell` / `subagent` deny.
- **Builder** — no `edit` deny-all (full edit, scoped by the brief), a real `.aiw/**` deny in both forms, then `.aiw/worklog.md` allow in both forms. `shell` / `webfetch` / `websearch` / `skill` / `question` allow; `subagent` deny.
- **Tester** — `edit` deny-all, then `.aiw/worklog.md` in both forms. `shell` / `webfetch` / `websearch` / `skill` / `question` allow; `subagent` deny.
- **Summarizer** — `edit` deny-all, then `.aiw/worklog.md` in both forms. `shell` / `subagent` / web tools / `skill` deny; `question` allow. Read-only.
- **Documenter** — `edit` deny-all, then `docs/**`, `README*` and `.aiw/worklog.md`, each in both forms. `webfetch` / `websearch` / `skill` / `question` allow; `shell` / `subagent` deny.
- **Model confirmation** — Before delegating to any subagent, the orchestrator asks the user whether to change the model. The selected model is recorded in the delegation brief.

## Project Structure

```
opencode-custom-agent/
├── agents/
│   ├── ALLICE.md          # Lead coordinator (A.L.L.I.C.E.); agent ID: allice
│   ├── planner.md         # Strategic planner
│   ├── explorer.md        # Read-only investigator
│   ├── builder.md         # Implementation specialist
│   ├── tester.md          # Verification specialist
│   ├── summarizer.md      # Summary specialist
│   └── documenter.md      # Documentation writer
├── .gitignore
└── README.md
```

## License

MIT
