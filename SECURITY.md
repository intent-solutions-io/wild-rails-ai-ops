# Security

## Reporting a vulnerability

Do not file a public GitHub issue for a suspected security vulnerability.

Email: jeremy@intentsolutions.io

Include:

- The affected repository, namespace, commit, or version
- A description of the issue and its impact
- Reproduction steps, if available
- Whether the issue may also exist in an original `wild-*` repository

## Supported versions

Wild is currently pre-release and has no tagged supported release. Security
fixes for the active implementation are made on the `main` branch of
[`jeremylongshore/wild`](https://github.com/jeremylongshore/wild).

The ten original `jeremylongshore/wild-*` repositories are frozen migration
sources and are not supported for new deployments. Reports against them are
still useful when the same behavior may have migrated into the consolidated
engine.

## Scope

This policy covers:

- The consolidated `jeremylongshore/wild` Rails engine and its ten namespaces
- This `intent-solutions-io/wild-rails-ai-ops` documentation and governance
  umbrella
- Migration-relevant vulnerabilities discovered in the ten frozen source
  repositories

See the [README](README.md) for the current repository and namespace map.
