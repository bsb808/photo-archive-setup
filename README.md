# photo-archive-setup

Notes, plans, scripts, and config for a self-hosted personal photo archive.

The guiding principle: **the files are the truth.** A date+event folder tree of plain image files, with captions and dates embedded in the files themselves (IPTC/EXIF). Any gallery app is a disposable read layer pointed at those files, so there's no proprietary database to get locked into and tools can be swapped freely.

## Target architecture

```
Layer 1  Local master      folder tree + digiKam (embedded metadata) = source of truth
Layer 2  Online layer       Immich (read-only external library) = mobile apps + family read-only sharing
Layer 3  Off-site backup    restic/Borg -> Backblaze B2 and/or rotated external drive
```

Remote access via Tailscale (trusted family) or Cloudflare Tunnel (public share links).

## Repo layout

| Path | Contents |
|------|----------|
| [docs/tradestudy.md](docs/tradestudy.md) | Full decision aid: requirements, candidate comparison, scoring, and the data-grounded review of the actual archive (Section 12). |
| [docs/plan.md](docs/plan.md) | Living plan & TODO. Phase 1 = lock decisions; Phase 2 = transition to the clean setup. Includes the decision log. |
| `scripts/` | Cleanup and metadata/date-normalization scripts (drafted, reviewed before running). |
| `config/` | Server config: Docker Compose, ignore patterns, backup config. |

## Status

Phase 1 (decisions & planning). See [docs/plan.md](docs/plan.md) for the current checklist and open decisions (D1–D7).

## Current archive (as reviewed 2026-05-29)

~89 GB, ~26,150 files across three trees: `PhotosDated/` (75 GB, `YYYYMMDD_event` folders), `PhotosScan/` (1.2 GB, scanned family photos named by decade), and an unsorted phone dump (13 GB). ~92% JPEG, one RAW file total. See the trade study for the full breakdown and cleanup/date-normalization tasks.
