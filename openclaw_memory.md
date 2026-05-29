# MEMORY.md — Long-term Memory

## User — Rob
- **Email:** grapesmc@spiralplanet.com
- **Tone:** Direct, casual, no corporate filter needed
- **Style:** Runs his own homelab (Proxmox, Control4, UniFi), comfortable with infra, Node.js, networking
- **Working style:** No-bullshit answers — verify before recommending, not guessing
- **Projects:** Amesbury house (smart home), OpenClaw homelab agent, digital twin project with Courtney

## How to Answer
- Tell him what information you actually need before answering
- Make zero assumptions — ask or look it up
- Verify links/products before recommending; if you can't verify, say so
- Only give recommendations after you have the full picture
- Search session/memory history before making suggestions
- **Why:** Rob gave this explicitly. He wants correct answers, not confident-sounding guesses.

## Hard Rules — Lessons Learned the Hard Way
- **NEVER test a state-modifying or destructive API endpoint against live data.**
  Use a fake/nonexistent value to verify the endpoint works, then clean up.
  This includes: alarm zone correction, garage toggles, thermostat commands, anything that touches real state.
  The garage door and Family audio were already accidents (2026-03-14).
- **Before calling any write/delete/modify endpoint as a "smoke test", ask: is there live data this could affect?**
  If yes → use a dummy value, or ask first.

## This Instance — OpenClaw Setup
- Running on **pve1** (Proxmox VE), VM: OpenClaw-001, IP: `192.168.5.64`
- Config: `/home/grapesmc/.openclaw/openclaw.json`
- Version: 2026.5.22
- **Primary model: `anthropic/claude-sonnet-4-6`** (changed 2026-05-26, was Haiku)
- Fallback model: `anthropic/claude-haiku-4-5-20251001`
- Auth: `anthropic:claude-personal` (OAuth token, sk-ant-oat01-... — same token as Claude Code)
- Gateway: local mode, port 18789
- Workspace: `/home/grapesmc/.openclaw/workspace/`
- Web search provider: `gemini`

## Amesbury Monitor (c4monitor)
- Self-hosted Control4 home automation dashboard — Node.js + vanilla HTML, no cloud dependency
- **Production location:** `/opt/amesbury_monitor/` (migrated 2026-05-25, was in workspace)
- **Service:** `sudo systemctl restart amesbury_monitor` (system-level, port 8766)
- **Service user:** `amesbury_monitor` (no login shell — need `sudo chown grapesmc` to edit files, restore after)
- **Public URL:** `https://amesburymonitor.r23.ai` via Cloudflare Tunnel → Cloudflare Access (OTP to email)
- **Repo:** `github.com/grapesmc/amesbury-monitor-tools`
- **Config:** `config.json` (IPs, device IDs), `.env` (secrets, incl. `ANTHROPIC_AUTH_TOKEN`)

### What it monitors
- Security zones (Control4), 2 garage doors, 3 thermostats
- Pool/spa (Pentair IntelliCenter at 192.168.20.204)
- Solar (Enphase IQ Gateway local + cloud fallback)
- 2 Teslas via Tessie API (Ghost = Model S Plaid, DISCO = Model Y)
- UniFi network, Uptime Kuma, Homebridge plugins
- 14 audio rooms via Control4

### AI Chat — Eamesbury
- Chat endpoint: `/api/chat` — **direct Anthropic SDK** (no OpenClaw dependency)
- Model: `claude-haiku-4-5-20251001` via `ANTHROPIC_AUTH_TOKEN` (OAuth token)
- Persona: Eamesbury — personality of Charles Eames, speaks first, asks if Rob, Julie, or guest
- Persistent per-user memory: Rob and Julie have cross-session history; guests purge after 24h
- Tools: control_light, control_garage, control_thermostat, control_pool_circuit, query_energy_history, query_sensor_history
- SQLite tables: `chat_messages`, `chat_sessions`, `user_memory` in `data/events.db`

### Ghost Zone Fix (2026-05-13)
- Root cause: API only exposes `OPEN_ZONE_COUNT` + `LAST_ZONE_FAULTED`; simultaneous door close + motion sensor means count never drops between polls
- Fix: `suspectCloses` mechanism queues prior door zone when `lastFaulted` transitions to a motion sensor; closes after 2 poll cycles (~60s) if count stays flat

### Open Issues
- HVAC mode buttons broken (Off/Cool/Heat/Auto taps do nothing — all thermostats)
- ntfy notifications not wired (garage open, alarm state, pool temp, Tesla plugged in, daily summary)
- Guest web app at `/guest` not built
- Lock control parked (Level lock, no API)
- Network/Services/Troubleshooting pages incomplete

## Network
- Gateway: UDMP at `10.0.255.1`
- DNS: `10.0.255.100`, `10.0.255.253`
- debiandocker: `10.0.255.243` — Docker host: Flame (5005), Uptime Kuma (3001), Portainer (9000)
- homebridge: `10.0.255.229`
- Servers: hal9000 (.230), lexx3000 (.235), sydney (.238), case (.253), plex (.251), pi4-1 (.100)
- IPMI: case (.162), hal9000 (.252), kipp (.249), plex (.182), tars (.137)
- Switches: US-16XG (.23), PM48PoE (.40), PM24PoE (.22), USW-Flex x4 (usw-flex-living added)
- 6x U6-Pro APs throughout the house
- **Topology diagram is known to be wrong** — needs correction

## Services / Credentials (session only — do not persist passwords)
- Homebridge: admin / (used in session, not stored)
- Uptime Kuma: grapesmc / (used in session, not stored)
- Flame API accessible unauthenticated at debiandocker:5005/api

## Smart Home — Known Issues & Fixes

### Sony TVs — WiFi Recovery After UDMP Restart
- Some Sony TVs lose WiFi if Dream Machine restarts — C4 remote useless, shows offline in Uptime Kuma
- **Fix:** Power on with Sony remote (not C4) → reconnects to WiFi → power off → C4 works again
- **If that fails:** Hard power cycle the TV at the unit
- They sometimes reconnect on their own after 1h+ — don't wait if you need the TV now

### Pool Speakers — Right Speaker Not Working
1. Disconnect/reconnect RCAs on Amp-2 Selection 1
2. Disconnect/reconnect speaker connectors on Amp-2 Selection 1
3. Disconnect/reconnect Audio 9 on the Audio Matrix Switch

### Alarm System
- Panel: **Honeywell VISTA-20P** via **4232CBM** (RS-232) → C4
- C4 driver: `Honeywell 4232CBM (Residential).c4z`
- Repeater on Master Bath Skylight + Master Bath Window — resolved ~2026-03-24

## Firewall (2026-03-18 cleanup)
- Deleted: `Allow CORE access to IoT` (confirmed duplicate of `Core access to IoT`)
- Key lesson: session notes were wrong; always verify live data before destructive action
- `Evil_IoT to External` is fully open (ANY→ANY) — Peloton lives there; left alone pending review
- UniFi API: `X-API-Key` from c4monitor `.env`, endpoint: `https://10.0.255.1/proxy/network/v2/api/site/default/firewall-policies`

## Digital Twin Project (architecture finalized 2026-03-20, VM not yet built)
- Rob & Courtney — long-term goal: agent acting fully on behalf of a person
- Dedicated VM (OpenClaw-02): 12 vCPU / 24GB RAM / 200GB / RTX 4060 passthrough, IP: `192.168.5.176`
- Stack: Neo4j, LlamaIndex, Ollama (nomic-embed-text + llama3.1:8b-instruct-q4), Docker
- Setup doc: `jl-twin-project/OVERVIEW.md` in this workspace
- Status: VM not built yet
