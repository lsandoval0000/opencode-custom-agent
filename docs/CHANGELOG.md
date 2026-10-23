# Changelog

User-facing notes on changes to the agent/instruction corpus. Newest first.

## 2026-10-23 — Multi-plan workspaces: global index, sub-plans, routed worklogs

The single-plan / single-worklog model is gone. A goal can now span **multiple sub-plans**, each with its own **worklog**, all indexed by one **global plan**. The root tracking folder is still `.aiw/`; only the layout and ownership rules changed. The rules live in all 7 `agents/*.md` files and `README.md` (the repo's deliberate sync convention).

### What changed

- **`.aiw/global-plan.md` is the root.** It is an index, not a task tree: a goal line, a sub-plan registry (`SP-N`, file, worklog, dependencies, status), an execution order, and an append-only `## Global Rulings` ledger (`GR-00N`).
- **Sub-plans live in `.aiw/plans/<slug>.md`.** The planner writes **one or more** self-contained sub-plans, each using the sub-plan template and scoped to its own slice. It never writes the global plan. It returns a manifest (files, titles, scopes, dependencies, confidence) instead of plan content.
- **The orchestrator owns the index and the routing.** After the planner returns, A.L.L.I.C.E. creates `global-plan.md`, assigns `SP-N` IDs in dependency order, creates one worklog per sub-plan, and records `GR-001`.
- **One worklog per sub-plan.** Worklogs live at `.aiw/worklogs/<slug>.md`; planning/re-planning uses `.aiw/worklogs/planning.md`. Every delegation brief now carries `GLOBAL PLAN`, `SUB-PLAN`, and `WORKLOG`, and subagents write **only** to the routed worklog. A brief without `WORKLOG` is invalid — the subagent must return BLOCKED.
- **Global plan gate on every session.** A.L.L.I.C.E. checks `.aiw/global-plan.md` before any delegation. If it is missing, she asks the user what to do (plan now, index existing sub-plans, or stop) and never invents one. If it exists, she verifies every referenced sub-plan and worklog before routing.
- **The template is reused per sub-plan.** The planner's output format is now explicitly the **Sub-Plan Template**, applied to every sub-plan and scoped to that sub-plan's context. Tree limits (10 wide / 5 deep) apply per sub-plan.
- **Rulings are split by scope.** Plan-local rulings (`R-00N`) stay in each sub-plan; set-level rulings (`GR-00N`) live in the global plan. Both remain append-only; corrections still need the user's prior approval and a visible-correction trail. Live progress markers are not rulings.
- **Legacy files are gone.** There is no `.aiw/plan.md` and no `.aiw/worklog.md`; references to them were removed from every agent and the README.

### Why it matters

- **Large goals scale:** one goal can be decomposed into independently executable, verifiable pieces without one giant plan file.
- **Clean audit trail:** each sub-plan's work is journaled in its own worklog, and set-level decisions have their own immutable ledger.
- **No guessed paths:** the orchestrator is the single router; subagents cannot drift into the wrong plan or log.

### Notes

- Scope: the 7 `agents/*.md` definitions, `README.md`, and this changelog entry. Historical changelog entries are left intact — they record the model as it was.
- Validation was a text-consistency pass: no fixed `.aiw/plan.md` or `.aiw/worklog.md` paths in the agent definitions or README; routing fields and the `Sub-plan:` entry line present in all executors; the planner template and manifest present; the README matches the implemented rules.
- Disclosure: this repo's `agents/` files were **not** copied to `~/.config/opencode/agents` in this change; the live config will be updated separately.

## 2026-09-30 — A correctable trail: journal worklog, append-only plan rulings, user-approved plan corrections

A trust overhaul of how the agents record work. The trail stays **complete end to end**, and it can now be **fixed without being hidden**. The rules are duplicated in all 7 `agents/*.md` files and `README.md` (the repo's deliberate sync convention).

### ✨ What changed

- **The worklog is now a journal, not "append-only".** `.aiw/worklog.md` shows everything from start to finish. An entry may be corrected in place — but nothing is ever silently rewritten.
- **A visible recorded correction is now legal and standardized.** A correction strikes the wrong value through, puts the correct value beside it, and states the reason:

  ```text
  ~~wrong~~ **correct** — corrected <YYYY-MM-DD HH:mm>: <reason>
  ```

  Example: `- Status: ~~DONE~~ **PARTIAL** — corrected 2026-09-30 14:05: concurrency test failed.`
  Deleting or pruning entries is forbidden. The rule applies to worklog entries and to **build records** (the builder's recorded evidence — its worklog entry, `FILES TOUCHED`, and `DECISIONS`).
- **Plan rulings are append-only.** `.aiw/plan.md` now ends with a `## Plan Rulings` ledger. A ruling is never edited or deleted; a later ruling supersedes an earlier one *by reference*:

  ```text
  - R-00N [YYYY-MM-DD HH:mm] <ruling> — by: <role> — basis: <evidence> · supersedes: <R-00N|none> · approved-by: <user|n/a>
  ```

- **Plan corrections now need your approval first.** The generated plan is the source of truth, but it can be validated and corrected later. Before any change to `.aiw/plan.md` beyond a live progress field or a new ruling, A.L.L.I.C.E. must notify you (what is wrong, the proposed fix, the impact) and wait for explicit approval. Every approved change is recorded as a ruling. This is a **separate gate** from the existing per-delegation model-selection approval.
- **Live progress stays frictionless.** The plan's `> Status:` line and the tree's status markers (`[ ]` pending · `[~]` in progress · `[x]` done · `[!]` blocked) are progress, not corrections — they keep updating in place with no approval and no ruling.

### 💡 Why it matters

- **To readers:** you can always see what happened, including what was changed and why — nothing meaningful gets quietly overwritten.
- **To agents:** mistakes in an entry are recoverable (fix them visibly) instead of frozen forever, while the plan's decisions across a project remain a stable, immutable record.
- **No new files:** rulings live inside `.aiw/plan.md`; no separate ledger to manage.

### 📝 Notes

- Scope of the change: the 7 `agents/*.md` definitions and `README.md`. No code and no build system — validation was a grep/consistency pass.
- Verification: the Validation Loop passed **11/11** on re-run (an earlier run was 10/11, where the single miss was a defect in the plan's own stated expectation, not a regression in the corpus).
- Three plan rulings were recorded during the work: **R-001** (plan created), **R-002** (user approved adding a literal correction example to `README.md`, aligning the README phrases, and making the L1.1 check intent-based), and **R-003** (user approved correcting the L2.2 expectation from 2 to 1).
- `.aiw/**` is gitignored runtime state, so plan and worklog changes are intentionally not committed.
