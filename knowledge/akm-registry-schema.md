# akm Registry Index Schema

The official registry publishes a static `index.json` (version 2). The akm
CLI fetches and caches that file, then searches it for matching kits. This
document describes the schema an agent or author needs to produce valid
entries — whether via auto-discovery (npm/GitHub) or a manual PR.

Schema file: [`scripts/registry-index.schema.json`](../scripts/registry-index.schema.json)

## Top-level shape

```json
{
  "version": 2,
  "updatedAt": "2026-04-24T00:00:00Z",
  "kits": [ /* Kit[] */ ]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `version` | `2` | yes | Consumers reject unknown versions. |
| `updatedAt` | ISO 8601 string | yes | Set by the build. |
| `kits` | array of Kit | yes | Order is not semantically meaningful. |

## Kit object

```json
{
  "id": "github:your-org/your-kit",
  "name": "Your Kit Name",
  "ref": "your-org/your-kit",
  "source": "github",
  "description": "One-line, agent-facing description.",
  "homepage": "https://github.com/your-org/your-kit",
  "tags": ["deploy", "review"],
  "assetTypes": ["script", "skill", "workflow"],
  "assets": [
    {
      "type": "skill",
      "name": "deploy-to-fly",
      "description": "Deploy a Node service to Fly.io."
    }
  ],
  "author": "your-org",
  "license": "MIT",
  "curated": true
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | string | yes | Stable unique ID. Canonical form `<source>:<ref>` (e.g. `github:owner/repo`, `npm:@scope/pkg`). |
| `name` | string | yes | Display name. |
| `ref` | string | yes | Install reference. For `source: github`, use `owner/repo`. For `source: npm`, use the package name. |
| `source` | enum | yes | `github` \| `npm` \| `git` \| `url` \| `local`. |
| `description` | string | recommended | First thing agents read — write as a trigger sentence. |
| `homepage` | URL | recommended | Human-visitable docs. |
| `tags` | string[] | recommended | Lowercase, kebab-case. Used for filter and rerank. |
| `assetTypes` | string[] | recommended | Subset of `script`, `skill`, `command`, `agent`, `knowledge`, `workflow`, `wiki`, `vault`, `memory`. Drives type filters in `akm search`. |
| `assets` | Asset[] | optional | Enumerate individual assets for finer-grained search. v2 only. |
| `author` | string | optional | Org or user. |
| `license` | SPDX string | recommended | Consumers surface this before install. |
| `curated` | boolean | optional | `true` means the entry came from `manual-entries.json` and overrides auto-discovery. |

## Asset object (inside `kit.assets`)

```json
{
  "type": "skill",
  "name": "deploy-to-fly",
  "description": "Deploy a Node service to Fly.io.",
  "tags": ["deploy", "fly"]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `type` | string | yes | One of the nine asset types above. |
| `name` | string | yes | Identifier *within* the kit (filename stem for most types; directory name for skills and wikis). |
| `description` | string | recommended | One line. |
| `tags` | string[] | optional | Additional filter keywords. |

## Manual entries

[`manual-entries.json`](../manual-entries.json) at the root of this repo holds
curated Kit objects identical in shape to the entries in `index.json`. The
build merges three sources in this order:

1. npm packages with `agentikit` or `akm-kit` in `keywords`
2. GitHub repos with `agentikit` or `akm-kit` as a topic
3. `manual-entries.json` — overrides and augments the above

A manual entry with the same `id` as an auto-discovered kit **replaces** the
auto-discovered metadata, so use it for curation (better descriptions, pinned
tags) as well as for kits that don't live on npm/GitHub.

## Validation

- `bun run validate` is a quick sanity check (version, kits array).
- Full JSON Schema validation: `scripts/registry-index.schema.json`.
- CI in this repo runs both on PRs.

## Minimum viable manual entry

```json
{
  "id": "github:your-org/your-kit",
  "name": "Your Kit",
  "ref": "your-org/your-kit",
  "source": "github"
}
```

Required fields only: `id`, `name`, `ref`, `source`. Everything else improves
discovery — add as much as you can truthfully supply.
