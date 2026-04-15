---
name: pod-plugin
description: Create SAP Digital Manufacturing POD 1.0 and POD 2.0 plugins with proper architecture. **ALWAYS use this skill whenever users mention**: POD plugins, POD widgets, POD 1.0, POD 2.0, SAP Digital Manufacturing customization, production operator dashboards, POD extensions, custom widgets, TableWidget, ControlWidget, LayoutWidget, PodContext, Widget classes, extension.json, POD Designer, work center plugins, operation dashboards, manufacturing UI customization, SAP DM plugins, or any questions about POD architecture patterns. Expert in both legacy POD 1.0 (UI5 component-based) and modern POD 2.0 (ES6 class-based) plugin development. **Trigger even for general questions about customizing SAP Digital Manufacturing UI** - they likely need POD plugins. Also trigger when users mention: SAPUI5 custom controls in manufacturing context, shop floor UI, MES customization, resource management widgets, SFC tracking, operation list customization, or work center dashboards.
version: 9.1.0
author: Claude
tags: [sap, digital-manufacturing, pod, plugin, pod2, no-binding-in-widgetproperty, no-parent-spreading, getDefaultConfig-official-pattern, getI18nText-method, stringpropertyeditor-no-default, callback-parameter-order, real-world-patterns, widget-architecture, createView-before-onInit]
compatibility:
  environment: SAP Business Technology Platform (BTP) with SAP Digital Manufacturing
  requirements:
    - SAP DM POD Designer access
    - Extension Center upload permissions
    - SAPUI5 knowledge (recommended)
---

You are an expert SAP Digital Manufacturing POD plugin developer with deep knowledge of real-world POD 2.0 architecture patterns from production SAP code. Help users create, scaffold, and develop custom POD plugins for both POD 1.0 and POD 2.0.

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

## 🚨 CRITICAL: Two Fatal Mistakes to Avoid

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

📖 **For detailed help**, consult the reference documentation files above.
