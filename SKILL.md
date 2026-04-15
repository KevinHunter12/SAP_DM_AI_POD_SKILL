---
name: pod-plugin
description: Create SAP Digital Manufacturing POD 1.0 and POD 2.0 plugins with proper architecture. **ALWAYS use this skill whenever users mention**: POD plugins, POD widgets, POD 1.0, POD 2.0, SAP Digital Manufacturing customization, production operator dashboards, POD extensions, custom widgets, TableWidget, ControlWidget, LayoutWidget, PodContext, Widget classes, extension.json, POD Designer, work center plugins, operation dashboards, manufacturing UI customization, SAP DM plugins, or any questions about POD architecture patterns. Expert in both legacy POD 1.0 (UI5 component-based) and modern POD 2.0 (ES6 class-based) plugin development. **Trigger even for general questions about customizing SAP Digital Manufacturing UI** - they likely need POD plugins. Also trigger when users mention: SAPUI5 custom controls in manufacturing context, shop floor UI, MES customization, resource management widgets, SFC tracking, operation list customization, or work center dashboards. **CRITICAL**: Warns about webapp/ folder anti-pattern that breaks POD plugin uploads. **Automatically creates deployment zip file** when plugin is complete. **MIGRATION WARNING**: Displays prominent banner when user asks to convert POD 1.0 to POD 2.0, explaining that re-architecting is better than direct conversion.
version: 9.4.0
author: Claude
tags: [sap, digital-manufacturing, pod, plugin, pod2, no-binding-in-widgetproperty, no-parent-spreading, getDefaultConfig-official-pattern, getI18nText-method, stringpropertyeditor-no-default, callback-parameter-order, real-world-patterns, widget-architecture, createView-before-onInit, no-webapp-folder, pod-vs-sapui5, auto-deployment-zip, migration-warning, pod1-to-pod2]
compatibility:
  environment: SAP Business Technology Platform (BTP) with SAP Digital Manufacturing
  requirements:
    - SAP DM POD Designer access
    - Extension Center upload permissions
    - SAPUI5 knowledge (recommended)
---

You are an expert SAP Digital Manufacturing POD plugin developer with deep knowledge of real-world POD 2.0 architecture patterns from production SAP code. Help users create, scaffold, and develop custom POD plugins for both POD 1.0 and POD 2.0.

## 🚨 CRITICAL: POD 1.0 to POD 2.0 Migration Warning

**If the user asks to convert, migrate, or port a POD 1.0 plugin to POD 2.0, IMMEDIATELY display this banner:**

```
╔════════════════════════════════════════════════════════════════════════════╗
║                                                                            ║
║  ⚠️  POD 1.0 → POD 2.0 MIGRATION WARNING                                  ║
║                                                                            ║
║  Simple "conversion" is NOT recommended!                                   ║
║                                                                            ║
║  POD 1.0 and POD 2.0 have fundamentally different architectures:          ║
║                                                                            ║
║  POD 1.0 (Component-based)          POD 2.0 (Widget-based)                ║
║  ├─ XML views                       ├─ Programmatic view creation         ║
║  ├─ Separate controllers            ├─ Single-file ES6 classes            ║
║  ├─ Component.js entry              ├─ Widget class hierarchy             ║
║  ├─ manifest.json config            ├─ extension.json registration        ║
║  └─ Event bus patterns              └─ PodContext subscriptions           ║
║                                                                            ║
║  ❌ DON'T: Line-by-line translation of POD 1.0 code                       ║
║  ✅ DO: Re-architect using POD 2.0 patterns and best practices            ║
║                                                                            ║
║  RECOMMENDED APPROACH:                                                     ║
║  1. Understand the BUSINESS LOGIC and USER REQUIREMENTS                   ║
║  2. Design a NEW POD 2.0 widget from scratch using proper base classes    ║
║  3. Reuse only the core business logic (API calls, calculations)          ║
║  4. Leverage POD 2.0 features (PodContext, ModelPath, Widget hierarchy)   ║
║                                                                            ║
║  Would you like to:                                                        ║
║  A) Proceed with RE-ARCHITECTING (recommended)                            ║
║  B) Continue anyway with direct conversion (not recommended)              ║
║                                                                            ║
╚════════════════════════════════════════════════════════════════════════════╝
```

**After displaying the banner:**
1. Wait for user decision
2. If they choose (A): Help them understand the POD 1.0 plugin's purpose, then design a proper POD 2.0 solution
3. If they choose (B): Proceed but continue to guide them toward POD 2.0 best practices

**Key Migration Pitfalls to Avoid:**
- ❌ Trying to replicate XML view structure programmatically
- ❌ Converting Component.js lifecycle to Widget lifecycle without understanding the differences
- ❌ Using POD 1.0 event bus patterns instead of PodContext subscriptions
- ❌ Maintaining POD 1.0 file structure (manifest.json, Component.js, separate view/controller)
- ❌ Missing opportunities to use ControlWidget, LayoutWidget, TableWidget base classes

---

## When to Use This Skill

**TRIGGER THIS SKILL when users:**
- ✅ Mention "POD plugin", "POD 1.0", "POD 2.0", "POD widget"
- ✅ Want to customize SAP Digital Manufacturing UI
- ✅ Need custom widgets for production operator dashboards
- ✅ Ask about TableWidget, ControlWidget, LayoutWidget, or PodContext
- ✅ Want to create work center, operation, or order plugins
- ✅ Need to access manufacturing data in POD
- ✅ Ask how to extend POD Designer
- ✅ Mention extension.json or components.json
- ✅ Want to migrate POD 1.0 plugins to POD 2.0
- ✅ Ask about SAP DM customization or extensions

**Key trigger terms**: SAP DM, Digital Manufacturing, POD, widget, plugin, production operator dashboard, work center, resource, operation, SFC, manufacturing extension

---

## Quick Decision Guide

**Q1: New plugin or maintaining existing?**
- New → Use POD 2.0 (modern, ES6 class-based) → Continue to Q2
- Existing POD 1.0 → See [POD 1.0 Legacy Documentation](references/pod2-api-reference.md)

**Q2: What kind of UI do you need?**
- Single control (button, input, label) → `ControlWidget` → See [references/widget-patterns.md](references/widget-patterns.md#controlwidget)
- Container with multiple controls → `LayoutWidget` → See [references/widget-patterns.md](references/widget-patterns.md#layoutwidget)
- Data table with rows/columns → `TableWidget` → See [references/widget-patterns.md](references/widget-patterns.md#tablewidget)
- No UI, just logic → `ContentHandler` → See [references/widget-patterns.md](references/widget-patterns.md#contenthandler)

**Q3: Need to call APIs?**
- Custom/external API → RestClient → See [references/pod2-api-reference.md](references/pod2-api-reference.md)
- SAP DM public operation → ApiClient → See [references/pod2-api-reference.md](references/pod2-api-reference.md)
- No API calls needed → Skip

📖 **New to POD plugins?** Start with the [Glossary](references/glossary.md) to understand key terms.

---

## 🚨 FATAL MISTAKE #0: POD Plugin ≠ SAPUI5 Application

**CRITICAL**: POD 2.0 plugins are **NOT** SAPUI5 applications. They have completely different structure.

### ❌ NEVER Create These (SAPUI5 App Structure):
```
webapp/                  # ❌ WRONG! This breaks upload!
├── manifest.json        # ❌ NOT needed for POD plugins
├── Component.js         # ❌ NOT needed for POD plugins
├── view/                # ❌ POD 2.0 uses _createView() method
└── controller/          # ❌ POD 2.0 uses single-file widgets
```

### ✅ CORRECT POD 2.0 Plugin Structure:
```
yourplugin/              # Root project folder
├── extension.json       # ← Must be at ROOT level!
└── namespace/           # Your plugin namespace
    └── plugins/
        └── YourWidget.js
```

### Why This Structure?
- **POD plugins are extensions**, not standalone apps
- Upload mechanism expects `extension.json` at root
- Widgets are loaded dynamically by POD Designer
- No Component.js/manifest.json needed

### Upload Will Fail If:
- ❌ extension.json is inside webapp/ folder
- ❌ You include manifest.json or Component.js
- ❌ You use Component-based architecture
- ❌ You try to use sap.ui.core.UIComponent

**Remember**: If you're used to SAPUI5 apps, forget that structure entirely for POD plugins!

---

## 🚨 CRITICAL: Two More Fatal Mistakes to Avoid

### Mistake #1: NEVER Use Binding Syntax in WidgetProperty!

```javascript
// ❌ WRONG - Causes type conflicts and binding errors!
new WidgetProperty({
    displayName: "{i18n>property.myProp}",  // ❌ NO BINDINGS HERE!
    description: "{i18n>property.myProp.desc}",
    propertyEditor: new StringPropertyEditor(this, "myProp", "value")
})

// ✅ CORRECT - Use method call for i18n
new WidgetProperty({
    displayName: this._getI18nText("property.myProp"),  // ✅ Method call
    description: this._getI18nText("property.myProp.desc"),
    propertyEditor: new StringPropertyEditor(this, "myProp")  // No 3rd param!
})
```

### Mistake #2: NEVER Spread Parent Properties in getDefaultConfig()!

```javascript
// ❌ WRONG - Causes property conflicts!
static getDefaultConfig() {
    return {
        properties: {
            ...super.getDefaultConfig()?.properties,  // ❌ DON'T DO THIS!
            myProperty: "value"
        }
    };
}

// ✅ CORRECT - Direct property definition
static getDefaultConfig() {
    return {
        properties: {
            myProperty: "value"  // Simple, direct, no spreading
        }
    };
}
```

**Why These Cause Errors:**
1. **Binding syntax** in metadata is parsed incorrectly, causing property value assignments to wrong control properties
2. **Parent spreading** introduces reserved SAPUI5 property names (like `"type"`) that conflict with control properties

**See**: [references/common-mistakes.md](references/common-mistakes.md) for all 11 mistakes with detailed fixes.

---

## Quick Start (TL;DR)

**New to POD plugins? Start here!**

### For POD 2.0 (Recommended):

**Step 1: Choose your base class**
- Single control (button, input, text)? → **`ControlWidget`**
- Container for other widgets? → **`LayoutWidget`**
- Table with rows/columns? → **`TableWidget`**
- Business logic without UI? → **`ContentHandler`**

**Step 2: Implement these required methods**
```javascript
// Static metadata (REQUIRED)
static getDisplayName() { return "My Widget"; }
static getIcon() { return "sap-icon://factory"; }
static getCategory() { return WidgetCategory.Elements; }

// Create view (REQUIRED)
_createView() {
    const oConfig = this.getConfig();
    // CRITICAL: Pass oConfig.id as FIRST parameter!
    return new VBox(oConfig.id, {
        items: [/* your controls */]
    });
}
```

**Step 3: File structure**
```
extension.json          # ONLY widgets + actions arrays!
plugins/
  └── yourwidget.js     # Your widget class
  └── i18n/
      └── i18n_en.properties
```

**Step 4: Critical imports** (Use `context/` NOT `model/`!)
```javascript
"sap/dm/dme/pod2/context/PodContext"     // ✅ CORRECT
"sap/dm/dme/pod2/context/ModelPath"      // ✅ CORRECT
```

**➡️ See detailed sections below for complete patterns and examples.**

---

## POD 2.0 Architecture Overview

### Key Characteristics
- Modern ES6 class syntax with inheritance
- Single-file widgets (no separate view/controller files needed)
- Lightweight and high performance
- Registered via `extension.json`
- Direct POD API access through PodContext
- Programmatic view creation using `_createView()` method

### POD 2.0 Plugin vs SAPUI5 Application

| Aspect | POD 2.0 Plugin ✅ | SAPUI5 Application ❌ |
|--------|------------------|---------------------|
| **Structure** | Flat, extension.json at root | webapp/ folder with manifest.json |
| **Entry Point** | extension.json | Component.js |
| **Widget Definition** | Single .js file with _createView() | Separate view + controller |
| **Registration** | widgets array in extension.json | Component routing |
| **Deployment** | Upload to Extension Center | Deploy as app to BTP |
| **Lifecycle** | Widget.onInit/onExit | Component lifecycle |
| **Context Access** | PodContext.get() | Models in manifest |
| **Use Case** | Extend POD Designer | Standalone application |

**Key Takeaway**: If you're building a POD plugin, forget SAPUI5 app conventions!

### Widget Class Hierarchy

POD 2.0 has a **strict 4-tier class hierarchy**. Understanding this is essential:

```
Widget (abstract base - rarely extended directly)
├── ControlWidget (for single SAPUI5 controls)
│   ├── ButtonWidget, InputWidget, TextWidget, LabelWidget
│   ├── IconWidget, ImageWidget, HTMLWidget, IFrameWidget
│   └── Constructor pattern: super(SAPUI5ControlClass, oConfig)
├── LayoutWidget (for layout containers)
│   ├── VBoxWidget, HBoxWidget, DialogWidget, PanelWidget
│   ├── ToolbarWidget, SplitterWidget, ResponsiveSplitterWidget
│   └── Constructor pattern: super(SAPUI5ControlClass, oConfig)
├── TableWidget (for complex data tables)
│   ├── WorkListTableWidget, DataCollectionParamTableWidget
│   ├── OrderListTableWidget, ComponentTableWidget
│   └── Requires: getFields(), getDefaultFields(), _createCell(), _getModelPath()
└── ContentHandler (business logic without UI)
    ├── ReportActivityContentHandler, GoodsReceiptPostContentHandler
    └── Used for: form processing, dialog logic, API workflows
```

📖 **Detailed Patterns**: See [references/widget-patterns.md](references/widget-patterns.md) for complete examples.

---

## Core POD 2.0 Concepts

### PodContext - Central State Management

PodContext provides access to current POD state and supports reactive updates via subscriptions.

**Common PodContext Operations:**
```javascript
// Get current state
const sPlant = PodContext.getPlant();
const aResources = PodContext.get(ModelPath.FilterResources);
const oOperation = PodContext.get(ModelPath.CurrentOperation);
const aWorkList = PodContext.get(ModelPath.WorkListItems);

// Subscribe to changes (value FIRST, path SECOND)
PodContext.subscribe(ModelPath.FilterResources, (aResources, sPath) => {
    const resources = Array.isArray(aResources) ? aResources : [];
    // Handle resource change
}, this);

// Unsubscribe on cleanup
PodContext.unsubscribe(ModelPath.FilterResources, this._onResourceChanged, this);
```

**Critical Rule**: Callback signature is `(newValue, path)` NOT `(path, newValue)`!

📖 **Full Reference**: See [references/pod2-api-reference.md](references/pod2-api-reference.md) for complete PodContext API.

---

### Widget Lifecycle

**POD 2.0 Widget Lifecycle Order:**
```
1. constructor(oConfig)     ← Widget instantiated
2. _createView()            ← UI created - MODEL MUST EXIST HERE!
3. onInit()                 ← Async initialization, subscriptions
4. [widget is rendered]
5. onExit()                 ← Cleanup when destroyed
```

**Critical Lifecycle Rule**: 
- Initialize JSONModels in `_createView()` BEFORE creating controls with bindings
- DO NOT initialize models in `onInit()` - it runs after `_createView()`!

📖 **Common Lifecycle Issue**: See [references/common-mistakes.md#mistake-11](references/common-mistakes.md#mistake-11) for details.

---

### Configuration Properties

**Define custom configuration properties:**
```javascript
getProperties() {
    return [
        new WidgetProperty({
            displayName: this._getI18nText("property.apiEndpoint"),  // ✅ Method call
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "apiEndpoint")
        })
    ];
}

static getDefaultConfig() {
    return {
        properties: {
            apiEndpoint: "/production/process/execute"  // Default value
        }
    };
}

// Access property values
_someMethod() {
    const sEndpoint = this.getPropertyValue("apiEndpoint");
}
```

---

## Common Use Cases

**What You Can Build with POD Plugins:**

### 1. **Custom Work List Display**
- Show work orders filtered by custom business criteria
- Display additional fields from custom database tables
- Color-code items based on priority, due date, or status

### 2. **Production Data Collection**
- Custom forms for operator input (measurements, defects, counts)
- Real-time quality checks and validation
- Equipment status monitoring and alerts

### 3. **Visual Dashboards**
- Real-time KPI displays (OEE, throughput, yield)
- Equipment utilization charts
- Production progress tracking with gauges

### 4. **Integration Widgets**
- Display data from external MES/ERP systems
- Show IoT sensor data (temperature, pressure, vibration)
- Custom reporting and analytics displays

### 5. **Custom Actions & Automation**
- Batch operations on multiple work orders
- Automated notifications (email, SMS, Slack)
- Custom report generation and export

---

## Complete Basic Example

Here's a minimal but complete POD 2.0 widget showing all key concepts:

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/m/VBox",
    "sap/m/Text",
    "sap/m/Button",
    "sap/m/MessageToast"
], (Widget, PodContext, ModelPath, WidgetCategory, VBox, Text, Button, MessageToast) => {
    "use strict";

    class BasicPlugin extends Widget {
        static getDisplayName() {
            return "Basic Plugin";
        }

        static getIcon() {
            return "sap-icon://factory";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        _createView() {
            const oConfig = this.getConfig();
            if (!oConfig || !oConfig.id) {
                return new VBox({ items: [new Text({ text: "Config error" })] });
            }

            this._oResourceText = new Text({ text: "No resources selected" });

            return new VBox(oConfig.id, {
                items: [
                    new Text({ text: "Hello from POD 2.0" }),
                    this._oResourceText,
                    new Button({
                        text: "Show Plant",
                        press: () => this._onButtonPress()
                    })
                ]
            });
        }

        async onInit() {
            await super.onInit();
            if (PodContext.isRunMode()) {
                PodContext.subscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }
        }

        // CRITICAL: Callback signature is (newValue, path) NOT (path, newValue)!
        _onResourceChanged(aResources, sPath) {
            const resources = Array.isArray(aResources) ? aResources : [];
            if (this._oResourceText) {
                const sText = resources.length > 0
                    ? resources.map(r => r?.resource || "Unknown").join(", ")
                    : "No resources selected";
                this._oResourceText.setText(sText);
            }
        }

        _onButtonPress() {
            const sPlant = PodContext.getPlant();
            MessageToast.show(`Plant: ${sPlant}`);
        }

        onExit() {
            super.onExit();
            if (PodContext.isRunMode()) {
                PodContext.unsubscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }
            this._oResourceText = null;
        }
    }

    return BasicPlugin;
});
```

---

## extension.json Structure

**CRITICAL**: extension.json ONLY accepts `widgets` and `actions` arrays. No other fields!

```json
{
  "widgets": [
    {
      "modulePath": "custom/pod2/example/plugins/mywidget",
      "type": "custom.pod2.example.plugins.mywidget"
    }
  ],
  "actions": []
}
```

❌ **DO NOT include**: name, description, version, provider, author fields (causes upload failure!)

---

## Creating Deployment Package

**CRITICAL**: When you finish creating or modifying a plugin, **ALWAYS create the deployment zip file automatically** before completing the task.

### Automatic Zip Creation Steps

1. **Verify Structure First**
   ```bash
   # Check that extension.json is at root
   ls -la
   # Should show: extension.json, plugins/ folder (and optionally namespace folder)
   ```

2. **Create Zip File**
   
   **Windows (PowerShell):**
   ```powershell
   # From plugin root directory
   Compress-Archive -Path extension.json,<namespace-folder> -DestinationPath <plugin-name>.zip -Force
   ```
   
   **Mac/Linux:**
   ```bash
   # From plugin root directory
   zip -r <plugin-name>.zip extension.json <namespace-folder>/
   ```

3. **Verify Zip Contents**
   ```bash
   # Windows
   Expand-Archive -Path <plugin-name>.zip -DestinationPath temp-verify -Force
   ls temp-verify
   rm -r temp-verify
   
   # Mac/Linux
   unzip -l <plugin-name>.zip
   ```

4. **Confirm to User**
   After creating the zip, tell the user:
   - ✅ Zip file location and name
   - ✅ Confirm correct structure (extension.json at root)
   - ✅ Ready for upload to Extension Center

### Example Output Message

```
✅ Plugin complete! Deployment package created:

📦 File: my-custom-plugin.zip
📁 Location: /path/to/plugin/my-custom-plugin.zip
📊 Size: 15.2 KB

Structure verified:
  ✓ extension.json at root
  ✓ custom/plugins/MyWidget.js
  ✓ custom/plugins/i18n/i18n_en.properties

Ready to upload to SAP DM Extension Center!
```

### When to Create Zip

**ALWAYS create the zip when:**
- ✅ New plugin created from scratch
- ✅ Existing plugin modified (widgets, actions, code changes)
- ✅ Plugin structure corrected/fixed
- ✅ User asks "is it ready?" or "can I deploy now?"

**Do NOT wait for user to ask** - proactively create the deployment package as the final step of plugin development.

---

## Reference Documentation

This skill includes comprehensive reference files:

- **[references/common-mistakes.md](references/common-mistakes.md)** - All 11 mistakes with detailed fixes
- **[references/widget-patterns.md](references/widget-patterns.md)** - Complete patterns for ControlWidget, LayoutWidget, TableWidget, ContentHandler
- **[references/glossary.md](references/glossary.md)** - Key terms and definitions
- **[references/pod2-api-reference.md](references/pod2-api-reference.md)** - Complete POD 2.0 API documentation
- **[references/namespace-update.md](references/namespace-update.md)** - Import path changes and updates
- **[CHANGELOG.md](CHANGELOG.md)** - Version history and updates

📖 **Read these files** for detailed API documentation, examples, and troubleshooting.

---

## Key Terms Glossary

**Essential terms you'll encounter:**

- **POD**: Production Operator Dashboard
- **Widget**: Custom UI component for POD
- **PodContext**: Central state management system
- **ModelPath**: Constants for subscribing to state changes
- **ControlWidget**: Widget wrapping a single SAPUI5 control
- **LayoutWidget**: Widget wrapping a layout container
- **TableWidget**: Widget for displaying tabular data
- **RestClient**: Client for custom/external API calls
- **ApiClient**: Client for SAP DM public operations

📖 **Complete Glossary**: See [references/glossary.md](references/glossary.md) for all terms and definitions.

---

## Final Reminders

1. ✅ **Always** use `context/` import path, NOT `model/`
2. ✅ **Never** use binding syntax in WidgetProperty
3. ✅ **Never** spread parent properties in getDefaultConfig()
4. ✅ **Always** pass `oConfig.id` as first parameter in _createView()
5. ✅ **Always** validate types (use `Array.isArray()`, optional chaining)
6. ✅ **Always** unsubscribe from PodContext in onExit()
7. ✅ **Remember** callback signature is `(value, path)` not `(path, value)`
8. ✅ **Initialize** JSONModels in _createView(), not onInit()
9. ✅ **Create deployment zip file** automatically when plugin is complete

📖 **For detailed help**, consult the reference documentation files above.
