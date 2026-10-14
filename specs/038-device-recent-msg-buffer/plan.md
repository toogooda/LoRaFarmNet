# Implementation Plan: 5-Message Circular Deduplication Buffer for Devices

## Target Architecture

### 1. `Device` Class Extensions
In `Gateway/LoRaNetGateway/src/Device.h`:
* Define `static const uint8_t RECENT_MSG_CACHE_SIZE = 5;`
* Add `uint16_t recentMsgIds[RECENT_MSG_CACHE_SIZE];`
* Add `uint8_t recentMsgIdx;`
* Add helper methods:
  * `bool isRecentMsgId(uint16_t msgId) const;`
  * `void recordMsgId(uint16_t msgId);`
  * `void clearRecentMsgIds();`

In `Gateway/LoRaNetGateway/src/Device.cpp`:
* In constructor: initialize `recentMsgIds` to 0, `recentMsgIdx = 0`.
* In `clearRecentMsgIds()`: reset `recentMsgIds` to 0, `recentMsgIdx = 0`.
* In `isRecentMsgId(uint16_t msgId)`:
  * If `msgId == 0`, return `false`.
  * Loop `i = 0` to `RECENT_MSG_CACHE_SIZE - 1`: if `recentMsgIds[i] == msgId`, return `true`.
  * Return `false`.
* In `recordMsgId(uint16_t msgId)`:
  * If `msgId == 0`, return early (do not poison buffer with 0).
  * Check if already recorded in buffer; if not already present:
    * `recentMsgIds[recentMsgIdx] = msgId;`
    * `recentMsgIdx = (recentMsgIdx + 1) % RECENT_MSG_CACHE_SIZE;`

### 2. `FarmNetwork::processNewMessage()` Deduplication Logic
In `Gateway/LoRaNetGateway/src/FarmNetwork.cpp`:
* In `processNewMessage()`:
  * Extract `newMI = msg.getPortValue(0).value;`
  * Evaluate duplicate status:
    ```cpp
    bool isDuplicate = (d->getHasData() && d->isRecentMsgId(newMI));
    ```
  * If `isDuplicate`:
    * If `isRoutedUpgrade`: apply router metadata update (`MR`, `RS`, `NR`, `timeLastMsg`, dynamic SSE) and return.
    * Else: print `Serial.print(" - Duplicate or out-of-order ignoring");` and return.
  * If NOT duplicate:
    * For existing device: record `newMI` into buffer via `d->recordMsgId(newMI);`.
    * For new device: record `newMI` into buffer via `d->recordMsgId(newMI);`.

### 3. Gateway Startup / SD Card Invariants
In `Gateway/LoRaNetGateway/src/SDHelper.h`:
* In `loadNetwork()`:
  * When iterating `d_refresh`, invoke `d_refresh->clearRecentMsgIds();` along with `d_refresh->setHasData(false);`.
  * This guarantees a clean deduplication state on boot.

---

## Verification Strategy
1. Build verification via `pio run -e gateway-test`.
2. Regression test suite execution: `python scripts/test_gateway_harness.py --ip <ip> --run-suite`.
3. Manual verification pause before pushing or PR.
