# Implementation Plan: GitHub OTA Firmware Updates & Dynamic Release Automation

**Branch**: `006-ota-using-github` | **Date**: 2026-08-15 | **Spec**: [specs/006-ota-using-github/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/006-ota-using-github/spec.md)

**Input**: Feature specification from `specs/006-ota-using-github/spec.md` and GitHub Issue #6 (`toogooda/LoRaFarmNet`).

---

## Summary

1. **Version Single Source of Truth**: Create `src/Version.h` with semantic version constants (`GATEWAY_VERSION_STRING`, major/minor/patch, build timestamp).
2. **Dynamic PlatformIO Release Target**: Implement `scripts/publish_release.py` and register `pio run -t release` in `platformio.ini` to dynamically read `src/Version.h`, compile the binary, and publish the release with asset to `toogooda/LoRaFarmNet` via GitHub CLI (`gh.exe`).
3. **OTA Engine & GitHub Client (`OTAHelper.h`)**:
   - Query GitHub Releases API (`api.github.com/repos/toogooda/LoRaFarmNet/releases/latest`) over HTTPS with SSL/root cert or bundle.
   - Parse SemVer tag, extract release notes, and locate `gateway-firmware-*.bin` download URL.
   - Provide SemVer comparison logic and developer simulation/force-flash override.
   - Perform atomic OTA flash writes to inactive partition with stream progress reporting.
   - Safeguard with ESP-IDF rollback protection (`esp_ota_mark_app_valid_cancel_rollback`).
4. **State Preservation**: Synchronously invoke `saveNetwork()`, `saveRegistry()`, and `saveCategories()` prior to commencing OTA flash writes.
5. **Settings UI & Async Endpoints**:
   - Add firmware version display, "Check for Updates" button, test override input, and update summary to `/settings`.
   - Implement `/ota/check`, `/ota/start`, and `/ota/status` async JSON endpoints.
   - Implement interactive JavaScript OTA progress modal with heartbeat poller that automatically reconnects and refreshes the page once reboot completes.
6. **Agent Release Rule**: Add assistant release automation instruction to `Gateway/LoRaNetGateway/.agent/rules/RepoRules.md`.
7. **Issue Closure**: Update and close GitHub Issue #6 upon completion.

---

## Technical Context

**Language/Version**: C++11 / Arduino framework on ESP32-WROVER (ESP-IDF underlying)  
**External Dependencies (lib_deps)**: **ZERO (0) new libraries added** — Uses existing project dependencies (`AsyncTCP`, `ESPAsyncWebServer`, `ArduinoJson`) and built-in ESP32 core libraries (`HTTPClient`, `WiFiClientSecure`, `Update`, `esp_ota_ops`).  
**Storage**: Dual OTA flash partitions (`app0`, `app1` at 3.4MB each in `partitions.csv`), SD card storage (`network.dat`, `registry.dat`, `categories.dat`)  
**Testing**: `pio run` build verification + hardware testing on ESP32-Wrover  
**Target Platform**: ESP32-Wrover-E (16MB Flash, 8MB PSRAM)  
**Project Type**: Embedded Gateway Firmware / Web Application  

---

## Constitution Check

- **Hardware Target Invariant**: Explicitly targets ESP32-WROVER with PSRAM enabled (`-DBOARD_HAS_PSRAM`). **PASS**
- **Always-On Device**: Gateway never enters deep sleep; performs active OTA and reboots smoothly. **PASS**
- **Binary Frame & Shared Libraries**: Non-interfering with core LoRa protocols. **PASS**
- **Build Verification & PR Workflow**: All changes verified with `pio run`; changes committed to feature branch and PR created for user review. **PASS**
- **GitHub Issue Lifecycle**: GitHub Issue #6 explicitly scheduled to be updated and closed upon completion. **PASS**

---

## Project Structure

### Documentation (this feature)

```text
specs/006-ota-using-github/
├── spec.md              # Feature specification
└── plan.md              # This technical implementation plan
```

### Source Code (Gateway/LoRaNetGateway repository)

```text
Gateway/LoRaNetGateway/
├── platformio.ini               # Add extra_scripts = post:scripts/publish_release.py
├── scripts/
│   └── publish_release.py       # Custom PlatformIO target for dynamic release publication
├── .agent/
│   └── rules/
│       └── RepoRules.md         # Add AI assistant release workflow rule
└── src/
    ├── Version.h                # Semantic version constants (Single Source of Truth)
    ├── OTAHelper.h              # GitHub release checker, version comparer, and OTA flasher
    ├── WebHelper.h              # Settings page UI enhancements & OTA progress modal
    └── main.cpp                 # Register /ota/check, /ota/start, /ota/status endpoints
```

---

## Detailed Technical Approach

### Phase 1: Semantic Versioning & Release Automation
1. **Create `src/Version.h`**:
   - Define `GATEWAY_VERSION_MAJOR`, `GATEWAY_VERSION_MINOR`, `GATEWAY_VERSION_PATCH`, `GATEWAY_VERSION_STRING`, and `GATEWAY_BUILD_TIMESTAMP`.
2. **Create `scripts/publish_release.py`**:
   - Python script registering custom SCons environment target `release`.
   - Parses `GATEWAY_VERSION_STRING` from `src/Version.h`.
   - Runs compilation of `esp-wrover-kit` firmware.
   - Invokes `gh release create v<VERSION>` on `toogooda/LoRaFarmNet`, attaches `.pio/build/esp-wrover-kit/firmware.bin` renamed as `gateway-firmware-v<VERSION>.bin`, and generates release notes.
3. **Update `platformio.ini`**:
   - Add `extra_scripts = post:scripts/publish_release.py`.
4. **Update `RepoRules.md`**:
   - Add rule for AI assistant to handle "create a release" requests.

### Phase 2: Core OTA Engine (`OTAHelper.h`)
1. **GitHub Release Fetcher**:
   - Use `WiFiClientSecure` with `setInsecure()` (or GitHub root CA bundle) and `HTTPClient`.
   - Send `User-Agent: LoRaFarmNetGateway-ESP32`.
   - Parse JSON response with `ArduinoJson` (tag name, release body, asset `browser_download_url`, asset size).
2. **Version Comparison & Test Override**:
   - Parse `vX.Y.Z` into integers `(major, minor, patch)` and compare.
   - Support `simulateVersion` parameter for developer testing on hardware.
   - Support `forceUpdate` flag.
3. **Data Preservation**:
   - Call `saveNetwork()`, `saveRegistry()`, and `saveCategories()` before OTA stream start.
4. **Partition Streaming & Flashing**:
   - Use `Update.begin(contentLength, U_FLASH)` and stream chunks from `WiFiClientSecure` to `Update.write()`.
   - Track progress percentage in thread-safe status structure (`OTAStatus`).
   - Call `Update.end()` and `Update.isFinished()`.
   - Call `esp_ota_set_boot_partition()` and trigger `ESP.restart()`.
5. **Bootloader Rollback Validation**:
   - In `main.cpp` `setup()`, call `esp_ota_mark_app_valid_cancel_rollback()` once system initializes successfully.

### Phase 3: Web UI Integration & Endpoints
1. **Settings Page (`WebHelper.h`)**:
   - Render "Firmware & Updates" card in `/settings`:
     - Current Version & Build Time.
     - "Check for Updates" button.
     - Release notes and latest available version display.
     - "Install Update" button.
     - Developer Settings section (Simulate Version input & Force Update checkbox).
2. **Async Endpoints (`main.cpp`)**:
   - `GET /ota/check`: Query GitHub and return JSON status.
   - `POST /ota/start`: Trigger async background OTA task and respond immediately.
   - `GET /ota/status`: Return current progress, state enum, and any error message.
3. **Interactive UI Progress Modal**:
   - JavaScript modal rendering step progression: `Saving Config` -> `Downloading (X%)` -> `Flashing` -> `Rebooting (Countdown)`.
   - Polling loop testing gateway accessibility during reboot; auto-refreshes page to `/settings` once back online.

---

## Planned Code Changes

#### [NEW] [Version.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/Version.h)
- Central definition of firmware semantic version and build timestamps.

#### [NEW] [publish_release.py](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/scripts/publish_release.py)
- SCons / PlatformIO target script to build and publish release to `toogooda/LoRaFarmNet`.

#### [NEW] [OTAHelper.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/OTAHelper.h)
- GitHub client, version comparator, pre-save hook, OTA stream downloader, and rollback validator.

#### [MODIFY] [platformio.ini](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/platformio.ini)
- Register `extra_scripts = post:scripts/publish_release.py`.

#### [MODIFY] [RepoRules.md](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/.agent/rules/RepoRules.md)
- Add AI assistant release workflow instructions.

#### [MODIFY] [WebHelper.h](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)
- Add firmware version display, update checking UI, test version simulation, and OTA progress modal script.

#### [MODIFY] [main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/main.cpp)
- Add `/ota/check`, `/ota/start`, `/ota/status` endpoints and boot rollback confirmation.

---

## Verification Plan

### Automated Build Verification
- Execute `pio run` in `Gateway/LoRaNetGateway` to confirm compilation without errors or memory overflow.

### Release Tooling Verification
- Execute `pio run -t release --dry-run` (or test execution) to verify version parsing from `Version.h`.

### Manual Hardware & Web Verification
1. Open Web UI Settings page (`/settings`), verify current version string `1.0.0` and build date are displayed.
2. Enter a simulated version `0.0.1` in developer override, click "Check for Updates", verify the latest GitHub release is detected as an available update.
3. Click "Install Update", observe live progress modal:
   - Verification that `network.dat` is saved to SD.
   - Verification that download progress bar advances.
   - Verification that device restarts and browser poller automatically reconnects and reloads the page.
4. Verify all devices, categories, and settings remain intact after reboot.
