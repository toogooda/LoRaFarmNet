# Feature Specification: Move to Separate Test and Release Builds

**Feature Branch**: `031-separate-test-release-builds`  
**GitHub Issue**: [#31](https://github.com/toogooda/LoRaFarmNet/issues/31)  
**Created**: 2026-08-23  
**Status**: Draft  
**Input**: User request: "Move to separate test and release builds" (Issue #31)

---

## Overview

Establish clean compile-time separation between **Development / Test Builds** and **Production / Release Builds** in the LoRaNetGateway firmware:
1. **Default Development Environment (`esp-wrover-kit-test` / `[env:esp-wrover-kit]` with `-DGATEWAY_TEST_BUILD`)**:
   - Compiles firmware with testing flags enabled.
   - Test mode endpoints (`/api/test/*`), memory snapshots, and simulation UI buttons are compiled in and always accessible without manual arming.
   - Renders a prominent **"TEST BUILD"** visual badge in the top navigation header on all web pages.
2. **Production Release Environment (`esp-wrover-kit-release`)**:
   - Strips out all test APIs, test endpoints, synthetic message injection routes, and demo simulation buttons via `#ifdef GATEWAY_TEST_BUILD` preprocessor blocks.
   - Produces a lean, secure, attack-surface-minimized production binary.
3. **Automated Release Command (`pio run -t release`)**:
   - Automatically builds the production environment and uploads the stripped `.bin` asset to GitHub Releases.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Default Test Environment & Always-Available Test APIs (Priority: P1 🎯 MVP)

As an engineer or AI agent developing on the Gateway, every standard build (`pio run`, `pio run -t upload`) compiles in test mode by default with `-DGATEWAY_TEST_BUILD`, enabling the synthetic message injection APIs, memory snapshot/restore APIs, and testing UI tools without needing manual activation or timing out.

**Why this priority**: Core developer/AI efficiency. Test harnesses and verification loops must run reliably and seamlessly out of the box during pairing, debugging, and feature development.

**Independent Test**:
- Compile with `pio run` (default environment).
- Flash with `pio run -t upload`.
- Send request to `GET /api/test/status` or `POST /api/test/message`; verifies response code is `200 OK` immediately without requiring prior `/settings?testmode=1`.

**Acceptance Scenarios**:
1. **Given** a firmware flashed with default test configuration, **When** `/api/test/status` is queried, **Then** it returns `200 OK` with system telemetry and `testBuild: true`.
2. **Given** test build firmware, **When** synthetic messages are posted to `/api/test/message`, **Then** the Gateway ingests them into the network object graph and broadcasts SSE events.

---

### User Story 2 - Prominent "Test Build" UI Badge (Priority: P1 🎯 MVP)

As a user or engineer viewing any page of the Gateway Web UI in a test build, a clear **"TEST BUILD"** visual badge (e.g. amber badge in the navbar) is displayed so there is zero ambiguity that test APIs and simulation tools are compiled into the running firmware.

**Why this priority**: Immediate visual feedback preventing accidental deployment or confusion between production and lab/test firmware.

**Independent Test**:
- Open `http://<ip>/` or `http://<ip>/adddevice`.
- Verify the navbar displays the `TEST BUILD` badge.

**Acceptance Scenarios**:
1. **Given** a test build, **When** browsing any page (`/`, `/adddevice`, `/settings`, `/map`), **Then** the navbar renders `<span class='badge bg-warning text-dark me-2'>TEST BUILD</span>`.
2. **Given** a release build, **When** browsing any page, **Then** no test badge is displayed.

---

### User Story 3 - Production Release Build Strips All Test Artifacts (Priority: P1 🎯 MVP)

As a farm owner deploying a production Gateway, release firmware compiled without `-DGATEWAY_TEST_BUILD` contains zero test routes, zero synthetic injection handlers, and zero demo simulator UI buttons, minimizing flash size and eliminating security attack surfaces.

**Why this priority**: Production safety, security, and binary size optimization.

**Independent Test**:
- Build release environment: `pio run -e esp-wrover-kit-release`.
- Query `/api/test/status`, `/api/test/message`, `/api/test/snapshot`.
- Verify `404 Not Found` (routes do not exist in binary).
- Open `/adddevice`; verify the `[🧪 Simulate Router Response (Demo)]` button is completely omitted from the HTML.

**Acceptance Scenarios**:
1. **Given** a release build, **When** an HTTP request is made to any `/api/test/*` endpoint, **Then** the Gateway responds with `404 Not Found`.
2. **Given** a release build, **When** `/adddevice` or `/settings` is viewed, **Then** test simulator buttons and test mode toggles are absent from the DOM.

---

### User Story 4 - Release Automation Pipeline (Priority: P2)

As a maintainer creating a release (`pio run -t release`), PlatformIO automatically compiles the `esp-wrover-kit-release` environment, verifies zero compile errors, and publishes the stripped production binary asset to GitHub Releases.

**Why this priority**: Eliminates human error when packaging releases.

**Independent Test**:
- Execute release target; verify it builds `esp-wrover-kit-release` and creates release with `gateway-firmware-vX.Y.Z.bin`.

**Acceptance Scenarios**:
1. **Given** `pio run -t release`, **When** the script executes, **Then** it builds using the release configuration and attaches the stripped `.bin` to GitHub.

---

## Edge Cases

- **What happens if someone flashes a Release build to the test bench and runs `test_gateway_harness.py`?**  
  The test harness will detect 404 on `/api/test/status` and report: *"Gateway is running in Production / Release mode (test APIs not present). Flash with test environment to run test suite."*
- **What happens to OTA firmware updates?**  
  OTA updates work identically on both builds. Flashing a release firmware replaces the test firmware and removes the test badge immediately.

---

## Requirements

### Functional Requirements
- **FR-001**: `platformio.ini` MUST define two distinct environments:
  - `[env:esp-wrover-kit-test]` (Default environment, includes `-DGATEWAY_TEST_BUILD`)
  - `[env:esp-wrover-kit-release]` (Production environment, excludes test build flags)
- **FR-002**: All test API routes (`/api/test/*`) in `WebHelper.h` MUST be enclosed in `#ifdef GATEWAY_TEST_BUILD`.
- **FR-003**: The test mode auto-timeout and enable switch (`testModeActive`, `/settings?testmode=1`) MUST be simplified: in test builds, test APIs are permanently active; in release builds, the code is omitted entirely.
- **FR-004**: The navbar header in `WebHelper.h` MUST render a `TEST BUILD` badge when `#ifdef GATEWAY_TEST_BUILD` is defined.
- **FR-005**: All UI simulation and demo triggers (e.g. `[🧪 Simulate Router Response (Demo)]`) MUST be enclosed in `#ifdef GATEWAY_TEST_BUILD`.
- **FR-006**: `publish_release.py` MUST target the release environment (`esp-wrover-kit-release`) when packaging production firmware assets.
