# Feature Specification: GitHub OTA Firmware Updates & Dynamic Release Automation

**Feature Branch**: `006-ota-using-github`

**Created**: 2026-08-15

**Status**: Draft

**Input**: User description from GitHub Issue #6 (`toogooda/LoRaFarmNet`):
> "ESP32Wrover-E 16mb want to be able to check for updates from settings screen and if a newer version of firmware exists in github id does an OTA update.
> Architectural Decision: This project has multiple repos the two options are LoRaFarmNet which is a parent repo where issues are raised and can be changed to public if it needs to be and https://github.com/toogooda/LoRaNetGateway which has the actual source for the device but can never be public. This decision must be made by me you need to think through the approach and explain the pros and cons so i can decide the the binary's will live.
> You need to decide how the device will know its newer (needs to be able to be faked for testing.
> How should we handle the user experience of a web connection when an OTA starts?
> should probably do a network save first also."

---

## Architectural Decision: Dynamic Public Release Hosting

**Decision**: **Option A — Public GitHub Releases on `toogooda/LoRaFarmNet`**.

### 1. Release Architecture & Dynamic Toolchain
* **Source Code Repository**: `toogooda/LoRaNetGateway` (Private repository where source code is maintained and built).
* **Release Distribution Repository**: `toogooda/LoRaFarmNet` (Public parent repository holding releases, changelogs, and binary assets).
* **Single Source of Truth**: Firmware version is strictly defined in `src/Version.h`:
  ```cpp
  #pragma once
  #define GATEWAY_VERSION_MAJOR 1
  #define GATEWAY_VERSION_MINOR 0
  #define GATEWAY_VERSION_PATCH 0
  #define GATEWAY_VERSION_STRING "1.0.0"
  #define GATEWAY_BUILD_TIMESTAMP __DATE__ " " __TIME__
  ```
* **Dynamic PlatformIO Release Target (`pio run -t release`)**:
  - Configured via PlatformIO `extra_scripts = post:scripts/publish_release.py`.
  - Dynamically extracts the current version from `src/Version.h`.
  - Automatically compiles the target environment `.pio/build/esp-wrover-kit/firmware.bin`.
  - Uses the system GitHub CLI (`gh.exe`) to create the release tag `v<VERSION>` on `toogooda/LoRaFarmNet`, attaches `gateway-firmware-v<VERSION>.bin`, and auto-generates release notes from git commits.
  - No hardcoded version numbers or manual asset uploads.

### 2. Assistant Automation Rule (`RepoRules.md`)
* The repository rules in `Gateway/LoRaNetGateway/.agent/rules/RepoRules.md` will be updated with an explicit workflow rule:
  - When the user prompts the AI assistant to *"create a release"*, the assistant will:
    1. Confirm/determine the SemVer increment (major, minor, or patch) and update `src/Version.h`.
    2. Execute `pio run -t release`.
    3. Output the new release tag, version string, and link to the published GitHub release.

---

## Versioning & Testing Simulation Strategy

### 1. Version Comparison Logic
* Current firmware version is compiled into the firmware from `src/Version.h`.
* GitHub releases follow tag convention `vX.Y.Z` (e.g., `v1.0.1`).
* Comparison logic on ESP32:
  * Parse Semantic Version tuples `(Major, Minor, Patch)`.
  * Remote version > Current version triggers `Update Available`.
  * Remote version <= Current version indicates `Up to Date`.

### 2. Test Version Simulation & Force-Update Mechanism
To allow testing and verification on real hardware without tagging false GitHub releases:
* **Settings UI Test Override Field**: On the Settings page under Developer / Testing Settings (or via URL query parameter `?check_version_override=0.0.1`), allow the user to simulate/fake the currently running firmware version.
* **Force Update Option**: An explicit button/option "Force Re-flash Current Version" allowing an OTA flash even if the remote tag matches the running version.

---

## Web Connection User Experience (UX) During OTA

1. **State Preservation**: The Gateway must automatically execute `saveNetwork()`, `saveRegistry()`, and `saveCategories()` prior to initiating flash writes.
2. **Dedicated OTA Progress View**:
   - When the user clicks **"Start OTA Update"**, the UI transitions to a dedicated status/progress overlay:
     - Step 1: Saving network configuration to SD (`saveNetwork()`).
     - Step 2: Downloading & streaming firmware binary to OTA partition (`Update.writeStream()` / `HTTPUpdate`).
     - Step 3: Verifying image CRC and finalizing partition.
     - Step 4: Countdown timer (15–20 seconds) while ESP32 reboots into the new slot (`ESP.restart()`).
   - The browser polls `/` or a lightweight ping endpoint via JavaScript `fetch()`.
   - Once the gateway responds with the updated version string, the UI displays **"Update Complete! Reloading..."** and refreshes to the updated Settings page.
   - If an error occurs (network drop, verification failure, partition error), a clear error alert is displayed with a **Retry** button without rebooting.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Dynamic Firmware Release Publishing (Priority: P1) 🎯 MVP

As a developer working in `LoRaNetGateway`, when ready for a new release, asking the assistant to create a release or running `pio run -t release` automatically compiles the binary and publishes a new release `vX.Y.Z` with attached firmware asset to `toogooda/LoRaFarmNet`.

**Why this priority**: Automates the distribution pipeline between the private source repo and the public release repo with zero hardcoded manual steps.

**Independent Test**: Bump version in `src/Version.h`, execute `pio run -t release`, and verify on `github.com/toogooda/LoRaFarmNet/releases` that the new release tag and `.bin` asset exist.

**Acceptance Scenarios**:
1. **Given** a version change in `src/Version.h`, **When** `pio run -t release` is executed, **Then** the firmware compiles and a GitHub release is created on `toogooda/LoRaFarmNet` with the matching tag and binary.

---

### User Story 2 - Check for Updates and Trigger GitHub OTA from Settings (Priority: P1) 🎯 MVP

From the Gateway Web UI Settings page (`/settings`), the user clicks "Check for Updates". The Gateway queries GitHub Releases, displays the latest release version, release notes/summary, and published date alongside the current running version. If a newer version is found, an "Install Update" button appears, allowing the user to initiate the OTA flash.

**Why this priority**: Core user-facing feature enabling remote device updates over WiFi.

**Independent Test**: Navigate to `/settings`, click "Check for Updates", verify the release metadata is displayed correctly, click "Install Update", and observe the firmware downloading and flashing into the secondary OTA partition.

**Acceptance Scenarios**:
1. **Given** the Gateway is connected to WiFi with Internet access, **When** the user clicks "Check for Updates", **Then** the Gateway retrieves the latest release tag from GitHub and displays it on the UI.
2. **Given** a newer version is available (or test override active), **When** the user confirms the update, **Then** the Gateway downloads the binary, writes to the inactive OTA slot (`app0`/`app1`), verifies the binary, and reboots into the new firmware.

---

### User Story 3 - Automated Network State Preservation Prior to Flashing (Priority: P1)

Before allocating OTA buffers or modifying flash partitions, the Gateway must commit all current runtime state, devices, categories, and registry entries to non-volatile SD storage (`saveNetwork()`, `saveRegistry()`, `saveCategories()`).

**Why this priority**: Prevents corruption or loss of device configurations and network topology across firmware reboots and upgrades.

**Independent Test**: Modify a device setting, initiate an OTA update without manually clicking "Save", verify upon reboot into the new firmware that all device settings and network topology are intact.

**Acceptance Scenarios**:
1. **Given** active device configurations or runtime state, **When** an OTA update is triggered, **Then** the system synchronously saves network, registry, and category data to SD storage prior to commencing flash writes.

---

### User Story 4 - Interactive Web UI UX During OTA & Post-Reboot Reconnection (Priority: P2)

When an update is initiated, the web browser UI presents an informative status dialog showing progress steps (Saving -> Downloading -> Flashing -> Rebooting) and an automatic reconnection poller so the user is never left with a broken connection or white screen.

**Why this priority**: Prevents user panic, accidental power cycling, or duplicate update requests while flash write and reboot occur.

**Independent Test**: Trigger an update from the browser. Observe continuous visual progress feedback, countdown timer during reboot, and automatic reload of the web dashboard when the gateway comes back online.

**Acceptance Scenarios**:
1. **Given** an OTA update in progress, **When** viewed in the browser, **Then** the interface displays an animated progress bar and status messages.
2. **Given** the Gateway restarts after a successful flash, **When** the gateway re-establishes WiFi and starts the WebServer, **Then** the browser reconnects automatically and confirms the new running version.

---

### User Story 5 - Version Simulation and Rollback / Force-Flash for Testing (Priority: P2)

Developers can simulate different running version numbers or force re-flashing the same version to test OTA pipelines repeatedly without tagging artificial GitHub releases.

**Why this priority**: Vital for automated and manual verification in development/staging environments.

**Independent Test**: Enter a simulated version `0.0.1` in the Developer Settings section or enable "Force Flash", verify that "Update Available" is shown, and perform the OTA cycle.

**Acceptance Scenarios**:
1. **Given** the Gateway is running the latest version, **When** a simulated lower version is set in the test interface, **Then** the UI treats the latest release as a newer version and enables the "Install Update" button.
2. **Given** "Force Flash" is selected, **When** the update is triggered, **Then** the system proceeds with flashing regardless of version comparison.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST define a compile-time semantic version string in `src/Version.h` (`GATEWAY_VERSION_STRING`, `GATEWAY_VERSION_MAJOR`, `MINOR`, `PATCH`, `BUILD_TIMESTAMP`).
- **FR-002**: The Settings page MUST display the currently running firmware version and compile date/time.
- **FR-003**: The Gateway MUST provide an endpoint/handler to query the GitHub Releases API over HTTPS for `toogooda/LoRaFarmNet`.
- **FR-004**: The system MUST parse the latest release tag, release notes, and extract the `gateway-firmware-*.bin` asset download URL.
- **FR-005**: The system MUST implement semantic version comparison between the running version (or test simulated version) and the remote GitHub release tag.
- **FR-006**: The system MUST support a developer test mechanism (URL query param or UI input) to simulate the current running version or force an update.
- **FR-007**: The system MUST execute `saveNetwork()`, `saveRegistry()`, and `saveCategories()` prior to initializing the OTA stream.
- **FR-008**: The system MUST stream the remote `.bin` binary directly into the target OTA partition (`Update.h` / `HTTPUpdate`) with progress tracking.
- **FR-009**: The system MUST validate partition write integrity and CRC before setting the boot partition and restarting.
- **FR-010**: The Web UI MUST provide a progress modal during the OTA process and an automated JS heartbeat poller to refresh the page after reboot.
- **FR-011**: The system MUST gracefully report error conditions (WiFi disconnection, download failure, partition full, CRC mismatch) without bricking the device or leaving the bootloader in an unbootable state.
- **FR-012**: If the OTA process fails before completion, the system MUST abort the update, keep the active partition unchanged, and report the specific error to the UI.
- **FR-013**: The project MUST provide a custom PlatformIO release target (`pio run -t release` via `scripts/publish_release.py`) that reads `src/Version.h`, compiles the binary, and publishes the release with assets to `toogooda/LoRaFarmNet` using `gh.exe`.
- **FR-014**: Gateway repository rules (`.agent/rules/RepoRules.md`) MUST include the release automation instruction for the AI assistant.
- **FR-015**: The `toogooda/LoRaFarmNet` parent repository MUST be configured to **Public** visibility so the Gateway ESP32 can query release endpoints and download binary assets over HTTPS without requiring authentication tokens.

### Key Entities

- **FirmwareReleaseInfo**: Structure holding `versionTag`, `releaseNotes`, `assetUrl`, `assetSize`, `publishedAt`.
- **OTAStatus**: State enum (`IDLE`, `CHECKING`, `DOWNLOADING`, `FLASHING`, `REBOOTING`, `ERROR`, `SUCCESS`) with progress percentage and error message.
- **OTASettings**: Configurable repository target (`owner`, `repo`, optional test version override).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Executing `pio run -t release` builds and publishes a tagged release with binary asset to `toogooda/LoRaFarmNet` in under 30 seconds.
- **SC-002**: Release check queries complete in under 5 seconds over a standard WiFi internet connection.
- **SC-003**: Complete OTA download and flash cycle finishes in under 60 seconds on standard broadband connections.
- **SC-004**: 100% of successful flashes reboot seamlessly into the updated partition with all network, device, category, and registry data intact.
- **SC-005**: If network connection drops mid-download, the device safely aborts without corrupting the running partition and recovers to normal operation.

---

## Assumptions & Dependencies

- Target board is ESP32-Wrover-E with 16MB Flash and standard dual OTA partition table (`app0` / `app1`).
- The `toogooda/LoRaFarmNet` parent repository visibility is changed from Private to **Public** (required for unauthenticated API and asset download access from ESP32).
- Gateway has active WiFi Internet connectivity via the existing `ESPWIFIConfigFromBLE` WiFi manager.
- SSL certificates for `api.github.com` and `objects.githubusercontent.com` are verified via bundle or secure client.
- GitHub CLI (`gh.exe`) is installed and authenticated on the development environment.
