# OpenCode Custom Agents

A multi-agent orchestration system for [OpenCode](https://github.com/opencode-ai/opencode) — seven specialized agents that collaborate to handle complex tasks from planning through implementation and verification.

## How It Works

An orchestrator (**A.L.L.I.C.E.**) coordinates the work. She breaks goals into a task tree, delegates each unit to the right specialist, and tracks progress end-to-end. The user never interacts with subagents directly — A.L.L.I.C.E. handles all routing.

```
User → Orchestrator → Planner → Explorer
                           ↓         ↓
                         Builder → Tester
                           ↓         ↓
                       Summarizer → Documenter
```

## Agents

| Agent | Role | Access |
|-------|------|--------|
| **A.L.L.I.C.E.** (Lead orchestrator) | Lead coordinator. Bootstraps tracking, creates the task tree, delegates, and reports progress. Asks the user before each delegation whether to change the model for the upcoming subagent. All output in English. | Read-only on code; full access to `.aiw/` tracking folder |
| **Planner** | Strategic planner. Researches best practices, analyzes the codebase, and produces a complete execution plan. | Read-only (code + web) |
| **Explorer** | Read-only investigator. Gathers facts from the project and returns cited findings. | Read-only only |
| **Builder** | Implementation specialist. Produces code, configs, docs, or any artifact at production quality. | Full edit/write/bash, scoped to brief |
| **Tester** | Verification specialist. Proves whether acceptance criteria hold via tests, builds, and validation checklists. | Bash (read-only on files) |
| **Summarizer** | Distills completed work into a fact-dense summary absorbable in under a minute. | Read-only only |
| **Documenter** | Writes human-readable documentation grounded in evidence from the worklog. | Edit `docs/` and `README*` only |

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
- **Append-only worklog** — Every delegation is recorded in `.aiw/worklog.md` for full auditability.

### Example

> "Add user authentication with JWT to my Express app"

A.L.L.I.C.E. will:
1. Delegate to the **Planner** to analyze the codebase and produce an execution plan
2. Send the **Explorer** to investigate existing auth patterns
3. Have the **Builder** implement each task node
4. Run the **Tester** to verify acceptance criteria
5. Ask the **Summarizer** to distill what was done
6. Have the **Documenter** write the final docs

### Agent Permissions

Each agent has carefully scoped permissions:

- **Read-only agents** (Explorer, Planner, Summarizer) cannot modify files or run commands
- **Builder** has full access but is strictly scoped to its brief
- **Tester** can run commands but never edits files
- **Documenter** can only write to `docs/` and README files
- **Orchestrator** can only write to `.aiw/` tracking folders
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
