# Runbook — do these in order (you must do the account/install/flash steps yourself)

Deadline: **today 16 Oct 2026, 11:00 PM IST**. Budget ~3–4 h.

## 0. Prep (15 min)
1. Sign up at https://developer.craftifai.com/dashboard?tab=downloads and download **FirmGen v0.4.1** (Apple Silicon, macOS 26+, ~8 GB free). Install prerequisites it lists; confirm it uses **ESP-IDF v5.5**.
2. Join the Discord (#community-support) — ask there if the form link has been posted.
3. Wire the ESP32 per PRD section 3. Different pins or a BME280 instead of DHT22? **Edit PRD section 3 now**, before upload.

## 1. Requirements (30 min)
1. Create a FirmGen project, target ESP32, upload `prd/AirSentinel_PRD.pdf` (or `.md`).
2. Review generated requirements. Check every REQ-* from the PRD appears (Explicit tag). Fix wrong Inferred ones. **Freeze** the set — graders look for this.
3. Pick **Auto Mode** if short on time; use Category-wise review only if it is fast.

## 2. Build, flash, monitor (60–90 min)
1. Let FirmGen implement, build, flash. Watch UART/Monitor for `[SAMPLE]` lines.
2. Walk through AC-01…AC-06 on the bench (hair-dryer for heat, unplug DHT22 data wire for FAULT). Record video + UART capture.
3. If the UART format drifts from REQ-LOG-01/02, ask FirmGen to fix it — don't hand-edit outside FirmGen.

## 3. BLE + dashboard (30 min)
1. `cd dashboard && python3 -m http.server 8000`, Chrome -> localhost:8000, Connect via BLE (Mac Bluetooth on).
2. AC-07 / AC-08: screenshot decoded values, click Acknowledge, see buzzer stop + `[ACK] source=BLE` in UART.
3. If the packet decodes wrongly, check byte order/layout against PRD REQ-BLE-03 and re-run FirmGen with a fix request.

## 4. Live Intelligence (20 min)
Open Live Intelligence panel -> **Enable Telemetry** -> rebuild/flash -> screenshot stack watermarks, heap, CPU (AC-09). Keep it open during the demo.

## 5. Evidence + submit (30 min)
1. Export the **traceability report** from FirmGen. Push the project via FirmGen's **Git view** to a GitHub repo; copy `prd/`, `dashboard/`, `docs/` into the same repo.
2. Collect items in `EVIDENCE_CHECKLIST.md`.
3. Paste `docs/SUBMISSION.md` into the FirmGen submission form (https://forms.gle/bnocK9CNLndiZLwf8). The form link may change — confirm on Discord/Luma. Submit before the deadline; don't wait for the last minutes.

## Fallbacks
- No DHT22 → BME280 or ESP32 internal temp sensor; update PRD before upload.
- BLE flaky on Mac → test with nRF Connect on a phone to isolate firmware vs. browser.
- Time short → drop AC-10 (watchdog) and REL-02, keep AC-01…AC-09.
