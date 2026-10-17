# droidhorde

A multi-source Android APK index. One static `index.html` (11.7 MB), served
from GitHub Pages: **https://crtghoul.github.io/droidhorde/**

v1.3: **7,823 apps / 21,245 builds**, deduped by Android package name.
Search, browse by category, check version histories per source, and compare
stock builds against modded ones.

Everything is free — no accounts, no paywalls, no download gates. The index
points at sources anyone can use; it just makes them findable in one place.

## Sources

9 sources. Dedup happens on the Android **package name** (`com.spotify.music`
etc.) — the one identifier every source agrees on.

| Source | Type | Modded |
|--------|------|--------|
| F-Droid | Curated open-source catalog, human-reviewed | no |
| IzzyOnDroid | F-Droid-style repo, curated | no |
| Aptoide | Commercial store API | no |
| APKPure | Commercial store API | no |
| APKCombo | Commercial store scraping | no |
| LiteAPKs | Modded APK site | **yes** |
| APKVision | Modded APK site | **yes** |
| HappyMod | Modded APK site | **yes** |
| AN1 | Modded APK site | **yes** |

Official sources render a `★ OFFICIAL` pill. Modded sources get a dashed
`MOD` badge so stock vs. repack is obvious at a glance. Apps that only exist
on mod sources are excluded from the "most wanted" / fresh-drops rails —
mod download counts are marketing numbers — but they stay fully indexed and
searchable.

Presigned/expiring direct links are **never stored**. Snapshots, the DB, and
the HTML carry only canonical download *page* URLs; direct links are resolved
transiently at build time and discarded (a presigned-leak self-check fails
the build if a tokenized URL slips through).

## Building the index

```bash
python3 build.py                  # full rebuild: fetch -> normalize -> dedupe -> trust -> render
python3 build.py --offline        # reuse sources/*.json snapshots, no network
python3 build.py --out out.html   # custom output path (default: index.html)
python3 build.py --skip-safety    # skip the URLhaus + VirusTotal scans
```

Pipeline: fetch the source snapshots, normalize to one record shape, dedupe
by package name, run the trust model (signature agreement, curated
provenance, version-vs-canonical, signer-change, mod keywords), render
`index.html` with the DB embedded as `const DB`.

Deterministic: identical inputs produce byte-identical output. The build
prints `db_hash=<16 hex>` so refreshes can skip no-op commits.

Per-source working files live next to `build.py` (probe caches, link
health, VT/URLhaus caches). They make rebuilds resumable and keep sources
that were unreachable on one run listed anyway.

## Refresh cadence

Manual, on request. There is no auto-refresh cron — run `build.py` when a
rebuild is wanted and push the resulting `index.html` + `fresh.xml`.

## Safety methodology

A build-time trust layer, not a guarantee. Treat every download as
"install unknown apps" and verify yourself.

- **URLhaus bulk sweep** — download URLs are checked against URLhaus
  (needs a free URLhaus auth key; without one it skips loudly). A listed
  URL gets a hard `bad` risk entry.
- **VirusTotal verdicts (modded sources)** — VT results on modded apps are
  shown as raw engine counts, e.g. "2/92 flagged". Never a binary
  clean/dirty badge — a single bad badge is worse than the counts.
- **Signature-fingerprint trust** — when multiple sources report the same
  app, their signing fingerprints are compared. Agreement raises trust;
  disagreement flags the app (`risk: bad`), because a signer change is the
  classic repack signal.
- **Version-vs-canonical** — a version newer than the F-Droid/Izzy
  canonical build is flagged as suspicious.

## Supporting files

- `seed_packages.json` — the package-name seed list driving per-source
  probing. F-Droid + Izzy package names plus previously successful probes,
  sorted, rewritten by every build. Feeds the query-based sources
  (Aptoide, APKPure, APKCombo).
- `fresh.xml` — RSS feed of builds from the last 7 days, written by
  `write_rss` during the build.

## License

MIT. See [LICENSE](LICENSE).
