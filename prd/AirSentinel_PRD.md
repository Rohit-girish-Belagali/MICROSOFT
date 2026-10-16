# AirSentinel — ESP32 BLE Environmental Safety Node
## Product Requirements Document v1.0

**Hackathon:** Embedded SF: CraftifAI Physical AI Hackathon (HackwithBangalore 4.0) — Track B (FirmGen)
**Target:** ESP32 dev board (ESP-IDF v5.5), BLE + GPIO + UART
**Bonus targeted:** Live Intelligence (telemetry / RTOS monitor)

---

## 1. Problem
Heat, humidity and sensor failures in server closets, labs, greenhouses and storage rooms go unnoticed until equipment or goods are damaged. People need a cheap always-on node that measures the environment, shows its status locally in a way that cannot be misread, alerts audibly, and streams readings to a phone or laptop over BLE with no infrastructure.

## 2. Product summary
AirSentinel samples temperature and humidity once per second, classifies the room as **OK / WARN / ALARM / FAULT** with debounce and hysteresis (so it never flaps), drives a traffic-light LED set and a buzzer, lets the user acknowledge an alarm with a button, and publishes readings and state over a BLE GATT service. All behaviour is logged over UART in a fixed format.

## 3. Hardware (confirm pins before freezing)
| Item | Default |
|---|---|
| Board | ESP32 DevKit (ESP32-WROOM-32) |
| Temp/humidity sensor | DHT22 (AM2302), data on GPIO4 |
| Green / Yellow / Red LEDs | GPIO25 / GPIO26 / GPIO27 (active high, with resistors) |
| Buzzer (active) | GPIO14 (active high) |
| Acknowledge button | GPIO32, to GND, internal pull-up |
| Status LED | On-board LED GPIO2 (BLE state) |
| UART console | UART0, 115200 8N1 |

If no DHT22 is available, the sensor driver may be swapped for BME280 on I2C (SDA GPIO21, SCL GPIO22) without changing any other requirement.

## 4. Functional requirements

### 4.1 Sensing
- **REQ-SENS-01** The firmware SHALL sample temperature and relative humidity once every 1000 ms (±50 ms).
- **REQ-SENS-02** Temperature SHALL be reported in 0.01 °C resolution and humidity in 0.01 %RH resolution.
- **REQ-SENS-03** A sample SHALL be marked invalid if the sensor read fails or returns a value outside −40…80 °C or 0…100 %RH.
- **REQ-SENS-04** After 3 consecutive invalid samples the system SHALL enter the FAULT state; one valid sample SHALL leave FAULT (re-evaluating normally).

### 4.2 State machine
States: `OK`, `WARN`, `ALARM`, `FAULT`.
- **REQ-STATE-01** WARN is entered when temperature ≥ 30.0 °C OR humidity ≥ 70 %RH.
- **REQ-STATE-02** ALARM is entered when temperature ≥ 35.0 °C.
- **REQ-STATE-03** *Debounce:* a transition to a more severe state SHALL occur only after 3 consecutive valid samples support it (≈3 s).
- **REQ-STATE-04** *Hysteresis:* a transition to a less severe state SHALL require the value to fall 1.0 °C (or 3 %RH for humidity) below the threshold that caused the current state, for 3 consecutive valid samples.
- **REQ-STATE-05** Every state change SHALL be logged once on UART (see REQ-LOG-01) and reflected on outputs within 200 ms of the transition.

### 4.3 Local indication
- **REQ-IND-01** OK → green LED solid. WARN → yellow LED solid. ALARM → red LED solid. FAULT → all three LEDs blinking together at 2 Hz (250 ms on / 250 ms off). Exactly one indication pattern is active at a time.
- **REQ-IND-02** In ALARM the buzzer SHALL pulse at 2 Hz (250 ms on / 250 ms off) until acknowledged or the state leaves ALARM.
- **REQ-IND-03** Pressing the button (debounced 50 ms, falling edge) while in ALARM SHALL silence the buzzer for 60 s. The red LED stays on. If still in ALARM after 60 s the buzzer resumes.
- **REQ-IND-04** Leaving ALARM SHALL stop the buzzer immediately and clear any acknowledgement.

### 4.4 BLE
- **REQ-BLE-01** The device SHALL advertise as `AirSentinel-XXXX` where XXXX is the last 2 bytes of the BT MAC in hex, starting within 2 s of boot, and restart advertising after disconnect.
- **REQ-BLE-02** The device SHALL expose a custom GATT service, UUID `7d1a0001-5a3c-4f6e-9b1a-0c5e5e5e0001`.
- **REQ-BLE-03** Characteristic **Reading** (UUID `7d1a0002-5a3c-4f6e-9b1a-0c5e5e5e0001`, Read + Notify) SHALL hold an 8-byte little-endian packet:

  | Offset | Type | Field |
  |---|---|---|
  | 0 | int16 | temperature, 0.01 °C (0x7FFF if invalid) |
  | 2 | uint16 | humidity, 0.01 %RH (0xFFFF if invalid) |
  | 4 | uint8 | state: 0=OK, 1=WARN, 2=ALARM, 3=FAULT |
  | 5 | uint8 | flags: bit0 = buzzer silenced by ack, bit1 = sample invalid |
  | 6 | uint16 | sequence counter, increments per sample, wraps at 65535 |

- **REQ-BLE-04** When a client has enabled notifications, the device SHALL notify once per sample (1 Hz).
- **REQ-BLE-05** Characteristic **Ack** (UUID `7d1a0003-5a3c-4f6e-9b1a-0c5e5e5e0001`, Write) SHALL accept a single byte `0x01` and behave exactly as a button press (REQ-IND-03).
- **REQ-BLE-06** The on-board status LED SHALL blink at 1 Hz while advertising and be solid while connected.

### 4.5 Logging and telemetry (Live Intelligence)
- **REQ-LOG-01** On every sample the firmware SHALL print one UART line: `[SAMPLE] seq=<n> t=<°C, 1 dp> h=<%RH, 1 dp> state=<OK|WARN|ALARM|FAULT> valid=<0|1>`.
- **REQ-LOG-02** On every state change: `[STATE] <old>-><new> t=<°C> h=<%RH>`. On ack: `[ACK] source=<BUTTON|BLE>`.
- **REQ-LOG-03** Runtime telemetry (FirmGen Enable Telemetry) SHALL expose per-task stack high-water marks, free heap, and CPU load so they can be shown live during the demo.

### 4.6 Reliability
- **REQ-REL-01** The task watchdog SHALL be enabled with a 5 s timeout covering the sampling task; a hang SHALL reset the device.
- **REQ-REL-02** Free heap SHALL not decrease by more than 2 KB over 30 minutes of operation with a connected notifying client.
- **REQ-REL-03** Boot to first indication (LED) SHALL take less than 3 s.

## 5. Non-functional
- **NFR-01** Language C, ESP-IDF v5.5, FreeRTOS tasks (sampling, indication, BLE) with clear separation; the state machine logic SHALL be in a hardware-independent module so it can be unit tested.
- **NFR-02** No dynamic allocation after initialization in the sampling path.
- **NFR-03** Thresholds and timings SHALL be `#define`d/Kconfig constants in one header.

## 6. Acceptance criteria (verified on the board)
| ID | Criterion | Evidence | Covers |
|---|---|---|---|
| AC-01 | Boot, serial shows `[SAMPLE]` line once per second, LED green within 3 s | UART capture + photo | SENS-01, REL-03, IND-01 |
| AC-02 | Warm the sensor (hand/hair-dryer) to ≥ 30 °C: yellow after 3 samples, `[STATE] OK->WARN` logged once | UART + photo | STATE-01/03/05 |
| AC-03 | Continue to ≥ 35 °C: red + buzzer pulsing, `[STATE] WARN->ALARM` | UART + video | STATE-02, IND-01/02 |
| AC-04 | Press button: buzzer silent, red stays on, `[ACK] source=BUTTON`; buzzer resumes after 60 s | UART + video | IND-03 |
| AC-05 | Cool below 34.0 °C for 3 samples: returns to yellow, buzzer off. Hovering at 34.5 °C does NOT flap | UART | STATE-04, IND-04 |
| AC-06 | Unplug sensor data wire: after 3 s all LEDs blink 2 Hz, `FAULT`; replug: recovers | UART + video | SENS-04, IND-01 |
| AC-07 | Phone/laptop sees `AirSentinel-XXXX`, connects, notifications arrive at 1 Hz with the packet decoded correctly | dashboard screenshot | BLE-01…04 |
| AC-08 | Writing 0x01 to Ack characteristic from dashboard silences buzzer | dashboard + UART | BLE-05 |
| AC-09 | Telemetry panel shows stack watermarks, free heap and CPU load live | screenshot | LOG-03 |
| AC-10 | Hang injected in sampling task resets the board within 5 s | UART reset banner | REL-01 |

## 7. Out of scope
Wi-Fi, OTA, persistent storage, BLE bonding/encryption (documented as future work).
