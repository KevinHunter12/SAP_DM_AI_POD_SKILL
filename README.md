# SAP Digital Manufacturing POD Plugin Skill

A Claude Code CLI skill for creating SAP Digital Manufacturing POD 1.0 and POD 2.0 plugins with correct architecture patterns.

## What This Skill Does

This skill transforms Claude into an expert SAP Digital Manufacturing POD plugin developer. It provides:

- **POD 2.0** (modern ES6 class-based) plugin development guidance
- **POD 1.0** (legacy SAPUI5 component-based) plugin support
- Critical mistake prevention (binding syntax, property spreading, import paths)
- Real-world patterns from production SAP code
- Complete scaffolding for `ControlWidget`, `LayoutWidget`, `TableWidget`, and `ContentHandler`

## Installation

### Step 1: Locate your Claude skills directory

Claude Code CLI stores skills in:

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

Copy the contents of this repository into a folder named **`pod-plugin`** inside your skills directory:

```
~/.claude/skills/
└── pod-plugin/          ← folder must be named exactly "pod-plugin"
    ├── SKILL.md
    ├── CHANGELOG.md
    ├── references/
    └── ...
```

> **Important:** The folder must be named `pod-plugin` exactly. Claude Code CLI uses the folder name to identify and load the skill.

### Step 3: Verify installation

In a terminal, check that the skill file exists:

```bash
# macOS / Linux
ls ~/.claude/skills/pod-plugin/SKILL.md

# Windows (PowerShell)
Test-Path "$env:USERPROFILE\.claude\skills\pod-plugin\SKILL.md"
```

### Step 4: Restart Claude Code CLI

If Claude Code CLI is already running, restart it so it picks up the new skill.

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
| Skill Version | 8.0.0 |

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/claude-code) installed
- Access to SAP Digital Manufacturing for deploying plugins
- SAP DM 2023.x or later for POD 2.0 features

## Folder Structure Reference

After installation your skills directory should look like:

```
~/.claude/skills/
└── pod-plugin/
    ├── SKILL.md                          # Main skill definition (loaded by Claude)
    ├── CHANGELOG.md                      # Version history
    ├── README.md                         # This file
    ├── NAMESPACE-UPDATE.md
    ├── POD2-API-REFERENCE.md
    ├── VERSION-3.0.0-UPDATES.md
    ├── VERSION-3.1.0-UPDATES.md
    ├── VERSION-3.2.0-UPDATES.md
    └── references/
        ├── ApiClient-Reference.md
        ├── complete-patterns.md
        ├── production-patterns-sap.md
        └── troubleshooting-flowchart.md
```

## Troubleshooting

**Skill not loading?**
- Confirm the folder is named `pod-plugin` (case-sensitive on macOS/Linux)
- Confirm `SKILL.md` is present at the root of the folder
- Restart Claude Code CLI after adding the skill

**Git push issues?**
- Ensure you have committed files first: `git add . && git commit -m "Initial commit"`
- Push with: `git push -u origin main`

## License

Internal SAP tooling – for use within SAP development environments.