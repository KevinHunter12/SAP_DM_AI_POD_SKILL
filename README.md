# SAP Digital Manufacturing POD Plugin Skill

A Claude Code skill for creating SAP Digital Manufacturing POD 1.0 and POD 2.0 plugins with correct architecture patterns.

## What This Skill Does

This skill transforms Claude into an expert SAP Digital Manufacturing POD plugin developer. It provides:

- **POD 2.0** (modern ES6 class-based) plugin development guidance
- **POD 1.0** (legacy SAPUI5 component-based) plugin support
- Critical mistake prevention (binding syntax, property spreading, import paths)
- Real-world patterns from production SAP code
- Complete scaffolding for `ControlWidget`, `LayoutWidget`, `TableWidget`, and `ContentHandler`

## Installation

### Step 1: Locate your Claude skills directory

Claude Code stores skills in:

```
~/.claude/skills/
```

On Windows:
```
C:\Users\<YourUsername>\.claude\skills\
```

### Step 2: Clone or copy the skill

**Option A – Clone from GitHub:**

```bash
git clone https://github.tools.sap/I331794/claudePODskill.git ~/.claude/skills/pod-plugin
```

**Option B – Copy manually:**

Copy the contents of this repository into a folder named **`pod-plugin`** inside your skills directory.

> **Important:** The folder must be named `pod-plugin` exactly. Claude Code uses the folder name to identify and load the skill.

### Step 3: Verify installation

```bash
# macOS / Linux
ls ~/.claude/skills/pod-plugin/SKILL.md

# Windows (PowerShell)
Test-Path "$env:USERPROFILE\.claude\skills\pod-plugin\SKILL.md"
```

### Step 4: Restart Claude Code

If Claude Code is already running, restart it so it picks up the new skill.

## Usage

Once installed, Claude will automatically apply this skill whenever you mention:

- POD plugin, POD widget, POD 1.0, POD 2.0
- SAP Digital Manufacturing customization
- Production Operator Dashboard (POD)
- `extension.json`, `components.json`
- `ControlWidget`, `LayoutWidget`, `TableWidget`, `PodContext`
- Work center plugins, operation dashboards
- SAP DM plugins or extensions

### Example prompts

```
Create a POD 2.0 ControlWidget that displays the current resource name
```

```
Scaffold a TableWidget that shows the SFC work list for the selected operation
```

```
How do I access the selected SFC in a POD 2.0 plugin?
```

```
Build a LayoutWidget with a refresh button and a data table
```

## Key Features

| Feature | Details |
|---------|---------|
| POD Version | POD 1.0 (legacy) and POD 2.0 (recommended) |
| Widget Types | ControlWidget, LayoutWidget, TableWidget, ContentHandler |
| Critical Patterns | No binding syntax in WidgetProperty, no parent spreading in getDefaultConfig |
| Import Paths | Correct `pod2/context/` paths to avoid 404 errors |
| Real-World Examples | Production patterns from actual SAP Digital Manufacturing code |
| Skill Version | 9.1.0 |

## Requirements

- [Claude Code](https://docs.anthropic.com/claude-code) installed
- Access to SAP Digital Manufacturing for deploying plugins
- SAP DM 2023.x or later for POD 2.0 features

## Folder Structure

```
pod-plugin/
├── SKILL.md                    # Main skill file (462 lines)
├── README.md                   # This file
├── CHANGELOG.md                # Version history
├── .gitignore                  # Git ignore rules
├── references/                 # Reference documentation
│   ├── common-mistakes.md      # 11 common mistakes with fixes
│   ├── glossary.md             # Key terms and definitions
│   ├── widget-patterns.md      # Complete widget code patterns
│   ├── pod2-api-reference.md   # Complete POD 2.0 API docs
│   └── namespace-update.md     # Import path changes
└── other-files/                # Non-essential files (gitignored)
    └── ...                     # Backups, old versions, scripts
```

## Reference Documentation

| File | Description |
|------|-------------|
| [references/common-mistakes.md](references/common-mistakes.md) | All 11 common mistakes with fixes |
| [references/glossary.md](references/glossary.md) | Key terms and definitions |
| [references/widget-patterns.md](references/widget-patterns.md) | Complete ControlWidget, LayoutWidget, TableWidget patterns |
| [references/pod2-api-reference.md](references/pod2-api-reference.md) | Complete POD 2.0 API documentation |
| [references/namespace-update.md](references/namespace-update.md) | Import path changes and updates |

## Troubleshooting

**Skill not loading?**
- Confirm the folder is named `pod-plugin` (case-sensitive on macOS/Linux)
- Confirm `SKILL.md` is present at the root of the folder
- Restart Claude Code after adding the skill

**Git push issues?**
- Ensure you have committed files first: `git add . && git commit -m "Initial commit"`
- Push with: `git push -u origin main`

## License

Internal SAP tooling – for use within SAP development environments.
