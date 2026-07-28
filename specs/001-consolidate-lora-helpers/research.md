# Feasibility & Pairwise Diff Audit: Single Header & Cpp Library Consolidation

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

Goal: Combine and simplify all 4 legacy files into **ONE Header file (`.h`) and ONE Implementation file (`.cpp`)** with `#pragma once` include guards.

---

## Key File Discrepancies Identified

### 1. `Ra01S.h`
* **Gateway**: Standard driver header without node timeout definitions.
* **Nodes**: Contains `SPI_Speed 500000`, `ON_AIR_TIMEOUT 1000`, and `BUSY_WAIT 5000` hardcoded macros at the top.
* **Include Guard**: Uses legacy `#ifndef _RA01S_H`. Standardize with `#pragma once`.

### 2. `LoRaHelper.h`
* **Gateway**: Has `#ifndef LoRaHelper_H` guard, declares `extern SX126x lora;`, defines RF parameters (915MHz, 500kHz BW, SF 11).
* **Nodes**: Missing include guards in node versions (e.g. `LoraNodeButton`). Includes `"Pinout.h"` directly (violates library decoupling—pinouts must be passed in setup or defined per node, not inside the core library).

### 3. `LoraMsg.h` & `LoraMsg.cpp`
* **Method Access Divergence**:
  * Gateway version contains public `uint8_t getFromByte(const uint8_t byteNumber);` and private `uint16_t getMessageID();`.
  * Node versions contain public `uint16_t getMessageID();` and lack `getFromByte()`.
* **Consolidation Solution**: Make both `getFromByte()` and `getMessageID()` public in the unified header.

---

## Single Header & Cpp Consolidation Architecture

Instead of maintaining 4 separate files per repo (`Ra01S.h`, `LoRaHelper.h`, `LoraMsg.h`, `LoraMsg.cpp`), combine them into a single library pair:

* **Header**: `LoRaHelper.h` (or `LoRaFarmNet.h`)
  - `#pragma once`
  - SX126x register constants & SPI commands (from `Ra01S.h`)
  - Default RF configuration macros (`RF_FREQUENCY`, `TX_OUTPUT_POWER`, `LORA_BANDWIDTH`, etc.)
  - `PortValue` struct definition
  - `SX126x` transceiver class declaration
  - `LoraMsg` message serialization class declaration

* **Implementation**: `LoRaHelper.cpp` (or `LoRaFarmNet.cpp`)
  - `#include "LoRaHelper.h"`
  - `SX126x` method implementations (SPI transfer, init, TX, RX, status check)
  - `LoraMsg` method implementations (addPortValue, encryptMessage, decryptMessage, CRC, etc.)

---

## Benefits of Single Pair Consolidation

1. **Simplicity**: 4 separate files reduced to 1 `.h` and 1 `.cpp`.
2. **Include Safety**: Single `#pragma once` prevents duplicate symbol compilation errors across PlatformIO projects.
3. **Decoupled Pinouts**: Removes direct `"Pinout.h"` inclusion from core library.
4. **Unified API**: Exposes both `getFromByte()` and `getMessageID()` globally.
