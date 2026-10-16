# Submission text (edit the bracketed parts to match what you actually achieved)

**Hackathon:** Embedded SF: CraftifAI Physical AI Hackathon — https://luma.com/5z8w42mq
**Track:** B — FirmGen (ESP32, ESP-IDF v5.5)
**Bonus:** Live Intelligence [keep only if you captured telemetry]

## Problem
Heat, humidity and failed sensors in server closets, labs, greenhouses and stockrooms go unnoticed until equipment or stock is damaged. Existing solutions need Wi-Fi, a cloud account or a subscription.

## Solution
AirSentinel is an ESP32 node that samples temperature/humidity at 1 Hz, classifies the environment as OK / WARN / ALARM / FAULT using debounce (3 samples) and hysteresis (1 °C / 3 %RH) so it never flaps, indicates state on traffic-light LEDs, sounds a buzzer in ALARM, supports acknowledgement by button or BLE, and streams an 8-byte packet over a custom BLE GATT service to a zero-install browser dashboard (Web Bluetooth).

## How FirmGen is load-bearing
- The firmware was generated from a written PRD (`prd/AirSentinel_PRD.md`) with 25+ numbered requirements, a defined BLE wire format and 10 board-level acceptance criteria. Requirement set reviewed and frozen in FirmGen before implementation.
- FirmGen built, flashed and monitored on a physical ESP32; UART output follows the PRD log format so acceptance can be read straight off the monitor.
- Traceability report links each REQ-* to code and unit tests. The state machine is a hardware-independent module so FirmGen's requirement-linked unit tests exercise it directly.
- Live Intelligence: telemetry enabled so the judges can see stack watermarks, heap and CPU load live while the board changes state.

## Evidence
FirmGen traceability report, UART captures AC-01…AC-0x, dashboard screenshots, videos: [links].

## Repo
[GitHub link — pushed via FirmGen Git view; includes prd/, dashboard/, docs/]

## Not done / future work
BLE bonding, Wi-Fi/MQTT uplink, OTA. [Add anything you skipped, e.g. watchdog test.]
