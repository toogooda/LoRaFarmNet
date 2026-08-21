# AI Autonomous Gateway Testing & Hardware Verification Workflow

## Core Objective
Enable the AI agent to implement firmware changes on the LoRa Gateway, flash the firmware to the hardware, monitor serial output for boot status and panics, execute synthetic LoRa message injection and state assertions in silo, and clean up test data before presenting fully verified work for human review.

---

## 1. Serial Port & Parameter Discovery
Before flashing or monitoring, inspect `Gateway/LoRaNetGateway/platformio.ini` to discover:
- **`upload_port` / `monitor_port`**: Active serial COM port (e.g. `COM3`).
- **`monitor_speed`**: Baud rate (default `115200`).
- **`monitor_filters`**: Filters used (`direct, time, esp32_exception_decoder`).

---

## 2. Hardware-in-the-Loop Test Loop

When implementing Gateway features, execute the following cycle:

### Step 1: Compilation Check
Run PlatformIO build verification:
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run
```

### Step 2: Flash Firmware & Serial IP Discovery
Upload the firmware to the connected Gateway:
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -t upload
```
Open a serial listener on the configured port at `115200` baud to monitor boot logs until the announced IP appears:
```text
IP Address:192.168.68.104
```
**Mandatory Settling Delay**: Wait **3 additional seconds** after the IP address is printed to ensure mDNS, web server tasks, and background FreeRTOS workers have fully initialized and are listening.

### Step 3: Enable Test Mode & Snapshot Baseline
Activate Test Mode (which arms the in-memory 1-hour safety timer) and snapshot `network.dat`:
- Enable Test Mode: `POST http://<gateway_ip>/settings/testmode?enable=1`
- Snapshot Baseline: `POST http://<gateway_ip>/api/test/snapshot`

### Step 4: Execute Automated Test Injections & Assertions
Use `Gateway/LoRaNetGateway/scripts/test_gateway_harness.py` or HTTP calls to:
- Inject synthetic sensor frames (`POST /api/test/message`) with source MAC, port map, RSSI, and SNR.
- Validate internal state (`GET /api/test/device?id=<mac>`) to inspect sensor values, computed values, entities, translations, and highlight rules.
- Test production web endpoints (e.g. `/device?deviceid=<mac>&setup=1`, `/checkonline`, `/entity`).

### Step 5: Clean Teardown & Reset
Restore the Gateway to its clean pre-test baseline:
- Restore Snapshot: `POST http://<gateway_ip>/api/test/restore`
- Disable Test Mode: `POST http://<gateway_ip>/settings/testmode?enable=0`

---

## 3. Security Invariants
- Test Mode MUST default to `OFF` on boot.
- All `/api/test/*` endpoints MUST return `403 Forbidden` unless Test Mode is actively enabled.
- Test Mode automatically expires after 1 hour or upon Gateway reboot to prevent test exposure in production.
