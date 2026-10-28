# Proposal

> Spec for GitHub issue #14 (pilot). Transcribed 1:1 from the issue body by the pipeline
> shepherd — scope decisions are the issue's, not this PR's.

## Why

AGENTS.md documents the manifest, entrypoint kwargs, hooks and seal flow well, but not the
read API every strategy lives on. Writing a first tenant that actually prices something means
reverse-engineering from framework source: `store.universe()`'s tuple shape, `store.route()`
and the `DataAtT` fields (`.book.best_ask`, `.mid`, degradation tier), and the `Order`
dataclass. Measured cost: minutes with an AI assistant reading the source, the bulk of hours
unaided.

## What Changes

- AGENTS.md gains a ~30-line "What a strategy reads and returns" section: the `universe()`
  tuple `(market_id, token_id, first_ts, last_ts, n_snaps)`, `route(market_id, token_id, ts)`
  -> `DataAtT` with its fields and degradation tiers, and the `Order` constructor shown via
  the sample tenant's canonical call.
- `docs/running.md` mirrors or links the same snippet.
- Docs only: no code, no config, no behavior change.

## Success criteria (bind the implementing PR, branch `agent/14-*`)

1. AGENTS.md contains a "What a strategy reads and returns" section covering
   `store.universe()` (tuple shape), `store.route()` -> `DataAtT` (fields incl.
   `.book.best_ask`, `.mid`, and the degradation tier), and the `Order` constructor shown via
   the sample tenant's canonical call.
2. Every field name and signature in the snippet matches the code in `desk/data/store.py` and
   `desk/types.py` at the PR's head SHA — a reader can write a working pricing entrypoint from
   AGENTS.md alone, without opening either source file.
3. `docs/running.md` mirrors or links the same snippet.
4. The offline golden gate stays green on the head SHA (`python3 -m desk.verify gate --json`,
   = `make eval-fast`) — docs-only, no behavior change.

## Evidence contract (what the implementing PR must show)

- The PR template's gate block: full `gate --json` output plus the exact head SHA it ran on.
- Criteria map: #1 and #3 point at the diff hunks (file + section anchor); #2 points at the
  matching definitions in `desk/data/store.py` / `desk/types.py` so the verifier confirms the
  names field-for-field against the tree, never against the author's claim; #4 is the gate
  block itself.

## Eval gate sections

Eval gate section(s) added or changed: none — docs-only change with no behavior and no new
checks; acceptance rides on criterion #2 (snippet-vs-source fidelity), which the spec-verifier
re-derives directly from the tree.

## Delivery note (two PRs)

The public tree ships without `openspec/changes/`, so this change lands in two PRs:

- **Spec-seed PR** (this one): introduces only this proposal. Its verifiable claims are that
  `openspec/changes/doc-strategy-read-api/proposal.md` exists with numbered success criteria,
  an evidence contract and the eval-section line, and that the gate stays green (no behavior
  change). The numbered criteria above do NOT bind the seed PR.
- **Implementing PR** (branch `agent/14-*`, opened by the lane-G build worker): bound by
  success criteria 1-4 above.
