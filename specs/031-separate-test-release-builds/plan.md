# Implementation Plan: Move to Separate Test and Release Builds

**Branch**: `031-separate-test-release-builds` | **Date**: 2026-08-23 | **Spec**: [`specs/031-separate-test-release-builds/spec.md`](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/031-separate-test-release-builds/spec.md)  
**GitHub Issue**: [#31](https://github.com/toogooda/LoRaFarmNet/issues/31)

---

## Summary

This feature establishes clean compile-time separation between **Development / Test Builds** and **Production / Release Builds** in the Gateway:
1. **Default Test Build (`[env:gateway-test]`)**:
   - Configured as `default_envs = gateway-test` in `platformio.ini` with `-DGATEWAY_TEST_BUILD`.
   - Test APIs (`/api/test/*`), synthetic message injection, memory snapshots, and simulation tools are compiled in and permanently active.
   - Device Types and Categories **Delete** capability is enabled only in test builds.
   - Message transmission and ACK Serial debug logs are enabled only in test builds.
   - Top navigation brand title is colored blue (`text-info`) with a prominent badge `<span class='badge bg-warning text-dark me-2'><i class='fa-solid fa-flask-vial me-1'></i>TEST BUILD</span>`.
2. **Production Release Build (`[env:gateway-release]`)**:
   - Strips all test APIs, test endpoints, synthetic injection routes, delete capabilities for device types/categories, demo simulator buttons, and verbose transmission Serial debug prints via `#ifdef GATEWAY_TEST_BUILD`.
   - Produces a lean, secure, attack-surface-minimized production binary.
3. **Release Automation (`pio run -t release`)**:
   - Automatically builds `gateway-release` and publishes the stripped production binary asset to GitHub Releases.

---

## Technical Context

- **Platform/Framework**: ESP32 WROVER (Arduino Framework via PlatformIO)
- **Primary Dependencies**: ESPAsyncWebServer, ArduinoJson 7.x, LoRaNetLibrary
- **Preprocessor Flag**: `-DGATEWAY_TEST_BUILD`
- **Environments in `platformio.ini`**:
  - `[env:gateway-test]` (Default for `pio run`, `pio run -t upload`)
  - `[env:gateway-release]` (Target for production releases)

---

## PlatformIO Configuration Design (`platformio.ini`)

```ini
[platformio]
name = LoRaNetGateway
default_envs = gateway-test

[env]
platform = espressif32
board = esp-wrover-kit
framework = arduino
lib_deps =
    knolleary/PubSubClient@^2.8
    https://github.com/dvarrel/AsyncTCP.git
    https://github.com/lacamera/ESPAsyncWebServer.git
    arduino-libraries/NTPClient@^3.2.1
    dvarrel/ESPping@^1.0.5
    bblanchon/ArduinoJson@^7.4.2
    https://github.com/toogooda/LoRaNetLibrary.git
monitor_speed = 115200
monitor_filters = direct, time, esp32_exception_decoder
upload_speed = 921600
upload_port = COM14
board_build.flash_mode = dout
board_build.f_flash = 40000000L
board_build.flash_size = 16MB
board_upload.flash_size = 16MB
board_upload.maximum_size = 16777216
board_build.partitions = partitions.csv

[env:gateway-test]
build_flags = 
    -DBOARD_HAS_PSRAM
    -mfix-esp32-psram-cache-issue
    -DGATEWAY_TEST_BUILD

[env:gateway-release]
build_flags = 
    -DBOARD_HAS_PSRAM
    -mfix-esp32-psram-cache-issue
extra_scripts = post:scripts/publish_release.py
```

---

## Scope of `#ifdef GATEWAY_TEST_BUILD`

1. **Test API Endpoints & Routes (`/api/test/*`)**:
   - Enclosed in `#ifdef GATEWAY_TEST_BUILD`.
   - In test builds, APIs are always active without manual arming or timeout.
2. **Device Types & Categories Delete Capabilities**:
   - In `WebHelper.h`: Delete buttons and delete URL handlers (`/settings?deldevtype=...`, `/settings?delcategory=...`, `/delete?file=/lfm/devtypes/...`) are enclosed in `#ifdef GATEWAY_TEST_BUILD`.
3. **Serial Transmission & ACK Debug Prints**:
   - In `FarmNetwork.cpp`: Gateway message transmission logs and return ACK prints enclosed in `#ifdef GATEWAY_TEST_BUILD`.
4. **UI Simulation Tools**:
   - In `WebHelper.h`: `[🧪 Simulate Router Response (Demo)]` button and simulator JS on `/adddevice` enclosed in `#ifdef GATEWAY_TEST_BUILD`.
5. **Navbar Visual Indicator**:
   - In `sendPageHeader()`: Brand title is blue (`text-info`), and when `GATEWAY_TEST_BUILD` is defined, renders `<span class='badge bg-warning text-dark me-2 font-monospace'><i class='fa-solid fa-flask-vial me-1'></i>TEST BUILD</span>`.

---

## Proposed Phase Breakdown

- **Phase 1**: Update `platformio.ini` with `gateway-test` (default) and `gateway-release`.
- **Phase 2**: Apply conditional `#ifdef GATEWAY_TEST_BUILD` blocks in `WebHelper.h`, `main.cpp`, and `FarmNetwork.cpp` (Test APIs, devtype delete, Serial logs, UI badge, demo simulator).
- **Phase 3**: Update `publish_release.py` to package the `gateway-release` binary.
- **Phase 4**: Update `test_gateway_harness.py` for always-active test endpoints and `TEST BUILD` badge verification.
- **Phase 5**: Build and compile-verify both environments (`pio run -e gateway-test` and `pio run -e gateway-release`).
- **Phase 6**: Flash `gateway-test` to Gateway hardware, run automated tests, and pause for human manual testing gate.
