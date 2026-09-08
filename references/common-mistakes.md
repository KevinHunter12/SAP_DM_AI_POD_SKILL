# Common Mistakes and Fixes

Quick reference guide to POD plugin development pitfalls organized by category.

**Latest Update (2026-09-03)**: Added Mistake #43 - BINDABLE_PROPERTIES does not work for manual getProperties() in Widget subclasses. Added Mistake #44 - getPropertyValue/setPropertyValue must be binding-aware when using BindBooleanPropertyEditor.

---

## Category Index

**🚀 Setup & File Generation (Mistakes #0-#2)**
- [#0: Generic Namespace](#mistake-0-generic-namespace-instead-of-working-directory-name)
- [#1: Creating Namespace Folders](#mistake-1-creating-namespace-folders-during-generation)
- [#2: Wrong extension.json Structure](#mistake-2-invalid-extensionjson-structure)

**💾 Lifecycle & Memory (Mistakes #3-#5)**
- [#3: Missing onExit() Cleanup](#mistake-3-missing-onexit-after-podcontextsubscribe)
- [#4: Not Destroying Dialogs](#mistake-4-not-destroying-dialogs-in-afterclose)
- [#5: Model Initialization Timing](#mistake-5-model-not-initialized-before-createview)

**📦 Imports & Dependencies (Mistakes #6-#9)**
- [#6: View ID Mismatch / Wrong Base Class](#mistake-6-view-id-mismatch-in-_createview) ⭐ CRITICAL
- [#7: No Defensive Type Checking](#mistake-7-no-defensive-type-checking)
- [#8: Wrong PlacementType Import](#mistake-8-wrong-placementtype-import)
- [#9: Wrong ModelPath Constants](#mistake-9-wrong-modelpath-constants)

**🎨 UI & Bindings (Mistakes #10-#13)**
- [#10: Binding in WidgetProperty](#mistake-10-using-binding-syntax-in-widgetproperty-metadata)
- [#11: Missing i18n ResourceBundle](#mistake-11-assuming-getresourcebundle-exists-on-widget)
- [#12: Using "class" Instead of styleClass](#mistake-12-using-class-instead-of-styleclass)
- [#12B: Using setHeight() on Button/Text Controls](#mistake-12b-using-setheight-on-buttontext-controls) ⭐ CRITICAL
- [#13: Expression Binding vs Formatter](#mistake-13-formatter-instead-of-expression-binding)

**⚙️ Configuration (Mistakes #14-#17)**
- [#14: Default Value in PropertyEditor](#mistake-14-passing-default-value-to-stringpropertyeditor)
- [#15: Spreading Parent Properties](#mistake-15-spreading-parent-properties-in-getdefaultconfig)
- [#16: Wrong Property Exclusion Pattern](#mistake-16-using-wrong-property-exclusion-pattern)
- [#17: Missing View ID](#mistake-17-missing-view-id-in-createview)

**🔄 Data & State (Mistakes #18-#22)**
- [#18: Callback Parameter Order](#mistake-18-wrong-callback-parameter-order)
- [#19: No Defensive Type Checking](#mistake-19-no-defensive-type-checking)
- [#20: Multi-Part Binding Null Checks](#mistake-20-multi-part-bindings-without-defensive-checks)
- [#21: Selection Sync](#mistake-21-missing-selection-synchronization-with-podcontext)
- [#22: Multiple PodContext Subscriptions](#mistake-22-subscribing-to-multiple-paths-with-multiple-calls)

**🚨 API & Performance (Mistake #29)** ⭐⭐⭐⭐⭐
- [#29: Fetching Data Already in PodContext](#mistake-29-fetching-data-already-in-podcontext) ⭐ CRITICAL

**⚙️ Property Editors (Mistakes #33, #43, #44)**
- [#33: Wrong SelectPropertyEditor Items Format](#mistake-33-wrong-selectpropertyeditor-items-format) ⭐ CRITICAL
- [#43: BINDABLE_PROPERTIES Doesn't Work for Manual getProperties()](#mistake-43-bindable_properties-doesnt-work-for-manual-getproperties) ⭐ CRITICAL
- [#44: getPropertyValue/setPropertyValue Not Binding-Aware](#mistake-44-getpropertyvalue--setpropertyvalue-not-binding-aware) ⭐ CRITICAL

**📦 Imports (Mistake #34)**
- [#34: Deprecated SAPUI5 Pseudo-Module Imports](#mistake-34-deprecated-sapui5-pseudo-module-imports) ⭐ CRITICAL

**📊 TableWidget Specific (Mistakes #23-#25)**
- [#23: Column/Cell Index Mismatch](#mistake-23-columncell-index-mismatch)
- [#24: GrowingJSONModel Page Increment](#mistake-24-not-incrementing-page-in-growingjsonmodel)
- [#25: CustomPanel Requirement](#mistake-25-using-sapm panel-instead-of-custompanel)

**🚨 Error Handling (Mistakes #26-#28)**
- [#26: Business Errors in Success Response](#mistake-26-not-checking-for-business-errors-in-success-response)
- [#27: CustomFieldData Parsing](#mistake-27-parsing-customfielddata-without-try-catch)
- [#28: Error State Management](#mistake-28-forgetting-to-clear-busy-state-on-error)

---

## 🚀 Setup & File Generation

## Mistake #0: Generic Namespace Instead of Working Directory Name

**Error**: Module path doesn't match actual folder structure, causing "file not found" errors on upload

**This is a critical mistake!** The working directory name IS the namespace. Using generic placeholders like "custom/pod2/something" or "mycompany" instead of detecting the actual folder name causes module resolution failures.

### ❌ WRONG - Using generic namespace
```json
// extension.json - Using generic namespace
{
  "widgets": [{
    "modulePath": "custom/pod2/mywidget/widget/MyWidget",  // ❌ Generic!
    "type": "custom.pod2.mywidget.widget.MyWidget"
  }]
}
```

### ✅ CORRECT - Detect and use actual working directory name

```bash
# ALWAYS detect working directory first!
$ pwd
/home/user/mycompany

$ basename $(pwd)
mycompany  # ← THIS is your namespace!

// extension.json - Using detected namespace
{
  "widgets": [{
    "modulePath": "mycompany/widget/MyWidget",  // ✅ Matches folder!
    "type": "mycompany.widget.MyWidget"
  }]
}
```

### Why This Matters:

1. **Working directory name = namespace** - The folder you're in IS your namespace
2. **Module paths must start with the actual folder name** - SAP DM resolves paths relative to this
3. **SAP DM looks for files relative to the namespace** - Wrong namespace breaks module loading
4. **Wrong namespace = "file not found" errors on upload** - Extension Center can't find your files

### The Rule: Run `basename $(pwd)` FIRST, use result everywhere!

**Mandatory First Step:**
```bash
# Get working directory name - this IS your namespace
NAMESPACE=$(basename $(pwd))
echo "Using namespace: $NAMESPACE"

# Use this in extension.json:
# "modulePath": "$NAMESPACE/widget/MyWidget"
# "type": "$NAMESPACE.widget.MyWidget"
```

**Never use:**
- ❌ Generic placeholders: `custom/pod2/something`, `mycompany`, `acme`
- ❌ Nested paths that don't match working directory
- ❌ Hardcoded namespace values

**Always use:**
- ✅ Actual working directory name detected with `basename $(pwd)`
- ✅ Module paths that start with detected namespace
- ✅ Validation before deployment

### Prevention:
1. Run `basename $(pwd)` BEFORE generating any files
2. Store result in variable: `NAMESPACE=$(basename $(pwd))`
3. Use `$NAMESPACE` in all module paths and type identifiers
4. Validate with: `grep "\"modulePath\": \"$NAMESPACE/" extension.json`

---

## Mistake #1: Missing onExit() After PodContext.subscribe()

**Error**: Memory leak - callback keeps firing after widget destroyed

| Issue | Solution |
|-------|----------|
| Subscribe without unsubscribe | Add matching `unsubscribe()` in `onExit()` |
| Callback fires on destroyed widget | Use same callback reference in both methods |
| Memory grows over time | Always implement cleanup lifecycle |

**Why**: Subscriptions persist after widget destruction, causing memory leaks in long-running POD sessions.

```javascript
// ❌ WRONG
onInit() {
    PodContext.subscribe(ModelPath.WorkInstructions, this._updateText, this);
}
// Missing onExit()!

// ✅ CORRECT
onInit() {
    PodContext.subscribe(ModelPath.WorkInstructions, this._updateText, this);
}
onExit() {
    super.onExit();
    PodContext.unsubscribe(ModelPath.WorkInstructions, this._updateText, this);
}
```

---

## Mistake #1B: Creating Namespace Folders During Generation

**Error**: Files in wrong location (`mycompany/extension.json` instead of root)

**Rule**: Working directory IS the namespace. Never create nested namespace folders.

| Wrong | Correct |
|-------|---------|
| `Write("mycompany/extension.json")` | `Write("extension.json")` |
| `myproject/myproject/extension.json` | `myproject/extension.json` |

**Why**: User is already IN the namespace folder. Creating nested folders breaks module resolution.

---

## Mistake #2: Binding Syntax in WidgetProperty Metadata

**Error**: Type mismatch when using `{i18n>key}` in property definitions

**Rule**: Use `getI18nText()` method, NOT binding syntax `{i18n>key}` in WidgetProperty.

| Wrong | Correct |
|-------|---------|
| `displayName: "{i18n>prop.name}"` | `displayName: this.getI18nText("prop.name")` |
| Binding in metadata definition | Method call for i18n text |

**Why**: WidgetProperty is metadata, not UI5 binding context. Binding syntax causes parser confusion.

---

## Mistake #3: Wrong PodContext Import Path ❌ → ✅

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

## Mistake #4: Passing Default Value to StringPropertyEditor ❌ → ✅

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

## Mistake #5: Wrong Callback Parameter Order ❌ → ✅

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

## Mistake #6: View ID Mismatch in _createView() ❌ → ✅

**Error**: `"CustomWorklist's getView method returned a view with a different ID than configuration"`

This is one of the **most common and confusing errors** in POD 2.0 development. It occurs when:
1. The root control returned from `_createView()` has a different ID than `oConfig.id`
2. Using `LayoutWidget` base class incorrectly
3. Adding suffixes to the root control ID

### Cause 1: Missing or Wrong Root Control ID

```javascript
// ❌ WRONG - No ID passed to root control
_createView() {
    return new VBox({
        items: [new Text({ text: "Hello" })]
    });
}

// ❌ WRONG - ID with suffix
_createView() {
    const oConfig = this.getConfig();
    return new VBox(oConfig.id + "-mainLayout", {  // ❌ Suffix breaks it!
        items: [new Text({ text: "Hello" })]
    });
}

// ✅ CORRECT - Pass oConfig.id EXACTLY as first parameter
_createView() {
    const oConfig = this.getConfig();
    return new VBox(oConfig.id, {  // ✅ Exact ID!
        items: [new Text({ text: "Hello" })]
    });
}
```

### Cause 2: Using LayoutWidget Incorrectly

**CRITICAL**: `LayoutWidget` and `ControlWidget` have special view wrapping logic. For complex custom layouts, **use base `Widget` class instead**.

```javascript
// ❌ WRONG - LayoutWidget with complex custom view
import LayoutWidget from "sap/dm/dme/pod2/widget/LayoutWidget";

class CustomWorklist extends LayoutWidget {
    _createView() {
        const oConfig = this.getConfig();
        // LayoutWidget wraps your view, causing ID mismatch!
        return new FixFlex(oConfig.id, { ... });  // 💥 Will fail!
    }
}

// ✅ CORRECT - Use base Widget class for complex layouts
import Widget from "sap/dm/dme/pod2/widget/Widget";

class CustomWorklist extends Widget {
    _createView() {
        const oConfig = this.getConfig();
        // Widget doesn't wrap - your ID is used directly
        return new Panel(oConfig.id, {
            content: [new FixFlex({ ... })]  // Inner controls can have suffixed IDs
        });
    }
}
```

### When to Use Each Base Class

| Base Class | Use When | Root Control |
|------------|----------|--------------|
| `Widget` | Complex custom layouts, multiple containers | Any control with `oConfig.id` |
| `ControlWidget` | Single SAPUI5 control (Button, Input, etc.) | The control passed to `super()` |
| `LayoutWidget` | Standard layout containers (VBox, HBox, Panel) | The control passed to `super()` |
| `TableWidget` | Data tables with rows/columns | Table (handled by base class) |

### Complete Working Pattern (from Production Code)

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/model/I18nResourceModel",  // ← POD 2.0 i18n model
    "sap/m/Panel",
    "sap/m/VBox"
], (Widget, I18nResourceModel, Panel, VBox) => {
    "use strict";

    // Static i18n model OUTSIDE class (important!)
    let oI18nModel = null;

    class MyComplexWidget extends Widget {

        static getI18nModel() {
            if (!oI18nModel) {
                oI18nModel = new I18nResourceModel({
                    bundleName: "my.namespace.i18n.i18n"
                });
            }
            return oI18nModel;
        }

        static getDisplayName() { return "My Complex Widget"; }
        static getIcon() { return "sap-icon://grid"; }
        static getCategory() { return "Custom"; }  // ✅ Use "Custom" for custom plugins (appears in "Custom" folder in POD Designer)

        // NO spreading of parent properties!
        static getDefaultConfig() {
            return {
                properties: {
                    myProperty: "default"
                }
            };
        }

        _createView() {
            const oConfig = this.getConfig();

            if (!oConfig || !oConfig.id) {
                return new Panel({
                    content: [new sap.m.Text({ text: "Config error" })]
                });
            }

            // Create complex inner layout (can use suffixed IDs)
            const oInnerLayout = new VBox(oConfig.id + "-inner", {
                items: [ /* your controls */ ]
            });

            // Root control MUST have exact oConfig.id
            return new Panel(oConfig.id, {
                width: "100%",
                height: "100%",
                expandable: false,
                expanded: true,
                backgroundDesign: "Transparent",
                content: [oInnerLayout]
            });
        }
    }

    return MyComplexWidget;
});
```

### Key Rules to Remember:

1. **Root control ID** = `oConfig.id` exactly (no suffix!)
2. **Inner controls** can have suffixed IDs: `oConfig.id + "-inner"`
3. **Use base `Widget`** for complex custom layouts
4. **Use `ControlWidget`/`LayoutWidget`** only for simple wrappers
5. **Use `I18nResourceModel`** (from `sap/dm/dme/pod2/model/I18nResourceModel`) with static getter pattern
6. **Never spread** parent class default config properties

---

## Mistake #7: No Defensive Type Checking ❌ → ✅

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

## Mistake #8: Invalid extension.json Structure ❌ → ✅

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

## Mistake #9: Spreading Parent Properties in getDefaultConfig() ❌ → ✅

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

## Mistake #10: Using "class" Instead of "styleClass" ❌ → ✅

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

## Mistake #12B: Using setHeight() on Button/Text Controls ❌ → ✅

**Error**: `TypeError: oButton.setHeight is not a function`

This is a **SAPUI5 framework limitation**: Most non-container controls (Button, Text, Label, Input) do not have a `setHeight()` method.

### ❌ WRONG - Button has no setHeight()

```javascript
import Button from "sap/m/Button";

const oButton = new Button({
    text: "Click Me",
    width: "200px"  // ✅ width works
});

oButton.setHeight("80px");  // 💥 ERROR: setHeight is not a function
```

### ✅ CORRECT - Use inline styles via onAfterRendering

```javascript
import Button from "sap/m/Button";

const oButton = new Button({
    text: "Click Me",
    width: "200px"  // ✅ width has setter
});

// Apply height via DOM manipulation
oButton.addEventDelegate({
    onAfterRendering: () => {
        const oDomRef = oButton.getDomRef();
        if (oDomRef) {
            oDomRef.style.height = "80px";
            oDomRef.style.backgroundColor = "#00FF00";
            oDomRef.style.color = "#FF0000";
        }
    }
});
```

### Which Controls Have setHeight()?

| Control Type | Has setHeight()? | Alternative |
|-------------|------------------|-------------|
| Button, Text, Label, Input, Title | ❌ NO | Inline style only |
| VBox, HBox, Panel, Table | ✅ YES | Use setter method |

### Complete Pattern for Styled Button

```javascript
_createView() {
    const oButton = super._createView();
    
    // ✅ Properties with setters
    oButton.setText(this.getI18nText("button.text"));
    oButton.setType(ButtonType.Emphasized);
    oButton.setEnabled(false);
    oButton.setWidth("200px");  // ✅ width has setter
    
    // ❌ NO setHeight() - use delegate instead
    oButton.addEventDelegate({
        onAfterRendering: () => {
            const oDom = oButton.getDomRef();
            if (oDom) {
                oDom.style.height = "80px";           // Set height
                oDom.style.backgroundColor = "#00FF00"; // Green background
                oDom.style.color = "#FF0000";          // Red text
                oDom.style.border = "2px solid #00CC00";
                oDom.style.fontSize = "18px";
                oDom.style.fontWeight = "bold";
            }
        }
    });
    
    return oButton;
}
```

### Quick Reference

**Controls WITHOUT setHeight():**
- `sap.m.Button`
- `sap.m.Text`
- `sap.m.Label`
- `sap.m.Input`
- `sap.m.Title`
- `sap.m.ComboBox`
- `sap.ui.core.Icon`

**Controls WITH setHeight():**
- `sap.m.VBox`
- `sap.m.HBox`
- `sap.m.Panel`
- `sap.m.Table`
- `sap.ui.table.Table`

**See also**: [sapui5-control-apis.md](sapui5-control-apis.md) for comprehensive SAPUI5 control styling patterns.

---

## Mistake #11: Third-Party Library Loading Fails ❌ → ✅

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

## Mistake #12: Assuming `getResourceBundle()` Exists on Widget ❌ → ✅

**Error**: `TypeError: this.getResourceBundle is not a function`

**CRITICAL**: `getResourceBundle()` is NOT available on Widget base classes in POD 2.0!

Unlike standard SAPUI5 controllers, POD 2.0 widgets don't inherit `getResourceBundle()` from their base class. You must manually load the resource bundle using `sap/base/i18n/ResourceBundle`.

```javascript
// ❌ WRONG - This method doesn't exist!
_getI18nText(sKey) {
    const oResourceBundle = this.getResourceBundle();  // 💥 Method doesn't exist!
    return oResourceBundle.getText(sKey);
}

// ✅ CORRECT - Manual resource bundle loading
// Step 1: Add required imports
sap.ui.define([
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/base/i18n/ResourceBundle",  // ✅ Required for i18n
    // ... other imports
], (ControlWidget, ResourceBundle, ...) => {

    class MyWidget extends ControlWidget {
        #oResourceBundle = null;  // ✅ Store resource bundle

        constructor(oConfig) {
            super(Button, oConfig);
        }

        // Step 2: Load resource bundle in onInit()
        async onInit() {
            await super.onInit();

            // Load i18n resource bundle
            try {
                const sModulePath = sap.ui.require.toUrl("your/namespace/here");
                this.#oResourceBundle = await ResourceBundle.create({
                    url: `${sModulePath}/i18n/i18n.properties`,
                    async: true
                });
            } catch (error) {
                console.error("Failed to load resource bundle:", error);
            }

            // ... rest of onInit
        }

        // Step 3: Use resource bundle with fallbacks
        _getI18nText(sKey) {
            if (this.#oResourceBundle) {
                return this.#oResourceBundle.getText(sKey) || sKey;
            }

            // Fallback to hardcoded defaults if bundle not loaded
            const fallbacks = {
                "button.text": "Default Button Text",
                "message.error": "An error occurred"
            };
            return fallbacks[sKey] || sKey;
        }

        // Step 4: Clean up in onExit()
        onExit() {
            super.onExit();
            this.#oResourceBundle = null;  // ✅ Clean up reference
        }
    }

    return MyWidget;
});
```

**File Structure for i18n:**

```
your-plugin/
├── extension.json
├── widget/
│   └── YourWidget.js
└── i18n/
    ├── i18n.properties       # Default (English)
    ├── i18n_en.properties    # English
    ├── i18n_de.properties    # German
    └── i18n_fr.properties    # French
```

**Key Points:**
- ✅ Always load ResourceBundle manually in onInit()
- ✅ Use `sap.ui.require.toUrl()` to get the correct module path
- ✅ Store bundle in instance variable (`this.#oResourceBundle`)
- ✅ Provide fallback defaults in case bundle fails to load
- ✅ Clean up reference in onExit()
- ❌ Never assume `getResourceBundle()` exists on Widget classes

**Why this happens**: Widget base classes in POD 2.0 don't extend SAPUI5 Controller, so controller convenience methods like `getResourceBundle()` are not available. You must handle resource bundle loading yourself.

**See Also**: [i18n Implementation Pattern](widget-patterns.md#i18n-internationalization-pattern) for complete working examples.

---

## Mistake #12: Wrong PlacementType Import ❌ → ✅

**Error**: `Module "sap/ui/core/library" failed to load`

```javascript
// ❌ WRONG - PlacementType is NOT in sap.ui.core!
import coreLibrary from "sap/ui/core/library";
const { PlacementType } = coreLibrary;

// Usage - causes errors
new Popover({
    placement: PlacementType.Auto  // 💥 PlacementType is undefined!
});

// ✅ CORRECT - Import PlacementType directly from sap.m
import PlacementType from "sap/m/PlacementType";

// Usage - works correctly
new Popover({
    placement: PlacementType.Auto      // ✅ PlacementType.Auto
    // Other values: PlacementType.Bottom, PlacementType.Top, etc.
});
```

**Why this happens**: PlacementType is part of the sap.m library, not sap.ui.core. Many SAPUI5 enums live in the library that defines the controls using them.

**Critical Rule**:
- ✅ Import PlacementType from `"sap/m/PlacementType"`
- ❌ DON'T import from `"sap/ui/core/library"`
- Use directly as enum: `PlacementType.Auto`, `PlacementType.Bottom`, `PlacementType.Top`, etc.

**Other Common Enum Imports:**
```javascript
// Correct imports for commonly used enums
import ButtonType from "sap/m/ButtonType";         // Button types
import ListMode from "sap/m/ListMode";             // List selection modes
import MessageType from "sap/m/MessageType";       // Message types
import ValueState from "sap/ui/core/ValueState";   // Value states (this one IS in core)
```

---

## Mistake #13: Wrong ModelPath Constants ❌ → ✅

**Error**: `Cannot read property 'resource' of undefined` or no subscription triggered

```javascript
// ❌ WRONG - These ModelPath constants DON'T EXIST!
PodContext.subscribe(
    ModelPath.SelectedWorkListItem,  // 💥 Doesn't exist!
    (oItem, sPath) => {
        console.log(oItem.sfc);  // oItem is undefined
    },
    this
);

PodContext.subscribe(
    ModelPath.SelectedSfc,  // 💥 Doesn't exist!
    (sSfc, sPath) => {
        console.log(sSfc);  // Never fires
    },
    this
);

// ✅ CORRECT - Use exact constant names (PLURAL for arrays!)
PodContext.subscribe(
    ModelPath.SelectedWorkListItems,  // ✅ Note: Items (plural)!
    (aItems, sPath) => {
        // aItems is an ARRAY
        const resources = Array.isArray(aItems) ? aItems : [];
        resources.forEach(oItem => {
            console.log(oItem.sfc);
        });
    },
    this
);

// Get all work list items (also returns array)
PodContext.subscribe(
    ModelPath.WorkListItems,  // ✅ Items (plural)
    (aItems, sPath) => {
        console.log(`Total items: ${aItems.length}`);
    },
    this
);
```

**Why this happens**: 
1. Work list related paths return **arrays** (plural names), not single items
2. There is no `SelectedWorkListItem` (singular) constant
3. ModelPath constant names must match EXACTLY what's defined in the framework

**Critical Rules**:
- ✅ `ModelPath.SelectedWorkListItems` - Returns **array** of selected items
- ✅ `ModelPath.WorkListItems` - Returns **array** of all items
- ✅ `ModelPath.WorkListCount` - Returns **number**
- ✅ `ModelPath.WorkListLoading` - Returns **boolean**
- ❌ `ModelPath.SelectedWorkListItem` - Doesn't exist!
- ❌ `ModelPath.SelectedSfc` - Doesn't exist!

**Correct Usage Pattern:**
```javascript
// Subscribe to selected work list items
PodContext.subscribe(
    ModelPath.SelectedWorkListItems,
    this._onWorkListSelectionChanged,
    this
);

_onWorkListSelectionChanged(aSelectedItems, sPath) {
    // ALWAYS validate - may be undefined, null, or empty array
    const items = Array.isArray(aSelectedItems) ? aSelectedItems : [];
    
    if (items.length === 0) {
        // No selection
        return;
    }
    
    // Process selected items
    items.forEach(oItem => {
        const sSfc = oItem.sfc;
        const sResource = oItem.resource;
        // ... use data
    });
}
```

**How to Find Correct Constants:**
1. Always check [references/pod2-api-reference.md](pod2-api-reference.md#modelpath-constants) for exact constant names
2. Look for "Items" (plural) suffix for arrays
3. Check return type (array vs single object vs primitive)
4. Use TypeScript definitions or JSDoc if available

**Common ModelPath Constants (Correct Names):**
```javascript
// Work List (arrays)
ModelPath.SelectedWorkListItems   // Array of selected items
ModelPath.WorkListItems           // Array of all items
ModelPath.WorkListCount           // Number
ModelPath.WorkListLoading         // Boolean

// Resources (arrays)
ModelPath.FilterResources         // Array of selected resources
ModelPath.CurrentResource         // Single resource object

// Operations
ModelPath.CurrentOperation        // Single operation object
ModelPath.SelectedOperationActivities // Array
```

**Prevention:**
- ✅ Check API reference before using any ModelPath constant
- ✅ Use `Array.isArray()` to validate array responses
- ✅ Test subscriptions to ensure they fire
- ❌ DON'T assume singular/plural naming without verification

---

## Mistake #14: Model Not Initialized Before _createView() ❌ → ✅

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

## Mistake #15: extension.json NOT at Zip Root ❌ → ✅

**Error**: `"Failed to load module"`, `"Missing file"`, or plugin widgets don't appear in POD Designer

This is a **CRITICAL packaging mistake** that breaks module path resolution.

```
// ❌ WRONG - extension.json inside namespace folder wrapper
mycompany.zip
└── mycompany/               # ❌ Wrong! No namespace folder wrapper!
    ├── extension.json       # ❌ NOT at zip root!
    └── widget/
        └── MyWidget.js
```

**Error Symptoms:**
- Upload succeeds but widgets don't load
- "Failed to load module" errors in browser console
- Extension appears in Extension Center but widgets missing from POD Designer
- Module path errors like "mycompany/widget/MyWidget not found"

### Why It's Wrong

**extension.json MUST be at the zip root, NOT inside a namespace folder!**

The namespace in module paths (e.g., `mycompany/widget/MyWidget`) is just a prefix in the path, NOT a folder wrapper in the zip.

If extension.json contains:
```json
{
  "widgets": [{
    "modulePath": "mycompany/widget/MyWidget"
  }]
}
```

And extension.json is inside a `mycompany/` folder in the zip, the Extension Center looks for:
- `mycompany/widget/MyWidget.js` relative to extension.json location
- But since extension.json is already in `mycompany/`, it looks for `mycompany/mycompany/widget/MyWidget.js` ❌

### The Fix ✅

**extension.json must be at ZIP ROOT:**

```
mycompany.zip
├── extension.json           # ✅ CORRECT - at zip root!
├── widget/
│   └── MyWidget.js
├── action/
│   └── MyAction.js
└── util/
    └── Helper.js
```

Now the module path `mycompany/widget/MyWidget` is just a namespace-prefixed path. The files are at zip root: `widget/MyWidget.js`.

### How to Create Correct Zip

**From INSIDE the working directory (namespace folder):**

```bash
# Mac/Linux - Zip the CONTENTS
NAMESPACE=$(basename $(pwd))
zip -r "$NAMESPACE.zip" extension.json widget action i18n util

# Windows PowerShell - Zip the CONTENTS
$NAMESPACE = Split-Path -Leaf (Get-Location)
Compress-Archive -Path extension.json,widget,action,i18n,util -DestinationPath "$NAMESPACE.zip" -Force
```

**❌ DON'T do this:**
```bash
# Wrong - creates namespace folder wrapper in zip
cd ..
zip -r mycompany.zip mycompany/
```

### How to Fix Existing Plugin

If you already created the wrong structure with namespace folder wrapper:

```bash
# Extract and fix
unzip mycompany.zip -d temp
cd temp/mycompany
# Rezip the contents directly (no wrapper)
zip -r ../../mycompany-fixed.zip extension.json widget action i18n util
cd ../..
rm -r temp
```

### Prevention

✅ **Before zipping:**
1. cd to your working directory (namespace folder)
2. Zip the CONTENTS: `zip -r name.zip extension.json widget action i18n util`
3. Verify: `unzip -l name.zip` - first entry should be `extension.json` (NOT `name/extension.json`)

### See Also
- [Glossary: File Structure](glossary.md#file-structure)
- [SKILL.md: Deployment Package Creation](../SKILL.md#creating-deployment-package)
- [Mistake #13: webapp/ folder](common-mistakes.md#mistake-13-using-webapp-folder-structure)

---

## Mistake #14: Missing onExit() Unsubscribe - Memory Leak! ❌

### The Problem

PodContext subscriptions must be cleaned up in `onExit()`. Failure to unsubscribe causes memory leaks.

### ❌ WRONG - Memory leak

```javascript
async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        // Subscribing to PodContext events
        PodContext.subscribe(
            ModelPath.SelectedOperationActivities,
            this._onSelectionChange,
            this
        );
    }
}
// ❌ Missing onExit()! Subscription persists after plugin destroyed
```

**What happens:**
1. User opens plugin → subscription registered
2. User closes plugin → widget destroyed BUT subscription still active
3. Context changes → callback fires on destroyed widget → errors
4. Memory leak accumulates with each open/close cycle

### ✅ CORRECT - Always unsubscribe

```javascript
async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(
            ModelPath.SelectedOperationActivities,
            this._onSelectionChange,
            this
        );
    }
}

onExit() {
    super.onExit();
    if (PodContext.isRunMode()) {
        // MUST unsubscribe from ALL subscriptions
        PodContext.unsubscribe(
            ModelPath.SelectedOperationActivities,
            this._onSelectionChange,
            this
        );
    }
    
    // Clean up private fields
    this.#oLog = null;
    this.#oModel = null;
}
```

### Why This Matters

**Production impact:**
- Memory leaks accumulate as users open/close plugins repeatedly
- Callbacks fire on destroyed widgets → runtime errors
- Production systems degrade over time
- Hard to debug - symptoms appear hours after deployment

**Real scenario:**
```
User workflow: Open plugin → Close → Open → Close (10x per day)
After 1 week: 50 orphaned subscriptions
Result: Browser slowdown, errors, crashes
```

### Complete Cleanup Checklist

When implementing `onExit()`, ensure you:

✅ Unsubscribe from ALL PodContext subscriptions  
✅ Call `super.onExit()` first  
✅ Nullify private fields (`#oLog`, `#oModel`, etc.)  
✅ Clear intervals/timeouts if used  
✅ Destroy any manual event listeners  
✅ Clean up file upload controls  
✅ Abort pending async operations if possible  

### Multiple Subscriptions Example

```javascript
class MyWidget extends ControlWidget {
    #oLog = Logger.getLogger("my.widget");
    #oModel = new JSONModel({});
    #iTimerId = null;

    async onInit() {
        await super.onInit();
        
        if (PodContext.isRunMode()) {
            // Multiple subscriptions
            PodContext.subscribe(
                ModelPath.SelectedOperationActivities,
                this._onOperationsChange,
                this
            );
            
            PodContext.subscribe(
                ModelPath.CurrentResource,
                this._onResourceChange,
                this
            );
            
            // Timer for polling
            this.#iTimerId = setInterval(() => {
                this._pollStatus();
            }, 5000);
        }
    }

    onExit() {
        super.onExit();
        
        if (PodContext.isRunMode()) {
            // Unsubscribe ALL subscriptions
            PodContext.unsubscribe(
                ModelPath.SelectedOperationActivities,
                this._onOperationsChange,
                this
            );
            
            PodContext.unsubscribe(
                ModelPath.CurrentResource,
                this._onResourceChange,
                this
            );
        }
        
        // Clear timer
        if (this.#iTimerId) {
            clearInterval(this.#iTimerId);
            this.#iTimerId = null;
        }
        
        // Clean up private fields
        this.#oLog = null;
        this.#oModel = null;
    }
}
```

### TableWidget Special Case

TableWidget has built-in subscription management for table data, but **custom subscriptions still need manual cleanup**:

```javascript
class MyTableWidget extends TableWidget {
    #oLog = Logger.getLogger("my.table");

    async onInit() {
        await super.onInit();
        
        if (PodContext.isRunMode()) {
            // TableWidget manages its own table subscriptions
            // But custom subscriptions need cleanup!
            PodContext.subscribe(
                ModelPath.CurrentWorkCenter,
                this._onWorkCenterChange,
                this
            );
        }
    }

    onExit() {
        super.onExit(); // ✅ Cleans up TableWidget's internal subscriptions
        
        if (PodContext.isRunMode()) {
            // ✅ Clean up YOUR custom subscriptions
            PodContext.unsubscribe(
                ModelPath.CurrentWorkCenter,
                this._onWorkCenterChange,
                this
            );
        }
        
        this.#oLog = null;
    }
}
```

### Common Patterns

**Pattern 1: Guard with isRunMode()**
```javascript
onExit() {
    super.onExit();
    
    // ✅ BEST PRACTICE: Same guard as onInit()
    if (PodContext.isRunMode()) {
        PodContext.unsubscribe(/* ... */);
    }
    
    // Always clean up fields (no guard needed)
    this.#oLog = null;
    this.#oModel = null;
}
```

**Pattern 2: Defensive unsubscribe**
```javascript
onExit() {
    super.onExit();
    
    // ✅ Unsubscribe even if not sure it was subscribed
    // (PodContext.unsubscribe() is safe to call multiple times)
    if (PodContext.isRunMode()) {
        PodContext.unsubscribe(
            ModelPath.SelectedOperationActivities,
            this._onSelectionChange,
            this
        );
    }
}
```

### How to Verify

**1. Check every subscribe has matching unsubscribe**
```bash
# Quick check in your widget file
grep -n "PodContext.subscribe" MyWidget.js
grep -n "PodContext.unsubscribe" MyWidget.js
# Should have same count!
```

**2. Ensure onExit() exists**
```javascript
// Every widget with subscribe() MUST have onExit()
async onInit() {
    PodContext.subscribe(/* ... */);
}

onExit() { // ← MUST exist!
    super.onExit();
    PodContext.unsubscribe(/* ... */);
}
```

**3. Test memory leaks**
```
1. Open browser dev tools → Memory tab
2. Take heap snapshot
3. Open plugin → Close plugin → Repeat 10x
4. Take another heap snapshot
5. Compare: Check for retained widget instances
```

### Prevention Checklist

Before submitting widget code:

- [ ] Every `PodContext.subscribe()` has matching `unsubscribe()`
- [ ] `onExit()` method exists and calls `super.onExit()`
- [ ] All private fields nullified in `onExit()`
- [ ] Timers/intervals cleared in `onExit()`
- [ ] Same `isRunMode()` guard in both `onInit()` and `onExit()`
- [ ] Tested: Open → Close → Open → Close (no errors)

### See Also

- [Widget Lifecycle](../SKILL.md#widget-lifecycle) - Complete lifecycle documentation
- [PodContext API](pod2-api-reference.md#podcontext) - subscribe/unsubscribe documentation
- [TableWidget Pattern](widget-patterns.md#tablewidget) - Complete TableWidget with onExit()

---

## Navigation

📖 **Back to main skill**: [SKILL.md](../SKILL.md)

**Other references**:
- [Widget Patterns](widget-patterns.md) - Complete code patterns
- [Glossary](glossary.md) - Key terms & definitions

---

## Mistake #15: Not Destroying Dynamic Dialogs/Popovers

**Problem**: Memory leaks from undestroyed dialogs

```javascript
// ❌ WRONG
const oDialog = new Dialog({ /* ... */ });
oDialog.open();  // Never destroyed!

// ✅ CORRECT
const oDialog = new Dialog({
    afterClose: () => { oDialog.destroy(); }
});
oDialog.open();
```

---

## Mistake #16: Generic No-Data Text

**Problem**: "No data" doesn't explain WHY

**Fix**: Update based on PodContext state

```javascript
if (!PodContext.getFilterResources()?.length) {
    oTable.setNoDataText(this.getI18nText("error.noResource"));
}
```

---

## Mistake #17: Formatter Instead of Expression Binding

**Problem**: Unnecessary complexity for simple conditional

```javascript
// ❌ WRONG - Overkill
_formatVisible(s) { return s === 'TEXT'; }
visible: { formatter: this._formatVisible }

// ✅ CORRECT
visible: "{= ${type} === 'TEXT' }"
```

---

## Mistake #18: Not Incrementing Page in GrowingJSONModel ⭐⭐⭐⭐⭐

**Problem**: Using GrowingJSONModel but always fetching page 0

**Fix**: Increment page counter

```javascript
// ❌ WRONG - Always fetches page 0!
async _fetchPostings() {
    const oResponse = await API.get({ page: 0, size: 20 });
    return oResponse;
}

// ✅ CORRECT - Increments page
#iPage = 0;

async _fetchPostings() {
    const iPage = this.#iPage++;  // Increment!
    const oResponse = await API.get({ page: iPage, size: 20 });
    return [oResponse.items, oResponse.totalCount];
}
```

**Why Critical:** Essential for pagination to work - without incrementing, same page loads repeatedly.

**See also:** [widget-patterns.md - GrowingJSONModel](widget-patterns.md#tablewidget-with-growingjsonmodel-pagination-pattern)

---

## Mistake #19: Not Destroying Dialogs in afterClose ⭐⭐⭐⭐⭐

**Problem**: Dialog remains in memory after closing, causing memory leaks

**Fix**: Always destroy in afterClose

```javascript
// ❌ WRONG - Memory leak!
new PodDialog({
    title: "My Dialog",
    afterClose: () => {
        // Dialog still exists in memory!
    }
});

// ✅ CORRECT - Destroys dialog
new PodDialog({
    title: "My Dialog",
    afterClose: () => this.destroy()  // Clean up!
});
```

**Why Critical:** Every dialog instance without `destroy()` leaks memory. In long-running POD sessions, this accumulates.

**See also:** [form-dialog-patterns.md](form-dialog-patterns.md)

---

## Mistake #20: Column/Cell Index Mismatch ⭐⭐⭐⭐

**Problem**: Dynamic columns added at different indices than cells

**Fix**: Use same index for both

```javascript
// ❌ WRONG - Indices don't match!
aColumns.push(customColumn);      // Added at end
aCells.splice(2, 0, customCell);  // Inserted at index 2 - MISMATCH!

// ✅ CORRECT - Same index
const iIdx = aColumns.length - 1;
aColumns.splice(iIdx, 0, customColumn);
aCells.splice(iIdx, 0, customCell);  // Same index!
```

**Why Critical:** Mismatched indices cause cells to appear in wrong columns, breaking table layout.

**See also:** [advanced-patterns.md#14-dynamic-column-creation-pattern](advanced-patterns.md#14-dynamic-column-creation-pattern)

---

## Mistake #21: Not Checking for Business Errors in Success Response ⭐⭐⭐⭐⭐

**Problem**: SAP APIs return HTTP 200 but include error flag in response

**Fix**: Always check `oLineItem.error` even in success response

```javascript
// ❌ WRONG - Misses business errors
try {
    const oResponse = await API.post(payload);
    MessageToast.show("Success!"); // But oResponse.lineItems[0].error might be true!
} catch (oError) {
    MessageToast.show("Error");
}

// ✅ CORRECT - Checks business errors
try {
    const oResponse = await API.post(payload);
    
    // Check for business error in success response
    if (oResponse.lineItems[0].error) {
        MessageHistory.showError(oResponse.lineItems[0].errorMessage);
        return;
    }
    
    MessageToast.show("Success!");
} catch (oError) {
    MessageHistory.showError(oError.message);
}
```

**Why Critical:** SAP DM APIs often return HTTP 200 with `error: true` flag for business validation failures.

**See also:** [advanced-patterns.md#15-error-handling--retry-pattern](advanced-patterns.md#15-error-handling--retry-pattern)

---

## Mistake #22: Parsing CustomFieldData Without Try-Catch ⭐⭐⭐⭐

**Problem**: JSON.parse() throws if data is malformed, crashing the widget

**Fix**: Always wrap in try-catch

```javascript
// ❌ WRONG - Throws if invalid JSON
const aFields = JSON.parse(oItem.customFieldData);
const oField = aFields.find(f => f.id === fieldId);
return oField.value;

// ✅ CORRECT - Safe parsing
try {
    const aFields = JSON.parse(oItem.customFieldData);
    const oField = aFields.find(f => f.id === fieldId);
    return oField?.value || "";
} catch (oError) {
    this.#oLog.error("Failed to parse custom field data", oError);
    return "";
}
```

**Why Critical:** Production data can be corrupted or malformed. Always parse defensively.

**See also:** [advanced-patterns.md#12-custom-field-extensibility](advanced-patterns.md#12-custom-field-extensibility)

---

## Summary: Critical Production Patterns

**Top 5 Most Critical Mistakes:**

1. **#21** - Not checking business errors in success response (causes silent failures)
2. **#19** - Not destroying dialogs (memory leaks)
3. **#18** - Not incrementing page counter (pagination breaks)
4. **#14** - Missing onExit() unsubscribe (memory leaks)
5. **#11** - Models created in onInit() instead of _createView() (binding fails)

**Prevention Checklist:**

✅ Always check `oLineItem.error` even on HTTP 200  
✅ Always `destroy()` dialogs in `afterClose`  
✅ Always increment page counter with GrowingJSONModel  
✅ Always unsubscribe in `onExit()`  
✅ Always create models in `_createView()` before bindings  
✅ Always wrap JSON.parse() in try-catch  
✅ Always use same index for columns and cells  

**See also:**
- [form-dialog-patterns.md](form-dialog-patterns.md) - Complete production patterns
- [advanced-patterns.md](advanced-patterns.md) - 15 enterprise patterns
- [widget-patterns.md](widget-patterns.md) - TableWidget, GrowingJSONModel


---

## Clarification: onExit() Unsubscribe - Best Practice vs Production

### Official Guidance

If you subscribe to PodContext in `onInit()`, you **should** unsubscribe in `onExit()` to prevent memory leaks.

```javascript
async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(
            ModelPath.SelectedWorkListItems,
            this._onSelectionChanged,
            this
        );
    }
}

onExit() {
    super.onExit();
    if (PodContext.isRunMode()) {
        PodContext.unsubscribe(
            ModelPath.SelectedWorkListItems,
            this._onSelectionChanged,
            this
        );
    }
}
```

### Production Code Observation

⚠️ **Note**: Some SAP-provided production widgets (MaterialImageWidget, OrderHeaderTextWidget) omit explicit `onExit()` unsubscribe. This suggests the framework MAY auto-cleanup subscriptions, but this is not officially documented.

**Recommendation**: Include explicit `onExit()` unsubscribe in custom plugins as best practice, even if the framework might handle it automatically.

### When onExit() is NOT Needed

You don't need `onExit()` if:
- ✅ Widget uses bindings instead of subscriptions (multi-part binding pattern)
- ✅ Widget has no cleanup (no event handlers, no timers)
- ✅ Widget only reads PodContext (no subscribe calls)

---

## Clarification: When to Spread Parent Properties in getDefaultConfig()

The rule for spreading parent properties is more nuanced than "never spread."

### Rule 1: NEVER Spread Widget/ControlWidget

```javascript
// ❌ WRONG
class MyWidget extends Widget {
    static getDefaultConfig() {
        return {
            properties: {
                ...super.getDefaultConfig()?.properties,  // ❌ NO!
                myProperty: "value"
            }
        };
    }
}
```

**Why**: Widget base class has reserved SAPUI5 property names (e.g., "type") that cause conflicts.

### Rule 2: Specialized Base Classes - Context Dependent

For specialized base classes (ProgressIndicatorWidget, ImageWidget, ExpandableTextWidget, etc.):

**Option A: Spread parent config** (if you want inherited defaults):
```javascript
class MyExpandableWidget extends ExpandableTextWidget {
    static getDefaultConfig() {
        return {
            properties: {
                ...ExpandableTextWidget.getDefaultConfig().properties,  // ✅ OK
                myProperty: "value"
            }
        };
    }
}
```

**Option B: Override completely** (if you don't need inherited defaults):
```javascript
class MyImageWidget extends ImageWidget {
    static getDefaultConfig() {
        return {
            properties: {
                height: "48px"  // ✅ Also OK
                // Doesn't inherit ImageWidget defaults
            }
        };
    }
}
```

### Rule 3: ALWAYS Spread TableWidget/LayoutWidget

```javascript
class MyTableWidget extends TableWidget {
    static getDefaultConfig() {
        return {
            properties: {
                ...super.getDefaultConfig().properties,  // ✅ REQUIRED!
                myProperty: "value"
            }
        };
    }
}
```

**Why**: TableWidget/LayoutWidget have essential defaults that must be inherited.

### Decision Guide

| Base Class | Spread? | Reason |
|------------|---------|--------|
| Widget | ❌ NEVER | Has reserved SAPUI5 properties |
| ControlWidget | ❌ NEVER | Inherits Widget's problems |
| TableWidget | ✅ ALWAYS | Essential defaults required |
| LayoutWidget | ✅ ALWAYS | Essential defaults required |
| ProgressIndicatorWidget | ⚠️ OPTIONAL | Depends on whether you need parent defaults |
| ImageWidget | ⚠️ OPTIONAL | Depends on whether you need parent defaults |
| ExpandableTextWidget | ⚠️ OPTIONAL | Depends on whether you need parent defaults |

---

## Mistake #21: Non-CustomPanel Root Control — Widget Not Draggable ❌ → ✅ (CRITICAL!)

**Severity**: BLOCKING in Designer — widget can be added but cannot be moved or resized.

**Symptom** (browser console warning):
```
[WARN] sap.dm.dme.pod2.designer.PreviewPanel: Widget custom.my.widget.MyWidget
returned view with control sap.m.VBox which is not draggable
```

The warning appears for **any** standard SAPUI5 layout returned as the root: `sap.m.VBox`, `sap.m.HBox`, `sap.m.Panel`, `sap.m.FlexBox`, etc.

**Root Cause**: POD Designer's `PreviewPanel` inspects the root control's constructor chain. Only `CustomPanel` (and controls derived from it) register as draggable/resizable. Standard SAPUI5 containers have no hook into the Designer's layout engine.

### ❌ WRONG — Any standard layout as root

```javascript
// All of these produce the "not draggable" warning:
return new VBox(oConfig.id, { ... });      // ❌ sap.m.VBox
return new HBox(oConfig.id, { ... });      // ❌ sap.m.HBox
return new Panel(oConfig.id, { ... });     // ❌ sap.m.Panel
return new FlexBox(oConfig.id, { ... });   // ❌ sap.m.FlexBox
```

### ✅ CORRECT — CustomPanel as root, standard layouts inside

```javascript
import CustomPanel from "sap/dm/dme/pod2/control/CustomPanel";
import VBox from "sap/m/VBox";

_createView() {
    const oConfig = this.getConfig();
    return new CustomPanel(oConfig.id, {   // ✅ root = CustomPanel with oConfig.id
        width: "100%",
        height: "100%",
        content: [
            new VBox({                      // ✅ inner layouts can be standard controls
                width: "100%",
                height: "100%",
                justifyContent: "Center",
                alignItems: "Center",
                items: [/* your controls */]
            })
        ]
    });
}
```

**Rules:**
- ✅ Root control (returned from `_createView()`) = **always `CustomPanel`** with `oConfig.id`
- ✅ Inner layout containers = standard `sap.m.VBox`, `HBox`, etc. are fine
- ✅ Leaf controls = standard `sap.m.*`, `sap.ui.core.*` etc. are fine
- ❌ Never return VBox/HBox/Panel/FlexBox as the root from `_createView()`

**Note**: This applies to widgets using the base `Widget` class. `ControlWidget` and `LayoutWidget` subclasses manage the root control internally — this rule applies when you override `_createView()` and return your own root.

---

## Mistake #22: Using toast() for Persistent Notifications ❌ → ✅

**Error**: Important messages disappear before user sees them

**Why It's Wrong**: `MessageHistory.toast()` is temporary. Use `showSuccess()`/`showError()` for important messages.

### ❌ WRONG - Toast for operation completion
```javascript
async _onExecute() {
    try {
        await ApiClient.execute(oRequest);
        MessageHistory.toast("Operation completed");  // ❌ Disappears!
    } catch (oError) {
        MessageHistory.toast(oError.message);  // ❌ Error disappears!
    }
}
```

### ✅ CORRECT - Persistent messages
```javascript
async _onExecute() {
    // Pre-validation with toast (temporary)
    if (!this._validateInput()) {
        MessageHistory.toast("Please fill required fields");  // ✅ Temporary guidance
        return;
    }
    
    try {
        await ApiClient.execute(oRequest);
        MessageHistory.showSuccess("Operation completed successfully");  // ✅ Persistent
    } catch (oError) {
        MessageHistory.showError(oError.message);  // ✅ Persistent
    }
}
```

### Decision Matrix

| Scenario | Method | Reason |
|----------|--------|--------|
| API success | `showSuccess()` | Persistent audit trail |
| API error | `showError()` | User needs to review/act |
| Validation failure | `showError()` | Needs user action |
| Informational message | `showInfo()` | Persistent info, no action needed |
| Blocking confirmation required | `messageBox()` | Returns Promise — awaitable |
| "No items selected" | `toast()` | Temporary guidance |
| Info message | `toast()` | Temporary, low importance |

### Full MessageHistory API

```javascript
import MessageHistory from "sap/dm/dme/pod2/context/MessageHistory";

MessageHistory.showSuccess(sMsg)    // ✅ persistent success message
MessageHistory.showError(sMsg)      // ✅ persistent error message
MessageHistory.showWarning(sMsg)    // ✅ persistent warning message
MessageHistory.showInfo(sMsg)       // ✅ persistent info message (often overlooked)
MessageHistory.toast(sMsg)          // ✅ temporary toast (auto-dismisses)
MessageHistory.push(oMessage)       // ✅ push to message history popover
MessageHistory.messageBox(oOpts)    // ✅ returns Promise — use for blocking confirmations
MessageHistory.dismissMessage(sId)  // ✅ programmatically dismiss a message
```

---

## Mistake #23: Forgetting to Clear Busy State on Error ❌ → ✅

**Error**: Widget stuck in busy state after error

**Why It's Wrong**: Without `finally`, busy indicator stays visible if operation throws error.

### ❌ WRONG - No finally block
```javascript
async _refresh() {
    const oView = this.getView();
    oView.setBusy(true);
    
    try {
        await ApiClient.getData(oRequest);
        oView.setBusy(false);  // ❌ Never reached if error!
    } catch (oError) {
        MessageHistory.showError(oError.message);
        // ❌ Busy state still true!
    }
}
```

### ✅ CORRECT - Use finally
```javascript
async _refresh() {
    const oView = this.getView();
    oView.setBusyIndicatorDelay(0);
    oView.setBusy(true);
    
    try {
        const oData = await ApiClient.getData(oRequest);
        this._oModel.setData(oData);
    } catch (oError) {
        MessageHistory.showError(oError.message);
    } finally {
        oView.setBusy(false);  // ✅ Always executes!
    }
}
```

---

## Mistake #24: Not Handling Expected API Error Codes ❌ → ✅

**Error**: Showing error messages for expected "no data" scenarios

**Why It's Wrong**: Some error codes represent normal conditions (like "no pending buyoffs"), not errors.

### ❌ WRONG - All errors treated the same
```javascript
try {
    const aLogs = await ApiClient.findBuyoffLogs(oRequest);
    this._oModel.setData(aLogs);
} catch (oError) {
    // ❌ Shows error even for "no data" case
    MessageHistory.showError(oError.message);
}
```

### ✅ CORRECT - Handle specific error codes
```javascript
try {
    const aLogs = await ApiClient.findBuyoffLogs(oRequest);
    this._oModel.setData(aLogs);
} catch (oError) {
    // Expected case: no data available
    if (oError?.body?.error?.code === "sfc.notInCompletePending") {
        this.#oLog.info("No buyoff logs found");  // Info, not error
        this._oModel.setData([]);
        return;  // Don't show error to user
    }
    
    // Unexpected errors
    this.#oLog.error("Failed to fetch buyoff logs", oError);
    MessageHistory.showError(oError.message);
}
```

**Common SAP DM Error Codes:**
- `sfc.notInCompletePending` - No pending data (handle as empty result)
- `sfc.notFound` - SFC doesn't exist (show error)
- `operation.alreadyStarted` - Duplicate operation (show info message)

---

## Mistake #25: Subscribing to Multiple Paths with Multiple Calls ❌ → ✅

**Error**: Code duplication and multiple callbacks for related data

**Why It's Wrong**: When multiple ModelPaths affect the same widget state, subscribe to all with one callback.

### ❌ WRONG - Multiple subscribe calls
```javascript
onInit() {
    PodContext.subscribe(
        ModelPath.SelectedWorkListItems,
        this._refresh,
        this
    );
    PodContext.subscribe(
        ModelPath.SelectedOperationActivities,
        this._refresh,
        this
    );
    PodContext.subscribe(
        ModelPath.FilterOperationActivities,
        this._refresh,
        this
    );
}
```

### ✅ CORRECT - Array subscription
```javascript
onInit() {
    // Subscribe to multiple paths with one callback
    PodContext.subscribe([
        ModelPath.SelectedWorkListItems,
        ModelPath.SelectedOperationActivities,
        ModelPath.FilterOperationActivities
    ], this._refresh, this);
}

onExit() {
    // Must unsubscribe with same array
    PodContext.unsubscribe([
        ModelPath.SelectedWorkListItems,
        ModelPath.SelectedOperationActivities,
        ModelPath.FilterOperationActivities
    ], this._refresh, this);
}
```

**Benefits:**
- ✅ Single callback handles all changes
- ✅ Cleaner code, less duplication
- ✅ Production pattern from SAP widgets

---

## Mistake #26: Using Wrong Property Exclusion Pattern

**Error**: Using EXCLUDE_PROPERTIES when you need IGNORE_TABLE_PROPERTIES or vice versa

**Found in**: Production POD 2.0 worklist widgets

### Understanding the Three Patterns

| Pattern | Type | Purpose | Use When |
|---------|------|---------|----------|
| EXCLUDE_PROPERTIES | static | Hide from POD Designer | Blacklist properties from property panel |
| INCLUDE_PROPERTIES | static | Show in POD Designer | Whitelist properties for property panel |
| IGNORE_TABLE_PROPERTIES | instance | Skip in table constructor | Properties used by widget but not sap.m.Table |

### ❌ WRONG - Excluding property that table needs

```javascript
class MyTableWidget extends TableWidget {
    static EXCLUDE_PROPERTIES = [
        "pageSize",  // ❌ Wrong! Widget still uses it internally
        "printButtonVisible"  // ❌ Wrong! Not a table property at all
    ];
}
```

**Problems**:
- If table constructor needs `pageSize`, excluding it breaks functionality
- If property is widget-specific (not table-specific), wrong pattern used

### ✅ CORRECT - Use right pattern for each property

```javascript
class OrderListTableWidget extends TableWidget {
    // Exclude from POD Designer property panel (user can't configure)
    static EXCLUDE_PROPERTIES = [
        ...TableWidget.EXCLUDE_PROPERTIES,
        "growing",           // ✅ Table property, hide from designer
        "growingThreshold"   // ✅ Table property, hide from designer
    ];
    
    // Don't pass to sap.m.Table constructor (widget-specific config)
    IGNORE_TABLE_PROPERTIES = [
        "pageSize",          // ✅ Widget uses it, but not for table constructor
        "printButtonVisible",  // ✅ Widget property, not table property
        "printConfigOrder",  // ✅ Custom config, not table property
        "printConfigLabel"   // ✅ Custom config, not table property
    ];
}
```

### Decision Tree

```
Is the property a standard sap.m.Table property?
├─ YES: Is it for user configuration?
│  ├─ YES: Don't exclude it
│  └─ NO: Use EXCLUDE_PROPERTIES (hide from designer)
└─ NO: Is it widget-specific configuration?
   └─ YES: Use IGNORE_TABLE_PROPERTIES (skip in constructor)
```

### Example Scenarios

**Scenario 1: Page Size**
- Property Purpose: Widget uses for pagination logic
- Not a sap.m.Table constructor property
- Solution: `IGNORE_TABLE_PROPERTIES`

**Scenario 2: Growing Table Settings**
- Property Purpose: sap.m.Table constructor properties
- Don't want user to configure (programmatic control)
- Solution: `EXCLUDE_PROPERTIES`

**Scenario 3: Print Button Config**
- Property Purpose: Widget toolbar configuration
- Has nothing to do with table
- Solution: `IGNORE_TABLE_PROPERTIES`

### ✅ CORRECT - Complete Example

```javascript
class ProductionTableWidget extends TableWidget {
    // Whitelist approach (tight control)
    static INCLUDE_PROPERTIES = [
        "alternateRowColors",
        "backgroundDesign",
        "inset",
        "visible",
        "width"
    ];
    
    // Widget-specific properties (not for table constructor)
    IGNORE_TABLE_PROPERTIES = [
        "pageSize",
        "refreshInterval",
        "showToolbar",
        "customActions"
    ];
}
```

### Prevention:
1. Understand property purpose
2. Check if property belongs to sap.m.Table API
3. Use EXCLUDE/INCLUDE for designer control
4. Use IGNORE_TABLE_PROPERTIES for widget-specific config
5. Never mix approaches without understanding

---

## Mistake #27: Missing Selection Synchronization with PodContext

**Error**: Table selections not synced with PodContext selections

**Found in**: Production POD 2.0 worklist table widgets

### ❌ WRONG - No bidirectional sync

```javascript
class MyTableWidget extends TableWidget {
    async onInit() {
        // Only subscribe, but don't sync table to PodContext
        PodContext.subscribe(ModelPath.SelectedWorkListItems, () => {
            // Missing: _syncSelectionsWithPodContext()
        }, this);
    }
    
    _onSelectionChange(oEvent) {
        // Only update PodContext, no sync logic
        const aSelected = this.getTable().getSelectedContexts()
            .map(ctx => ctx.getObject());
        PodContext.setSelectedWorkListItems(aSelected);  // ❌ Loses existing selections!
    }
}
```

**Problems**:
- External selection changes don't reflect in table
- Multi-select scenarios break
- Selection order not preserved
- User loses selections when clicking

### ✅ CORRECT - Complete bidirectional sync

```javascript
class OrderListTableWidget extends TableWidget {
    async onInit() {
        // Subscribe to external selection changes
        PodContext.subscribe(ModelPath.SelectedWorkListItems, () => {
            this._syncSelectionsWithPodContext();  // ✅ Sync table to match PodContext
        }, this);
        
        // Initial sync
        this._syncSelectionsWithPodContext();  // ✅ Critical!
    }
    
    /**
     * Sync table selections to match PodContext (external → table)
     */
    _syncSelectionsWithPodContext() {
        const oTable = this.getTable();
        const aSelectedWorkListItems = PodContext.getSelectedWorkListItems();
        const aSelectedIdentifiers = Array.isArray(aSelectedWorkListItems) ?
            aSelectedWorkListItems.map((oWorkListItem) => oWorkListItem.getIdentifier()) :
            [];

        oTable.getItems().forEach((oListItem) => {
            const oWorkListItem = oListItem.getBindingContext().getObject();
            oListItem.setSelected(aSelectedIdentifiers.includes(oWorkListItem.getIdentifier()));
        });
    }
    
    /**
     * Handle user selection in table (table → external)
     */
    _onSelectionChange(oEvent) {
        const aCurrentPodContextSelection = PodContext.getSelectedWorkListItems() || [];
        const aCurrentTableSelection = this.getTable().getSelectedContexts()
            .map((oContext) => oContext.getObject());

        const aNewSelection = [];
        const oSelectedIdentifiers = new Set(
            aCurrentTableSelection.map((oItem) => oItem.getIdentifier())
        );
        
        // Preserve existing selections that are still selected
        for (const oWorkListItem of aCurrentPodContextSelection) {
            if (oSelectedIdentifiers.has(oWorkListItem.getIdentifier())) {
                aNewSelection.push(oWorkListItem);
            }
        }

        // Add newly selected items
        if (oEvent.getParameter("selected")) {
            const aModifiedListItems = oEvent.getParameter("listItems")
                .map((oListItem) => oListItem.getBindingContext().getObject());
            aNewSelection.push(...aModifiedListItems);
        }

        PodContext.setSelectedWorkListItems(aNewSelection);
    }
    
    /**
     * Handle item press without losing multi-selection
     */
    _onItemPress(oEvent) {
        const oListItem = oEvent.getParameter("listItem");
        const oPressedWorkListItem = oListItem.getBindingContext().getObject();

        // Keep other selections, move clicked item to end (most recent)
        const aSelectedWorkListItems = PodContext.getSelectedWorkListItems() ?? [];
        const sSelectedIdentifier = oPressedWorkListItem.getIdentifier();
        const aNewSelections = [];

        for (const oSelectedWorkListItem of aSelectedWorkListItems) {
            if (oSelectedWorkListItem.getIdentifier() !== sSelectedIdentifier) {
                aNewSelections.push(oSelectedWorkListItem);
            }
        }
        aNewSelections.push(oPressedWorkListItem);
        PodContext.setSelectedWorkListItems(aNewSelections);
    }
    
    onExit() {
        super.onExit();
        
        // Critical: Unsubscribe
        PodContext.unsubscribe(
            ModelPath.SelectedWorkListItems,
            this._syncSelectionsWithPodContext,
            this
        );
    }
}
```

### Key Points:
1. **Bidirectional**: PodContext → table AND table → PodContext
2. **Initial sync**: Call `_syncSelectionsWithPodContext()` in `onInit()`
3. **Preserve selections**: Don't discard existing selections on change
4. **Item press**: Special handling to keep multi-select
5. **Use Set**: Efficient identifier lookup
6. **Always unsubscribe**: Prevent memory leaks

### Prevention:
1. Always implement `_syncSelectionsWithPodContext()`
2. Subscribe AND call sync in `onInit()`
3. Preserve existing selections in `_onSelectionChange()`
4. Handle `_onItemPress()` separately from selection change
5. Use identifiers (not object references) for comparison
6. Test multi-select scenarios

---

## Mistake #28: Multi-Part Bindings Without Defensive Checks

**Error**: Formatters crash on null/undefined values in multi-part bindings

**Found in**: Production POD 2.0 date range and composite cells

### ❌ WRONG - No null checks

```javascript
_createPlannedDateRangeCell() {
    return new Text({
        text: {
            parts: [
                { path: "orderPlannedStartDate" },
                { path: "orderPlannedCompleteDate" }
            ],
            formatter: (oStartDate, oEndDate) => {
                // ❌ Crashes if dates are null/undefined!
                return `${DateTimeUtils.localeDate(oStartDate)} – ${DateTimeUtils.localeDate(oEndDate)}`;
            }
        }
    });
}
```

**Problems**:
- Crashes when data is incomplete
- Shows "undefined – undefined"
- Poor user experience
- Production runtime errors

### ✅ CORRECT - Always validate parameters

```javascript
_createPlannedDateRangeCell() {
    return new Text({
        text: {
            parts: [
                { path: "orderPlannedStartDate" },
                { path: "orderPlannedCompleteDate" }
            ],
            formatter: (oStartDate, oEndDate) => {
                // ✅ CRITICAL: Defensive null checking
                if (!oStartDate || !oEndDate) {
                    return "";  // Return empty string, not error
                }
                return `${DateTimeUtils.localeDate(oStartDate)} – ${DateTimeUtils.localeDate(oEndDate)}`;
            }
        }
    });
}
```

### More Examples

**Quantity with UOM**:
```javascript
// ❌ WRONG
formatter: (fQty, sUom) => {
    return `${fQty} ${sUom}`;  // ❌ Shows "undefined undefined"
}

// ✅ CORRECT
formatter: (fQty, sUom) => {
    if (fQty == null) return "";  // ✅ Handle null/undefined
    return `${fQty} ${sUom || ""}`.trim();  // ✅ Handle missing UOM
}
```

**Material with Description**:
```javascript
// ❌ WRONG
formatter: (sMaterial, sDesc) => {
    return `${sMaterial} - ${sDesc}`;  // ❌ Shows "undefined - undefined"
}

// ✅ CORRECT
formatter: (sMaterial, sDesc) => {
    if (!sMaterial) return "";  // ✅ No material, no display
    return sMaterial ? `${sMaterial} - ${sDesc || ""}` : "";  // ✅ Handle missing description
}
```

**Status with Error Message**:
```javascript
// ❌ WRONG
formatter: (sStatus, sError) => {
    if (sStatus === "ERROR") {
        return sError;  // ❌ Might be undefined!
    }
    return StatusFormatter.getStatusText(sStatus);
}

// ✅ CORRECT
formatter: (sStatus, sError) => {
    if (sStatus === "ERROR") {
        return sError || this.getI18nText("unknownError");  // ✅ Fallback
    }
    return StatusFormatter.getStatusText(sStatus);
}
```

### Defensive Formatter Checklist

```javascript
formatter: (param1, param2, param3) => {
    // 1. Check for null/undefined
    if (param1 == null) return "";
    
    // 2. Provide fallbacks for optional params
    const sValue2 = param2 || "default";
    
    // 3. Validate before complex operations
    if (!Array.isArray(param3) || param3.length === 0) {
        return "";
    }
    
    // 4. Use optional chaining for objects
    const sName = param1?.name || "";
    
    // 5. Return empty string (not null/undefined)
    return result || "";
}
```

### Rule of Thumb:
- **ALWAYS** validate ALL formatter parameters
- Return empty string `""` instead of null/undefined
- Use `||` for fallback values
- Use `?.` for optional chaining
- Test with incomplete data

### Prevention:
1. Add null checks at start of every formatter
2. Return empty string for invalid data
3. Use optional chaining for nested properties
4. Provide sensible defaults
5. Test with incomplete/missing data

---

**See Also**:
- [widget-patterns.md](widget-patterns.md) - Complete widget templates
- [advanced-patterns.md](advanced-patterns.md) - Complex patterns
- [binding-patterns.md](binding-patterns.md) - Data binding techniques
- [form-dialog-patterns.md](form-dialog-patterns.md) - Form validation patterns

---

## Mistake #29: Fetching Data Already in PodContext ❌ → ✅

**⭐⭐⭐⭐⭐ CRITICAL - Performance & Unnecessary API Calls**

### The Problem

Making API calls to fetch data that is **already available** in PodContext objects causes:
- Unnecessary network requests
- Slower user experience
- Wasted backend resources
- More complex code

### Common Scenario: Custom Data Display

**❌ WRONG - Fetching data you already have:**

```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";

_onWorkListSelectionChanged(aSelectedItems) {
    const oItem = aSelectedItems[0];
    const sSfc = oItem.sfc;
    
    // ❌ WRONG - Making unnecessary API call!
    this._fetchCustomData(sSfc);
}

async _fetchCustomData(sSfc) {
    // ❌ This data is already in the WorkListItem!
    const oResponse = await fetch(`/sfcs?sfc=${sSfc}`);
    const oData = await oResponse.json();
    
    // oData.customFields === oWorkListItem.customValues (same data!)
    this._displayCustomValues(oData.customFields);
}
```

**✅ CORRECT - Use data directly from PodContext:**

```javascript
// No ApiClient import needed!

_onWorkListSelectionChanged(aSelectedItems) {
    const oItem = aSelectedItems[0];
    
    // ✅ CORRECT - customFields is an OBJECT with key-value pairs!
    if (oItem.customFields && typeof oItem.customFields === "object") {
        Object.keys(oItem.customFields).forEach(sKey => {
            const sValue = oItem.customFields[sKey];
            // Display: sKey = attribute, sValue = value
        });
    }
}
    }
}

_displayCustomValues(aCustomValues) {
    // Process the data directly - no API call needed!
    aCustomValues.forEach(cv => {
        // Display attribute and value
    });
}
```

### What's Already in WorkListItem?

```javascript
const oWorkListItem = PodContext.getLastSelectedWorkListItem();
// Contains:
{
    sfc: "SFC_12345",           // ✅ Already loaded
    material: "MATERIAL1",       // ✅ Already loaded
    order: "SHOP_ORDER_001",     // ✅ Already loaded
    workCenter: "WC-001",        // ✅ Already loaded
    operationActivity: "OP10",   // ✅ Already loaded
    customFields: {              // ✅ Custom data ALREADY HERE! (OBJECT, not array!)
        "SHOP_ORDER.BREWFATHER_FG": "0.9",
        "ITEM.SAPJAPAN": "Demo custom data",
        "ITEM.MATERIAL_CUSTOM": "Test value"
    },
    sfcStatusCode: "402",        // ✅ Already loaded
    sfcQuantity: 10,             // ✅ Already loaded
    // ... and more
}
```

### When to Use PodContext vs API Calls

| Scenario | Solution | Why |
|----------|----------|-----|
| Display custom data for SFC | `oWorkListItem.customFields` | Already in worklist (object with key-value pairs) |
| Display SFC, material, order | `oWorkListItem` properties | Already in worklist |
| Get current operation | `PodContext.getLastSelectedOperationActivity()` | Already loaded |
| Get filtered resources | `PodContext.getFilterResources()` | Already loaded |
| Get filtered work centers | `PodContext.getFilterWorkCenters()` | Already loaded |
| **Start/Complete SFC** | `ApiClient.sfc.sfcStart/Complete()` | **Action required** |
| **Post production quantity** | `ApiClient.execution.sfcComplete()` or `ActivityConfirmationDelegate` | **Action required** |
| **Fetch material master details** | `ApiClient.material.getMaterial()` | **Not in worklist** |
| **Fetch routing/BOM** | API call | **Not in PodContext** |

### Golden Rules

**✅ DO:**
- Check PodContext objects FIRST before making API calls
- Use worklist data for display purposes
- Use OperationActivity/WorkListItem for selection data
- Use getFilter* methods for filter values

**❌ DON'T:**
- Fetch SFC details when you have the worklist item
- Fetch custom data when it's in `customValues`
- Fetch operation data when you have OperationActivity
- Make API calls for data that's already loaded

### Other Common "Already Have It" Scenarios

**Scenario 1: Getting Plant**
```javascript
// ❌ WRONG - No API needed
await ApiClient.plant.getCurrentPlant();

// ✅ CORRECT
const sPlant = PodContext.getPlant();
```

**Scenario 2: Getting User**
```javascript
// ❌ WRONG
await ApiClient.user.getCurrentUser();

// ✅ CORRECT
const sUserId = PodContext.getUserId();
```

**Scenario 3: Getting Resource**
```javascript
// ❌ WRONG
const sResource = oOperationActivity.resource;  // Often null!

// ✅ CORRECT
const aResources = PodContext.getFilterResources();
const sResource = aResources?.[0]?.resource || null;
```

### Exception: ApiClient is NOT a Generic HTTP Client

**❌ WRONG - ApiClient doesn't have these methods:**

```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";

// ❌ These DON'T exist!
await ApiClient.get(sUrl);
await ApiClient.post(sUrl, oPayload);
await ApiClient.request({ method: "GET", url: sUrl });
```

**✅ CORRECT - ApiClient provides pre-built methods:**

```javascript
// ✅ Use specific methods
await ApiClient.sfc.sfcStart(oRequest);
await ApiClient.sfc.sfcComplete(oRequest);
await ApiClient.execution.sfcComplete(oRequest);   // v2 execution path

// ✅ For custom endpoints, use fetch()
const oContext = PodContext.getContext();
const oResponse = await fetch(sUrl, {
    headers: {
        "Authorization": `Bearer ${oContext.token}`
    }
});
```

### Prevention Checklist

Before writing ANY API call code:

- [ ] Is this data already in WorkListItem? (sfc, material, order, customValues, etc.)
- [ ] Is this data already in OperationActivity? (operation, workCenter, stepId)
- [ ] Is this data from PodContext getters? (plant, userId, filter values)
- [ ] Am I performing an ACTION (start, complete, post) or just DISPLAYING data?
- [ ] If displaying → Use PodContext
- [ ] If action or data NOT in PodContext → Use ApiClient or fetch()

### Quick Reference

**Data Sources Priority:**
1. **First**: Check PodContext objects (WorkListItem, OperationActivity, FilterResources)
2. **Second**: Check PodContext getters (getPlant, getUserId, getFilter*)
3. **Last**: Make API call only if data truly not available

**When API Calls ARE Needed:**
- Triggering backend actions (start, complete, post, serialize)
- Fetching master data not in worklist (material master, BOM, routing)
- Custom business logic endpoints
- Reporting/analytics data

### Real-World Example: Custom Data Viewer

**❌ WRONG (48 lines with unnecessary API):**
```javascript
async _fetchCustomDataForSfc(sSfc) {
    try {
        this._updateStatus("Loading...");
        const sPlant = PodContext.getPlant();
        const oContext = PodContext.getContext();
        const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
        const sUrl = `${sBaseUrl}/sfcs?plant=${sPlant}&sfc=${sSfc}`;
        
        const oResponse = await fetch(sUrl, {
            method: "GET",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${oContext.token}`
            }
        });
        
        if (!oResponse.ok) {
            throw new Error(`HTTP ${oResponse.status}`);
        }
        
        const oData = await oResponse.json();
        this._parseCustomValues(oData.customFields);
    } catch (oError) {
        this._updateStatus("Error loading data");
    }
}
```

**✅ CORRECT (5 lines, no API):**
```javascript
_onWorkListSelectionChanged(aSelectedItems) {
    const oItem = aSelectedItems[0];
    if (oItem?.customValues) {
        this._parseCustomValues(oItem.customFields);
    }
}
```

**Result:**
- **90% less code**
- **Instant response** (no network call)
- **Simpler** (no error handling for network failures)
- **More reliable** (no API dependency)

### Summary

**Before making ANY API call, ask:**
1. "Is this data already in PodContext?"
2. "Am I performing an action or just displaying data?"
3. "Do I really need to fetch this?"

**If displaying data that's in PodContext → DON'T call the API!**

---

## Mistake #30: ES2022 Private Class Fields (`#fieldName`) — JSTokenizer Crash ⭐⭐⭐⭐⭐

**Severity**: BLOCKING — Plugin fails to load entirely.

**Symptom**:
```
JSTokenizer-dbg.js:60 Uncaught (in promise) SyntaxError: Unexpected '#'
```

**Root Cause**: SAPUI5's `JSTokenizer` pre-scans every widget source file as a single text stream before the browser parses it as JavaScript. JSTokenizer predates ES2022 and treats `#` as a syntax error **anywhere** it appears in the file — including inside string literals, comments, and CSS hex colors. ANY `#` character anywhere in your `.js` file will crash plugin loading.

### ❌ WRONG — ES2022 private fields

```javascript
class MyWidget extends Widget {
    static #oI18nModel = new I18nResourceModel({...});  // 💥 JSTokenizer crash!

    static getI18nModel() {
        return this.#oI18nModel;  // 💥 Crash!
    }

    #privateMethod() { ... }  // 💥 Crash!
}
```

### ❌ ALSO WRONG — Hex color codes in embedded CSS strings

```javascript
_getHtml() {
    return '<style>.box { background: #fff; color: #000; }</style>';
    //                                ^^^^         ^^^^
    //                                💥 JSTokenizer crashes on these!
}
```

### ✅ CORRECT — Module-scoped variable instead of private field

```javascript
sap.ui.define([...], function (Widget, I18nResourceModel, ...) {
    "use strict";

    // Module-scoped "private" state — replaces static #field
    var _oI18nModel = new I18nResourceModel({
        bundleName: "namespace.i18n.i18n"
    });

    class MyWidget extends Widget {
        static getI18nModel() {
            return _oI18nModel;  // ✅ Closure access
        }

        _privateMethod() { ... }  // ✅ Underscore convention, not #
    }

    return MyWidget;
});
```

### ✅ CORRECT — `rgb()` instead of hex colors in CSS strings

```javascript
_getHtml() {
    // Use rgb() / rgba() — ZERO '#' characters in the source file
    return '<style>.box { background: rgb(255,255,255); color: rgb(0,0,0); }</style>';
}
```

### ❌ ALSO WRONG — CSS ID selectors in injected style strings

CSS ID selectors start with `#` and will also crash JSTokenizer if the containing JS string line is double-quoted:

```javascript
// 💥 JSTokenizer crash — # inside double-quoted string
el.textContent =
    "#ndm-panel{background:rgb(255,255,255);...}" +
    "#ndm-close{position:absolute;...}";
```

### ❌ DANGEROUS WRONG FIX — Attribute selector with inner double quotes

A common (broken) attempt is to replace `#id` with `[id="id"]`:

```javascript
// 💥 BREAKS JS SYNTAX — unescaped " inside a "-delimited string
el.textContent =
    "[id="ndm-panel"]{background:rgb(255,255,255);...}" +
//        ^        ^
//        These close the outer JS string! Parser sees bare identifier 'ndm'
```

This causes `SyntaxError: Unexpected identifier 'ndm'` — a hard JS parse error, worse than the JSTokenizer crash.

### ✅ CORRECT — Single-quoted JS strings for lines containing `#`

The `#` hazard only triggers when JSTokenizer encounters it outside a valid string context. Inside a **single-quoted** JS string, `#` is safe because the tokenizer treats the entire `'...'` as a string token and does not scan its content for identifiers:

```javascript
// ✅ SAFE — # inside single-quoted strings
el.textContent =
    '#ndm-panel{background:rgb(255,255,255);}' +
    '#ndm-panel:hover{background:rgb(230,230,230);}' +
    '.ndmOtherClass{color:rgb(0,0,0);}';   // double quotes fine for non-# lines
```

**Rule:** Any concatenation segment containing `#` must be a single-quoted string (`'...'`). Segments without `#` can remain double-quoted.

### ⚠️ WATCH OUT — Single-quoted strings that also contain inner single quotes

If a CSS rule with a `#` selector also contains a font name with single quotes (e.g. `'Segoe UI'`, `'Courier New'`), switching the outer delimiter to single quotes will break the string:

```javascript
// 💥 BREAKS JS — inner 'Segoe UI' terminates the outer single-quoted string
'#ndm-screen.empty{font-family:'Segoe UI',Arial;}' +
//                             ^        ^
//                             These close and re-open the JS string!
// Parser sees: '#ndm-screen.empty{font-family:' then bare identifier Segoe
```

**Fix:** Split the concatenation at the point where the inner quote begins — keep the `#selector` portion in a single-quoted segment, move the font-family value to a double-quoted continuation:

```javascript
// ✅ CORRECT — split at the inner-quote boundary
'#ndm-screen.empty{color:rgb(171,171,171);' +
"font-family:'Segoe UI',Arial,sans-serif;" +
"font-size:16px;...}" +
```

**Systematic check before deploying** — finds single-quoted JS string lines that also have inner single quotes (broken):

```python
for i, l in enumerate(open('widget/YourWidget.js').readlines()):
    s = l.strip()
    if s.startswith("'") and '#' in s:
        inner = s[1:].rstrip("' +").rstrip("'")
        if "'" in inner:
            print(f"BROKEN line {i+1}: {s[:100]}")
```

### Rule of Thumb

**CSS ID selectors (`#id`), hex colours (`#fff`), and private fields (`#field`) all contain `#` and must be handled:**

| Contains `#` | Safe approach |
|---|---|
| Private class field `static #x` | Use module-scoped `var _x` instead |
| Hex colour in string `"#ffffff"` | Use `rgb()` in double-quoted string |
| CSS ID selector in string `"#id{}"` | Use single-quoted string `'#id{}'` |
| HTML numeric entity `"&#x2715;"` | Use literal Unicode char or named entity `"&times;"` |

**To verify before deploying**: `grep -c "#" widget/YourWidget.js` should return **0** for double-quoted occurrences. The safe check is:

```bash
# Should print 0 — no double-quoted strings containing #
python3 -c "
import re, sys
content = open('widget/YourWidget.js').read()
bad = [i+1 for i, l in enumerate(content.splitlines())
       if '#' in l and l.strip().startswith('\"') and not l.strip().startswith('//')]
print(len(bad)); [print('  line', n) for n in bad]
"
```

### When You Need Hex Colors (Alternative)

If you prefer to keep hex notation entirely, move CSS into a separate file:
- Move CSS into a separate `.css` file in your extension
- Load it via `jQuery.sap.includeStyleSheet()` in `onInit()`
- The external `.css` file is never seen by JSTokenizer

```javascript
onInit() {
    super.onInit();
    jQuery.sap.includeStyleSheet(
        sap.ui.require.toUrl("namespace/css/styles.css")
    );
}
```

---

## Mistake #31: Using `Widget.extend()` Instead of ES6 `class extends` ⭐⭐⭐⭐⭐

**Severity**: BLOCKING — Plugin fails to load with `Widget.extend is not a function`.

**Symptom**:
```
ModuleError: failed to execute module factory for 'namespace/widget/MyWidget.js':
Widget.extend is not a function
```

**Root Cause**: The POD 2.0 base class `sap/dm/dme/pod2/widget/Widget` is an **ES6 class**, NOT a classic UI5 `ManagedObject`/`Control` subclass. It does NOT expose a static `.extend()` method. You must use ES6 `class extends Widget` syntax.

This is different from classic UI5 controls like `sap.m.Button` or `sap.ui.core.Control`, which DO support `.extend("...", { ... })`.

### ❌ WRONG — Classic UI5 extend pattern

```javascript
// 💥 Widget.extend is not a function
var MyWidget = Widget.extend("namespace.widget.MyWidget", {
    constructor: function (oConfig) {
        Widget.prototype.constructor.call(this, oConfig);
    },
    _createView: function () { ... }
});

MyWidget.getDisplayName = function () { return "..."; };
return MyWidget;
```

### ✅ CORRECT — ES6 class extends (required for POD 2.0)

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/model/I18nResourceModel"
], function (Widget, I18nResourceModel) {
    "use strict";

    var _oI18nModel = new I18nResourceModel({
        bundleName: "namespace.i18n.i18n"
    });

    class MyWidget extends Widget {

        constructor(oConfig) {
            super(oConfig);  // ✅ ES6 super, NOT Widget.prototype.constructor.call
        }

        static getDisplayName() { return "My Widget"; }
        static getIcon() { return "sap-icon://..."; }
        static getCategory() { return "Custom"; }
        static getI18nModel() { return _oI18nModel; }
        static getDefaultConfig() { return { properties: {} }; }

        _createView() {
            // ... create and return root control
        }

        onExit() {
            super.onExit();  // ✅ ES6 super
        }
    }

    return MyWidget;
});
```

### Why This Differs From Classic UI5

| Class type | Pattern | Why |
|---|---|---|
| `sap.ui.core.Control`, `sap.m.Button`, etc. | `Control.extend("...", { ... })` | Classic ManagedObject — has `.extend()` static |
| `sap.dm.dme.pod2.widget.Widget` | `class X extends Widget` | Modern ES6 class — no `.extend()` static |

POD 2.0 was built on a newer codebase that assumes ES6 syntax. Mixing classic UI5 patterns with POD 2.0 base classes will fail.

### Combine With Mistake #30

The CORRECT POD 2.0 widget template:
- ✅ ES6 `class extends Widget` (this mistake)
- ✅ No `#` private fields anywhere — use module-scoped `var _x` (Mistake #30)
- ✅ `super(oConfig)` / `super.onExit()` (not `Widget.prototype.X.call(this)`)

---

## Mistake #32: Curly Braces `{` in Property Values — BindingParser Crash ⭐⭐⭐⭐⭐

**Severity**: BLOCKING — Widget fails to render with cryptic JSTokenizer error.

**Symptom**:
```
JSTokenizer-dbg.js:60 Uncaught (in promise) SyntaxError: Unexpected 'r'
  at resolveEmbeddedBinding (BindingParser-dbg.js:443)
  at extract (BindingInfo-dbg.js:193)
  at ManagedObject constructor
  at _createView ...
```

**Key indicator**: The stack trace shows `BindingParser` → `BindingInfo` → `ManagedObject` constructor. The error character (`r`, `f`, `.`, etc.) varies depending on what comes after the first `{` in your content.

**Root Cause**: When you pass a property value to a UI5 control's constructor like `new HTML(id, { content: sHtml })`, the `ManagedObject` constructor runs **every property value** through `BindingParser` to detect binding expressions like `"{model>path}"` or `"{= ${a} + ${b} }"`. If your value contains a literal `{` character (e.g. from embedded CSS rules like `.box { ... }`, or HTML inline styles like `style="..."` with object-like content), UI5 sees the `{` and tries to parse the next characters as a binding path — choking on whatever non-binding character follows.

This affects ANY property where you embed raw HTML/CSS/JSON-like content:
- `sap.ui.core.HTML` `content`
- `sap.m.Text` `text` (if HTML-like)
- `sap.m.FormattedText` `htmlText`
- Any string property containing `{`

### ❌ WRONG — Constructor property triggers BindingParser

```javascript
var sHtml = '<style>.box { font-family: Arial; }</style><div>...</div>';

// 💥 BindingParser sees '{' in '.box {' and crashes:
// "Unexpected 'f'" (the f from "font-family")
var oHtml = new HTML(oConfig.id + "-html", {
    content: sHtml,           // 💥 PARSED AS BINDING
    sanitizeContent: false
});
```

### ✅ CORRECT — Create empty, then call setter

Setters do NOT run binding parsing. The constructor is the only entry point that does.

```javascript
var sHtml = '<style>.box { font-family: Arial; }</style><div>...</div>';

// Create the control with NO property bag
var oHtml = new HTML(oConfig.id + "-html");

// Use setters — these bypass BindingParser entirely
oHtml.setSanitizeContent(false);
oHtml.setContent(sHtml);  // ✅ Raw value, no binding parsing

return new VBox(oConfig.id, {
    width: "100%",
    height: "100%",
    items: [oHtml]
});
```

### ✅ ALSO CORRECT — Inject via DOM in afterRendering callback

When the HTML contains `@keyframes`, complex CSS selectors, or anything with many `{...}` blocks, the safest approach is to bypass UI5's content pipeline entirely and write directly to the DOM after the control renders:

```javascript
// 1. Pass a safe placeholder to the constructor — no curly braces
var oHtml = new HTML(oConfig.id + "-html", {
    content: "<div></div>",
    afterRendering: function() {
        var oDom = oHtml.getDomRef();
        if (oDom) {
            oDom.innerHTML = buildMyHtml();  // ✅ Pure DOM — no UI5 parsing at all
        }
    }
});

function buildMyHtml() {
    return [
        '<style>',
        '  @keyframes spin { from { transform: rotateY(0deg); } to { transform: rotateY(360deg); } }',
        '  .label { animation: spin 4s linear infinite; }',
        '</style>',
        '<span class="label">Hello</span>'
    ].join('\n');
}
```

Use this approach when:
- HTML contains `@keyframes` with multiple `{ ... }` blocks
- HTML contains many CSS rules with complex selectors
- You need to update the content dynamically after a property change

For dynamic updates, store the HTML control reference and call `getDomRef().innerHTML = ...` directly — do NOT call `setContent()` again, as that re-renders the control and may trigger binding parsing on the new value.

### Alternative: Escape Curly Braces

If you must use the constructor pattern, escape every `{` as `\\{` and `}` as `\\}`:

```javascript
var sHtml = '<style>.box \\{ font-family: Arial; \\}</style>';
// Ugly, error-prone, breaks copy-paste of CSS. Prefer the setter or afterRendering approach.
```

### Quick Diagnostic

If you see `JSTokenizer` errors with a stack trace that includes `BindingParser` or `BindingInfo`:
1. ✅ It's NOT a source-file syntax issue
2. ✅ It IS a property value with `{` characters being parsed as a binding
3. ✅ Fix: move the property assignment from constructor to setter

### Rule of Thumb for Embedded HTML/CSS in POD 2.0 Widgets

**Never pass HTML/CSS strings through control constructor property bags.** Always:
1. Create the control empty: `new HTML(id)`
2. Set HTML/CSS-containing properties via setters: `oHtml.setContent(sHtml)`
3. Other safe properties (numbers, booleans, simple strings without `{`) can still go through the constructor

---

## ⚠️ Embedded HTML/CSS Widget — Combined Gotcha Checklist

When building a POD 2.0 widget that renders raw HTML/CSS via `sap.ui.core.HTML` (e.g. for pixel-perfect mockups, custom visualizations, or third-party HTML embeds), ALL FOUR of these gotchas hit at once:

| Gotcha | Mistake # | Symptom | Fix |
|---|---|---|---|
| `#` anywhere in `.js` file | #30 | `JSTokenizer: Unexpected '#'` | Use `rgb()` not hex; no private fields |
| `Widget.extend()` | #31 | `Widget.extend is not a function` | Use ES6 `class extends Widget` |
| `{` in constructor property | #32 | `JSTokenizer: Unexpected 'r'` (via BindingParser) | Use setters or `afterRendering` DOM injection |
| Non-CustomPanel root | #21 | `"returned view with control sap.m.VBox which is not draggable"` | Return `new CustomPanel(oConfig.id, {...})` as root |

**Pre-deploy validation for HTML-embedding widgets:**

```bash
# 1. Verify no '#' characters in source
grep -c '#' widget/YourWidget.js   # MUST be 0

# 2. Verify ES6 class syntax
grep -n 'class .* extends Widget' widget/YourWidget.js   # MUST find a match
grep -n 'Widget.extend' widget/YourWidget.js             # MUST be empty

# 3. Verify setter pattern for HTML content (OR afterRendering approach)
grep -n 'new HTML.*content:' widget/YourWidget.js        # MUST be empty (no constructor content)
#    OR verify the afterRendering pattern is used:
grep -n 'afterRendering' widget/YourWidget.js            # must find a match if above fails

# 4. Verify CustomPanel as root
grep -n 'CustomPanel' widget/YourWidget.js               # MUST find a match
grep -n 'return new VBox\|return new HBox\|return new Panel' widget/YourWidget.js  # MUST be empty
```

All four checks must pass before deploying a widget that embeds raw HTML/CSS.

---

## Mistake #33: Wrong SelectPropertyEditor Items Format ⭐⭐⭐⭐

**Severity**: VISIBLE — Dropdown shows `[object Object]` or `Element sap.ui.core.Item#__item79` instead of option text.

**Symptoms**:
- Dropdown displays `[object Object]` for each item → you passed `[{ key, text }]` objects
- Dropdown displays `Element sap.ui.core.Item#__item79` for each item → you passed `Item` instances

**Root Cause**: `SelectPropertyEditor` expects `vItems` as a **plain JS object** where keys become option keys and values become display text. It does NOT accept arrays of objects or `sap.ui.core.Item` instances.

### ❌ WRONG — Array of objects (shows `[object Object]`)

```javascript
new SelectPropertyEditor(this, "layout", [
    { key: "layout1", text: "Layout 1" },
    { key: "layout2", text: "Layout 2" }
])
```

### ❌ WRONG — Array of Item instances (shows `Element sap.ui.core.Item#__item79`)

```javascript
new SelectPropertyEditor(this, "layout", [
    new Item({ key: "layout1", text: "Layout 1" }),
    new Item({ key: "layout2", text: "Layout 2" })
])
```

### ✅ CORRECT — Plain `{ key: "display text" }` object

```javascript
new SelectPropertyEditor(this, "layout", {
    layout1: "Layout 1",
    layout2: "Layout 2",
    layout3: "Layout 3"
}, "layout1")

// With i18n text:
new SelectPropertyEditor(this, "layout", {
    layout1: this.getI18nText("layout.option1"),
    layout2: this.getI18nText("layout.option2")
}, "layout1")
```

### ✅ ALSO CORRECT — Array of strings (key === text for each option)

```javascript
new SelectPropertyEditor(this, "size", ["small", "medium", "large"], "medium")
```

### Complete Real-World Example

```javascript
getProperties() {
    return [
        new WidgetProperty({
            displayName: this.getI18nText("property.regionHost"),
            category: "Main",
            propertyEditor: new SelectPropertyEditor(this, "regionHost", {
                "eu10.dmc.cloud.sap":      "eu10.dmc.cloud.sap",
                "eu20.dmc.cloud.sap":      "eu20.dmc.cloud.sap",
                "us10.dmc.cloud.sap":      "us10.dmc.cloud.sap",
                "us20.dmc.cloud.sap":      "us20.dmc.cloud.sap",
                "test.eu10.dmc.cloud.sap": "test.eu10.dmc.cloud.sap",
                "test.eu20.dmc.cloud.sap": "test.eu20.dmc.cloud.sap"
            }, "eu10.dmc.cloud.sap")
        })
    ];
}
```

**Rule**: `vItems` = plain object `{ key: "text" }`. Never an array of objects. Never `Item` instances.

---

## Mistake #34: Deprecated SAPUI5 Pseudo-Module Imports ⭐ CRITICAL

**Severity**: VISIBLE — produces console deprecation warnings on every page load; fails linting in newer UI5 versions.

**Symptoms**:
```
Importing the pseudo module 'sap/m/ButtonType' is deprecated. To access the type
'sap.m.ButtonType', please import 'sap/m/library'.
```

### ❌ WRONG — Importing enum/type constants as direct modules

```javascript
sap.ui.define([
    "sap/m/ButtonType",           // ❌ Deprecated pseudo-module
    "sap/m/FlexAlignItems",       // ❌ Deprecated pseudo-module
    "sap/m/PlacementType",        // ❌ Deprecated pseudo-module
    "sap/ui/core/ValueState"      // ❌ Deprecated pseudo-module
], (ButtonType, FlexAlignItems, PlacementType, ValueState) => {
    // 💥 Deprecation warnings in browser console
});
```

### ✅ CORRECT — Import from parent library, extract as vars

```javascript
sap.ui.define([
    "sap/m/library",              // ✅ Covers ALL sap/m enum types
    "sap/ui/core/library"         // ✅ Covers ALL sap/ui/core enum types
], (mobileLibrary, coreLibrary) => {
    "use strict";

    var ButtonType         = mobileLibrary.ButtonType;
    var FlexAlignItems     = mobileLibrary.FlexAlignItems;
    var FlexJustifyContent = mobileLibrary.FlexJustifyContent;
    var FlexWrap           = mobileLibrary.FlexWrap;
    var PlacementType      = mobileLibrary.PlacementType;
    var ListMode           = mobileLibrary.ListMode;
    var ListSeparators     = mobileLibrary.ListSeparators;
    var MessageType        = mobileLibrary.MessageType;
    var ValueState         = coreLibrary.ValueState;
    var TextAlign          = coreLibrary.TextAlign;
    var TextDirection      = coreLibrary.TextDirection;

    // Use exactly the same as before — no other code changes needed
    new Button({ type: ButtonType.Emphasized });
});
```

### Which paths are affected?

**Rule**: If the import path is `sap/m/<Name>` or `sap/ui/core/<Name>` and it refers to an **enum or constant** (not a control class), it is a deprecated pseudo-module.

| Pseudo-module (❌) | Library (✅) | Extraction |
|--------------------|-------------|------------|
| `"sap/m/ButtonType"` | `"sap/m/library"` | `mobileLibrary.ButtonType` |
| `"sap/m/FlexAlignItems"` | `"sap/m/library"` | `mobileLibrary.FlexAlignItems` |
| `"sap/m/FlexJustifyContent"` | `"sap/m/library"` | `mobileLibrary.FlexJustifyContent` |
| `"sap/m/FlexWrap"` | `"sap/m/library"` | `mobileLibrary.FlexWrap` |
| `"sap/m/PlacementType"` | `"sap/m/library"` | `mobileLibrary.PlacementType` |
| `"sap/m/ListMode"` | `"sap/m/library"` | `mobileLibrary.ListMode` |
| `"sap/m/ListSeparators"` | `"sap/m/library"` | `mobileLibrary.ListSeparators` |
| `"sap/m/MessageType"` | `"sap/m/library"` | `mobileLibrary.MessageType` |
| `"sap/ui/core/ValueState"` | `"sap/ui/core/library"` | `coreLibrary.ValueState` |
| `"sap/ui/core/TextAlign"` | `"sap/ui/core/library"` | `coreLibrary.TextAlign` |
| `"sap/ui/core/TextDirection"` | `"sap/ui/core/library"` | `coreLibrary.TextDirection` |

Control classes (`Button`, `VBox`, `Input`, `Text`, etc.) are **not** affected — keep importing them directly.

**See**: [sapui5-control-apis.md - Deprecated Pseudo-Module Imports](sapui5-control-apis.md#deprecated-pseudo-module-imports)

---

---

## Mistake #43: BINDABLE_PROPERTIES Doesn't Work for Manual getProperties() ⭐ CRITICAL

**Category**: Property Editors  
**Impact**: Property appears non-bindable in POD Designer; no binding expression input shown

### The Mistake

Adding `static BINDABLE_PROPERTIES = ["visible"]` to a `Widget` subclass that uses a **manual** `getProperties()` implementation has no effect.

```javascript
// ❌ WRONG - BINDABLE_PROPERTIES is silently ignored for manual getProperties()
class MyWidget extends Widget {
    static BINDABLE_PROPERTIES = ["visible"];  // Does nothing here!

    getProperties() {
        return [
            new WidgetProperty({
                displayName: "Visible",
                category: PropertyCategory.Main,
                propertyEditor: new BooleanPropertyEditor(this, "visible")  // Still not bindable
            })
        ];
    }
}
```

### Why It Fails

`BINDABLE_PROPERTIES` is only checked inside `Widget._getPropertyEditor()` — the auto-generation path used by `ControlWidget` subclasses that pass a SAPUI5 control class to `super()`. For `Widget` subclasses that return explicit `WidgetProperty` instances from `getProperties()`, the framework never reaches that code path. The `BooleanPropertyEditor` you specified is used directly, with no binding support.

### The Fix

Import `BindBooleanPropertyEditor` (or `BindStringPropertyEditor` for string properties) and use them directly in `getProperties()`:

```javascript
// ✅ CORRECT - use Bind*PropertyEditor directly
sap.ui.define([
    ...
    "sap/dm/dme/pod2/propertyeditor/BooleanPropertyEditor",
    "sap/dm/dme/pod2/propertyeditor/BindBooleanPropertyEditor",  // ← add this
    ...
], (
    ...
    BooleanPropertyEditor,
    BindBooleanPropertyEditor,  // ← add this
    ...
) => {

    class MyWidget extends Widget {
        // No BINDABLE_PROPERTIES needed

        getProperties() {
            return [
                new WidgetProperty({
                    displayName: "Visible",
                    description: "Show or hide the widget panel",
                    category: PropertyCategory.Main,
                    propertyEditor: new BindBooleanPropertyEditor(this, "visible")  // ← bindable!
                }),
                new WidgetProperty({
                    displayName: "Show Table",
                    category: PropertyCategory.Main,
                    propertyEditor: new BooleanPropertyEditor(this, "showTable")  // non-bindable is fine
                })
            ];
        }
    }
});
```

### When to Use Each Editor

| Scenario | Editor to use |
|---|---|
| `Widget` subclass, property should be bindable | `BindBooleanPropertyEditor` / `BindStringPropertyEditor` |
| `Widget` subclass, property is static only | `BooleanPropertyEditor` / `StringPropertyEditor` |
| `ControlWidget` subclass, auto-generated properties | Add name to `static BINDABLE_PROPERTIES = [...]` |

### Import Paths

```javascript
"sap/dm/dme/pod2/propertyeditor/BindBooleanPropertyEditor"
"sap/dm/dme/pod2/propertyeditor/BindStringPropertyEditor"
```

These are the same editors the framework creates internally when `BINDABLE_PROPERTIES` triggers — you're just using them directly.

---

## Mistake #44: getPropertyValue / setPropertyValue Not Binding-Aware ⭐ CRITICAL

**Category**: Property Editors  
**Impact**: Composite binding expressions like `{= ... }` are silently swallowed or cause a runtime error; the "Provide an expression that evaluates to true or false" dialog error appears, OR the binding is accepted but immediately discarded

**Prerequisite**: Only relevant when using `BindBooleanPropertyEditor` (see Mistake #43)

### The Mistake

After correctly switching to `BindBooleanPropertyEditor`, the widget's `getPropertyValue` and `setPropertyValue` overrides still assume the stored value is always a `boolean`. When the user enters a binding expression (a string like `{= %{/model/path} === "X" }`), these overrides destroy it:

```javascript
// ❌ WRONG - coerces the stored binding string to true (string !== false)
getPropertyValue(sName) {
    var vValue = super.getPropertyValue(sName);
    if (sName === "visible") { return vValue !== false; }  // string "{= ...}" → returns true, loses binding!
    return vValue;
}

// ❌ WRONG - passes raw string to setVisible(), SAPUI5 rejects it
setPropertyValue(sName, vValue) {
    if (sName === "visible" && this._oPanel) {
        this._oPanel.setVisible(vValue);  // crashes if vValue is a string
    }
    super.setPropertyValue(sName, vValue);
}
```

### Why It Fails

`BindBooleanPropertyEditor` stores the expression as a **string** in the widget config. Both overrides must check `typeof vValue === "string"` and handle that case separately:
- `getPropertyValue`: pass the string through unchanged so the editor can display it
- `setPropertyValue`: call `bindProperty()` instead of `setVisible()` so SAPUI5 evaluates it against the live model

### The Fix

```javascript
// ✅ CORRECT - pass binding expressions through unchanged
getPropertyValue(sName) {
    var vValue = super.getPropertyValue(sName);
    if (sName === "visible") {
        return typeof vValue === "string" ? vValue : vValue !== false;
    }
    return vValue;
}

// ✅ CORRECT - use bindProperty() for string expressions, setVisible() for booleans
setPropertyValue(sName, vValue) {
    if (sName === "visible" && this._oPanel) {
        if (typeof vValue === "string") {
            this._oPanel.bindProperty("visible", vValue);
        } else {
            this._oPanel.setVisible(vValue !== false);
        }
    }
    super.setPropertyValue(sName, vValue);
}
```

### The General Pattern for Any Bindable Boolean Property

Whenever a property can hold either a `boolean` or a binding expression `string`, apply this pattern to every override that touches it:

```javascript
// getPropertyValue: string → return as-is; boolean → apply default logic
if (typeof vValue === "string") { return vValue; }
return vValue !== false;  // or whatever the boolean default logic is

// setPropertyValue: string → bindProperty(); boolean → direct setter
if (typeof vValue === "string") {
    oControl.bindProperty("propertyName", vValue);
} else {
    oControl.setPropertyName(vValue !== false);
}
```

### Worked Example: Composite Binding Expression

The expression `{= %{/workList/selected/0/sfcStatusCode} === "403" }` is valid but:
- It is a **string**, not a boolean
- `getPropertyValue` with `vValue !== false` returns `true` (a non-empty string is truthy), discarding the expression
- `setVisible(stringValue)` throws because SAPUI5 expects a boolean

With the fix, the string is stored and replayed correctly via `bindProperty()`.
