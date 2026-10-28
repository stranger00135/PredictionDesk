# Proposal

> Spec for GitHub issue #12 (pilot). Transcribed 1:1 from the issue body by the pipeline
> shepherd — scope decisions are the issue's, not this PR's.

## Why

Runtime data (capture, tracker, driver side-files) defaults to the checkout-anchored root in
`desk/runtime/paths.py` (`data_root()` -> `<repo>/desk`), while the control-plane audit log,
notifier state, and ops logs default to `~/polyedge-data`
(`desk/control_plane.py:control_data_root`, `desk/mcp_server.py:_data_root`). Both defaults are
documented and each is true for a different subsystem, so a user cannot form one mental model of
where their data lives, and the audit trail lands outside the checkout they think is the desk.

## What Changes

- `desk/control_plane.py:control_data_root()` defaults to `desk.runtime.paths.data_root()`
  (checkout-anchored) instead of `~/polyedge-data`; `desk/mcp_server.py:_data_root()` follows the
  same single default (it is documented to match `control_data_root`).
- `DESK_DATA_DIR` stays the single override for everything: setting it relocates runtime data,
  the audit JSONL, notifier state, and ops logs under one directory, as it already does today.
- `docs/mcp.md` and the ops docs state one default in one sentence; no doc still names
  `~/polyedge-data` as a default.
- No API change, no schedule change, no config-row or seal change.

## Success criteria (bind the implementing PR, branch `agent/12-*`)

1. With no `DESK_DATA_DIR` set, `desk.control_plane.control_data_root()` returns exactly
   `desk.runtime.paths.data_root()`, so the audit JSONL
   (`<root>/audit/control_audit.jsonl`), the driver ops log path, and the MCP server's
   notifier-state path all resolve under that one root — every subsystem writes under one
   default.
2. Setting `DESK_DATA_DIR=<abs dir>` relocates all of the paths in criterion 1 under that
   directory (behavior unchanged from today; the criterion pins that unification did not break
   the override).
3. An eval check lands in the implementing PR that FAILS on the pre-change dual default
   (red-first): with the env unset, `control_data_root() == runtime.paths.data_root()` — a check
   that would still pass with the fix reverted is tautological and rejected.
4. `docs/mcp.md` and the ops docs state the single default in one sentence; no shipped doc still
   presents `~/polyedge-data` as a default.
5. The offline golden gate stays green on the head SHA (`python3 -m desk.verify gate --json`,
   = `make eval-fast`), with the `control_api` and `notifier` sections passing on the unified
   default.

## Evidence contract (what the implementing PR must show)

- The PR template's gate block: full `gate --json` output plus the exact head SHA it ran on.
- Criteria map: #1 and #2 point at the diff hunks plus the new eval check's output; #3 points at
  the check's red run (recorded before the fix) and green run (after); #4 points at the doc
  hunks; #5 is the gate block itself.

## Eval gate sections

Eval gate section(s) added or changed: changes `control_api` (HARD, in `--fast`) — it gains the
unified-default pin of criterion 3 (fails while the two defaults coexist). `notifier` (HARD, in
`--fast`) must stay green on the moved notifier-state home. No new section.

## Delivery note (two PRs)

The public tree ships without `openspec/changes/`, so this change lands in two PRs:

- **Spec-seed PR** (this one): introduces only this proposal. Its verifiable claims are that
  `openspec/changes/unify-data-root-default/proposal.md` exists with numbered success criteria,
  an evidence contract and the eval-section line, and that the gate stays green (no behavior
  change). The numbered criteria above do NOT bind the seed PR.
- **Implementing PR** (branch `agent/12-*`, opened by the lane-G build worker): bound by
  success criteria 1-5 above.
