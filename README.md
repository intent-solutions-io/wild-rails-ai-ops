# wild-rails-ai-ops

> **Current topology:** Wild is one consolidated, pre-release Rails engine at
> [`jeremylongshore/wild`](https://github.com/jeremylongshore/wild). Its ten
> capabilities are Ruby namespaces inside that gem, not ten independently
> developed products.

This repository is the Wild ecosystem umbrella. It holds cross-repository
documentation, governance, audit records, migration tracking, and the cutover
plan for the ten original repositories. Runtime implementation belongs in
`jeremylongshore/wild`; no Ruby application code lives here.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/U5S225PTME)

## Ecosystem map

```text
intent-solutions-io/wild-rails-ai-ops       umbrella: docs, governance, audits
                    |
                    `-- jeremylongshore/wild
                          one Rails engine / one gem
                          ten Wild::* namespaces
                                    |
                                    `-- ten frozen source repositories
                                        awaiting redirect and archive
```

## Active implementation

| Repository | Role | Release posture |
|---|---|---|
| [`jeremylongshore/wild`](https://github.com/jeremylongshore/wild) | The only active runtime implementation and the only Wild repository accepting code changes | Pre-release, no tags; release gates remain open |

The engine contains these namespaces:

| Namespace | Responsibility | Original repository |
|---|---|---|
| `Wild::Introspection` | Governed Rails introspection | [`wild-rails-safe-introspection-mcp`](https://github.com/jeremylongshore/wild-rails-safe-introspection-mcp) |
| `Wild::AdminTools` | Governed administrative operations | [`wild-admin-tools-mcp`](https://github.com/jeremylongshore/wild-admin-tools-mcp) |
| `Wild::CapabilityGate` | Capability policy evaluation | [`wild-capability-gate`](https://github.com/jeremylongshore/wild-capability-gate) |
| `Wild::Telemetry::Collector` | Privacy-aware telemetry collection | [`wild-session-telemetry`](https://github.com/jeremylongshore/wild-session-telemetry) |
| `Wild::Telemetry::Pipeline` | Transcript normalization and dispatch | [`wild-transcript-pipeline`](https://github.com/jeremylongshore/wild-transcript-pipeline) |
| `Wild::Telemetry::Analysis` | Telemetry and capability-gap analysis | [`wild-gap-miner`](https://github.com/jeremylongshore/wild-gap-miner) |
| `Wild::Hooks` | Hook registration and execution | [`wild-hook-ops`](https://github.com/jeremylongshore/wild-hook-ops) |
| `Wild::Analyzers::Permission` | Permission-model analysis | [`wild-permission-analyzer`](https://github.com/jeremylongshore/wild-permission-analyzer) |
| `Wild::Analyzers::TestFlakes` | Test-flake forensics | [`wild-test-flake-forensics`](https://github.com/jeremylongshore/wild-test-flake-forensics) |
| `Wild::Skillops` | Internal skill registry; opt-in, with `enabled` defaulting to `false` in [engine configuration](https://github.com/jeremylongshore/wild/blob/e47ee2cfe5ffe729692e4a24703f7dd111e71002/lib/wild/configuration.rb#L345-L368) | [`wild-skillops-registry`](https://github.com/jeremylongshore/wild-skillops-registry) |

## Migration status

The ten original repositories are historical source repositories. Their code
has been moved into the namespaces above and they are frozen for new
implementation. They remain public and unarchived until the consolidated engine
passes its adoption and release gates; the final cutover will add prominent
redirects and archive them.

Track that work in [umbrella issue #2](https://github.com/intent-solutions-io/wild-rails-ai-ops/issues/2).
New defects and implementation work belong in
[`jeremylongshore/wild`](https://github.com/jeremylongshore/wild/issues), not in
the frozen repositories.

## Release posture

At the 2026-09-14 audit baseline, the consolidated engine was not yet a
released, installable product. Its library-level test and security gates passed,
but the install generator, MCP executables, end-to-end telemetry contract,
fresh-host boot path, packaging, and stopwatch adoption gate still required
work.

The dated [2026-09-14 ecosystem deep review](000-docs/001-AA-AUDT-wild-ecosystem-deep-code-review-2026-09-14.md)
records the evidence, release blockers, and linked remediation docket. Consult
the active [`wild` README](https://github.com/jeremylongshore/wild#readme) for
current implementation status.

## Documentation

- [Architecture](ARCHITECTURE.md) — current engine topology and namespace map
- [Audit records](000-docs/000-INDEX.md) — dated cross-repository evidence
- [Contributor policy](CONTRIBUTING.md) — current contribution posture
- [Security policy](SECURITY.md) — private vulnerability reporting and scope

## License

This umbrella repository is Intent Solutions Proprietary; see [LICENSE](LICENSE).
The active `jeremylongshore/wild` implementation is separately licensed under
the [MIT License](https://github.com/jeremylongshore/wild/blob/e47ee2cfe5ffe729692e4a24703f7dd111e71002/LICENSE).
Each historical repository retains its own license file.

## Contributing

External contributions are not currently accepted. For internal contributor
guidance, see [CONTRIBUTING.md](CONTRIBUTING.md).
