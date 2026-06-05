# Photo Archive & Distribution — Trade Study

Decision aid for a personal photo archive: a durable local master plus a mobile, read-only online layer for family, with strong guarantees against vendor lock-in.

*Prepared 2026-05-29; grounded in a scan of the actual `/data/Vault/Photos` archive (Section 3).*

---

## 1. Requirements

| # | Requirement | Priority |
|---|-------------|----------|
| R1 | Plain files on disk as the durable master; never trapped in a proprietary database | Critical |
| R2 | Local deep-backup copy stays in sync with the online sharing copy | Critical |
| R3 | Captions/notes embedded in file metadata (IPTC/EXIF), XMP sidecar as fallback | High |
| R4 | Mobile (iOS + Android) browse/search, e.g. find a 1996 photo by date | High |
| R5 | Read-only sharing for friends & family | High |
| R6 | Self-host on an existing Linux-friendly desktop (with guidance) | High |
| R7 | Fixed cost strongly preferred; subscriptions disliked but acceptable | Medium |
| R8 | Date+event folder organization preserved; flexible on tooling adding a timeline | Medium |
| R9 | Off-site disaster protection | High |
| R10 | Viewers reach the share with a single shared password — no per-person accounts or app installs | High |

Context: ~89 GB archive (measured, Section 3); Linux desktop with room for a dedicated drive; eager to self-host; skeptical of lock-in; family browses but never uploads.

---

## 2. Core principle: the files are the truth

Polished photo apps (Google Photos, Immich, PhotoPrism, Synology Photos) are database-driven — the index, thumbnails, and search live in their database, and so can the meaning you add (captions, tags, albums, ratings). Keep that meaning in the files instead: folders plus embedded IPTC/EXIF. Any gallery then becomes a disposable read layer you can rebuild or swap freely. This one decision satisfies R1, R3, and most of R8 no matter which app you pick.

Metadata specifics: JPEG, HEIC, TIFF, and PNG take captions and tags cleanly in-file. Only camera RAW needs an `.xmp` sidecar, since writing into proprietary RAW containers risks corruption (Adobe, digiKam, darktable, and exiftool all follow this open convention). Your archive holds exactly one RAW file (Section 3), so embed in-file and skip the sidecar workflow. digiKam is the metadata authority; gallery apps only read what it writes.

---

## 3. The actual archive (measured 2026-05-29)

| Tree | Size | Notes |
|------|------|-------|
| `PhotosDated/` | 75 GB, 365 event folders | `YYYYMMDD_event` convention, back to `19980713_montreal_trip`. Clean and consistent. |
| `PhotosScan/` | 1.2 GB, 23 folders | Scanned family photos named by theme/decade (`1950s_withers`, `1972_christmas`, `1985_vail_beavercreek`). |
| `BrianPhoneGalaxyS20_June2023/` | 13 GB, 2,557 loose files | Phone dump, `YYYYMMDD_HHMMSS.jpg` filenames. Unorganized. |
| Total | ~89 GB, ~26,150 files | Bottom of the 100 GB–1 TB tier. |

File mix: ~24,000 JPEG (92%), 438 HEIC, ~870 video (mp4/mov/avi/wmv/3gp), one RAW.

What this implies:

- Cost is a rounding error. At ~89 GB, off-site cloud backup runs ~$0.50/month and the whole archive fits the cheapest drive sold. R7 is trivially met.
- One required step — date-normalize the scans. `PhotosScan/` files carry no real capture date (blank or scan-date EXIF) and live in decade-named folders, so a timeline would pile them at "today," breaking "find a 1972 photo by date" for exactly the oldest photos. Fix: a one-time exiftool/digiKam batch writing an approximate `DateTimeOriginal` (e.g. `1972-01-01`) derived from each folder name.
- Cleanup before indexing: 344 `@eaDir` Synology caches, 178 `.db`/`.ini`/`Thumbs.db`, 349 old `.html`/`.htm`/`.css` gallery exports, and 15 stray non-photo files (`.bag`/`.mat`/`.m`/`.dcm`). Exclude at index time, delete from the master once reviewed.
- The phone dump has usable dates in filenames and EXIF, so it timelines correctly even before you sort it into event folders.

---

## 4. Candidates

### Group A — Self-hosted gallery (the online layer)

| Tool | What it is | Plain-file fit | Metadata writeback | Mobile | Read-only share | Lock-in |
|------|-----------|----------------|--------------------|--------|-----------------|---------|
| Immich | Modern Google-Photos-style server, very active | External Library indexes the tree read-only, in place (no copying) | Reads EXIF/IPTC well; writeback limited — not the caption authority | Excellent native iOS + Android | Shared albums + public links | Low (read-only external library) |
| PhotoPrism | Go indexer, originals-first | Read-only originals mode; folder view first-class | Good; writes XMP sidecars | PWA, no native app | Albums + share links | Low |
| Nextcloud + Memories | File-sync platform + photo timeline | Files stay plain; true bidirectional desktop sync (its R2 superpower) | Memories can write EXIF; files stay authoritative | Native iOS + Android | Shared folders/links, read-only | Low |
| Photoview / Lychee / LibrePhotos | Lightweight galleries | Read-only folder index, simple | Minimal | PWA | Basic share links | Low, fewer features |

### Group B — Managed services (for contrast)

| Service | Strengths | Conflict with priorities |
|---------|-----------|--------------------------|
| Google Photos | Best-in-class search & mobile | No real folders, DB-owned metadata, recurring fee, Takeout-only escape. Fails R1/R8. |
| Amazon Photos | Free full-res for Prime | Tied to Prime, weak organization, lock-in. Fails R1/R7. |
| iCloud Photos | Great if Apple-centric | Hostile to Linux; DB-owned. Fails R6. |
| SmugMug / Flickr | Polished public sharing | Subscription, cloud-owned masters. Fails R1/R7. |
| Synology Photos | Fixed hardware cost, folder-respecting, good apps, no monthly fee | Requires buying a NAS, not free-form Linux. Closest managed-feel option that still respects folders. |

---

## 5. Weighted scoring

Criteria weighted by priority (5 = best fit).

| Criterion (weight) | Immich | PhotoPrism | Nextcloud+Memories | Google Photos | Synology |
|--------------------|--------|------------|--------------------|---------------|----------|
| No lock-in / plain files (×5) | 4 | 5 | 5 | 1 | 4 |
| Local↔online sync (×5) | 3 | 3 | 5 | 2 | 4 |
| Embedded metadata authority (×4) | 2 | 4 | 3 | 1 | 3 |
| Mobile browse/search (×4) | 5 | 3 | 4 | 5 | 4 |
| Family read-only sharing (×4) | 4 | 4 | 4 | 4 | 4 |
| Self-host on existing Linux box (×4) | 5 | 5 | 5 | 0 | 1 |
| Fixed cost (×3) | 5 | 5 | 5 | 2 | 4 |
| Search/timeline quality (×3) | 5 | 4 | 3 | 5 | 4 |
| Weighted total | ~129 | ~134 | ~138 | ~78 | ~118 |

Scores are directional. The top three are close; the pick turns on which Critical item dominates: mobile/search polish (Immich), bidirectional local sync (Nextcloud), or folder+sidecar fidelity (PhotoPrism). This study weights mobile experience highest — see Section 6.

---

## 6. Recommended architecture

A three-layer design where the files are the truth and the gallery is disposable.

```
   LAYER 1: MASTER     Existing desktop + dedicated drive
   (truth lives here)  /photos/YYYY-MM-DD_event/ ...
                       digiKam = catalog + captions → embedded IPTC/EXIF
                              │ (read-only index, no copy)
   LAYER 2: ONLINE     Gallery server (Immich) on the same machine, Docker.
   (disposable)        Mobile apps + family read-only share.
                       Remote access via Cloudflare Tunnel.
                              │ (encrypted, dedup backup)
   LAYER 3: OFF-SITE   restic/Borg → Backblaze B2 + rotated external drive
   (disaster recovery)
```

Layer 1 — local master. Keep the `YYYY-MM-DD_event/` tree on a dedicated drive (a 2–4 TB drive, ~$60–120, is years of headroom). digiKam catalogs and writes embedded IPTC/EXIF. This layer is independent of any server: if every gallery app vanished, you still have organized, captioned, portable files.

Layer 2 — online. Immich as a read-only External Library pointing at Layer 1. It indexes in place, never moves files, and gives the best phone experience for "find a 1996 photo by date." Edit captions in digiKam, not Immich (limited writeback). If literal local↔online sync (R2) later matters more than mobile polish, swap to Nextcloud + Memories at no data cost — because the truth is in the files, the switch costs only re-indexing time.

Layer 3 — off-site. Apply 3-2-1 (3 copies, 2 media, 1 off-site). `restic`/`Borg` to Backblaze B2 (~$0.50/mo at this size) and/or a rotated external drive at a relative's house. A home server is one fire or ransomware event from total loss, so this is non-negotiable. Test a restore once — an untested backup is a hope, not a backup.

Recommendation: digiKam (metadata authority) + Immich (read-only mobile/family layer) + off-site backup.

---

## 7. Remote access and the shared-password share (R5, R10)

Two separate questions: reachability (how a phone reaches your server) and authentication (what a viewer must prove).

Reachability. Cloudflare Tunnel (free) publishes a clean `https://photos.yourdomain` with no open ports; a viewer just opens the link in a browser. Tailscale is simpler and safer but requires each viewer to install the app and join your network, which fails R10's "no app, no account" — keep it for your own devices, not casual family.

Authentication — one password, no account, read-only, cleanest first:

- Immich password-protected share link (recommended). Immich is account-based for you, but generates public share links scoped to an album or selection, with optional password and expiry. The recipient opens the link, types the password, and browses read-only — no account, no app — and you revoke by regenerating. Caveat: links are per-album, not the whole timeline, so "share everything" means one large shared album or a few album links.
- Whole-site gate (alternative). Cloudflare Access (a PIN/password challenge) or a Caddy reverse proxy with HTTP Basic Auth puts one password over the entire site, behind which a single shared read-only Immich account gives the full timeline. The trade: read-only is softer on a shared account, and you share credentials rather than a link password.
- Nextcloud public link. Password + expiry on a whole folder, read-only, mature — the cleanest "one password for the whole collection" if that outweighs Immich's mobile polish.

Recommendation for R10: Cloudflare Tunnel + Immich password share links. Serve only over HTTPS (the tunnel provides it) and set an expiry; the shared password guards a read-only view, never the master, which never leaves Layer 1.

---

## 8. Cost

| Item | One-time | Recurring |
|------|----------|-----------|
| Dedicated 2–4 TB drive | $60–$120 | — |
| Gallery software (open source) | $0 | $0 |
| Cloudflare Tunnel / Tailscale | $0 | $0 |
| Off-site cloud backup (B2, ~89 GB) | $0 | ~$0.50/mo (or $0 with rotated drive) |
| Domain name (optional, for public links) | — | ~$12/yr |

One-time hardware plus near-zero recurring, matching the fixed-cost preference. The only optional subscription is cloud backup, replaceable by a rotated drive.

---

## 9. Self-host vs. managed

| Dimension | Self-host (recommended) | Managed (e.g. Google Photos) |
|-----------|-------------------------|------------------------------|
| Control / lock-in | Own everything; switch tools freely | Vendor owns the experience; export-only escape |
| Cost | One-time hardware, ~$0/mo | Recurring subscription, grows with storage |
| Mobile/search | Very good (Immich), you maintain it | Best-in-class, zero maintenance |
| Maintenance | Updates, backups, occasional troubleshooting | None |
| Uptime | Depends on home power/network | Always on |
| Privacy | Photos stay on your hardware | Scanned by the provider |
| Failure mode | You must keep off-site backup | Provider handles redundancy |

The real cost of self-hosting is your time and attention. In exchange you get permanent control and no subscription, which fits the stated values — with off-site backup (Layer 3) as the non-negotiable condition, since you are now your own redundancy.

---

## 10. Action plan

1. Install `exiftool` (not currently on the box) and digiKam.
2. Add the dedicated drive; migrate the `YYYY-MM-DD_event/` tree as-is.
3. Cleanup pass: remove/ignore `@eaDir`, `Thumbs.db`/`.db`/`.ini`, old `.html`/`.css`; relocate stray `.bag`/`.mat`/`.m`/`.dcm`.
4. Date-normalize `PhotosScan/` with a one-time `exiftool` batch (dry-run one folder, verify, then apply).
5. Caption in digiKam (embedded IPTC/EXIF); confirm the data lands in the files.
6. Stand up Immich via Docker as a read-only External Library; install the mobile app; validate "find a 1972 / 1996 photo by date."
7. Set up Cloudflare Tunnel + an Immich password share link; add one family member to confirm read-only access.
8. Configure `restic`/`Borg` → B2 and/or a rotated drive; run a backup; test a restore.

---

## 11. Open questions

- How many family members, and how tech-comfortable? (Affects whether the whole-site gate is worth it over per-album links.)
- Face recognition / scene search wanted, or is date+event browsing enough? (Immich excels at the former; PhotoPrism/Nextcloud are lighter.)
- Still own the Synology that left `@eaDir` behind? If so, Synology Photos is a low-effort fallback.
- Handling of `.psd`/`.xcf` edit files and `.zip`/`.rar` archives found in the tree.

RAW and video are settled by Section 3: ~1 RAW (embed in-file, no sidecars) and ~870 videos (Immich transcodes for mobile) — neither changes the plan.
