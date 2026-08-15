# Tasks: GitHub OTA Firmware Updates & Dynamic Release Automation

**Input**: Design documents from `/specs/006-ota-using-github/` (`spec.md`, `plan.md`)

---

## Phase 1: Setup, Semantic Versioning & Release Tooling

**Goal**: Establish the central version definition, PlatformIO dynamic release publisher, and agent repository rule.

- [x] T001 [P] Create `Gateway/LoRaNetGateway/src/Version.h` with `GATEWAY_VERSION_MAJOR`, `GATEWAY_VERSION_MINOR`, `GATEWAY_VERSION_PATCH`, `GATEWAY_VERSION_STRING`, and `GATEWAY_BUILD_TIMESTAMP`
- [x] T002 [P] Create `Gateway/LoRaNetGateway/scripts/publish_release.py` to dynamically parse `Version.h`, compile the binary, and publish the release with asset to `toogooda/LoRaFarmNet` via `gh.exe`
- [x] T003 Configure `Gateway/LoRaNetGateway/platformio.ini` with `extra_scripts = post:scripts/publish_release.py`
- [x] T004 Update `Gateway/LoRaNetGateway/.agent/rules/RepoRules.md` with the AI assistant release workflow instruction
- [x] T005 Verify compilation with `pio run`

---

## Phase 2: Core OTA Engine & State Preservation (`OTAHelper.h`)

**Goal**: Implement GitHub HTTPS release query, SemVer comparison, testing simulation override, synchronous network preservation, and partition streaming flash with rollback protection.

- [x] T006 Create `Gateway/LoRaNetGateway/src/OTAHelper.h` implementing:
  - `checkGitHubRelease()`: HTTPS GET query to `api.github.com/repos/toogooda/LoRaFarmNet/releases/latest` using `WiFiClientSecure` / `HTTPClient` and parsing JSON with `ArduinoJson`
  - SemVer parsing & comparison (`isNewerVersion()`)
  - Test version simulation override & force-flash logic
  - Synchronous pre-flash hook invoking `saveNetwork()`, `saveRegistry()`, and `saveCategories()`
  - Partition streaming write via `Update.begin()`, `Update.writeStream()`, CRC verification, and partition activation
  - ESP-IDF bootloader rollback confirmation (`esp_ota_mark_app_valid_cancel_rollback()`)
- [x] T007 Integrate rollback check into `Gateway/LoRaNetGateway/src/main.cpp` `setup()`
- [x] T008 Verify compilation with `pio run`

---

## Phase 3: Web UI Integration & Async Endpoints

**Goal**: Add Firmware & Updates section to Settings page, register async endpoints, and implement live progress modal with auto-reconnection poller.

- [x] T009 [P] Register `/ota/check`, `/ota/start`, and `/ota/status` endpoints in `Gateway/LoRaNetGateway/src/main.cpp`
- [x] T010 Update `Gateway/LoRaNetGateway/src/WebHelper.h` `sendSettingsPage()` to include:
  - Current running version and build timestamp display
  - "Check for Updates" button and latest release details card
  - Developer Testing Settings (Simulate Version input & Force Update option)
  - Interactive JavaScript OTA progress modal and post-reboot heartbeat poller
- [x] T011 Verify compilation with `pio run`

---

## Phase 4: Verification & Release Target Testing

**Goal**: Verify end-to-end build integrity and test release tooling.

- [x] T012 Run full clean build verification `pio run` in `Gateway/LoRaNetGateway`
- [x] T013 Verify PlatformIO release target execution / dry-run

---

## Phase 5: Hardware Manual Testing & Verification

**Goal**: Deploy to physical ESP32 Gateway and verify functionality.

- [x] T014 Upload firmware to Gateway and verify Settings UI renders version and "Check for Updates"
- [x] T015 Test version simulation override with a mock version and verify OTA download and flash cycle
- [x] T016 Pause for user hardware testing and verification


---

## Phase 6: Pull Request & Issue Closure

**Goal**: Finalize feature branch, create PR, and close GitHub Issue #6.

- [x] T017 Ask user permission to push branch `006-ota-using-github` and create Pull Request
- [x] T018 Update and close GitHub Issue #6 upon PR approval/merge

