# Implementation Plan: Online Device Type Resolution, Minimum Firmware Enforcement & Auto-Upgrade

**Branch**: `008-online-device-types-and-auto-upgrade` | **Date**: 2026-08-17 | **Spec**: [specs/008-online-device-types-and-auto-upgrade/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/008-online-device-types-and-auto-upgrade/spec.md)

**Input**: Feature specification from `specs/008-online-device-types-and-auto-upgrade/spec.md` (GitHub Issue #8)

---

## Summary

Enable the LoRa Gateway to dynamically resolve, download, and configure newly purchased/released physical LoRa device types from GitHub over-the-air.
1. **Unknown Device Type Detection**: When a packet arrives with a `DT` ID not in `registry.dat`, the gateway displays **"Unknown Device Type (ID: X)"** with a blue **"Check Online"** button and yellow **"Ignore"** button (and NO "Setup" button).
2. **Online Resolution & Ingestion**:
   - "Check Online" queries `https://raw.githubusercontent.com/toogooda/LoRaFarmNet/master/templates/manifest.json` (skipping any entries with `"enabled": false`).
   - Downloads the corresponding `.typ` file to SD card `/lfm/devtypes/<filename>.typ`.
   - Parses the XML `<header>` block (`<dtid>`, `<dtcat>`, `<dthw>`, `<minfw>`).
   - If the category does not exist locally, syncs its name and icon from `templates/categories.json`, assigning next `displayOrder` and `isMini = false`.
   - Atomically updates `registry.dat` and `categories.dat`.
3. **Minimum Firmware Version (`minfw`) Enforcement**:
   - If `current_firmware >= minfw` (or `minfw == "0"`): applies template immediately, transitioning device card to "New Device (Auto-Configured)" with green **"Setup"** and yellow **"Ignore"** buttons.
   - If `current_firmware < minfw`: displays *"Requires Gateway Firmware v<minfw>+ (Current: v<current>)"* with an **"Update Gateway Firmware"** action button.
   - Post-reboot boot scanner in `loadNetwork()` automatically detects pending un-setup devices with `DT` ports, re-evaluates their `minfw` constraint against the upgraded firmware, and auto-applies the template.
4. **Template Authoring Wizard**:
   - On gateway web UI, "Create Template" prompts for `dtid`, `catid`, `dthw`, `minfw` (default `0`), and filename.
   - Writes the self-describing `<header>` to SD card and triggers a direct browser file download for committing to the repository.
5. **Settings Screen**:
   - Displays a clean read-only list of Supported Devices (Name and HW version).
   - Retains Category display ordering and Mini/Full view mode configuration.

---

## Technical Context

**Language/Version**: C++11 (ESP-IDF / Arduino Framework on ESP32-WROVER)
**Primary Dependencies**: `HTTPClient`, `WiFiClientSecure`, `ArduinoJson` (v7), `SD`, `ESPAsyncWebServer`
**Storage**: SD Card FAT32 (`/lfm/devtypes/registry.dat`, `/lfm/devtypes/categories.dat`, `/lfm/devtypes/*.typ`, `/lfm/data/network.dat`)
**Target Platform**: ESP32-WROVER with 8MB PSRAM
**Testing**: Manual hardware testing using a physical node transmitting `DT = 0`

---

## Constitution Check

- **I. Target Platform**: ESP32-WROVER with PSRAM mandatory. *Compliant.*
- **II. Power Integrity & Airtime**: Gateway does not sleep. LoRa communication follows established invariants. *Compliant.*
- **III. Frame Protocol**: Fixed-pair binary stream (`DT` sensor value port parsed from message). *Compliant.*
- **IV. Mandatory PR Workflow**: Feature branch `008-online-device-types-and-auto-upgrade` & GitHub PR required; manual user approval required. *Compliant.*
- **V. Issue Lifecycle**: Closing GitHub Issue #8 is scheduled as the final task upon merge. *Compliant.*

---

## Project Structure & Planned Changes

```text
LoRaFarmNet/
├── templates/                                 # [NEW] Central repository templates
│   ├── categories.json                        # Category definitions (ID, Name, FontAwesome icon)
│   ├── manifest.json                          # Device Type index (DTID, HW, MinFW, CatID, Filename, enabled)
│   ├── test_dt0_v1_0.typ                      # Initial test template for DT = 0
│   └── ...                                    # Existing standard templates
└── Gateway/LoRaNetGateway/
    ├── src/
    │   ├── FarmNetwork.h                      # Add minFirmware field to DeviceTypeDefinition
    │   ├── FarmNetwork.cpp                    # Auto-configuration evaluation & minfw validation logic
    │   ├── SDHelper.h                         # Template XML header parsing, template saving with header, boot scanner
    │   ├── WebHelper.h                        # Unknown DT card, "Check Online" handler, template download endpoint, settings UI
    │   └── Version.h                          # Version bump
```

---

## Proposed Implementation Phases

### Phase 1: Data Model & Repository Artifacts
1. Extend `DeviceTypeDefinition` in `FarmNetwork.h` with `char minFirmware[16]`.
2. Update `SDHelper.h` `loadRegistry()` and `saveRegistry()` to persist `minFirmware` (with backward compatibility defaulting to `"0"`).
3. Create `templates/categories.json`, `templates/manifest.json`, and initial `.typ` template files in the root `templates/` directory.

### Phase 2: Template XML Header Parsing & Generation
1. Update `SDHelper.h` `loadTemplate()` and `proccessTemplate()` to parse the `<header>` block (`<dtid>`, `<dtcat>`, `<dthw>`, `<minfw>`).
2. Update `SDHelper.h` `saveTemplateFromDevice()` to write the `<header>` block.
3. Add HTTP endpoint `/downloadtemplate?file=...` in `WebHelper.h` for direct browser download of `.typ` files.

### Phase 3: Online Resolution Engine ("Check Online")
1. Implement `checkOnlineDeviceType(uint16_t dtid, Print* logOut)` in `WebHelper.h` / helper:
   - Fetches `https://raw.githubusercontent.com/toogooda/LoRaFarmNet/master/templates/manifest.json`.
   - Iterates entries, skips `"enabled": false`, finds matching `dtid`.
   - If found, downloads `.typ` file via `WiFiClientSecure` / `HTTPClient` to `/lfm/devtypes/<filename>.typ`.
   - Parses header, syncs category from `templates/categories.json` if new.
   - Registers in `registry.dat`.
   - Compares `minfw` against `GATEWAY_VERSION_STRING`:
     - If satisfied: applies template immediately via `loadTemplate(d, filename)`.
     - If not satisfied: flags device as requiring firmware upgrade.
2. In `WebHelper.h` `sendDevice()`:
   - For `DeviceType::New` devices with 0 configured entities:
     - Check if `DT` sensor value exists.
     - If `DT` is not in registry: render **"Unknown Device Type (ID: X)"** with **"Check Online"** and **"Ignore"** buttons.
     - If `DT` is in registry but `minfw > current_firmware`: render **"Requires Gateway Firmware v<minfw>+ (Current: v<current>)"** with **"Update Gateway Firmware"** button.
     - If `DT` is in registry and `minfw` is satisfied: render standard **"New Device (Auto-Configured)"** with **"Setup"** and **"Ignore"** buttons.

### Phase 4: Post-Reboot Boot Scanner
1. In `loadNetwork()` in `SDHelper.h`:
   - After loading devices, scan any devices in `DeviceType::New` with 0 entities.
   - If device has a `DT` port matching an installed registry template whose `minfw` is satisfied by the current firmware, auto-apply `loadTemplate(d, dt->filename)`.

### Phase 5: Settings Page Streamlining
1. Update `/settings` in `WebHelper.h`:
   - Display a clean read-only table of Supported Devices (Name, HW Version).
   - Retain Category display ordering and Mini/Full view mode configuration.
   - In template creation form, prompt for `dtid`, `catid`, `dthw`, `minfw`, and filename, saving to SD and triggering browser download.

### Phase 6: Verification, PR & Release
1. Build with `pio run`.
2. Conduct manual hardware test using `DT = 0`.
3. Create Pull Request and publish release upon user merge.
4. Close GitHub Issue #8.
