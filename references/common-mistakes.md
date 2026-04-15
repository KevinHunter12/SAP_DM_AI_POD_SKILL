# Common Mistakes and Fixes

Complete guide to the most common POD plugin development mistakes and their solutions.

---

## Mistake #1: Using Binding Syntax in WidgetProperty Metadata ❌ → ✅

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

## Mistake #2: Wrong PodContext Import Path ❌ → ✅

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

## Mistake #3: Passing Default Value to StringPropertyEditor ❌ → ✅

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

## Mistake #4: Wrong Callback Parameter Order ❌ → ✅

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

## Mistake #5: Missing View ID in _createView() ❌ → ✅

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

## Mistake #6: No Defensive Type Checking ❌ → ✅

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

## Mistake #7: Invalid extension.json Structure ❌ → ✅

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

## Mistake #8: Spreading Parent Properties in getDefaultConfig() ❌ → ✅

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

## Mistake #9: Using "class" Instead of "styleClass" ❌ → ✅

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

## Mistake #10: Third-Party Library Loading Fails ❌ → ✅

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

## Mistake #11: Model Not Initialized Before _createView() ❌ → ✅

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

## Mistake #12: extension.json Outside Namespace Folder ❌ → ✅

**Error**: `"Failed to load module"`, `"Missing file"`, or plugin widgets don't appear in POD Designer

This is a **CRITICAL packaging mistake** that breaks module path resolution.

```
// ❌ WRONG - extension.json at zip root
mycompany.zip
├── extension.json           # ❌ Outside namespace folder!
└── mycompany/
    └── widget/
        └── MyWidget.js
```

**Error Symptoms:**
- Upload succeeds but widgets don't load
- "Failed to load module" errors in browser console
- Extension appears in Extension Center but widgets missing from POD Designer
- Module path errors like "mycompany/widget/MyWidget not found"

### Why It's Wrong

**Module paths in extension.json are relative to extension.json's location!**

If extension.json contains:
```json
{
  "widgets": [{
    "modulePath": "mycompany/widget/MyWidget"
  }]
}
```

And extension.json is at zip root (outside `mycompany/`), the Extension Center looks for:
- `<extension-root>/mycompany/widget/MyWidget.js`

But the file is actually at:
- `<extension-root>/mycompany/mycompany/widget/MyWidget.js` ❌ (path is wrong!)

### The Fix ✅

**extension.json must be INSIDE the namespace folder:**

```
mycompany.zip
└── mycompany/               # ← Namespace folder in zip
    ├── extension.json       # ← Inside namespace folder
    └── widget/
        └── MyWidget.js
```

Now the module path `mycompany/widget/MyWidget` resolves correctly from extension.json's location.

### How to Create Correct Zip

**From PARENT directory of namespace folder:**

```bash
# Mac/Linux
zip -r mycompany.zip mycompany/

# Windows PowerShell
Compress-Archive -Path mycompany -DestinationPath mycompany.zip
```

**❌ DON'T do this:**
```bash
# Wrong - zips contents instead of folder
cd mycompany
zip -r ../mycompany.zip *
```

### How to Fix Existing Plugin

If you already created the wrong structure:

```bash
# Extract and fix
unzip mycompany.zip -d temp
mkdir temp/fixed
mv temp/mycompany temp/fixed/
mv temp/extension.json temp/fixed/mycompany/

# Rezip correctly
cd temp/fixed
zip -r ../../mycompany-fixed.zip mycompany/
cd ../..
rm -r temp
```

### Prevention

✅ **Before zipping:**
1. Verify extension.json is inside namespace folder
2. cd to PARENT directory of namespace folder
3. Zip the namespace folder itself: `zip -r name.zip namespacefolder/`
4. Verify zip contents: first entry should be the namespace folder

### See Also
- [Glossary: File Structure](glossary.md#file-structure)
- [SKILL.md: Deployment Package Creation](../SKILL.md#creating-deployment-package)
- [Mistake #13: webapp/ folder](common-mistakes.md#mistake-13-using-webapp-folder-structure)

---

## Mistake #13: Using webapp/ Folder Structure ❌ → ✅

## Mistake #13: Using webapp/ Folder Structure ❌ → ✅

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

## Navigation

📖 **Back to main skill**: [SKILL.md](../SKILL.md)

**Other references**:
- [Widget Patterns](widget-patterns.md) - Complete code patterns
- [Glossary](glossary.md) - Key terms & definitions
