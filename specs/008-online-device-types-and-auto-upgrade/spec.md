# Feature Specification: Online Device Type Resolution, Minimum Firmware Enforcement & Auto-Upgrade

**Feature Branch**: `008-online-device-types-and-auto-upgrade`

**Created**: 2026-08-17

**Last Updated**: 2026-08-17 (Refined Settings display & Template creation export workflow)

**Status**: Draft

**Input**: User description from GitHub Issue #8 & Architecture Clarifications:
> "Device type not found then download new type and if needed upgrade device.
> As a user who owns existing devices and a LoRa gateway when I purchase a new device type that was created after the gateway was programmed:
> 1) When new device shows up it will not find a match for device type in the list, instead of configuring entities and offering a setup button it will display Unknown Device Type and a Button 'Check Online' or the normal Ignore Button.
> 2) The Gateway will look for type file with matching type file and if it finds one downloads to SD card with other type files then checks the minimum firmware to see if it can support the new device, if it can it does the normal auto entity setup and displays the normal Setup or Ignore buttons if it can't it explains and requests to do firmware update to latest version and follows the normal process. Because it will reboot we need to take care that the un-setup device has another go at matching and auto entity setup and not left empty with no entities.
> Architecture Clarifications:
> - Category Icons belong at the Category level, NOT inside individual Device Type headers.
> - Categories and their icons will exist as a separate central source artifact (`templates/categories.json`) in the public repository.
> - When downloading a new Device Type, the gateway ensures the category definition (ID, Name, Icon) is synced from the central category artifact, while allowing the user to configure Category Display Order and Mini vs Full view mode.
> - Testing will be conducted manually using the user's physical node with Device Type 0 (`DT = 0`).
> - Template source files and manifest will reside in `toogooda/LoRaFarmNet/templates/`.
> - `manifest.json` supports `"enabled": true/false` for commenting out / swapping options during testing.
> - `/settings` page only needs to show a clean read-only list of supported device names.
> - The download/export requirement is during **Template Creation** (when creating a template from a manually configured device, prompt for header items and provide a downloadable `.typ` file for committing to GitHub)."

---

## User Scenarios & Testing

### User Story 1 - Unknown Device Type Detection & Online Resolution (Priority: P1)

As a user deploying a physical LoRa node with a Device Type (`DT`) ID not yet present on my gateway SD card (such as `DT = 0` during testing or newly released hardware), I want the gateway to display that an unknown device type was received and offer a **"Check Online"** button, so that the gateway can automatically query GitHub, download the matching template (`.typ`), sync any new category from the central categories artifact, and auto-configure the device.

**Why this priority**:
Core MVP requirement. Allows newly introduced device types to be discovered, downloaded, and configured over-the-air from GitHub without manual file copying.

**Manual Test with DT = 0**:
1. Remove or unregister the definition for `DT = 0` from the gateway.
2. Transmit a packet from the physical node with `DT = 0`.
3. Verify the device card on the home page displays:
   - **"Unknown Device Type (ID: 0)"**
   - Blue **"Check Online"** button
   - Yellow **"Ignore"** button
   - (No "Setup" button)
4. Click **"Check Online"**:
   - Gateway queries the repository manifest (`templates/manifest.json`), skipping any entries with `"enabled": false`.
   - Downloads the corresponding `.typ` template file to `/lfm/devtypes/`.
   - Checks if the referenced `catid` exists locally; if not, syncs its name and icon from `templates/categories.json`.
   - Ingests the template into `registry.dat`.
   - Applies the template to the device in memory.
5. Verify the device card dynamically transitions to **"New Device (Auto-Configured)"** with green **"Setup"** and yellow **"Ignore"** buttons.

**Acceptance Scenarios**:
1. **Given** a new node transmits a packet with `DT = X` not in the local registry, **When** the gateway renders the device on the home page, **Then** it shows "Unknown Device Type (ID: X)" with "Check Online" and "Ignore" buttons (no "Setup" button).
2. **Given** the user clicks "Check Online" and the template exists on GitHub (and is enabled), **When** the download completes, **Then** the `.typ` file is saved to SD, category definition is synced if new, registry is updated, the template is applied, and the device card switches to "New Device (Auto-Configured)" with "Setup".
3. **Given** the user clicks "Check Online" and no template exists online for `DT = X` (or is disabled), **When** the check completes, **Then** the user is notified with an informative alert ("No template found online for Device Type X").

---

### User Story 2 - Minimum Firmware Version Enforcement & OTA Upgrade Path (Priority: P1)

As a user installing a new device type that requires specialized firmware capabilities (e.g. custom router protocol, interactive buttons, or gate control), I want the gateway to compare the template's minimum required firmware version (`minfw`) against its currently running firmware version (`current_firmware`), and if the gateway is outdated, guide me to perform an OTA firmware update while guaranteeing that the un-setup device will auto-configure upon reboot.

**Why this priority**:
Prevents devices with complex interactions from malfunctioning on older gateway firmware that lacks the necessary handlers.

**Manual Test**:
1. In `templates/manifest.json`, activate a test template for `DT = 0` requiring `minfw = "2.0.0"` on a gateway running `v1.0.3`.
2. Click "Check Online". Verify the gateway downloads and registers the template, but does not apply entity configurations yet.
3. Verify the device card displays:
   - *"Requires Gateway Firmware v2.0.0+ (Current: v1.0.3)"*
   - Blue **"Update Gateway Firmware"** button linking to the OTA update flow.
4. Trigger an OTA update.
5. After reboot, verify that the boot scanner in `loadNetwork()` automatically detects the pending un-setup device, re-evaluates its `DT` requirement against the new firmware, and applies the template.

**Acceptance Scenarios**:
1. **Given** a downloaded template specifies `minfw = "1.2.0"` and gateway is on `1.0.3`, **When** evaluated, **Then** the device card displays "Requires Gateway Firmware v1.2.0+ (Current: v1.0.3)" and provides an "Update Gateway Firmware" button.
2. **Given** a device is pending firmware upgrade, **When** the gateway is updated and reboots, **Then** the boot process re-evaluates the device, applies the template, and presents the device as "New Device (Auto-Configured)".

---

### User Story 3 - Clean Separation of Categories & Device Type Templates (Priority: P2)

As a system architect and user, I want Category definitions (Name, Icon) to live in a single central repository artifact (`templates/categories.json`) and Device Types to live in standalone `.typ` files with concise headers (`dtid`, `catid`, `dthw`, `minfw`), so that icons and categories are managed cleanly at the category level and newly downloaded templates automatically inherit their proper category icon.

**Why this priority**:
Eliminates duplication, ensures consistency across device types sharing the same category, and allows the user to configure display order and Mini vs Full view modes locally.

**Acceptance Scenarios**:
1. **Given** a new device type is downloaded referencing `catid = 12`, **When** `catid = 12` does not exist locally on the gateway, **Then** the gateway fetches `templates/categories.json`, adds category 12 with its official name and FontAwesome icon, sets `displayOrder = (max_order + 1)`, and sets `isMini = false` (Full view default).
2. **Given** existing categories on the gateway, **When** the user configures Category View Order or toggles Mini vs Full mode in `/settings`, **Then** the user's custom layout preferences are preserved in `/lfm/devtypes/categories.dat`.

---

### User Story 4 - Template Creation Wizard with Header Prompts & Direct File Download (Priority: P2)

As a developer and gateway user who has manually configured a new device on the gateway web interface, I want to create a new Device Type Template from that device by filling in the header prompts (`dtid`, `catid`, `dthw`, `minfw`, filename), saving it locally to SD card, and immediately downloading the generated `.typ` file to my computer, so that I can easily commit it to `toogooda/LoRaFarmNet/templates/`.

**Why this priority**:
Provides the authoring tool for developers to generate valid, self-describing `.typ` files directly from live working devices.

**Acceptance Scenarios**:
1. **Given** a configured device on the gateway, **When** the user chooses "Create Device Type Template", **Then** a modal prompts for `dtid`, `catid`, `dthw`, `minfw` (default `0`), and filename.
2. **Given** the form is submitted, **When** the gateway generates the `.typ` file with the XML `<header>` block, **Then** it saves it to `/lfm/devtypes/` and provides a direct browser download of the `.typ` file.

---

### User Story 5 - Clean Supported Devices Display in Settings (Priority: P3)

As a user visiting `/settings`, I want to view a clean read-only list of all currently supported device types (names and hardware versions) and manage category layout (display order, Mini vs Full view).

**Why this priority**:
Improves UI ergonomics and aligns the settings screen with the new template-driven architecture.

**Acceptance Scenarios**:
1. **Given** the user views `/settings`, **When** looking at Supported Devices, **Then** a clean reference list displays the names and hardware versions of all installed device types.
2. **Given** the user edits a category's display order or view mode (Mini/Full), **When** saved, **Then** the preference persists in `/lfm/devtypes/categories.dat`.

---

## Edge Cases

1. **GitHub Unreachable / Offline Operation**:
   - If "Check Online" fails due to no internet, DNS resolution error, or GitHub downtime, display an alert/notification: *"Unable to connect to GitHub. Please check internet connection."* The device card remains in the "Unknown Device Type" state and the "Check Online" button remains enabled and clickable so the user can freely click it again to retry.
2. **Testing Overrides via `manifest.json`**:
   - Entries in `manifest.json` with `"enabled": false` or prefixed with `_` are skipped by the gateway parser, allowing seamless swapping of test cases on GitHub.
3. **Category Already Exists Locally**:
   - If the device type references an existing local category ID, the existing user-configured display order and Mini/Full view preferences are preserved without being overwritten.
4. **Corrupted Template / Incomplete Download**:
   - Download is performed atomically via a `.tmp` file. If corrupted or missing closing tags, the file is deleted and the registry is left untouched.
5. **Multiple Devices of Same Unknown Type**:
   - Downloading a template immediately applies to all active in-memory devices matching that `DT`.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST detect incoming packets where sensor value `DT` does not match any entry in `registry.dat`.
- **FR-002**: For unknown device types, system MUST display **"Unknown Device Type (ID: <id>)"** with **"Check Online"** and **"Ignore"** buttons on the device card.
- **FR-003**: System MUST query `toogooda/LoRaFarmNet/templates/manifest.json` on GitHub when "Check Online" is invoked, ignoring any entries with `"enabled": false`.
- **FR-004**: System MUST download matching `.typ` template files to SD card `/lfm/devtypes/<filename>.typ` using atomic writes.
- **FR-005**: All `.typ` template files MUST contain an XML `<header>` block with:
  - `<dtid>` (uint16_t): Device Type ID
  - `<dtcat>` (uint16_t): Category ID
  - `<dthw>` (float): Hardware Version (e.g. 1.0)
  - `<minfw>` (string): Minimum required Gateway firmware version (default `"0"`)
- **FR-006**: Central categories MUST be defined in `toogooda/LoRaFarmNet/templates/categories.json` containing `id`, `name`, and `icon`.
- **FR-007**: When a template is downloaded referencing an unknown `catid`, system MUST fetch `categories.json`, register the category with its official name and icon, set `displayOrder = (max + 1)`, and default to Full view (`isMini = false`).
- **FR-008**: System MUST compare `minfw` against currently running firmware:
  - If `current_firmware >= minfw` or `minfw == "0"`: Apply template immediately and enable **Setup**.
  - If `current_firmware < minfw`: Display *"Requires Gateway Firmware v<minfw>+ (Current: v<current>)"* with an **"Update Gateway Firmware"** action button.
- **FR-009**: On boot following an OTA update, `loadNetwork()` MUST re-evaluate all un-setup devices with `DT` ports against the active firmware and auto-apply templates whose `minfw` is now satisfied.
- **FR-010**: Template creation wizard on the gateway MUST prompt for `dtid`, `catid`, `dthw`, `minfw` (default `0`), and filename, write the XML `<header>` block to SD card, and trigger a browser download of the generated `.typ` file.
- **FR-011**: The `/settings` page MUST display a clean, read-only list of Supported Devices (Name and HW Version) while allowing user configuration of Category display order and Mini/Full view modes.

---

## Central Repository Artifact Structure

The `toogooda/LoRaFarmNet` repository will contain a new `templates/` folder:

```
LoRaFarmNet/
├── templates/
│   ├── categories.json        # Central Category definitions (ID, Name, FontAwesome Icon)
│   ├── manifest.json          # Index of Device Types (DTID, HW, MinFW, CatID, Filename, enabled flag)
│   ├── watertank_v1_0.typ     # Individual template XML files with <header> blocks
│   ├── solar_v1_0.typ
│   ├── gatecontroller_v1_0.typ
│   ├── ...
```

---

## Measurable Success Criteria

- **SC-001**: Unknown device types (`DT = 0` during manual test) show "Check Online" within 1 second of packet arrival.
- **SC-002**: "Check Online" successfully downloads `.typ` and syncs category definitions from GitHub in under 5 seconds over standard Wi-Fi.
- **SC-003**: 100% of downloaded templates with satisfied `minfw` auto-configure device entities and show the "Setup" button without requiring reboot.
- **SC-004**: Devices requiring higher firmware version display upgrade guidance and auto-configure upon boot following the OTA update.
- **SC-005**: Template creation wizard outputs a valid `.typ` file with header and downloads it to the user's browser in a single interaction.
- **SC-006**: User-customized Category display order and Mini/Full view modes remain persistent across template downloads.
