---
name: install-akm-kit
description: Use when the user wants to install or clone an akm kit (or individual asset) from npm, GitHub, a git URL, or a local directory into their stash — covers search, source registration, install, and verification.
---

# Install an akm Kit

This skill is for adding a kit to the user's stash so its assets (skills,
commands, agents, knowledge, workflows, wikis, vaults, memories) become
searchable and usable via `akm show`, `akm run`, or agent plugins.

## When to use

- "Install the X kit"
- "Add the GitHub repo Y as an akm source"
- "Grab just the deploy skill from kit Z"

Prerequisite: run the `akm-quickstart` skill first if `akm info` fails.

## Decision tree

1. **Don't know the exact ref?** → `akm search "<keywords>" --source registry`
   then `akm curate "<query>"` to get a summary of the top candidates.
2. **Have a kit ref (e.g. `owner/repo`, `@scope/pkg`)?** → `akm add`.
3. **Only want a single asset?** → `akm clone <ref>` — copies into the stash
   without registering the whole kit as a managed source.

## Install patterns

### From the official registry

```bash
akm search "kubernetes deploy" --source registry
akm add github:owner/kubernetes-kit           # add as managed GitHub source
akm update                                    # pull latest refs
akm index                                     # refresh search index
```

### From npm

```bash
akm add npm:@acme/review-kit
akm show skill:code-review                    # verify
```

### From a git URL or branch/tag

```bash
akm add git+https://github.com/owner/repo.git#v1.2.3
# pinning a tag is strongly preferred over tracking main
```

### From a local directory (for in-progress kits)

```bash
akm add ./path/to/kit --name local-kit
```

### Single asset via clone

```bash
akm clone github:owner/repo//skill:deploy     # copies just the skill
akm clone npm:@acme/kit//knowledge:runbook    # copies just one knowledge doc
```

Type subdirectories (`skills/`, `knowledge/`, etc.) are appended automatically
on the destination side.

## Verification

```bash
akm list                                      # confirm the source appears
akm search <asset-name>                       # confirm assets are indexed
akm show <ref>                                # inspect a specific asset
```

## Private kits

For private GitHub repos, set `GITHUB_TOKEN` in the environment (or in the akm
vault):

```bash
akm vault set GITHUB_TOKEN ghp_xxx
akm add github:your-org/private-kit
```

## Uninstall / replace

```bash
akm remove <source-id>      # id from `akm list`
akm update                  # reindex
```

## Pitfalls

- **Two kits export the same asset name** — always pass the fully-qualified ref
  (`github:owner/repo//skill:name`) to `akm show`/`akm run`.
- **Kit keeps re-downloading** — check `akm config get cache.ttl`; very short
  TTLs cause repeated clones.
- **`akm add` succeeded but `search` finds nothing** — you forgot `akm index`.
