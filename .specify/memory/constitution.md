# LoRaFarmNet Constitution

## Core Principles

### I. Hardware Addressing & Target Architecture
* **MCU Target Invariants**: Firmware must explicitly target specified hardware platforms:
  * **ESP32-WROVER**: Gateway (`PSRAM` mandatory).
  * **ATmega644PA**: Low-power field nodes & repeaters built via Arduino framework in PlatformIO.
* **Hardware Addressing**: Every device extracts its unique 6-byte hardware address from onboard I2C EEPROM/MAC chip at boot. Hardcoded MACs or device IDs in source code are strictly prohibited.

### II. Power Integrity & Airtime Minimization (NON-NEGOTIABLE)
* **Not all nodes sleep**: Gateway, Dual Gate Controller, and Repeater nodes MUST NEVER enter deep sleep.
* **Supercapacitor Power Topology**: Nodes operate on battery-free 10-15F supercapacitors. Minimizing awake time and radio airtime is critical.
* **Deep Sleep Management**: Periodic field nodes MUST enter deep sleep (15 min default) via `nano64DeepSleep.h`. Peripherals and sensors must be GPIO power-gated off prior to sleep entry.
* **No Blocking Delays**: Blocking `delay()` calls on sleeping nodes are forbidden unless protected by an explicit inline `// AI-PROTECT:` hardware-settling comment.
* **Always-On Nodes**: Gateway, Dual Gate Controller, and Repeater nodes MUST NEVER enter deep sleep.

### III. Fixed-Pair Binary Frame Protocol
* **Frame Layout**: Over-the-air messages follow a fixed-pair binary stream: `Source Address (6B)` + `Destination Address (6B)` + `Payload Stream (N*4B)` + `CS Checksum (4B)`.
* **Stream Markers**: `MI` (Message ID) MUST be the first key pair. `CS` (Checksum/CRC) MUST be the final key pair terminating stream parsing.
* **ACK Enforcement**: Default sensor transmission is best-effort (transmit once, NO ACK, immediate sleep). Gateway config targeting Always-On nodes and Button nodes require explicit ACK.

### IV. Core Shared Libraries
* All transceivers and power logic must build upon standardized core libraries:
  * `Ra01S.h`: Transceiver driver (SX126x/Ra-01S).
  * `LoRaHelper.h`: State machine & transmission retries.
  * `LoraMsg.h`: Binary serialization, parsing, encryption, CRC.
  * `nano64DeepSleep.h`: Hardware sleep timers & wake interrupts.

### V. Build Verification & Mandatory PR Workflow
* **Compilation Gate**: All code modifications must pass local build verification (`pio run`) before completion.
* **Pull Request Policy**: Changes are delivered exclusively via feature branches and GitHub Pull Requests. PRs must be approved and merged manually by the user; local merging is prohibited.
* **Track Documentation**: Tasks follow Conductor track standards (`spec.md`, `plan.md`, metadata).

---

## Hardware & System Constraints

* **Field Upload Speeds**: ATmega644PA repeaters require low ISP speed (100–200kHz, `-B 10` or `-B 5` via AVRISP mkII).
* **Device Registry**: Device Type Registry (ID, DeviceName, Filename) is maintained in external storage (e.g. `/lfm/devtypes/registry.dat`), distinct from `network.dat`.

---

## Development Workflow & Quality Gates

1. **Task Identification & Feature Branching**: Obtain user confirmation prior to creating feature branches.
2. **Conductor Spec & Plan**: Maintain track documentation in `conductor/tracks/`.
3. **Build & Manual Verification**: Execute `pio run` after significant changes and pause for hardware verification.
4. **Pull Request & Cleanup**: Push branch, create PR for user review. Upon merge, pull master and remove local/remote branches.

---

## Governance

* This Constitution supersedes informal guidelines.
* Amendments require documented justification, version increment, and synchronization across dependent templates.
* **Version**: 1.0.0 | **Ratified**: 2026-07-28 | **Last Amended**: 2026-07-28
