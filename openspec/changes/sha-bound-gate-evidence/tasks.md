# Tasks

> Tier T1. Per the repo rule (openspec-lint, "openspec/config.yaml proposal rule"): tasks are not
> checkable without eval evidence — each group names the eval section covering it, and the final
> `python3 -m desk.verify gate` (= `make eval-fast`) result is recorded below before the change is
> considered implemented. Groups 1→2 are deliberately red-first: the checks land and fail before
> the behavior change. Evidence cells marked _implementing PR_ are filled on branch `agent/3-*`,
> not by the spec-seed PR.

## 1. Red: pin the fields before they exist — eval section `verify_cli`

- [ ] 1.1 In `desk/eval/run_eval.py:check_verify_cli`, add checks that every `--json`
      subcommand's single JSON object carries `git_sha` (40-hex, equal to
      `git rev-parse HEAD` of the tree under test), `dirty` (bool), and `ts` (ISO-8601).
- [ ] 1.2 Run `python3 -m desk.verify gate --json`: `verify_cli` must FAIL on exactly the new
      checks. Record the red evidence here:

| Red run (pre-change) | Evidence |
|---|---|
| Failing check names + section counts | _implementing PR_ |

## 2. Green: emit the fields — eval section `verify_cli`

- [ ] 2.1 In `desk/verify.py`, add one read-only helper computing
      `{"git_sha": <HEAD>, "dirty": <bool>, "ts": <iso8601>}` via `git rev-parse HEAD` /
      `git status --porcelain` child processes, and merge it into every subcommand's `--json`
      object. PROTECTED PATH: this edit requires explicit gate-1 authorization per the
      proposal's delivery note — do not start group 2 without it.
- [ ] 2.2 Confirm the static guards still hold (framework-only imports, no config-writing
      identifiers, no control POSTs): `verify_cli`'s existing checks, unmodified, stay green.
- [ ] 2.3 Run the gate again: `verify_cli` PASSES including the group-1 checks. Record here:

| Green run (post-change) | Evidence |
|---|---|
| `gate --json` result + failing=none | _implementing PR_ |
| Clean-tree run: `git_sha` == `git rev-parse HEAD`, `dirty: false` (criterion 1) | _implementing PR_ |
| Dirty-tree run: `dirty: true` (criterion 2) | _implementing PR_ |

## 3. Document the convention — eval section: none (docs line; covered by the gate run in group 4)

- [ ] 3.1 AGENTS.md verification-workflow section gains the one-line convention: PR evidence
      blocks paste the `gate --json` object; the reviewer diffs its `git_sha` against the merge
      ref. (`.github/pull_request_template.md` is a protected path and is NOT edited.)

## 4. Final gate evidence (required before the change is implemented)

- [ ] 4.1 Re-run `python3 -m desk.verify gate --json` after the final commit and record the
      result + the head SHA it ran on:

| Final gate | Value |
|---|---|
| Head SHA (`git rev-parse HEAD`) | _implementing PR_ |
| `RESULT` | _implementing PR_ |
| `verify_cli` passed/failed | _implementing PR_ |
