# AI Precise — Findings & Fixes, 2026-06-11

One day: root-caused the paige freezes, hardened the WiFi, deep-reviewed the app + firmware (~40 findings), fixed everything, and deployed it live to the bench — including a new 16-channel firmware on the Controllino.

---

## TL;DR

| Area | Found | Fixed |
|---|---|---|
| Paige freezes (Mon/Tue) | WiFi driver wedge, **not** GPU/camera load | WiFi hardened; box healthy |
| Firing path | Would brick minutes into every real field run | Heartbeat + ACK-truth + health gate, live on hardware |
| Silent failures | Camera/app could die while looking healthy | Self-heal + honest health, live-tested |
| Firmware | 6 channels, floodable watchdog, zero cooldown | 0.2.0: 16ch, hardened, flashed + verified |
| Performance | GPU stuck at half speed | 110→73ms inference (~6.3 fps/cam, was ~4.5) |
| Cameras | "Are they optimized?" | **Yes — settings were already right** (verified + now self-verifying) |

---

## 1. Why paige froze Monday/Tuesday (and not Wednesday)

Two different failure modes — both explained:

- **Mon + Tue afternoon:** the known DCE display-engine wedge (headless box churning a dead display). Already fixed by going headless — zero recurrence.
- **Tue 8:12pm (the hard freeze):** the forensic capture caught it — **paige's WiFi link died 4 minutes before the hang** ('AxisAg -5G' vanished mid-reconfigure), and the Realtek RTL8822CE vendor driver sat in a failed re-association loop until the whole SoC went silent. Pipeline, GPU, and cameras were healthy to the final 200ms. Kernel logged nothing; HW watchdog reset the box (it worked).
- **Wednesday was clean because both triggers were absent** — no WiFi drops *and* no detector running (Ethernet bench day). It proved nothing; the diagnosis above does.
- The old "Argus/GPU load" theory is retired: those nvargus errors and GPU thrash appear at identical rates on perfectly healthy days.

**Applied:** PSC-Guest declared the permanent uplink → powersave OFF (ping 263→78ms), infinite autoconnect retries, stale SSIDs disabled. Also fixed a wedged Tailscale tunnel (both ends) and documented the recovery path. RTC battery = Mark's solder iron.

## 2. App + firmware review — the killers we found

Three parallel deep-review agents over the detection path, firing path, and camera stack:

1. **Firing bricked itself on every real run** — firmware watchdog latches after 2.5s of wire silence; the Jetson never sent heartbeats. First weed-free 2.5s = no more fires until power-cycle, while the UI kept counting kills.
2. **Every nozzle self-suppressed after its first fire** — thermal "UNKNOWN" was counted as failure.
3. **Firmware drove 6 channels; configs declared 16** — connect couldn't even complete.
4. **"Silently dead while green"** — a capture-service restart froze that camera's feed forever; pipeline-launch failures fell back to UI-only with `/health` still 200; no liveness signal existed anywhere.
5. **Two firing engines shared one un-locked UDP socket** — crossed ACKs → timeouts → duplicate steam fires.
6. **The operator's ALL-OFF button was a stub** — it did nothing.
7. Three wire-protocol bugs (a *failed* thermal kill decoded as success; channel FAULT decoded as "unknown"; watchdog-trip state hidden).
8. Firmware accepted any 2-byte datagram from anything as "liveness" and an unbounded packet flood could starve the valve auto-off safety tick.

## 3. What we fixed (all of it — committed, tested, deployed)

**Firing path:** heartbeat thread (500ms) + liveness probe; I/O lock; ACK-truth policy (refused fires aren't counted and the weed stays killable; timeouts are counted-but-flagged); `is_healthy` gate (dead/E-stopped controller → fires suppressed, not faked); thermal-UNKNOWN fix; real ALL-OFF; **12 golden-vector contract tests derived from firmware bytes** (4 failed against the old code).

**Self-heal & honesty:** bridge re-attach on frozen feeds (both sides); capture service reopens a dead camera or exits for systemd; `/health` returns 503 with named reasons when anything requested isn't running (per-camera liveness in the body); app exits non-zero on launch failure; log firehose off by default (it had been vacuuming our own forensics); memory bounds on every unbounded set.

**Firmware 0.2.0:** 16 channels (pins 22-37); watchdog refreshes only on valid frames from the pinned Jetson IP; RX drain bounded (floods can't starve safety ticks); cooldown measured from valve *close* (was effectively zero); channel-count handshake so mismatches fail loudly by name.

**Deploy/config:** unit template now actually selects the real controller (it silently ran the mock before); retry-forever for the offline field unit; `jetson-clocks.service` (GPU was pinned at 612MHz of 1300 at 99% utilization — inference 110→73ms); PATCH validation so a typo'd class name can't silently kill detection; ISP read-back verification at camera start.

**Verified:** 594 tests passing, firmware compile-checked on the real toolchain, ruff clean.

## 4. Camera audit (direct answer: they're optimized correctly)

- Exposure is **bounded auto** (100µs–4ms ≈ max 3.6px motion blur at boom speed) — *not* locked; right for variable field light.
- Locked WB 4600K, sharpness 0, HD1200@30fps — all correct calls. Dropping to 15fps would add ±15–30mm of aim jitter against a 12mm margin for zero gain.
- BGR→RGB chain, letterbox math, clock-domain fix: all verified correct in code.
- New: the ISP profile now *verifies itself* at startup (read-back confirmed live: exposure bound took).

## 5. Deployed and proven on hardware

- 4 commits pushed; paige image rebuilt; orchestrated restart clean.
- **Firmware 0.2.0 flashed at the bench** — and the first flash immediately caught a real bug no test could: a CAN-era 8-byte length guard silently ate the new 9-byte status reply. The app's new fail-loud behavior pointed straight at it; fixed and reflashed within minutes.
- **App is live on the real Controllino**: 16-channel handshake passed, heartbeats confirmed feeding the watchdog (~2.1s of 2.5s remaining at every probe).
- **Self-heal live-tested**: restarted a camera capture service under the running app — the feed rode through (+101 frames in 15s, one clean re-attach). The old code would have silently killed that camera.
- Current state is fail-safe: fires double-blocked by the unwired E-stop + the pre-connect watchdog latch. No steam possible.

## 6. Still on the bench list (operator)

1. Tap reset once to clear the watchdog latch (heartbeats hold it forever after).
2. Dry LED test, channels 0–15, with `BENCH_BYPASS_ESTOP=1` (or D8→GND) — also verifies the relay pin map for new channels 6–15.
3. Replace the paige RTC battery (forensics timestamps).
4. Minor: gain-range read-back shows an odd unit scale — eyeball at the bench.
5. Deferred perf A/Bs when convenient: fp32 compile (~10–25%), preprocess-at-560 — *not* fp16/TRT (known confidence collapse).

---
*Commits: `dfbaeb9` (app hardening), `e898b1a` (firmware 0.2.0), `78c15af` (deploy), + `can_send` guard fix — `mboysen/ai-precise` @ `feat/zed-gmsl-camera`. Full detail: marvin decision log 2026-06-11 (×3 entries) + `sessions/2026-06-11.md`.*
