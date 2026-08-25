# REVIEW.md

Reviewer law for `wild-rails-ai-ops`, consumed by the advisory MiniMax lanes in
`.github/workflows/minimax-review.yml`. Report only what the pull request introduces, and verify
every finding against the file it touches.

## What this repository is (read this before reviewing anything)

`wild-rails-ai-ops` is the **umbrella repo** for the wild ecosystem: 10 Ruby gems that run AI agents
inside Rails applications under explicit capability control. **No Ruby code lives here.** Each gem is
an independent repo at `jeremylongshore/wild-*`. What lives here is the ecosystem's public account of
itself: `README.md` (landing page plus the per-repo table), `ARCHITECTURE.md` (four runtime layers,
the dependency graph, the known v1 to v2 gaps), the governance files, and the beads state.

The consequence for a reviewer: **the product of this repo is claims about safety properties.** A
false line in `ARCHITECTURE.md` is not a typo, it is an integrator being told a gate exists that does
not. Review for claim truth first, drift second, hygiene last.

## Top defect classes, in order of risk

1. **A safety property asserted more strongly than the gem delivers.** The four load-bearing claims
   are: `wild-rails-safe-introspection-mcp` is read only (no writes, no console, no admin, bounded
   record reads), every privileged action in `wild-admin-tools-mcp` is gated by
   `wild-capability-gate`, `wild-session-telemetry` redacts PII by default, and gate decisions plus
   privileged actions are audited with enough context to reconstruct them. Flag any diff that widens
   one of these into an absolute ("all", "never", "guaranteed", "cannot"), or that quietly narrows one
   without a dated correction. If a diff describes a new tool on either MCP server, ask in the comment
   which side of the read/write boundary it sits on and which gate decides it.
2. **Planned adoption presented as shipped.** `wild-hook-ops` is extracted but **not yet consumed**:
   the two `*-mcp` gems still carry their own ad hoc HookEmitter classes. `wild-skillops-registry` is
   v0.1.0 and **standalone**: nothing registers with it. Both are v2 targets. Any prose that reads as
   though either is already wired in is a defect, and so is deleting the "Known v1 to v2 adoption
   gaps" section without replacing it with evidence that the gap closed.
3. **Dependency map drift.** `ARCHITECTURE.md` states exactly one hard runtime dependency between
   Layer 1 and Layer 2: `wild-admin-tools-mcp` to `wild-capability-gate`, declared in admin-tools'
   Gemfile. The README ASCII map and the ARCHITECTURE map must agree with each other and with that
   sentence. A new arrow added to one map and not the other is a finding. A new hard dependency added
   in prose without naming the gem whose Gemfile declares it is a finding.
4. **Status table drift against the source of truth.** The per-repo status table in the workspace
   `wild/CLAUDE.md` section 11 is authoritative. The README table here **mirrors** it. A PR that edits
   version, archetype, or mission in the README without saying the workspace table moved first is
   editing a mirror, which is exactly backwards. Same class: a repo renamed or moved between the
   `jeremylongshore` and `intent-solutions-io` orgs in one link and not the others.
5. **Layer and archetype misassignment.** Every gem belongs to exactly one archetype (A product
   facing, B data pipeline, C SDLC companion, D coordination) and one runtime layer. Flag a gem that
   appears under two, an archetype letter changed in the table but not in the layer narrative, or an
   engineer facing Layer 4 tool described as if it ran in the request path.
6. **Leakage into a proprietary public repo.** This repo is public and Intent Solutions Proprietary.
   Flag any host application name, customer name, internal hostname, tailnet address, path under
   `/home/`, token, key, or sample telemetry payload containing real data. **Never reproduce a
   suspected secret in a review comment.** Name the file and line and the remediation only.
7. **Governance contradiction.** `CONTRIBUTING.md` says external contributions are not accepted and
   `SECURITY.md` routes disclosure to email, not a public issue. Flag a diff that invites public
   issues, PRs, or vulnerability reports, or that adds a license or contribution term conflicting with
   the proprietary LICENSE.
8. **Ordinary correctness in what little machinery exists.** Broken relative links, a table row whose
   cell count does not match its header, a fenced block that does not close, an anchor pointing at a
   heading that the same PR renamed, and any YAML or workflow this repo gains later.

## Invariants that must never regress

- **INV-1 Read only means read only.** `wild-rails-safe-introspection-mcp` exposes no write path. Any
  document change implying otherwise is wrong until the gem's own repo says so first.
- **INV-2 One gate.** Every privileged or administrative action reaches `wild-capability-gate` before
  it executes. No second decision point, no direct path around it, no per-tool exemption.
- **INV-3 Every decision is audited.** A gate decision or privileged action that is not recorded does
  not exist. Removing an audit sink from a described flow is a regression, not a simplification.
- **INV-4 Redaction is the default, richer collection is opt in.** Never the reverse.
- **INV-5 The mirror never leads.** The workspace `wild/CLAUDE.md` section 11 table leads, this
  README follows.
- **INV-6 Historical records are corrected, not rewritten.** The 2026-05-28 truth audit and the
  per-repo appaudits describe what was known then. Supersede them with a dated correction; never
  silently edit the past into agreement with the present.
- **INV-7 One umbrella, ten members.** The count and the membership of the ecosystem are the same in
  the README table, the archetype reference, the ARCHITECTURE layers, and `SECURITY.md` scope.

## What "fail closed" means here

For the gems this repo documents: when the capability gate cannot reach its policy source, cannot
evaluate a rule, or cannot write its decision to the audit sink, the correct behavior is to **deny the
tool call**, not to allow it and log a warning. Telemetry that cannot run its redactor **drops the
event** rather than emitting the raw one. Any prose in this repo describing a degraded mode as
permissive ("falls back to allowing", "skips the gate when", "logs and continues") is a top severity
finding even though the code is elsewhere, because this repo is where an integrator reads the
contract.

For this repo's own tooling: an advisory reviewer that errors is simply absent. It must never be the
reason a merge is blocked, and it must never be added to a required check set.

## Generated, mirrored, or otherwise not hand edited

- `.beads/issues.jsonl` and `.beads/interactions.jsonl` are **exports** from the local Dolt database.
  Hand edits are lost on the next flush. Flag any manual edit and direct the change to `bd`.
- `.beads/hooks/*` are installed by `bd init`. Do not hand tune them in a PR.
- The beads integration blocks in `CLAUDE.md` and `AGENTS.md`, fenced by
  `<!-- BEGIN BEADS INTEGRATION -->` and `<!-- END BEADS INTEGRATION -->`, are generated and carry a
  content hash. Edits inside the fence are overwritten. Edit outside it.
- The README per-repo table mirrors the workspace source of truth (INV-5).
- Per-gem implementation detail belongs in each gem's own `000-docs/`, not here. Flag a PR that starts
  duplicating a gem's internals into this umbrella: that creates a second source of truth.

## Do not spend comments on

- Markdown style, heading case, table alignment, line length, or Oxford commas.
- Prose polish, rewording for tone, or "consider adding a section" suggestions.
- Asking for tests or code coverage. There is no code in this repo and no tests to add.
- Restating anything a linter or link checker would catch mechanically.
- Re-litigating the archetype taxonomy, the four layer model, the proprietary license, or the closed
  contribution policy. Those are settled.
- Demanding that a v2 gap be closed in this PR. Naming the gap accurately is the requirement.

## Anti-ratchet

On a re-review after new pushes the bar does not rise. Drop findings the update resolved, and do not
raise new objections on unchanged lines you already accepted. Prefer a few high conviction findings
over breadth. If the change is truthful, consistent across both maps and the table, and leaks nothing,
reply `lgtm`. Both lanes are advisory and neither blocks a merge.
