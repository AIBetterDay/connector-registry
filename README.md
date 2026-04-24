# Better Connector Registry

Curated catalog the [Better](https://github.com/AIBetterDay) host fetches at
startup to populate its **Connectors store**. This repo is a single source of
truth — listing here = appearing in the store; un-listing = quiet de-list
(already-installed users keep working until they uninstall).

## How it works

```
Better host (apps/server)
   │
   │ GET https://raw.githubusercontent.com/AIBetterDay/connector-registry/main/index.json
   │ (cached with ETag, refreshed on app start + every N hours)
   │
   ▼
index.json
   │
   │ For each entry → render a "Not installed" card in the store
   │ User clicks "Install" → host hits installFromGithub(entry.repo)
   │   which downloads the latest release's .bcx, validates manifest,
   │   extracts under <userData>/installed-connectors/<vendor>/<id>/,
   │   activates skills subprocess.
   ▼
Installed in user's Better
```

## Adding your connector

Submit a PR to this repo. Edit `index.json`, add an entry that conforms to
`schema.json`. Reviewers (AIBetterDay maintainers) check:

1. **Repo exists and has a release.** The release must have a `.bcx` asset
   produced by `npx better-connector pack` from the repo's source.
2. **`id` matches `<vendor>.<name>`** and the `vendor` segment equals your
   GitHub user / org name (case-insensitive). This is the same rule the host
   enforces at install — entries that fail this can't be installed even if
   merged, so the PR check just spares users a confusing failure.
3. **`official: true` is only for AIBetterDay-owned repos.** Anyone else gets
   `official: false` and shows up in the store without the violet badge.
4. **No `id` collisions.** First-come, first-served on the namespaced id
   (`pazchen.wishes` and `aibetterday.wishes` can both exist; two
   `pazchen.wishes` from different repos can't).

## What this registry does NOT do

- **Audit code.** This is a catalog, not a security review. The Better host
  enforces sandboxing (manifest permissions, subprocess isolation) regardless
  of how a connector got installed. Index merging is "yes this developer is
  publicly listing this artifact" — nothing more.
- **Host the artifacts.** The `.bcx` always lives in the developer's GitHub
  release. We just point to where it is.
- **Manage versions.** The host always installs the *latest* release of the
  listed repo. To freeze a version, pin via the install URL flow instead of
  the registry.

## Schema

See `schema.json`. Validate before opening a PR:

```bash
npx ajv-cli validate -s schema.json -d index.json
```
