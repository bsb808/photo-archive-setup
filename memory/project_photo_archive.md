---
name: project_photo_archive
description: "photo-archive-setup repo — purpose, guiding principle, and current phase of the self-hosted photo archive design"
metadata: 
  node_type: memory
  type: project
  originSessionId: 9e92d44f-0ea5-4bd2-9fc3-b538e36fe2a3
---

`photo-archive-setup` is a planning/design repo (not application code) for a self-hosted personal photo archive, designed collaboratively with the user.

Guiding principle: **the files are the truth** — a `YYYYMMDD_event` folder tree of plain images with captions/dates embedded in IPTC/EXIF. Any gallery app is a disposable read layer; no proprietary DB lock-in.

Target architecture (3 layers): local master = folder tree + digiKam (embedded metadata, source of truth); online = Immich as read-only external library; off-site = restic/Borg → Backblaze B2 and/or rotated drive. Remote access via Tailscale or Cloudflare Tunnel.

As of 2026-06-04 the project is in **Phase 1 (lock decisions D1–D7), nothing destructive yet**. Phase 2 (transition) is gated on those decisions + a verified backup. Live state lives in [docs/plan.md](../../../../WorkingCopies/photo-archive-setup/docs/plan.md) (checklist, decision log) and [docs/tradestudy.md](../../../../WorkingCopies/photo-archive-setup/docs/tradestudy.md) (comparison + Section 12 archive review) — read those for current detail rather than relying on this memory.

Archive reviewed 2026-05-29: ~89 GB, ~26,150 files across `PhotosDated/`, `PhotosScan/`, and an unsorted phone dump; ~92% JPEG. Writing-style guidance in [[feedback_avoid_ai_tells]] applies to the docs here.
