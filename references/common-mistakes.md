# Common Mistakes and Fixes

Quick reference guide to POD plugin development pitfalls organized by category.

**Latest Update (2026-05-08)**: Added comprehensive view ID and base class guidance to Mistake #6.

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

## Mistake #16: Using webapp/ Folder Structure ❌ → ✅

## Mistake #15: Using webapp/ Folder Structure ❌ → ✅

**Error**: `"Failed to create custom extension"` or plugin doesn't appear in POD Designer

This is a **FATAL structural mistake** that breaks the plugin upload mechanism.

```
// ❌ WRONG - SAPUI5 application structure
yourplugin/
└── webapp/              # ❌ FATAL: Wrong structure!
    ├── extension.json
    ├── manifest.json    # ❌ Not needed
    ├── Component.js     # ❌ Not needed
    └── namespace/
        └── plugins/
            └── Widget.js
```

**Error Symptoms:**
- Upload fails with "Failed to create custom extension"
- Plugin doesn't appear in POD Designer
- Silent failure during zip upload
- Cryptic error about missing extension.json

### Why It's Wrong

POD 2.0 plugins are **extensions**, not SAPUI5 applications:
- No manifest.json needed
- No Component.js needed
- No webapp/ folder structure
- extension.json must be inside namespace folder at root level

### The Fix ✅

**Official SAP Pattern (from Developer's Guide):**
```
mycompany/               # Namespace folder
├── extension.json       # ✅ Inside namespace folder
├── widget/              # ✅ Widgets folder (SAP recommended)
│   └── MyWidget.js
├── action/              # ✅ Actions folder (SAP recommended)
│   └── MyAction.js
└── util/                # ✅ Utilities folder (SAP recommended)
    └── Helper.js
```

**Module Path Convention:**
- Namespace folder becomes namespace prefix (e.g., `mycompany`, `acme`)
- Use `widget/`, `action/`, `util/` subfolders (official SAP recommendation)
- Module path format: `namespacefolder/subfolder/ClassName`
- Example: `mycompany/widget/MyWidget`

**Alternative Simple Structure (single widget):**
```
simpleplugin/
├── extension.json       # ✅ At root of namespace folder
└── MyWidget.js          # ✅ Widget directly at root
```

**Correct Zip Structure:**
```
mycompany.zip
└── mycompany/           # ← Namespace folder in zip
    ├── extension.json   # ← Inside namespace folder
    ├── widget/
    │   └── MyWidget.js
    ├── action/
    │   └── MyAction.js
    └── util/
        └── Helper.js
```

### How to Fix Existing Plugin

If you already created webapp/ folder:

```bash
# Move everything up one level
mv webapp/* .
rmdir webapp

# Verify structure
ls -la
# Should see: extension.json at root level of namespace folder
```

### Why SAPUI5 Developers Make This Mistake

SAPUI5 applications use:
```
webapp/
├── manifest.json
├── Component.js
└── view/
```

POD plugins are **completely different** - they're dynamically loaded extensions, not standalone apps.

### Prevention

✅ **Before creating a POD plugin, remember:**
- No webapp/ folder
- No manifest.json
- No Component.js
- extension.json goes inside namespace folder
- Use `widget/`, `action/`, `util/` folders (official SAP pattern)
- Namespace folder name = namespace prefix
- Widgets are single files (not view + controller)

### See Also
- [Glossary: File Structure](glossary.md#file-structure)
- [extension.json Structure](../SKILL.md#extensionjson-structure)
- **Official SAP Developer's Guide**: "Set Up Your Project" section
- [Mistake #12: extension.json placement](common-mistakes.md#mistake-12-extensionjson-outside-namespace-folder)

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

## Mistake #21: Using sap.m.Panel Instead of CustomPanel ❌ → ✅ (CRITICAL!)

**Error**: Widget not draggable in POD Designer

**Why It's Wrong**: Regular `sap.m.Panel` doesn't support POD Designer drag-and-drop. Must use `CustomPanel` for Designer compatibility.

### ❌ WRONG - Using sap.m.Panel
```javascript
import Panel from "sap/m/Panel";

_createView() {
    return new Panel({  // ❌ Not Designer-compatible!
        content: [/* controls */]
    });
}
```

### ✅ CORRECT - Using CustomPanel
```javascript
import CustomPanel from "sap/dm/dme/pod2/control/CustomPanel";
import CustomVBox from "sap/dm/dme/pod2/control/CustomVBox";

_createView() {
    return new CustomPanel({
        id: this.getId(),  // CRITICAL: Pass widget ID
        width: "100%",
        height: "100%",
        content: [
            new CustomVBox({
                paddingTop: "Small",
                items: [/* controls */]
            })
        ]
    });
}
```

**When to Use:**
- ✅ Top-level container: CustomPanel
- ✅ Layout containers: CustomVBox
- ✅ Regular controls inside: Use standard sap.m controls

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
| Info message | `toast()` | Temporary, low importance |
| "No items selected" | `toast()` | Temporary guidance |

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

