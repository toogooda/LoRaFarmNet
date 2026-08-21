# Feature Specification: Dynamic Add / Pair Device Page & Remote Router Discovery

**Feature Branch**: `009-new-add-device-pairing-page`  
**Created**: 2026-08-21  
**Status**: Draft  
**Input**: GitHub Issue #9 ("New add device page shows list of received items not setup dynamically show devices as messages arrive")

---

## Overview
Re-architect the device discovery and pairing experience on the Gateway. The new **Add / Pair Device** screen (`/adddevice`) replaces static lists with an interactive real-time stream (Server-Sent Events) that dynamically pops in new unconfigured devices as their LoRa packets arrive over the air. It also enables querying farm Routers for remotely heard unconfigured nodes, highlights the safest physical routes based on RSSI comparisons, and provides single-click router assignment.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Real-Time Live Discovery & Dynamic Card Streaming (Priority: P1 🎯 MVP)

When setting up new hardware in the farm field or workshop, the user opens the "Add Device" screen on a phone or laptop. The screen displays an active discovery indicator ("Listening for LoRa devices..."). As soon as a field node powers up and broadcasts its first telemetry packet, the Gateway captures it and pushes it over SSE, dynamically rendering the full device card (`sendDevice()`) on screen without page reloads.

**Why this priority**: Core value of Issue #9—provides instant visual feedback to installers in the field that their new device is communicating with the Gateway.

**Independent Test**:
- Open `/adddevice` in a browser.
- Inject a synthetic or live LoRa packet from an unknown MAC (`DeviceType::New`).
- Verify the device card dynamically appears in the DOM with live RSSI, port values, and Setup/Ignore buttons.

**Acceptance Scenarios**:
1. **Given** a user is viewing `/adddevice`, **When** a LoRa packet from an unconfigured device is received by the Gateway, **Then** the device card appears immediately at the top of the list via Server-Sent Events (SSE).
2. **Given** a device card is rendered on `/adddevice`, **When** the user clicks "Configure / Setup", **Then** the standard template/entity setup workflow is triggered.
3. **Given** a device card is rendered on `/adddevice`, **When** the user clicks "Ignore", **Then** the device is set to `DeviceType::Ignore` and removed from the active new list.

---

### User Story 2 - Historical 24-Hour Buffer & Filter Toggles (Priority: P1 🎯 MVP)

Field nodes often transmit periodically (e.g. every 15 minutes or hour). If a node transmitted before the user opened `/adddevice`, it should still be discoverable. The page checks `d->getTimeLastMsg()` and presents unconfigured devices heard in the last 24 hours as "Older Unsetup Devices".

**Why this priority**: Prevents technicians from having to wait up to an hour for sleeping sensor nodes to transmit their next periodic heartbeat.

**Independent Test**:
- Ensure an unconfigured device exists in memory with `timeLastMsg` within the last 24 hours.
- Open `/adddevice`.
- Toggle "Older Unsetup Devices" ON/OFF and verify the device card toggles visibility.

**Acceptance Scenarios**:
1. **Given** an unconfigured device (`DeviceType::New`) transmitted within the last 24 hours, **When** `/adddevice` loads, **Then** it is cataloged under "Older Unsetup Devices".
2. **Given** filter toggles at the top of `/adddevice`, **When** "Older Unsetup Devices" is toggled, **Then** historical 24-hour devices show/hide accordingly.
3. **Given** an ignored unconfigured device (`DeviceType::Ignore`), **When** "Ignored Unsetup Devices" is toggled ON, **Then** it appears with an "Unignore / Add" option.
4. **Given** any device with `DeviceType::Configured`, **When** `/adddevice` renders, **Then** it is strictly filtered out.

---

### User Story 3 - Remote Router Query & Multi-Route Smart Evaluation (Priority: P2)

On large farms, remote nodes may be out of direct Gateway radio range and only heard by one or more field Routers. The user can click "Query Router" for specific routers. The Gateway sends a LoRa `QN: 1` query to that Router, which replies with individual Option A messages for each unsetup node it has heard in the last 24 hours (`M1`, `M2`, `M3`, `DT`, `RS`). The UI aggregates routes heard across direct Gateway and Routers and calculates the safest route.

**Why this priority**: Solves multi-hop farm pairing where devices cannot reach the Gateway directly.

**Independent Test**:
- Inject a router query response (`From: Router1, M1..M3: TargetMAC, DT: 2, RS: 95`).
- Query direct Gateway (`RS: 118`).
- Verify Router 1 is badged with green **"Safest Route (-95 dBm)"** and Gateway direct is badged in amber **"Weak Link (-118 dBm)"**.

**Acceptance Scenarios**:
1. **Given** one or more configured Routers on the network, **When** `/adddevice` renders, **Then** a "Query Router" toggle button is displayed for each router.
2. **Given** a user clicks "Query Router", **When** the Gateway transmits `QN: 1` to the router, **Then** the router sends back its list of unsetup nodes seen in the last 24 hours via Option A messages (`M1`, `M2`, `M3`, `DT`, `RS`).
3. **Given** a device heard across multiple paths (Gateway direct and/or Routers), **When** evaluated on `/adddevice`, **Then** the path with the lowest attenuation / best RSSI is marked with a **Green "Safest Route"** badge.
4. **Given** any route with RSSI weaker than -115 dBm (> 115 dBm attenuation), **When** displayed, **Then** its RSSI badge is highlighted in **Amber / Warning**.

---

### User Story 4 - Single-Click Router Assignment & Telemetry Forwarding (Priority: P2)

When a node is discovered via a Router, the UI shows its MAC, Device Type (`DT`), RSSI, and an "Add via Router" button. Clicking this button sends an `addRoutedNode` command to that Router. The Router saves the rule and forwards the last cached message for that device to the Gateway, promoting it to a fully-populated New Device card on the Gateway screen.

**Why this priority**: Seamlessly closes the loop on remote node pairing without requiring manual routing ID configuration.

**Independent Test**:
- On a router-discovered node, click "Add via Router".
- Verify `addRoutedNode` command is queued/transmitted to the Router.
- Verify the router forwards the device frame, rendering the full sensor card.

**Acceptance Scenarios**:
1. **Given** a node discovered via Router query, **When** the user clicks "Add via Router", **Then** the Gateway executes `d->addRoutedNode(mac, routerId)` and queues the configuration packet to the Router.
2. **Given** the Router accepts the routing rule, **When** it transmits the node's cached telemetry frame, **Then** the Gateway receives the packet and renders the full `DeviceType::New` card on `/adddevice`.

---

## Edge Cases

- **Router Query Timeout / Offline Router**: If a Router is asleep or unreachable when queried with `QN: 1`, the UI displays a subtle "Router query timed out" message without blocking the live Gateway discovery stream.
- **Simultaneous Multiple Transmissions**: If multiple new nodes transmit at the same time, the SSE pipeline pushes discrete events per message without race conditions or memory corruption.
- **Browser Reconnection**: If the browser tab loses WiFi connection and reconnects, the EventSource automatically reconnects to `/api/pair/events` and synchronizes the latest in-memory 24-hour unsetup list.
- **Repeated Messages from Same New Device**: If a new device transmits repeatedly while the user is viewing `/adddevice`, its existing card is updated in-place with the latest sensor values and RSSI rather than duplicating cards.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Gateway MUST expose a dedicated Web UI at `/adddevice` (and alias `/pair`) accessible via the navigation bar "Add Device" icon button (`fa-circle-plus`).
- **FR-002**: Gateway MUST provide a Server-Sent Events (SSE) stream at `/api/pair/events` that pushes real-time JSON payloads whenever an unconfigured (`DeviceType::New`) LoRa packet is processed.
- **FR-003**: The Add Device web page MUST use `EventSource` to listen to `/api/pair/events` and dynamically insert or update device cards in the DOM in real time.
- **FR-004**: Gateway MUST filter all configured devices (`DeviceType::Configured`) out of the Add Device display list.
- **FR-005**: Gateway MUST evaluate `now - d->getTimeLastMsg() <= 86400` to categorize devices heard within the past 24 hours as "Older Unsetup Devices".
- **FR-006**: Add Device UI MUST provide interactive toggle filters:
  - `Toggle Older Unsetup Devices` (Default: OFF)
  - `Toggle Ignored Unsetup Devices` (Default: OFF)
  - Individual `Query Router [Router Name]` buttons for all registered routers on the farm network.
- **FR-007**: Gateway MUST support LoRa command port `QN` (`Query New Devices`) sent to Routers to solicit their list of unconfigured devices heard in the past 24 hours.
- **FR-008**: Routers MUST respond to `QN` using Option A individual messages (`M1`, `M2`, `M3` for MAC, `DT` for Device Type, `RS` for RSSI).
- **FR-009**: Gateway MUST maintain an in-memory route comparison table for all unconfigured MACs heard directly and/or reported by Routers.
- **FR-010**: Add Device UI MUST dynamically display a **Green "Safest Route"** badge on the path with the strongest RSSI.
- **FR-011**: Add Device UI MUST display an **Amber Warning** badge on any route with RSSI weaker than -115 dBm.
- **FR-012**: Router-discovered items MUST render with an "Add via Router" button that issues `addRoutedNode` to the target Router and awaits the forwarded telemetry packet.
- **FR-013**: Navigation bar across all Gateway pages MUST include the "Add Device" link.

---

## REST & SSE Endpoint Contracts

### 1. SSE Event Stream: `GET /api/pair/events`
- **Protocol**: `text/event-stream`
- **Events**:
  - `event: new_device` &rarr; `data: {"id":"fc0fe71463c5","name":"Device 63C5","type":"New","rssi":-105,"timeLastMsg":1724230000,"ports":{"IV":325,"SM":15,"DT":0}}`
  - `event: router_device` &rarr; `data: {"routerId":101,"routerName":"North Hill Router","id":"fc0fe71454d8","dt":2,"rssi":-92}`
  - `event: heartbeat` &rarr; `data: {"uptime":12345}`

### 2. Router Query Trigger: `POST /api/pair/query_router?routerId=<id>`
- **Response**: `{"success": true, "routerId": 101, "status": "Query QN transmitted"}`

### 3. Setup Route Assignment: `POST /api/pair/add_route?routerId=<id>&deviceId=<hex>`
- **Response**: `{"success": true, "routerId": 101, "deviceId": "fc0fe71454d8", "status": "Routing command queued"}`

---

## Success Criteria

1. **Zero Refresh Dynamic Addition**: An unconfigured LoRa field node powering up appears on `/adddevice` within < 1 second of Gateway packet reception without manual browser refresh.
2. **24-Hour Memory**: Unconfigured devices heard earlier in the day appear under "Older Unsetup Devices" when toggled ON.
3. **Multi-Router Safest Path**: Discovered devices heard across multiple routers or gateway directly display the Green "Safest Route" badge on the optimal path.
4. **Automated Verification**: End-to-end injection and SSE event reception verified using `test_gateway_harness.py`.
