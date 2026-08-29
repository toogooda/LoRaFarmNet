# Feature Specification: Unified Add Device Experience & Enhanced `sendDevice()` Not-Setup View

**Feature Branch**: `037-unified-add-device-senddevice`

**Created**: 2026-08-28

**Status**: Draft

**Input**: User requirement:
> "Combine Add Device page (009) and online device type resolution & auto-upgrade (008) into a unified architecture using `sendDevice()` for unsetup devices. Enhance `sendDevice()` not-setup view with Via Router badge, right-aligned RSSI badge below ID, next message countdown badge on bottom left, category icon + 'New [Device Name]' title, and consistent FontAwesome button icons."

---

## Background & Problem Statement

1. **Track 008** added online template resolution on the Main Page (`/`):
   - When an unrecognized `DT` arrives, `sendDevice()` displays **"Unknown Device Type (ID: X)"** with **[Check Online]** to fetch `.typ` templates from GitHub.
   - When a template requires newer gateway firmware, `sendDevice()` shows **"Requires Firmware vX+"** with **[Update Firmware]**.
   - When auto-configured, it displays **[Setup]** and **[Ignore]**.
2. **Track 009** created the `/adddevice` page with real-time SSE packet discovery (`/api/pair/events`), but built a separate, complex client-side JS card representation that lacked 008 online resolution and auto-upgrade workflows.
3. **Objective**:
   - Make `sendDevice()` the single source of truth for unsetup device presentation across both `/` and `/adddevice`.
   - Enhance `sendDevice()`'s `DeviceType::New` view with key operational metadata (Via Router badge, RSSI, Countdown timer) and consistent FontAwesome button icons.
   - Ensure all new metadata is strictly isolated to `DeviceType::New` (invisible once configured).
   - Support real-time dynamic SSE card updates on `/adddevice` when routed packets arrive.

---

## User Scenarios & Requirements

### User Story 1 - Informative Unsetup Card in `sendDevice()` (Priority: P1)
As a farm network operator viewing unsetup devices on the Home Page (`/`) or Add Device Page (`/adddevice`), I want `sendDevice()` to clearly display the device name, whether it was heard via a router (`Via <Router Name>`), its radio RSSI below the ID, the expected next message arrival time, and consistent icon-enabled action buttons.

**Acceptance Scenarios**:
1. **Row 1 (Header)**:
   - Displays device name `d->getName()` on the left.
   - If `MR >= 100` (routed via repeater/router), displays a blue badge `<span class='badge bg-info text-dark'><i class='fa-solid fa-satellite-dish me-1'></i>Via [Router Name]</span>` in the center between the name and ID.
   - Displays `ID: <6-byte MAC>` on the right.
2. **Row 2 (Status & RSSI)**:
   - Displays `[Category Icon] + "New " + [Device Name]` on the left (e.g. `<i class='fa-solid fa-water text-primary me-1'></i> New Water Tank Level`).
   - If unknown DT: `<i class='fa-solid fa-circle-question text-info me-1'></i> Unknown Device Type (ID: [ID])`.
   - If requires newer firmware: `<i class='fa-solid fa-triangle-exclamation text-warning me-1'></i> Requires Firmware v[MinFW]+ (Current: v[Current])`.
   - If unconfigured (no DT): `<i class='fa-solid fa-microchip text-secondary me-1'></i> New Device (Unconfigured)`.
   - Displays right-aligned RSSI badge below ID (e.g. `<span class='badge bg-secondary font-monospace'>-82 dBm</span>`, with amber warning if `< -115 dBm`).
3. **Row 3 (Timing & Actions)**:
   - Displays next message arrival countdown on bottom left (`Next ~X min` or `Due anytime` computed from `SM` port and elapsed time).
   - Displays matching icon-enabled action buttons on bottom right:
     - **Setup**: `<button class='btn btn-outline-success slowbutton'><i class='fa-solid fa-wrench me-1'></i> Setup</button>`
     - **Ignore**: `<button class='btn btn-outline-warning slowbutton'><i class='fa-solid fa-eye-slash me-1'></i> Ignore</button>`
     - **Check Online**: `<button class='btn btn-outline-primary slowbutton'><i class='fa-solid fa-cloud-arrow-down me-1'></i> Check Online</button>`
     - **Update Firmware**: `<button class='btn btn-outline-info slowbutton'><i class='fa-solid fa-rotate me-1'></i> Update Firmware</button>`
4. **Configured Isolation**:
   - Once a device is configured (`d->getType() == DeviceType::Configured`), all "New" status badges (RSSI, Via Router, countdown) are hidden, rendering only normal configured entity tiles.

---

### User Story 2 - Unified Server-Rendered Add Device Page (`/adddevice`) (Priority: P1)
As a farm network operator visiting `/adddevice`, I want the page to render unsetup devices directly using `sendDevice()` and smoothly update them in real time over SSE when packets arrive.

**Acceptance Scenarios**:
1. When `/adddevice` is loaded, unsetup devices are rendered using `sendDevice()`.
2. When a live packet arrives, `evtSource` receives `new_device` event, fetches `GET /api/device/card?id=<mac>`, and inserts/replaces `document.getElementById('device-card-' + mac).outerHTML` in real-time.
3. If an unrecognized device type is displayed, clicking **[Check Online]** downloads the template from GitHub, auto-configures the device, and keeps the user on `/adddevice` with the card updated to `New [Device Name]` and **[Setup]**.
4. If a firmware update is required, clicking **[Update Firmware]** navigates to `/settings?autoupdate=1` to run the OTA update and reboot to the home page.

---

### User Story 3 - Remote Router Discovery Candidates (Priority: P2)
As a farm network operator querying a remote router via `Query Router [Name]`, I want candidate nodes to be styled consistently with `sendDevice()`, featuring an **[Add via <Router>]** button and arrival countdown timer.

**Acceptance Scenarios**:
1. Router candidates display in clean card styling matching `sendDevice()`.
2. Clicking **[Add via Router]** queues the route command (`CA: 1`) and shows the pending modal.
3. When the subsequent routed packet arrives over the air, the candidate card is automatically replaced by the active `sendDevice()` card with the `Via [Router Name]` badge.
