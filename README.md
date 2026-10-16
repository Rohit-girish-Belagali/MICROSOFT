# AirSentinel — ESP32 BLE Environmental Safety Node

Entry for **Embedded SF: CraftifAI Physical AI Hackathon** (HackwithBangalore 4.0), **Track B — FirmGen**, bonus: **Live Intelligence**.
Event: https://luma.com/5z8w42mq

## What it is
An ESP32 node that senses temperature/humidity every second, classifies the room as OK / WARN / ALARM / FAULT with debounce and hysteresis, shows it on traffic-light LEDs, sounds a buzzer in ALARM (button or BLE acknowledge), and streams an 8-byte packet over BLE to a browser dashboard.

## Repo layout
| Path | What |
|---|---|
| `prd/AirSentinel_PRD.md` (+ `.pdf`) | The PRD uploaded to FirmGen. Requirement IDs, BLE packet format, 10 board acceptance criteria. |
| `dashboard/index.html` | Web Bluetooth dashboard (decodes the PRD packet, sends Ack, charts, event log). Has a clearly-labelled simulated Demo mode. |
| `docs/RUNBOOK.md` | Step-by-step: what to click in FirmGen, in order, with time budget. |
| `docs/SUBMISSION.md` | Paste-ready project description for the form. |
| `docs/EVIDENCE_CHECKLIST.md` | Screenshots/captures to collect. |
| `firmware/` | **Created by FirmGen** (push via its Git view). Not hand-written. |

## Run the dashboard
```bash
cd dashboard && python3 -m http.server 8000
# open http://localhost:8000 in Chrome
```
Demo mode needs no hardware. Real mode: Connect via BLE -> pick `AirSentinel-XXXX`.

## Honest status
The PRD and dashboard are done. The firmware, traceability report and board evidence come from running FirmGen on the ESP32; nothing in `firmware/` should be claimed until that is done.
