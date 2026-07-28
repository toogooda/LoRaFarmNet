# Feasibility & Pairwise Diff Audit: LoRa Helper Consolidation

**Feature**: `001-consolidate-lora-helpers`  
**Date**: 2026-07-28  
**Author**: Antigravity Agent  

---

## Executive Summary

Audit of `Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, and `LoraMsg.cpp` across all 7 device repositories in `LoRaFarmNet`:
1. `Gateway/LoRaNetGateway`
2. `Nodes/LoraNodeButton`
3. `Nodes/LoraNodeDualGateController`
4. `Nodes/LoraNodeDualPIR`
5. `Nodes/LoraNodeRepeater`
6. `Nodes/LoraNodeVictron`
7. `Nodes/LoraNodeWaterTankLevel`

---

## Key File Discrepancies Identified

### 1. `Ra01S.h`
* **Gateway**: Standard driver header without timeout definitions.
* **Nodes**: Contains `SPI_Speed 500000`, `ON_AIR_TIMEOUT 1000`, and `BUSY_WAIT 5000` hardcoded macros at the top of the header.
* **Missing Guard**: Standard `#ifndef _RA01S_H` guard exists, but requires `#pragma once` standardization.

### 2. `LoRaHelper.h`
* **Gateway**: Has `#ifndef LoRaHelper_H` guard, declares `extern SX126x lora;`, defines RF parameters (frequency 915MHz, bandwidth 500kHz, spreading factor 11).
* **Nodes**: Missing include guards in node versions (e.g. `LoraNodeButton`). Includes `"Pinout.h"` directly (violates library decoupling—pinouts must be passed in setup or defined per node, not inside the core library).

### 3. `LoraMsg.h` & `LoraMsg.cpp`
* **Method Access Divergence**:
  * Gateway version contains public `uint8_t getFromByte(const uint8_t byteNumber);` and private `uint16_t getMessageID();`.
  * Node versions contain public `uint16_t getMessageID();` and lack `getFromByte()`.
* **Consolidation Solution**: Make both `getFromByte()` and `getMessageID()` public in the unified `LoraMsg.h`.

---

## Consolidation & Architecture Proposal

Create a new private GitHub repository: **`toogooda/LoRaFarmNetCore`** (or `LoRaFarmNetLibrary`).

### Repository Structure
```text
LoRaFarmNetCore/
├── library.json
├── README.md
├── src/
│   ├── Ra01S.h
│   ├── Ra01S.cpp
│   ├── LoRaHelper.h
│   ├── LoRaHelper.cpp
│   ├── LoraMsg.h
│   └── LoraMsg.cpp
```

### Invariants & Enhancements
1. `#pragma once` at the top of all header files.
2. Remove `"Pinout.h"` from `LoRaHelper.h` to keep library completely decoupled from target hardware pins.
3. Unify `LoraMsg.h` public interface to include both `getFromByte()` and `getMessageID()`.
