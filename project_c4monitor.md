---
name: project-c4monitor
description: "Amesbury home dashboard (c4monitor) — what it is, how it works, open issues"
metadata: 
  node_type: memory
  type: project
  originSessionId: ca389924-dc34-49d0-aadf-4196a83ac71f
---

c4monitor is Rob's self-hosted Control4 home automation dashboard. Node.js + vanilla HTML, no cloud dependency after setup.

**Why:** Monitor and control the Amesbury house (security, pool, HVAC, energy, Tesla, network, audio) from a local dashboard.

**How to apply:** Reference when working on smart home integrations or dashboard features.

## Stack
- Node.js, SQLite (better-sqlite3), vanilla HTML/JS, @anthropic-ai/sdk
- Location: `/opt/amesbury_monitor/`
- Service: `sudo systemctl restart amesbury_monitor` (system-level, port 8766)
- Service user: `amesbury_monitor` (no login shell)
- Certs: `/opt/amesbury_monitor/certs/`
- Config: `config.json` (IPs, device IDs), `.env` (secrets)

## What it monitors
- Security zones (Control4), 2 garage doors, 3 thermostats
- Pool/spa (Pentair IntelliCenter at 192.168.20.204)
- Solar (Enphase IQ Gateway local + cloud fallback)
- 2 Teslas via Tessie API (Ghost = Model S Plaid, DISCO = Model Y)
- UniFi network, Uptime Kuma, Homebridge plugins
- 14 audio rooms via Control4

## External access (as of 2026-05-24)
- Public URL: `https://amesburymonitor.r23.ai` via Cloudflare Tunnel (existing Docker cloudflared)
- Auth: Cloudflare Access (Zero Trust) — One-time PIN to email, allowed emails: Rob + wife
- Tunnel routes to `http://192.168.5.64:8766`
- Two fixes needed to make it work:
  1. `isAllowedIp()` in server.js — was checking `x-forwarded-for` first (Cloudflare sets this to public IP), fixed to trust socket IP first (cloudflared connects from 127.0.0.1)
  2. `const API` in dashboard.html — was hardcoded to `http://${hostname}:8766`, fixed to use relative URLs when not on port 8766 so API calls route through the tunnel
- Local access (`http://192.168.5.64:8766`) still works with no auth

## Navigation (as of 2026-05-24)
- Full MD3 rebuild (sections 1–6) complete — Material Design 3 dark/light tokens, elevation, state layers
- Navigation Drawer replaces tab bar (tab bar had 13 sections, too many for MD3 tabs spec)
- Hamburger icon (≡) in topbar → slides in from left, scrim overlay, Escape/scrim-tap to close
- 13 nav items with emoji icons, 56px touch targets, active state highlighted in primary color

## AI Chat
- Endpoint: `/api/chat` — direct Anthropic SDK call (no OpenClaw dependency)
- Model: `claude-haiku-4-5-20251001` via OAuth token (`ANTHROPIC_API_KEY` in .env)
- Context: live `state.json` snapshot + `knowledge.md` file (no Neo4j/Ollama)
- Persistent history: SQLite `chat_messages` table, keyed by `sessionId` (UUID in browser localStorage)
- History endpoint: `/api/chat/history?sessionId=...`
- Tool use: control_light, control_garage, control_thermostat, control_pool_circuit, query_energy_history

## Open issues (from wishlist.md / HITLIST.md)
- HVAC mode buttons broken (tap Off/Cool/Heat/Auto does nothing — all thermostats)
- Cars page needs Tessie API token confirmed
- Lock control parked (Level lock, no API support yet)
- Guest web app at `/guest` not yet built
- ntfy notifications not yet wired (garage open alert, alarm state, pool temp, Tesla plugged in, daily summary)
- Status/troubleshooting pages not built

## Ghost zone fix (implemented 2026-05-13)
Root cause: API only exposes OPEN_ZONE_COUNT + LAST_ZONE_FAULTED. When a door closes simultaneously with a motion sensor firing, the count never drops between polls, so the reconciler never triggers a close. Added `suspectCloses` mechanism: when `lastFaulted` transitions from a tracked door zone to a motion sensor, the prior door zone is queued as a suspect. After 2 poll cycles (~60s) with count still flat, the ghost is automatically closed. Cancelled if the zone re-faults or count drops naturally.
