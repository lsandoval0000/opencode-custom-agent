# OpenCode Custom Agents

A multi-agent orchestration system for [OpenCode](https://github.com/opencode-ai/opencode) — seven specialized agents that collaborate to handle complex tasks from planning through implementation and verification.

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
| **A.L.L.I.C.E.** (Lead orchestrator) | Lead coordinator. Bootstraps tracking, creates the task tree, delegates, and reports progress. **All tasks must go through ALL phases (explorer → planner → builder → tester). No exceptions, regardless of task size.** Asks the user before each delegation whether to change the model for the upcoming subagent. All output in English. | Read-only on code; full access to `.aiw/` tracking folder |
| **Planner** | Senior-level strategic planner. Researches best practices, analyzes the codebase, and produces a complete execution plan with maximum detail. Evidence-based planning using available tools (Context7, websearch). | Read-only (code + web); writes `.aiw/plan.md` and `.aiw/worklog.md` |
| **Explorer** | Senior-level read-only investigator. Never assumes — verifies with evidence. Leverages available tools (Context7, websearch) to gather facts from the project and returns cited findings. | Read-only (code, webfetch, websearch); appends to `.aiw/worklog.md` |
| **Builder** | Senior-level implementation specialist. Never assumes — searches and applies design patterns. Produces code, configs, docs, or any artifact at production quality. Best possible change with maximum quality (not minimal diff). Avoids overengineering. | Full edit/write/bash, scoped to brief; appends to `.aiw/worklog.md` |
| **Tester** | Senior QA engineer. Never assumes — tests edge cases thoroughly. Researches unfamiliar tools/libraries before testing. Proves whether acceptance criteria hold via tests, builds, and validation checklists. | Bash (code, webfetch, websearch); appends to `.aiw/worklog.md` |
| **Summarizer** | Distills completed work into a fact-dense summary absorbable in under a minute. | Read-only; appends to `.aiw/worklog.md` |
| **Documenter** | Writes human-readable documentation grounded in evidence from the worklog. | Edit `docs/` and `README*`; appends to `.aiw/worklog.md` |

## Installation

Copy the `agents/` folder into your OpenCode configuration directory:

```bash
# Linux / macOS
cp -r agents/ ~/.config/opencode/agents/

# Windows
xcopy agents\ %USERPROFILE%\.config\opencode\agents\ /E /I
```

Or symlink it:

```bash
ln -s /path/to/opencode-custom-agent/agents ~/.config/opencode/agents
```

## Usage

Once installed, the agents are available in OpenCode. The orchestrator is the primary entry point — just describe your goal and A.L.L.I.C.E. will coordinate the rest.

### Key Features

- **Model selection on delegation (enforced)** — Before every delegation, A.L.L.I.C.E. must ask the user whether to change the model. The chosen model is recorded in each delegation brief's MODEL field. This is a hard constraint, not optional.
- **English only** — All output, logs, briefs, and communication are enforced in English.
- **Task tree tracking** — Progress is tracked in `.aiw/plan.md` with live status updates.
- **Self-sufficient subagents** — Each subagent writes its own outputs directly to files (plan, worklog) and returns only a short summary to the orchestrator. The orchestrator never handles full deliverable content.
- **Append-only worklog** — Every subagent appends its own entry to `.aiw/worklog.md` for full auditability. The orchestrator reads but does not write worklog entries.

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

Each agent has carefully scoped permissions:

- **Orchestrator** — Read-only on code; writes only to `.aiw/plan.md` (node statuses). Does NOT write worklog entries. Enforces mandatory phase workflow for all tasks.
- **Planner** — Read-only on code + web; writes `.aiw/plan.md` and `.aiw/worklog.md`. Senior-level expertise with evidence-based planning.
- **Explorer** — Read-only on code, webfetch, and websearch; appends to `.aiw/worklog.md`. Senior-level investigation with evidence verification.
- **Builder** — Full edit/write/bash scoped to brief; appends to `.aiw/worklog.md`. Senior-level implementation with design pattern research.
- **Tester** — Bash access with webfetch and websearch; appends to `.aiw/worklog.md`. Senior QA with edge case testing. Never edits project files.
- **Summarizer** — Read-only; appends to `.aiw/worklog.md`
- **Documenter** — Writes to `docs/` and README files; appends to `.aiw/worklog.md`
- **Model confirmation** — Before delegating to any subagent, the orchestrator asks the user whether to change the model. The selected model is recorded in the delegation brief.

## Project Structure

```
opencode-custom-agent/
├── agents/
│   ├── orchestrator.md    # Lead coordinator (A.L.L.I.C.E.)
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
