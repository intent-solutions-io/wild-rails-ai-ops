# wild-rails-ai-ops

A set of 10 Ruby gems for running AI agents inside Rails applications under explicit capability control, with audited execution and privacy-aware telemetry.

This repo is the umbrella for the `wild-*` constellation. Each gem is its own independently developed and released repository under `jeremylongshore/wild-*`; this repo contains the ecosystem-level documentation, the dependency map, and the per-repo status table.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/U5S225PTME)

## What's in the ecosystem

| Repo | Archetype | What it does |
|------|-----------|--------------|
| [wild-rails-safe-introspection-mcp](https://github.com/jeremylongshore/wild-rails-safe-introspection-mcp) | A — Product-Facing Operational | Safe, governed, read-only Rails production introspection for AI agents via MCP. Three MCP tools, full safety controls, no write ops. |
| [wild-admin-tools-mcp](https://github.com/jeremylongshore/wild-admin-tools-mcp) | A — Product-Facing Operational | Administrative tooling MCP server for ops and management actions exposed to AI agents under capability gating. |
| [wild-capability-gate](https://github.com/jeremylongshore/wild-capability-gate) | D — Coordination / Registry | Privileged tool access gating and capability control. Used as a dependency by `wild-admin-tools-mcp` to police tool invocation. |
| [wild-session-telemetry](https://github.com/jeremylongshore/wild-session-telemetry) | B — Data Pipeline / Analytics | Privacy-aware session telemetry collection and export for AI-agent sessions. |
| [wild-transcript-pipeline](https://github.com/jeremylongshore/wild-transcript-pipeline) | B — Data Pipeline | Transcript ingestion, normalization, and processing. |
| [wild-gap-miner](https://github.com/jeremylongshore/wild-gap-miner) | B — Data Pipeline / Analytics | Gap analysis from telemetry and transcript data. |
| [wild-hook-ops](https://github.com/jeremylongshore/wild-hook-ops) | C — SDLC Companion | Hook lifecycle management — registration, execution, auditing, health monitoring for agent workflow extension points. Generalizes ad-hoc hook patterns from the `*-mcp` gems. |
| [wild-permission-analyzer](https://github.com/jeremylongshore/wild-permission-analyzer) | C — SDLC Companion | Permission model analysis and audit tooling for engineers. |
| [wild-test-flake-forensics](https://github.com/jeremylongshore/wild-test-flake-forensics) | C — SDLC Companion | Test flake detection, root cause analysis, and triage. |
| [wild-skillops-registry](https://github.com/jeremylongshore/wild-skillops-registry) | D — Coordination / Registry | Skill/capability registry and coordination control plane. v0.1.0 — currently standalone; consumer adoption planned for v2. |

## Archetype reference

| Letter | Meaning | Examples |
|--------|---------|----------|
| A | Product-Facing Operational | The MCP servers an agent talks to in production |
| B | Data Pipeline / Analytics | Telemetry, transcripts, gap analysis |
| C | SDLC Companion | Engineer-facing tools (analyzer, flake forensics, hook ops) |
| D | Coordination / Registry | Cross-gem coordination primitives (capability gate, skill registry) |

Full taxonomy definition lives in the `wild-rails-safe-introspection-mcp` and `wild-admin-tools-mcp` repos as `000-docs/006-DR-STND-repo-startup-standard.md`.

## Dependency map

```
[ AI Agent ]
     │
     ├─► wild-rails-safe-introspection-mcp ──┐
     │                                       ├─► (audit log)
     └─► wild-admin-tools-mcp ──► wild-capability-gate
                                            │
                                            └─► (decision log)

[ Runtime instrumentation ]
     │
     └─► wild-session-telemetry ──► wild-transcript-pipeline ──► wild-gap-miner

[ Engineer / SDLC ]
     │
     ├─► wild-permission-analyzer (run before deploy)
     ├─► wild-test-flake-forensics (CI integration)
     └─► wild-hook-ops (v2 — to be adopted by *-mcp gems)

[ Cross-gem coordination ]
     │
     └─► wild-skillops-registry (v0.1.0 — standalone in current state)
```

Detailed per-repo runtime architecture, integration points, and failure modes are documented in each repo's `000-docs/NNN-AT-AUDT-appaudit-2026-05-28.md`.

## Status

All 10 repos are v1 (or v0.1.0 for `wild-skillops-registry`). Per-repo test counts and detail live in the [per-repo status table in the ecosystem CLAUDE.md](https://github.com/intent-solutions-io/wild-rails-ai-ops/blob/main/CLAUDE.md#11-per-repo-status-table).

## License

Intent Solutions Proprietary. Each individual `wild-*` repo carries the same license. See [LICENSE](./LICENSE).

## Contributing

External contributions are not currently accepted. For internal contributor guidelines, see [CONTRIBUTING.md](./CONTRIBUTING.md).

For security disclosures, see [SECURITY.md](./SECURITY.md).
