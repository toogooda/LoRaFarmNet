# Implementation Plan: Non-Blocking AsyncMqttClient Migration & BLE-Gated Task Watchdog

**Branch**: `040-gateway-rx-mqtt-stall` | **GitHub Issue**: [toogooda/LoRaFarmNet#39](https://github.com/toogooda/LoRaFarmNet/issues/39) | **Spec**: [`specs/040-gateway-rx-mqtt-stall/spec.md`](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/040-gateway-rx-mqtt-stall/spec.md)

---

## Summary

Migrate the Gateway from the synchronous, blocking `PubSubClient` library to the asynchronous, event-driven `AsyncMqttClient` library (which shares the existing `AsyncTCP` engine with `ESPAsyncWebServer`). This completely eliminates all blocking socket connection timeouts, retry loops, and delays from the main execution thread when the MQTT broker or home network drops offline. In addition, configure an ESP32 Task Watchdog Timer (TWDT) for `loop()` (60–90s threshold) with an explicit gating mechanism that pauses/feeds the watchdog whenever Bluetooth Low Energy (BLE) configuration sessions or client connections are active.

---

## Technical Context

- **MCU Target**: ESP32-WROVER (8MB PSRAM mandatory).
- **Core Framework**: Arduino-ESP32 / ESP-IDF.
- **Key Libraries**:
  - `AsyncTCP`: Existing low-level async networking engine.
  - `AsyncMqttClient`: Event-driven asynchronous MQTT client replacing `PubSubClient`.
  - `ESPAsyncWebServer`: Web server running asynchronously on Core 0.
  - `esp_task_wdt.h`: ESP-IDF Task Watchdog Timer.
- **Constraints**:
  - Zero blocking socket calls in Arduino `loop()`.
  - BLE configuration and credential entry can take several minutes without triggering watchdog resets.
  - All existing Home Assistant discovery and retained sensor topic formats MUST remain 100% backward compatible.
  - Passes regression test suite (`scripts/test_gateway_harness.py`).

---

## Phase 0: Research & Dependency Selection

1. **`AsyncMqttClient` Package**:
   - The ESP32 Arduino ecosystem has `heman/AsyncMqttClient-esphome@^2.1.0` and `marvinroger/AsyncMqttClient@^0.9.0`. Both compile directly against `AsyncTCP`.
   - Update `Gateway/LoRaNetGateway/platformio.ini` to add `AsyncMqttClient` and verify compilation.
2. **BLE Exception Gating**:
   - In `ESPWIFIConfigFromBLE.h`, add a public method `bool isBleActiveOrConnected() const` returning `(_bleIsOn || _deviceConnected)`.
   - In `ESPWIFIConfigFromBLE.cpp`, inside blocking wait loops (`getBLESelection`, `getBLEWord`), explicitly call `esp_task_wdt_reset()` on each loop pass or yield so that active BLE typing does not starve the watchdog.
3. **Task Watchdog Implementation**:
   - Use ESP-IDF Task Watchdog API (`esp_task_wdt_init(60, true)` and `esp_task_wdt_add(NULL)` in `setup()`).
   - At the beginning of `loop()` in `main.cpp`, call `esp_task_wdt_reset()`.

---

## Phase 1: Architecture & Module Changes

```text
c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/
├── platformio.ini               # Replace PubSubClient with AsyncMqttClient
└── src/
    ├── MQTTHelper.h             # Update API prototypes for AsyncMqttClient
    ├── MQTTHelper.cpp           # Event-driven AsyncMqttClient callbacks & non-blocking publish
    ├── ESPWIFIConfigFromBLE.h   # Expose isBleActiveOrConnected()
    ├── ESPWIFIConfigFromBLE.cpp # Feed watchdog during interactive BLE setup prompts
    ├── main.cpp                 # Initialize TWDT, feed in loop(), handle non-blocking MQTT
    └── SensorValue.cpp          # Non-blocking publishRetained() invocations
```

### Detailed Component Logic
1. **`MQTTHelper.cpp`**:
   - Instantiate `AsyncMqttClient mqttClient`.
   - Attach event callbacks:
     - `onConnect`: Log connection, reset drop counter, publish retained registration/discovery payloads.
     - `onDisconnect`: Log disconnect reason, increment telemetry drops, schedule non-blocking reconnect timer.
     - `onError`: Handle socket errors cleanly without looping.
   - `publishRetained(topic, payload)`:
     - If `!mqttClient.connected()`, silently drop or return (best effort).
     - If connected, call `mqttClient.publish(topic, 0, true, payload)`.
     - Zero `delay()` calls, zero while-loops.
2. **Watchdog Management**:
   - Initialize TWDT in `setup()` with 60–90 second panic timeout.
   - In `loop()`:
     - Reset TWDT: `esp_task_wdt_reset()`.
   - In `ESPWIFIConfigFromBLE.cpp`:
     - Inside wait loops: reset TWDT to accommodate slow typing/phone interaction.

---

## Constitution & Workflow Check
- [x] Compilation verification: `pio run -e gateway-test` and `pio run -e gateway-release`.
- [x] Dual-surface hardware testing workflow per `Gateway/LoRaNetGateway/.agent/rules/ai_testing_workflow.md`.
- [x] Mandatory Human Manual Testing Gate prior to commit/PR.
- [x] GitHub Issue #39 lifecycle tracked.
