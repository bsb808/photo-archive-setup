# Photo Archive — Plan & TODO

Living checklist for moving from the current `/data/Vault/Photos` archive to a clean, self-hosted setup.
Companion to [tradestudy.md](tradestudy.md).

_Started 2026-05-29._

**Working rule:** Phase 1 (decide + plan) completes before Phase 2 (transition) begins. Nothing destructive happens to the master until decisions are locked and a backup exists.

---

## Phase 0 — Prerequisites

- [ ] Install exiftool — **run manually (needs sudo password):**
  ```bash
  sudo apt-get update && sudo apt-get install -y libimage-exiftool-perl
  ```
- [ ] Confirm `exiftool -ver` works.
- [ ] Identify the target drive for the clean master (existing free space vs. new HDD/SSD to add).
- [ ] Note current free space: `df -h /data`

---

## Phase 1 — Workflow choices & plan (DO THIS FIRST)

Decisions to lock before touching the archive. Each is a discussion point with Claude.

### D1 — Online/sharing layer
- [ ] **Decide:** Immich (best mobile/search, read-only external library) vs. Nextcloud+Memories (true local↔online sync) vs. PhotoPrism (folder+sidecar native).
  - Leaning: **Immich** per trade study. Revisit if literal bidirectional sync matters more than mobile polish.

### D2 — Metadata authority & caption workflow
- [ ] **Decide:** confirm **digiKam** as the single place captions/tags/dates are edited, writing **embedded IPTC/EXIF** (sidecars unnecessary — archive is ~all JPEG).
- [ ] **Decide:** caption strategy — folder-level notes only, or per-photo captions for select highlights? (Captioning 26k photos by hand is unrealistic; pick what gets captioned.)

### D3 — Date normalization policy for scans
- [ ] **Decide:** date convention for `PhotosScan/` folders named by decade/year.
  - `1972_christmas` → `1972-12-25`? or `1972-01-01` placeholder?
  - `1950s_withers` → `1955-01-01`? (mid-decade placeholder)
  - Goal: every scan gets a plausible `DateTimeOriginal` so the timeline works.

### D4 — Folder/organization policy going forward
- [ ] **Decide:** keep `YYYYMMDD_event` as the canonical convention (it's clean and consistent).
- [ ] **Decide:** plan for the phone dump — leave loose (dates work from filenames) vs. sort into event folders over time.

### D5 — Remote access & sharing
- [ ] **Decide:** Tailscale (trusted-family VPN, simplest) vs. Cloudflare Tunnel (public share links).
- [ ] **Decide:** who gets access, and how non-technical they are.

### D6 — Off-site backup strategy
- [ ] **Decide:** restic/Borg → Backblaze B2 (~$0.50/mo at current size) vs. rotated external drive vs. both.
- [ ] **Decide:** backup cadence and where the off-site drive lives.

### D7 — Cleanup destructiveness
- [ ] **Decide:** delete cruft outright vs. move to a quarantine folder for review first.

**Exit criteria for Phase 1:** D1–D7 answered, a written transition plan agreed, and a full backup of the current archive exists.

---

## Phase 2 — Transition to clean setup

> Do NOT start until Phase 1 exit criteria are met. Order matters: back up first, work on a copy, validate, then cut over.

### Step 1 — Safety backup
- [ ] Make a full, verified copy of `/data/Vault/Photos` before any edits (this is the rollback point).

### Step 2 — Cleanup pass *(script to be drafted — item #2 from review)*
- [ ] Draft cleanup script (Claude to write, user reviews before running).
- [ ] Remove/quarantine **344 `@eaDir`** Synology cache dirs.
- [ ] Remove/quarantine **178** `.db` / `.ini` / `Thumbs.db` files.
- [ ] Remove/quarantine **349** old web-gallery `.html` / `.htm` / `.css` files.
- [ ] Relocate **15** stray non-photo files (`.bag`, `.mat`, `.m`, `.dcm`) out of the photo tree.
- [ ] Review `.zip` / `.rar` / `.psd` / `.xcf` files — keep, extract, or archive separately?
- [ ] Re-run the file-type tally to confirm a clean tree.

### Step 3 — Date normalization for scans *(script to be drafted — item #2 from review)*
- [ ] Draft `exiftool` batch script that sets `DateTimeOriginal` from each `PhotosScan/` folder name per the D3 policy (Claude to write, user reviews).
- [ ] Dry-run on one folder; verify with `exiftool` readout.
- [ ] Apply to all scan folders.
- [ ] Spot-check that scans now carry plausible dates.

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

- Do you still own the Synology that left `@eaDir` behind? (Possible low-effort fallback.)
- Family count and technical comfort (affects D5).
- Want face recognition / scene search, or is date+event enough? (affects D1).
- Handling of `.psd`/`.xcf` edit files and `.zip`/`.rar` archives in the tree.

---

## Decision log

_Record locked decisions here as Phase 1 progresses (date — decision — rationale)._

- _(none yet)_
