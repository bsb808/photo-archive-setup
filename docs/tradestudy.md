# Photo Archive & Distribution — Trade Study

A decision aid for managing a personal digital photo archive: a durable local master plus a mobile-accessible online layer for read-only family sharing, with strong guarantees against vendor lock-in.

*Prepared 2026-05-29. Updated 2026-05-29 after reviewing the actual archive (see Section 12).*

> **Read Section 12 first** if you want the bottom line. It records what the real `/data/Vault/Photos` archive contains and how it sharpens the recommendation. The body below (Sections 1–11) is the general study; Section 12 is the data-grounded amendment.

------------------------------------------------------------------------

## 1. Your requirements (as stated)

| \# | Requirement | Priority |
|------------------|-------------------------------|------------------------|
| R1 | Plain files on disk as the durable master; never trapped in a proprietary database | **Critical** |
| R2 | Local deep-backup copy stays in sync with the online sharing copy | **Critical** |
| R3 | Captions/notes embedded **in file metadata** (IPTC/EXIF), XMP sidecar as fallback | **High** |
| R4 | Mobile access (iOS + Android) to browse/search low-res versions, e.g. find a 1996 photo by date | **High** |
| R5 | Read-only sharing for friends & family | **High** |
| R6 | Self-host at home on an existing Linux-friendly desktop (with guidance) | **High** |
| R7 | Fixed cost strongly preferred; subscriptions disliked but acceptable if necessary | **Medium** |
| R8 | Date+event folder organization preserved; flexible on tooling adding a date timeline on top | **Medium** |
| R9 | Off-site disaster protection (advise) | **High** |

### Your context, distilled

- Archive size: **100 GB – 1 TB**. Comfortably within reach of a single added drive and cheap off-site backup.
- Ecosystem: **Linux-friendly**, comfortable adding an HDD/SSD to an existing desktop.
- Appetite: **eager to self-host with guidance**, wants an honest comparison against managed services, skeptical of lock-in.
- Folders: **flexible**, but originals must remain plain files you can always get back.
- Sharing: **read-only for family** (they browse/search, they don't upload).

------------------------------------------------------------------------

## 2. The central design tension

Almost every polished photo app (Google Photos, Immich, PhotoPrism, Synology Photos) is **database-driven**: it builds a fast index, thumbnails, face/scene search, and an editing layer in its own database. That is what makes browsing and search feel good. The risk is that the *meaning* you add (captions, tags, albums, ratings) gets trapped in that database.

Your lock-in concern is really about **where the truth lives**:

- **Truth in the files** (folders + embedded metadata + sidecars): any tool can read it, you can switch apps freely, and a database is just a disposable cache you can rebuild. This is the durable, future-proof model.
- **Truth in the app's database**: faster and prettier, but switching tools means an export/migration, and some metadata may not survive.

**The resolution this study recommends:** keep the *truth in the files*, and treat any gallery app as a **disposable read layer** pointed at those files. You get the nice mobile/search experience without betting your archive on any one product.

That single decision satisfies R1, R3, and most of R8 no matter which gallery app you pick, and it lets you swap the online layer later without data loss.

------------------------------------------------------------------------

## 3. Embedded metadata vs. sidecars (your R3, explained)

You chose "embed in-file when possible, XMP sidecar otherwise." That is the industry-standard, professionally durable choice. Here is why the "otherwise" matters:

- **JPEG, TIFF, PNG, HEIC**: captions/tags write cleanly into the file as IPTC/XMP/EXIF. No sidecar needed.
- **Camera RAW (CR2, NEF, ARW, DNG, etc.)**: writing into proprietary RAW containers can corrupt them, so the standard practice is a **`.xmp` sidecar** sitting next to the original (e.g. `IMG_1234.NEF` + `IMG_1234.NEF.xmp`). Adobe, digiKam, darktable, and exiftool all follow this convention, so sidecars are *not* a lock-in; they are a portable open standard.
- **Best practice**: write embedded **and** keep sidecars where applicable, so the data is readable both ways.

The tool that does this most faithfully on Linux is **digiKam** (see below). It is the metadata authority in the recommended architecture; the gallery apps only *read* what digiKam writes.

------------------------------------------------------------------------

## 4. Candidate solutions

### Group A — Self-hosted gallery/online layer (the "online" copy)

| Tool | What it is | Folder/plain-file fit | Metadata writeback | Mobile apps | Family read-only share | Lock-in risk |
|-----------|-----------|-----------|-----------|-----------|-----------|-----------|
| **Immich** | Modern Google-Photos-style server, very active | **External Library** mode indexes your folder tree **read-only, in place** (no copying) | Reads EXIF/IPTC well; **writeback to files is limited** — don't make it your caption authority | **Excellent native iOS + Android** | Shared albums + public links | Low if used in read-only external-library mode |
| **PhotoPrism** | Go-based indexer, originals-first design | **Read-only originals** mode; folder hierarchy is a first-class view; optional XMP sidecar writeback | Good; can write **XMP sidecars** | PWA (web app), no first-class native app | Albums + share links | Low |
| **Nextcloud + Memories** | File-sync platform + photo timeline app | Files stay plain; **true bidirectional desktop sync** (this is its superpower for R2) | Memories can write EXIF; files stay authoritative | Native iOS + Android (Nextcloud + Memories) | Shared folders/links, granular read-only | Low |
| **Photoview / Lychee / LibrePhotos** | Lightweight galleries | Photoview indexes folder tree read-only; simple | Minimal | PWA | Basic share links | Low, but fewer features |

### Group B — Managed services (for contrast)

| Service | Strengths | Why it conflicts with your priorities |
|----------------|----------------|----------------------------------------|
| **Google Photos** | Best-in-class search & mobile; effortless | No real folders, DB-owned metadata, recurring Google One fee, export-only "freedom" via Takeout. Fails R1/R8. |
| **Amazon Photos** | Free full-res for Prime members | Tied to Prime subscription, weak organization, lock-in. Fails R1/R7. |
| **iCloud Photos** | Great if Apple-centric | Hostile to Linux; DB-owned. Fails R6. |
| **SmugMug / Flickr** | Polished public sharing | Subscription, cloud-owned masters. Fails R1/R7. |
| **Synology Photos** (appliance) | Fixed hardware cost, free software, folder-respecting, good native apps, no monthly fee | Not free-form Linux; you'd buy a Synology NAS. **Closest managed-feel option that still respects folders and avoids subscriptions** — worth knowing about. |

------------------------------------------------------------------------

## 5. Weighted scoring

Criteria weighted by your stated priorities (5 = best fit).

| Criterion (weight) | Immich (ext. lib) | PhotoPrism | Nextcloud+Memories | Google Photos | Synology Photos |
|------------|------------|------------|------------|------------|------------|
| No lock-in / plain files (×5) | 4 | 5 | 5 | 1 | 4 |
| Local↔online sync (×5) | 3 | 3 | **5** | 2 | 4 |
| Embedded metadata authority (×4) | 2 | 4 | 3 | 1 | 3 |
| Mobile browse/search (×4) | **5** | 3 | 4 | 5 | 4 |
| Family read-only sharing (×4) | 4 | 4 | 4 | 4 | 4 |
| Self-host on existing Linux box (×4) | 5 | 5 | 5 | n/a (0) | 1 |
| Fixed cost (×3) | 5 | 5 | 5 | 2 | 4 (hw) |
| Search/timeline quality (×3) | **5** | 4 | 3 | 5 | 4 |
| **Weighted total** | **\~129** | **\~134** | **\~138** | \~78 | \~118 |

> Scores are directional, not precise. The top three are close; the right pick depends on which of your two "Critical" items dominates: **mobile/search polish (Immich)** vs. **bidirectional sync of the local copy (Nextcloud)** vs. **folder+sidecar fidelity (PhotoPrism)**.

------------------------------------------------------------------------

## 6. Recommended architecture

A **three-layer design** where the files are the truth and the gallery is disposable.

```         
                       ┌─────────────────────────────────────────┐
   LAYER 1: MASTER     │  Existing desktop + new dedicated drive   │
   (truth lives here)  │  /photos/YYYY-MM-DD_event/ ...            │
                       │  digiKam = catalog + captions →           │
                       │  embedded IPTC/EXIF + .xmp sidecars       │
                       └───────────────┬───────────────────────────┘
                                       │ (read-only index, no copy)
                       ┌───────────────▼───────────────────────────┐
   LAYER 2: ONLINE     │  Gallery server (Immich / PhotoPrism /     │
   (disposable read    │  Nextcloud+Memories) on same machine,      │
    layer)             │  Docker. Mobile apps + family read-only    │
                       │  share. Remote access via Tailscale or     │
                       │  Cloudflare Tunnel.                        │
                       └───────────────┬───────────────────────────┘
                                       │ (encrypted, dedup backup)
                       ┌───────────────▼───────────────────────────┐
   LAYER 3: OFF-SITE   │  restic/Borg → Backblaze B2 (or Wasabi)    │
   (disaster recovery) │  + annual rotated external drive off-site  │
                       └────────────────────────────────────────────┘
```

### Layer 1 — Local master (your deep backup + metadata authority)

- Keep your `YYYY-MM-DD_event/` folder tree. Add a dedicated internal drive (SSD for the working library, or a large HDD). For 100 GB–1 TB, a single 2–4 TB drive gives years of headroom.
- Use **digiKam** for cataloging and captions. Configure it to **write metadata to the files** and to create **XMP sidecars** for RAW. This is the one place you edit captions/tags, and it satisfies R3 permanently.
- This layer is independent of any server. If every gallery app vanished tomorrow, you still have organized, captioned, portable files.

### Layer 2 — Online sharing layer (mobile + family)

Pick one based on which "Critical" you weight highest:

- **If mobile experience and search matter most → Immich**, configured as a **read-only External Library** pointing at the Layer-1 folders. Immich never owns or moves your files; it just indexes them and serves great mobile apps. Caveat: do your caption editing in digiKam, not Immich, since Immich's writeback to files is limited.
- **If bidirectional sync of the local copy is the priority (R2 literally) → Nextcloud + Memories.** The Nextcloud desktop client keeps your local folder and the server genuinely in sync both ways, files stay plain, and Memories gives the timeline + native mobile apps. This is the most faithful match to "local copies synced with the online archive."
- **If folder-native + sidecar fidelity matters most → PhotoPrism** in read-only originals mode. Strong folder view, writes XMP sidecars, weaker mobile (PWA only).

> My single recommendation given everything: start with **Immich as an external read-only library**, with **digiKam** as the metadata authority. It gives the best phone experience for "find a 1996 photo by date" and the lowest day-to-day friction, while keeping the files untouched. If, after living with it, the literal local↔online *sync* in R2 feels more important than mobile polish, switch the Layer-2 app to Nextcloud+Memories. Because the truth is in the files, that switch costs you nothing but re-indexing time.

### Layer 3 — Off-site backup (your R9, the advice you asked for)

A home server is one fire, theft, or ransomware event from total loss. Apply **3-2-1**: 3 copies, 2 media types, 1 off-site.

- **Off-site cloud**: `restic` or `Borg` to **Backblaze B2** (or Wasabi). Encrypted client-side, deduplicated, incremental. For \~500 GB, B2 runs roughly **\$3/month** (about \$6/TB/mo). This is variable but tiny and scales with your archive.
- **Zero-recurring alternative**: a rotated **external drive** stored at a relative's house, refreshed a few times a year. Fully fixed cost, at the price of manual effort and staleness between rotations.
- **Best of both**: do the cheap cloud backup for recency *and* an annual cold drive for the "everything burned down" case.
- Whatever you choose, **test a restore** once. An untested backup is a hope, not a backup.

------------------------------------------------------------------------

## 7. Remote access (how family reaches it)

You do not need to expose your home server to the open internet.

- **Tailscale** (free tier, fixed cost \$0): a private mesh VPN. You and family install the app; everyone reaches the server as if on the same LAN. Simplest and safest. Good when "family" is a handful of trusted people.
- **Cloudflare Tunnel** (free): publishes a clean `https://photos.yourdomain` link with no open ports, suitable for public read-only album links. Slightly more setup; better when you want to hand out a plain web link.
- Immich/PhotoPrism/Nextcloud all support read-only shared albums and public share links once reachable.

------------------------------------------------------------------------

## 8. Cost summary

| Item | One-time | Recurring |
|------------------|--------------------------|----------------------------|
| Dedicated 2–4 TB drive | \$60–\$120 | — |
| Gallery software (Immich/PhotoPrism/Nextcloud) | \$0 (open source) | \$0 |
| Tailscale (personal) | \$0 | \$0 |
| Off-site cloud backup (B2/Wasabi, \~500 GB) | \$0 | \~\$3/mo (or \$0 with rotated drive) |
| Domain name (optional, for public links) | — | \~\$12/yr |

Net: a **one-time hardware spend plus near-zero or zero recurring**, which matches your fixed-cost preference. The only optional subscription is the off-site cloud backup, and even that is replaceable with rotated drives.

------------------------------------------------------------------------

## 9. Honest tradeoffs: self-host vs. managed

| Dimension | Self-host (recommended) | Managed (e.g. Google Photos) |
|------------------|-------------------------|-----------------------------|
| Control / lock-in | You own everything; switch tools freely | Vendor owns the experience; export-only escape |
| Cost model | One-time hardware, \~\$0/mo | Recurring subscription, grows with storage |
| Mobile/search polish | Very good (Immich) but you maintain it | Best-in-class, zero maintenance |
| Maintenance burden | Updates, backups, occasional troubleshooting | None |
| Uptime/availability | Depends on your home power/network | Always on |
| Privacy | Photos stay on your hardware | Scanned/analyzed by the provider |
| Failure mode | You must have off-site backup or risk loss | Provider handles redundancy |

The honest cost of self-hosting is **your time and attention**: occasional Docker updates, watching that backups run, the rare "why won't it start" evening. In exchange you get permanent control and no subscription. Given your stated values, the trade favors self-hosting, with the explicit caveat that **Layer 3 off-site backup is non-negotiable** because you are now your own redundancy.

------------------------------------------------------------------------

## 10. Recommended next steps

1.  **Add the drive** to the desktop and lay down the `YYYY-MM-DD_event/` structure (migrate existing folders as-is).
2.  **Install digiKam**, point it at the library, and configure metadata writeback (embedded + XMP sidecars). Caption a test event to confirm the data lands in the files.
3.  **Stand up Immich via Docker** as a read-only External Library pointing at the photo folder. Install the mobile app and test the "find a 1996 photo by date" flow.
4.  **Set up Tailscale** and add one family member's phone to validate read-only sharing.
5.  **Configure restic → Backblaze B2** (or set up the rotated-drive routine), run a backup, then **test a restore**.
6.  Live with it for a few weeks. If literal local↔online sync (R2) outweighs Immich's mobile polish, swap Layer 2 to **Nextcloud + Memories** at no data cost.

------------------------------------------------------------------------

## 11. Open questions to revisit later

- ~~Do you shoot **RAW**?~~ **Answered (§12): essentially none — 1 RAW file in the whole archive.** Embed metadata in-file; skip the sidecar workflow.
- ~~Are there **videos** mixed in?~~ **Answered (§12): yes but modest — \~870 files.** Doesn't change the size tier. Immich transcodes for mobile.
- How many **family members**, and are any non-technical enough that Tailscale install is a barrier? (May tip toward Cloudflare public links.)
- Do you want **face recognition / scene search**, or is date+event browsing enough? (Immich excels at the former; if you don't need it, PhotoPrism or Nextcloud are lighter.)
- Do you still own the **Synology** that left `@eaDir` artifacts behind? If so, Synology Photos is a low-effort fallback (§12).

------------------------------------------------------------------------

## 12. Archive review findings & revised guidance (2026-05-29)

I scanned the three sample trees you pointed at. The data confirms the architecture in Sections 6–7 and simplifies two of your requirements.

### 12.1 What's actually in the archive

| Tree | Size | Notes |
|-----------------------|-----------------------|--------------------------|
| `PhotosDated/` | **75 GB**, 365 event folders | Your `YYYYMMDD_event` convention, back to `19980713_montreal_trip`. Clean and consistent. |
| `PhotosScan/` | **1.2 GB**, 23 folders | Scanned family photos named by **theme/decade** (`1950s_withers`, `1972_christmas`, `1985_vail_beavercreek`). |
| `BrianPhoneGalaxyS20_June2023/` | **13 GB**, 2,557 loose files | Phone dump, `YYYYMMDD_HHMMSS.jpg` filenames. Unorganized, as you noted. |
| **Total** | **\~89 GB, \~26,150 files** | Bottom of the 100 GB–1 TB tier. |

**File mix:** \~24,000 JPEG (92%), 438 HEIC, \~870 video (mp4/mov/avi/wmv/3gp), and **exactly one RAW file**.

### 12.2 What this changes

1.  **Cost drops to a rounding error.** At \~89 GB, off-site cloud backup (Backblaze B2/Wasabi) is roughly **\$0.50/month**, and the whole archive fits on the cheapest drive you can buy. The fixed-cost preference (R7) is trivially satisfied; even a rotated external drive alone is plenty.

2.  **R3 gets much simpler — embed in-file, forget sidecars.** With one RAW file in 26,150, the "XMP sidecar fallback" is a non-issue. Captions, tags, and dates write cleanly into your JPEG/HEIC files. The sidecar machinery in Section 3 effectively doesn't apply to your data.

3.  **New required step: date-normalize the scans.** This is the one finding that adds work. Your scanned photos carry **no real capture date** in EXIF (it's blank or the scan date), and their folders are named by decade/year, not `YYYYMMDD`. A timeline view would pile all of them at "today" — so your headline use case, "find a 1972 photo by date," would fail for exactly the oldest, most precious photos. **Fix:** a one-time batch that writes an approximate `DateTimeOriginal` (e.g. `1972-01-01`) into each scanned file, derived from its folder name. digiKam does this in its GUI, or a short `exiftool` script can do it folder-by-folder. After that, scans sort correctly on the timeline and date-search works on mobile.

4.  **A cleanup pass belongs before indexing.** The trees carry artifacts that would clutter any gallery:

    - **344 `@eaDir` directories** — Synology NAS thumbnail caches. (This archive previously lived on a DiskStation, which is also why Synology Photos is a reasonable fallback if you still have the unit.)
    - **178** `.db` / `.ini` / `Thumbs.db` OS-and-NAS cruft.
    - **349** `.html` / `.htm` / `.css` files from old exported web galleries.
    - **15** stray non-photo files (`.bag`, `.mat`, `.m`, `.dcm`) that wandered in from robotics/MATLAB/medical work — move these out of the photo tree. These are safe to exclude at index time (Immich/PhotoPrism let you ignore patterns) and better to delete from the master once reviewed.

5.  **Phone dump needs date-from-filename or EXIF triage.** The 2,557 loose `YYYYMMDD_HHMMSS.jpg` files do have usable dates (in the filename and likely EXIF), so they'll timeline correctly even before you organize them into event folders. You can let them sit loose and clean up gradually without breaking date-browsing.

### 12.3 Revised recommendation (unchanged in shape, sharper in detail)

The Section 6 architecture stands and is reinforced. Concrete sequence for your data:

1.  **Install `exiftool` and digiKam** (both Linux-native; digiKam uses exiftool under the hood). `exiftool` is currently **not installed** on the box.
2.  **Cleanup pass:** remove/ignore `@eaDir`, `Thumbs.db`/`.db`/`.ini`, the old `.html`/`.css` galleries, and relocate the stray `.bag`/`.mat`/`.m`/`.dcm` files.
3.  **Date-normalize `PhotosScan/`:** batch-write approximate `DateTimeOriginal` from each folder name. (One-time, scriptable.)
4.  **Caption in digiKam**, writing embedded IPTC/EXIF into the JPEGs. This is your durable, app-agnostic layer.
5.  **Point Immich at the tree as a read-only External Library**, add the mobile app, validate the "find a 1972 / 1996 photo by date" flow end to end.
6.  **Off-site backup:** `restic`/`Borg` to B2 (\~\$0.50/mo) and/or a rotated external drive. Test a restore.

No change to the tool choice: **digiKam (metadata authority) + Immich (read-only mobile/family layer) + off-site backup**, with Nextcloud+Memories as the swap-in if literal local↔online sync matters more than mobile polish.