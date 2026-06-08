# Photo Archive — Plan & TODO

Living checklist for moving from the current `/data/Vault/Photos` archive to a clean, self-hosted setup.
Companions: [tradestudy.md](tradestudy.md) (the *why*) and [design.md](design.md) (the *what* + rollout). Decisions D1–D8 are locked in the design doc; this file tracks tasks.

_Started 2026-05-29. Decisions locked 2026-06-08._

**Working rule:** Phase 1 (decide + plan) completes before Phase 2 (transition) begins. Nothing destructive happens to the master until decisions are locked and a backup exists.

---

## Phase 0 — Prerequisites

- [ ] Install exiftool — **run manually (needs sudo password):**
  ```bash
  sudo apt-get update && sudo apt-get install -y libimage-exiftool-perl
  ```
- [ ] Confirm `exiftool -ver` works.
- [ ] Install digiKam.
- [x] Target drive decided: buy a dedicated internal SATA SSD (~2 TB); stage on the existing internal SSD for now and migrate later (design.md §4).
- [ ] Note current free space: `df -h /data`

---

## Phase 1 — Workflow choices & plan (DONE)

All decisions locked in [design.md](design.md) (D1–D8); recorded in the decision log below. Summary:

### D1 — Online/sharing layer
- [x] **Immich**, read-only External Library. Nextcloud+Memories is the documented swap-in if literal local↔online sync later outweighs mobile polish.

### D2 — Metadata authority & caption workflow
- [x] **digiKam** is the single editor, writing embedded IPTC/EXIF (no sidecars — archive is ~all JPEG, 1 RAW).
- [x] Caption strategy: folder-level for ~all photos; per-photo names on the scanned historical photos.

### D3 — Date normalization policy for scans
- [x] Known year → `YYYY-01-01`; decade → mid-decade (`1955-01-01`). Every scan gets a plausible `DateTimeOriginal`.

### D4 — Folder/organization policy going forward
- [x] Keep flat `YYYYMMDD_event` as canonical; normalize legacy folder names to it (date formats + dash/underscore separators).
- [x] Phone dump → year-level folders (`YYYY_phone/`) by capture date; promote to events later if wanted.

### D5 — Remote access & sharing
- [x] **Cloudflare Tunnel** (public HTTPS) + **Immich password share links** — a single shared password, no accounts or apps (R10).
- [ ] Audience: who gets the link (family count/comfort still open — see parking lot).

### D6 — Off-site backup strategy
- [x] Both: `restic`/`Borg` → Backblaze B2 (~$0.50/mo) + an annual rotated USB HDD off-site. Full 3-2-1.

### D7 — Cleanup destructiveness
- [x] Quarantine → review → delete. Nothing deleted before the safety backup exists.

### D8 — Videos (added during design review)
- [x] Separate `Videos/` store — backed up (Layer 3), not served by Immich. Photos-first.

**Exit criteria for Phase 1:** D1–D8 answered (✔ design.md), written plan agreed (✔ design.md), full backup of the current archive exists (✔ external drive). Phase 1 complete.

---

## Phase 2 — Transition to clean setup

> Do NOT start until Phase 1 exit criteria are met. Order matters: back up first, work on a copy, validate, then cut over.
> Prototype each scripted step on a small sample (a couple of event folders + a few scans) before running it on the full archive (design.md §2).

### Step 1 — Safety backup
- [x] Full copy of the current archive made (external drive) — the rollback point.

### Step 2 — Cleanup pass *(script to be drafted — item #2 from review)*
- [ ] Draft cleanup script (Claude to write, user reviews before running).
- [ ] Quarantine **344 `@eaDir`** Synology cache dirs → review → delete.
- [ ] Quarantine **178** `.db` / `.ini` / `Thumbs.db` files → review → delete.
- [ ] Quarantine **349** old web-gallery `.html` / `.htm` / `.css` files → review → delete.
- [ ] Relocate **15** stray non-photo files (`.bag`, `.mat`, `.m`, `.dcm`) out of the photo tree.
- [ ] Review `.zip` / `.rar` / `.psd` / `.xcf` files — keep, extract, or archive separately?
- [ ] Separate the ~870 video files into a `Videos/` store (D8) — backed up, not served.
- [ ] Normalize folder names to `YYYYMMDD_event` (standardize date formats and dash/underscore separators); semi-automated, review proposed renames before applying.
- [ ] Re-run the file-type tally to confirm a clean tree.

### Step 3 — Date normalization *(script to be drafted — item #2 from review)*
- [ ] Draft `exiftool` batch script that sets `DateTimeOriginal` from each `PhotosScan/` folder name per the D3 policy (Claude to write, user reviews).
- [ ] Dry-run on one folder; verify with `exiftool` readout.
- [ ] Apply to all scan folders.
- [ ] Spot-check that scans now carry plausible dates.
- [ ] Sort the phone dump into `YYYY_phone/` folders by EXIF capture year (one-time `exiftool` pass; D4).

### Step 4 — Captioning (digiKam)
- [ ] Install digiKam; point it at the cleaned tree.
- [ ] Configure metadata writeback (embedded IPTC/EXIF).
- [ ] Caption per the D2 strategy. Confirm captions land in the files (`exiftool` check).

### Step 5 — Online layer (Immich or chosen tool)
- [ ] Stand up the server via Docker.
- [ ] Configure as **read-only external library** pointing at the master tree (no copying/moving).
- [ ] Add ignore patterns for any remaining cruft.
- [ ] Install mobile app; validate end to end: **find a 1972 and a 1996 photo by date**.

### Step 6 — Remote access & sharing
- [ ] Set up Tailscale or Cloudflare Tunnel per D5.
- [ ] Add one family member; confirm read-only access works.

### Step 7 — Off-site backup
- [ ] Configure restic/Borg → B2 and/or rotated drive per D6.
- [ ] Run first backup.
- [ ] **Test a restore** (non-negotiable).

### Step 8 — Cutover & retire old layout
- [ ] Confirm clean setup is the working master.
- [ ] Decommission the old Synology footprint / stale copies once confident.

---

## Open items / parking lot

- Domain name to register for the Cloudflare Tunnel (~$12/yr).
- Audience for the share: family count and technical comfort (D5 sub-item).
- Do you still own the Synology that left `@eaDir` behind? (Possible low-effort fallback.)
- Want face recognition / scene search, or is date+event enough? (Immich does the former; doesn't change D1, just which features to enable.)
- Handling of `.psd`/`.xcf` edit files and `.zip`/`.rar` archives in the tree.
- Off-site drive logistics: cadence and where the rotated HDD lives (D6 detail).

---

## Decision log

_Locked decisions (date — decision — rationale). Full detail in [design.md](design.md)._

- 2026-06-08 — **D1 Immich** (read-only External Library) — best mobile/search; never moves files; truth stays in the files so it's swappable.
- 2026-06-08 — **D2 digiKam, embedded IPTC/EXIF**; folder-level captions for ~all, per-photo names on scans — single metadata authority; hand-captioning 26k is unrealistic.
- 2026-06-08 — **D3 scan dates**: year → `YYYY-01-01`, decade → mid-decade — plausible `DateTimeOriginal` so old scans sort on the timeline.
- 2026-06-08 — **D4 folders**: keep flat `YYYYMMDD_event`, normalize legacy names; phone dump → `YYYY_phone/` — clean, consistent, date-sortable.
- 2026-06-08 — **D5 Cloudflare Tunnel + Immich password share links** — single shared password, no accounts/apps (R10); read-only by construction.
- 2026-06-08 — **D6 B2 + rotated USB HDD** — full 3-2-1; cloud for recency (~$0.50/mo), cold drive for the worst case.
- 2026-06-08 — **D7 quarantine → review → delete** — reversible; nothing deleted before the safety backup.
- 2026-06-08 — **D8 videos in a separate `Videos/` store**, backed up but not served — photos-first; video size/serving deferred.
