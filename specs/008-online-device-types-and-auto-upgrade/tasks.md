# Tasks: Online Device Type Resolution, Minimum Firmware Enforcement & Auto-Upgrade

**Input**: Design documents from `specs/008-online-device-types-and-auto-upgrade/`
**Prerequisites**: [plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/008-online-device-types-and-auto-upgrade/plan.md), [spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/008-online-device-types-and-auto-upgrade/spec.md)

---

## Phase 1: Setup & Initial Repository Artifacts

**Purpose**: Create repository templates folder, category definitions, manifest index, and test template.

- [x] T001 Create `templates/categories.json` in repository root with standard category IDs, names, and FontAwesome icons.
- [x] T002 Create `templates/manifest.json` in repository root with index of device types, supporting `"enabled": true/false`.
- [x] T003 Create initial `.typ` template files (including `test_dt0_v1_0.typ` for manual hardware testing of `DT = 0`) in `templates/`.

---

## Phase 2: Foundational Data Structures & Template Headers

**Purpose**: Core data model extensions and XML header serialization.

- [x] T004 Extend `DeviceTypeDefinition` in `Gateway/LoRaNetGateway/src/FarmNetwork.h` to include `minFirmware` version string.
- [x] T005 Update `SDHelper.h` `loadRegistry()` and `saveRegistry()` to persist and load `minFirmware` with backward compatibility default `"0"`.
- [x] T006 Update `SDHelper.h` `saveTemplateFromDevice()` to write self-describing XML `<header>` block (`<dtid>`, `<dtcat>`, `<dthw>`, `<minfw>`).
- [x] T007 Update `SDHelper.h` `loadTemplate()` and `proccessTemplate()` to parse `<header>` tags from `.typ` files.

---

## Phase 3: User Story 1 - Unknown Device Type Detection & Online Resolution (Priority: P1) 🎯 MVP

**Goal**: When a device with an unrecognized `DT` arrives, render "Unknown Device Type (ID: X)" with a "Check Online" button that downloads the `.typ` file and category from GitHub and auto-configures entities.

- [x] T008 [US1] Update `sendDevice()` in `Gateway/LoRaNetGateway/src/WebHelper.h` to detect unknown `DT` on new devices and render "Unknown Device Type (ID: X)" with blue "Check Online" button and yellow "Ignore" button (no "Setup" button).
- [x] T009 [US1] Implement `checkOnlineDeviceType(uint16_t dtid)` in `Gateway/LoRaNetGateway/src/WebHelper.h`:
  - Fetch `templates/manifest.json` from GitHub via `WiFiClientSecure` / `HTTPClient`, skipping `"enabled": false`.
  - Download matching `.typ` file to `/lfm/devtypes/<filename>.typ` atomically.
  - If category does not exist locally, sync category name and icon from `templates/categories.json`.
  - Ingest into `registry.dat` and `categories.dat`.
- [x] T010 [US1] Implement `checkonline` GET request handler in `WebHelper.h` to invoke `checkOnlineDeviceType()` and provide feedback/redirect to the user with retry capability on connection failure.

---

## Phase 4: User Story 2 - Minimum Firmware Enforcement & Boot Scanner (Priority: P1)

**Goal**: Enforce `minfw` check on downloaded templates, guide user to OTA upgrade when outdated, and automatically auto-configure un-setup devices upon reboot.

- [x] T011 [US2] Update template evaluation in `FarmNetwork.cpp` / `WebHelper.h` to compare `minfw` against `GATEWAY_VERSION_STRING`.
- [x] T012 [US2] Update `sendDevice()` in `WebHelper.h` to render "Requires Gateway Firmware v<minfw>+ (Current: v<current>)" and "Update Gateway Firmware" button when `current_firmware < minfw`.
- [x] T013 [US2] Implement boot scanner in `SDHelper.h` `loadNetwork()` to inspect un-setup `DeviceType::New` devices with `DT` ports on reboot and auto-apply templates whose `minfw` is now satisfied.

---

## Phase 5: User Story 4 & 5 - Template Creation Export & Settings Streamlining (Priority: P2/P3)

**Goal**: Prompt for `minfw` in template creation with direct browser download, and clean up Settings page.

- [x] T014 [US4] Add `/downloadtemplate` HTTP GET endpoint in `WebHelper.h` to stream `.typ` files from `/lfm/devtypes/` to browser.
- [x] T015 [US4] Update "Create Template" modal in `WebHelper.h` to prompt for `dtid`, `catid`, `dthw`, `minfw` (default `0`), and filename, then trigger file download.
- [x] T016 [US5] Update `/settings` in `WebHelper.h` to display a clean read-only list of Supported Devices (Name and HW version) while preserving Category order and view mode configuration.

---

## Phase 6: Build Verification, Hardware Testing & Release

**Goal**: Build firmware, conduct manual hardware test with `DT = 0`, bump version, and create Pull Request.

- [x] T017 Run local PlatformIO build verification (`pio run`) in `Gateway/LoRaNetGateway`.
- [x] T018 Conduct manual verification using physical node transmitting `DT = 0`.
- [x] T019 Bump gateway firmware version to `v1.4.0` in `src/Version.h`.
- [x] T020 Push feature branch `008-online-device-types-and-auto-upgrade` and create Pull Request on `toogooda/LoRaNetGateway`.
- [x] T021 Close GitHub Issue #8 upon user PR approval and merge.
