# CLAUDE.md — wild-rails-ai-ops

This file provides guidance to Claude Code when working in this repository.

## Identity

- **Repo:** `wild-rails-ai-ops`
- **Role:** Documentation, governance, audit, and migration umbrella for Wild
- **Active implementation:** [`jeremylongshore/wild`](https://github.com/jeremylongshore/wild), one Rails engine with ten `Wild::*` namespaces
- **Status:** The consolidated engine is pre-release. The ten original `wild-*` repositories are frozen pending redirect and archive.

## What this repo contains

- `README.md` — public-facing ecosystem and migration map
- `ARCHITECTURE.md` — current engine topology and repository boundaries
- `000-docs/` — dated cross-repository audits and operating records
- `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` — standard governance
- `LICENSE` — Intent Solutions Proprietary
- `.github/workflows/` — advisory document and claim review
- `.beads/` — ecosystem work tracking; Dolt is the issue-state authority

## What this repo does NOT contain

- No runtime Ruby code. All active implementation belongs in `jeremylongshore/wild`.
- No independently released ten-gem product. The old repositories are historical migration sources.
- No authority to declare the engine release-ready. Release evidence belongs in the active implementation repository.

## When to edit

- The active engine changes topology or repository ownership: update
  `README.md`, `ARCHITECTURE.md`, `REVIEW.md`, the review prompts, and
  `SECURITY.md` together.
- A dated ecosystem audit or decision is accepted: add it to `000-docs/` and
  update `000-docs/000-INDEX.md`.
- A migration or archival state changes: update the Bead and linked GitHub
  issue first, then mirror the new state in current-facing documentation.
- Do not rewrite dated audit records to match later implementation changes.

## Source-of-truth boundaries

- Runtime behavior, version, and release posture: `jeremylongshore/wild` source,
  tests, README, and `000-docs/`.
- Cross-repository audits and migration record: this umbrella's `000-docs/`.
- Work state: Beads plus linked GitHub issues.
- Public summary: this README mirrors those authorities and must not lead them.

## Repository policy

- Direct implementation fixes only to `jeremylongshore/wild`.
- The ten original `jeremylongshore/wild-*` repositories remain frozen until
  the cutover gate authorizes redirect and archive work.
- This umbrella accepts documentation, governance, audit, workflow, and Beads
  changes within its scope.
- External contributions remain closed; security disclosures go to the private
  address in `SECURITY.md`, never a public issue.

## Review starting point

- Read `README.md` and `ARCHITECTURE.md` for current topology.
- Read `000-docs/000-INDEX.md` for dated evidence.
- Read `REVIEW.md` before editing the automated review workflow.
- Run `bd prime` before changing tracked ecosystem work.


<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:7510c1e2 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->
