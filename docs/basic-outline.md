# AC Controller — Project Context & Decisions

## Problem Statement

Physical AC remotes have several pain points:
- Run out of battery
- Poor displays, not enough information
- No remote control outside physical proximity

**Goal:** Replace the physical remote with a internet-connected IR blaster controllable from a phone via a web UI.

---

## Scope & Constraints

- Web-based UI only (no app store involvement)
- Minimum viable hardware — low cost, low compute
- Solve for self first, then opensource

---

## Hardware Decisions

| Component | Decision | Reason |
|---|---|---|
| Main compute | **Raspberry Pi Zero W** | Smallest official Pi, full Linux, built-in WiFi, SSH-able, Python-friendly |
| IR transmitter | IR LED + transistor circuit | ~$2, blasts IR signals to AC unit |
| IR receiver | IR receiver module (e.g. TSOP38238) | Only needed during setup/learning phase |
| Power | USB power adapter | Always-on, simple |

**Total hardware cost estimate: ~$20**

Pi Zero W dimensions: 65mm × 30mm (stick of gum sized — can be mounted behind/near AC unit)

### Why not ESP32?
ESP32 was considered (cheaper ~$5, native IRremoteESP8266 support) but rejected because:
- No Linux — C++ Arduino-style only, not Python
- Limited web server capability
- Requires physical reflashing to update code
- Harder to extend (Tailscale, auth, logging etc.)

---

## Target AC Unit

- **Model:** Mitsubishi Electric MSY-GE10VA
- **IR Protocol:** `MITSUBISHI_AC` (confirmed supported)
- **Library:** `IRremoteESP8266` — has full high-level protocol implementation for this family

### Available controls via library:
```
setPower(ON/OFF)
setTemp(16–31°C)
setMode(COOL / HEAT / DRY / FAN / AUTO)
setFanSpeed(AUTO / QUIET / LOW / MED / HIGH)
setVane(AUTO / ...)   # airflow direction
send()
```

No manual IR code recording needed — protocol is fully implemented.

---

## IR Code Source

No need to manually record IR codes. Resources confirmed:
- **IRremoteESP8266** (`github.com/crankyoldgit/IRremoteESP8266`) — gold standard, has `MITSUBISHI_AC` protocol with full parameter control
- MSY-GE series falls under the same `MITSUBISHI_AC` protocol family as MSZ-GE/SF/ZW series (all confirmed in `SupportedProtocols.md`)

---

## Phased Delivery Plan

### Phase 1 — Local Network Control
- Pi runs a **FastAPI** web server
- Phone hits `http://raspberrypi.local` on same WiFi
- Web UI sends button press → Pi sends IR signal → AC responds
- IR codes sourced from IRremoteESP8266 library (no manual recording)
- **Estimated effort:** 1–2 weekends

**Stack:**
- OS: Raspberry Pi OS Lite
- Language: Python
- IR library: `pigpio` or `python-irda` (wrapping IRremoteESP8266 logic)
- Web server: FastAPI
- Config: `config.yaml` storing AC model + IR parameters

### Phase 2 — Remote Access (Outside Home Network)
- **Preferred approach: Tailscale**
  - Install on Pi + phone
  - Private mesh VPN — Pi accessible from anywhere
  - Zero infra cost, no open ports, ~20 min setup
- Alternative: Cloudflare Tunnel (free, no port forwarding)
- **No VPS needed for personal use**

### Phase 3 — Open Source / Self-Hostable
- `setup.sh` script to automate Pi setup
- `config.yaml` for AC brand + IR codes (or interactive learning mode)
- GitHub repo with clean README: flash Pi OS → run script → done
- Optional: hosted demo UI on Cloudflare Pages or a ~$5/mo VPS
- Budget: ~$10 one-time for any hosting/traffic costs

---

## Open Questions / Future Considerations

- [ ] Confirm IR line-of-sight placement near AC unit
- [ ] Decide on Python IR library for Pi (pigpio vs alternatives)
- [ ] Web UI design (minimal — power, temp +/-, mode, fan speed)
- [ ] Auth for the web UI (at minimum before Phase 2 goes live)
- [ ] OTA code updates via SSH (already solved by Pi choice)
- [ ] Profiling whether Pi Zero W is right size or if Pi Zero 2 W is worth the marginal cost bump
