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
| `automerge` (via `packageRules`) | enabled for `minor`, `patch`, `pin`, `digest` only, and never for the custom manager below | Non-major updates automerge once checks pass; **major updates require manual review**. The `compatibility_date` bump is excluded because its version numbers carry no severity: every bump looks like a patch, while the change it opts into is a runtime behaviour change. |
| `minimumReleaseAge` | `3 days` | A freshly published release (potentially broken or compromised) is not automerged until it has been out for 3 days. |
| `internalChecksFilter` | `strict` | Hold updates that have not yet satisfied `minimumReleaseAge` instead of merging them early. |
| `minimumReleaseAgeBehaviour` (via `packageRules`) | `timestamp-optional` for `docker` deps on `mcr.microsoft.com` and `ghcr.io` | Those registries serve tag lists without release timestamps, and the default reads a missing timestamp as not-yet-aged — so `strict` holds the update indefinitely rather than for the three days above, with nothing to show why. Docker Hub does return timestamps and keeps aging normally. |
| `prConcurrentLimit` | `5` | Cap concurrent Renovate PRs so consumer repos are not flooded. Raise it per-repo if you have more capacity. |
| `timezone` | `Asia/Tokyo` | Reference timezone for any scheduling a consumer adds. |
| `configMigration` | `true` | Renovate opens a PR to migrate deprecated config fields automatically. |
| `groupName` (via `packageRules`) | `playwright` for the Playwright npm packages and the `mcr.microsoft.com/playwright` image | The image bundles browsers built for one Playwright version, so updating the packages and the image in separate PRs breaks E2E runs. Renovate's monorepo data groups the npm packages with each other but does not reach the Docker image, which comes from the `docker` datasource. |
| `groupName` (via `packageRules`) | `cloudflare workers-sdk` for anything sourced from `cloudflare/workers-sdk` | `@cloudflare/vite-plugin` asserts at load time that the installed `wrangler` satisfies its peer range and throws outright when it does not — a mismatch its own source notes a deduplicating package manager can produce — so updating the two in separate PRs leaves the build broken until both land. Renovate's monorepo data carries no `cloudflare` entry, so `group:monorepos` does not cover this. The rule matches on source URL rather than package name so that `@cloudflare/workers-types` stays out, since it ships from `cloudflare/workerd` on that repo's own cadence. The trade-off is that other packages published from the same repo (`create-cloudflare`, `@cloudflare/kv-asset-handler`) join the group as well, which batches unrelated updates together rather than breaking anything. |
| `customManagers` | a regex manager tracking `compatibility_date` in `wrangler.json` / `wrangler.jsonc` against [`cloudflare/workerd`](https://github.com/cloudflare/workerd) releases | Workers pin runtime behaviour by date, and wrangler **requires** the field — so it is set once at scaffold time and then silently freezes, with nothing to distinguish a deliberate pin from a forgotten one. Dates come from workerd's release tags rather than today's date, so a proposed value can never be one the deployed runtime has not shipped yet. |

## Assumptions

Automerge only protects you if your CI gates it. Each consumer repo **should enable branch protection with required status checks** — otherwise a non-major update can merge with no CI run. Renovate uses the platform's native auto-merge (`platformAutomerge`, on by default), so the required-check gate is enforced by the platform.

To opt a repo out of automerge, override it locally:

```json
{
  "extends": ["github>nemolize/renovate-config"],
  "packageRules": [{ "matchUpdateTypes": ["minor", "patch", "pin", "digest"], "automerge": false }]
}
```
