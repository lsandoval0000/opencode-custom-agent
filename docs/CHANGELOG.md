# Changelog

User-facing notes on changes to the agent/instruction corpus. Newest first.

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
