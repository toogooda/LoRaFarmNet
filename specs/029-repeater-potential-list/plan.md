# Implementation Plan: Repeater PotentialList & QN Discovery Response Stream

**Branch**: `029-repeater-potential-list` | **Date**: 2026-08-26 | **Spec**: [specs/029-repeater-potential-list/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/029-repeater-potential-list/spec.md)

**Input**: Feature specification from [specs/029-repeater-potential-list/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/029-repeater-potential-list/spec.md) based on GitHub Issue #29.

---

## Summary
Refactor `LoRaNetRepeaterNode` firmware ([Nodes/LoraNodeRepeater/src/main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)) to maintain a dynamic singly-linked list `PotentialList` of overheard unrouted field devices. Integrate a runtime free SRAM safety check (`MIN_FREE_MEMORY = 1024` bytes) with low-RAM auto-purge of the oldest entry, autonomous 1-minute aging and $2 \times \text{SM} + 1$ reliability pruning, LoRa `QN: 1` command response streaming with Option A frames, automatic deletion from `PotentialList` when a node is added to `routedNodes` via `CA: 1`, and reporting the active potential node count in heartbeat telemetry port `PN`.

---

## Technical Context

- **Target MCU**: ATmega644PA (8MHz external oscillator, 4KB SRAM, 64KB Flash)
- **Framework**: Arduino / Atmel AVR via PlatformIO
- **Core Libraries**: `LoRaNetLibrary` (`LoRaHelper.h`, `LoraMsg.h`, `SX126x.h`), `nano64DeepSleep.h`
- **Memory Management**:
  - Struct `PotentialNode` allocated on heap (~13 bytes per node with pointer).
  - `#define MIN_FREE_MEMORY 1024` reserves 1KB SRAM safety headroom for stack and FreeRTOS/interrupt operations on AVR.
  - Scale: Supports 50–100+ overheards dynamically, self-throttling when free memory approaches 1024 bytes.
- **Timing & Loop Mechanics**: `loop()` executes 500ms light sleep ticks. 120 ticks = 1 minute ticker.

---

## Proposed Changes

### Component 1: Repeater Node Firmware (`Nodes/LoraNodeRepeater`)

#### [MODIFY] [main.cpp](file:///c:/Users/USER/Projects/LoRaFarmNet/Nodes/LoraNodeRepeater/src/main.cpp)

1. **Memory Safety & `PotentialNode` Linked List Definition**:
   ```cpp
   #define MIN_FREE_MEMORY 1024

   int getFreeMemory() {
   #if defined(__AVR__)
     extern int __heap_start, *__brkval;
     int v;
     return (int)&v - (__brkval == 0 ? (int)&__heap_start : (int)__brkval);
   #else
     return 4096; // Future STM32 / ARM fallback
   #endif
   }

   struct PotentialNode {
     uint8_t address[6];
     uint16_t deviceType;
     uint8_t lastRssi;          // Absolute dBm value (e.g. 95)
     uint8_t sleepMins;         // Expected heartbeat interval (default 15)
     uint8_t minutesSinceLastMsg;
     PotentialNode* next = nullptr;
   };

   PotentialNode* potentialHead = nullptr;
   ```

2. **Linked List Helper Functions**:
   - `uint16_t getPotentialNodesCount()`: Traverses `potentialHead` and returns count.
   - `void purgeOldestPotentialNode()`: Unlinks and `delete`s the node with the largest `minutesSinceLastMsg`.
   - `void removePotentialNode(const uint8_t* addr)`: Searches by 6-byte MAC, unlinks, and `delete`s the node.
   - `void recordPotentialNode(LoraMsg* lMsg, uint8_t rssi)`:
     - Extracts source MAC (`msgBytes + 6`).
     - Extracts `DT` (defaults to 0), `SM` (defaults to 15).
     - Searches `potentialHead`:
       - If found: updates `lastRssi`, `deviceType`, `sleepMins`, and resets `minutesSinceLastMsg = 0` (zero allocation).
       - If not found:
         - Checks if `getFreeMemory() < MIN_FREE_MEMORY`; if so, calls `purgeOldestPotentialNode()`.
         - Allocates `new PotentialNode`, populates fields, and links to `potentialHead`.

3. **1-Minute Aging & Pruning**:
   - In `loop()`, every 120 ticks (1 minute):
     - Traverse `potentialHead`:
       - Increment `minutesSinceLastMsg++`.
       - If `minutesSinceLastMsg > (2 * sleepMins + 1)`, unlink and `delete` the node.

4. **`QN: 1` Gateway Query Response Stream**:
   - In `lMsg->isForMe(fromAddress)` command processing:
     - Detect `getPortVal(lMsg, "QN", qnVal) && qnVal == 1`:
       - Count $N =$ `getPotentialNodesCount()`.
       - If $N == 0$:
         - Send single ACK: `MI`, `QN: 255`, `QC: 0`, `CS`.
       - If $N > 0$:
         - Iterate `potentialHead`:
           - Build Option A frame: `MI`, `M1`, `M2`, `M3`, `DT`, `RS`, `SM`, `MM`.
           - If this is the last entry in the list, append `QN: 255`.
           - Calculate `CS` and transmit with `sendMsg()`.
           - Add 50ms pacing delay between transmissions.

5. **`CA: 1` Route Addition Auto-Pruning**:
   - In existing `CA: 1` handling, after `routedNodes[slot]` is updated:
     - Call `removePotentialNode(target)` to unlink and free that MAC from `PotentialList`.

6. **Heartbeat Port `PN`**:
   - In `loop()` heartbeat generator:
     - Append `PortValue pnVal = { { 'P', 'N' }, getPotentialNodesCount() };` before `CS`.

---

## Verification Plan

### Automated Tests
1. **Compilation Check**:
   ```powershell
   & "C:\Users\USER\.platformio\penv\Scripts\platformio.exe" run -d "c:\Users\USER\Projects\LoRaFarmNet\Nodes\LoraNodeRepeater" --environment ATmega644P
   ```

### Manual Verification
1. Flash repeater node using AVRISP mkII (`pio run -t upload`).
2. Monitor serial output on configured serial port (`monitor_port` in `platformio.ini`) at 115200 baud.
3. Transmit packets from unrouted node; verify `PotentialList` logs ingestion and updates.
4. Verify Gateway `/adddevice` router query discovers the remote node.
5. Close GitHub Issue #29 upon successful deployment and merge.
