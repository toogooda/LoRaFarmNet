---
name: speckit
description: GitHub Spec Kit spec-driven development workflow for establishing project constitution, drafting feature specs, creating implementation plans, generating tasks, and executing implementations.
---

# GitHub Spec Kit Skill Instructions

Use this skill whenever the user mentions `speckit`, `spec-kit`, `specify`, or asks to follow a spec-driven development workflow.

## Available Spec Kit Workflows

### 1. Project Constitution (`speckit constitution`)
* **Purpose**: Establish or update project core principles, architectural constraints, governance, and quality gates in `.specify/memory/constitution.md`.
* **Command**: Execute `specify.exe` workflow or update `.specify/memory/constitution.md` directly following the project template and rules.

### 2. Feature Specification (`speckit specify`)
* **Purpose**: Draft or update feature specifications in `specs/[###-feature-name]/spec.md`.
* **Template**: Use `.specify/templates/spec-template.md`.

### 3. Implementation Plan (`speckit plan`)
* **Purpose**: Create technical implementation plans, research findings, and architecture designs in `specs/[###-feature-name]/plan.md`.
* **Template**: Use `.specify/templates/plan-template.md`.

### 4. Task Generation (`speckit tasks`)
* **Purpose**: Break down implementation plans into actionable tasks in `specs/[###-feature-name]/tasks.md`.
* **Template**: Use `.specify/templates/tasks-template.md`.

### 5. Implementation Execution (`speckit implement`)
* **Purpose**: Execute tasks defined in `tasks.md` sequentially, validating with `pio run` build checks.

### 6. Codebase Convergence (`speckit converge`)
* **Purpose**: Compare current codebase state against tasks and sync remaining items.

## Usage Guidelines
* Always check [.specify/memory/constitution.md](file:///c:/Users/USER/Projects/LoRaFarmNet/.specify/memory/constitution.md) before planning or implementation.
* Always enforce the mandatory repo workflow: ask before creating feature branches, explain code changes before editing source files, verify with `pio run`, and create PRs without auto-merging.
