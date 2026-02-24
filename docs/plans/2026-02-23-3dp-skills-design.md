# Design: 3D Printing Expert Skills for Claude Code

**Date:** 2026-02-23
**Status:** Approved

## Overview

Five domain-specific Claude Code skills for 3D printing firmware and slicer expertise. Each skill is independently invocable, laser-focused on its domain, and capable of: looking up documentation, generating/editing configs, troubleshooting issues, and comparing/recommending settings.

## Skills

### `3dp:klipper-expert`
Expert on Klipper firmware configuration, macros, kinematics, probes, KAMP, Moonraker, Mainsail/Fluidd.

**Resources:**
- `https://www.klipper3d.org/Config_Reference.html`
- `https://www.klipper3d.org/G-Codes.html`
- KAMP GitHub: `https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging`
- WebSearch for Klipper-specific queries

**Capabilities:**
- Answer config questions with specific parameter names and values
- Generate and edit properly-formatted `.cfg` blocks
- Troubleshoot from error messages or symptoms
- Explain macros and Jinja2 templating
- Know about KAMP, Mainsail, Fluidd, Moonraker extensions

**Domain rules:**
- Never suggest manually editing the `#*# <--- SAVE_CONFIG --->` block
- Always distinguish probe coordinates vs nozzle coordinates
- Flag anything requiring physical recalibration (input shaper, z_offset, PID, extrusion)

---

### `3dp:marlin-expert`
Expert on Marlin firmware — `Configuration.h`, `Configuration_adv.h`, board definitions, stepper drivers, flashing.

**Resources:**
- `https://marlinfw.org/docs/configuration/configuration.html`
- `https://marlinfw.org/docs/features/`
- Marlin GitHub: `https://github.com/MarlinFirmware/Marlin`
- WebSearch for Marlin-specific queries

**Capabilities:**
- Explain and edit `#define` blocks with correct syntax
- Help with feature enables/disables and their dependencies
- Identify correct board definitions and stepper driver settings
- Troubleshoot compile errors
- Guide the firmware flashing process

---

### `3dp:orcaslicer-expert`
Expert on OrcaSlicer printer profiles, process profiles, filament profiles, and calibration workflows.

**Resources:**
- OrcaSlicer GitHub wiki: `https://github.com/SoftFever/OrcaSlicer/wiki`
- OrcaSlicer GitHub repo: `https://github.com/SoftFever/OrcaSlicer`
- WebSearch for OrcaSlicer-specific queries

**Capabilities:**
- Understand and edit the JSON schema for printer/process/filament profiles
- Generate or modify profile JSON blocks
- Explain settings and their interactions (e.g., pressure advance in OrcaSlicer vs Klipper)
- Recommend settings for specific use cases (speed, quality, etc.)
- Guide calibration workflows: flow rate, pressure advance, temp tower, tolerance tests

---

### `3dp:gcode-reference`
G-code and M-code command reference for Marlin and Klipper only.

**Resources:**
- Marlin: `https://marlinfw.org/docs/gcode/`
- Klipper: `https://www.klipper3d.org/G-Codes.html`
- WebSearch for specific command queries

**Capabilities:**
- Look up any G/M-code command with syntax and parameters
- Note firmware-specific differences between Marlin and Klipper
- Generate start/end G-code sequences for slicers
- Troubleshoot issues caused by specific commands

**Scope:** Marlin and Klipper only — no RepRap wiki.

---

### `3dp:web-research`
General 3D printing research agent — spans domains, searches community resources.

**Resources:**
- WebSearch across Reddit (r/klippers, r/ender3, r/3Dprinting), GitHub Issues, Voron docs, community forums
- WebFetch for specific pages

**Capabilities:**
- Research solutions to problems that span domains
- Find community guides and resources
- Research specific hardware (boards, probes, hotends, extruders)
- Summarize findings with source links

---

## Skill File Structure

Skills are stored as markdown files in the Claude Code plugins directory. Each skill contains:

1. **Identity** — what the agent is and its scope
2. **Workflow** — step-by-step process for handling a request
3. **Resources** — specific URLs and search strategies
4. **Output format** — code blocks for configs, structured reasoning for troubleshooting
5. **Domain caveats** — domain-specific rules and warnings

## What Is Not Included

- No RepRap wiki coverage in gcode-reference (Marlin + Klipper only)
- No router/dispatcher skill — user picks the appropriate skill directly
- No automated deployment or printer control — these are research/config skills only
