# Tasks: Repeater PotentialList & QN Discovery Response Stream

**Input**: Design documents from `specs/029-repeater-potential-list/`  
**Prerequisites**: [spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/029-repeater-potential-list/spec.md), [plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/029-repeater-potential-list/plan.md)  
**Target Repository**: `toogooda/LoRaNetRepeaterNode` ([Nodes/LoraNodeRepeater](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater))

---

## Phase 1: Setup & Data Structures

- [x] T001 Define `MIN_FREE_MEMORY 1024` and implement `getFreeMemory()` in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)
- [x] T002 Define `struct PotentialNode` and singly-linked list head `PotentialNode* potentialHead = nullptr` in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)
- [x] T003 Implement linked list management helpers (`getPotentialNodesCount()`, `purgeOldestPotentialNode()`, `removePotentialNode()`) in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)

---

## Phase 2: Dynamic Ingestion & Free RAM Protection (User Stories 1 & 2)

- [x] T004 Implement `recordPotentialNode(LoraMsg* lMsg, uint8_t rssi)` to parse `DT`, `SM`, update existing nodes or allocate new nodes with `MIN_FREE_MEMORY` safety check in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)
- [x] T005 Hook `recordPotentialNode` into incoming packet reception path for unrouted non-self packets in `loop()` in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)

---

## Phase 3: Autonomous Aging & Pruning (User Story 3)

- [x] T006 Implement 1-minute ticker (120 light sleep ticks) in `loop()` to increment `minutesSinceLastMsg++` across `potentialHead` in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)
- [x] T007 Implement auto-pruning logic when `minutesSinceLastMsg > (2 * sleepMins + 1)` in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)

---

## Phase 4: QN Gateway Query Stream & CA Route Promotion (User Stories 4 & 5)

- [x] T008 Implement `QN: 1` command response handler in `loop()` (empty list `QN: 255, QC: 0` vs Option A stream with final `QN: 255`) in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)
- [x] T009 Add `removePotentialNode(target)` call inside existing `CA: 1` route addition handling in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)

---

## Phase 5: Heartbeat Port PN & Verification (User Story 6)

- [x] T010 Add port `PN` (`Potential Nodes Count`) to periodic heartbeat generation in [Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)
- [x] T011 Run PlatformIO build verification (`pio run --environment ATmega644P`)
- [ ] T012 Perform manual hardware testing with Gateway discovery page (`/adddevice`)
- [ ] T013 Close GitHub Issue #29 upon successful merge
