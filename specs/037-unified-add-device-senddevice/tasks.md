# Implementation Tasks: Unified Add Device Experience & Enhanced `sendDevice()` Not-Setup View

**Branch**: `037-unified-add-device-senddevice` | **Spec**: [specs/037-unified-add-device-senddevice/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/037-unified-add-device-senddevice/spec.md) | **Plan**: [specs/037-unified-add-device-senddevice/plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/037-unified-add-device-senddevice/plan.md)

## Tasks

- [x] **Task 1: Setup Feature Branch**
  - Create and switch to feature branch `037-unified-add-device-senddevice` in `Gateway/LoRaNetGateway`.

- [x] **Task 2: Enhance `sendDevice()` in `WebHelper.h`**
  - Add `id='device-card-%s'` to the container `<div>`.
  - In `DeviceType::New` view:
    - Add `Via [Router Name]` blue badge on Row 1 when `MR >= 100`.
    - Add Category Icon + `New [Device Name]` and right-aligned RSSI badge on Row 2.
    - Add Next message countdown badge on bottom left of Row 3.
    - Add FontAwesome icons to `[Setup]` (`fa-wrench`) and `[Ignore]` (`fa-eye-slash`) buttons.

- [x] **Task 3: Refactor Add Device Page (`/adddevice`) to use `sendDevice()`**
  - Server-render unsetup devices in C++ using `sendDevice()`.
  - Add `GET /api/device/card?id=<mac>` endpoint to stream single `sendDevice()` card snippet.
  - Update SSE `new_device` handler to fetch snippet and update `device-card-[mac]` in-place.
  - Add `redirect` param support to `checkonline=1` so users stay on `/adddevice`.
  - Harmonize Remote Router Candidates section to match `sendDevice()` styling.

- [x] **Task 4: Build Verification & Regression Testing**
  - Run PlatformIO compilation: `pio run -e gateway-test`.
  - Run regression test suite: `python scripts/test_gateway_harness.py --ip <ip_address> --run-suite`.
  - Run SSE test suite: `python scripts/test_pair_sse.py <ip_address>`.

- [ ] **Task 5: Mandatory Manual Testing Pause & PR**
  - Pause for user manual browser/hardware testing.
  - Upon approval, ask for permission to push and create Pull Request on `toogooda/LoRaNetGateway`.
