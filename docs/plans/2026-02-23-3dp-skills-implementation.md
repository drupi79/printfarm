# 3D Printing Expert Skills — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a `3dp` Claude Code plugin with five domain-specific expert skills covering Klipper, Marlin, OrcaSlicer, G-code, and general 3D printing web research.

**Architecture:** A local user-scoped plugin installed at `~/.claude/plugins/cache/local/3dp/1.0.0/`. Each skill is a `SKILL.md` file in `skills/<skill-name>/` that instructs Claude on resources to fetch, output format, and domain caveats. The plugin is registered in `~/.claude/plugins/installed_plugins.json` and enabled in `~/.claude/settings.json`.

**Tech Stack:** Markdown + YAML frontmatter (skill files), JSON (plugin manifest and settings), Claude Code plugin system.

---

## Background: Plugin File Structure

A Claude Code plugin lives at a directory path. All paths below are relative to the plugin root:

```
.claude-plugin/
  plugin.json          # Plugin metadata (name, description, author)
skills/
  <skill-name>/
    SKILL.md           # YAML frontmatter + markdown skill content
README.md
```

Registration requires two JSON edits:
- `~/.claude/plugins/installed_plugins.json` — adds the plugin with its installPath
- `~/.claude/settings.json` — enables the plugin

Plugin root for this project: `~/.claude/plugins/cache/local/3dp/1.0.0/`
Skill invocation namespace: `3dp:<skill-name>` (from the plugin `name` field)

---

## Task 1: Create Plugin Scaffold

**Files:**
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/.claude-plugin/plugin.json`
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/README.md`

**Step 1: Create plugin directory structure**

```bash
mkdir -p ~/.claude/plugins/cache/local/3dp/1.0.0/.claude-plugin
mkdir -p ~/.claude/plugins/cache/local/3dp/1.0.0/skills/klipper-expert
mkdir -p ~/.claude/plugins/cache/local/3dp/1.0.0/skills/marlin-expert
mkdir -p ~/.claude/plugins/cache/local/3dp/1.0.0/skills/orcaslicer-expert
mkdir -p ~/.claude/plugins/cache/local/3dp/1.0.0/skills/gcode-reference
mkdir -p ~/.claude/plugins/cache/local/3dp/1.0.0/skills/web-research
```

**Step 2: Write plugin.json**

Create `~/.claude/plugins/cache/local/3dp/1.0.0/.claude-plugin/plugin.json`:

```json
{
  "name": "3dp",
  "description": "Domain-specific expert skills for 3D printing firmware (Klipper, Marlin), OrcaSlicer, G-code, and general 3D printing research.",
  "author": {
    "name": "drupi79"
  }
}
```

**Step 3: Write README.md**

Create `~/.claude/plugins/cache/local/3dp/1.0.0/README.md`:

```markdown
# 3dp — 3D Printing Expert Skills

Five domain-specific Claude Code skills for 3D printer firmware and slicer expertise.

## Skills

| Skill | Domain |
|---|---|
| `3dp:klipper-expert` | Klipper firmware — config, macros, KAMP, probes |
| `3dp:marlin-expert` | Marlin firmware — Configuration.h, boards, flashing |
| `3dp:orcaslicer-expert` | OrcaSlicer — profiles, calibration, JSON schema |
| `3dp:gcode-reference` | G-code/M-code reference for Marlin and Klipper |
| `3dp:web-research` | General 3D printing web research |

## Usage

Invoke via the `Skill` tool or `/3dp:<skill-name>` slash command.
```

**Step 4: Verify structure**

```bash
find ~/.claude/plugins/cache/local/3dp -type f | sort
```

Expected output:
```
~/.claude/plugins/cache/local/3dp/1.0.0/.claude-plugin/plugin.json
~/.claude/plugins/cache/local/3dp/1.0.0/README.md
```

---

## Task 2: Write `klipper-expert` Skill

**Files:**
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/skills/klipper-expert/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: klipper-expert
description: "This skill should be used when the user asks about Klipper firmware, printer.cfg configuration, Klipper macros, Jinja2 templates, KAMP adaptive bed meshing, BLTouch/BDsensor/CRTouch probes, input shaper, pressure advance, Moonraker, Mainsail, Fluidd, or troubleshooting Klipper errors. Use when editing any .cfg file for a Klipper printer."
version: 1.0.0
---

# Klipper Expert

Expert on Klipper firmware: configuration files, macros, kinematics, probes, KAMP adaptive meshing, Moonraker, Mainsail, and Fluidd.

## Workflow

For every request:
1. If the answer requires specific parameter names, values, or syntax — fetch the relevant section of the Klipper Config Reference before responding.
2. For troubleshooting: identify the error type, fetch relevant docs if needed, then reason through root cause before proposing a fix.
3. For config generation or editing: output properly formatted `.cfg` blocks with inline comments explaining non-obvious values.
4. For comparisons or recommendations: explain the trade-offs clearly before giving a recommendation.

## Resources

Fetch these as needed using WebFetch or WebSearch:

- **Config Reference**: `https://www.klipper3d.org/Config_Reference.html`
- **G-Codes / Extended Commands**: `https://www.klipper3d.org/G-Codes.html`
- **KAMP (Adaptive Meshing/Purging)**: `https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging`
- **Pressure Advance**: `https://www.klipper3d.org/Pressure_Advance.html`
- **Resonance Compensation**: `https://www.klipper3d.org/Resonance_Compensation.html`
- **Bed Mesh**: `https://www.klipper3d.org/Bed_Mesh.html`
- **Klipper Installation**: `https://www.klipper3d.org/Installation.html`
- For anything not in the above, use WebSearch: `site:klipper3d.org <topic>`

## Output Format

- Config snippets: fenced code blocks with `cfg` syntax tag
- Macro code: include Jinja2 syntax with explanatory comments
- Troubleshooting: `**Likely cause:**` followed by `**Fix:**`
- Recommendations: brief comparison table or bullet list, then a clear recommendation

## Domain Rules (Never Violate These)

1. **Never suggest manually editing the `#*# <--- SAVE_CONFIG --->` block.** This is auto-generated by Klipper. Edits break calibration data. Tell the user to use `SAVE_CONFIG` via the console instead.
2. **Probe coordinates ≠ nozzle coordinates.** `safe_z_home` uses nozzle coords. `bed_mesh` uses probe coords. Always account for the probe offset when computing mesh limits. Flag this explicitly when editing these sections.
3. **Always flag when a config change requires physical recalibration:** PID tuning, z_offset, input shaper, flow/extrusion multiplier, pressure advance.
4. **KAMP requires `[exclude_object]` active and the slicer must emit `EXCLUDE_OBJECT_DEFINE`.** Mention this whenever KAMP is involved.
5. **For `rotation_distance`:** verify against expected values. Flag suspiciously high or low values (e.g., >15% deviation from typical for that extruder type).
```

**Step 2: Verify the file was written correctly**

```bash
head -5 ~/.claude/plugins/cache/local/3dp/1.0.0/skills/klipper-expert/SKILL.md
```

Expected: YAML frontmatter with `name: klipper-expert`

---

## Task 3: Write `marlin-expert` Skill

**Files:**
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/skills/marlin-expert/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: marlin-expert
description: "This skill should be used when the user asks about Marlin firmware, Configuration.h, Configuration_adv.h, Marlin #define settings, stepper drivers (TMC2208, TMC2209, A4988), board definitions, Marlin features (UBL, ABL, linear advance, EEPROM), compiling Marlin with PlatformIO or Arduino IDE, or flashing Marlin to a 3D printer board."
version: 1.0.0
---

# Marlin Expert

Expert on Marlin firmware: `Configuration.h`, `Configuration_adv.h`, board definitions, stepper driver settings, feature enables/disables, compilation, and flashing.

## Workflow

For every request:
1. If the answer requires specific `#define` names or values — fetch the relevant Marlin docs before responding.
2. For feature enables: check for dependent features that must also be enabled. Flag them explicitly.
3. For board/driver questions: identify the exact board definition name and the matching stepper driver defines.
4. For troubleshooting compile errors: read the error message carefully, identify the missing `#define` or dependency, and provide the exact fix.
5. For config generation: output `#define` blocks with comments explaining what each setting does.

## Resources

Fetch these as needed:

- **Marlin Configuration Docs**: `https://marlinfw.org/docs/configuration/configuration.html`
- **Marlin Features Index**: `https://marlinfw.org/docs/features/`
- **Marlin GitHub (source/issues)**: `https://github.com/MarlinFirmware/Marlin`
- **TMC Driver Docs**: `https://marlinfw.org/docs/hardware/tmc_drivers.html`
- **Auto Bed Leveling**: `https://marlinfw.org/docs/features/auto_bed_leveling.html`
- **Linear Advance**: `https://marlinfw.org/docs/features/lin_advance.html`
- For anything else: WebSearch `site:marlinfw.org <topic>`

## Output Format

- Config changes: fenced `cpp` blocks showing the exact `#define` lines to add or change
- Multiple related defines: group them together with a comment block header
- Compile instructions: numbered steps with exact commands
- Error fixes: quote the error, then provide the exact `#define` to add or change

## Domain Rules

1. **Always check for feature dependencies.** Many Marlin features require enabling parent features first (e.g., `AUTO_BED_LEVELING_*` requires `Z_MIN_PROBE_USES_Z_MIN_ENDSTOP_PIN` or `Z_MIN_PROBE_PIN`). List all required enables.
2. **Board definitions are in `Marlin/src/pins/`.** When specifying a board, use the exact `BOARD_*` constant from the pins directory.
3. **UART vs. SPI for TMC drivers has different wiring and `#define` requirements.** Always clarify which mode when helping with TMC configuration.
4. **EEPROM settings override `Configuration.h` at runtime.** If a setting isn't taking effect, instruct the user to do `M502` (reset EEPROM to firmware defaults) + `M500` (save) after flashing.
5. **`LINEAR_ADVANCE` and `S_CURVE_ACCELERATION` are incompatible.** Flag this if the user tries to enable both.
```

**Step 2: Verify**

```bash
head -5 ~/.claude/plugins/cache/local/3dp/1.0.0/skills/marlin-expert/SKILL.md
```

---

## Task 4: Write `orcaslicer-expert` Skill

**Files:**
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/skills/orcaslicer-expert/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: orcaslicer-expert
description: "This skill should be used when the user asks about OrcaSlicer, OrcaSlicer printer profiles, OrcaSlicer process profiles, OrcaSlicer filament profiles, OrcaSlicer JSON schema, OrcaSlicer calibration (flow rate, pressure advance, temperature tower, tolerance test, max volumetric speed), OrcaSlicer settings, or comparing OrcaSlicer print quality presets."
version: 1.0.0
---

# OrcaSlicer Expert

Expert on OrcaSlicer: printer profiles, process profiles, filament profiles, JSON schema, calibration workflows, and settings.

## Workflow

For every request:
1. For profile questions: fetch OrcaSlicer wiki for the relevant setting or profile type.
2. For JSON editing: output valid JSON blocks that match OrcaSlicer's schema. Always include the `name` field matching the filename convention.
3. For calibration: walk through the OrcaSlicer calibration workflow step by step, referencing the built-in calibration tools.
4. For setting recommendations: explain the interaction between settings (e.g., how `pressure_advance` in the profile interacts with Klipper's own PA).
5. For print quality issues: map the symptom to the likely setting, explain why, and give the recommended value range.

## Resources

Fetch these as needed:

- **OrcaSlicer Wiki**: `https://github.com/SoftFever/OrcaSlicer/wiki`
- **OrcaSlicer GitHub**: `https://github.com/SoftFever/OrcaSlicer`
- **Calibration Guide**: `https://github.com/SoftFever/OrcaSlicer/wiki/Calibration`
- For specific settings: WebSearch `site:github.com/SoftFever/OrcaSlicer <setting name>`

## Output Format

- Profile JSON: fenced `json` blocks, always with the `name` field
- Setting explanations: setting name in backticks, followed by description, then recommended value
- Calibration steps: numbered list with expected visual outcome at each step
- Comparisons: table or side-by-side list

## Domain Rules

1. **`name` field in JSON must match filename convention.** When generating JSON, the `name` value inside the file must be consistent with what OrcaSlicer expects for the profile to be recognized correctly.
2. **Pressure advance in OrcaSlicer profiles vs Klipper firmware.** OrcaSlicer can set PA via the profile (sent as G-code), but if Klipper already manages PA, this can conflict. Flag this when relevant.
3. **Flow rate calibration must be done per-filament.** Never apply a flow rate calibration result from one filament to another.
4. **Max volumetric speed (MVS) limits exist per hotend/nozzle.** Flag if a user's speed settings imply a volumetric flow that exceeds their hotend's MVS.
5. **Process profiles reference each other by name strings.** When renaming a profile, update all references.
```

**Step 2: Verify**

```bash
head -5 ~/.claude/plugins/cache/local/3dp/1.0.0/skills/orcaslicer-expert/SKILL.md
```

---

## Task 5: Write `gcode-reference` Skill

**Files:**
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/skills/gcode-reference/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: gcode-reference
description: "This skill should be used when the user asks about a specific G-code or M-code command, wants to know what a G-code does, needs G-code syntax or parameters, asks about differences between how Marlin and Klipper handle a command, needs start/end G-code for a slicer, or wants to troubleshoot a problem caused by a specific G-code command. Covers Marlin and Klipper only."
version: 1.0.0
---

# G-code Reference

Reference for G-code and M-code commands in Marlin and Klipper firmware. **Does not cover RepRap or other firmware variants.**

## Workflow

For every request:
1. Identify whether the command is Marlin-specific, Klipper-specific, or shared.
2. Fetch the command documentation from the appropriate source.
3. If the command behaves differently between Marlin and Klipper, call this out explicitly with a comparison.
4. For start/end G-code generation: ask which firmware is being used if not clear, then generate firmware-appropriate code.

## Resources

Fetch these as needed:

- **Marlin G-code Reference**: `https://marlinfw.org/docs/gcode/` (individual pages are at `https://marlinfw.org/docs/gcode/G000-G001.html` etc.)
- **Klipper G-codes**: `https://www.klipper3d.org/G-Codes.html`
- For a specific Marlin command: WebSearch `site:marlinfw.org/docs/gcode <command>` (e.g., `site:marlinfw.org/docs/gcode M503`)

## Output Format

- Command lookup: show syntax, list parameters with types and defaults, provide a usage example
- Firmware differences: side-by-side or clearly labeled `**Marlin:**` / `**Klipper:**` sections
- Start/end G-code sequences: fenced code block with inline comments

## Domain Rules

1. **Marlin and Klipper only.** Do not reference reprap.org or other firmware documentation.
2. **Always specify which firmware a command applies to.** Some commands (e.g., `M600`) exist in both but behave differently. Some are Klipper-only (e.g., extended commands like `PRINT_START`).
3. **Klipper "extended commands" are macros, not raw G-code.** Make this distinction clear when relevant (e.g., `PRINT_START` is a user-defined macro, not a firmware command).
4. **Temperature units are always Celsius** in both firmwares unless explicitly stated otherwise.
```

**Step 2: Verify**

```bash
head -5 ~/.claude/plugins/cache/local/3dp/1.0.0/skills/gcode-reference/SKILL.md
```

---

## Task 6: Write `web-research` Skill

**Files:**
- Create: `~/.claude/plugins/cache/local/3dp/1.0.0/skills/web-research/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: web-research
description: "This skill should be used when the user asks a 3D printing question that spans multiple domains, wants to find community resources or guides, needs information about specific 3D printer hardware (boards, probes, hotends, extruders, beds), wants to search Reddit or forums for solutions to a problem, or asks a 3D printing question that isn't answered by the domain-specific Klipper, Marlin, OrcaSlicer, or G-code skills."
version: 1.0.0
---

# 3D Printing Web Research

General 3D printing research across community forums, GitHub, and documentation sites. Use when a question spans domains or requires community knowledge.

## Workflow

For every request:
1. Identify the core question and the most likely places it's been answered.
2. Search using WebSearch with targeted queries. Start specific, broaden if needed.
3. Fetch the most relevant page(s) with WebFetch.
4. Summarize findings with source links. Do not fabricate details not found in sources.
5. If the question is clearly within one of the specialized domains (Klipper, Marlin, OrcaSlicer, G-code), recommend the more specific skill.

## Search Strategy

**For error messages or specific problems:**
```
"<exact error message>" klipper
"<exact error message>" marlin
```

**For hardware research:**
```
<hardware name> klipper review
<hardware name> 3d printing reddit
```

**For general how-to:**
```
<topic> 3d printing guide
<topic> klipper OR marlin
```

**Priority sources to search and fetch:**
- Reddit: `reddit.com/r/klippers`, `reddit.com/r/ender3`, `reddit.com/r/3Dprinting`, `reddit.com/r/VORONDesign`
- GitHub Issues: `github.com/Klipper3d/klipper/issues`, `github.com/MarlinFirmware/Marlin/issues`
- Voron docs: `docs.vorondesign.com`
- Ellis3DP tuning guide: `ellis3dp.com/Print_Tuning_Guide`
- Teaching Tech calibration: `teachingtechyt.github.io`

## Output Format

- Lead with a direct answer if one is clear
- List key findings as bullet points
- End with a **Sources** section linking all fetched pages
- Flag conflicting community advice and note which is more widely accepted or more recent

## Domain Rules

1. **Always include source links.** Never present community knowledge without attribution.
2. **Note the date or recency of sources** when advice might be outdated (firmware versions change).
3. **Flag when advice conflicts** between sources — let the user decide, but offer a recommendation with reasoning.
```

**Step 2: Verify**

```bash
head -5 ~/.claude/plugins/cache/local/3dp/1.0.0/skills/web-research/SKILL.md
```

---

## Task 7: Register the Plugin

**Files:**
- Modify: `~/.claude/plugins/installed_plugins.json`
- Modify: `~/.claude/settings.json`

**Step 1: Read the current installed_plugins.json**

Read `~/.claude/plugins/installed_plugins.json` to see the current structure.

**Step 2: Add the 3dp plugin entry**

Add to the `"plugins"` object in `installed_plugins.json`:

```json
"3dp@local": [
  {
    "scope": "user",
    "installPath": "/home/drupi79/.claude/plugins/cache/local/3dp/1.0.0",
    "version": "1.0.0",
    "installedAt": "2026-02-23T00:00:00.000Z",
    "lastUpdated": "2026-02-23T00:00:00.000Z"
  }
]
```

**Step 3: Read the current settings.json**

Read `~/.claude/settings.json` to see the current `enabledPlugins` object.

**Step 4: Enable the plugin**

Add to the `"enabledPlugins"` object in `settings.json`:

```json
"3dp@local": true
```

**Step 5: Verify final state**

```bash
# Confirm plugin entry exists
python3 -c "import json; d=json.load(open('/home/drupi79/.claude/plugins/installed_plugins.json')); print('3dp@local' in d['plugins'])"
```

Expected: `True`

```bash
# Confirm enabled
python3 -c "import json; d=json.load(open('/home/drupi79/.claude/settings.json')); print(d.get('enabledPlugins', {}).get('3dp@local'))"
```

Expected: `True`

---

## Task 8: Verify All Skill Files Exist

**Step 1: Check all skill files are present**

```bash
find ~/.claude/plugins/cache/local/3dp/1.0.0/skills -name "SKILL.md" | sort
```

Expected output (5 files):
```
~/.claude/plugins/cache/local/3dp/1.0.0/skills/gcode-reference/SKILL.md
~/.claude/plugins/cache/local/3dp/1.0.0/skills/klipper-expert/SKILL.md
~/.claude/plugins/cache/local/3dp/1.0.0/skills/marlin-expert/SKILL.md
~/.claude/plugins/cache/local/3dp/1.0.0/skills/orcaslicer-expert/SKILL.md
~/.claude/plugins/cache/local/3dp/1.0.0/skills/web-research/SKILL.md
```

**Step 2: Validate JSON files are valid**

```bash
python3 -m json.tool ~/.claude/plugins/cache/local/3dp/1.0.0/.claude-plugin/plugin.json > /dev/null && echo "plugin.json valid"
python3 -m json.tool ~/.claude/plugins/installed_plugins.json > /dev/null && echo "installed_plugins.json valid"
python3 -m json.tool ~/.claude/settings.json > /dev/null && echo "settings.json valid"
```

Expected: all three print their `valid` message.

**Step 3: Start a new Claude Code session and verify the skills appear**

In a new session, run:
```
/skills
```

The 5 `3dp:*` skills should appear in the skill list.

---

## Notes

- **No git config is set** on this machine, so commit steps are omitted. Set `git config --global user.email` and `git config --global user.name` when ready to commit.
- **Plugin hot-reload**: Claude Code may need to be restarted for the new plugin to appear. Start a fresh session after Task 7.
- **Skill descriptions are the trigger mechanism.** If a skill isn't auto-invoked when expected, the description likely needs stronger trigger phrases — edit `SKILL.md` and restart the session.
