# Specification: 5-Message Circular Deduplication Buffer for Devices

## Context & Motivation
Currently, the LoRaNetGateway uses a strictly monotonic sequence check in `FarmNetwork.cpp`:
```cpp
bool isDuplicate = (d->getHasData() && newMI <= lastMI && newMI != 0 && !overdue);
```
Where `lastMI` is retrieved from `d->getLastMsgId()`, which inspects the device's `"MI"` port value.

### Problems with Strict Monotonic Sequence Checking
1. **Permanent Lockout on Node Reboots**:
   - Field nodes keep `LoraMsg::messageCounter` in volatile RAM, which resets to 0 on reboot or power loss/brownout.
   - The Gateway persists sensor port values (including `MI`) to `/lfm/data/network.dat` on SD card (e.g. `lastMI = 5061`).
   - If a rebooted node begins transmitting (e.g. Message ID 1, 2, 40, 44), the Gateway rejects every packet because `newMI <= 5061`.
   - The only recovery mechanism currently is either `newMI == 0` (which is easily lost over lossy LoRa links) or `overdue`.
2. **Overdue Calculation Lockout**:
   - When a node reports `SM: 999` (packaging mode sentinel), `overdue` is computed as `diffSeconds > (999 * 60 * 2) = 33.3 hours`. The Gateway refuses to accept out-of-order/reboot packets for over 33 hours.
3. **Repeater / Router Bursts**:
   - Routers delay repeated messages by 10 to 20 seconds. If a node sends a burst of packets (e.g. 101, 102, 103), the direct packets arrive, followed 10 seconds later by repeated copies from the router.
   - The Gateway must drop these repeated copies to prevent double-processing deltas, alerts, and MQTT updates.

---

## Requirements

### Functional Requirements
* **FR-001**: Each `Device` in `FarmNetwork` MUST maintain a circular buffer of the 5 most recent unique `MessageID` values (`recentMsgIds[5]`).
* **FR-002**: A packet MUST be recognized as duplicate if its `MessageID` (`newMI != 0`) matches any non-zero entry in the device's 5-message recent buffer.
* **FR-003**: If a packet is identified as a duplicate, but has `newMR >= 100` and the device's current `MR < 100` (`isRoutedUpgrade`), the Gateway MUST apply the routed upgrade (updating `MR`, `RS`, `NR`, `timeLastMsg`, and SSE) without triggering deltas or MQTT.
* **FR-004**: If a packet is NOT a duplicate:
  * The Gateway MUST record `newMI` into the device's circular buffer (`recordMsgId(newMI)`).
  * The Gateway MUST update device sensor values, process deltas, and publish MQTT messages normally.
* **FR-005**: If a node reboots and transmits any message ID (e.g. 1, 2, 40, 44), and that ID is not in the recent 5-slot buffer, the Gateway MUST accept it immediately without requiring Message ID 0 or waiting for overdue timeout.
* **FR-006**: During Gateway boot (`loadNetwork()`), each device's recent message buffer MUST be cleared/reset to 0.

### Non-Functional Requirements
* **NFR-001**: Memory overhead per `Device` MUST not exceed 16 bytes.
* **NFR-002**: All automated regression tests in `scripts/test_gateway_harness.py` MUST pass.
