# Feature Specification: Investigate & Resolve Intermittent Gateway Stall of LoRa RX & MQTT

**Feature Branch**: `040-gateway-rx-mqtt-stall`  
**GitHub Issue**: [toogooda/LoRaFarmNet#39](https://github.com/toogooda/LoRaFarmNet/issues/39)  
**Created**: 2026-10-24  
**Status**: Draft  

**Input**: User problem description:
> "Track as issue in github that you will create. The issue is that intermittently the gateway stops recording new lora messages coming in and all devices show as messages hours late. A restart always fixes it.  
> Context 1: When we see the issue the web page is fully responsive so serving pages not an issue, but devices appear to have not sent messages for hours (normally every 15 mins). The reason I know it's the gateway and not a node issue is that every device goes missing at the same time.  
> Context 2: The gateway has three things it is trying to do at the same time:  
> 1) Webserver serving pages to clients - high priority, an active client should not wait too long.  
> 2) Receive new lora messages and update values of farmNetwork in-memory values which pages use to show remote information to clients. This is the second most important and still a high priority.  
> 3) Send the latest value to MQTT for systems like Home Assistant to use the data - best efforts but most should get through.  
> Context 3: Internet outages can happen at house level, mqtt level and gateway level - at least one of those every day so there is a lot of retry code, some of which is blocking.  
> The issue seems to be that although 1) webserver is still working, 2) and 3) are blocked somehow. My hunch is it might relate to MQTT retries in some forever loop when a restart is needed instead."

---

## Background & Problem Statement

The LoRaNetGateway is an ESP32-WROVER system acting as a central hub for a farm-wide private LoRa network. In normal operation, remote field sensor nodes transmit binary LoRa packets approximately every 15 minutes. The Gateway processes these packets, updates the in-memory `FarmNetwork` model in PSRAM, updates web clients, and pushes MQTT state updates.

### Symptoms of Failure
- **Simultaneous Device Disappearance**: All devices across the entire network stop updating simultaneously and show on the web dashboard as hours late.
- **Asymmetric Failure**: The asynchronous HTTP web server (`ESPAsyncWebServer`) remains completely responsive and serves pages rapidly without error, while LoRa packet ingestion and MQTT publishing are completely stalled.
- **Transient Recovery**: A physical power cycle or soft restart always immediately restores normal LoRa packet reception and MQTT publishing.

### System Concurrency Priorities
1. **Web Server (`ESPAsyncWebServer`)**: Serving UI and API requests to human operators and local browsers (High priority - low interactive latency required).
2. **LoRa Message Ingestion (`RadioLib` / `RA01S` / `FarmNetwork`)**: Capturing incoming RF packets, parsing binary payloads, updating `SensorValues` in PSRAM, and broadcasting SSE events (High priority - core telemetry pipeline).
3. **MQTT Telemetry Forwarding (`AsyncMqttClient`)**: Publishing state topics and Home Assistant discovery payloads over Wi-Fi (Best-effort priority - packet loss during upstream WAN outages is tolerable, but MUST NOT compromise priority 1 or 2).

### Architectural Decisions
1. **Migrate to `AsyncMqttClient`**:
   - `PubSubClient` relies on synchronous, blocking TCP socket operations (`client.connect()`, `client.publish()`) and retry loops inside `publishRetained()`. When the broker or network is down, lwIP socket timeouts block `loop()` repeatedly on Core 1 for up to dozens of seconds per packet, preventing `lora.Receive()` from running.
   - Replace `PubSubClient` with `AsyncMqttClient` (built on `AsyncTCP`, which is already used by `ESPAsyncWebServer`). `AsyncMqttClient` is 100% event-driven and non-blocking. It connects and reconnects in the background, and publish calls never block `loop()`.
2. **Hardware/Task Watchdog Timer (WDT) with BLE Setup Exception Gating**:
   - Enable an ESP32 Task/Hardware Watchdog Timer on the Arduino `loop()` (e.g. 60–120 seconds).
   - **Critical Exception Constraint**: Bluetooth Low Energy (BLE) serial configuration (`wifiConf.handleBLE()`, `connectWithBLE()`) can pause or occupy execution for several minutes while a human types Wi-Fi or MQTT credentials. The Watchdog MUST be explicitly paused or continuously fed whenever BLE is active, advertising, or in an active client session.

---

## Requirements

### Functional Requirements
- **FR-001**: Network / MQTT connectivity failure MUST NEVER block the main execution loop or delay LoRa message reception.
- **FR-002**: MQTT operations MUST be implemented using `AsyncMqttClient`, utilizing non-blocking asynchronous callbacks (`onConnect`, `onDisconnect`, etc.).
- **FR-003**: If the MQTT broker or upstream network is unreachable, `AsyncMqttClient` MUST handle reconnection timers asynchronously in the background without blocking `loop()`.
- **FR-004**: If disconnected from MQTT, publishing calls MUST safely drop or defer without blocking `FarmNetwork` or `SensorValue::updateValue()`.
- **FR-005**: The system MUST implement a Task Watchdog Timer (WDT) on `loop()` (threshold 60–120 seconds) that triggers a clean `ESP.restart()` if `loop()` halts.
- **FR-006**: The Watchdog MUST be conditionally bypassed or fed during active BLE configuration and connection states (`isBleActive()` / client connected).
- **FR-007**: Diagnostic Telemetry: The Gateway web UI and status API (`/api/test/status`) MUST expose MQTT connection status, drop counts, and loop watchdog state.

### Non-Functional Requirements
- **NFR-001**: Web server response latency for connected browser clients MUST remain snappy (< 200ms) under all network conditions.
- **NFR-002**: Zero memory leaks in FreeRTOS buffers during prolonged WAN disconnection.
- **NFR-003**: Passes build verification in PlatformIO (`pio run -e gateway-test` and `pio run -e gateway-release`).

---

## User Scenarios & Testing

### User Story 1 - Non-Blocking MQTT via AsyncMqttClient (Priority: P1)
As a farm operator during an internet or MQTT broker outage, I want the Gateway to continue receiving all LoRa sensor packets and updating the local web dashboard without delay, so that local farm monitoring and direct device-to-device automation remain 100% functional.

**Acceptance Scenarios**:
1. **Given** the Gateway is running and receiving LoRa packets from nodes,
   **When** the MQTT broker or home router WAN drops offline for 4+ hours,
   **Then** the Gateway continues receiving LoRa messages, in-memory `FarmNetwork` timestamps update every 15 minutes, and the local web dashboard shows all devices online.

### User Story 2 - Resilient Watchdog Auto-Recovery with BLE Exception (Priority: P1)
As a farm operator, if any unexpected loop stall or peripheral lockup occurs, I want the Gateway to automatically restart itself, while allowing uninterrupted manual BLE configuration sessions.

**Acceptance Scenarios**:
1. **Given** normal operation, **When** `loop()` is starved or frozen for > 90 seconds, **Then** the hardware WDT triggers a clean reboot.
2. **Given** BLE setup mode is active or a Bluetooth client is connected, **When** a user interacts via BLE for several minutes, **Then** the WDT does NOT trigger a reboot.

---

## Success Criteria

- **SC-001**: LoRa message reception never ceases when MQTT or Wi-Fi connectivity is lost.
- **SC-002**: No blocking loops or indefinite delays exist in MQTT or networking routines.
- **SC-003**: Active BLE setup sessions can proceed without triggering watchdog resets.
- **SC-004**: Passes automated regression test harness in `scripts/test_gateway_harness.py`.
