# akm Registry

Static registry index for [akm](https://github.com/itlackey/akm) stash discovery.
The CLI ships with this registry pre-configured and uses it for `akm registry search`
and `akm search --source registry`.

## How it works

This repo publishes a static `index.json` file. In akm v0.7.0, the official
registry contract is the static-index provider backed by a v3 registry index.
The CLI fetches and caches that file locally, then searches it for matching
stashes. Current 0.7.0 support includes:

- Registry index `version: 3`
- Auto-discovered stashes from npm and GitHub
- `stashes[]` entries with required `id`, `name`, `ref`, `source`
- Optional asset-level metadata via each entry's `assets` array for `akm registry search --assets`
- Multiple configured registries via the `registries` config field
- Forward-compatible extra fields, with the legacy `curated` boolean ignored and no longer published

Supported `source` values in 0.7.0 are: `npm`, `github`, `git`, `local`.

## Getting listed

There are two ways to get listed:

- Publish an npm package with `akm-stash` in `keywords`
- Add the `akm-stash` GitHub topic to your repository

Manual entry example:

```json
{
  "id": "github:your-org/your-stash",
  "name": "Your Stash Name",
  "description": "One-line description of what the stash provides",
  "ref": "your-org/your-stash",
  "source": "github",
  "homepage": "https://github.com/your-org/your-stash",
  "tags": ["deploy", "review"],
  "assetTypes": ["script", "skill"],
  "author": "your-name",
  "license": "MIT"
}
```

Required fields: `id`, `name`, `ref`, `source`.

`assetTypes` should reflect the stash's published asset kinds. In 0.7.0 that
includes the standard built-in kinds such as `script`, `skill`, `command`,
`agent`, `knowledge`, `workflow`, `wiki`, `vault`, `memory`, and `lesson`.

`assets` entries support:

- `type` (required)
- `name` (required)
- `description` (optional)
- `tags` (optional)
- `estimatedTokens` (optional)

The legacy top-level `curated` boolean used by older registry entries was
removed before the 0.7.0 release. akm still ignores it for compatibility, but
new registry data should omit it.

## Index format

```json
{
  "version": 3,
  "updatedAt": "2026-04-24T00:00:00Z",
  "stashes": [
    {
      "id": "github:your-org/deploy-stash",
      "name": "deploy-stash",
      "description": "Deployment scripts and skills",
      "ref": "your-org/deploy-stash",
      "source": "github",
      "tags": ["deploy", "infrastructure"],
      "assetTypes": ["script", "skill"],
      "latestVersion": "v1.2.0",
      "assets": [
        {
          "type": "script",
          "name": "deploy.sh",
          "description": "Deploy to production",
          "tags": ["deploy", "production"],
          "estimatedTokens": 120
        }
      ],
      "author": "your-org"
    }
  ]
}
```

## Configuring akm

akm uses this registry by default. You can add or override registries with:

```bash
akm registry list
akm registry add https://your-company.com/registry/index.json --name team
export AKM_REGISTRY_URL=https://your-company.com/registry/index.json
```

In config, registries live under the `registries` array.

## Official akm stash

In addition to the registry, there is an official stash repo is installable as an akm stash:

```bash
akm add github:itlackey/akm-stash
akm index
akm show skill:akm-quickstart
```
