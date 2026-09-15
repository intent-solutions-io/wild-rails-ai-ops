# Architecture — wild-rails-ai-ops

This document describes the current Wild topology. The active implementation
is one Rails engine in [`jeremylongshore/wild`](https://github.com/jeremylongshore/wild).
This umbrella contains documentation and governance, not runtime Ruby code.

## System boundary

```text
Rails host
  |
  `-- Wild::Engine
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

The diagram is a namespace and responsibility map. It is not a claim that every
cross-namespace journey is release-ready. In particular, the 2026-09-14 audit
reproduced an incompatible Collector-to-Analysis export contract and a fresh
Rails-host boot failure. Both are release blockers tracked in the active
implementation repository.

## Runtime responsibilities

### Governed operations

- `Wild::Introspection` owns bounded, read-oriented Rails inspection.
- `Wild::AdminTools` owns privileged administrative operations.
- `Wild::CapabilityGate` owns capability-policy evaluation used by governed
  operations.

### Telemetry

- `Wild::Telemetry::Collector` receives and aggregates privacy-filtered events.
- `Wild::Telemetry::Pipeline` normalizes and dispatches transcript data.
- `Wild::Telemetry::Analysis` analyzes exported telemetry for operational and
  capability gaps.

The intended Collector-to-Analysis round trip is not currently compatible and
must not be represented as shipped until the central integration gate passes.

### Extension and analysis

- `Wild::Hooks` owns hook registration and execution.
- `Wild::Analyzers::Permission` inspects permission models.
- `Wild::Analyzers::TestFlakes` analyzes test-run evidence.
- `Wild::Skillops` is internal, disabled by default, and remains outside the
  supported adoption path until its lifecycle and history work is complete.

## Repository ownership

| Surface | Repository | Change policy |
|---|---|---|
| Runtime engine and all ten namespaces | [`jeremylongshore/wild`](https://github.com/jeremylongshore/wild) | Active implementation; fixes land here |
| Ecosystem documentation, governance, audits, migration | [`intent-solutions-io/wild-rails-ai-ops`](https://github.com/intent-solutions-io/wild-rails-ai-ops) | Current umbrella; no runtime Ruby code |
| Ten original `jeremylongshore/wild-*` repositories | Linked from the [README namespace map](README.md#active-implementation) | Frozen; redirect and archive at cutover |

## Topology decision and migration

The project began as ten separately developed gems. A 2026-05-29 architecture
council rejected that operating model and selected one Rails engine containing
ten namespaces. The implementation was consolidated into `jeremylongshore/wild`;
the original repositories remain readable for history but are not development
targets.

Cutover is deliberately incomplete. The original repositories will be
redirected and archived only after the consolidated engine clears its release
gates. [Umbrella issue #2](https://github.com/intent-solutions-io/wild-rails-ai-ops/issues/2)
owns that transition.

## Evidence and current status

- The active implementation README and its `000-docs/` directory own current
  code and release status.
- This umbrella owns cross-repository audits and migration records.
- Beads and linked GitHub issues own planned remediation state.
- Dated audits remain historical evidence and are not silently rewritten when
  the implementation changes.

See the [2026-09-14 ecosystem deep review](000-docs/001-AA-AUDT-wild-ecosystem-deep-code-review-2026-09-14.md)
for the latest cross-repository baseline recorded here.

## License boundary

This umbrella is Intent Solutions Proprietary. The consolidated
`jeremylongshore/wild` engine is MIT-licensed. Historical repositories retain
their individual license files.
