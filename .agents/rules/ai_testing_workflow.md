# AI Autonomous Gateway Testing & Hardware Verification Workflow

## Core Objective
Enable the AI agent to implement firmware changes on the LoRa Gateway, flash the firmware to the hardware, monitor serial output for boot status and panics, execute synthetic LoRa message injection and state assertions in silo, verify both API and human-facing HTML Web UI surfaces, clean up test data, and **pause for mandatory human manual verification** before any commit or Pull Request is created.

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

### Step 2: Flash Firmware & Dynamic Serial IP Discovery
Upload the firmware to the connected Gateway:
```powershell
& "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -t upload
```
Open a serial listener on the configured port at `115200` baud to monitor boot logs until the announced IP address appears (matching regex `IP Address:\s*(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})`):
```text
IP Address:<ip_address>
```
Extract that dynamic `<ip_address>` and use it for all downstream API and Web UI test calls (`http://<ip_address>/...`).

**Mandatory Settling Delay**: Wait **3 additional seconds** after the IP address is announced to ensure mDNS, web server tasks, and background FreeRTOS workers have fully initialized and are listening.

### Step 3: Dual-Surface Verification (API + Human Web UI)
Never test the JSON API in isolation if the feature impacts the browser UI. The automated test suite (`test_gateway_harness.py`) MUST verify both:
1. **Web UI Surface**:
   - Fetch the HTML page (e.g. `GET /settings`) and assert that status badges (e.g. `DISABLED`), buttons, and links render correctly.
   - Trigger user actions via the exact URL/link used in the UI (e.g. `GET /settings?testmode=1`) and assert that the rendered HTML reflects the new state (`ACTIVE`, `Enabled`).
2. **API Surface**:
   - Take pre-test baseline snapshot (`POST /api/test/snapshot`).
   - Query telemetry and status (`GET /api/test/status`).
   - Query baseline device inventory (`GET /api/test/devices`).
   - Inject synthetic sensor frames (`POST /api/test/message`) using isolated synthetic MAC addresses (e.g. `FA0000000001`).
   - Validate internal object tree state (`GET /api/test/device?id=<mac>`).

### Step 4: Clean Teardown & Reset
Restore the Gateway to its clean pre-test baseline:
- Restore Snapshot: `POST http://<gateway_ip>/api/test/restore`
- Verify Device Inventory: Query `GET /api/test/devices` and assert 0 duplicate MACs, 0 leftover test MACs, and exact baseline device count preservation.
- Disable Test Mode via Web UI or API: `GET http://<gateway_ip>/settings?testmode=0`
- Assert that HTML page reflects `DISABLED` state upon reload.

### Step 5: MANDATORY USER MANUAL TESTING GATE (CRITICAL RULE)
Once automated self-tests pass:
1. The AI **MUST STOP** immediately.
2. The AI **MUST NOT** run `git commit`, `git push`, or create any Pull Request.
3. The AI **MUST** inform the user that the firmware is flashed, report the automated test results, and explicitly state:
   > *"Firmware is flashed and verified. I am pausing here for you to manually test and verify the changes on your hardware and browser."*
4. The AI **MUST WAIT** until the user has completed their manual verification and explicitly gives permission to commit and open a Pull Request.

---

## 3. Architecture & Code Invariants
- **Memory Purge on Reload (`clearDevices()`)**: `loadNetwork()` in `SDHelper.h` MUST always invoke `farmNet->clearDevices()` before reading `network.dat` from SD card to prevent duplicate device nodes or leftover test devices in PSRAM.
- **Global State in `main.cpp` Only**: In this ESP32 PlatformIO C++11 environment, *never* define shared state variables using `inline` in header files. All global state must be explicitly instantiated in `main.cpp` and declared `extern` in `.h` headers to guarantee 100% singleton linkage across all translation units.
- **Test Mode Security**: Test Mode MUST default to `OFF` on boot. All `/api/test/*` endpoints MUST return `403 Forbidden` unless Test Mode is actively enabled. Test Mode automatically expires after 1 hour or upon Gateway reboot to prevent test exposure in production.
