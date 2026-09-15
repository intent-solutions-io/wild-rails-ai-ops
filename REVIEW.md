# REVIEW.md

Reviewer law for `wild-rails-ai-ops`, consumed by the advisory MiniMax lanes in
`.github/workflows/minimax-review.yml`.

## What this repository is

`wild-rails-ai-ops` is the documentation, governance, audit, and migration
umbrella for Wild. The only active runtime implementation is
[`jeremylongshore/wild`](https://github.com/jeremylongshore/wild): one
pre-release Rails engine and one gem containing ten `Wild::*` namespaces.

The ten original `jeremylongshore/wild-*` repositories are frozen migration
sources. They remain unarchived only until the consolidated engine clears its
release gates and the P4 redirect/archive cutover is authorized. No new runtime
implementation belongs in those repositories or in this umbrella.

This repository contains public claims about topology, maturity, safety,
ownership, and migration. Review those claims for evidence and consistency.

## Review order

1. **Topology regression.** Flag any current-facing text that presents ten
   independently developed or released gems as the active product. The current
   product shape is one `wild` gem with ten namespaces.
2. **Readiness overclaim.** The engine is pre-release. Flag claims that the
   install generator, MCP executables, end-to-end telemetry path, fresh-host
   boot, packaging, or stopwatch adoption gate are ready without new evidence
   from the active implementation repository.
3. **Safety overclaim.** A statement that an operation is read-only, gated,
   redacted, or audited needs implementation and test evidence from
   `jeremylongshore/wild`. This umbrella asserting a property is not proof.
4. **Ownership drift.** Runtime work belongs in `jeremylongshore/wild`;
   ecosystem documentation, audits, and migration tracking belong here. Flag
   links or instructions that send new implementation to a frozen repository.
5. **Migration drift.** Namespace membership and historical-repository mapping
   must agree across `README.md`, `ARCHITECTURE.md`, `CLAUDE.md`, `SECURITY.md`,
   this file, and the automated-review prompts.
6. **Historical-record rewriting.** Dated audits and decisions describe the
   evidence available at a pinned point in time. Correct current-facing
   documents and add later records; do not silently rewrite old evidence.
7. **Governance contradiction.** This umbrella is proprietary; the active
   engine has a separately maintained
   [MIT License](https://github.com/jeremylongshore/wild/blob/e47ee2cfe5ffe729692e4a24703f7dd111e71002/LICENSE);
   external contributions are closed; and security disclosure goes to private
   email. Flag language that collapses those boundaries or invites public
   vulnerability reports.
8. **Sensitive-data leakage.** Flag customer names, internal hostnames,
   tailnet addresses, private filesystem paths, credentials, or real telemetry.
   Never reproduce a suspected secret in a review comment.
9. **Ordinary correctness.** Check links, tables, anchors, Markdown fences,
   workflow syntax, permissions, event boundaries, and action pinning.

## Invariants

- **INV-1 — One implementation:** `jeremylongshore/wild` is the only active
  Wild runtime repository and the only one accepting implementation fixes.
- **INV-2 — One gem, ten namespaces:** the former products now map to ten
  namespaces inside the consolidated engine.
- **INV-3 — Frozen means frozen:** the original repositories receive only the
  authorized redirect/archive cutover, not new feature or defect work.
- **INV-4 — Umbrella boundary:** this repository contains no runtime Ruby code.
- **INV-5 — Evidence leads:** current engine source, tests, release artifacts,
  and implementation documentation lead; this umbrella summarizes them.
- **INV-6 — Dated evidence stays dated:** a commit-pinned cross-repository audit
  may contain implementation citations and reproduction receipts without
  becoming the current runtime source of truth.
- **INV-7 — Maturity is explicit:** no release exists until the active
  repository publishes one and all required adoption gates pass.
- **INV-8 — License boundaries stay explicit:** umbrella proprietary, active
  engine licensed under its separate
  [MIT License](https://github.com/jeremylongshore/wild/blob/e47ee2cfe5ffe729692e4a24703f7dd111e71002/LICENSE),
  historical repositories governed by their own license files.

## Sources reviewers may use

- Current topology summary: `README.md` and `ARCHITECTURE.md`
- Dated cross-repository evidence: `000-docs/000-INDEX.md`
- Work state: Beads and linked GitHub issues
- Runtime behavior and release state: `jeremylongshore/wild`
- Accepted topology decision:
  [`ADR-0001`](https://github.com/jeremylongshore/wild/blob/e47ee2cfe5ffe729692e4a24703f7dd111e71002/000-docs/adr/ADR-0001-topology.md)
- Active-engine license:
  [`LICENSE`](https://github.com/jeremylongshore/wild/blob/e47ee2cfe5ffe729692e4a24703f7dd111e71002/LICENSE)

If a pull request makes a current runtime claim but provides only umbrella
prose, ask for the smallest source, test, run receipt, or release artifact that
supports it. Do not treat a planning document or an open issue as evidence that
behavior shipped.

## Generated or managed surfaces

- Never hand-edit `.beads/issues.jsonl` or `.beads/interactions.jsonl`; use `bd`.
- Never hand-tune `.beads/hooks/*`; those hooks are managed by Beads.
- Do not edit inside generated Beads integration fences in `CLAUDE.md` or
  `AGENTS.md`.
- The automated-review prompts are mirrors of this file. Update this law first,
  then keep both workflow prompts aligned in the same pull request.

## Do not spend comments on

- Pure prose preference, heading style, table alignment, or Oxford commas
- Demanding runtime tests from this documentation-only repository
- Re-litigating the accepted one-engine topology in a routine documentation PR
- Requiring the legacy repositories to be archived before the tracked cutover
  gates pass
- Repeating a finding already resolved by a later push

## Anti-ratchet

On re-review, drop findings that the update resolved and do not raise new
objections on unchanged lines already accepted. Prefer a few high-confidence,
diff-scoped findings. Both review lanes are advisory and must never block a
merge.
