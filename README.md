# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset.

## Usage

Extend it from a repository's Renovate config:

```json
{
  "extends": ["github>nemolize/renovate-config"]
}
```

## What this preset sets

On top of [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended):

| Option | Value | Why |
|---|---|---|
| `automerge` (via `packageRules`) | enabled for `minor`, `patch`, `pin`, `digest` only | Non-major updates automerge once checks pass; **major updates require manual review**. |
| `minimumReleaseAge` | `3 days` | A freshly published release (potentially broken or compromised) is not automerged until it has been out for 3 days. |
| `internalChecksFilter` | `strict` | Hold updates that have not yet satisfied `minimumReleaseAge` instead of merging them early. |
| `prConcurrentLimit` | `5` | Cap concurrent Renovate PRs so consumer repos are not flooded. Raise it per-repo if you have more capacity. |
| `timezone` | `Asia/Tokyo` | Reference timezone for any scheduling a consumer adds. |
| `configMigration` | `true` | Renovate opens a PR to migrate deprecated config fields automatically. |

## Assumptions

Automerge only protects you if your CI gates it. Each consumer repo **should enable branch protection with required status checks** — otherwise a non-major update can merge with no CI run. Renovate uses the platform's native auto-merge (`platformAutomerge`, on by default), so the required-check gate is enforced by the platform.

To opt a repo out of automerge, override it locally:

```json
{
  "extends": ["github>nemolize/renovate-config"],
  "packageRules": [{ "matchUpdateTypes": ["minor", "patch", "pin", "digest"], "automerge": false }]
}
```
