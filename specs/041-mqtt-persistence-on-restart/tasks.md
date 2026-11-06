# Implementation Tasks: 041 - MQTT Settings Persistence Across Restarts

**Branch**: `041-mqtt-persistence-on-restart` | **Spec**: [specs/041-mqtt-persistence-on-restart/spec.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/041-mqtt-persistence-on-restart/spec.md) | **Plan**: [specs/041-mqtt-persistence-on-restart/plan.md](file:///c:/Users/USER/Projects/LoRaFarmNet/specs/041-mqtt-persistence-on-restart/plan.md)

## Tasks

- [x] **Task 1: Setup Feature Branch**
  - Create and switch to feature branch `041-mqtt-persistence-on-restart` in `Gateway/LoRaNetGateway` and `LoRaFarmNet`.

- [x] **Task 2: Code Changes Confirmation & Implementation**
  - Explain proposed modifications and obtain user confirmation before touching source code.
  - In `Gateway/LoRaNetGateway/src/WebHelper.h`:
    - Add `saveNetwork()` call when global MQTT toggle changes in `sendSettingsPage()`.
    - Add `saveNetwork()` call when MQTT server IP / port changes in `sendSettingsPage()`.
    - Add `saveNetwork()` call when device-level MQTT toggle changes in `sendDevicePage()`.
  - In `Gateway/LoRaNetGateway/src/SDHelper.h`:
    - Fix `curD->setUseMQTT((bool)atoi(v))` in `proccessItem()`.

- [x] **Task 3: Build Verification**
  - Compile with `pio run -e gateway-test` in `Gateway/LoRaNetGateway`.

- [x] **Task 4: Hardware Upload & Live Verification**
  - Upload test build to test gateway: `pio run -e gateway-test -t upload`.
  - Assert `/settings?usemqtt=0` immediately updates `network.dat` to `<gusemqtt>0</gusemqtt>`.
  - Assert `/settings?usemqtt=1` updates `network.dat` to `<gusemqtt>1</gusemqtt>`.
  - Run full regression suite: `python scripts/test_gateway_harness.py --ip 192.168.68.63 --run-suite`.

- [ ] **Task 5: User Manual Verification Gate**
  - Pause for user manual confirmation before any push or PR creation.

- [ ] **Task 6: Pull Request & Archival**
  - Push branch and create Pull Request on GitHub.
  - Wait for user merge.
  - Switch to master, pull, and delete local and remote branches.
