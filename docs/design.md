# Photo Archive — Design Document

Status: Phase 1 design, 2026-06-05. Companion to [tradestudy.md](tradestudy.md) (the *why*) and [plan.md](plan.md) (the task checklist). This document is the *what* and the rollout.

---

## 1. Design summary

The files are the truth: a flat `YYYYMMDD_event/` tree of plain images with captions and dates embedded in IPTC/EXIF. Three layers sit around that master — digiKam edits the metadata, Immich serves a disposable read-only view to phones and family, and an off-site backup protects against disaster. Any gallery app can be swapped without data loss because nothing of value lives only in its database.

```
   LAYER 1: MASTER     Dedicated drive, /photos/YYYYMMDD_event/ ...
   (truth)             digiKam = catalog + captions → embedded IPTC/EXIF
                              │ (read-only index, no copy)
   LAYER 2: ONLINE     Immich (Docker), read-only External Library.
   (disposable)        Mobile apps + family password share links.
                       Public reach via Cloudflare Tunnel.
                              │ (encrypted, dedup backup)
   LAYER 3: OFF-SITE   restic/Borg → Backblaze B2  +  rotated USB HDD off-site
```

Locked decisions:

| # | Decision | Choice | Note |
|---|----------|--------|------|
| D1 | Online layer | Immich, read-only External Library | Best mobile/search; never moves files |
| D2 | Metadata authority | digiKam, embedded IPTC/EXIF | Single place captions/dates are edited |
| D2b | Caption scope | Folder-level for ~all; per-photo names on scanned historical photos | Captioning 26k by hand is unrealistic |
| D3 | Scan date policy | Known year → `YYYY-01-01`; decade → mid-decade (`1955-01-01`) | Plausible `DateTimeOriginal` so the timeline works |
| D4 | Folder convention | Keep flat `YYYYMMDD_event`; phone dump sorted into `YYYY_phone/` by capture year | Light, consistent bucketing — promote to events later if wanted |
| D5 | Remote access | Cloudflare Tunnel + Immich password share links | Public HTTPS link + one shared password (R10) |
| D6 | Off-site backup | Both: B2 cloud + annual rotated USB HDD | Full 3-2-1 |
| D7 | Cleanup | Quarantine → review → delete | Reversible; nothing deleted before the safety backup |
| D8 | Videos | Separate `Videos/` store — backed up (Layer 3), not served by Immich | Photos-first; video size/serving deferred |

Scope — photos first. This design covers photos. The ~870 video files are moved out of the served photo tree into a separate `Videos/` store that is backed up like everything else (Layer 3) but not served by Immich — video size and streaming are a separate problem we're deliberately not taking on now. The folder, metadata, and backup approach here would extend to video later if that changes.

---

## 2. Build roadmap (high level)

Each phase is deliberately coarse here; we'll add command-level detail as we work through them. Approach: every phase that runs a script over the archive is prototyped on a small sample first — a couple of event folders plus a few scans — and validated end to end before it touches the full tree.

Phase 0 — Prep & staging
- Install `exiftool` and digiKam.
- Safety copy of the current archive (the rollback point) — done: backed up to an external drive. Nothing destructive happens before this exists.
- Pick the small prototype sample (a couple of event folders + a few scans) and run the whole pipeline on it first — cleanup, date-normalization, digiKam writeback, Immich indexing — before applying anything to the full archive.
- Start on the existing internal SSD copy as the working master; migrate to the dedicated drive later (cheap: re-point Immich's library path, re-index, re-target backups).

Phase 1 — Clean the tree
- Quarantine cruft (344 `@eaDir`, 178 `.db`/`.ini`/`Thumbs.db`, 349 old `.html`/`.css`, 15 stray non-photo files) → review → delete.
- Separate videos: move the ~870 video files out of the photo tree into the `Videos/` store (D8) — backed up, not served.
- Normalize folder names to the `YYYYMMDD_event` convention — standardize mixed date formats (→ `YYYYMMDD`) and dash/underscore separators. Semi-automated: a script proposes the renames, you review before applying.
- Date-normalize the scans per D3 (one-time scripted `exiftool` pass; dry-run one folder first).
- Sort the phone dump into year-level folders (`YYYY_phone/`) by capture date — a one-time `exiftool` pass.

Phase 2 — Metadata layer
- Point digiKam at the clean tree; configure embedded IPTC/EXIF writeback.
- Add folder-level descriptions; add per-photo names on the scanned historical photos.

Phase 3 — Online layer
- Stand up Immich via Docker as a read-only External Library on the master tree. (Docker because Immich ships as a multi-service stack — web/API server, machine-learning, PostgreSQL, Redis — that the project officially packages and supports only as a Docker Compose deployment. Compose runs and version-pins all of it from one file, isolates it from the host OS, and makes upgrades and rollbacks clean; installing those services bare-metal is unsupported and fragile.)
- Install mobile apps; validate the headline flow: find a 1972 and a 1996 photo by date.

Phase 4 — Remote access & sharing
- Register a domain; set up Cloudflare Tunnel to publish `https://photos.<domain>`.
- Create an Immich password share link; validate read-only access with one family member.

Phase 5 — Backup (3-2-1)
- Configure `restic`/`Borg` → Backblaze B2 (~$0.50/mo), scheduled.
- Set up the annual rotated USB HDD kept off-site.
- Test a restore (non-negotiable).

Phase 6 — Cutover
- Migrate the master to the dedicated drive; confirm Immich and backups follow.
- Confirm the clean setup is the working master; retire stale copies.

---

## 3. Maintenance plan (high level)

- Adding photos: drop into a `YYYYMMDD_event/` folder; caption in digiKam; Immich auto-rescans the external library.
- One metadata authority: edit captions/tags/dates only in digiKam. Immich stays read-only so the two never fight.
- Backups: B2 runs on a schedule; rotate the cold drive a few times a year; periodically confirm a backup actually ran.
- Restore drills: test a restore on a cadence (e.g. yearly), not just once at setup.
- Updates: occasional Docker image bumps for Immich; skim release notes for breaking changes before updating.
- Capacity: keep an eye on free space on the master drive and on B2 size as the archive grows.

---

## 4. Hardware recommendation (to confirm)

Live master drive (always-on, served by Immich): internal SATA SSD, ~2 TB. Quiet, reliable, no spin-up, well-suited to an always-on server. The archive is only ~89 GB today, so 2 TB is years of headroom including Immich's thumbnail cache. A 3.5" internal HDD also works and is cheaper, but at this small size the SSD's quiet reliability is worth it.

On the USB-vs-internal question: prefer an internal SATA connection for the live master. USB drives can sleep or drop off under an always-on Docker server; a quality USB SSD enclosure is acceptable only if you disable USB autosuspend, so internal is the lower-hassle default.

Cold backup drive (off-site, mostly unpowered): external USB HDD. Spinning disks tolerate long unpowered shelf storage better than SSDs (which can slowly lose charge over years), so HDD is the right choice for the rotated copy.

Staging now: build on the existing internal SSD copy and move to the dedicated drive in Phase 6. Migration is low-cost — copy the tree, re-point Immich's external-library path, re-index, and re-target the backup source.

---

## 5. Open items to refine

Domain name for the Cloudflare Tunnel (Phase 4) — need to pick/register one (~$12/yr).

digiKam captioning workflow for the scans — confirm the per-photo "who's in it" naming approach once digiKam is up.
