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
   neither `*-mcp` gem declares it as a dependency, and neither carries hook emitter code at all. The
   emitter each one would eventually migrate is a design sketch in that gem's own docs, not a shipped
   class: `WildAdminToolsMcp::Telemetry::HookEmitter` is recorded as "Defined (conceptual, not
   coded)", and the introspection gem's counterpart is named `Telemetry::Emitter` and carries the
   status "Planned, interface definition only, not yet implemented". `wild-skillops-registry` is
   v0.1.0 and **standalone**: nothing registers with it. Both are v2 targets. Any prose that reads as
   though either is already wired in is a defect, and so is deleting the "Known v1 to v2 adoption
   gaps" section without replacing it with evidence that the gap closed.
3. **Dependency map drift.** `ARCHITECTURE.md` states exactly one hard runtime dependency between
   Layer 1 and Layer 2: `wild-admin-tools-mcp` to `wild-capability-gate`. That sentence attributes the
   declaration to admin-tools' Gemfile, which is imprecise: the runtime declaration is
   `spec.add_dependency 'wild-capability-gate', '~> 0.1'` in the gemspec, and the Gemfile only pins
   where that gem is fetched from. Check the gemspec, not the Gemfile, when deciding whether a hard
   dependency is real. The README ASCII map and the ARCHITECTURE map must agree with each other and
   with that sentence. A new arrow added to one map and not the other is a finding. A new hard
   dependency added in prose without naming the gemspec that declares it is a finding.
4. **Status table drift against the source of truth.** The per-repo status table in the workspace
   `wild/CLAUDE.md` section 11 is authoritative. The README table here **mirrors** it. A PR that edits an
   archetype letter, a mission line, or a version written into one of those cells without saying the
   workspace table moved first is editing a mirror, which is exactly backwards. Same class: a repo renamed or moved between the
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
- **INV-3 Every decision is audited, and audit is a sink, not a veto.** A gate decision or privileged
  action that is not recorded does not exist as a reviewable event, and removing an audit sink from a
  described flow is a regression, not a simplification. Scope it accurately: when an audit writer is
  configured the gate emits an event for every evaluation, but a failing audit write is deliberately
  swallowed so that a broken log cannot take the gate down. Do not read this invariant as "an
  unwritable audit sink denies the call". It does not, and prose in this repo claiming it does is
  itself the defect.
- **INV-4 Redaction is the default, richer collection is opt in.** Never the reverse.
- **INV-5 The mirror never leads.** The workspace `wild/CLAUDE.md` section 11 table leads, this
  README follows.
- **INV-6 Historical records are corrected, not rewritten.** The 2026-05-28 truth audit and the
  per-repo appaudits describe what was known then. Supersede them with a dated correction; never
  silently edit the past into agreement with the present.
- **INV-7 One umbrella, ten members.** The count and the membership of the ecosystem are the same in
  the README table, the archetype reference, the ARCHITECTURE layers, and `SECURITY.md` scope.

## What "fail closed" means here

For the gems this repo documents, fail closed has three legs and they are not the same shape:

- **Cannot evaluate a rule: deny.** `Wild::CapabilityGate::Gate#evaluate` never raises. Any exception
  raised during evaluation is converted into a denial carrying the reason `:evaluation_error`.
- **Cannot reach its policy source: refuse to start.** A broken config raises out of
  `Gate#initialize` instead of being swallowed later, so no half configured gate ever answers a call.
  That is fail closed at startup, not a per call denial, and prose describing it as a per call denial
  is inaccurate.
- **Cannot write its decision to the audit sink: the decision stands.** The audit write is
  deliberately non fatal, so a broken log does not turn an allow into a deny. This is the one leg
  where shipped behavior is permissive on purpose. Do not flag documentation that states this
  correctly, and do not accept a diff that restates it as a denial.

Telemetry that cannot run its privacy filter **drops the event** rather than emitting the raw one:
the receiver filters before it validates and converts any exception to `nil`, so a filter failure
never reaches the store.

Any prose in this repo describing the *gate decision* path as permissive ("falls back to allowing",
"skips the gate when", "logs and continues") is a top severity finding even though the code is
elsewhere, because this repo is where an integrator reads the contract.

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

## Sources

Every code grounded claim above was opened and read at these commits: this repo at `c8ca364` (the
branch head this section was written against), and the gem repos at
`wild-rails-safe-introspection-mcp@49134a3`, `wild-admin-tools-mcp@572eda1`,
`wild-capability-gate@526ef21`, `wild-session-telemetry@ee646bb`, `wild-hook-ops@9f72fbc`,
`wild-skillops-registry@8f3d165`. Paths without a repo prefix are in this repo. A claim that is
documented here but enforced nowhere is cited to the document that asserts it, not to code, and is
labeled as such.

### What this repository is

- Umbrella, no Ruby, 10 members: `CLAUDE.md:8-9`, `CLAUDE.md:20-21`, `README.md:3-5`,
  `README.md:13-22` (ten table rows)
- Four runtime layers, dependency graph, v1 to v2 gaps: `ARCHITECTURE.md:11-39`,
  `ARCHITECTURE.md:41-50`, `ARCHITECTURE.md:52-58`
- Gems live at `jeremylongshore/wild-*`: `README.md:13-22`, `CLAUDE.md:20`

### Defect class 1, the four load bearing safety claims

- Read only introspection, asserted: `ARCHITECTURE.md:15`, `README.md:13`
- Read only introspection, enforced: `wild-rails-safe-introspection-mcp/lib/wild_rails_safe_introspection/adapter/write_prevention.rb:6-43`
  (forbidden writer methods plus a write SQL pattern), three tools and only three:
  `.../lib/wild_rails_safe_introspection/server/server_factory.rb:6-10`
- Every privileged admin action gated, asserted: `ARCHITECTURE.md:16`
- Every privileged admin action gated, enforced: `wild-admin-tools-mcp/lib/wild_admin_tools_mcp/identity/authenticated_pipeline.rb:12-25`
  (anonymous rejected, gate consulted, denial short circuits execution) and
  `.../lib/wild_admin_tools_mcp/identity/gate_client.rb:10-27` (unconfigured gate raises `GateError`)
- Telemetry redacts by default, asserted: `ARCHITECTURE.md:66`
- Telemetry redacts by default, enforced: `wild-session-telemetry/lib/wild_session_telemetry/configuration.rb:10`
  (`@privacy_mode = :strict`), `.../lib/wild_session_telemetry/collector/event_receiver.rb:9` (the
  filter is the default, not an opt in), `.../lib/wild_session_telemetry/privacy/filter.rb:6-29`
  (allowlist of top level keys and per event metadata, plus a forbidden field list)
- Decisions audited, asserted: `ARCHITECTURE.md:64`, `ARCHITECTURE.md:67`
- Decisions audited, enforced: `wild-capability-gate/lib/wild/capability_gate/evaluator.rb:102-116`
  (emitted after every evaluation when a writer is configured; write failure swallowed, see INV-3)

### Defect class 2, extracted is not adopted

- Neither `*-mcp` gem depends on hook-ops: `wild-admin-tools-mcp/wild-admin-tools-mcp.gemspec:22`
  and `wild-admin-tools-mcp/Gemfile:1-20` name only `wild-capability-gate`;
  `wild-rails-safe-introspection-mcp/wild-rails-safe-introspection-mcp.gemspec:20-22` names
  `activerecord`, `mcp`, `yaml` and nothing wild
- No hook emitter code in either gem, only a design sketch:
  `wild-admin-tools-mcp/000-docs/018-AT-ADEC-telemetry-emission-hook-interface.md:41` (the class
  sketch) and `:178` ("HookEmitter interface | Defined (conceptual, not coded)");
  `wild-rails-safe-introspection-mcp/000-docs/019-AT-ADEC-telemetry-emission-hook-interface.md:5`
  ("Planned, interface definition only, not yet implemented") and `:133-138` (`Telemetry::Emitter`)
- Hook-ops shipped but unadopted, in its own words:
  `wild-hook-ops/000-docs/007-AT-AUDT-appaudit-2026-05-28.md:8` and `:103`
- Skillops standalone at v0.1.0: `wild-skillops-registry/lib/wild_skillops_registry/version.rb:4`,
  `wild-skillops-registry/000-docs/007-AT-AUDT-appaudit-2026-05-28.md:159` and `:163`
- The gaps section this class protects: `ARCHITECTURE.md:52-58`

### Defect class 3, the one hard dependency

- The sentence: `ARCHITECTURE.md:43`
- The actual runtime declaration: `wild-admin-tools-mcp/wild-admin-tools-mcp.gemspec:22`
- The Gemfile, which pins the source only: `wild-admin-tools-mcp/Gemfile:7-13`
- The two maps that must agree: `README.md:37-59`, `ARCHITECTURE.md:45-50`

### Defect classes 4 through 7

- Workspace table leads, README mirrors: `CLAUDE.md:26`, `CLAUDE.md:30-32`
- Archetype letters and their meanings: `README.md:26-31`
- One archetype and one layer per gem, cross checked row by row: `README.md:13-22` against
  `ARCHITECTURE.md:11-39` (all ten agree)
- Layer 4 is engineer facing, not request path: `ARCHITECTURE.md:33-35`
- Public and proprietary: `README.md:69`, `LICENSE`
- External contributions closed: `CONTRIBUTING.md:3`
- Disclosure by email, not a public issue: `SECURITY.md:5`, `SECURITY.md:7`

### Invariants

- INV-1: `.../adapter/write_prevention.rb:6-43`, `.../server/server_factory.rb:6-10`
- INV-2: `.../identity/authenticated_pipeline.rb:12-25`, `.../identity/gate_client.rb:10-27`.
  Scope note: the introspection gem carries its own in repo
  `WildRailsSafeIntrospection::Identity::CapabilityGate`
  (`.../lib/wild_rails_safe_introspection/identity/capability_gate.rb:18-41`), a v1 stub that permits
  every authenticated caller. It is not the gate gem and it guards no privileged action, because that
  gem has no write path. INV-2 governs privileged and administrative actions, which live in
  admin-tools.
- INV-3: `wild-capability-gate/lib/wild/capability_gate/evaluator.rb:102-116`
- INV-4: `wild-session-telemetry/lib/wild_session_telemetry/configuration.rb:10`,
  `.../collector/event_receiver.rb:9`
- INV-5: `CLAUDE.md:30-32`
- INV-6: the audit it points at, `CLAUDE.md:36`; the per gem appaudits it points at, one per gem, for
  example `wild-capability-gate/000-docs/012-AT-AUDT-appaudit-2026-05-28.md` and
  `wild-skillops-registry/000-docs/007-AT-AUDT-appaudit-2026-05-28.md` (the file
  `ARCHITECTURE.md:57` cites by name; it exists at that exact path)
- INV-7: `README.md:13-22`, `ARCHITECTURE.md:11-39`, `SECURITY.md:20`

### Fail closed

- Cannot evaluate a rule, denies with `:evaluation_error`:
  `wild-capability-gate/lib/wild/capability_gate/gate.rb:36-44` and `:63-69`
- Cannot reach its policy source, raises at construction:
  `wild-capability-gate/lib/wild/capability_gate/gate.rb:27-33`
- Audit write failure is non fatal: `wild-capability-gate/lib/wild/capability_gate/evaluator.rb:102-116`
- Telemetry drops what it cannot filter:
  `wild-session-telemetry/lib/wild_session_telemetry/collector/event_receiver.rb:12-22`
- Admin-tools converts a gate error into a denial:
  `wild-admin-tools-mcp/lib/wild_admin_tools_mcp/identity/authenticated_pipeline.rb:33-37`

### Generated, mirrored, or otherwise not hand edited

- JSONL is a passive export of the Dolt database: `CLAUDE.md:65`, `.beads/.gitignore:1-3`
- Hooks are installed and managed by beads: `.beads/hooks/pre-commit:2-3` (the same managed marker
  appears in each of the five hooks)
- Fenced beads blocks carry a content hash: `CLAUDE.md:45` and `:91`, `AGENTS.md:50` and `:96`
  (both `hash:7510c1e2`)

### The review lane itself

- Advisory, `pull_request` only, same repo guard, kill switch:
  `.github/workflows/minimax-review.yml:49-53`, `:76-78`, `:186-189`
- No required check exists to be added to: `main` has no branch protection on this repository, so
  nothing is required today
- The action reads the diff through the GitHub API and never checks out or runs PR code:
  `jeremylongshore/minimax-code-review@d1314b96c1b261d5bf026d0a669823a7e5ce6b46`,
  `src/index.js:54` (`octokit.rest.pulls.listFiles`) and `action.yml:35-37` (a `node20` action with a
  single `dist/index.js` entry point, no checkout and no shell step)
- The three fork specific behaviors the workflow header claims: per reviewer sticky marker
  `src/index.js:14-27` and `:257-258`, reasoning `<think>` stripping `src/index.js:175-191`, and
  `INCLUDE_PR_BODY` `action.yml:27-30` and `src/index.js:206`
- The secret and variables the header says are already set: `MINIMAX_API_KEY` exists, and
  `ENABLE_MINIMAX_REVIEW=true` and `MINIMAX_MODEL=MiniMax-M3` are set on this repository (checked via
  the repository Actions secrets and variables API, names and values of variables only)

### Open discrepancies found while verifying, not fixed here

These are real findings in files this pull request does not touch. They are recorded rather than
silently corrected, because correcting them belongs in a diff of their own.

- `README.md:65` says "All 10 repos are v1 (or v0.1.0 for `wild-skillops-registry`)". Every gem
  checked reports `VERSION = '0.1.0'` and the published tags are `v0.1.0`. The singling out of
  skillops is wrong: nothing has reached v1.
- `README.md:65` links the status table to this repo's `CLAUDE.md#11-per-repo-status-table`. This
  repo's `CLAUDE.md` has no section 11 and no status table; the table it means is the workspace
  `wild/CLAUDE.md` (`CLAUDE.md:32`), which is not a link a public reader can follow.
- `CLAUDE.md:22` says the repo has "No CI workflows beyond markdown lint and link check". Neither
  exists, and before this pull request `.github/workflows/` did not exist at all.
- `ARCHITECTURE.md:15` glosses the three tools as "model reflection, schema inspection, bounded
  record read". The three registered tools are `InspectModelSchema`, `LookupRecordById` and
  `FindRecordsByFilter`: one schema tool and two record reads. `ModelReflector` is an internal
  adapter, not a tool.
