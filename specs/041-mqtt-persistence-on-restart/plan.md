# Implementation Plan: 041 - MQTT Settings Persistence Across Restarts

**Branch**: `041-mqtt-persistence-on-restart` | **Spec**: [specs/041-mqtt-persistence-on-restart/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/041-mqtt-persistence-on-restart/spec.md)

---

## Proposed Changes

### 1. `Gateway/LoRaNetGateway/src/WebHelper.h`
- In `sendSettingsPage()`:
  - Add `saveNetwork()` when `request->hasParam("usemqtt")` changes `farmNet->getUseMQTT()`.
  - Add `saveNetwork()` when `request->hasParam("ip")` or `request->hasParam("port")` changes the MQTT server or port.
- In `sendDevicePage()`:
  - Add `saveNetwork()` when `request->hasParam("usemqtt")` toggles device-level `d->setUseMQTT()`.

### 2. `Gateway/LoRaNetGateway/src/SDHelper.h`
- In `proccessItem()`:
  - Fix line 543 from `curD->setUseMQTT((bool)v);` to `curD->setUseMQTT((bool)atoi(v));`.

---

## Verification Plan

### Automated Build Verification
- Execute `pio run -e gateway-test` in `Gateway/LoRaNetGateway`.

### Hardware Verification & Test Harness
- Upload firmware to test gateway (`pio run -e gateway-test -t upload`).
- Verify live persistence via HTTP API and SD download endpoint.
- Run `python scripts/test_gateway_harness.py --ip 192.168.68.63 --run-suite`.
- Pause for manual human testing confirmation.
