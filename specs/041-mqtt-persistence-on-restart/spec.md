# Feature Specification: 041 - MQTT Settings Persistence Across Restarts

**Feature Branch**: `041-mqtt-persistence-on-restart`  
**Created**: 2026-11-06  
**Status**: In Progress  
**Target Repository**: `Gateway/LoRaNetGateway`  
**MCU Target**: ESP32-WROVER (8MB PSRAM)  
**Input**: User report: "investigate the mechanisim for turning MQTT off, I have a prod and a test gateway running no matter what i try to turn of prod withing a day or two both are sending results to home assistant. i switch the slider on settings pages but it doesn't seem to save on restart."

---

## 1. Problem Statement & Motivation
1. **Missing SD Persistence on Global MQTT Toggle & Server IP/Port Updates**:
   When the user toggles the "Use MQTT" switch on the Settings page (`/settings?usemqtt=0`), `WebHelper.h` updates `farmNet->setUseMQTT(false)` in RAM and sets `changedUseMQTT = true`, but never calls `saveNetwork()`.
   Similarly, updating the MQTT IP and Port fields (`/settings?ip=...&port=...`) updates RAM without calling `saveNetwork()`.
   Consequently, the SD card file `/lfm/data/network.dat` continues to store `<gusemqtt>1</gusemqtt>`.

2. **Unwanted Re-enabling on Gateway Restarts**:
   Whenever the gateway automatically or manually reboots (e.g. Wi-Fi outage recovery reboot after 2 minutes, router ping failure, low heap watchdog reboot, power cycle), `loadNetwork()` parses `network.dat` from SD card and reloads `<gusemqtt>1</gusemqtt>`, reactivating MQTT transmission to Home Assistant.

3. **Device-Level `dusemqtt` Pointer Cast Bug in XML Parsing**:
   In `SDHelper.h` line 543:
   `curD->setUseMQTT((bool)v);`
   `v` is a `const char*` pointer. Casting a non-null pointer directly to `bool` in C++ always evaluates to `true`, ignoring the parsed `"0"` or `"1"` string content. Even if an individual device had `<dusemqtt>0</dusemqtt>` on SD card, it is always restored as `true` on boot.

4. **Missing SD Persistence on Per-Device MQTT Toggle**:
   When toggling MQTT on an individual device page (`/device?deviceid=...&usemqtt=0`), `WebHelper.h` updates `d->setUseMQTT(false)` in RAM and deregisters from Home Assistant, but does not invoke `saveNetwork()`.

---

## 2. Requirements & Expected Behavior

### Requirement 1: Global MQTT Toggle & Server IP/Port Persistence
- When `/settings?usemqtt=<0|1>` is received and changes `useMQTT`, invoke `saveNetwork()`.
- When `/settings?ip=...&port=...` is received and changes the IP or Port, invoke `saveNetwork()`.
- Verify `/lfm/data/network.dat` immediately reflects `<gusemqtt>0</gusemqtt>` or `<gusemqtt>1</gusemqtt>`.

### Requirement 2: Device-Level XML Parsing Fix
- In `proccessItem()` in `SDHelper.h`, change `curD->setUseMQTT((bool)v)` to `curD->setUseMQTT((bool)atoi(v))`.
- When `network.dat` is loaded with `<dusemqtt>0</dusemqtt>`, the device's `useMQTT` state remains `false`.

### Requirement 3: Per-Device MQTT Toggle Persistence
- When `/device?deviceid=...&usemqtt=<0|1>` is requested, invoke `saveNetwork()` to persist the per-device MQTT preference.

---

## 3. Verification & Acceptance Criteria
1. **Compilation**: `pio run -e gateway-test` builds with zero errors.
2. **Global Persistence Test**:
   - Issue `GET /settings?usemqtt=0`.
   - Download `/download?path=/lfm/data/&file=network.dat` and assert `<gusemqtt>0</gusemqtt>`.
   - Issue `GET /settings?usemqtt=1`.
   - Download `/download?path=/lfm/data/&file=network.dat` and assert `<gusemqtt>1</gusemqtt>`.
3. **Regression Suite**:
   - Run `python scripts/test_gateway_harness.py --ip 192.168.68.63 --run-suite` and assert 100% pass rate.
