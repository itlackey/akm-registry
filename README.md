# akm Registry

Static registry index for [akm](https://github.com/itlackey/akm) stash discovery.
The CLI ships with this registry pre-configured and uses it for `akm registry search`
and `akm search --source registry`.

This repo is **also itself an akm stash** — the official onboarding stash. It
ships skills, knowledge, workflows, commands, and a librarian subagent that
teach agents how to discover, install, author, and publish akm assets. See
[Bundled onboarding stash](#bundled-onboarding-stash) below.

> **Terminology note.** Historically akm called a shareable directory of
> assets a "kit." We've renamed that concept to "stash" across this repo.
> The only remaining "kit" references are in places we don't control yet
> (the CLI's wire format and auto-discovery topics) — tracked in
> [`TERMINOLOGY-MIGRATION.md`](./TERMINOLOGY-MIGRATION.md).

## How it works

This repo publishes a static `index.json` file. The akm CLI fetches and caches
that file locally, then searches it for matching stashes. The index is built
here by a pinned `akm-cli` install via `akm registry build-index`. Current
CLI builds support:

- Registry index `version: 2`
- Auto-discovered stashes from npm and GitHub
- Curated manual entries that can override or enrich discovered metadata
- Optional asset-level metadata via each entry's `assets` array
- Multiple configured registries via the `registries` config field

The build merges three sources:

- npm packages with the `akm-stash` keyword *(new — see migration note)*
- GitHub repos with the `akm-stash` topic *(new — see migration note)*
- `manual-entries.json` for curated additions and overrides

> Until the CLI auto-discovery updates to the new topic/keyword names, the
> merge also still honors the legacy `agentikit` and `akm-kit` topic /
> keyword. Publishers should prefer `akm-stash` going forward.

## Getting listed

There are three ways to get listed:

- Publish an npm package with `akm-stash` in `keywords`
- Add the `akm-stash` GitHub topic to your repository
- Open a PR updating `manual-entries.json` for curated additions or overrides

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

Supported `assetTypes` values match the CLI's current asset model:
`script`, `skill`, `command`, `agent`, `knowledge`, `workflow`, `wiki`,
`vault`, `memory`.

## Index format

```json
{
  "version": 2,
  "updatedAt": "2026-04-24T00:00:00Z",
  "kits": [
    {
      "id": "github:your-org/deploy-stash",
      "name": "deploy-stash",
      "description": "Deployment scripts and skills",
      "ref": "your-org/deploy-stash",
      "source": "github",
      "tags": ["deploy", "infrastructure"],
      "assetTypes": ["script", "skill"],
      "assets": [
        {
          "type": "script",
          "name": "deploy.sh",
          "description": "Deploy to production"
        }
      ],
      "author": "your-org",
      "curated": true
    }
  ]
}
```

The top-level `kits` array key is wire format fixed by the CLI and is
intentionally **not** renamed; see
[`TERMINOLOGY-MIGRATION.md`](./TERMINOLOGY-MIGRATION.md) for why.

The schema lives at `scripts/registry-index.schema.json`.

The local wrapper lives at `scripts/build-index.ts`, and the pinned CLI install
is managed by `scripts/install-akm.ts` using `bun add --exact --no-save`.

## Configuring akm

akm uses this registry by default. You can add or override registries with:

```bash
akm registry list
akm registry add https://your-company.com/registry/index.json --name team
export AKM_REGISTRY_URL=https://your-company.com/registry/index.json
```

In config, registries live under the `registries` array.

## Local development

```bash
bun run install:akm
bun run build
bun run validate
```

The current pin is `akm-cli@0.0.22`, chosen as the next patch after
`../agentikit/package.json`'s `0.0.21` while the CLI release for
`akm registry build-index` is still in flight.

## Bundled onboarding stash

In addition to serving `index.json`, this repo is installable as an akm stash:

```bash
akm add github:itlackey/akm-registry
akm index
akm show skill:akm-quickstart
```

Layout:

```
skills/
  akm-quickstart/SKILL.md        Bootstrap akm in a fresh environment.
  install-akm-stash/SKILL.md     Install or clone a stash from npm/GitHub/git/local.
  publish-akm-stash/SKILL.md     Publish a directory of assets as a discoverable stash.
knowledge/
  akm-overview.md                Working stash / source / registry and the nine asset types.
  akm-cli-reference.md           Top-level CLI command reference (v0.5.0).
  akm-stash-structure.md         Canonical stash layout and asset frontmatter.
  akm-registry-schema.md         Registry index.json schema and manual-entry format.
workflows/
  publish-stash.md               End-to-end stash publishing workflow.
  onboard-agent.md               Bootstrap a coding agent onto akm.
commands/
  akm-find.md                    Prompt template: ranked shortlist for a stated need.
  akm-review-stash.md            Prompt template: evaluate a stash before install.
agents/
  akm-librarian.md               Read-only subagent that recommends akm assets.
```

These assets target the v0.5.0 release (2026-04-24), including the new
first-class `workflow`, `wiki`, and `vault` asset types.
