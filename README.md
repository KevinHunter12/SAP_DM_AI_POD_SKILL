# SAP Digital Manufacturing POD 2.0 Plugin Skill for Claude Code

A Claude Code skill that enables AI-assisted development of SAP Digital Manufacturing (SAP DM) POD 2.0 plugins. This skill gives Claude deep, production-verified knowledge of the POD 2.0 widget architecture so it can scaffold complete, working plugins from a single prompt.

> **Please note that this skill is an independent community project and is not an official SAP release. As such, it is provided without warranty, support, or endorsement from SAP.**

---

## What This Skill Does

When you describe a POD plugin in plain language, Claude will:

- Ask for your namespace, plugin name, category, and i18n language requirements
- Select the correct base class (`ControlWidget`, `LayoutWidget`, or `TableWidget`)
- Generate all required files (JS, XML view, i18n properties, `extension.json`) at the **project root** — never inside a `webapp/` folder
- Apply production-correct import paths, adaptive PodContext patterns, and proper delegate usage
- Avoid the 14+ most common mistakes that cause runtime errors in SAP DM

### Triggers

Claude activates this skill when you mention: POD plugins, POD widgets, POD 2.0, SAP DM customization, TableWidget, ControlWidget, LayoutWidget, PodContext, extension.json, work center plugins.

---

## Installation

### Prerequisites

- [Claude Code CLI](https://claude.ai/code) installed
- Git

### Step 1 — Clone this repository

```bash
git clone https://github.com/KevinHunter12/SAP_DM_AI_POD_SKILL.git
```

### Step 2 — Register the skill with Claude Code

Copy the skill folder into your Claude Code skills directory:

**macOS / Linux**
```bash
cp -r SAP_DM_AI_POD_SKILL ~/.claude/skills/pod-plugin
```

**Windows (PowerShell)**
```powershell
Copy-Item -Recurse SAP_DM_AI_POD_SKILL "$env:USERPROFILE\.claude\skills\pod-plugin"
```

> The folder name inside `~/.claude/skills/` must be `pod-plugin` — that is the registered skill name.

### Step 3 — Verify installation

Open a new Claude Code session and run:

```
/pod-plugin
```

If the skill is installed correctly, Claude will respond with guidance on creating a POD 2.0 plugin.

---

## Usage

Start a Claude Code session in your plugin project directory, then describe what you want to build:

```
Create a POD plugin that displays the current SFC status and lets the operator start production.
```

Claude will ask four questions before generating any files:

1. **Namespace** — e.g. `custom/pod2/myproject`
2. **Plugin Name** — e.g. `ProductionStatus`
3. **Category** — default: `Custom`
4. **i18n Languages** — core (EN/DE/ZH/JA), extended (9 languages), all 28 SAP DM languages, or custom

Once you answer, Claude generates the complete plugin package ready to zip and upload to the SAP DM Extension Center.

---

## Skill Structure

```
pod-plugin/
├── SKILL.md                        # Skill definition and critical rules
└── references/                     # Knowledge base read by Claude before generation
    ├── QUICK-REFERENCE.md          # Essential patterns (read first)
    ├── widget-patterns.md          # Base class templates
    ├── common-mistakes.md          # 14+ verified production mistakes to avoid
    ├── PATTERN-INDEX.md            # Pattern selection guide
    ├── tablewidget-complete.md     # TableWidget-specific patterns
    ├── tablecell-patterns.md       # Custom table cell patterns
    ├── delegate-architecture.md    # Delegate usage patterns
    ├── delegates-reference.md      # All delegates API reference (source-derived)
    ├── podcontext-api-reference.md # PodContext deep reference (source-derived)
    ├── pod2-api-reference.md       # PodContext + ModelPath comprehensive reference
    ├── utilities-reference.md      # DateTimeUtils, Logger, ValidationUtils, etc.
    ├── sapdm-api-reference.md      # SAP DM REST APIs
    ├── sapui5-control-apis.md      # SAPUI5 control quirks and styling
    ├── sap-dm-languages.md         # All 28 supported languages with ISO codes
    ├── error-handling.md           # Error decision matrix
    ├── binding-patterns.md         # Data binding patterns
    ├── form-dialog-patterns.md     # Form and dialog patterns
    ├── notification-patterns.md    # Notification and messaging patterns
    ├── advanced-patterns.md        # Advanced plugin patterns
    ├── glossary.md                 # SAP DM terminology
    └── production-patterns-unified.md  # Unified production patterns
```

---

## SAP DM Requirements

To deploy generated plugins you need:

- SAP Business Technology Platform (BTP) with SAP Digital Manufacturing
- SAP DM POD Designer access
- Extension Center upload permissions

---

## License

MIT — see [LICENSE](LICENSE) for details.
