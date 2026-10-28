# Proposal

> Spec for GitHub issue #3 (pilot). Transcribed 1:1 from the issue body by the pipeline
> shepherd — scope decisions are the issue's, not this PR's.

## Why

PR evidence today is pasted gate output. Nothing ties a `RESULT: PASS` to the tree it ran on, so
a pasted pass cannot be distinguished from one produced on a different commit. Evidence is not
attempt-bound, which weakens every review that relies on it.

## What Changes

- `python3 -m desk.verify gate --json` (and the other subcommands' `--json` objects: `tenants`,
  `tick`, `health`, `feature-map`) include `{"git_sha": "<HEAD>", "dirty": <bool>,
  "ts": "<iso8601>"}` in the single JSON object each run already emits.
- Convention: PR evidence blocks paste that JSON; the reviewer diffs `git_sha` against the merge
  ref. The convention line lands in the AGENTS.md verification-workflow section (NOT in
  `.github/pull_request_template.md`, which is a protected path).
- The `verify_cli` eval section pins the new fields so they cannot silently disappear.
- The CLI stays read-only: computing `git_sha`/`dirty` uses read-only `git rev-parse` /
  `git status --porcelain` child processes, within the `verify_cli` section's static guards (no
  new desk imports, no config writes, no control POSTs).

## Success criteria (bind the implementing PR, branch `agent/3-*`)

1. On a clean tree, `python3 -m desk.verify gate --json` emits exactly one JSON object whose
   `git_sha` equals `git rev-parse HEAD`, whose `dirty` is `false`, and whose `ts` is a valid
   ISO-8601 timestamp.
2. With uncommitted changes in the tree, the same command reports `dirty: true`.
3. Every other `--json` subcommand (`tenants`, `tick`, `health`, `feature-map`) carries the same
   three fields in its single JSON object.
4. The `verify_cli` eval section pins the new fields (red-first: the new checks land and FAIL
   before the emission change, then pass with it; a check that still passes with the feature
   reverted is tautological and rejected).
5. The offline golden gate stays green on the head SHA (`python3 -m desk.verify gate --json`,
   = `make eval-fast`), including `verify_cli`'s existing static read-only guards.

## Evidence contract (what the implementing PR must show)

- The PR template's gate block: full `gate --json` output plus the exact head SHA it ran on —
  which, once this change lands, is self-binding: the pasted object itself carries `git_sha`.
- Criteria map: #1–#3 point at the diff hunks plus a pasted clean-tree and dirty-tree run; #4
  points at the red run recorded in `tasks.md` task 1.2 and the green gate after; #5 is the gate
  block itself.

## Eval gate sections

Eval gate section(s) added or changed: changes `verify_cli` (HARD, in `--fast`) — it gains
checks pinning `git_sha` (equal to the tree's HEAD), `dirty`, and `ts` in every `--json`
object. No new section.

## Delivery note (two PRs + a protected path)

The public tree ships without `openspec/changes/`, so this change lands in two PRs:

- **Spec-seed PR** (this one): introduces this proposal and `tasks.md` only. Its verifiable
  claims are that `openspec/changes/sha-bound-gate-evidence/` exists with numbered success
  criteria, an evidence contract, the eval-section line and tier-T1 tasks, and that the gate
  stays green (no behavior change). The numbered criteria above do NOT bind the seed PR.
- **Implementing PR** (branch `agent/3-*`, opened by the lane-G build worker): bound by success
  criteria 1-5 above.

Protected-path note for gate 1: the emission change edits `desk/verify.py`, which is on the
protected-paths list (the PR checklist exempts only tier:T0). This issue is tier:T1, so the
gate-1 reviewer must either explicitly authorize that single-file edit in the spec approval or
re-tier the issue; the implementing worker must not improvise around it.
