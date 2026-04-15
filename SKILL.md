---
name: pod-plugin
description: Create SAP Digital Manufacturing POD 1.0 and POD 2.0 plugins with proper architecture. **ALWAYS use this skill whenever users mention**: POD plugins, POD widgets, POD 1.0, POD 2.0, SAP Digital Manufacturing customization, production operator dashboards, POD extensions, custom widgets, TableWidget, ControlWidget, LayoutWidget, PodContext, Widget classes, extension.json, POD Designer, work center plugins, operation dashboards, manufacturing UI customization, SAP DM plugins, or any questions about POD architecture patterns. Expert in both legacy POD 1.0 (UI5 component-based) and modern POD 2.0 (ES6 class-based) plugin development. **Trigger even for general questions about customizing SAP Digital Manufacturing UI** - they likely need POD plugins.
version: 8.0.0
author: Claude
tags: [sap, digital-manufacturing, pod, plugin, pod2, no-binding-in-widgetproperty, no-parent-spreading, getDefaultConfig-official-pattern, getI18nText-method, stringpropertyeditor-no-default, callback-parameter-order, real-world-patterns, widget-architecture, createView-before-onInit]
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

## 🚨 CRITICAL: Two Fatal Mistakes to Avoid (Updated 2026-04-12)

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

**Common Error Message:**
```
"[value] is of type string, expected sap.m.InputType for property "type"
```

**Official SAP Pattern Sources:**
- SAP Help Portal: "Add Widget Properties"
- URL: https://help.sap.com/docs/help/95abdf318cec40bb84bc487fdaa03691/8dbdab1343184bf19ed36cf26f6aaf08.html

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

## Document Navigation

| Topic | Section | What You'll Learn |
|-------|---------|-------------------|
| **Real-World Examples** | Real-World Use Cases | What you can build |
| **Top Mistakes** | Top 9 Mistakes | Common pitfalls with fixes |
| **Decision Help** | Decision Tree | Choose the right approach |
| **POD 2.0 Basics** | POD 2.0 Architecture | Modern approach (recommended) |
| **Import Paths** | Correct POD 2.0 Import Paths | CRITICAL - avoid 404 errors |
| **PodContext** | POD 2.0 Context Access | State management & subscriptions |
| **ControlWidget** | ControlWidget Pattern | Single control pattern |
| **LayoutWidget** | LayoutWidget Pattern | Container pattern |
| **TableWidget** | TableWidget Pattern | Complex table pattern |
| **Troubleshooting** | Common Issues and Solutions | Debug & fix problems |
| **Testing** | Testing Your Plugin | Before & after upload |
| **Migration** | Migration Path | POD 1.0 → POD 2.0 |
| **POD 1.0 Legacy** | POD 1.0 Architecture | Legacy approach |

---

## Real-World Use Cases

**What You Can Build with POD Plugins:**

### 1. **Custom Work List Display**
- Show work orders filtered by custom business criteria
- Display additional fields from custom database tables
- Color-code items based on priority, due date, or status
- Add custom sorting and filtering options

### 2. **Production Data Collection**
- Custom forms for operator input (measurements, defects, counts)
- Real-time quality checks and validation
- Equipment status monitoring and alerts
- Scan barcode/RFID for material tracking

### 3. **Visual Dashboards**
- Real-time KPI displays (OEE, throughput, yield)
- Equipment utilization charts
- Production progress tracking with gauges
- Live status boards for work center visibility

### 4. **Integration Widgets**
- Display data from external MES/ERP systems
- Show IoT sensor data (temperature, pressure, vibration)
- Integration with quality management systems
- Custom reporting and analytics displays

### 5. **Custom Actions & Automation**
- Batch operations on multiple work orders
- Automated notifications (email, SMS, Slack)
- Custom report generation and export
- Workflow automation triggers

**Example**: A custom widget that displays machine status from IoT sensors, allows operators to log downtime reasons, and automatically creates maintenance notifications in SAP.

---

## Top 10 Mistakes (with Fixes)

### Mistake #1: Using Binding Syntax in WidgetProperty Metadata ❌ → ✅

**Error**: `"/production/process/execute" is of type string, expected sap.m.InputType for property "type"`

This error occurs when using i18n binding syntax in `WidgetProperty` displayName or description fields.

```javascript
// ❌ WRONG - Binding syntax NOT supported in WidgetProperty!
getProperties() {
    return [
        new WidgetProperty({
            displayName: "{i18n>property.myProp}",  // 💥 Causes binding confusion!
            description: "{i18n>property.myProp.description}",
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "myProp", "defaultValue")
        })
    ];
}

// ✅ CORRECT - Use getI18nText() method (Official SAP pattern)
getProperties() {
    return [
        new WidgetProperty({
            displayName: this._getI18nText("property.myProp"),  // ✅ Method call
            description: this._getI18nText("property.myProp.description"),
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "myProp")  // No default value!
        })
    ];
}

// Helper method (Widget base class provides this, or create your own)
_getI18nText(sKey, aParams) {
    try {
        const oResourceBundle = this.getView()?.getModel("i18n")?.getResourceBundle();
        if (oResourceBundle) {
            return oResourceBundle.getText(sKey, aParams);
        }
    } catch (oError) {
        this.#oLog.warn(`Failed to get i18n text for key: ${sKey}`, oError);
    }
    return sKey;  // Fallback to key
}
```

**Why this happens**: 
1. Binding syntax `"{i18n>property.apiEndpoint}"` contains the word "property"
2. POD 2.0 property editor framework parses this string looking for property references
3. The parser finds `"property.apiEndpoint"` and tries to resolve it as a path
4. This creates incorrect bindings that assign your property value to the wrong control property (like `type`)
5. The `CustomInput` control's `type` property expects `InputType` enum, but gets a string instead

**Critical Rule**: `WidgetProperty` is a **metadata definition class**, NOT part of SAPUI5 binding context!
- ✅ Use bindings in: SAPUI5 controls (in `_createView()`), XML views
- ❌ DON'T use bindings in: `WidgetProperty` constructor, metadata definitions

**Official SAP Documentation Note:**
> "The getProperties example above uses a hard coded display name and description in English. To enable support in other languages, the widget should **use the getI18nText method** provided by the Widget base class to get a translated string."

**Source**: SAP Help Portal - "Add Widget Properties"  
https://help.sap.com/docs/help/95abdf318cec40bb84bc487fdaa03691/8dbdab1343184bf19ed36cf26f6aaf08.html

---

### Mistake #2: Wrong PodContext Import Path ❌ → ✅

**Error**: `404 - Failed to load PodContext.js`

```javascript
// ❌ WRONG - These paths don't exist!
import PodContext from "sap/dm/dme/pod2/model/PodContext";
import ModelPath from "sap/dm/dme/pod2/model/ModelPath";

// ✅ CORRECT - Use context/ not model/
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";
```

**Why this happens**: SAP moved these to the `context/` package but many examples still show the old path.

---

### Mistake #2: Passing Default Value to StringPropertyEditor ❌ → ✅

**Issue**: Passing a default value as the 3rd parameter to `StringPropertyEditor` can cause issues.

```javascript
// ❌ WRONG - Don't pass default value to property editor
new StringPropertyEditor(
    this,
    "apiEndpoint",
    "/production/process/execute"  // ❌ Don't pass this!
)

// ✅ CORRECT - Let default come from getDefaultConfig()
new StringPropertyEditor(
    this,
    "apiEndpoint"  // Only 2 parameters!
)
```

**Why**: Default values should be defined in `getDefaultConfig()` and handled by `getPropertyValue()`, not passed to the property editor constructor.

**Official SAP Pattern:**
```javascript
// Define default in getDefaultConfig()
static getDefaultConfig() {
    return {
        properties: {
            apiEndpoint: "/production/process/execute"
        }
    };
}

// Handle runtime defaults in getPropertyValue()
getPropertyValue(sName) {
    const vValue = super.getPropertyValue(sName);
    switch (sName) {
        case "apiEndpoint":
            return vValue || "/production/process/execute";
    }
    return vValue;
}
```

---

### Mistake #3: Wrong Callback Parameter Order ❌ → ✅

**Error**: `TypeError: aResources.map is not a function`

```javascript
// ❌ WRONG - Parameters in wrong order!
PodContext.subscribe(ModelPath.FilterResources, (sPath, aResources) => {
    // sPath is actually the DATA array!
    // aResources is actually the PATH string!
    const list = aResources.map(r => r.resource); // 💥 CRASH!
});

// ✅ CORRECT - Data FIRST, path SECOND
PodContext.subscribe(ModelPath.FilterResources, (aResources, sPath) => {
    // aResources is the DATA (first parameter)
    // sPath is the PATH (second parameter)
    const resources = Array.isArray(aResources) ? aResources : [];
    const list = resources.map(r => r?.resource || "Unknown");
});
```

**Critical Rule**: Callback signature is `(newValue, path)` NOT `(path, newValue)`!

**Why this happens**: Most frameworks use (path, value) order, but PodContext uses (value, path).

---

### Mistake #3: Missing View ID in _createView() ❌ → ✅

**Error**: `"getView method returned a view with a different ID than configuration"`

```javascript
// ❌ WRONG - No ID passed to view
_createView() {
    return new VBox({
        items: [new Text({ text: "Hello" })]
    });
}

// ✅ CORRECT - Pass oConfig.id as FIRST parameter
_createView() {
    const oConfig = this.getConfig();

    if (!oConfig || !oConfig.id) {
        return new VBox({
            items: [new Text({ text: "Configuration error" })]
        });
    }

    // Pass ID as first parameter (UI5 constructor pattern)
    return new VBox(oConfig.id, {
        items: [new Text({ text: "Hello" })]
    });
}
```

**Why this happens**: POD 2.0 requires the view ID to match the widget configuration ID for proper lifecycle management.

---

### Mistake #4: No Defensive Type Checking ❌ → ✅

**Error**: `Cannot read property 'map' of undefined`

```javascript
// ❌ WRONG - Assumes data is always an array
_onResourceChanged(aResources, sPath) {
    const list = aResources.map(r => r.resource).join(", ");
    this._oText.setText(list);
}

// ✅ CORRECT - Always validate types!
_onResourceChanged(aResources, sPath) {
    // Step 1: Coerce to array type
    const resources = Array.isArray(aResources) ? aResources : [];

    // Step 2: Use optional chaining for properties
    const list = resources.length > 0
        ? resources.map(r => r?.resource || "Unknown").join(", ")
        : "No resources selected";

    // Step 3: Safely update UI
    if (this._oText) {
        this._oText.setText(list);
    }
}
```

**Why this happens**: PodContext can send `undefined`, `null`, `[]`, or non-array types depending on state.

---

### Mistake #5: Invalid extension.json Structure ❌ → ✅

**Error**: `"Failed to create custom extensions: Error encountered when processing the extension components file"`

```javascript
// ❌ WRONG - Extra metadata fields cause upload failure
{
  "name": "My Plugin",           // ❌ NOT SUPPORTED
  "description": "...",           // ❌ NOT SUPPORTED
  "version": "1.0.0",             // ❌ NOT SUPPORTED
  "provider": "Acme Corp",        // ❌ NOT SUPPORTED
  "widgets": [...]
}

// ✅ CORRECT - ONLY widgets and actions arrays!
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

**Why this happens**: SAP DM Extension Center only accepts `widgets` and `actions` arrays. All other fields cause parsing errors.

---

### Mistake #6: Spreading Parent Properties in getDefaultConfig() ❌ → ✅

**Error**: `"/production/process/execute" is of type string, expected sap.m.InputType for property "type"`

This error occurs when dragging and dropping the widget in POD Designer or opening the properties panel.

```javascript
// ❌ WRONG - Spreading parent properties can cause type conflicts!
static getDefaultConfig() {
    const oParentConfig = super.getDefaultConfig();
    const oParentProperties = oParentConfig?.properties || {};

    return {
        properties: {
            ...oParentProperties,  // 💥 May introduce "type" or other reserved properties!
            myProperty: "default"
        }
    };
}

// ✅ CORRECT - Direct property definition (Official SAP pattern)
static getDefaultConfig() {
    return {
        properties: {
            myProperty: "default"  // Simple, direct, no spreading
        }
    };
}
```

**Why this happens**: 
1. Parent classes may have properties like `"type"`, `"enabled"`, `"visible"` that conflict with SAPUI5 control properties
2. When property editors create controls (like `sap.m.Input`), they expect `type` to be an `InputType` enum, not a string
3. Spreading parent properties introduces these conflicts into your widget configuration

**Critical Rule**: 
- **NEVER spread parent properties** - Official SAP documentation never shows this pattern
- Define your widget's properties directly and simply
- If you need runtime default handling for missing properties, override `getPropertyValue()` instead

**Additional Pattern - Runtime Default Coalescing** (from official SAP docs):
```javascript
// Override getPropertyValue to handle missing properties at runtime
getPropertyValue(sName) {
    const vValue = super.getPropertyValue(sName);
    switch (sName) {
        case "myProperty":
            // Return default if value is absent/undefined
            return vValue || "defaultValue";
    }
    return vValue;
}
```

**Source**: SAP Help Portal - "Add Widget Properties" documentation
https://help.sap.com/docs/help/95abdf318cec40bb84bc487fdaa03691/8dbdab1343184bf19ed36cf26f6aaf08.html

---

### Mistake #7: Using "class" Instead of "styleClass" ❌ → ✅

**Error**: `Assertion failed: ManagedObject.apply: encountered unknown setting 'class' for class 'sap.m.VBox'`

```javascript
// ❌ WRONG - "class" is not a valid UI5 property!
return new VBox(oConfig.id, {
    width: "90%",
    class: "sapUiSmallMargin",  // 💥 ERROR!
    items: [/* controls */]
});

// ✅ CORRECT - Use addStyleClass() method instead
return new VBox(oConfig.id, {
    width: "90%",
    items: [/* controls */]
}).addStyleClass("sapUiSmallMargin");

// ✅ ALSO CORRECT - Multiple classes
return new VBox(oConfig.id, {
    width: "90%",
    items: [/* controls */]
}).addStyleClass("sapUiSmallMargin").addStyleClass("myCustomClass");
```

**Why this happens**: Unlike HTML where `class` is an attribute, UI5 controls don't have a `class` property. Use the `addStyleClass()` method to add CSS classes.

---

### Mistake #8: Third-Party Library Loading Fails ❌ → ✅

**Error**: `ReferenceError: moment is not defined` or library not available even after script loads

When including third-party libraries (moment.js, lodash, etc.) in POD 2.0 plugins:

```javascript
// ❌ WRONG - Async loading, library not ready when used
jQuery.sap.includeScript("path/to/moment.min.js");
this.#moment = moment;  // 💥 moment is undefined!

// ❌ WRONG - UI5 module system doesn't work with non-AMD libraries
sap.ui.define([
    "myPlugin/thirdPartyLib/moment.min"  // 💥 Doesn't export properly
], (moment) => {
    // moment is undefined
});

// ❌ WRONG - Script loads but UMD detects AMD loader and calls define() instead
const script = document.createElement("script");
script.src = "path/to/moment.min.js";
document.head.appendChild(script);
// moment.js sees define.amd and uses AMD pattern, not setting window.moment!

// ✅ CORRECT - Use Function() constructor to bypass AMD detection
#loadExternalLibraries() {
    // Get base path for plugin resources
    let sBasePath = "";
    try {
        const sModuleUrl = sap.ui.require.toUrl("my/plugin/namespace/plugins/MyWidget");
        sBasePath = sModuleUrl.substring(0, sModuleUrl.lastIndexOf("/"));
    } catch (e) {
        console.error("Failed to resolve base path", e);
        return;
    }

    // Helper to load script with AMD bypassed
    const loadScriptNoAmd = (url, libName) => {
        try {
            const xhr = new XMLHttpRequest();
            xhr.open("GET", url, false); // Synchronous
            xhr.send(null);

            if (xhr.status === 200) {
                // Function() constructor shadows 'define', 'exports', 'module'
                // This forces UMD libraries to use global export pattern
                const executor = new Function("define", "exports", "module", xhr.responseText);
                executor.call(window, undefined, undefined, undefined);
                console.log(libName + " loaded (AMD bypassed)");
                return true;
            }
        } catch (e) {
            console.error("Failed to load " + libName, e);
        }
        return false;
    };

    // Load libraries
    loadScriptNoAmd(sBasePath + "/thirdPartyLib/moment.min.js", "moment.js");
    loadScriptNoAmd(sBasePath + "/thirdPartyLib/lodash.min.js", "lodash");

    // Now safe to reference globals
    this.#moment = (typeof window.moment !== "undefined") ? window.moment : null;
    this.#_ = (typeof window._ !== "undefined") ? window._ : null;
}
```

**File Structure for Third-Party Libraries:**
```
plugins/
├── YourWidget.js
├── i18n/
└── thirdPartyLib/          # Include libraries here
    ├── moment.min.js
    └── lodash.min.js
```

**Why AMD bypass is needed**: Libraries like moment.js use UMD (Universal Module Definition) which detects AMD loaders via `define.amd`. Since SAPUI5 uses an AMD loader, the library calls `define()` instead of setting `window.moment`. The `new Function("define", ...)` approach creates a scope where `define` is shadowed with `undefined`, forcing the library to use the global export pattern.

---

### Mistake #9: Model Not Initialized Before _createView() ❌ → ✅

**Error**: Widget UI flashes/flickers repeatedly, or bindings like `{/layout}` don't work

This is a **critical lifecycle issue**. In POD 2.0, `_createView()` is called BEFORE `onInit()`. If you initialize your JSONModel in `onInit()` and create controls with bindings in `_createView()`, the bindings fail because the model doesn't exist yet.

```javascript
// ❌ WRONG - Model initialized in onInit(), but _createView() runs first!
class MyWidget extends Widget {
    #oModel = null;

    async onInit() {
        await super.onInit();
        // This runs AFTER _createView()!
        this.#oModel = new JSONModel({ layout: "OneColumn", items: [] });
    }

    _createView() {
        const oConfig = this.getConfig();
        // 💥 this.#oModel is still null here!
        return new FlexibleColumnLayout(oConfig.id, {
            layout: "{/layout}"  // Binding fails - no model!
        }).setModel(this.#oModel);  // Setting null model!
    }
}

// ✅ CORRECT - Initialize model in _createView() BEFORE creating controls
class MyWidget extends Widget {
    #oModel = null;

    async onInit() {
        await super.onInit();
        // Model may already be initialized by _createView()
        if (!this.#oModel) {
            this._initializeModel();
        }
        // Continue with subscriptions, data loading, etc.
    }

    _createView() {
        const oConfig = this.getConfig();

        // CRITICAL: Initialize model BEFORE creating controls with bindings!
        if (!this.#oModel) {
            this._initializeModel();
        }

        const oFCL = new FlexibleColumnLayout(oConfig.id, {
            layout: "{/layout}"  // ✅ Now works - model exists!
        });

        oFCL.setModel(this.#oModel);  // ✅ Model is initialized
        return oFCL;
    }

    _initializeModel() {
        this.#oModel = new JSONModel({
            layout: "OneColumn",
            items: [],
            busy: false
        });
    }
}
```

**POD 2.0 Widget Lifecycle Order:**
```
1. constructor(oConfig)     ← Widget instantiated
2. _createView()            ← UI created - MODEL MUST EXIST HERE!
3. onInit()                 ← Async initialization, subscriptions
4. [widget is rendered]
5. onExit()                 ← Cleanup when destroyed
```

**Critical Rule**: Always initialize your JSONModel at the START of `_createView()` before creating any controls that use data bindings. Check `if (!this.#oModel)` to avoid re-initialization if called multiple times.

**Why this happens**: The POD 2.0 framework calls `_createView()` to get the widget's UI before `onInit()` runs. This is different from typical UI5 patterns where the controller's `onInit` runs first.

---

## Decision Tree

**Not sure which approach to use? Follow this tree:**

```
START: What are you building?
  │
  ├─ Q: Need to display data in a table with columns?
  │   ├─ YES → Use TableWidget
  │   │        - Supports: sorting, pagination, column config
  │   │        - Required methods: getFields(), _createCell(), _getModelPath()
  │   │        → See: TableWidget Pattern section
  │   └─ NO → Continue
  │
  ├─ Q: Need a single UI control (button, input, text, icon)?
  │   ├─ YES → Use ControlWidget
  │   │        - Wraps: Button, Input, Text, Label, Icon, Image
  │   │        - Constructor: super(ControlClass, oConfig)
  │   │        → See: ControlWidget Pattern section
  │   └─ NO → Continue
  │
  ├─ Q: Need a container to hold other widgets or controls?
  │   ├─ YES → Use LayoutWidget
  │   │        - Supports: VBox, HBox, Panel, Dialog, Toolbar
  │   │        - Constructor: super(LayoutClass, oConfig)
  │   │        → See: LayoutWidget Pattern section
  │   └─ NO → Continue
  │
  ├─ Q: Need business logic without UI (forms, dialogs, workflows)?
  │   ├─ YES → Use ContentHandler
  │   │        - Use for: form processing, API workflows, dialog logic
  │   │        - No visual representation in designer
  │   │        → See: ContentHandler Pattern section
  │   └─ NO → Continue
  │
  └─ Q: Need something completely custom?
      └─ YES → Extend Widget directly (rare)
               - Full control over everything
               - More complex, use only if other patterns don't fit
               → See: Widget Class Hierarchy section
```

**Quick Selection Guide:**

| I Want To... | Use This | Example |
|--------------|----------|---------|
| Show a list/table of items | `TableWidget` | Work list, material list |
| Add a button that triggers action | `ControlWidget` | Submit button, refresh button |
| Group widgets in a panel | `LayoutWidget` | VBox with multiple controls |
| Process form data via API | `ContentHandler` | Order confirmation dialog |
| Display current resource/operation | `Widget` + PodContext | Status display widget |

---

## Core Knowledge

### POD Types Overview

There are two types of SAP Digital Manufacturing PODs (Production Operator Dashboards):

- **POD 1.0**: Traditional SAPUI5 component-based architecture (legacy)
- **POD 2.0**: Modern ES6 class-based architecture (recommended for new development)

**Key Decision Point**: Always clarify with users which POD version they're targeting. Recommend POD 2.0 for all new development unless they have legacy requirements.

---

## POD 2.0 Architecture (RECOMMENDED)

### Key Characteristics
- Modern ES6 class syntax with inheritance
- Single-file widgets (no separate view/controller files needed)
- Lightweight and high performance
- Registered via `extension.json`
- Direct POD API access through PodContext
- Programmatic view creation using `_createView()` method
- SAPUI5 controls wrapped in ES6 widget classes

### Widget Class Hierarchy (CRITICAL)

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

**Decision tree for choosing base class:**
- Need a single UI control (button, input, text)? → **ControlWidget**
- Need a container for other widgets? → **LayoutWidget**
- Need a table with columns, sorting, pagination? → **TableWidget**
- Need business logic without UI (dialog processing)? → **ContentHandler**
- Custom widget with unique behavior? → Extend **Widget** (rare)

### Base Classes (POD 2.0)

1. **Widget** - `sap/dm/dme/pod2/widget/Widget` - Abstract base class (rarely extend directly)
2. **ControlWidget** - `sap/dm/dme/pod2/widget/ControlWidget` - For single SAPUI5 controls
3. **LayoutWidget** - `sap/dm/dme/pod2/widget/LayoutWidget` - For layout containers
4. **TableWidget** - `sap/dm/dme/pod2/widget/core/TableWidget` - For complex data tables
5. **ContentHandler** - For business logic without UI
6. **Action** - `sap/dm/dme/pod2/action/Action` - For action buttons

### CRITICAL: Correct POD 2.0 Import Paths

```javascript
// ✅ CORRECT POD 2.0 imports:
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",              // Base widget
    "sap/dm/dme/pod2/widget/ControlWidget",       // For controls
    "sap/dm/dme/pod2/widget/LayoutWidget",        // For layouts
    "sap/dm/dme/pod2/widget/core/TableWidget",    // For tables
    "sap/dm/dme/pod2/context/PodContext",         // POD state/data
    "sap/dm/dme/pod2/context/ModelPath",          // Model binding paths
    "sap/dm/dme/pod2/api/ApiClient",              // API calls (preferred)
    "sap/dm/dme/pod2/api/RestClient",             // REST API calls
    "sap/dm/dme/pod2/Logger",                     // Logging
    "sap/dm/dme/pod2/context/MessageHistory",     // User messages
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",    // Categories
    "sap/dm/dme/pod2/widget/metadata/WidgetProperty",    // Properties
    "sap/dm/dme/pod2/propertyeditor/PropertyCategory",   // Property groups
    "sap/m/library",                              // SAPUI5 controls
    "sap/ui/core/library"                         // Core UI5 types
], (Widget, ControlWidget, LayoutWidget, TableWidget, PodContext, ModelPath,
    ApiClient, RestClient, Logger, MessageHistory, WidgetCategory, WidgetProperty,
    PropertyCategory, SapMLibrary, SapUiCoreLibrary) => {
    "use strict";

    // Destructure commonly used enums/classes
    const { Button, Input, Text, Dialog } = SapMLibrary;
    const { ValueState, TextAlign } = SapUiCoreLibrary;

    class MyWidget extends ControlWidget {
        static getDisplayName() { return "My Widget"; }
        static getIcon() { return "sap-icon://factory"; }
        static getCategory() { return WidgetCategory.Elements; }

        constructor(oConfig) {
            super(Text, oConfig);  // Pass SAPUI5 control class to super!
        }

        _createView() {
            const oControl = super._createView();
            // Additional setup...
            return oControl;
        }
    }

    return MyWidget;
});

// ❌ WRONG POD 1.0 paths (never use these in POD 2.0):
"sap/dm/dme/podfoundation/component/production/ProductionUIComponent"
"sap/dm/dme/podfoundation/controller/PluginViewController"
"sap/dm/dme/model/AjaxUtil"  // Use pod2/api/ApiClient or pod2/api/RestClient

// ❌ ALSO WRONG (Incorrect POD 2.0 paths):
"sap/dm/dme/pod2/model/PodContext"  // ❌ Should be pod2/context/
"sap/dm/dme/pod2/model/ModelPath"   // ❌ Should be pod2/context/
"sap/dm/dme/pod2/property/WidgetProperty"  // ❌ Should be pod2/widget/metadata/
"sap/dm/dme/pod2/property/editor/BooleanPropertyEditor"  // ❌ Should be pod2/propertyeditor/
```

**CRITICAL:**
- Use `pod2/context/` NOT `pod2/model/` for PodContext and ModelPath!
- Use `pod2/widget/metadata/WidgetProperty` NOT `pod2/property/WidgetProperty`!
- Use `pod2/propertyeditor/` NOT `pod2/property/editor/` for property editors!
- Use `pod2/api/ApiClient` (preferred) or `pod2/api/RestClient` for REST API calls!
- Use `WidgetCategory.Elements` or other constants for widget categories!

### CRITICAL: Correct Property Editor Imports

```javascript
// CORRECT imports for properties and property editors:
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/widget/metadata/WidgetProperty",  // ✅ widget/metadata/ NOT property/
    "sap/dm/dme/pod2/propertyeditor/BooleanPropertyEditor",  // ✅ propertyeditor/ NOT property/editor/
    "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor",
    "sap/dm/dme/pod2/propertyeditor/SelectPropertyEditor"
], (Widget, WidgetProperty, BooleanPropertyEditor, StringPropertyEditor, SelectPropertyEditor) => {

    class MyWidget extends Widget {
        getProperties() {
            return [
                new WidgetProperty({
                    displayName: "Enable Feature",
                    description: "Enable or disable the feature",
                    category: "Main",  // ✅ Use string, NOT PropertyCategory.Main
                    propertyEditor: new BooleanPropertyEditor(this, "enableFeature", true)
                }),
                new WidgetProperty({
                    displayName: "API Key",
                    description: "API key for service",
                    category: "Main",
                    propertyEditor: new StringPropertyEditor(this, "apiKey")
                }),
                new WidgetProperty({
                    displayName: "Mode",
                    description: "Operating mode",
                    category: "Main",
                    propertyEditor: new SelectPropertyEditor(
                        this,
                        "mode",
                        ["auto", "manual", "scheduled"],  // ✅ Simple string array
                        "auto"  // default value
                    )
                })
            ];
        }
    }
    return MyWidget;
});
```

### Plugin Registration (POD 2.0: extension.json)

**CRITICAL**: The extension.json file must have ONLY `widgets` and `actions` arrays. No other metadata fields are supported!

**CORRECT extension.json format:**
```json
{
  "widgets": [
    {
      "modulePath": "custom/pod2/example/plugins/YourPlugin",
      "type": "custom.pod2.example.plugins.YourPlugin"
    }
  ],
  "actions": [
    {
      "modulePath": "custom/pod2/example/actions/YourAction",
      "type": "custom.pod2.example.actions.YourAction"
    }
  ]
}
```

**WRONG - DO NOT include these fields:**
```json
{
  "name": "...",           // ❌ NOT SUPPORTED - Will cause upload error
  "description": "...",    // ❌ NOT SUPPORTED - Will cause upload error
  "version": "...",        // ❌ NOT SUPPORTED - Will cause upload error
  "provider": "...",       // ❌ NOT SUPPORTED - Will cause upload error
  "widgets": [...]
}
```

**Common Upload Error:**
```
Failed to create custom extensions: Error encountered when processing the extension components file
```
This error typically means you have extra fields in extension.json that are not supported.

**Key Rules:**
- ✅ ONLY include `widgets` and `actions` arrays
- ✅ `actions` array can be empty but must be present: `"actions": []`
- ✅ Each widget requires `modulePath` and `type` fields
- ✅ The `type` must match `modulePath` with dots instead of slashes
- ✅ No file extension (.js) in modulePath
- ❌ NO other metadata fields at root level

**Correct Folder Structure (POD 2.0)**:
```
project/
├── extension.json          # At root
├── plugins/
│   └── YourPlugin.js      # File at: custom/pod2/example/plugins/YourPlugin.js
├── actions/
│   └── YourAction.js      # File at: custom/pod2/example/actions/YourAction.js
└── util/
    └── Helper.js          # Shared utilities
```

**Common Mistakes**:
- ❌ Don't create nested folders: `custom/pod2/plugins/YourPlugin/YourPlugin.js`
- ✅ Do use flat structure: `plugins/YourPlugin.js` with modulePath: `custom/pod2/plugins/YourPlugin`

### Static Metadata Methods (POD 2.0 - Required)

Every POD 2.0 widget MUST implement these static methods:

```javascript
class MyWidget extends ControlWidget {
    /**
     * Display name shown in POD Designer
     * @override
     * @returns {string}
     */
    static getDisplayName() {
        return "My Custom Widget";
    }

    /**
     * Icon for widget palette (SAP icon name)
     * @override
     * @returns {string}
     */
    static getIcon() {
        return "sap-icon://table-view";
    }

    /**
     * Category in widget palette
     * @override
     * @returns {string}
     */
    static getCategory() {
        return WidgetCategory.Elements;
    }

    /**
     * Default configuration for new instances
     * @override
     * @returns {Object}
     */
    static getDefaultConfig() {
        return {
            properties: {
                text: this.getDisplayName(),
                enabled: true,
                visible: true
            }
        };
    }

    /**
     * Optional: Help documentation URL
     * @override
     * @returns {string}
     */
    static getHelpUrl() {
        return "https://help.sap.com/docs/...";
    }
}
    _createView() {
        // Load fragment or create view programmatically
        return new VBox({
            items: [/* controls */]
        });
    }
}
```

### POD 2.0 Context Access

```javascript
// CORRECT imports (pod2/context/ not pod2/model/)
sap.ui.define([
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath"
], (PodContext, ModelPath) => {

    // Get POD context (static - available everywhere)
    const sPlant = PodContext.getPlant();
    const aResources = PodContext.get(ModelPath.FilterResources);
    const sResource = aResources?.[0]?.resource || null;

    // Set context data
    PodContext.set("/customData", oData);

    // Subscribe to context changes
    onInit() {
        super.onInit();
        if (PodContext.isRunMode()) {
            PodContext.subscribe(ModelPath.FilterResources, this._onResourceChanged, this);
        }
    }

    // CRITICAL: Callback signature is (newValue, path) NOT (path, newValue)!
    // IMPORTANT: Parameters are in REVERSE order from what you might expect!
    _onResourceChanged(aResources, sPath) {
        // PodContext can send undefined, null, or non-array types
        // Always coerce to expected type to prevent crashes
        const resources = Array.isArray(aResources) ? aResources : [];
        this._updateResourceDisplay(resources);
    }

    // Defensive update method with type checking
    _updateResourceDisplay(aResources) {
        if (this._oResourceText) {
            // Double-check array type before array operations
            const sResourceList = Array.isArray(aResources) && aResources.length > 0
                // Use optional chaining for safe property access
                ? aResources.map(r => r?.resource || "Unknown").join(", ")
                : "No resources selected";
            this._oResourceText.setText(sResourceList);
        }
    }

    onExit() {
        super.onExit();
        if (PodContext.isRunMode()) {
            PodContext.unsubscribe(ModelPath.FilterResources, this._onResourceChanged, this);
        }
    }
});
```

**Available PodContext Methods:**
- `PodContext.getPlant()` - Get current plant
- `PodContext.get(path)` - Get value from context path
- `PodContext.set(path, value)` - Set value in context path
- `PodContext.subscribe(path, handler, context)` - Subscribe to changes
- `PodContext.unsubscribe(path, handler, context)` - Unsubscribe from changes
- `PodContext.isRunMode()` - Check if in runtime mode (not design mode)
- `PodContext.getLastSelectedWorkListItem()` - Get last selected work item

**Available ModelPath Constants:**
- `ModelPath.FilterResources` - Selected resources in the POD
- `ModelPath.DataCollectionGroups` - Data collection groups
- `ModelPath.SelectedWorkListItems` - Selected work list items

**IMPORTANT:** `PodContext.getUserId()` is NOT available in POD 2.0. For user information, access it differently or omit.

**CRITICAL - PodContext Subscription Callback Parameter Order:**

🚨 **IMPORTANT:** The callback signature is `(newValue, path)` NOT `(path, newValue)`!

This is the **OPPOSITE** of what you might expect. The data comes FIRST, the path comes SECOND.

```javascript
// ❌ WRONG - Parameters in wrong order
PodContext.subscribe(ModelPath.FilterResources, (sPath, aResources) => {
    // sPath is actually the DATA array
    // aResources is actually the PATH string
    // This will crash!
});

// ✅ CORRECT - Data first, path second
PodContext.subscribe(ModelPath.FilterResources, (aResources, sPath) => {
    // aResources is the DATA (first parameter)
    // sPath is the PATH (second parameter)
    console.log("Data:", aResources);
    console.log("Path:", sPath);
});
```

**CRITICAL - Defensive Coding for Subscriptions:**

PodContext subscription callbacks can receive various data types:
- `[]` - Empty array when no items
- `undefined` - When data is cleared
- `null` - When context is reset
- Non-array types in edge cases

**Always validate data types:**
```javascript
// ❌ BAD - Wrong parameter order AND no type checking
_onResourceChanged(sPath, aResources) {
    const list = aResources.map(r => r.resource).join(", ");
}

// ✅ GOOD - Correct parameter order with type coercion and validation
_onResourceChanged(aResources, sPath) {
    const resources = Array.isArray(aResources) ? aResources : [];
    const list = resources.map(r => r?.resource || "Unknown").join(", ");
}
```

### POD 2.0 Lifecycle Methods

```javascript
class MyWidget extends Widget {
    // REQUIRED: Create and return the view
    _createView() {
        const oConfig = this.getConfig();

        // Validate configuration has an ID
        if (!oConfig || !oConfig.id) {
            return new VBox({
                items: [new Text({ text: "Configuration error" })]
            });
        }

        // Store references to controls that need to be updated later
        this._oMyText = new Text({
            text: "Initial value"
        });

        // CRITICAL: Pass oConfig.id as FIRST parameter to set view ID
        const oView = new VBox(oConfig.id, {
            items: [
                new Text({ text: "Hello POD 2.0" }),
                this._oMyText,  // Store reference for later access
                new Button({
                    text: "Click Me",
                    press: () => this._onButtonPress()
                })
            ]
        });

        return oView;
    }

    // Called after widget initialization
    onInit() {
        super.onInit();

        // Subscribe to context changes
        if (PodContext.isRunMode()) {
            PodContext.subscribe(ModelPath.FilterResources, this._onResourceChanged, this);
        }
    }

    // Update controls using stored references
    _onButtonPress() {
        if (this._oMyText) {
            this._oMyText.setText("Button pressed!");
        }
    }

    // Cleanup
    onExit() {
        super.onExit();

        // Unsubscribe from context changes
        if (PodContext.isRunMode()) {
            PodContext.unsubscribe(ModelPath.FilterResources, this._onResourceChanged, this);
        }

        // Clean up control references
        this._oMyText = null;
    }
}
```

**CRITICAL Requirements:**
1. **View ID**: Always pass `oConfig.id` as the FIRST parameter when creating the root view control
2. **Config Validation**: Check that `oConfig` and `oConfig.id` exist before using them
3. **No createId/byId**: Don't use `this.createId()` or `this.byId()` - store direct references instead

### POD 2.0 Configuration Properties

#### Static Configuration Arrays (For ControlWidget/LayoutWidget)

```javascript
class MyWidget extends ControlWidget {
    // Properties that can bind to context model
    static BINDABLE_PROPERTIES = ["text", "enabled", "visible"];

    // Events to expose in action configuration
    static INCLUDE_EVENTS = ["press", "change"];

    // Properties to hide from designer
    static EXCLUDE_PROPERTIES = ["busy", "busyIndicatorDelay"];

    // Override property categories
    static PROPERTY_CATEGORY_OVERRIDE = {
        text: PropertyCategory.Main,
        icon: PropertyCategory.Appearance,
        enabled: PropertyCategory.Behavior,
        width: PropertyCategory.Dimension
    };
}
```

#### Property Editor Types

- `StringPropertyEditor` - Text input
- `IntegerPropertyEditor` - Number input
- `BooleanPropertyEditor` - Checkbox
- `EnumPropertyEditor` - Dropdown selection
- `TableColumnsPropertyEditor` - Column configuration
- `HotKeyPropertyEditor` - Keyboard shortcuts

#### Property Categories

- `Main` - Primary configuration
- `Appearance` - Visual styling
- `Behavior` - Functional behavior
- `Dimension` - Size and spacing
- `Data` - Data binding
- `Accessibility` - A11y options

#### Custom Properties Example

```javascript
class MyPlugin extends Widget {
    static PropertyId = Object.freeze({
        EnableFeature: "enableFeature",
        PluginId: "pluginId",
        MaxRows: "maxRows"
    });

    getProperties() {
        return [
            new WidgetProperty({
                displayName: "Enable Feature",
                description: "Enable or disable the sample feature",
                category: PropertyCategory.Main,
                propertyEditor: new BooleanPropertyEditor(
                    this,
                    this.constructor.PropertyId.EnableFeature,
                    true // default value
                )
            }),
            new WidgetProperty({
                displayName: "Plugin ID",
                description: "ID of the plugin to call",
                category: PropertyCategory.Main,
                propertyEditor: new StringPropertyEditor(
                    this,
                    this.constructor.PropertyId.PluginId
                )
            }),
            new WidgetProperty({
                displayName: "Max Rows",
                description: "Maximum number of rows to display",
                category: PropertyCategory.Main,
                propertyEditor: new IntegerPropertyEditor(
                    this,
                    this.constructor.PropertyId.MaxRows,
                    10 // default
                )
            })
        ];
    }

    // Access property value
    _someMethod() {
        const bEnabled = this.getPropertyValue(this.constructor.PropertyId.EnableFeature);
        const sPluginId = this.getPropertyValue(this.constructor.PropertyId.PluginId)?.trim();
        const iMaxRows = this.getPropertyValue(this.constructor.PropertyId.MaxRows);
    }
}
```

### POD 2.0 API Calls

**CRITICAL DECISION:** Choose the right API client for your use case!

#### Quick Decision Guide

```
Need to call an API?
│
├─ Standard SAP DM public API? (SFC, Order, Material, etc.)
│  └─ YES → Use ApiClient public APIs
│          Example: ApiClient.sfc.getSfcs()
│          ⚠️ NEVER use ApiClient.internal.* (for SAP only!)
│
├─ Your custom extension API deployed to SAP DM?
│  └─ YES → Use RestClient
│          Example: RestClient.post("/myextension/endpoint", data)
│
└─ Third-party or external API?
   └─ YES → Use RestClient
           Example: RestClient.get("https://external-api.com")
```

#### RestClient - For Custom & External APIs

**Import:**
```javascript
import RestClient from "sap/dm/dme/pod2/api/RestClient";
```

**When to use:**
- ✅ Your custom extension APIs
- ✅ Third-party APIs
- ✅ Need custom headers or authentication
- ✅ Non-JSON responses (blobs, HTML, etc.)

**Features:**
- Automatic SAP DM headers (`x-dme-plant`, `x-dme-industry-type`)
- CSRF token management
- Session timeout detection
- Cross-tab activity synchronization
- Smart content-type handling (JSON/text/HTML)

**Methods:**
```javascript
// GET with query params
const data = await RestClient.get("/api/endpoint", {
    param1: "value",
    param2: "value"
});

// POST with body
const result = await RestClient.post("/api/endpoint", {
    key: "value"
});

// PUT, PATCH, DELETE
await RestClient.put(url, body);
await RestClient.patch(url, body);
await RestClient.delete(url);

// Custom headers
const result = await RestClient.post(url, body, {
    headers: {
        "Authorization": "Bearer token",
        "X-Custom-Header": "value"
    }
});

// Utility methods
const queryString = RestClient.objectToQueryString({ key: "value" });
const cleanJson = RestClient.objectToJsonString(obj); // Removes null/undefined/empty
const baseUrl = RestClient.getBaseUrl();
const lastActivity = RestClient.getLastActivityTime();
```

#### ApiClient - For SAP DM Public Operations

**Import:**
```javascript
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";
```

**When to use:**
- ✅ Standard SAP DM operations
- ✅ Need type safety and IntelliSense
- ✅ Automatic plant/auth context
- ✅ Using documented public APIs

**⚠️ CRITICAL WARNING:**
`ApiClient.internal.*` is **ONLY for SAP standard widgets/actions**. It is **subject to change without notice** and **must NOT be used in custom widgets**. Use public APIs instead!

**📖 COMPREHENSIVE REFERENCE:**
For complete ApiClient documentation including all methods, parameter structures, and response payloads, see:
`references/ApiClient-Reference.md`

This reference includes:
- All 18 public API namespaces (sfc, material, order, etc.)
- Every method with exact signatures discovered from runtime inspection
- Correct parameter format (CRITICAL: single object parameter, not multiple params)
- Response payload structures with real examples
- Working code examples for every namespace
- Common usage patterns and best practices

**Public API Structure:**
```javascript
ApiClient
├── alert              // Public Alert APIs
├── assembly           // Public Assembly APIs
├── bom                // Public BOM APIs
├── datacollection     // Public Data Collection APIs (lazy-loaded)
├── datafields         // Public Data Fields APIs
├── execution          // Public Execution APIs
├── inventory          // Public Inventory APIs
├── material           // Public Material APIs
├── mdo                // Public MDO APIs
├── operationactivity  // Public Operation Activity APIs
├── order              // Public Order APIs
├── processorder       // Public Process Order APIs
├── resource           // Public Resource APIs
├── sfc                // Public SFC APIs
├── uom                // Public UOM APIs
├── user               // Public User APIs
├── workcenter         // Public Work Center APIs
└── workinstruction    // Public Work Instruction APIs
```

**Examples:**
```javascript
// SFC operations
const sfcs = await ApiClient.sfc.getSfcs({
    plant: PodContext.getPlant(),
    sfc: "SFC*"
});

// Material operations
const materials = await ApiClient.material.getMaterials({
    plant: PodContext.getPlant(),
    material: "MAT*"
});

// Order operations
const order = await ApiClient.order.getOrder({
    plant: PodContext.getPlant(),
    order: "ORDER_001"
});

// Lazy-loaded APIs (like datacollection)
await ApiClient.ready();  // Wait for lazy-loaded modules
const dcGroups = await ApiClient.datacollection.getDataCollectionGroups({
    plant: PodContext.getPlant()
});
```

**Each public API includes links to official SAP documentation:**
- SFC: https://api.sap.com/api/sapdme_sfc/resource/SFC_Processing
- Material: https://api.sap.com/api/sapdme_material/resource/Material
- Order: https://api.sap.com/api/sapdme_order/resource/Order
- Assembly: https://api.sap.com/api/sapdme_assembly/resource/Assembly
- BOM: https://api.sap.com/api/sapdme_bom/resource/BOM
- And more...

#### Comparison Matrix

| Feature | RestClient | ApiClient Public |
|---------|-----------|------------------|
| **Use Case** | Custom/External APIs | SAP DM Public APIs |
| **Authentication** | Automatic (SAP DM headers) | Automatic + plant context |
| **Type Safety** | None (any) | Strong typing (JSDoc) |
| **IntelliSense** | Limited | Full support |
| **Custom Headers** | Full control | Limited |
| **CSRF Tokens** | Automatic | Automatic |
| **Session Mgmt** | Yes (cross-tab sync) | Inherited from RestClient |
| **Best For** | Your extensions | Standard SAP DM ops |

#### Error Handling Pattern (Updated)

Always use try/catch with proper timeout:

```javascript
import Logger from "sap/dm/dme/pod2/Logger";
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";
import RestClient from "sap/dm/dme/pod2/api/RestClient";

const logger = Logger.getLogger("MyWidget");
const TIMEOUT_MS = 30000;

// Example 1: Using ApiClient public API
async function fetchSfcs() {
    try {
        const result = await Promise.race([
            ApiClient.sfc.getSfcs({
                plant: PodContext.getPlant(),
                sfc: "SFC*"
            }),
            createTimeoutPromise(TIMEOUT_MS)
        ]);
        
        logger.info("SFCs fetched successfully");
        return result;
        
    } catch (error) {
        logger.error("Failed to fetch SFCs", error);
        MessageHistory.showError("Failed to load SFC data");
        throw error;
    }
}

// Example 2: Using RestClient for custom API
async function callCustomApi() {
    try {
        const result = await Promise.race([
            RestClient.post("/myextension/validate", {
                plant: PodContext.getPlant(),
                data: "value"
            }),
            createTimeoutPromise(TIMEOUT_MS)
        ]);
        
        logger.info("Custom API call successful");
        return result;
        
    } catch (error) {
        logger.error("Custom API call failed", error);
        MessageHistory.showError("Failed to validate data");
        throw error;
    }
}

function createTimeoutPromise(timeout) {
    return new Promise((_, reject) => {
        setTimeout(() => reject(new Error("Request timeout")), timeout);
    });
}
```

#### Real-World Examples

**Example 1: Get Materials (ApiClient)**
```javascript
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";

async function loadMaterials(searchTerm) {
    const [materials, totalCount] = await ApiClient.material.getMaterials({
        plant: PodContext.getPlant(),
        material: `${searchTerm}*`  // Wildcard search
    });
    
    console.log(`Found ${totalCount} materials`);
    return materials;
}
```

**Example 2: Custom Extension API (RestClient)**
```javascript
import RestClient from "sap/dm/dme/pod2/api/RestClient";

async function validateSfc(sfc) {
    // Your custom extension API deployed to SAP DM
    const result = await RestClient.post("/myextension/validation/sfc", {
        sfc: sfc,
        validationType: "quality",
        plant: PodContext.getPlant()
    });
    
    return result.isValid;
}
```

**Example 3: External Service (RestClient)**
```javascript
import RestClient from "sap/dm/dme/pod2/api/RestClient";

async function sendSlackNotification(message) {
    // External Slack webhook
    await RestClient.post("https://hooks.slack.com/services/YOUR/WEBHOOK", {
        text: message,
        channel: "#production-alerts"
    }, {
        headers: {
            "Content-Type": "application/json"
        }
    });
}
```

**Example 4: Compare Good vs Bad**
```javascript
// ❌ BAD - Using internal API in custom widget
async function badExample() {
    // Internal APIs are for SAP standard widgets only!
    const result = await ApiClient.internal.sfc.start({
        plant: PodContext.getPlant(),
        sfc: "SFC_001"
    });
    // Subject to change without notice!
}

// ✅ GOOD - Using public API
async function goodExample() {
    const result = await ApiClient.sfc.sfcStart({
        plant: PodContext.getPlant(),
        sfc: "SFC_001",
        resource: "RES_001",
        operationActivity: "OP_001"
    });
    // Stable, documented, supported
}

// ✅ ALSO GOOD - Using RestClient for custom endpoint
async function customApiExample() {
    const result = await RestClient.post("/custom/start", {
        plant: PodContext.getPlant(),
        sfc: "SFC_001",
        customParam: "value"
    });
    // Full control, works with any endpoint
}
```

#### Data Delegates - Centralized State Management

**What are Delegates?**

Delegates are stateful service classes that:
1. **Centralize data fetching** for specific domains (work lists, data collection, activities, etc.)
2. **Manage PodContext state** - fetch data from APIs and update PodContext automatically
3. **Coordinate multiple widgets** - ensure all widgets display consistent, synchronized data
4. **Handle caching & lifecycle** - prevent duplicate API calls, manage refresh intervals

**Key Distinction:**
```
ApiClient/RestClient → Low-level HTTP calls, YOU manage state
Delegates          → High-level operations, DELEGATE manages PodContext state
```

**When to Use Delegates vs ApiClient/RestClient:**

| Scenario | Use | Reason |
|----------|-----|--------|
| Display work list data in a table | **Delegate** | Delegate syncs data to PodContext, all widgets auto-refresh |
| Submit activity confirmation | **ApiClient** (then delegate refresh) | Mutation, then refresh shared state |
| Load data collection parameters | **Delegate** | Shared data, multiple widgets need sync |
| Validate custom business rule | **RestClient** | One-off call, no shared state needed |
| Fetch SFC details for display | **ApiClient public API** | Standard query, local use only |
| Refresh activity summaries after posting | **Delegate** | Update shared state across widgets |

**Available Delegates:**

| Delegate | Import Path | Key Methods | Auto-Refreshes On |
|----------|-------------|-------------|-------------------|
| **ActivityConfirmationDelegate** | `sap/dm/dme/pod2/context/data/ActivityConfirmationDelegate` | `refreshActivitySummaries({ force, clear })` | SelectedOperationActivities |
| **DataCollectionDelegate** | `sap/dm/dme/pod2/context/data/DataCollectionDelegate` | `refreshGroups()`, `refreshLog()`, `refreshLoggedData()` | Manual refresh only |
| **GoodsReceiptDelegate** | `sap/dm/dme/pod2/context/data/GoodsReceiptDelegate` | `refreshSummary()` | SelectedWorkListItems |
| **OperationActivityDelegate** | `sap/dm/dme/pod2/context/data/OperationActivityDelegate` | `refresh({ force })` | SelectedWorkListItems + WebSocket notifications |
| **QualityInspectionDelegate** | `sap/dm/dme/pod2/context/data/QualityInspectionDelegate` | `refresh({ request, force })` | SelectedOperationActivities |
| **QuantityConfirmationDelegate** | `sap/dm/dme/pod2/context/data/QuantityConfirmationDelegate` | `refresh()`, `fetchNextPage()` | SelectedWorkListItems, SelectedOperationActivities |
| **WorkInstructionDelegate** | `sap/dm/dme/pod2/context/data/WorkInstructionDelegate` | `init()`, `refresh({ request, force })` | SelectedWorkListItems, SelectedOperationActivities, FilterResources |
| **WorkListDelegate** | `sap/dm/dme/pod2/context/data/WorkListDelegate` | `refresh({ filter, ignoreRefreshInterval, abortPendingRequest })`, `fetchNextPage()` | WebSocket notifications (SFC_START, SFC_SIGNOFF, SFC_COMPLETE) |

**Real-World Usage Patterns:**

**Pattern 1: Widget Initialization with Delegate**
```javascript
// From official DataCollectionGroupTableWidget
import DataCollectionDelegate from "sap/dm/dme/pod2/context/data/DataCollectionDelegate";

async onInit() {
    super.onInit();
    
    // Check if data already loaded by another widget
    if (!PodContext.getDataCollectionGroups()) {
        // Delegate fetches data AND updates PodContext automatically
        await DataCollectionDelegate.refreshGroups();
    }
    
    // Subscribe to PodContext changes (delegate manages the data)
    PodContext.subscribe(ModelPath.DataCollectionGroups, this._onDataChanged, this);
}

_onDataChanged(aGroups, sPath) {
    // Widget automatically gets fresh data from PodContext
    // NO need to manually assign - binding handles it!
}
```

**Pattern 2: Refresh After Mutation**
```javascript
// From official GoodsReceiptPostContentHandler
import GoodsReceiptDelegate from "sap/dm/dme/pod2/context/data/GoodsReceiptDelegate";
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";

async onPostGoodsReceipt(oPostingData) {
    try {
        // 1. Perform mutation via ApiClient (standard API)
        await ApiClient.internal.goodsreceipt.postGoodsReceipt(oPostingData);
        
        // 2. Refresh shared state via Delegate (updates PodContext for all widgets)
        await GoodsReceiptDelegate.refreshSummary({ force: true });
        
        MessageToast.show("Goods receipt posted successfully");
    } catch (oError) {
        // Handle error
    }
}
```

**Pattern 3: Checking Cached Data Before Loading**
```javascript
// From official ActivityConfirmationTableWidget
import ActivityConfirmationDelegate from "sap/dm/dme/pod2/context/data/ActivityConfirmationDelegate";

async onInit() {
    super.onInit();
    
    if (PodContext.isRunMode()) {
        // Delegate manages lifecycle - init() checks cache and loads if needed
        await ActivityConfirmationDelegate.init();
        
        // All data now available in PodContext for all widgets
        PodContext.subscribe(ModelPath.ActivitySummaries, 
            () => this._updateReportButtonEnabled(), 
            this
        );
    }
}
```

**Pattern 4: Pagination via Delegate**
```javascript
// Work list delegate handles pagination state automatically
import WorkListDelegate from "sap/dm/dme/pod2/context/data/WorkListDelegate";

async onLoadMore() {
    // Delegate manages page state, appends to PodContext array
    await WorkListDelegate.fetchNextPage();
    // All subscribed widgets auto-update with new items
}

async onRefresh() {
    // Force full refresh (clears cache)
    await WorkListDelegate.refresh({
        abortPendingRequest: true,
        ignoreRefreshInterval: true
    });
}
```

**Pattern 5: Multiple Widgets Sharing Delegate Data**
```javascript
// Widget A: Data Collection Group Table
class DataCollectionGroupTableWidget extends TableWidget {
    async onInit() {
        super.onInit();
        if (!PodContext.getDataCollectionGroups()) {
            await DataCollectionDelegate.refreshGroups();  // Loads once
        }
        // Binds to PodContext.dataCollectionGroups
    }
}

// Widget B: Data Collection Parameter Table (same page)
class DataCollectionParamTableWidget extends TableWidget {
    async onInit() {
        super.onInit();
        // NO API CALL - data already in PodContext from Widget A!
        // Just subscribe to changes
        DataCollectionDelegate.refreshLog();  // Only logs, no fetch if cached
    }
}

// Result: ONE API call, TWO widgets synchronized automatically
```

**Delegate Method Patterns:**

```javascript
// Common delegate methods (not all delegates have all methods):

// Initialize (check cache, load if needed)
await SomeDelegate.init();

// Force refresh (ignore cache)
await SomeDelegate.refresh({ force: true });

// Refresh with options
await SomeDelegate.refreshSummary({ 
    force: true,           // Bypass cache
    abortPendingRequest: true  // Cancel in-flight requests
});

// Clear before refresh (prevents showing stale data during load)
await ActivityConfirmationDelegate.refreshActivitySummaries({ 
    force: true, 
    clear: true  // Clears PodContext data immediately
});

// Specific domain operations
await DataCollectionDelegate.refreshGroups();
await DataCollectionDelegate.refreshLog();
await DataCollectionDelegate.refreshLoggedData();
await ActivityConfirmationDelegate.refreshActivitySummaries({ force: true });
await WorkListDelegate.fetchNextPage();
await QuantityConfirmationDelegate.fetchNextPage();
```

**Advanced Patterns:**

**Pattern 6: Abort Pending Requests (from WorkListDelegate source)**
```javascript
// WorkListDelegate, WorkInstructionDelegate, and QuantityConfirmationDelegate
// support aborting in-flight requests when new data is needed

import WorkListDelegate from "sap/dm/dme/pod2/context/data/WorkListDelegate";

async onUserClickedRefresh() {
    // Cancel any pending request and start fresh
    await WorkListDelegate.refresh({
        abortPendingRequest: true,  // Cancels in-flight request
        ignoreRefreshInterval: true,  // Bypasses MIN_REFRESH_INTERVAL
        force: true
    });
}
```

**Pattern 7: Real-Time Updates via WebSocket Notifications**
```javascript
// WorkListDelegate and OperationActivityDelegate automatically subscribe
// to WebSocket notifications and refresh when events occur

// From WorkListDelegate source code - automatic subscription
// Subscribes to: SFC_START, SFC_SIGNOFF, SFC_COMPLETE
// When these events occur, delegate automatically calls refreshIfContainsSfc()

// From OperationActivityDelegate source code - automatic subscription  
// Subscribes to: SFC_START, SFC_SIGNOFF, SFC_COMPLETE, OPERATION_START, OPERATION_COMPLETE
// When these events occur, delegate automatically refreshes operation activities

// NO CODE NEEDED - delegates handle this automatically after init()!
```

**Pattern 8: Preventing Duplicate Requests (from delegate source)**
```javascript
// All delegates track pending requests to prevent duplicates
// From ActivityConfirmationDelegate source:

// First call - makes API request
await ActivityConfirmationDelegate.refreshActivitySummaries();

// Second call (while first is pending) - skipped automatically
await ActivityConfirmationDelegate.refreshActivitySummaries();  // Logs: "refresh skipped. A request is already pending."

// After first completes, same data request - skipped automatically
await ActivityConfirmationDelegate.refreshActivitySummaries();  // Skipped (same data)

// Force option bypasses deduplication
await ActivityConfirmationDelegate.refreshActivitySummaries({ force: true });  // ✅ Makes request
```

**Critical Rules:**

1. **Delegates update PodContext** - Don't manually set data after delegate calls
2. **Check before loading** - Use `PodContext.getSomeData()` to avoid duplicate loads
3. **Subscribe to PodContext paths** - Widgets react to delegate-managed state
4. **Mutation → API, Read → Delegate** - Use ApiClient for writes, delegates for reads
5. **Force refresh after mutations** - `await Delegate.refresh({ force: true })`

**Anti-Patterns (DON'T DO THIS):**

```javascript
// ❌ WRONG - Manual state management defeats delegate purpose
import DataCollectionDelegate from "sap/dm/dme/pod2/context/data/DataCollectionDelegate";
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";

// DON'T manually fetch and set when delegate exists
const aGroups = await ApiClient.datacollection.getGroups(...);  // ❌ Delegate should do this!
PodContext.setDataCollectionGroups(aGroups);  // ❌ Delegate manages this!

// ✅ CORRECT - Let delegate handle it
await DataCollectionDelegate.refreshGroups();  // Fetches AND updates PodContext

// ❌ WRONG - Not refreshing delegate after mutation
await ApiClient.internal.activityconfirmation.postActivity(data);
// Forget to call: ActivityConfirmationDelegate.refreshActivitySummaries()
// Result: Widgets show stale data!

// ✅ CORRECT - Always refresh delegate state after mutations
await ApiClient.internal.activityconfirmation.postActivity(data);
await ActivityConfirmationDelegate.refreshActivitySummaries({ force: true });
```

**Decision Tree:**

```
Need to work with manufacturing data?
│
├─ Is this for display/query across multiple widgets?
│  ├─ YES → Check if delegate exists for this domain
│  │  ├─ Delegate exists (work lists, data collection, activities, etc.)
│  │  │  └─ ✅ Use Delegate (manages PodContext automatically)
│  │  └─ No delegate
│  │     └─ ✅ Use ApiClient public API (manage state yourself)
│  │
│  └─ NO → Is this a mutation (create/update/delete)?
│     ├─ Standard SAP DM operation
│     │  └─ ✅ Use ApiClient public API, then delegate.refresh()
│     └─ Custom/extension operation
│        └─ ✅ Use RestClient, then delegate.refresh() if needed
│
└─ One-off query? No shared state?
   └─ ✅ Use ApiClient public API or RestClient
```

#### Migration from POD 1.0 AjaxUtil to POD 2.0

**CRITICAL:** POD 1.0's `AjaxUtil` causes 404 errors in POD 2.0!

| POD 1.0 (AjaxUtil) | POD 2.0 (RestClient/ApiClient) |
|--------------------|-------------------------------|
| `AjaxUtil.get(url, success, error)` | `await RestClient.get(url)` |
| `AjaxUtil.post(url, payload, success, error)` | `await RestClient.post(url, payload)` OR `await ApiClient.sfc.sfcStart(...)` |
| Callback-based | Promise-based (async/await) |
| Manual error handling | Try/catch |
| No type safety | Type-safe (ApiClient public APIs) |
| `sap/dm/dme/model/AjaxUtil` | `sap/dm/dme/pod2/api/RestClient` or `sap/dm/dme/pod2/api/ApiClient` |

**Example Migration:**

```javascript
// ❌ POD 1.0 - Callbacks
import AjaxUtil from "sap/dm/dme/model/AjaxUtil";  // 404 in POD 2.0!

AjaxUtil.post("/api/sfc/start", payload, 
    function(result) {
        console.log("Success:", result);
    },
    function(error) {
        console.error("Error:", error);
    }
);

// ✅ POD 2.0 Option 1 - Public API (Preferred)
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";

try {
    const result = await ApiClient.sfc.sfcStart(payload);
    console.log("Success:", result);
} catch (error) {
    console.error("Error:", error);
}

// ✅ POD 2.0 Option 2 - RestClient (For custom endpoints)
import RestClient from "sap/dm/dme/pod2/api/RestClient";

try {
    const result = await RestClient.post("/custom/endpoint", payload);
    console.log("Success:", result);
} catch (error) {
    console.error("Error:", error);
}
```

#### Best Practices Summary

1. **✅ Use ApiClient public APIs** for standard SAP DM operations
   - Type-safe, validated, handles auth automatically
   - Documented at api.sap.com
   - Stable and supported

2. **⚠️ NEVER use ApiClient.internal** in custom widgets
   - Only for SAP standard components
   - Subject to change without notice
   - Use public APIs instead

3. **✅ Use RestClient** for:
   - Your custom extension APIs
   - Third-party/external services
   - Custom headers or authentication needs

4. **✅ Wait for lazy-loaded APIs**
   ```javascript
   await ApiClient.ready();  // Before using datacollection, etc.
   ```

5. **✅ Always use timeout protection**
   - Wrap API calls in Promise.race()
   - Set reasonable timeouts (30s default)
   - Handle errors gracefully

6. **❌ NEVER use POD 1.0 AjaxUtil**
   - Causes 404 errors in POD 2.0
   - Use RestClient or ApiClient instead
| Callback-based | Promise-based (async/await) |
| `sap/dm/dme/model/AjaxUtil` | `sap/dm/dme/pod2/api/ApiClient` |

---

## Logging and Messages

### Logging

```javascript
import Logger from "sap/dm/dme/pod2/Logger";

class MyWidget extends Widget {
    #oLog = Logger.getLogger("sap.dm.dme.pod2.widget.custom.MyWidget");

    async _loadData() {
        this.#oLog.info("Loading data...");

        try {
            const oData = await ApiClient.custom.get("/data");
            this.#oLog.debug("Data loaded", oData);
        } catch (oError) {
            this.#oLog.error("Failed to load data", oError);
            throw oError;
        }
    }
}
```

Log levels: `trace`, `debug`, `info`, `warn`, `error`, `fatal`

### User Messages

```javascript
import MessageHistory from "sap/dm/dme/pod2/context/MessageHistory";
import MessageBox from "sap/m/MessageBox";

// Toast (auto-dismiss)
MessageHistory.toast({
    message: "Operation successful",
    type: MessageHistory.Success
});

// Error dialog
MessageHistory.showError("An error occurred");

// Warning dialog
MessageHistory.showWarning("Please confirm");

// Info dialog
MessageHistory.showInfo("Information");

// Custom dialog with callback
const oMessage = MessageHistory.showWarning("Proceed?", {
    actions: [MessageBox.Action.OK, MessageBox.Action.CANCEL],
    onClose: (sAction) => {
        if (sAction === MessageBox.Action.OK) {
            // Handle OK
        }
        MessageHistory.dismissMessage(oMessage);
    }
});
```

---

## POD 1.0 Architecture (LEGACY)

### Key Characteristics
- Full SAPUI5 Component framework with MVC pattern
- Requires multiple files (Component.js, manifest.json, controllers, views, PropertyEditor)
- Higher complexity, slower development
- Registered via `designer/components.json`
- Uploaded to POD Designer

### Base Classes (POD 1.0)

1. **ProductionUIComponent** - `sap/dm/dme/podfoundation/component/production/ProductionUIComponent` - View plugins
2. **ProductionComponent** - `sap/dm/dme/podfoundation/component/production/ProductionComponent` - Execution plugins
3. **PluginViewController** - `sap/dm/dme/podfoundation/controller/PluginViewController` - Controller base

### POD 1.0 Component Structure

```javascript
// Component.js
sap.ui.define([
    "sap/dm/dme/podfoundation/component/production/ProductionUIComponent"
], function(ProductionUIComponent) {
    "use strict";

    var Component = ProductionUIComponent.extend("sap.ext.example.Component", {
        metadata: {
            manifest: "json"
        }
    });

    return Component;
});
```

### POD 1.0 Controller

```javascript
sap.ui.define([
    "sap/dm/dme/podfoundation/controller/PluginViewController",
    "sap/ui/model/json/JSONModel",
    "sap/m/MessageToast"
], function(PluginViewController, JSONModel, MessageToast) {
    "use strict";

    return PluginViewController.extend("sap.ext.example.controller.PluginView", {

        onInit: function() {
            if (PluginViewController.prototype.onInit) {
                PluginViewController.prototype.onInit.apply(this, arguments);
            }

            var oModel = new JSONModel();
            this.getView().setModel(oModel);
        },

        onBeforeRenderingPlugin: function() {
            // Subscribe to events
            this.subscribe("PodSelectionChangeEvent", this.onPodSelectionChangeEvent, this);
            this.subscribe("WorklistSelectEvent", this.onWorklistSelectEvent, this);
        },

        onPodSelectionChangeEvent: function(sChannelId, sEventId, oData) {
            if (this.isEventFiredByThisPlugin(oData)) {
                return; // Ignore events from this plugin
            }

            var oPodSelectionModel = this.getPodSelectionModel();
            var sResource = oPodSelectionModel.getResource();
            var sOperation = oPodSelectionModel.getOperation();

            // Handle event
        },

        onExit: function() {
            // Unsubscribe from events
            this.unsubscribe("PodSelectionChangeEvent", this.onPodSelectionChangeEvent, this);
            this.unsubscribe("WorklistSelectEvent", this.onWorklistSelectEvent, this);

            if (PluginViewController.prototype.onExit) {
                PluginViewController.prototype.onExit.apply(this, arguments);
            }
        }
    });
});
```

### POD 1.0 View (XML)

```xml
<mvc:View
    xmlns:mvc="sap.ui.core.mvc"
    xmlns="sap.m"
    xmlns:core="sap.ui.core"
    controllerName="sap.ext.example.controller.PluginView">

    <Panel width="100%" height="100%">
        <VBox class="sapUiSmallMargin">
            <Label text="{i18n>resource}"/>
            <Text text="{/resource}"/>

            <Label text="{i18n>operation}"/>
            <Text text="{/operation}"/>

            <Table items="{/items}">
                <columns>
                    <Column><Text text="{i18n>columnName}"/></Column>
                </columns>
                <items>
                    <ColumnListItem>
                        <cells><Text text="{name}"/></cells>
                    </ColumnListItem>
                </items>
            </Table>

            <Button
                text="{i18n>buttonText}"
                press="onButtonPress"/>
        </VBox>
    </Panel>
</mvc:View>
```

### POD 1.0 Property Editor

```javascript
sap.ui.define([
    "sap/dm/dme/podfoundation/control/PropertyEditor"
], function(PropertyEditor) {
    "use strict";

    return PropertyEditor.extend("sap.ext.example.builder.PropertyEditor", {

        constructor: function(sId, mSettings) {
            PropertyEditor.apply(this, arguments);

            this.setI18nKeyPrefix("examplePlugin.");
            this.setResourceBundleName("sap.ext.example.i18n.builder");
            this.setPluginResourceBundleName("sap.ext.example.i18n.i18n");
        },

        addPropertyEditorContent: function(oPropertyFormContainer) {
            var oData = this.getPropertyData();

            this.addSwitch(oPropertyFormContainer, "notificationsEnabled", oData);
            this.addInputField(oPropertyFormContainer, "customProperty", oData);
            this.addInputField(oPropertyFormContainer, "maxRows", oData);
        },

        getDefaultPropertyData: function() {
            return {
                "notificationsEnabled": true,
                "customProperty": "",
                "maxRows": "10"
            };
        }
    });
});
```

### POD 1.0 Registration (designer/components.json)

```json
{
    "components": [
        {
            "id": "exampleView",
            "type": "VIEW_PLUGIN",
            "allowMultipleInstances": true,
            "name": "sap.ext.exampleplugins.exampleView",
            "propertyEditor": "sap.ext.exampleplugins.exampleView.builder.PropertyEditor",
            "i18n": "sap.ext.exampleplugins.exampleView.i18n.i18n",
            "supportedPodTypes": ["WORK_CENTER", "OPERATION", "ORDER", "OTHER", "MONITOR"]
        }
    ]
}
```

### POD 1.0 API Access

```javascript
// In Controller
onSomeEvent: function() {
    // Get POD controller
    var oPodController = this.getPodController();
    var sPlant = oPodController.getUserPlant();
    var sUserId = oPodController.getUserId();

    // Get selection model
    var oPodSelectionModel = this.getPodSelectionModel();
    var sResource = oPodSelectionModel.getResource();
    var aSelections = oPodSelectionModel.getSelections();

    // Get configuration
    var oConfiguration = this.getConfiguration();
    var bEnabled = oConfiguration.notificationsEnabled;

    // Execute action button
    this.executeActionButton("buttonId");

    // Publish event
    this.publish("CustomEvent", { data: "value" });

    // Show error
    this.showErrorMessage("Error occurred", true, true);
}
```

### POD 1.0 Project Structure

```
plugin/
├── Component.js                    # Main component
├── manifest.json                   # Component metadata
├── controller/
│   └── PluginView.controller.js   # Controller logic
├── view/
│   └── PluginView.view.xml        # UI definition
├── builder/
│   └── PropertyEditor.js          # Property configuration
├── i18n/
│   ├── i18n.properties           # Translations
│   └── builder.properties        # Property editor i18n
└── css/
    └── style.css                  # Styles

designer/
└── components.json                # Registration
```

---

## ControlWidget Pattern (For Single Controls)

ControlWidget wraps single SAPUI5 controls. This is the most common pattern for simple widgets.

### Basic ControlWidget Pattern

```javascript
sap.ui.define([
    "sap/m/Button",
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/propertyeditor/PropertyCategory"
], (Button, ControlWidget, WidgetCategory, PropertyCategory) => {
    "use strict";

    /**
     * @alias sap.dm.dme.pod2.widget.core.ButtonWidget
     * @extends sap.dm.dme.pod2.widget.ControlWidget
     */
    class ButtonWidget extends ControlWidget {
        static getDisplayName() {
            return "Button Widget";
        }

        static getIcon() {
            return "sap-icon://iphone-2";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        static getDefaultConfig() {
            return {
                properties: {
                    text: this.getDisplayName(),
                    type: "Ghost"  // ButtonType.Ghost
                }
            };
        }

        // Static configuration arrays (optional but recommended)
        static BINDABLE_PROPERTIES = ["text", "enabled", "visible"];
        static INCLUDE_EVENTS = ["press"];
        static EXCLUDE_PROPERTIES = ["somePropertyToHide"];
        static PROPERTY_CATEGORY_OVERRIDE = {
            text: PropertyCategory.Main,
            icon: PropertyCategory.Appearance,
            width: PropertyCategory.Dimension
        };

        /**
         * Constructor - pass SAPUI5 control class to super
         * @param {Object} oConfig
         */
        constructor(oConfig) {
            super(Button, oConfig);  // Pass the control class!
        }

        /**
         * Override for custom initialization
         * @override
         */
        _createView() {
            // Add custom logic before/after
            const oControl = super._createView();
            // Additional setup...
            return oControl;
        }
    }

    return ButtonWidget;
});
```

### Common ControlWidget Types

**Text Display:**
- `TextWidget` - Simple text
- `LabelWidget` - Form labels
- `TitleWidget` - Section titles
- `ExpandableTextWidget` - Collapsible text

**Input:**
- `InputWidget` - Text input
- `TextAreaWidget` - Multi-line input
- `DateTimeTextWidget` - Date/time display

**Buttons:**
- `ButtonWidget` - Standard button
- `MenuButtonWidget` - Button with dropdown

**Visual:**
- `IconWidget` - SAP icons
- `ImageWidget` - Images
- `HTMLWidget` - HTML content
- `IFrameWidget` - Embedded content

---

## LayoutWidget Pattern (For Containers)

LayoutWidget wraps layout containers that hold other widgets.

```javascript
sap.ui.define([
    "sap/m/VBox",
    "sap/dm/dme/pod2/widget/LayoutWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory"
], (VBox, LayoutWidget, WidgetCategory) => {
    "use strict";

    class VBoxWidget extends LayoutWidget {
        static getDisplayName() {
            return "Vertical Box";
        }

        static getIcon() {
            return "sap-icon://vertical-grip";
        }

        static getCategory() {
            return WidgetCategory.Layout;
        }

        constructor(oConfig) {
            super(VBox, oConfig);  // Pass container class
        }
    }

    return VBoxWidget;
});
```

### Common LayoutWidget Types

- `VBoxWidget`, `HBoxWidget` - Basic boxes
- `FlexBoxWidget` - Flexible layout
- `PanelWidget` - Panel with header
- `DialogWidget` - Modal dialog (Hidden category)
- `ToolbarWidget` - Toolbar
- `SplitterWidget` - Resizable split panes
- `ResponsiveSplitterWidget` - Responsive split panes

---

## TableWidget Pattern (For Complex Tables)

TableWidget is the most complex base class, used for displaying tabular data with columns, sorting, and pagination.

### Complete TableWidget Example

```javascript
sap.ui.define([
    "sap/m/library",
    "sap/ui/core/library",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/widget/core/TableWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory"
], (
    SapMLibrary,
    SapUiCoreLibrary,
    ModelPath,
    PodContext,
    TableWidget,
    WidgetCategory
) => {
    "use strict";

    const { ListMode, Text, ObjectIdentifier } = SapMLibrary;
    const { Priority, TextAlign } = SapUiCoreLibrary;

    /**
     * @alias sap.dm.dme.pod2.widget.custom.MyTableWidget
     * @extends sap.dm.dme.pod2.widget.core.TableWidget
     */
    class MyTableWidget extends TableWidget {

        /**
         * Define field names as enum for type safety
         * @enum {string}
         */
        static Field = Object.freeze({
            SFC: "sfc",
            Material: "material",
            Quantity: "quantity",
            Status: "status"
        });

        static getDisplayName() {
            return "My Table Widget";
        }

        static getIcon() {
            return "sap-icon://table-view";
        }

        /**
         * Define available columns with metadata
         * @override
         * @returns {Array}
         */
        static getFields() {
            const { Field } = this;
            return [
                {
                    field: Field.SFC,
                    text: "{i18n>sfc}",           // i18n key
                    importance: Priority.High,
                    width: "150px",
                    sortable: true
                },
                {
                    field: Field.Material,
                    text: "{i18n>material}",
                    width: "180px",
                    sortable: true
                },
                {
                    field: Field.Quantity,
                    text: "{i18n>quantity}",
                    width: "80px",
                    sortable: true,
                    hAlign: TextAlign.End      // Right-align numbers
                },
                {
                    field: Field.Status,
                    text: "{i18n>status}",
                    width: "100px",
                    sortable: false
                }
            ];
        }

        /**
         * Define which fields to show by default
         * @override
         * @returns {Array<string>}
         */
        static getDefaultFields() {
            const { Field } = this;
            return [Field.SFC, Field.Material, Field.Quantity];
        }

        /**
         * Define default configuration
         * @override
         */
        static getDefaultConfig() {
            // CRITICAL: Defensive null check - base Widget returns null!
            const oParentConfig = super.getDefaultConfig();
            const oParentProperties = oParentConfig?.properties || {};

            return {
                properties: {
                    ...oParentProperties,
                    mode: ListMode.SingleSelectMaster,
                    growingScrollToLoad: true,
                    pageSize: 100,
                    defaultSorting: [{
                        sortBy: this.Field.SFC,
                        descending: false
                    }]
                }
            };
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        static EXCLUDE_PROPERTIES = [
            ...TableWidget.EXCLUDE_PROPERTIES,
            "headerText"  // Hide specific properties
        ];

        constructor(oConfig) {
            super(oConfig);
        }

        /**
         * REQUIRED: Specify model path for table data
         * @override
         * @returns {string}
         */
        _getModelPath() {
            return ModelPath.WorkListItems;  // Or custom path like "/materials"
        }

        /**
         * Optional: Specify count path for pagination
         * @override
         * @returns {string}
         */
        _getCountPath() {
            return ModelPath.WorkListCount;
        }

        /**
         * REQUIRED: Create cell controls for each column
         * @override
         * @param {Object} oColumnConfig
         * @returns {sap.ui.core.Control}
         */
        _createCell(oColumnConfig) {
            switch (oColumnConfig.field) {
                case MyTableWidget.Field.SFC:
                    // Clickable identifier
                    return this._createIdentifierCell(oColumnConfig, "sfc");

                case MyTableWidget.Field.Material:
                case MyTableWidget.Field.Quantity:
                    // Simple text
                    return this._createTextCell(oColumnConfig, oColumnConfig.field);

                case MyTableWidget.Field.Status:
                    // Custom control
                    return new Text({
                        text: "{statusCode}"
                    });

                default:
                    // Handle custom fields
                    if (oColumnConfig.field.startsWith("customFields/")) {
                        return this._createTextCell(oColumnConfig, oColumnConfig.field);
                    }
                    throw new Error(`Unsupported field: ${oColumnConfig.field}`);
            }
        }

        /**
         * Optional: Handle sorting changes
         * @override
         */
        _onSort(aSorting) {
            PodContext.setWorkListSorting(aSorting);
            // Trigger data refresh
        }
    }

    return MyTableWidget;
});
```

### TableWidget Helper Methods

TableWidget provides these helper methods for creating cells:

```javascript
// Text cell with binding
_createTextCell(oColumnConfig, vBindPath)

// Identifier cell (clickable object name)
_createIdentifierCell(oColumnConfig, vBindPath)

// Date cell with formatting
_createDateCell(oColumnConfig, vBinding, fnFormatter)

// Quantity bullet chart
_createQuantityBulletChartCell(oBindPaths)
```

---

## ContentHandler Pattern (Business Logic)

ContentHandlers encapsulate complex business logic without UI. Used for form processing, dialogs, and workflows.

```javascript
sap.ui.define([
    "sap/m/Dialog",
    "sap/m/Button",
    "sap/m/ButtonType",
    "sap/ui/model/json/JSONModel",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/api/ApiClient",
    "sap/dm/dme/pod2/Logger",
    "sap/dm/dme/pod2/context/MessageHistory"
], (Dialog, Button, ButtonType, JSONModel, PodContext, ApiClient, Logger, MessageHistory) => {
    "use strict";

    /**
     * @alias sap.dm.dme.pod2.widget.custom.MyContentHandler
     */
    class MyContentHandler {
        #oModel = new JSONModel();
        #oDialog;
        #oLog = Logger.getLogger("sap.dm.dme.pod2.widget.custom.MyContentHandler");

        /**
         * Opens the content as a dialog
         * @param {Object} oData
         */
        async openAsDialog(oData) {
            this.#oModel.setData(oData);

            const oDialog = new Dialog({
                title: "My Dialog",
                content: [this._createForm()],
                buttons: [
                    new Button({
                        text: "Confirm",
                        type: ButtonType.Emphasized,
                        press: () => this._onConfirm()
                    }),
                    new Button({
                        text: "Cancel",
                        press: () => oDialog.close()
                    })
                ],
                afterClose: () => oDialog.destroy()
            });

            this.#oDialog = oDialog;
            oDialog.setModel(this.#oModel);
            oDialog.open();
        }

        _createForm() {
            // Create form controls
        }

        async _onConfirm() {
            const oData = this.#oModel.getData();

            try {
                await ApiClient.custom.post("/myEndpoint", oData);
                MessageHistory.toast({
                    message: "Success",
                    type: MessageHistory.Success
                });
            } catch (oError) {
                this.#oLog.error("Error", oError);
                MessageHistory.showError("Operation failed");
            }

            this.#oDialog.close();
        }
    }

    return MyContentHandler;
});
```

---

## Widget Categories

Available categories from `WidgetCategory`:
- `Elements` - Basic UI elements (buttons, inputs, text)
- `Layout` - Layout containers (panels, boxes, dialogs)
- `WorkList` - Work list related widgets
- `Order` - Order-related widgets
- `SFC` - Shop Floor Control widgets
- `DataCollection` - Data collection widgets
- `QuantityConfirmation` - Quantity reporting
- `ActivityConfirmation` - Activity confirmation
- `Assembly` - Assembly/component widgets
- `GoodsReceipt` - Goods receipt widgets
- `Hidden` - Not shown in widget palette (for DialogWidget, etc.)

---

## Common Patterns

### Quick Reference - Copy & Paste Examples

**Pattern 1: Display Current Resource**
```javascript
_createView() {
    this._oText = new Text({ text: "No resource" });
    return new VBox(this.getConfig().id, { items: [this._oText] });
}

onInit() {
    super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(ModelPath.FilterResources, (aRes, sPath) => {
            const resources = Array.isArray(aRes) ? aRes : [];
            this._oText.setText(resources[0]?.resource || "None");
        }, this);
    }
}
```

**Pattern 2: Button that Calls Custom API**
```javascript
_createView() {
    return new VBox(this.getConfig().id, {
        items: [
            new Button({
                text: "Load Data",
                press: () => this._onLoadData()
            })
        ]
    });
}

async _onLoadData() {
    try {
        const oData = await ApiClient.custom.post("/myEndpoint", {
            plant: PodContext.getPlant(),
            resource: PodContext.get(ModelPath.FilterResources)?.[0]?.resource
        });
        MessageHistory.toast({ message: "Success", type: MessageHistory.Success });
    } catch (oError) {
        Logger.error("Load failed", oError);
        MessageHistory.showError("Failed to load data");
    }
}
```

**Pattern 3: Simple Table Display**
```javascript
_createView() {
    this._oTable = new Table({
        columns: [
            new Column({ header: new Label({ text: "Item" }) }),
            new Column({ header: new Label({ text: "Status" }) })
        ]
    });
    return new VBox(this.getConfig().id, { items: [this._oTable] });
}

_updateTable(aItems) {
    this._oTable.removeAllItems();
    aItems.forEach(item => {
        this._oTable.addItem(new ColumnListItem({
            cells: [
                new Text({ text: item.name }),
                new Text({ text: item.status })
            ]
        }));
    });
}
```

**Pattern 4: Read Custom Property**
```javascript
static PropertyId = Object.freeze({
    ApiEndpoint: "apiEndpoint",
    RefreshInterval: "refreshInterval"
});

getProperties() {
    return [
        new WidgetProperty({
            displayName: "API Endpoint",
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "apiEndpoint", "/default")
        }),
        new WidgetProperty({
            displayName: "Refresh Interval (seconds)",
            category: "Main",
            propertyEditor: new IntegerPropertyEditor(this, "refreshInterval", 30)
        })
    ];
}

_someMethod() {
    const sEndpoint = this.getPropertyValue("apiEndpoint");
    const iInterval = this.getPropertyValue("refreshInterval");
    // Use values...
}
```

**Pattern 5: Subscribe to Work List Selection**
```javascript
onInit() {
    super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(ModelPath.SelectedWorkListItems, (aItems, sPath) => {
            const items = Array.isArray(aItems) ? aItems : [];
            if (items.length > 0) {
                this._handleSelection(items[0]);
            }
        }, this);
    }
}

_handleSelection(oItem) {
    const sSfc = oItem?.sfc;
    const sOrder = oItem?.order;
    const sMaterial = oItem?.material;
    // Use data...
}
```

---

### Basic Widget Example (POD 2.0)

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/m/VBox",
    "sap/m/Text",
    "sap/m/Button",
    "sap/m/MessageToast"
], (Widget, PodContext, ModelPath, VBox, Text, Button, MessageToast) => {
    "use strict";

    class BasicPlugin extends Widget {

        static getDisplayName() {
            return "Basic Plugin";
        }

        static getIcon() {
            return "sap-icon://factory";
        }

        static getCategory() {
            return "Custom Widgets";
        }

        static getDescription() {
            return "Basic example plugin with event subscription";
        }

        onInit() {
            super.onInit();

            // Subscribe to resource changes
            if (PodContext.isRunMode()) {
                PodContext.subscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }
        }

        _createView() {
            const oConfig = this.getConfig();

            // Validate config
            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [new Text({ text: "Configuration error" })]
                });
            }

            // Store control reference for updates
            this._oResourceText = new Text({
                text: "No resources selected"
            });

            // Pass oConfig.id as first parameter
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

        // CRITICAL: Callback signature is (newValue, path) NOT (path, newValue)!
        _onResourceChanged(aResources, sPath) {
            // Coerce to array to prevent crashes
            const resources = Array.isArray(aResources) ? aResources : [];
            this._updateResourceDisplay(resources);
        }

        _updateResourceDisplay(aResources) {
            if (this._oResourceText) {
                // Double-check type and use optional chaining
                const sText = Array.isArray(aResources) && aResources.length > 0
                    ? aResources.map(r => r?.resource || "Unknown").join(", ")
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

            // Always unsubscribe to prevent memory leaks
            if (PodContext.isRunMode()) {
                PodContext.unsubscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }

            // Clean up control references
            this._oResourceText = null;
        }
    }

    return BasicPlugin;
});
```

---

## Testing Your Plugin

### Pre-Upload Checklist

**Before creating your deployment zip:**

✅ **Validate extension.json structure**
- Run through JSON validator (jsonlint.com)
- Ensure ONLY `widgets` and `actions` arrays exist at root
- No `name`, `description`, `version`, or `provider` fields
- `modulePath` matches physical file location
- `type` uses dots not slashes
- No `.js` extension in modulePath

✅ **Verify file structure**
```
your-extension/
├── extension.json     # ✅ At root
└── plugins/           # ✅ At root
    ├── widget1.js
    └── widget2.js
```

✅ **Code validation**
- All JavaScript files use ES6 class syntax
- All widgets extend correct base class (Widget, ControlWidget, etc.)
- All required static methods implemented (getDisplayName, getIcon, getCategory)
- `_createView()` returns valid UI5 control
- All imports use correct paths (`pod2/context/` not `pod2/model/`)

✅ **Create deployment zip correctly**
```bash
# Windows PowerShell
cd your-extension-folder
Compress-Archive -Path extension.json,plugins -DestinationPath my-extension.zip -Force

# Mac/Linux
cd your-extension-folder
zip -r my-extension.zip extension.json plugins/
```

✅ **Verify zip contents**
```bash
# Check that extension.json is at root (not nested in folder)
unzip -l my-extension.zip
# Should show:
#   extension.json
#   plugins/widget1.js
#   plugins/widget2.js
```

---

### Post-Upload Testing

**After uploading to SAP DM Extension Center:**

#### 1. Check Browser Console
- Open Developer Tools (F12)
- Look for JavaScript errors or 404s
- Common errors:
  - `404` → Wrong modulePath or import path
  - `Module not found` → File location mismatch
  - `Unexpected token` → Syntax error in code

#### 2. Verify Widget Appears in POD Designer
- Navigate to POD Designer
- Check widget palette for your widget
- If missing:
  - Check console for errors
  - Verify widget extends correct base class
  - Ensure `getCategory()` returns valid category
  - Check that `getDisplayName()` is implemented

#### 3. Test in Configuration Mode
- Drag widget onto canvas
- Verify it appears without errors
- Open properties panel
- Test property editors
- Check that default values are applied

#### 4. Test in Run Mode
- Save POD configuration
- Switch to Run Mode
- Verify widget renders correctly
- Test PodContext subscriptions
- Verify data updates when context changes

#### 5. Test API Calls
- Open Network tab in Developer Tools
- Trigger API calls from your widget
- Verify:
  - Correct endpoints are called
  - Request payloads are correct
  - Responses are handled properly
  - Errors are handled gracefully

---

### Common Testing Scenarios

#### Test: Resource Change Subscription
```javascript
// 1. Add console.log to verify callback fires
_onResourceChanged(aResources, sPath) {
    console.log("Resource changed:", aResources, sPath);
    const resources = Array.isArray(aResources) ? aResources : [];
    // ... rest of logic
}

// 2. In POD Run Mode, change resource selection
// 3. Check console for log output
// 4. Verify widget updates correctly
```

#### Test: API Call with Error Handling
```javascript
async _fetchData() {
    try {
        console.log("Fetching data...");
        const oData = await ApiClient.custom.get("/myEndpoint");
        console.log("Data received:", oData);
        this._updateDisplay(oData);
    } catch (oError) {
        console.error("API call failed:", oError);
        MessageHistory.showError("Failed to load data");
    }
}

// Check Network tab for:
// - Request sent to correct endpoint
// - Response status (200, 404, 500, etc.)
// - Response data structure
```

#### Test: Property Changes
```javascript
// 1. Add console.log in property handler
setPropertyValue(sName, vValue) {
    console.log(`Property ${sName} changed to:`, vValue);
    super.setPropertyValue(sName, vValue);
    this._handlePropertyChange(sName, vValue);
}

// 2. Change property in POD Designer
// 3. Verify console output
// 4. Check that widget updates accordingly
```

---

### Troubleshooting Testing Issues

**Widget not updating when context changes:**
```javascript
// Check 1: Are you subscribing?
onInit() {
    super.onInit();
    if (PodContext.isRunMode()) {  // ← Must check isRunMode()!
        PodContext.subscribe(ModelPath.FilterResources, this._handler, this);
    }
}

// Check 2: Correct parameter order?
_handler(aData, sPath) {  // ← Data FIRST, path SECOND!
    const data = Array.isArray(aData) ? aData : [];
    // ...
}

// Check 3: Are you unsubscribing on exit?
onExit() {
    super.onExit();
    if (PodContext.isRunMode()) {
        PodContext.unsubscribe(ModelPath.FilterResources, this._handler, this);
    }
}
```

**TypeError in subscription callback:**
```javascript
// Always use defensive type checking:
_handler(aData, sPath) {
    // Step 1: Coerce to expected type
    const data = Array.isArray(aData) ? aData : [];

    // Step 2: Use optional chaining
    const safe = data.map(item => item?.property || "default");

    // Step 3: Handle empty arrays
    if (safe.length === 0) {
        this._showEmpty();
        return;
    }

    this._showData(safe);
}
```

**API calls not working:**
```javascript
// Check 1: Using correct client?
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";  // ✅ POD 2.0
// NOT: import AjaxUtil from "sap/dm/dme/model/AjaxUtil";  // ❌ POD 1.0

// Check 2: Using async/await?
async _loadData() {  // ← Must be async!
    try {
        const oData = await ApiClient.custom.get("/endpoint");  // ← await!
        return oData;
    } catch (oError) {
        Logger.error("Load failed", oError);
        throw oError;
    }
}

// Check 3: Timeout protection?
const oData = await Promise.race([
    ApiClient.custom.get("/endpoint"),
    this._createTimeoutPromise(30000)
]);
```

---

## Best Practices

### 🏭 Production Patterns from SAP Code

**These patterns are extracted from real SAP POD 2.0 production code.** See [references/production-patterns-sap.md](references/production-patterns-sap.md) for complete details.

**Top 10 Production Patterns:**

1. **Use Object.freeze for Enums**
   ```javascript
   static PropertyId = Object.freeze({
       ...super.PropertyId,  // Spread parent!
       CustomField: "customField"
   });
   ```

2. **Use Private Fields (#)**
   ```javascript
   #oLog = Logger.getLogger("custom.MyWidget");
   #oTable;  // Store control references as private
   ```

3. **Check isRunMode() Before Subscribing**
   ```javascript
   onInit() {
       super.onInit();
       if (PodContext.isRunMode()) {  // ← CRITICAL!
           PodContext.subscribe(ModelPath.X, this._handler, this);
       }
   }
   ```

4. **Simple Direct Properties** (Official SAP Pattern)
   ```javascript
   static getDefaultConfig() {
       return {
           properties: {
               myProp: "value"  // Direct, no spreading!
           }
       };
   }
   ```

5. **Runtime Default Coalescing** (Optional - for missing properties)
   ```javascript
   getPropertyValue(sName) {
       const vValue = super.getPropertyValue(sName);
       switch (sName) {
           case "myProp":
               return vValue || "defaultValue";
       }
       return vValue;
   }
   ```

5. **Type Cast After getView()**
   ```javascript
   const oControl = /** @type {sap.m.Table} */(this.getView());
   ```

6. **Always Destroy Dialogs**
   ```javascript
   const oDialog = new Dialog({
       afterClose: () => oDialog.destroy()  // ← Prevents memory leaks!
   });
   ```

7. **Defensive Array Handling**
   ```javascript
   PodContext.subscribe(ModelPath.FilterResources, (aResources) => {
       if (Array.isArray(aResources) && aResources.length !== 0) {
           // Safe to use
       }
   });
   ```

8. **Use JSDoc Documentation**
   ```javascript
   /**
    * @override
    * @extensible
    * @returns {string}
    */
   static getDisplayName() {
       return "My Widget";
   }
   ```

9. **Different Controls for Design vs Run Mode**
   ```javascript
   constructor(oConfig) {
       if (PodContext.isDesignMode()) {
           super(SimplePreviewControl, oConfig);
       } else {
           super(FullFeaturedControl, oConfig);
       }
   }
   ```

10. **ContentHandler Pattern**
    ```javascript
    class MyContentHandler {
        _oModel;
        _oDialog;

        async openAsDialog(oData) {
            this._oModel.setData(oData);
            // Create and open dialog...
        }
    }
    ```

---

### ✅ DO:

1. **Use the correct base class**
   - `ControlWidget` for single controls
   - `LayoutWidget` for containers
   - `TableWidget` for tables
   - Never extend `Widget` directly unless creating a new base pattern

2. **Always call super methods first**
   ```javascript
   async onInit() {
       await super.onInit();  // FIRST!
       // your code
   }
   ```

3. **Handle null/undefined**
   ```javascript
   const oItem = PodContext.getLastSelectedWorkListItem();
   if (!oItem) {
       this.#oLog.error("No item selected");
       return;
   }
   ```

4. **Clean up subscriptions**
   ```javascript
   onExit() {
       super.onExit();
       PodContext.unsubscribe(ModelPath.X, this._handler, this);
   }
   ```

5. **Use i18n for all text**
   ```javascript
   text: "{i18n>myKey}"
   // or
   text: PodContext.getI18nText("myKey", [param1])
   ```

6. **Use private fields for state**
   ```javascript
   #oLog = Logger.getLogger("...");
   #oModel = new JSONModel();
   ```

7. **Type safety with Field enums**
   ```javascript
   static Field = Object.freeze({
       SFC: "sfc",
       Material: "material"
   });
   ```

### ❌ DON'T:

1. **Don't mix POD 1.0 and POD 2.0 paths**
   ```javascript
   // WRONG
   "sap/dm/dme/podfoundation/..."

   // CORRECT
   "sap/dm/dme/pod2/..."
   ```

2. **Don't forget super calls**
   ```javascript
   // WRONG
   onInit() {
       this._doSomething();
   }

   // CORRECT
   async onInit() {
       await super.onInit();
       this._doSomething();
   }
   ```

3. **Don't create controls in constructor**
   ```javascript
   // WRONG
   constructor(oConfig) {
       super(Button, oConfig);
       this.myControl = new Text();  // Too early!
   }

   // CORRECT
   _createView() {
       const oView = super._createView();
       this.myControl = new Text();
       return oView;
   }
   ```

4. **Don't skip error handling**
   ```javascript
   // WRONG
   async loadData() {
       const oData = await ApiClient.getData();
       this.setData(oData);
   }

   // CORRECT
   async loadData() {
       try {
           const oData = await ApiClient.custom.get("/data");
           this.setData(oData);
       } catch (oError) {
           this.#oLog.error("Load failed", oError);
           MessageHistory.showError("Failed to load data");
       }
   }
   ```

5. **Don't forget to unsubscribe**
   ```javascript
   // Memory leak!
   onInit() {
       PodContext.subscribe(ModelPath.X, this._handler, this);
       // Missing unsubscribe in onExit()!
   }
   ```

---

## Common Issues and Solutions

### POD 2.0 Deployment Errors

**Error: "Failed to create custom extensions: Error encountered when processing the extension components file"**

**Cause:** Invalid extension.json structure

**Solutions:**
1. ✅ Remove ALL metadata fields from extension.json root level:
   - Remove: `name`, `description`, `version`, `provider`, `author`
   - Keep ONLY: `widgets` and `actions` arrays

2. ✅ Verify extension.json has correct structure:
   ```json
   {
     "widgets": [...],
     "actions": []
   }
   ```

3. ✅ Check modulePath matches physical file location:
   - modulePath: `custom/pod2/demo/plugins/mywidget`
   - File location: `plugins/mywidget.js` (NOT `plugins/custom/pod2/demo/mywidget.js`)

4. ✅ Ensure type field uses dots not slashes:
   - Correct: `"type": "custom.pod2.demo.plugins.mywidget"`
   - Wrong: `"type": "custom/pod2/demo/plugins/mywidget"`

5. ✅ Verify zip file structure:
   - Zip root should contain `extension.json` and `plugins/` folder
   - NOT a root folder containing these files

**Error: "Widget not appearing in POD Designer"**

**Solutions:**
1. ✅ Check browser console for JavaScript errors
2. ✅ Verify widget class extends correct base class: `sap/dm/dme/pod2/widget/Widget`
3. ✅ Ensure all static methods are implemented:
   - `static getDisplayName()`
   - `static getIcon()`
   - `static getCategory()`
   - `static getDescription()`
4. ✅ Check that `_createView()` returns a valid UI5 control
5. ✅ Verify ES6 class syntax is used (NOT `.extend()`)

**Error: "Module not found" or "Failed to load module"**

**Solutions:**
1. ✅ Check modulePath in extension.json matches file location exactly
2. ✅ Ensure no `.js` extension in modulePath
3. ✅ Verify file is in correct directory (`plugins/` not nested namespace folders)
4. ✅ Check for typos in modulePath and type fields
5. ✅ Ensure class name matches file name (case-sensitive)

### POD 2.0 Runtime Errors

**Error: "TypeError: Cannot read properties of null (reading 'properties')" in getDefaultConfig()**

**Cause:** The base `Widget` class's `getDefaultConfig()` returns `null`, not an empty object

**Solution:**
```javascript
// ❌ WRONG - Crashes when super returns null OR introduces property conflicts
static getDefaultConfig() {
    return {
        properties: {
            ...super.getDefaultConfig().properties,  // 💥 null.properties OR type conflicts!
            myProperty: "default"
        }
    };
}

// ✅ CORRECT - Simple direct properties (Official SAP pattern)
static getDefaultConfig() {
    return {
        properties: {
            myProperty: "default"  // Direct, no spreading
        }
    };
}

// ✅ ALSO CORRECT - Runtime default coalescing for missing properties
getPropertyValue(sName) {
    const vValue = super.getPropertyValue(sName);
    switch (sName) {
        case "myProperty":
            return vValue || "default";  // Coalesce if absent
    }
    return vValue;
}
```

**Error: "PodContext is not defined" or "404 - Failed to load PodContext.js"**

**Cause:** Incorrect import paths for PodContext and ModelPath

**Solution:**
```javascript
// ❌ WRONG - These paths don't exist
"sap/dm/dme/pod2/model/PodContext"
"sap/dm/dme/pod2/model/ModelPath"

// ✅ CORRECT - Use context/ not model/
"sap/dm/dme/pod2/context/PodContext"
"sap/dm/dme/pod2/context/ModelPath"
```

**Full correct import:**
```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",    // ← Use context/
    "sap/dm/dme/pod2/context/ModelPath"      // ← Use context/
], (Widget, PodContext, ModelPath) => {
```

**Error: "getView method returned a view with a different ID than configuration"**

**Cause:** View was created without passing the configuration ID

**Solution:**
Always pass `oConfig.id` as the FIRST parameter when creating the root view:

```javascript
// ❌ WRONG - No ID passed
_createView() {
    return new VBox({
        items: [...]
    });
}

// ✅ CORRECT - Pass oConfig.id as first parameter
_createView() {
    const oConfig = this.getConfig();

    // Validate config
    if (!oConfig || !oConfig.id) {
        return new VBox({
            items: [new Text({ text: "Configuration error" })]
        });
    }

    // Pass ID as first parameter (UI5 pattern)
    const oView = new VBox(oConfig.id, {
        items: [...]
    });

    return oView;
}
```

**Why:** SAP UI5 controls accept ID as the first constructor parameter. POD 2.0 requires the view ID to match the widget configuration ID.

**Error: "this.createId is not a function" or "this.byId is not a function"**

**Cause:** Using Controller/Component methods that don't exist in Widget class

**Solution:**
Don't use `this.createId()` or `this.byId()` - these are not available in POD 2.0 widgets.

Instead, store direct references to controls:

```javascript
// ❌ WRONG - createId/byId not available in widgets
_createView() {
    return new VBox({
        items: [
            new Text({
                id: this.createId("myText"),  // ❌ Error!
                text: "Hello"
            })
        ]
    });
}

_someMethod() {
    const oText = this.byId("myText");  // ❌ Error!
}

// ✅ CORRECT - Store direct references
_createView() {
    this._oMyText = new Text({
        text: "Hello"
    });

    return new VBox({
        items: [this._oMyText]
    });
}

_someMethod() {
    if (this._oMyText) {
        this._oMyText.setText("Updated!");  // ✅ Works!
    }
}

// Clean up in onExit
onExit() {
    super.onExit();
    this._oMyText = null;
}
```

**Error: "Cannot read property of undefined" when accessing context**

**Solution:**
```javascript
// Always check PodContext.isRunMode() before subscribing
onInit() {
    super.onInit();
    if (PodContext.isRunMode()) {  // ← Add this check
        PodContext.subscribe(ModelPath.FilterResources, this._onResourceChanged, this);
    }
}
```

**Error: "TypeError: aResources.map is not a function" or "Cannot read property 'map' of undefined"**

**Cause:** PodContext subscription callbacks can receive various data types (undefined, null, non-array).

**Solution:**
Always validate data types in subscription handlers AND use correct parameter order:

```javascript
// ❌ WRONG - Wrong parameter order AND assumes data is always an array
_onResourceChanged(sPath, aResources) {
    const list = aResources.map(r => r.resource).join(", ");  // ❌ Crashes!
    this._oText.setText(list);
}

// ✅ CORRECT - Correct parameter order with defensive type checking
_onResourceChanged(aResources, sPath) {
    // Step 1: Coerce to array type
    const resources = Array.isArray(aResources) ? aResources : [];

    // Step 2: Pass validated data
    this._updateResourceDisplay(resources);
}

_updateResourceDisplay(aResources) {
    if (this._oText) {
        // Step 3: Double-check with Array.isArray()
        const sText = Array.isArray(aResources) && aResources.length > 0
            // Step 4: Use optional chaining (?.) for properties
            ? aResources.map(r => r?.resource || "Unknown").join(", ")
            : "No resources selected";

        this._oText.setText(sText);
    }
}
```

**Why This Happens:**
PodContext can send different data types:
- `[]` - Empty array (no items selected)
- `undefined` - Data cleared or not yet loaded
- `null` - Context reset
- Non-array types in edge cases

**Defensive Coding Pattern:**
```javascript
// Always follow this pattern for subscription handlers:

// CRITICAL: Parameters are (newValue, path) NOT (path, newValue)!
// 1. Coerce to expected type in handler
_onDataChanged(vData, sPath) {
    const data = Array.isArray(vData) ? vData : [];
    this._updateDisplay(data);
}

// 2. Verify type again in update method (defense in depth)
_updateDisplay(aData) {
    if (!Array.isArray(aData)) {
        console.warn("Expected array, got:", typeof aData);
        return;
    }

    // 3. Use optional chaining for property access
    const items = aData.map(item => item?.name || "Unknown");

    // 4. Handle empty arrays gracefully
    const text = items.length > 0 ? items.join(", ") : "No data";
}
```

**Error: Memory leaks or widget not updating**

**Solution:**
```javascript
// Always unsubscribe in onExit
onExit() {
    super.onExit();
    if (PodContext.isRunMode()) {
        PodContext.unsubscribe(ModelPath.FilterResources, this._onResourceChanged, this);
    }
}
```

### POD 2.0 Common Coding Issues

1. **CRITICAL: Use correct base class** - `sap/dm/dme/pod2/widget/Widget` (NOT podfoundation paths!)
2. **Use ES6 class syntax** - `class MyWidget extends Widget` (NOT `.extend()`)
3. **Implement _createView()** - Must return view or controls
4. **Use static methods** - `static getDisplayName()` not `getPluginName: function()`
5. **Always cleanup in onExit()** - Unsubscribe from context changes
6. **Check PodContext.isRunMode()** - Before subscribing to avoid errors in config mode
7. **modulePath must match files** - Don't create nested namespace folders
8. **Validate user input** - Prevent injection attacks
9. **Handle async errors** - Use try/catch with async/await
10. **Implement timeout protection** - For all external API calls
11. **CRITICAL: Defensive type checking in subscription handlers** - Always use `Array.isArray()` and optional chaining
12. **Coerce data to expected type** - PodContext can send undefined, null, or non-array types
13. **🚨 CRITICAL: PodContext callback parameter order** - Signature is `(newValue, path)` NOT `(path, newValue)`!

### POD 2.0 Packaging Checklist

Before uploading your extension:

- [ ] extension.json has ONLY `widgets` and `actions` arrays
- [ ] No metadata fields in extension.json (name, version, description, provider)
- [ ] modulePath matches physical file location
- [ ] type field uses dots (not slashes)
- [ ] No .js extension in modulePath
- [ ] All widget files are in `plugins/` folder
- [ ] All action files are in `actions/` folder (if any)
- [ ] Zip contains extension.json at root level
- [ ] Zip contains plugins/ folder at root level
- [ ] No nested root folder in zip
- [ ] All JavaScript files use ES6 class syntax
- [ ] All widgets extend Widget base class
- [ ] All static metadata methods implemented
- [ ] _createView() method returns valid controls

### Creating the Deployment Package

**Correct zip structure:**
```
your-extension.zip
├── extension.json
└── plugins/
    ├── widget1.js
    ├── widget2.js
    └── i18n/
        ├── i18n_en.properties
        └── i18n_de.properties
```

**WRONG - Do NOT include root folder:**
```
your-extension.zip
└── your-extension/        ← ❌ Remove this nested folder
    ├── extension.json
    └── plugins/
```

**How to create zip correctly:**

**On Windows (PowerShell):**
```powershell
cd your-extension-folder
Compress-Archive -Path extension.json,plugins -DestinationPath my-extension.zip -Force
```

**On Mac/Linux:**
```bash
cd your-extension-folder
zip -r my-extension.zip extension.json plugins/
```

**Verify zip contents before upload:**
```powershell
# Extract to temp folder and check structure
Expand-Archive -Path my-extension.zip -DestinationPath temp-check -Force
Get-ChildItem -Path temp-check -Recurse
```

Expected output should show `extension.json` and `plugins/` at the root level, NOT nested in another folder.

### IMPORTANT: Namespace Information for Upload

**When creating a POD plugin, ALWAYS notify the user of the namespace used. This information is REQUIRED during the upload process in SAP Digital Manufacturing.**

After generating a plugin, provide a clear summary:

```
📋 PLUGIN NAMESPACE INFORMATION

Your plugin uses the following namespace:
  Namespace: custom/pod2/yourcompany
  Module Path: custom/pod2/yourcompany/plugins/yourplugin
  Type: custom.pod2.yourcompany.plugins.yourplugin

⚠️ IMPORTANT FOR UPLOAD:
When uploading this extension to SAP Digital Manufacturing, you will be
asked to provide the namespace. Use: custom/pod2/yourcompany

The namespace is used by SAP DM to:
- Organize custom extensions
- Prevent naming conflicts
- Enable/disable extensions by namespace
```

**Why This Matters:**
1. SAP DM Extension Center requires namespace during upload
2. Namespace groups related plugins together
3. Enables selective activation/deactivation
4. Prevents conflicts with other extensions

**Namespace Format:**
- Structure: `vendor/pod2/identifier`
- Example: `custom/pod2/acme` for ACME company
- Example: `custom/pod2/manufacturing` for manufacturing department
- Must be lowercase, use forward slashes

**What to Tell Users:**
When you create a plugin, end your response with:

```
🎯 NAMESPACE TO USE DURING UPLOAD: custom/pod2/yourcompany

Make note of this namespace - you'll need it when uploading the extension
to SAP Digital Manufacturing's Extension Center.
```

### POD 1.0 Common Issues

1. **Always call parent lifecycle methods** - Use `.apply(this, arguments)`
2. **Subscribe in onBeforeRenderingPlugin** - Not in onInit
3. **Always unsubscribe in onExit** - Prevent memory leaks
4. **Check isEventFiredByThisPlugin()** - Avoid circular event loops
5. **Complete folder structure** - Include all required files (manifest, i18n, CSS)
6. **Correct namespace in components.json** - Must match folder structure
7. **Execution plugins must call complete()** - For async operations

---

## Migration Path: POD 1.0 → POD 2.0

### Step-by-Step Migration

1. **Component → Widget class**
   - Replace ProductionUIComponent with Widget
   - Move controller logic into widget class

2. **Controller lifecycle → Widget lifecycle**
   - `onInit` → `onInit()` with `super.onInit()`
   - `onBeforeRenderingPlugin` → Move to `onInit()`
   - Event subscriptions → PodContext subscriptions

3. **XML View → _createView() method**
   - Convert XML markup to programmatic UI5 controls

4. **PropertyEditor → getProperties() method**
   - Convert to WidgetProperty array with property editors

5. **components.json → extension.json**
   - Update registration format
   - Deploy to Manage PODs 2.0

6. **API access patterns**
   - `this.getPodController()` → `PodContext.getPlant()`
   - `this.subscribe()` → `PodContext.subscribe()`
   - `this.getConfiguration()` → `this.getPropertyValue()`
   - `AjaxUtil.post(url, data, success, error)` → `await ApiClient.custom.post("/endpoint", data)`
   - `AjaxUtil.get(url, success, error)` → `await ApiClient.custom.get("/endpoint")`

---

## Quick Decision Guide

### Choose Your Base Class:

| Need | Use |
|------|-----|
| Single UI control (button, input, text) | **ControlWidget** |
| Container/layout | **LayoutWidget** |
| Table with data | **TableWidget** |
| Business logic without UI | **ContentHandler** |

### Essential Static Methods:

Every widget needs:
- `getDisplayName()` - Widget name
- `getIcon()` - Icon for palette
- `getCategory()` - Category in palette
- `getDefaultConfig()` - Default properties

### Essential Lifecycle:

- `constructor(oConfig)` - Initialize
- `async onInit()` - After construction (always call super first!)
- `_createView()` - Create UI
- `onExit()` - Cleanup (always unsubscribe!)

### Common Imports:

```javascript
"sap/dm/dme/pod2/widget/ControlWidget"       // or LayoutWidget/TableWidget
"sap/dm/dme/pod2/context/PodContext"         // State management
"sap/dm/dme/pod2/context/ModelPath"          // Model paths
"sap/dm/dme/pod2/api/ApiClient"              // API calls
"sap/dm/dme/pod2/Logger"                     // Logging
"sap/dm/dme/pod2/context/MessageHistory"     // User messages
"sap/dm/dme/pod2/widget/metadata/WidgetCategory"
"sap/dm/dme/pod2/widget/metadata/WidgetProperty"
"sap/dm/dme/pod2/propertyeditor/PropertyCategory"
"sap/m/library"                              // SAPUI5 controls
```

---

## When to Use This Skill

Use this skill when users ask to:
- Create a POD plugin or POD 1.0/2.0 plugin
- Scaffold a Digital Manufacturing extension
- Add functionality to POD
- Work with POD Designer
- Access work center, resource, or operation data
- Create custom POD tables or actions
- Migrate POD 1.0 plugins to POD 2.0

## Instructions

When helping users:

1. **Clarify POD Version**: Ask if they're targeting POD 1.0 or POD 2.0 (recommend POD 2.0)

2. **For POD 2.0 - Collect Required Information**:
   - **Plugin Name** (lowercase letters only, e.g., "mywidget", "worklistplugin")
     - Used for the JavaScript class name
     - Must be lowercase and contain only letters
   - **POD Designer Name** (display name, e.g., "My Custom Widget")
     - The name displayed in the POD Designer UI
     - Used in `static getDisplayName()` method
   - **POD Designer Group** (category name, e.g., "Custom Widgets", "Work Center Tools")
     - The group/category in POD Designer where the plugin appears
     - Used in `static getCategory()` method
   - **Icon** (SAP Icon, e.g., "sap-icon://factory", "sap-icon://locate-me-2")
     - Select from SAP Icon Explorer: https://ui5.sap.com/test-resources/sap/m/demokit/iconExplorer/webapp/index.html
     - Used in `static getIcon()` method
   - **Description** (brief description of plugin functionality)
     - Used in `static getDescription()` method
   - **Version** (default: "0.0.1")
     - Semantic versioning for the plugin
   - **Namespace** (e.g., "custom/pod2/mycompany")
     - Base namespace for the plugin files
     - Used in modulePath and file structure
   - **Languages** (i18n support)
     - Languages to generate i18n property files for
     - Default: ["en"]
     - See **SAP DM Supported Languages** section below for full list

3. **For POD 1.0 - Collect Required Information**:
   - Plugin type (VIEW_PLUGIN or EXECUTION_PLUGIN)
   - Component namespace
   - Plugin display name
   - Supported POD types (WORK_CENTER, OPERATION, ORDER, OTHER, MONITOR)

4. **Create Structure**: Generate proper directory structure with extension.json or components.json
5. **Implement Plugin**: Use correct base class and lifecycle methods
6. **Add Configuration**: Include property editors if needed
7. **Provide Examples**: Show how to access POD context and call APIs
8. **Document**: Explain deployment process
9. **CRITICAL: Notify Namespace**: Always provide clear namespace information that will be needed during upload

### Example POD 2.0 Plugin Creation Flow

When user requests a POD 2.0 plugin, ask:
```
To create your POD 2.0 plugin, I need the following information:

1. Plugin Name (lowercase letters only): _____
2. POD Designer Name (display name): _____
3. POD Designer Group (category): _____
4. Icon (from SAP Icon Explorer): _____
5. Description: _____
6. Version (default 0.0.1): _____
7. Namespace: _____ (IMPORTANT: Required for upload to SAP DM)
8. Languages for i18n (default: en): _____
```

Then generate:
- Main plugin JavaScript file with proper class name and metadata
- extension.json with correct modulePath and type
- i18n property files for each language
- Project folder structure
- README with deployment instructions
- **NAMESPACE NOTIFICATION** with upload instructions

### Parameter Usage Example

Given these inputs:
- Plugin Name: `worklistwidget`
- POD Designer Name: `Work List Widget`
- POD Designer Group: `Production Tools`
- Icon: `sap-icon://list`
- Description: `Displays work list for selected resource`
- Version: `1.0.0`
- Namespace: `custom/pod2/mycompany`
- Languages: `["en", "de"]`

Generate this structure:
```
project/
├── extension.json
├── plugins/
│   ├── worklistwidget.js
│   └── i18n/
│       ├── i18n_en.properties
│       └── i18n_de.properties
└── README.md
```

**extension.json**:
```json
{
  "widgets": [
    {
      "modulePath": "custom/pod2/mycompany/plugins/worklistwidget",
      "type": "custom.pod2.mycompany.plugins.worklistwidget"
    }
  ],
  "actions": []
}
```

**IMPORTANT**: Do NOT include `name`, `description`, `version`, or `provider` fields in extension.json. Only include `widgets` and `actions` arrays.

**plugins/worklistwidget.js**:
```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/m/VBox"
], (Widget, VBox) => {
    "use strict";

    class WorklistWidget extends Widget {

        static getDisplayName() {
            return "Work List Widget";
        }

        static getIcon() {
            return "sap-icon://list";
        }

        static getCategory() {
            return "Production Tools";
        }

        static getDescription() {
            return "Displays work list for selected resource";
        }

        _createView() {
            return new VBox({
                items: [
                    // Widget UI controls
                ]
            });
        }
    }

    return WorklistWidget;
});
```

**plugins/i18n/i18n_en.properties**:
```properties
# Work List Widget - English
worklistwidget.title=Work List
worklistwidget.noData=No work items available
```

**plugins/i18n/i18n_de.properties**:
```properties
# Work List Widget - German
worklistwidget.title=Arbeitsliste
worklistwidget.noData=Keine Arbeitsaufträge verfügbar
```

---

## SAP DM Supported Languages

SAP Digital Manufacturing supports the following languages for i18n. Use the locale codes when creating i18n property files:

| Language | Locale Code | File Name |
|----------|-------------|-----------|
| Bulgarian | bg | i18n_bg.properties |
| Simplified Chinese | zh_CN | i18n_zh_CN.properties |
| Traditional Chinese | zh_TW | i18n_zh_TW.properties |
| Croatian | hr | i18n_hr.properties |
| Czech | cs | i18n_cs.properties |
| Danish | da | i18n_da.properties |
| Dutch | nl | i18n_nl.properties |
| English | en | i18n_en.properties |
| French | fr | i18n_fr.properties |
| German | de | i18n_de.properties |
| Hungarian | hu | i18n_hu.properties |
| Italian | it | i18n_it.properties |
| Japanese | ja | i18n_ja.properties |
| Korean | ko | i18n_ko.properties |
| Lithuanian | lt | i18n_lt.properties |
| Polish | pl | i18n_pl.properties |
| Portuguese (Brazilian) | pt | i18n_pt.properties |
| Romanian | ro | i18n_ro.properties |
| Russian | ru | i18n_ru.properties |
| Serbian (Latin) | sr | i18n_sr.properties |
| Slovak | sk | i18n_sk.properties |
| Slovenian | sl | i18n_sl.properties |
| Spanish | es | i18n_es.properties |
| Swedish | sv | i18n_sv.properties |
| Thai | th | i18n_th.properties |
| Turkish | tr | i18n_tr.properties |
| Ukrainian | uk | i18n_uk.properties |
| Vietnamese | vi | i18n_vi.properties |

**Notes:**
- Always include `i18n.properties` (without locale code) as the default fallback
- Spanish: Only standard Spanish is supported (other variants are not supported)
- The default file `i18n.properties` should contain English text as the fallback

**Example i18n folder structure with all languages:**
```
plugins/
└── i18n/
    ├── i18n.properties        # Default (English fallback)
    ├── i18n_bg.properties     # Bulgarian
    ├── i18n_cs.properties     # Czech
    ├── i18n_da.properties     # Danish
    ├── i18n_de.properties     # German
    ├── i18n_en.properties     # English
    ├── i18n_es.properties     # Spanish
    ├── i18n_fr.properties     # French
    ├── i18n_hr.properties     # Croatian
    ├── i18n_hu.properties     # Hungarian
    ├── i18n_it.properties     # Italian
    ├── i18n_ja.properties     # Japanese
    ├── i18n_ko.properties     # Korean
    ├── i18n_lt.properties     # Lithuanian
    ├── i18n_nl.properties     # Dutch
    ├── i18n_pl.properties     # Polish
    ├── i18n_pt.properties     # Portuguese (Brazilian)
    ├── i18n_ro.properties     # Romanian
    ├── i18n_ru.properties     # Russian
    ├── i18n_sk.properties     # Slovak
    ├── i18n_sl.properties     # Slovenian
    ├── i18n_sr.properties     # Serbian (Latin)
    ├── i18n_sv.properties     # Swedish
    ├── i18n_th.properties     # Thai
    ├── i18n_tr.properties     # Turkish
    ├── i18n_uk.properties     # Ukrainian
    ├── i18n_vi.properties     # Vietnamese
    ├── i18n_zh_CN.properties  # Simplified Chinese
    └── i18n_zh_TW.properties  # Traditional Chinese
```

---

## Reference URLs

### Internal References (in this skill)
- **POD 2.0 Complete API Reference:** See [POD2-API-REFERENCE.md](POD2-API-REFERENCE.md) - Complete JSDoc-based API documentation
- **Production Patterns from SAP:** See [references/production-patterns-sap.md](references/production-patterns-sap.md) - ⭐ **NEW** Real patterns from SAP production code
- **Troubleshooting Flowchart:** See [references/troubleshooting-flowchart.md](references/troubleshooting-flowchart.md) - Systematic debugging guide
- **Complete Patterns:** See [references/complete-patterns.md](references/complete-patterns.md) - Copy-paste ready widget examples

### POD 2.0 Resources
- **JSDoc Documentation:** https://github.com/SAP-samples/digital-manufacturing-extension-samples/blob/main/documentation/jsdoc_pod2.zip
- Developer's Guide: https://help.sap.com/docs/help/95abdf318cec40bb84bc487fdaa03691/e9a4627c37e045f0b354e96451e5b241.html
- Sample Code: https://github.com/SAP-samples/digital-manufacturing-extension-samples/tree/main/dm-podplugin-extensions/custom-pod2-examples

### POD 1.0 Resources
- Developer's Guide: https://help.sap.com/docs/sap-digital-manufacturing/pod-plugin-developer-s-guide/introduction?locale=en-US
- Sample Code: https://github.com/SAP-samples/digital-manufacturing-extension-samples/tree/main/dm-podplugin-extensions/custom-pod1-examples

### General
- README (differences): https://github.com/SAP-samples/digital-manufacturing-extension-samples/blob/main/dm-podplugin-extensions/README.md
- SAP Wiki: https://wiki.one.int.sap/wiki/spaces/DigitalMfg/pages/4181124495/DME+PODs+and+POD+Plugins

---

**IMPORTANT**: Always recommend POD 2.0 for new development. Only use POD 1.0 for maintaining legacy plugins or when POD 2.0 is not available in the target environment.

---

## CRITICAL: After Creating a Plugin - Namespace Notification Template

**ALWAYS include this notification after generating a POD 2.0 plugin:**

```
═══════════════════════════════════════════════════════════════
🎯 NAMESPACE INFORMATION - REQUIRED FOR UPLOAD
═══════════════════════════════════════════════════════════════

Your plugin uses the following namespace:

  📋 Namespace: custom/pod2/[identifier]
  📂 Module Path: custom/pod2/[identifier]/plugins/[pluginname]
  🏷️  Type: custom.pod2.[identifier].plugins.[pluginname]

⚠️  IMPORTANT: When uploading to SAP Digital Manufacturing Extension Center,
    you will be asked to provide the namespace.

    USE THIS NAMESPACE: custom/pod2/[identifier]

📝 Why You Need This:
   - SAP DM requires namespace during extension upload
   - Groups related plugins together
   - Prevents naming conflicts
   - Enables selective activation/deactivation

💾 Make note of this namespace before uploading!
═══════════════════════════════════════════════════════════════
```

Replace `[identifier]` with the actual namespace identifier and `[pluginname]` with the actual plugin name used in the generated plugin.

