# Autonomous Testing & Hardware Verification Rules

This document outlines autonomous testing workflows across the LoRaFarmNet ecosystem.

---

## 1. Gateway Testing Workflow
The LoRa Gateway is the central coordinator with WiFi/Ethernet, web server, and REST test APIs.
For detailed Gateway test harness rules, dynamic serial IP discovery, and dual-surface verification, refer to:
👉 **[`Gateway/LoRaNetGateway/.agent/rules/ai_testing_workflow.md`](file:///c:/Users/USER/Projects/LoRaFarmNet/Gateway/LoRaNetGateway/.agent/rules/ai_testing_workflow.md)**

---

## 2. Field Node & Repeater Testing Workflow
Field nodes (ATmega644PA / AVR) communicate solely over sub-GHz LoRa radio without direct WiFi/AP interfaces.
- **Build Verification**: Compile with `pio run` in the respective node directory (e.g. `Nodes/LoraNodeRepeater`).
- **Flash via ISP**: Upload using configured programmer (e.g., AVRISP mkII with `-B 10` or `-B 5`).
- **End-to-End Simulation**: Test node protocol interactions by injecting equivalent binary LoRa frames into the Gateway Test Harness (`POST /api/test/message`).

---

## 3. Mandatory Human Manual Testing Gate (Global Rule)
Across all repos:
1. AI builds and uploads firmware.
2. AI executes automated regression scripts.
3. AI **MUST STOP and PAUSE** for human verification on live devices before committing, pushing, or creating Pull Requests.
