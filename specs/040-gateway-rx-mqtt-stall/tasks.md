# Tasks: Non-Blocking FreeRTOS MQTT & Network Monitor Architecture

**Feature Branch**: `040-gateway-rx-mqtt-stall`  
**GitHub Issue**: [toogooda/LoRaFarmNet#39](https://github.com/toogooda/LoRaFarmNet/issues/39)  
**Prerequisites**: [`spec.md`](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/040-gateway-rx-mqtt-stall/spec.md), [`plan.md`](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/040-gateway-rx-mqtt-stall/plan.md)

---

## Phase 1: Setup & Dependencies

- [x] T001 Restore canonical `knolleary/PubSubClient@^2.8` and remove conflicting `AsyncMqttClient` / `AsyncTCP-esphome` libraries to prevent `_poll()` serial flooding and web server latency.

---

## Phase 2: Foundational (Task Watchdog with BLE Exception Gating)

- [x] T002 In `Gateway/LoRaNetGateway/src/ESPWIFIConfigFromBLE.h` and `ESPWIFIConfigFromBLE.cpp`, expose `isBleActive()` / `isBleConnected()` and add watchdog feeds (`esp_task_wdt_reset()`) inside interactive prompt wait loops (`getBLESelection`, `getBLEWord`, `connectWithBLE`).
- [x] T003 In `Gateway/LoRaNetGateway/src/main.cpp`, initialize ESP32 Task Watchdog Timer (`esp_task_wdt_init(90, true)` and `esp_task_wdt_add(NULL)`) in `setup()` and feed `esp_task_wdt_reset()` at the start of `loop()`.

---

## Phase 3: Non-Blocking FreeRTOS MQTT Architecture

- [x] T004 In `Gateway/LoRaNetGateway/src/MQTTHelper.h` and `MQTTHelper.cpp`:
  - Implement thread-safe `mqttQueue` (`std::deque<MqttQueueItem>` with `SemaphoreHandle_t`).
  - Make `publishRetained()` sub-microsecond non-blocking: pushes to queue with 0 tick wait.
  - Implement dedicated FreeRTOS `mqttWorkerTask` pinned to Core 0 (rate-limited connection attempt every 10s, drains queue, maintains keepalives without disconnecting between messages).

---

## Phase 4: Non-Blocking Network Monitor & Resilient Boot

- [x] T005 In `Gateway/LoRaNetGateway/src/ESPWIFIConfigFromBLE.cpp`, update boot WiFi failure: if saved configuration cannot connect on boot, immediately `ESP.restart()` rather than looping or dropping into BLE fallback.
- [x] T006 In `Gateway/LoRaNetGateway/src/main.cpp`:
  - Remove blocking 25-attempt ping loop (`checkWiFiConnection()`) and destructive `reconnectWiFi()`.
  - Implement `networkMonitorTask` on Core 0 running every 10 seconds:
    - Ignores transient 2–3s network glitches without resetting.
    - If WiFi link or local router gateway is continuously down for 2 minutes (120s), cleanly reboots via `ESP.restart()`.
    - Updates `detailedStatus` in background for web UI badges.

---

## Phase 5: Verification & Automated Hardware Testing

- [x] T007 Compile firmware using PlatformIO:
  - `pio run -e gateway-test` (SUCCESS)
- [x] T008 Flash test firmware (`pio run -e gateway-test -t upload`), verify dynamic serial IP (`192.168.68.97`), and execute full automated regression suite:
  - `python scripts/test_gateway_harness.py --ip 192.168.68.97 --run-suite` (ALL 12 TESTS PASSED)
- [x] T009 **MANDATORY HUMAN MANUAL TESTING GATE (CRITICAL RULE)**:
  - Pause and notify user to manually verify web dashboard and fire off a physical LoRa message from a node to verify over-the-air RF reception. (COMPLETED & VERIFIED)

---

## Phase 6: Pull Request & Issue Closure

- [x] T010 Ask user permission to push feature branch `040-gateway-rx-mqtt-stall` and open GitHub Pull Request.
- [ ] T011 Upon PR merge, pull `master`, delete local and remote branches, update and close GitHub issue [toogooda/LoRaFarmNet#39](https://github.com/toogooda/LoRaFarmNet/issues/39).
