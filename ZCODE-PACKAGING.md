# ZCode packaging of frontend-design

A thin packaging shell around the `frontend-design` plugin that ships inside
[anthropics/claude-plugins-public](https://github.com/anthropics/claude-plugins-public)
at `plugins/frontend-design`. Upstream content is mirrored as-is by a daily sync.

## What the packaging adds, and what it does not touch

Upstream content needs no behaviour change on ZCode: the plugin is skills-only
and registers **no hooks**, so it never hits the executable-bit problem that
breaks plugins like superpowers (see `kingmaozi/superpowers-zcode`). What it
lacks is a ZCode manifest and any version at all, so:

| Path | Owner | Note |
| --- | --- | --- |
| `skills/`, `README.md`, `LICENSE` | upstream | mirrored verbatim, replaced on every sync |
| `.zcode-plugin/plugin.json`, `.claude-plugin/plugin.json` | **packaging** | ZCode manifest, declares `skills` |
| `scripts/sync-from-upstream.sh`, `.github/`, `SYNCED-FROM` | **packaging** | sync machinery |

## Versioning

Upstream declares **no version** — not in the plugin manifest, not in the
marketplace entry of its own repository. A marketplace that reads its version
therefore sees nothing to compare, which is why the official listing installs as
`0.0.0`.

This packaging owns a version of its own, starting at `1.0.0`. `sync-from-upstream.sh`
takes a patch bump on every sync in which upstream moved, which is what makes
ZCode offer an update; it would adopt an upstream version only if upstream ever
declared one higher than ours.

## Syncing

`.github/workflows/zcode-sync-upstream.yml` runs the sync daily (`23 3 * * *`)
and on demand. Upstream is a 300+ plugin monorepo, so the script uses a partial,
sparse clone (`--filter=blob:none --sparse`, then
`sparse-checkout set plugins/frontend-design`) and only that one subtree is
fetched.

The script derives its root from `$0` and then runs `rsync --delete`, so it
refuses to start unless that root looks like this repository
(`.zcode-plugin/plugin.json` and `skills/` present, and a git work tree).

Keep this repository public: ZCode resolves a `{"source":"github"}` entry by
fetching `api.github.com/repos/<owner>/<repo>/zipball/<ref>` with no
Authorization header, so a private packaging repo answers 404 and an installed
plugin is never offered an update. Only the marketplace may be private.

After a sync lands, ZCode still needs the plugin update action — a market refresh
updates the catalog, not the installed copy.
