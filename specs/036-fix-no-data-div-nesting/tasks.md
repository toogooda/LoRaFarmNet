# Implementation Tasks: Fix Div Nesting and Layout for Devices with No Data

**Branch**: `036-fix-no-data-div-nesting` | **Spec**: [specs/036-fix-no-data-div-nesting/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/036-fix-no-data-div-nesting/spec.md) | **Plan**: [specs/036-fix-no-data-div-nesting/plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/036-fix-no-data-div-nesting/plan.md)

## Tasks

- [x] **Task 1: Setup Feature Branch & GitHub Issue Status**
  - Create feature branch `036-fix-no-data-div-nesting` in `Gateway/LoRaNetGateway`.
  - Post "Started" comment on GitHub Issue #36.

- [x] **Task 2: Fix `<div>` Balancing in `sendDevice()`**
  - In `Gateway/LoRaNetGateway/src/WebHelper.h`:
    - In `DeviceType::New` sub-branches, change closing tags from `</div></div></div>` to `</div></div>`.
    - In `!s || !d->getHasData()` branch, change closing tags from `</div></div>` to `</div>`.
    - Retain trailing `response->print("</div>");` at line 1073.

- [x] **Task 3: Build Verification & Hardware Upload**
  - Run `pio run -e gateway-test` to compile.
  - Run `pio run -e gateway-test -t upload` to flash Gateway hardware on `COM3`.

- [x] **Task 4: Automated & Live Surface Verification**
  - Fetch `http://192.168.68.104/` and verify live DOM structure.
  - Run regression test suite: `python scripts/test_gateway_harness.py --ip 192.168.68.104 --run-suite`.

- [ ] **Task 5: Mandatory Manual Testing Pause & PR**
  - Pause for user manual browser verification.
  - Once approved, push branch and create Pull Request on `toogooda/LoRaNetGateway`.
