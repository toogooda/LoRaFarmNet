# Feature Specification: Repeater PotentialList & QN Discovery Response Stream

**Feature Branch**: `029-repeater-potential-list`  
**Created**: 2026-08-26  
**Last Updated**: 2026-08-26  
**Status**: Draft  
**Input**: GitHub Issue #29 ("Repeater Node: Implement PotentialList with autonomous 2*SM+1 reliability pruning and QN query response stream")  
**Target Repository**: `toogooda/LoRaNetRepeaterNode` ([Nodes/LoraNodeRepeater](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater))

---

## Overview
Implement a dynamic in-memory `PotentialList` linked list and the `QN: 1` query response protocol on the LoRa Repeater node firmware. This completes the node/firmware half of the Gateway dynamic Add/Pair Device discovery system (Issue #9).

Because farm environments may contain 50–100+ neighboring field devices, `PotentialList` cannot be statically capped at a small number. Instead, it operates as a dynamic singly-linked list with runtime free-memory protection (`MIN_FREE_MEMORY`), reserving stack/heap headroom (1KB default on ATmega644PA, configurable for future STM32 40KB+ targets). It overhears unrouted field nodes, tracks signal strength (`RS`), heartbeat interval (`SM`), and elapsed minutes (`MM`), automatically prunes dead nodes after $2 \times \text{SM} + 1$ missed intervals or when memory is low, reports potential candidate count (`PN`) in its heartbeat, and streams candidate devices to the Gateway on demand via LoRa `QN: 1` queries.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Dynamic Ingestion & Overhearing of Unrouted Nodes (Priority: P1 🎯 MVP)

When unrouted field sensor nodes (e.g. water tanks, gate controllers, solar monitors) transmit periodic telemetry packets within RF range of the Repeater, the Repeater inspects the incoming frame. If the source MAC is not in its `routedNodes` list, the Repeater records the node into its dynamic `PotentialList` linked list along with its Device Type (`DT`), received RSSI (`RS`), sleep interval (`SM`), and sets elapsed time `minutesSinceLastMsg = 0`. If the node already exists in `PotentialList`, its RSSI, DT, SM, and elapsed time counter are updated in place without memory allocation.

**Why this priority**: Core overhearing capability required for remote discovery.

**Independent Test**:
- Power up a repeater node.
- Broadcast valid LoRa packets from unconfigured nodes (e.g. 50 distinct MACs).
- Verify the Repeater dynamically allocates and tracks all nodes in `PotentialList`.

**Acceptance Scenarios**:
1. **Given** a valid LoRa packet received from an unrouted MAC, **When** the MAC is not present in `routedNodes` and not in `PotentialList`, **Then** allocate a new `PotentialNode` and prepend/append to `PotentialList` with `minutesSinceLastMsg = 0`.
2. **Given** a MAC already present in `PotentialList`, **When** a new packet arrives from that MAC, **Then** update `lastRssi`, `deviceType`, `sleepMins` in place and reset `minutesSinceLastMsg = 0` (zero allocation).
3. **Given** a MAC already present in `routedNodes`, **When** a packet arrives, **Then** it is processed for repeating and NOT added to `PotentialList`.

---

### User Story 2 - Memory Safety Threshold & Low-RAM Auto-Purge (Priority: P1 🎯 MVP)

The repeater microcontroller has finite SRAM (4KB on ATmega644PA, 40KB+ on future STM32). To prevent stack collisions or Out-Of-Memory hangs:
- A configurable constant `MIN_FREE_MEMORY` (default `1024` bytes on AVR) defines the minimum reserved memory for stack/system operations.
- When a new unrouted node is heard, the repeater checks available free SRAM.
- If free memory is below `MIN_FREE_MEMORY`, the repeater triggers an aggressive purge: it finds and deletes the oldest / highest `minutesSinceLastMsg` entry from `PotentialList` before allocating the new node.
- If memory remains insufficient, the allocation is rejected safely without crashing.

**Why this priority**: Guarantees firmware stability under heavy RF traffic and enables seamless portability to STM32.

**Independent Test**:
- Simulate low free memory (< `MIN_FREE_MEMORY`).
- Receive a new unrouted packet.
- Verify the oldest potential entry is pruned to maintain the 1KB stack safety floor.

**Acceptance Scenarios**:
1. **Given** free SRAM is below `MIN_FREE_MEMORY`, **When** a new node needs to be allocated, **Then** the repeater purges the oldest entry in `PotentialList` to free heap memory.
2. **Given** `MIN_FREE_MEMORY` is defined as a preprocessor macro/constant, **When** compiling for ATmega644PA vs STM32, **Then** the threshold can be adjusted without code refactoring.

---

### User Story 3 - Autonomous 1-Minute Aging & 2*SM+1 Reliability Pruning (Priority: P1 🎯 MVP)

Every minute, a timer in `loop()` increments `minutesSinceLastMsg` for all nodes in `PotentialList`. When `minutesSinceLastMsg > (2 * SM + 1)` (meaning the node has missed two consecutive scheduled transmissions plus a 1-minute buffer), the Repeater unlinks and `delete`s the node from `PotentialList`.

**Why this priority**: Keeps the linked list bounded, eliminates phantom nodes from the Gateway UI, and ensures only actively transmitting nodes are suggested for pairing.

**Independent Test**:
- Insert an entry with `SM = 5` and `minutesSinceLastMsg = 10`.
- Advance the aging timer by 2 minutes (`minutesSinceLastMsg` becomes 12 > $2 \times 5 + 1 = 11$).
- Verify the node is deleted and memory is freed.

**Acceptance Scenarios**:
1. **Given** active entries in `PotentialList`, **When** 1 minute elapses, **Then** `minutesSinceLastMsg` increments by 1 for all entries.
2. **Given** an entry with `SM` interval, **When** `minutesSinceLastMsg > (2 * SM + 1)`, **Then** the node is unlinked and freed from memory.
3. **Given** an entry with missing or zero `SM`, **When** evaluated, **Then** default `SM = 15` (pruning threshold = 31 minutes) is applied.

---

### User Story 4 - Responding to Gateway `QN: 1` Query Streams (Priority: P1 🎯 MVP)

When the Gateway sends a LoRa packet addressed to the Repeater with `QN: 1`:
- If `PotentialList` is empty (`head == nullptr`), the Repeater replies immediately with 1 frame: `MI`, `QN: 255`, `QC: 0`, `CS`.
- If `PotentialList` has entries, the Repeater iterates through the list and streams Option A frames:
  - Each frame contains: `MI`, `M1` (bytes 0-1), `M2` (bytes 2-3), `M3` (bytes 4-5), `DT`, `RS` (absolute RSSI), `SM`, `MM` (`minutesSinceLastMsg`).
  - The final frame in the stream also contains `QN: 255` to signal completion to the Gateway.

**Why this priority**: Direct protocol interface for Gateway Add/Pair Device page (`/adddevice`) router query.

**Independent Test**:
- Send `QN: 1` from Gateway to Repeater.
- Verify empty list returns `QN: 255, QC: 0`.
- Verify populated list streams all entries with `QN: 255` on the last frame.

**Acceptance Scenarios**:
1. **Given** an empty `PotentialList`, **When** `QN: 1` is received, **Then** the Repeater sends `MI`, `QN: 255`, `QC: 0`, `CS`.
2. **Given** $N$ entries in `PotentialList`, **When** `QN: 1` is received, **Then** the Repeater sequentially sends $N$ packets with `M1..M3`, `DT`, `RS`, `SM`, `MM`, with `QN: 255` appended to the $N$-th packet.
3. **Given** streaming multiple packets, **When** transmitting, **Then** insert small pacing delays (50–100ms) between transmissions to avoid RF collisions.

---

### User Story 5 - Automatic Pruning on Route Assignment (`CA: 1`) (Priority: P2)

Route addition via LoRa command `CA: 1` (`I0..I5` + `RF`) is already implemented and stores the rule into `routedNodes[10]`. The new behavior required is: upon successfully accepting and saving the new route in `routedNodes`, the Repeater automatically checks `PotentialList` for the matching MAC address, unlinks it, and `delete`s the node (freeing its SRAM).

**Why this priority**: Keeps `PotentialList` clean and instantly frees SRAM when an overheard node transitions to an active routed node.

**Independent Test**:
- Add MAC `01:02:03:04:05:06` to `PotentialList`.
- Send `CA: 1` targeting that MAC.
- Verify `routedNodes` has the entry and `PotentialList` no longer contains that MAC.

**Acceptance Scenarios**:
1. **Given** a MAC in `PotentialList`, **When** `CA: 1` for that MAC is accepted by existing routing logic, **Then** the matching node in `PotentialList` is unlinked and deleted from memory.

---

### User Story 6 - Telemetry Heartbeat Port `PN` (Priority: P2)

Heartbeat telemetry ports `RE` (Routing Enabled) and `RN` (Routed Nodes Count) are already implemented. The new requirement is to also include `PN` (`Potential Nodes Count`) in the Repeater's periodic heartbeat packet, reporting the active count of items currently in `PotentialList`.

**Why this priority**: Allows the Gateway and operators to see on the dashboard whether a repeater is overhearing unconfigured devices.

**Independent Test**:
- With 3 items in `PotentialList`, trigger a heartbeat.
- Verify packet contains `PN: 3` alongside `RE` and `RN`.

**Acceptance Scenarios**:
1. **Given** $K$ items in `PotentialList`, **When** heartbeat is constructed, **Then** port `PN` with value $K$ is appended before `CS`.

---

## Edge Cases
- **Daisy-Chained Routers**: Packets repeated by upstream routers (`MR >= 100`) heard by this router will be cataloged in `PotentialList` if not already routed by this router.
- **Corrupted Packets / Bad Checksums**: Packets failing `messageValid()` MUST NOT be added to `PotentialList`.
- **Self Messages & Gateway Messages**: Packets originating from this repeater itself or from the Gateway MUST NOT be added to `PotentialList`.
- **Heap Fragmentation**: Using a fixed-size `struct PotentialNode` for all dynamic allocations ensures fixed-size heap blocks, eliminating heap fragmentation.

---

## Requirements *(mandatory)*

### Functional Requirements
- **FR-001**: Repeater MUST implement a dynamic singly-linked list (`PotentialNode* potentialHead`) for storing potential devices.
- **FR-002**: Each `PotentialNode` struct MUST store:
  - `uint8_t address[6]`
  - `uint16_t deviceType`
  - `uint8_t lastRssi` (absolute positive dBm value, e.g. 95)
  - `uint8_t sleepMins` (default 15 if not present in packet)
  - `uint8_t minutesSinceLastMsg`
  - `PotentialNode* next`
- **FR-003**: Repeater MUST define a memory safety threshold constant:
  - `#define MIN_FREE_MEMORY 1024` (reserved bytes for stack/system headroom).
- **FR-004**: Repeater MUST implement a `getFreeMemory()` utility function compatible with AVR (and portable for future STM32 architectures).
- **FR-005**: If `getFreeMemory() < MIN_FREE_MEMORY` when allocating a new entry, Repeater MUST purge the entry with the largest `minutesSinceLastMsg` before allocating.
- **FR-006**: Upon receiving any valid packet not addressed to this repeater and not in `routedNodes`, Repeater MUST insert or update the device in `PotentialList`.
- **FR-007**: Repeater MUST run an aging routine every 1 minute that increments `minutesSinceLastMsg` on all `PotentialList` entries and prunes entries where `minutesSinceLastMsg > (2 * sleepMins + 1)`.
- **FR-008**: Upon receiving `QN: 1` command addressed to this repeater:
  - If `PotentialList` is empty, transmit `MI`, `QN: 255`, `QC: 0`, `CS`.
  - If `PotentialList` has entries, stream Option A frames (`M1..M3`, `DT`, `RS`, `SM`, `MM`), appending `QN: 255` to the final frame.
- **FR-009**: Repeater MUST remove and delete a MAC from `PotentialList` whenever that MAC is added to `routedNodes` via `CA: 1`.
- **FR-010**: Repeater MUST include port `PN` (Potential Nodes count) in its periodic heartbeat packet.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes
- **SC-001**: System scales dynamically to 50–100+ overheards as long as free SRAM $\ge$ `MIN_FREE_MEMORY`.
- **SC-002**: Firmware maintains at least 1024 bytes of free SRAM at all times, preventing stack overflows and AVR crashes.
- **SC-003**: Pruning occurs deterministically when `minutesSinceLastMsg > 2*SM + 1` or during low-memory conditions.
- **SC-004**: Gateway `/api/pair/query_router` generates a complete, non-corrupted stream of `router_device` SSE events on the Gateway UI.
- **SC-005**: Firmware passes compilation check (`pio run`) in `LoraNodeRepeater` without warnings.
