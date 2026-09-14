# Wild Ecosystem Deep Code Review

**Document ID:** 001-AA-AUDT

**Date:** 2026-09-14

**Owner:** Jeremy Longshore / Intent Solutions

**Scope:** `jeremylongshore/wild`, the ten frozen `jeremylongshore/wild-*`
repositories, and the `intent-solutions-io/wild-rails-ai-ops` umbrella

**Method:** AppAudit, repository sweep, test-policy audit, adversarial source
review, focused runtime reproductions, Git/GitHub inspection

**Tracking:** Bead `wild-rails-ai-ops-8l5` and children `.1` through `.11`

**Verdict:** Core libraries are substantially hardened; product adoption and
release remain blocked.

## 1. System in five minutes

Wild is no longer an estate of ten independently developed gems. On 2026-05-29,
a seven-seat architecture council rejected that topology and selected one Rails
engine gem, `jeremylongshore/wild`, containing ten `Wild::*` namespaces. The
workspace records both the decision and namespace mapping at
`wild/CLAUDE.md:109-141`, and explicitly says that only `wild/wild/` accepts code
commits at `wild/CLAUDE.md:145-161`.

The active codebase is therefore:

```text
Rails host
  |
  `-- wild gem / Wild::Engine
       |-- Wild::Introspection
       |-- Wild::AdminTools ---- Wild::CapabilityGate
       |-- Wild::Telemetry::Collector
       |     `-- Wild::Telemetry::Pipeline
       |           `-- Wild::Telemetry::Analysis
       |-- Wild::Hooks
       |-- Wild::Analyzers::Permission
       |-- Wild::Analyzers::TestFlakes
       `-- Wild::Skillops (disabled by default)
```

Today, the in-process Ruby libraries are the working product. The adoption
surface is not. The public engine README accurately says that the install
generator, two MCP binaries, prompts, stopwatch test, first release, and legacy
archive are unfinished (`wild/README.md:12-22`). The generator only prints a
pending message (`wild/lib/generators/wild/install/install_generator.rb:18-25`),
both MCP binaries exit with status 1 (`wild/bin/wild-mcp-introspection:11-14`,
`wild/bin/wild-mcp-admin:11-14`), and the gem declares no executables
(`wild/wild.gemspec:43-47`).

The umbrella repository tells the superseded story. Its README calls Wild ten
independent gems (`README.md:3-5`), its architecture describes a multi-gem
runtime (`ARCHITECTURE.md:1-50`), and its agent/reviewer instructions insist
that no Ruby code exists centrally (`CLAUDE.md:7-22`, `REVIEW.md:7-17`). That is
not a harmless stale paragraph: it is the public integration contract and the
automated reviewer law.

### Immediate operating rule

Do not release `wild`, enable Skillops, or resume implementation in the ten
legacy repositories. Repair the P0/P1 findings centrally, complete the
adoption path, pass the five-minute gate, then redirect and archive the legacy
repositories.

## 2. Executive assessment

### 2.1 What changed during the apparent four-month gap

The project was not untouched for four continuous months. The consolidated
repository has 107 commits: 43 in May, 7 in June, none in July, 57 in August,
and none in September before this audit. The original build paused mid-P1 on
2026-06-02. A substantial review/fix wave completed on 2026-08-26 and repaired
important telemetry redaction, retention locking, audit-shape, gate-rescue,
nonce-atomicity, and CI defects. The after-action status records the wave and
its prior 3,325-spec receipt at `wild/000-docs/006-OD-STAT-status.md:24-44`.

This review therefore evaluates the post-wave code, not the older May
snapshots, while using the ten frozen repositories to determine whether
behavior was correctly migrated.

### 2.2 Current health

| Dimension | Score | Assessment |
|---|---:|---|
| Core implementation | 82/100 | Cohesive namespaces, strong parameterization, validation, redaction, and focused tests |
| Test execution | 90/100 | Large deterministic suite and clean static/security gates; important integration gaps remain |
| Security design | 72/100 | Strong default boundaries in several paths, but audit liveness and error-disclosure policy are unresolved |
| Product adoption | 32/100 | Generator, transports, prompts, real engine route, and stopwatch journey are unfinished |
| Release engineering | 28/100 | Artifact is overbroad and workflow can release from the wrong ref without readiness proof |
| Documentation truth | 25/100 | Umbrella and several status/test documents materially contradict current topology and code |
| Operational hygiene | 48/100 | Protected central main, but stale issues, branches, worktrees, Dependabot failures, and unarchived repos |
| **Overall** | **66/100** | **Promising pre-release library; not yet an installable, supportable product** |

### 2.3 Release disposition

**NO-GO.** Two reproduced P0 product blockers and multiple P1 safety/correctness
defects remain. No tag or GitHub release exists, and version 0.0.1 remains the
right signal. The report does not recommend weakening any gate to create a
release.

### 2.4 Positive baseline

The result is not a condemnation of the codebase. The central implementation
has several unusually strong properties:

- Introspection uses authentication, model and column allowlists,
  parameterized lookups, result filtering, row ceilings, and time ceilings.
- AdminTools has a coherent external trust chain: authentication, per-action
  capabilities, allowlists, validation, rate limiting, blast checks, atomic
  nonce consumption, two-phase confirmation, and audited wrapping.
- Telemetry content and metadata redaction are substantive, not placeholders.
- Collector JSONL storage now uses process and cross-process locking plus
  atomic, fsync-backed rewrites.
- CI currently enforces RSpec, RuboCop, Packwerk, Brakeman, Bundler Audit, and
  CodeQL through a fan-in check.
- The central README is candid about what does and does not work.

Those strengths justify finishing the product rather than restarting it.

## 3. Verified baseline

The following commands were run against clean central source at commit
`e47ee2c` on 2026-09-14.

| Verification | Result |
|---|---|
| `bundle check` | Dependencies satisfied |
| `bundle exec rspec` | 3,327 examples, 0 failures, seed 13205 |
| SimpleCov persisted result | 6,360 / 6,471 lines, 98.28% |
| `bundle exec rubocop` | 508 files, 0 offenses |
| `bundle exec packwerk validate` | Configuration valid |
| `bundle exec packwerk check` | 218 files, 0 offenses |
| `bundle exec brakeman ...` | 0 warnings, 0 errors |
| `bundle exec bundler-audit check --update` | 0 vulnerabilities |
| `./scripts/audit-harness verify` | Reported `harness-hash: OK` |

All available legacy RSpec suites except one passed. Session Telemetry produced
six failures because hard-coded dates aged beyond the 90-day retention window;
the central suite replaced those fixtures. Exact legacy totals appear in the
repository assessments below. All ten legacy RuboCop runs that could resolve
dependencies were clean. Gitleaks found no findings in the five Group B
histories. These results are evidence of a strong unit baseline, not proof of
end-to-end compatibility.

## 4. Severity-ranked findings

### F-01 — P0: a normal Rails host cannot boot with unused AdminTools

The engine always bridges AdminTools after initialization
(`wild/lib/wild/engine.rb:47`). Central configuration creates default AdminTools
adapter sentinels (`wild/lib/wild/configuration.rb:81-96`), and the bridge
validates them in `wild/lib/wild/admin_tools.rb:136-180`.

A fresh-host reproduction returned:

```text
Wild::ConfigurationError: Wild.config.admin_tools.job_adapter is :default but no backend gem is loaded...
exit=42
```

The dummy app masks this condition by installing abstract stub job and feature
flag adapters (`wild/spec/dummy/config/application.rb:20-34`). That contradicts
the flagship journey, which says safe defaults leave admin tools disabled and
the app boots without errors (`wild/000-docs/004-PP-UJRN-user-journey.md:15-24`).

**Impact:** the advertised first-run journey fails before the user invokes any
Wild feature. This blocks the generator, stopwatch test, and release.

**Required fix:** add a real `admin_tools.enabled` defaulting to false, skip
validation and bridging while disabled, validate only adapters needed by an
enabled surface, and add a fresh Rails 7.1 host boot spec without dummy stubs.

**Tracking:** `wild-rails-ai-ops-8l5.1`.

### F-02 — P0: Collector output is incompatible with Analysis

Collector does not emit the `source_id` that Analysis requires. The producer
header is built at `wild/lib/wild/telemetry/collector/export/record_builder.rb:11-18`
and emitted at `.../export/exporter.rb:62-73`. The consumer rejects a missing
field at `wild/lib/wild/telemetry/analysis/models/export_header.rb:11-22` and
`.../ingestion/export_parser.rb:51-55`.

A direct producer-to-consumer probe returned:

```text
Wild::Telemetry::Analysis::SchemaError: export header missing required fields
```

There is a second contract mismatch. Collector emits outcomes as objects
containing `count` and `percentage`
(`.../collector/aggregation/engine.rb:44-54`). Analysis returns that entire
object (`.../analysis/models/outcome_distribution.rb:10-20`), then compares it
to a float (`.../analysis/analyzers/denial_analyzer.rb:25-27`). The runtime
probe raised:

```text
TypeError: no implicit conversion of Float into Hash
value={"count"=>1,"percentage"=>0.5}
```

The green model specs use flattened fixtures rather than real collector output
(`wild/spec/wild/telemetry/analysis/models/outcome_distribution_spec.rb:15-36`).

**Impact:** the telemetry analysis product cannot analyze its own real export.

**Required fix:** define a single versioned export schema, make `source_id` an
explicit producer input, define the outcome representation once, and add a
Collector -> JSONL -> Analysis round-trip using only production serializers.

**Tracking:** `wild-rails-ai-ops-8l5.3`, related to prior Beads
`wild-rails-ai-ops-cs3` and `wild-rails-ai-ops-llx`.

### F-03 — P1: an AdminTools mutation can succeed without an audit record

`AdminTools::Audit::Recorder#record` executes `yield` before appending its
success record (`wild/lib/wild/admin_tools/audit/recorder.rb:19-31`). If append
fails, rescue attempts another append through the same failing store and
re-raises. A failing-store reproduction produced:

```text
reported=IOError:disk full mutations=1
```

The mutation happened once, no record persisted, and the caller received a
failure that could invite a duplicate retry. Existing recorder tests cover an
action exception but not audit-store failure
(`wild/spec/wild/admin_tools/audit/recorder_spec.rb:145-171`). The behavior
contradicts the module's audit-before-mutation claim
(`wild/lib/wild/admin_tools.rb:13-18`).

**Required fix:** persist a durable intent before execution and deny if that
write fails; correlate an idempotent completion/failure record; if completion
persistence fails after mutation, surface a specifically non-retryable
`indeterminate_after_execution` result and durable recovery signal.

**Tracking:** `wild-rails-ai-ops-8l5.2`.

### F-04 — P1: Skillops lifecycle and version integrity are not trustworthy

The Skill model turns any lifecycle string into a symbol without validating an
allowed set (`wild/lib/wild/skillops/models/skill.rb:26-39`); `:wat` was
accepted. Registrar updates can change lifecycle and capabilities outside the
LifecycleManager, while change computation omits those fields
(`wild/lib/wild/skillops/registry/registrar.rb:28-115`). A retired skill was
updated back to active with different capabilities, yet its version count
remained `1 -> 1`.

VersionManager owns snapshots separately
(`wild/lib/wild/skillops/versioning/version_manager.rb:8-46`), while export
serializes `RegistryEntry#versions`
(`wild/lib/wild/skillops/models/registry_entry.rb:8-40`). After registration,
the manager reported one snapshot and export reported zero versions. Skillops
is disabled by default, which contains current exposure but does not make it
safe to enable.

**Required fix:** validate lifecycle at the model boundary; route every
transition through one service; version lifecycle and capability changes; use
one canonical snapshot history; make registration transactional or explicitly
recoverable.

**Tracking:** `wild-rails-ai-ops-8l5.4`.

### F-05 — P1: three migrated analyzers retain correctness defects

1. **Hook registration race.** Duplicate checking and insertion happen in
   separate critical sections (`wild/lib/wild/hooks/registry/definition_store.rb:13-19`).
   Two same-name concurrent registrations can both pass validation and one can
   overwrite the other. Tests cover only serial duplication.
2. **Partial permission audits.** Capabilities and grants loaders silently
   discard malformed rows through compacting logic
   (`wild/lib/wild/analyzers/permission/loaders/capabilities_loader.rb:18-62`,
   `.../grants_loader.rb:18-55`). A security analyzer can therefore report clean
   results after omitting the evidence it could not parse.
3. **Invalid flake rates.** Flake detection counts rows, not unique executions
   (`wild/lib/wild/analyzers/test_flakes/detection/flake_detector.rb:26-37`,
   `.../models/flake_record.rb:21-42`). Three contradictory rows with one
   `run_id` were reported as three runs and one flake. Generated run IDs use
   second-level time resolution (`.../parsers/base.rb:27-29`), creating further
   collisions.

**Required fix:** make hook check-and-insert one atomic operation; make malformed
security rows fatal or prominent in every report; normalize flake observations
by an explicit collision-resistant execution ID.

**Tracking:** `wild-rails-ai-ops-8l5.5`.

### F-06 — P1: the gem artifact leaks internal operational material

The gemspec builds `spec.files` from a broad `git ls-files` result and rejects a
small list (`wild/wild.gemspec:35-40`). A local gem build succeeded but packaged
316 files, including `.env.sops`, `.sops.yaml`, `.claude/settings.json`,
`.beads/`, `.beads-hooks/`, `.audit-harness/`, agent instructions, test-audit
documents, and internal scripts. The encrypted SOPS file is not a plaintext
secret, but it and the other operational artifacts do not belong in a runtime
gem.

**Impact:** unnecessary disclosure, avoidable supply-chain surface, confusing
consumer installs, and no deterministic evidence that the release artifact is
the intended product.

**Required fix:** use an allowlist for runtime files, licenses, and intentionally
published documentation; unpack and assert the manifest in CI; generate SBOM
and provenance from the exact verified artifact.

**Tracking:** `wild-rails-ai-ops-8l5.6`.

### F-07 — P1: release automation does not enforce its own release law

The workflow says it should run only after the stopwatch and council fixes are
closed (`wild/.github/workflows/release.yml:3-5`). Its readiness step runs
ordinary code gates only (`:57-72`). It does not verify main, current protected
HEAD, generator behavior, MCP transports, prompts, artifact manifest,
stopwatch, or release-blocking Beads. It then commits, tags, and pushes the
selected workflow ref (`:127-172`) and creates a GitHub release (`:174-188`).
There is no protected environment or approver.

**Impact:** a manual dispatch can manufacture a release from a non-main branch
while the advertised adoption path is still stubbed.

**Required fix:** require `refs/heads/main`, equality with protected
`origin/main`, a protected release environment, machine-readable readiness
manifest, artifact inspection, stopwatch test, and zero open release blockers.

**Tracking:** `wild-rails-ai-ops-8l5.6`.

### F-08 — P1: claimed Rails support is not tested

The gem declares Ruby 3.2+ and Rails 7.1+ without an upper bound
(`wild/wild.gemspec:24`, `:50-52`). The lock resolves Rails 8.1.3.1, and CI
varies Ruby 3.2/3.3/3.4 but not Rails
(`wild/.github/workflows/ci.yml:96-115`). `config.load_defaults 7.1` does not
test against Rails 7.1.

**Required fix:** add Appraisal-style Rails 7.1, 8.0, and 8.1 lanes, including
a real fresh-host boot. State the supported Ruby/Rails matrix and remove any
combination that cannot be continuously proven.

**Tracking:** `wild-rails-ai-ops-8l5.7`.

### F-09 — P1: audit liveness and durable audit configuration are unresolved

CapabilityGate intentionally lets an ALLOW survive audit-writer failure
(`wild/lib/wild/capability_gate/evaluator.rb:201-248`), and the spec pins that
behavior (`wild/spec/wild/capability_gate/audit/audit_liveness_spec.rb:256-355`).
That decision is still awaiting owner acceptance under central Bead `wild-0c3`.

Introspection constructs its capability gate with only a config path
(`wild/lib/wild/introspection/identity/capability_gate.rb:64-71`); the gate only
creates an audit writer when given `audit_log_path`
(`wild/lib/wild/capability_gate/gate.rb:35-39,75-81`). Its separate tool audit
warns and drops by default (`wild/lib/wild/introspection/audit/audit_logger.rb:11-18`).
AdminTools' factory defaults to volatile MemoryStore
(`wild/lib/wild/admin_tools/server/server_factory.rb:22-25`). Capability and
admin JSONL writers also lack the collector store's cross-process locking and
fsync treatment.

**Impact:** public claims about reconstructable decisions and actions outrun
the configured default behavior.

**Required fix:** make the audit-liveness posture an explicit, owner-approved
policy; ensure privileged release paths are not silently dark; expose one
configuration surface for authentication and durable audit destinations; add
failure-path tests under process and storage faults.

**Tracking:** `wild-rails-ai-ops-8l5.10`, related to
`wild-rails-ai-ops-zgd` and central `wild-0c3`.

### F-10 — P1: umbrella documentation and reviewer law enforce rejected architecture

The umbrella README describes ten active, independently released repositories
(`README.md:3-5`, `:11-22`, `:63-65`) and its dependency graph represents the
superseded runtime (`README.md:35-59`). Architecture makes the same claim
(`ARCHITECTURE.md:1-50`). CLAUDE and REVIEW tell agents that the ten-gem model
is authoritative (`CLAUDE.md:7-32`, `REVIEW.md:7-50`). MiniMax embeds that model
again in its prompts (`.github/workflows/minimax-review.yml:93-184`).

The actual decision is unambiguous: one gem, ten namespaces
(`wild/CLAUDE.md:109-141`), with old repositories frozen and only the central
repo accepting code (`wild/CLAUDE.md:145-161`). The locked plan says to archive
and redirect rather than leave the old repos as-is
(`build-orchestration/README.md:9-23`).

**Impact:** users, maintainers, issue reporters, and the automated reviewer are
directed toward the wrong repositories and architecture.

**Required fix:** make the umbrella a current landing/cutover page led by
`jeremylongshore/wild`; retain the old graph only as dated history; update
agent/reviewer/security scope; migrate live issue intent centrally; archive
all ten repos at P4 with prominent redirect READMEs.

**Tracking:** `wild-rails-ai-ops-8l5.9`.

### F-11 — P2: test governance no longer describes the executable suite

The code gates are green, but the evidence documents contradict the source and
one another:

- `wild/tests/TESTING.md:5-12,27` calls the policy stale and says namespace work
  remains placeholder work.
- Its mutation, property, CRAP, and integration waivers depend on placeholder
  code and no dummy app (`wild/tests/TESTING.md:62-65`); neither premise remains
  true.
- `wild/tests/RTM.md:15-39` says 12 of 14 MUST requirements are uncovered and
  labels implemented namespaces/configuration as skeletons.
- `wild/tests/RTM.md:86` says no other tests exist although 256 spec files do.
- `wild/TEST_AUDIT.md:5-8` reports 3,325 examples and 507 linted files, versus
  the current 3,327 and 508.
- The repository-wide coverage threshold is 85%, but the per-file 75% floor is
  commented out on obsolete placeholder grounds
  (`wild/spec/spec_helper.rb:34-40`). Codecov upload is nonblocking
  (`wild/.github/workflows/ci.yml:117-124`) and component statuses are not
  required by branch protection.
- `.harness-hash` is empty. The harness reports it as OK and reports no
  architecture tool configured, making those two checks weak evidence.

**Required fix:** rebuild RTM and policy from current requirements and real
tests, decide rather than inherit advanced-test waivers, and either make chosen
per-file/component thresholds required or record the owner-approved exception.

**Tracking:** `wild-rails-ai-ops-8l5.8`, related to
`wild-rails-ai-ops-wmd`.

### F-12 — P2: Packwerk has a known enforcement hole

All ten namespace entry files are excluded from Packwerk
(`wild/packwerk.yml:73-85`). The configuration itself explains that constants
and cross-boundary calls through those files are invisible to CI
(`wild/packwerk.yml:53-68`). Bridge and factory behavior lives precisely in
those files. Privacy enforcement was removed because the needed extension is
absent.

**Required fix:** move behavior into package-owned facades and leave entry files
as require-only shims, then choose and enforce a real privacy-boundary tool.

**Tracking:** `wild-rails-ai-ops-8l5.7`.

### F-13 — P2: additional security and data-quality gaps

These findings are not individually P0, but together they define the remaining
hardening program:

- Introspection filters model columns but not association foreign keys
  (`wild/lib/wild/introspection/guard/query_guard.rb:25`,
  `.../adapter/schema_inspector.rb:39`). Blocked column names can reappear.
- Admin blast estimation defaults missing estimates to one
  (`wild/lib/wild/admin_tools/guard/pipeline.rb:102`), which fails open under an
  incomplete preview.
- Both MCP surfaces expose raw exception messages
  (`wild/lib/wild/admin_tools/server/response_formatter.rb:20-31`,
  `wild/lib/wild/introspection/server/tool_handler.rb:14-16`).
- Capability audit context is bounded but not secret-sanitized before
  serialization (`wild/lib/wild/capability_gate/audit/event.rb:89-111`).
- Telemetry store limits are declared but not enforced opportunistically by the
  MemoryStore or JsonLinesStore (`wild/lib/wild/telemetry/collector/store/memory_store.rb:8`,
  `.../json_lines_store.rb:25`).
- Timestamp validation accepts an unanchored ISO-like prefix
  (`wild/lib/wild/telemetry/collector/schema/validator.rb:11`).
- Transcript JSONL parsing silently drops malformed lines
  (`wild/lib/wild/telemetry/pipeline/ingestion/claude_code_adapter.rb:38`).
- Markdown export interpolates untrusted content and metadata without escaping
  (`wild/lib/wild/telemetry/pipeline/export/markdown_exporter.rb:40,99`).
- Telemetry filter/validation/envelope exceptions are swallowed without a
  counter (`wild/lib/wild/telemetry/collector/collector/event_receiver.rb:43-52`).

**Tracking:** `wild-rails-ai-ops-8l5.10`.

## 5. Architecture and load-bearing paths

### 5.1 Active repository structure

| Path | Responsibility | Review significance |
|---|---|---|
| `lib/wild/configuration.rb` | Root configuration and nested surface config | Intended single public configuration object; currently incomplete |
| `lib/wild/engine.rb` | Rails engine lifecycle and bridge initialization | Contains the fresh-host boot blocker |
| `lib/wild/introspection/` | Read-side MCP/library boundary | Strong allowlisting; audit writer and association filtering gaps |
| `lib/wild/admin_tools/` | Privileged mutation boundary | Strong guard chain; audit-before-mutation and default adapter defects |
| `lib/wild/capability_gate/` | Policy decisions and decision audit | Fail-closed evaluation, but audit-outage ALLOW policy unresolved |
| `lib/wild/telemetry/collector/` | Privacy-aware event collection/export | Strong filtering and storage locking; declared bounds not automatic |
| `lib/wild/telemetry/pipeline/` | Transcript normalization/export | Secret metadata fix landed; malformed data is still silently lost |
| `lib/wild/telemetry/analysis/` | Gap/denial analysis | Structurally migrated, incompatible with real Collector output |
| `lib/wild/hooks/` | Hook registry and execution | Registration race; parallel configuration is inert |
| `lib/wild/analyzers/permission/` | Capability/grant analysis | Cycle fix landed; malformed rows still silently disappear |
| `lib/wild/analyzers/test_flakes/` | Test-result ingestion and flake detection | Uses observation rows instead of unique runs |
| `lib/wild/skillops/` | Skill registry/lifecycle | Correctly disabled; lifecycle/version model is not safe to enable |
| `spec/dummy/` | Rails integration fixture | Useful, but masks default-adapter boot failure |
| `tests/` and `TEST_AUDIT.md` | Test policy and traceability | Materially stale and internally inconsistent |
| `.github/workflows/` | CI, release, dependency/review automation | CI strong; release and advisory automation need correction |

### 5.2 Configuration split

The gemspec promises one configuration block, but runtime ownership remains
split. Root Introspection exposes only three fields
(`wild/lib/wild/configuration.rb:69-79`), while namespace configuration owns API
keys and the audit path (`wild/lib/wild/introspection/configuration.rb:13-25`).
The bridge copies only two paths (`wild/lib/wild/introspection.rb:97-104`). Root
AdminTools exposes adapters (`wild/lib/wild/configuration.rb:81-96`), while
namespace configuration separately owns gate, policy, and audit settings
(`wild/lib/wild/admin_tools/configuration.rb:6-17`); its bridge copies only
adapters (`wild/lib/wild/admin_tools.rb:152-157`).

The resulting API cannot configure authentication and durable auditing through
the promised single surface. The repair should either truly consolidate or
explicitly document a stable two-level design. It should not preserve two
partially overlapping sources of truth.

### 5.3 Eager load and transport mismatch

`require "wild"` eagerly loads all namespaces and the Rails meta-gem
(`wild/lib/wild.rb:3-17`, `wild/lib/wild/engine.rb:3-11`). This conflicts with
the goal of using only selected namespaces and with plain non-Rails MCP
transports. The engine currently has no internal controller routes; a runtime
probe found zero `Wild::Engine` routes. The mount spec proves only that a host
can mount the engine, not that an HTTP/MCP endpoint exists
(`wild/spec/engine/engine_spec.rb:27-40`).

Before optimizing load behavior, decide the supported deployment shapes:

1. Rails engine with in-process library API.
2. Rails-hosted MCP transport.
3. Standalone MCP transport using selected namespaces.

Each shape needs one real system test and one documented startup contract.

## 6. Repository-by-repository audit

The ten legacy repositories are frozen archaeological sources. Scores reflect
their safety/readiness as standalone released repositories, not the quality of
the migration work. None should receive new implementation commits.

### 6.1 `wild-rails-safe-introspection-mcp` — 40/100 legacy, 80/100 migrated

Local HEAD `6f8f82f` is three remote presentation/reviewer commits behind.
Locked dependency resolution prevented a fresh legacy RSpec/RuboCop execution.

The legacy authenticated-user gate is a stub
(`wild-rails-safe-introspection-mcp/lib/.../identity/capability_gate.rb:37`);
central code now fails closed (`wild/lib/wild/introspection/identity/capability_gate.rb:45`).
However, association foreign keys still bypass blocked-column filtering in the
central `query_guard.rb:25` and `schema_inspector.rb:39`. Raw tool errors and
pre-recorder audit gaps also remain centrally.

**Disposition:** redirect to `Wild::Introspection`, migrate only surviving
defects, then archive. Legacy issues #20-22 must be closed or mapped centrally.

### 6.2 `wild-admin-tools-mcp` — 30/100 legacy, 80/100 migrated

Verification: 439 examples, 0 failures; RuboCop 91 files, 0 offenses. Legacy
nonce check/use is racy, but central code fixed it with atomic consumption at
`wild/lib/wild/admin_tools/guard/nonce_store.rb:48`. Internal nonce-oracle
details were also removed.

Surviving central defects are more important than the frozen code: missing
blast estimates default to one (`guard/pipeline.rb:102`), raw exception text is
returned (`server/response_formatter.rb:20`), fresh hosts require unused
adapters, and mutation occurs before durable audit persistence.

**Disposition:** redirect to `Wild::AdminTools`, close obsolete legacy issues
#10-12, then archive.

### 6.3 `wild-capability-gate` — 40/100 legacy, 70/100 migrated

Verification: 224 examples, 0 failures; RuboCop 42 files, 0 offenses. Central
code fixed audit emission on evaluation errors, but session authorization cache
keys still exclude changing prerequisite context
(`wild/lib/wild/capability_gate/session.rb:13-33`); its spec deliberately pins
that behavior (`wild/spec/wild/capability_gate/session_spec.rb:103`). Audit
context is not secret-sanitized, and JSONL append has no cross-process lock.

**Disposition:** redirect to `Wild::CapabilityGate`; close legacy #11 only with
the narrow fix evidence, while keeping broader central audit work open.

### 6.4 `wild-session-telemetry` — 30/100 legacy, 70/100 migrated

Verification: 325 examples, 6 date-rotted failures; RuboCop 40 files, 0
offenses. Central retention compaction fixed the legacy rewrite race, but
normal stores still do not enforce declared count/byte/age limits. Timestamp
validation is weak, and the missing `source_id` makes the exported stream
unreadable by Analysis.

**Disposition:** redirect to `Wild::Telemetry::Collector`; preserve legacy #14
and CLI intent centrally; archive only after the canonical telemetry contract
passes end to end.

### 6.5 `wild-transcript-pipeline` — 50/100 legacy, 80/100 migrated

Verification: 324 examples, 0 failures; RuboCop 45 files, 0 offenses. Central
metadata redaction materially fixes the legacy raw tool input/output exposure
(`wild/lib/wild/telemetry/pipeline/privacy/metadata_redactor.rb:36`). Malformed
JSONL is still silently discarded and Markdown output remains injection-prone
in permissive renderers.

**Disposition:** redirect to `Wild::Telemetry::Pipeline`; map legacy issue #2
to the central cross-namespace integration work.

### 6.6 `wild-gap-miner` — 45/100

Verification: 276 examples, 0 failures; RuboCop 61 files, 0 offenses. Structural
migration into `Wild::Telemetry::Analysis` is complete, but the real producer
contract is broken in two ways described by F-02. The parser also loads the
entire export into memory (`wild/lib/wild/telemetry/analysis/ingestion/export_parser.rb:17-23`).

**Disposition:** redirect to `Wild::Telemetry::Analysis`; transfer legacy #2
to the central P0; archive after the round-trip gate.

### 6.7 `wild-hook-ops` — 68/100

Verification: 247 examples, 0 failures; RuboCop 45 files, 0 offenses. The
registry race survived migration. Configuration accepts sequential or parallel
mode, but the central runner always uses sequential iteration
(`wild/lib/wild/hooks/execution/runner.rb:37-57`). Timeout protection relies on
asynchronous `Timeout.timeout` in the legacy design.

**Disposition:** redirect to `Wild::Hooks`; make registration atomic before
depending on the registry for safety-sensitive extension points.

### 6.8 `wild-permission-analyzer` — 72/100

Verification: 217 examples, 0 failures; RuboCop 49 files, 0 offenses. The legacy
depth-limited DFS falsely reported cycles; central tri-color traversal fixed it
(`wild/lib/wild/analyzers/permission/analyzers/prerequisite_analyzer.rb:39-122`).
The remaining problem is silent omission of malformed capability/grant rows.

**Disposition:** close legacy issue #3 with the central fix evidence, map row
validation centrally, redirect and archive.

### 6.9 `wild-test-flake-forensics` — 61/100

Verification: 277 examples, 0 failures; RuboCop 55 files, 0 offenses. Parsing
and reporting migrated cleanly, but core flake semantics remain wrong: rows are
counted as executions, IDs can collide within one second, and invalid parser
entries can disappear silently.

**Disposition:** redirect to `Wild::Analyzers::TestFlakes`; repair run identity
and data-quality reporting centrally before using results for CI policy.

### 6.10 `wild-skillops-registry` — 43/100

Verification: 251 examples, 0 failures; RuboCop 52 files, 0 offenses. Structural
migration is complete, and disabling the namespace by default is correct. Core
lifecycle, version, export, and transaction semantics remain internally
inconsistent. Version validation is also unanchored and accepts trailing junk
(`wild/lib/wild/skillops/models/skill.rb:74-78`).

**Disposition:** keep disabled, repair centrally, redirect and archive. Do not
advertise registry integrity until round-trip history and transition tests pass.

### 6.11 Umbrella — 25/100 current truthfulness

The umbrella is cleanly presented but now describes history as the current
system. It has no deterministic markdown/link workflow despite CLAUDE claiming
one; only an advisory MiniMax workflow exists. Main is unprotected, and secret
scanning/push protection are disabled. In a public documentation repository
whose product is its claims, those are meaningful governance gaps.

**Disposition:** use this report as the transition record, correct current
topology in P4, add deterministic document validation, update the reviewer law,
and preserve the old model under a dated history section.

## 7. CI, GitHub, and repository sweep

### 7.1 Central repository

- Public `jeremylongshore/wild`; secret scanning and push protection enabled.
- Main requires `CI OK` and `CodeQL`, but strict updating, reviews,
  conversation resolution, signatures, and admin enforcement are not required.
- Five open PRs are Dependabot updates. Their core CI is green; MiniMax fails
  because `pull_request` does not expose `MINIMAX_API_KEY` to Dependabot. The
  same-repository guard does not address that actor.
- Do not auto-merge the MCP 1.x widening proposal. Both actual transports are
  stubs, and central Bead `wild-rvv.14` requires a separate compatibility
  evaluation.
- Weekly Packwerk updates fail because Packwerk 3.3 requires Ruby 3.3 while the
  gem supports Ruby 3.2 and pins `~> 3.2` (`wild/wild.gemspec:24,63`). Encode a
  compatible ignore/update policy.
- No tags or releases exist. Version is 0.0.1.
- Git hygiene includes 46 local branches, 15 linked worktrees, 23 branches with
  gone upstreams, and one lingering remote feature branch. Review each target
  before safe cleanup; do not bulk-delete.
- GitHub issues #47, #50, #60, and #66 appear implemented or documented but
  remain open. Six namespace-move Beads remain `in_progress` after completed
  merges. Reconcile status with evidence.

### 7.2 Legacy repositories

All ten are public, `archived=false`, and retain their old v0.1.0-era package
presentation. Their August 25 commits added funding and advisory review
automation rather than runtime fixes. No open PRs exist. Twenty legacy GitHub
issues remain open across the estate, primarily May audit and migration work.
Branch protection has no effective required checks/reviews, and legacy CI is
generally limited to RSpec and RuboCop without dependency audit, CodeQL, SBOM,
or release verification.

Modernizing CI in frozen code is poor-value work. The correct control is to
freeze, redirect, map living issue intent centrally, and archive at P4.

### 7.3 Umbrella

Main was unprotected during inspection. The repo is public and proprietary,
with secret scanning and push protection disabled. Its only workflow is an
advisory AI review whose prompt encodes stale architecture. Deterministic
Markdown syntax and link checks claimed by `CLAUDE.md:22` are absent.

## 8. Test strategy assessment

The suite is large enough to make regressions expensive, but test count should
not be used as a quality proxy. The most important failures found here were
outside current fixture boundaries:

- The dummy host configured adapters that a real first-run host would not.
- Analysis fixtures flattened data rather than consuming Collector output.
- Audit tests exercised action failures, not audit-storage failure after an
  action.
- Duplicate registry tests were serial rather than concurrent.
- Flake tests treated rows as independent runs without adversarial duplicate
  IDs.
- Skillops tests did not prove exported history matched transition history.

The next test phase should focus on judgment-bearing journeys:

1. Fresh Rails 7.1/8.0/8.1 install and boot with safe defaults.
2. Real MCP request/response over each supported transport.
3. Preview, confirmation, mutation, durable audit, and storage-outage behavior.
4. Collector -> serialized export -> Pipeline/Analysis cross-namespace flow.
5. Concurrent hook registration and multiprocess audit-store behavior.
6. Malformed permission, transcript, telemetry, and test-result inputs with
   explicit error/counter expectations.
7. Skill lifecycle transition -> version snapshot -> export -> reload.
8. Gem build -> unpack -> manifest/SBOM/provenance verification.
9. Five-minute adoption stopwatch from a genuinely blank application.

Mutation/property testing should be introduced only where it answers a named
risk: gate truth tables, nonce state machines, lifecycle transitions, schema
round-trips, and redaction invariants are good candidates. Broad vanity
mutation scores are not.

## 9. Security and privacy model

### 9.1 Trust boundaries

The principal untrusted inputs are MCP requests, capability policy/config,
host-app model metadata, telemetry/transcript files, permission/grant files,
test reports, and skill definitions. The sensitive outputs are privileged
mutations, schema/association metadata, audit trails, raw error information,
telemetry exports, and generated Markdown.

The system should apply four consistent rules:

1. Authorization failure or inability to evaluate denies by default.
2. A privileged mutation does not begin until its intent is durably recorded.
3. Client errors reveal a stable code and correlation ID, not internal detail.
4. Input that cannot be validated is rejected or counted visibly; it is never
   silently removed from an audit or analysis.

Current code meets the first rule in most evaluation paths and partially meets
the others. F-03, F-09, and F-13 define the remaining work.

### 9.2 Secrets and artifact hygiene

No plaintext credential was found in the audited histories. However, package
hygiene must not rely on encryption or Git cleanliness: the runtime artifact
should not include internal SOPS metadata, task databases, agent settings, or
review instructions. A minimal allowlist is easier to audit than a growing
denylist.

### 9.3 Dependency posture

Bundler Audit was clean centrally. Frozen repositories generally could not run
it because it was not installed; that is an evidence gap, not evidence of a
vulnerability. Central automation should evaluate runtime changes separately
from development-tool updates. Mutable major-version action references should
be SHA-pinned consistently, as MiniMax already is.

## 10. Performance, cost, and scalability

This is primarily an embedded Ruby library, so current direct infrastructure
cost is low. The dominant risks are memory, I/O, and host-process contention:

- Telemetry Analysis loads a full export into memory. Stream records or enforce
  a documented maximum before production-scale use.
- MemoryStore and normal JSONL append paths do not automatically enforce the
  configured retention/size bounds. Unbounded data can become both cost and
  availability risk.
- JSONL audit writers with only process-local mutexes do not provide reliable
  multi-process semantics under Puma/Sidekiq-style deployment.
- Eager loading all namespaces and Rails raises memory/startup cost for users
  that need only one analysis library or standalone transport.
- Parallel hook mode is accepted but not implemented; callers can mistakenly
  design latency budgets around an inert setting.
- Rate limiting and blast estimation operate in-process unless the host injects
  durable/shared adapters. Document the deployment model before promising
  cross-process limits.

Performance work should follow a valid end-to-end product journey. Do not
optimize the broken telemetry pipeline or stub transports before their
contracts are made correct.

## 11. Operations and contributor playbook

### 11.1 Safe local verification

```bash
bundle check
bundle exec rspec
bundle exec rubocop
bundle exec packwerk validate
bundle exec packwerk check
bundle exec brakeman --force -p . --skip-files spec/ --no-pager --quiet --no-progress
bundle exec bundler-audit check --update
./scripts/audit-harness verify
```

Treat warnings printed repeatedly by the full suite—especially dropped audit
records—as test evidence to classify, not harmless noise.

### 11.2 Change ownership

- Runtime fixes go only to `jeremylongshore/wild`.
- Ecosystem truth, cutover, and this audit record live in
  `intent-solutions-io/wild-rails-ai-ops`.
- Legacy repositories receive only the planned redirect/archive change at P4.
- Durable work lives in Beads. Do not create parallel Markdown task ledgers.
- Update the central status source and public mirror in the same change when
  topology or release state changes.

### 11.3 Release runbook preconditions

Before any non-dry-run release dispatch, require evidence for all of the
following:

1. P0 and release-blocking P1 Beads are closed with test receipts.
2. Protected main equals the commit being released.
3. Rails support matrix and system journeys are green.
4. Fresh-host boot and five-minute stopwatch pass.
5. Both declared transports execute real calls.
6. Gem manifest contains only intended files.
7. SBOM, provenance, changelog, and version agree.
8. Protected environment approval is recorded.
9. The built artifact, not a rebuilt local variant, is attached/published.
10. Post-release smoke and rollback instructions are executable.

## 12. Prioritized roadmap

### First 48 hours: restore truth and contain release risk

1. Keep release workflow disabled or dry-run-only until F-01 through F-08 have
   explicit gate coverage.
2. Fix fresh-host boot and telemetry schema P0s.
3. Fix audit-before-mutation before changing any other AdminTools ergonomics.
4. Update umbrella entry copy and reviewer law enough to point contributors to
   the central repository and this report.
5. Reconcile issue/Bead items that are already complete so ready-work selection
   becomes trustworthy.

### First week: repair correctness contracts

1. Repair Skillops state/history and keep it disabled until the round-trip
   lifecycle test passes.
2. Make hook registration atomic.
3. Make permission and parser data loss visible.
4. Correct flake execution identity and rates.
5. Resolve the audit-outage posture with an explicit owner decision.
6. Consolidate authentication and durable audit destinations into the supported
   configuration API.

### First month: finish the product path

1. Implement and system-test the install generator.
2. Implement real MCP transports and executable declarations.
3. Add versioned prompts and the real engine/transport routes required by the
   supported deployment shape.
4. Add Rails 7.1/8.0/8.1 matrix and fresh-host applications.
5. Rebuild RTM/test policy, enforce meaningful coverage and boundary gates.
6. Make the release pipeline fail closed and prove the gem artifact.
7. Run the five-minute stopwatch repeatedly on clean environments.

### P4 cutover

1. Cut v0.1.0 only after all adoption and release gates pass.
2. Replace each legacy README with a dated deprecation/redirect page.
3. Close or transfer all surviving legacy issues with explicit links.
4. Archive all ten legacy repositories; preserve history, releases, and links.
5. Convert this umbrella into the current ecosystem landing page and retain the
   old ten-gem graph under a clearly labeled historical section.

## 13. Durable work docket

This audit created the following children under Bead
`wild-rails-ai-ops-8l5`:

| Bead | Priority | GitHub | Outcome |
|---|---:|---|---|
| `.1` | P0 | [`wild#94`](https://github.com/jeremylongshore/wild/issues/94) | Fresh Rails host boots with AdminTools disabled by default |
| `.2` | P1 | [`wild#95`](https://github.com/jeremylongshore/wild/issues/95) | Durable intent exists before any AdminTools mutation |
| `.3` | P0 | [`wild#96`](https://github.com/jeremylongshore/wild/issues/96) | Telemetry producer/consumer round-trip is canonical and green |
| `.4` | P1 | [`wild#97`](https://github.com/jeremylongshore/wild/issues/97) | Skillops lifecycle, history, export, and registration are coherent |
| `.5` | P1 | [`wild#98`](https://github.com/jeremylongshore/wild/issues/98) | Hook, permission, and flake correctness defects are fixed |
| `.6` | P1 | [`wild#99`](https://github.com/jeremylongshore/wild/issues/99) | Release and gem artifact gates fail closed |
| `.7` | P1 | [`wild#100`](https://github.com/jeremylongshore/wild/issues/100) | Rails matrix is proved and package boundaries are enforced |
| `.8` | P2 | [`wild#101`](https://github.com/jeremylongshore/wild/issues/101) | Test policy, RTM, coverage, and harness evidence are current |
| `.9` | P1 | [`umbrella#2`](https://github.com/intent-solutions-io/wild-rails-ai-ops/issues/2) | Umbrella topology is corrected and legacy repos are archived at cutover |
| `.10` | P1 | [`wild#102`](https://github.com/jeremylongshore/wild/issues/102) | Residual audit, storage, disclosure, privacy, and observability gaps close |
| `.11` | P2 | [`umbrella#3`](https://github.com/intent-solutions-io/wild-rails-ai-ops/issues/3) | GitHub, Beads, branch, Dependabot, and advisory CI state is reconciled |

The new docket is related to existing Beads for `source_id`, fictional
telemetry integration, test-harness rollout, and capability audit completeness
rather than silently duplicating or deleting that history.

Cutover mirrors were also filed in every frozen repository so the final action
has an owner-visible landing point: safe-introspection
[#24](https://github.com/jeremylongshore/wild-rails-safe-introspection-mcp/issues/24),
admin-tools [#14](https://github.com/jeremylongshore/wild-admin-tools-mcp/issues/14),
capability-gate [#13](https://github.com/jeremylongshore/wild-capability-gate/issues/13),
session-telemetry [#17](https://github.com/jeremylongshore/wild-session-telemetry/issues/17),
transcript-pipeline [#4](https://github.com/jeremylongshore/wild-transcript-pipeline/issues/4),
gap-miner [#4](https://github.com/jeremylongshore/wild-gap-miner/issues/4),
hook-ops [#3](https://github.com/jeremylongshore/wild-hook-ops/issues/3),
permission-analyzer [#5](https://github.com/jeremylongshore/wild-permission-analyzer/issues/5),
test-flake-forensics [#3](https://github.com/jeremylongshore/wild-test-flake-forensics/issues/3),
and skillops-registry
[#4](https://github.com/jeremylongshore/wild-skillops-registry/issues/4).

## 14. Evidence limits and open decisions

This review did not change runtime code, publish packages, merge Dependabot
updates, close old issues, archive repositories, or delete branches/worktrees.
It did not prove behavior against production Rails applications because the
fresh-host path currently fails. Legacy Introspection dependencies were not
installed solely to make a frozen checkout testable.

Owner decisions still required during remediation:

- Whether an ALLOW may survive a capability audit outage. The recommendation
  for a safety-sensitive default is no, unless the caller receives and enforces
  an explicit degraded state.
- Whether the supported product includes standalone non-Rails MCP transports.
  If yes, eager Rails loading must be removed from that path.
- Whether the central MIT license and umbrella/legacy proprietary presentation
  are intentional. The current estate is inconsistent.
- Which per-file/component coverage floor is meaningful. The recommendation is
  to choose the threshold after RTM repair, then enforce it rather than display
  an advisory badge.
- Whether Rails versions newer than the tested matrix are supported or merely
  allowed to attempt installation. Avoid an unbounded compatibility promise.

## 15. Final conclusion

The August review wave materially improved Wild. The central code is much
closer to a high-quality library than the stale umbrella suggests, and its
green test/security baseline is real. The remaining risk is concentrated at
the boundaries between namespaces, between the gem and a fresh host, between a
mutation and its durable evidence, and between source and release artifact.

That concentration is good news: the project does not need another broad
rewrite. It needs a strict release moratorium, eleven bounded remediation
outcomes, and a truthful cutover. Complete those in priority order and Wild can
move from a strong collection of internal libraries to an installable,
auditable Rails product without discarding the engineering already done.

---

### Audit team

The review was executed in parallel by three independent code-audit lanes plus
the coordinating reviewer: consolidated Rails engine; legacy repositories
Group A; legacy repositories Group B and umbrella reconciliation. Each lane was
read-only and supplied exact file/line evidence and runnable checks. The
coordinator independently reproduced or cross-checked the release-blocking
claims before filing the durable docket.
