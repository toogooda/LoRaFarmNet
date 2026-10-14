# Implementation Tasks: 5-Message Circular Deduplication Buffer for Devices

**Branch**: `038-device-recent-msg-buffer` | **Spec**: [specs/038-device-recent-msg-buffer/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/038-device-recent-msg-buffer/spec.md) | **Plan**: [specs/038-device-recent-msg-buffer/plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/038-device-recent-msg-buffer/plan.md)

## Tasks

- [x] **Task 1: Setup Feature Branch**
  - Create and switch to feature branch `038-device-recent-msg-buffer` in `Gateway/LoRaNetGateway`.

- [x] **Task 2: Add 5-Message Buffer to `Device` Class**
  - Update `Gateway/LoRaNetGateway/src/Device.h`:
    - Declare `RECENT_MSG_CACHE_SIZE = 5`.
    - Declare `recentMsgIds[5]` and `recentMsgIdx`.
    - Declare `isRecentMsgId()`, `recordMsgId()`, and `clearRecentMsgIds()`.
  - Update `Gateway/LoRaNetGateway/src/Device.cpp`:
    - Implement buffer initialization in constructor and `clearRecentMsgIds()`.
    - Implement `isRecentMsgId(uint16_t msgId)`.
    - Implement `recordMsgId(uint16_t msgId)`.

- [x] **Task 3: Update `FarmNetwork.cpp` and `SDHelper.h`**
  - Update `FarmNetwork.cpp`:
    - Replace `newMI <= lastMI` check with `d->isRecentMsgId(newMI)`.
    - Record `newMI` on accepted messages (`d->recordMsgId(newMI)`).
    - Maintain `isRoutedUpgrade` logic.
  - Update `SDHelper.h`:
    - Clear recent message IDs on boot in `loadNetwork()`.

- [x] **Task 4: Build Verification & Regression Testing**
  - Compile test build: `pio run -e gateway-test`.
  - Flash firmware and verify dynamic IP discovery: `192.168.68.109`.
  - Run full regression suite (`test_gateway_harness.py`): 12/12 tests passed.
  - Run ring buffer verification suite: normal sequential, duplicate rejection, reboot acceptance (5061 -> 40), gap acceptance (40 -> 44), duplicate 44 rejection, and 5-slot rotation all passed 100%.

- [x] **Task 5: Mandatory Manual Testing Pause & PR**
  - Inform user that firmware is flashed and automated tests pass.
  - User confirmed testing and requested commit, PR, and release.
  - Committed changes, pushed branch `038-device-recent-msg-buffer`, and opened PR #73.
  - Published release `v1.7.3` to `toogooda/LoRaFarmNet`.
