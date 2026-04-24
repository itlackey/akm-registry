# Terminology Migration: "kit" → "stash"

This document records a one-time rename we landed in `itlackey/akm-registry`:
every prose and asset-name reference to a **kit** (a shareable, publishable
directory of assets) is now **stash**. The local editable directory akm has
always called a "stash" is disambiguated as a **working stash** where
confusion is possible.

This was done as a clean break because nothing had adopted the registry
yet — but it deliberately stops short of changing things we don't own. The
"Remaining work" section below is the punch list for fully closing out the
rename across the akm ecosystem.

## What changed in this repo

### Directory/file renames

| Before | After |
|---|---|
| `skills/install-akm-kit/SKILL.md` | `skills/install-akm-stash/SKILL.md` |
| `skills/publish-akm-kit/SKILL.md` | `skills/publish-akm-stash/SKILL.md` |
| `knowledge/akm-kit-structure.md` | `knowledge/akm-stash-structure.md` |
| `workflows/publish-kit.md` | `workflows/publish-stash.md` |
| `commands/akm-review-kit.md` | `commands/akm-review-stash.md` |

### Asset refs updated in `manual-entries.json` and `index.json`

| Before | After |
|---|---|
| `skill:install-akm-kit` | `skill:install-akm-stash` |
| `skill:publish-akm-kit` | `skill:publish-akm-stash` |
| `knowledge:akm-kit-structure` | `knowledge:akm-stash-structure` |
| `workflow:publish-kit` | `workflow:publish-stash` |
| `command:akm-review-kit` | `command:akm-review-stash` |
| Stash display name: `"akm Official Onboarding Kit"` | `"akm Official Onboarding Stash"` |

### Frontmatter `name` fields (YAML) updated to match filenames

`install-akm-stash`, `publish-akm-stash`, `publish-stash`, `akm-review-stash`.

### Prose changes (scope)

- README, all SKILL.md files, all knowledge/*.md, both workflows, both
  commands, and the librarian agent: every mention of "kit" / "Kit" /
  "kits" / "Kits" is now "stash" / "Stash" / "stashes" / "Stashes."
- Where "stash" is ambiguous between the local working directory and a
  publishable kit, prose now explicitly says **working stash** or
  **published stash**.
- "Agent Kit Manager" → "Agent Stash Manager" in skills.
- New publishing guidance uses the `akm-stash` npm keyword and GitHub topic.

## What we deliberately did NOT change

### 1. The `"kits"` JSON array key in `index.json`

Wire format. The pinned `akm-cli@0.0.22` parses this exact key when it
fetches the index. Renaming it to `"stashes"` would require a coordinated
CLI release. Until then:

- `index.json` top level keeps `"kits": [...]`.
- `scripts/registry-index.schema.json` (not touched in this pass) still
  validates `kits`.
- Our `bun run validate` npm script and `.github/workflows/update-index.yml`
  both reference `idx.kits` for the same reason.
- Documented with an in-line note in `knowledge/akm-registry-schema.md`.

### 2. Auto-discovery topics/keywords `akm-kit` and `agentikit`

The registry build merges three sources, two of which are auto-discovered
from external metadata:

- npm packages with `akm-kit` or `agentikit` in `keywords`
- GitHub repos with `akm-kit` or `agentikit` as a topic

These are baked into the current `akm-cli` build-index behavior. We now
recommend **`akm-stash`** going forward in our publishing docs, and call out
the legacy names as honored during the migration window. Flipping off the
legacy names requires an akm-cli change.

### 3. External URLs and package names

- `npm:akm-cli` — package name unchanged.
- `github:itlackey/akm` — repo name unchanged.
- `github:itlackey/akm-plugins` — repo name unchanged.
- `github:itlackey/akm-registry` — repo name unchanged.

These are identifiers owned by the akm author; renaming them would break
every existing consumer and is a separate decision.

### 4. Upstream documentation in `itlackey/akm/docs`

The upstream akm docs still use "kit" (e.g. "Kit Maker's Guide",
"manage… kits"). We reference those externally but do not link to them by
title from our prose any more.

### 5. The stale `../agentikit/package.json` reference in README

Line 131 of `README.md` retains the comment about the pin being chosen from
`../agentikit/package.json`. This is a historical note from when the CLI
repo was named `agentikit`; changing it risks misrepresenting why the
specific pin was selected. Flag for a maintainer cleanup.

## Remaining work (outside this repo)

The rename is only complete once the following land in other repos. Each
item below is a separate PR.

### In `itlackey/akm` (the CLI)

- [ ] **`build-index` output key.** Emit `stashes: [...]` (and stop emitting
      `kits: [...]`) in a major CLI release. Requires every registry consumer
      to upgrade in the same release cycle.
- [ ] **Index-format version bump.** Ship `version: 3` that uses `stashes`;
      keep reading `version: 2` `kits` for one minor cycle, then drop it.
- [ ] **Schema file.** Regenerate `registry-index.schema.json` to reflect
      the new key (and re-publish it here).
- [ ] **Auto-discovery topics/keywords.** Add `akm-stash` to the set of npm
      keywords and GitHub topics scanned by `akm registry build-index`.
      Optionally deprecate `akm-kit` and `agentikit` with warnings before
      removal.
- [ ] **CLI help text.** Every `akm <cmd> --help` message currently says
      "kit" where it means a published stash. Grep and replace.
- [ ] **Docs sweep.** `docs/concepts.md`, `docs/cli.md`,
      `docs/kit-makers-guide.md` (rename → `stash-authors-guide.md`),
      `docs/registry.md`, `docs/ref-format.md` (if it exists), plus the
      README.
- [ ] **Release notes / CHANGELOG.** A clear "breaking: kit → stash" entry
      with a pointer to this doc.

### In `itlackey/akm-plugins`

- [ ] Update prose in the Claude Code skill and OpenCode tool descriptions
      to say "stash" where they currently say "kit."
- [ ] Update tool names / slash-command names that contain "kit."

### In `itlackey/akm-registry` (this repo — follow-ups)

- [ ] Flip `index.json` top-level key to `stashes` once the CLI release
      above lands.
- [ ] Update `package.json` `validate` script and
      `.github/workflows/update-index.yml` in the same PR.
- [ ] Update `scripts/registry-index.schema.json` to the new key.
- [ ] Re-point `scripts/install-akm.ts` pin to the first CLI version that
      emits the new key.
- [ ] Recheck README's `../agentikit/package.json` reference (see §5 above).
- [ ] Announce the flip day on the akm discussions/issues so downstream
      registries can follow suit.

### For stash authors (external)

These are recommendations we'll want to surface in release notes / the
registry README once the CLI supports the new names end-to-end:

- [ ] Replace `"agentikit"` / `"akm-kit"` in `package.json` keywords with
      `"akm-stash"`.
- [ ] Replace the `agentikit` / `akm-kit` GitHub topic with `akm-stash`.
- [ ] Rename any `kit.json` manifest files to `akm.json` (or `stash.json`).
- [ ] Update READMEs and asset frontmatter descriptions from "kit" to
      "stash."

## Compatibility guarantees

Until the CLI release that flips the wire format lands, this registry is
**fully backwards-compatible** with existing akm-cli consumers:

- `index.json` still serves the `"kits"` top-level array.
- Auto-discovery still honors `akm-kit` and `agentikit` topics/keywords
  (nothing changed in the auto-discovery side; we only changed our prose
  recommendations).
- Every legacy asset ref that used to work (`skill:install-akm-kit`,
  `workflow:publish-kit`, etc.) is **gone** — these refs now return 404
  against `akm show`. This is the one breaking change we accepted because
  no consumer had adopted them yet.

## Why "stash"?

The akm docs already used "stash" for the user's local editable directory.
Overloading the word is a minor cost; the upside is that every asset — local
or shared — lives in "a stash," which removes the local-vs-shared mental
split agents had to maintain under the kit/stash dichotomy. Where the
distinction actually matters, the docs now say "working stash" vs.
"published stash."
