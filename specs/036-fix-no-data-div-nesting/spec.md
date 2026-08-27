# Feature Specification: Fix Div Nesting and Layout for Devices with No Data

**Feature Branch**: `036-fix-no-data-div-nesting`

**Created**: 2026-08-27

**Status**: Draft

**Input**: User description: "speckit specify issue #36: Devices in mode 'No data received yet' not rendering correctly in Mini view"

## Background & Problem Statement
When the Gateway boots or restarts (or when newly paired devices have not yet transmitted their first telemetry frame), devices exist in memory with `hasData = false` ("No data received yet").

In `sendDevice()` ([`WebHelper.h`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/src/WebHelper.h)), the `!s || !d->getHasData()` branch and `DeviceType::New` branch prematurely closed the outer container `<div>` with an extra closing tag (`</div></div>` and `</div></div></div>`), while a trailing `response->print("</div>");` unconditionally executed at the end of `sendDevice()`.

### Live Hardware Reproduction Trace (from `GET http://192.168.68.104/`)
When restarting the Gateway running test build with 2 routers (`Router Top` ID:100, `Test Router` ID:101) in Category 7 (`Repeater` set to Mini view):
```html
<div class='card mb-4 border-0 shadow-sm' style='max-width: 600px; margin: 0 auto;'>
  <div class='card-header d-flex justify-content-between align-items-center bg-dark text-white'>
    <h5 class='mb-0'><i class='fas me-2'>&#xf012;</i>Repeater</h5>
    <div class='form-check form-switch'>
      <input class='form-check-input view-mode-toggle' type='checkbox' data-catid='7' id='toggle-switch-7' style='cursor: pointer;'>
      <label class='form-check-label text-muted view-mode-label-7' for='toggle-switch-7' style='font-size: 0.85rem; cursor: pointer;'>Mini</label>
    </div>
  </div>
  <div class='card-body bg-light' id='cat-card-body-7'>
    <div class='full-view-container-7 d-none'>
      <!-- Router Top (hasData = false): sendDevice emits an extra </div> -->
      <div class='container border shadow-lg rounded'...>...</div>
    </div> <!-- full-view-container-7 prematurely closed by Router Top! -->
    <br>
    <!-- Test Router is now outside full-view-container-7 and remains visible in Mini view! -->
    <div class='container border shadow-lg rounded'...>...</div>
  </div> <!-- cat-card-body-7 prematurely closed by Test Router! -->
  <br>
</div> <!-- Card container prematurely closed! -->

<!-- mini-view-container-7 is stranded completely OUTSIDE the category card! -->
<div class='mini-view-container-7 '>
  <div class='d-flex flex-wrap gap-3 justify-content-start' style='padding: 10px 0;'>
    <a href='/device?deviceid=fc0fe7142e54'...>...</a>
    <a href='/device?deviceid=fc0fe7147df9'...>...</a>
  </div>
</div>
```

This causes two severe visual defects:
1. When Category 7 is set to Mini view, the second full router card (`Test Router`) is visible because it escaped `.full-view-container-7 d-none`.
2. The mini view icon badges are rendered outside and below the white card container.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Clean Layout for Devices in "No data received yet" State (Priority: P1)

As a farm network operator, when I restart the Gateway or view devices that have not yet sent data, each category card on the Home Page (`/`) must maintain a clean, unbroken visual container in both Full view and Mini view.

**Why this priority**: Layout corruption displaces device cards across categories and breaks the toggle switches.

**Independent Test**:
Flash test firmware or reboot Gateway so all devices reset to `hasData = false`. Navigate to `/` and inspect the DOM:
- All device cards are strictly contained within their respective `<div class='card-body'>` and view containers.
- Categories with multiple devices (such as Routers or Water Tanks) display cleanly without displaced elements.

**Acceptance Scenarios**:
1. **Given** a Gateway with 1 or more devices in `hasData = false` state, **When** `/` is loaded in Full view, **Then** all device cards are rendered inside `<div class='full-view-container-[catId]'>` with 0 unclosed or extra `<div>` tags.
2. **Given** a category set to Mini view with devices in `hasData = false` state, **When** `/` is loaded, **Then** the orange clock icon badges (`&#xf017;`) render within `<div class='mini-view-container-[catId]'>` inside the category card body.

---

### User Story 2 - Smooth View Toggle Between Full and Mini Modes (Priority: P2)

As a farm network operator, when I toggle a category between Full and Mini views, the category card body must dynamically show/hide the correct container without spilling elements onto the page.

**Why this priority**: Guarantees responsive UI behavior and prevents client-side rendering glitches.

**Independent Test**:
Click the view mode switch (Mini/Full) on category cards containing multiple devices with and without data:
- Toggling immediately swaps visibility between `.full-view-container-[id]` and `.mini-view-container-[id]`.
- No orphan badges or misplaced cards appear outside the category container.

**Acceptance Scenarios**:
1. **Given** a category card with multiple established devices, **When** the operator toggles the switch from Mini to Full, **Then** `.mini-view-container` is hidden (`d-none`) and `.full-view-container` is displayed cleanly inside the card body.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: `sendDevice()` in `WebHelper.h` MUST maintain strict `<div>` balancing where the outer `<div class='container...'>` opened at the start of `sendDevice()` is closed exactly once at the end of the function.
- **FR-002**: The `d->getType() == DeviceType::New` branch in `sendDevice()` MUST only close its own internal sub-containers (`py-2`, `mb-2`, `d-flex`), delegating the outer container closure to the function's trailing statement.
- **FR-003**: The `!s || !d->getHasData()` branch in `sendDevice()` MUST only close its own `position:relative` wrapper (`</div>`), delegating the outer container closure to the function's trailing statement.
- **FR-004**: The established data `else` branch in `sendDevice()` MUST close its `row no-pad` container (`</div>`), delegating the outer container closure to the function's trailing statement.
- **FR-005**: All device elements rendered via `HomePageChunkedResponse.cpp` MUST remain strictly nested inside their respective category container cards.

---

## Edge Cases

- **Multiple Pending Devices in Single Category**: If a category contains 3+ routers or nodes all in `hasData = false` state, DOM depth must remain balanced with zero leaking `</div>` tags.
- **Mixed State Category**: A category containing 1 `DeviceType::New`, 1 `hasData = false`, and 2 active devices must render each device in its proper full or mini sub-container without layout breakage.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Inspecting the rendered HTML of `/` with multiple devices in `hasData = false` reveals 0 mismatched `<div>` / `</div>` tags in the document tree.
- **SC-002**: Automated regression suite (`test_gateway_harness.py`) passes 100% with valid HTTP 200 responses and properly structured HTML.
- **SC-003**: Visual inspection confirms all category cards render at equal alignment with zero displaced elements.
