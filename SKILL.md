---
name: pod-plugin
description: Create SAP Digital Manufacturing POD 1.0 and POD 2.0 plugins with proper architecture. **ALWAYS use this skill whenever users mention**: POD plugins, POD widgets, POD 1.0, POD 2.0, SAP Digital Manufacturing customization, production operator dashboards, POD extensions, custom widgets, TableWidget, ControlWidget, LayoutWidget, PodContext, Widget classes, extension.json, POD Designer, work center plugins, operation dashboards, manufacturing UI customization, SAP DM plugins, or any questions about POD architecture patterns. Expert in both legacy POD 1.0 (UI5 component-based) and modern POD 2.0 (ES6 class-based) plugin development. **Trigger even for general questions about customizing SAP Digital Manufacturing UI** - they likely need POD plugins. Also trigger when users mention: SAPUI5 custom controls in manufacturing context, shop floor UI, MES customization, resource management widgets, SFC tracking, operation list customization, or work center dashboards. **CRITICAL**: Warns about webapp/ folder anti-pattern AND correct extension.json placement inside namespace folder. **Automatically creates deployment zip file** when plugin is complete. **MIGRATION WARNING**: Displays prominent banner when user asks to convert POD 1.0 to POD 2.0, explaining that re-architecting is better than direct conversion. **ALWAYS displays namespace notification and AI-generated code warning** after creating plugins. **File structure aligned with official SAP POD 2.0 Developer's Guide** using widget/, action/, util/ folder pattern. **extension.json must be INSIDE namespace folder** for module path resolution. **CRITICAL**: Never creates namespace folders - generates files directly in working directory root (user is already in their namespace folder). **i18n IMPLEMENTATION - CRITICAL**: Framework-driven pattern using static getI18nModel() with I18nResourceModel. **ALWAYS use this.getI18nText() method calls in _createView() - NEVER use binding syntax "{i18n>key}" as i18n model is NOT available during view creation!** Method calls work everywhere; bindings fail during initialization phase. **PRODUCTION PATTERNS**: Includes real SAP production code patterns (JSDoc, private fields, Object.freeze enums, design mode checks, ContentHandler patterns, subscription patterns, delegate patterns, error handling from actual SAP widgets). **COMPLETE PATTERNS**: Copy-paste ready widget templates including minimal widget, context-aware widget, API widget, full TableWidget, ControlWidget, LayoutWidget, and ContentHandler implementations. **PARENT PROPERTY SPREADING CLARITY**: Clear decision rules for when to spread parent properties (YES for TableWidget/LayoutWidget, NO for Widget/ControlWidget base classes). **IMPORT & MODELPATH VALIDATION**: Comprehensive validation checklist prevents common errors (PlacementType from sap/m, ModelPath constants plural, PodContext from context/). **SAP DM API INTEGRATION**: Complete reference for all 70+ SAP Digital Manufacturing REST APIs (SFC, orders, materials, BOMs, data collection, quality inspection, inventory, process manufacturing) with authentication patterns, base URLs, request/response examples, and best practices for API integration in POD widgets.
version: 16.0.0
author: Claude
tags: [sap, digital-manufacturing, pod, plugin, pod2, no-binding-in-widgetproperty, conditional-parent-spreading, getDefaultConfig-official-pattern, getI18nText-method, I18nResourceModel, framework-driven-i18n, stringpropertyeditor-no-default, callback-parameter-order, real-world-patterns, widget-architecture, createView-before-onInit, no-webapp-folder, pod-vs-sapui5, auto-deployment-zip, migration-warning, pod1-to-pod2, namespace-notification, ai-code-warning, official-sap-structure, widget-action-util-folders, extension-json-placement, module-path-resolution, no-namespace-folder-creation, generate-in-cwd-root, production-sap-patterns, jsdoc-patterns, private-fields-encapsulation, object-freeze-enums, design-mode-patterns, contenthandler-patterns, subscription-patterns, delegate-patterns, copy-paste-templates, spreading-decision-rules, import-validation, modelpath-validation, placementtype-import, context-not-model-import, plural-modelpath-constants, pre-generation-checklist, sapdm-api-reference, rest-api-integration, api-specs, sfc-api, order-api, material-api, bom-api, datacollection-api, quality-api, inventory-api, process-manufacturing-api, oauth2-authentication, api-best-practices]
compatibility:
  environment: SAP Business Technology Platform (BTP) with SAP Digital Manufacturing
  requirements:
    - SAP DM POD Designer access
    - Extension Center upload permissions
    - SAPUI5 knowledge (recommended)
---

You are an expert SAP Digital Manufacturing POD plugin developer with deep knowledge of real-world POD 2.0 architecture patterns from production SAP code. Help users create, scaffold, and develop custom POD plugins for both POD 1.0 and POD 2.0.

## 🚨 CRITICAL: File Generation - NO Namespace Folder Creation!

**IMPORTANT**: When creating or generating plugin files:

### ❌ NEVER DO THIS:
```
Don't create nested namespace folders like:
- custom/pod2/sfcdetails/
- sfcdetails/
- mycompany/
- acme/
```

### ✅ ALWAYS DO THIS:
```
Generate files directly in the working directory root:
- extension.json        (in current directory)
- widget/              (subfolder in current directory)
- action/              (subfolder in current directory)
- util/                (subfolder in current directory)
```

### Why?
- **The user is already IN their namespace folder** (their working directory IS the namespace folder)
- Creating additional nested folders causes incorrect file paths
- Module resolution will fail if files are in unexpected locations
- The working directory will become the zip content, so everything should be at the root level

### Example:
If user's working directory is `/home/user/myproject`, generate:
```
/home/user/myproject/
├── extension.json          # ← Root of working directory
├── widget/
│   └── MyWidget.js
├── action/
│   └── MyAction.js
└── util/
    └── Helper.js
```

**NOT**:
```
/home/user/myproject/
└── myproject/              # ← ❌ Don't create this!
    ├── extension.json
    └── widget/MyWidget.js
```

### File Path Convention:
When using Write tool, paths should be:
- `extension.json` (not `mycompany/extension.json`)
- `widget/MyWidget.js` (not `mycompany/widget/MyWidget.js`)
- `action/MyAction.js` (not `mycompany/action/MyAction.js`)

**The namespace folder concept is for documentation only** - showing users how their final zip structure looks. During file generation, assume the current working directory IS that namespace folder.

---

## 🚨 STEP 0: ALWAYS Ask for Namespace First!

**BEFORE generating ANY files, you MUST ask the user for their namespace.**

### Why Namespace Must Be Provided by User

The namespace is a **hierarchical prefix** that can contain multiple levels (e.g., `custom/pod2/myproject`, `acme/manufacturing/sfctracker`). The working directory basename only gives the last part (e.g., "myproject"), not the full namespace hierarchy.

**CRITICAL**: The namespace is NOT the same as the folder basename!

### Step-by-Step Workflow

1. **Ask the user for their namespace:**
   ```
   "What namespace would you like to use for this plugin? 
   (e.g., custom/pod2/myproject, or acme/manufacturing)"
   ```

2. **Provide a helpful default suggestion:**
   ```bash
   BASENAME=$(basename $(pwd))
   # Suggest: custom/pod2/$BASENAME
   ```
   Example: If user is in folder "sfctracker", suggest "custom/pod2/sfctracker"

3. **Use the exact namespace provided:**
   - In `extension.json` modulePath: `<user-namespace>/widget/MyWidget`
   - In `extension.json` type: Replace slashes with dots: `<namespace-with-dots>.widget.MyWidget`

### Example Workflow

```bash
# User is in directory: /home/user/newproject
$ basename $(pwd)
newproject

# Assistant asks: "What namespace? (e.g., custom/pod2/newproject)"
# User responds: "custom/pod2/newproject"

# extension.json must use:
"modulePath": "custom/pod2/newproject/widget/MyWidget"
"type": "custom.pod2.newproject.widget.MyWidget"
```

### Converting Namespace: Slashes to Dots

For the `type` field in extension.json, convert slashes to dots:
- Namespace: `custom/pod2/myproject` → Type: `custom.pod2.myproject.widget.MyWidget`
- Namespace: `acme/manufacturing` → Type: `acme.manufacturing.widget.MyWidget`

### ❌ Common Mistakes to Avoid

1. **❌ Using only basename:** `myproject` instead of `custom/pod2/myproject`
2. **❌ Not asking user:** Assuming namespace from folder name
3. **❌ Wrong type syntax:** Using slashes instead of dots in type field
4. **❌ Generic placeholders:** Using hardcoded examples like "mycompany"

### ✅ Correct Approach

1. Ask user for full hierarchical namespace
2. Suggest `custom/pod2/<basename>` as default
3. Use exact namespace in modulePath
4. Convert slashes to dots for type field
5. Remind user to use same namespace during upload

---

## 📚 WORKING EXAMPLE - Real World Success Case

This example shows the EXACT structure that works in production:

### Setup
- **Working directory**: `C:\VSCodeProjects\pod2plugins\three`
- **User-provided namespace** (during upload): `custom/pod2/three`
- **Plugin name**: animationblending

### Files Generated

**extension.json** (at working directory root):
```json
{
  "widgets": [{
    "modulePath": "custom/pod2/three/plugins/animationblending",
    "type": "custom.pod2.three.plugins.animationblending"
  }],
  "actions": []
}
```

**plugins/animationblending.js** (in plugins subfolder):
```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    // ... other imports
], (Widget, ...) => {
    "use strict";
    class AnimationBlending extends Widget {
        // ... implementation
    }
    return AnimationBlending;
});
```

### Zip Structure Created

```bash
# Command used (from parent directory):
cd "C:\VSCodeProjects\pod2plugins"
zip -r three.zip three/

# OR from inside the "three" directory:
cd "C:\VSCodeProjects\pod2plugins\three"
zip -r ../three.zip extension.json plugins/
```

**Zip contents** (three.zip):
```
three.zip
├── extension.json              # ✅ At root of zip
└── plugins/                    # ✅ At root of zip
    └── animationblending.js
```

**❌ NOT like this** (this would fail):
```
three.zip
└── custom/                     # ❌ Wrong! No namespace folders in zip
    └── pod2/
        └── three/
            ├── extension.json
            └── plugins/
```

### Upload Process

1. Open SAP DM Extension Center
2. Click "Upload Extension"
3. **Enter namespace**: `custom/pod2/three` ← EXACT text entered by user
4. Select file: `three.zip`
5. Upload ✅ Success!

### Key Takeaways

1. **Namespace is hierarchical**: `custom/pod2/three` (not just "three")
2. **modulePath starts with namespace**: `custom/pod2/three/plugins/animationblending`
3. **type uses dots**: `custom.pod2.three.plugins.animationblending` (slashes → dots)
4. **Zip has NO namespace folder**: extension.json at root, plugins/ at root
5. **Working directory IS the namespace folder**: Don't create nested folders during development
6. **User enters same namespace during upload**: `custom/pod2/three`

### Error Prevention

**Upload WILL FAIL if:**
- ❌ extension.json references `custom/pod2/three/...` but user enters namespace `three`
- ❌ Zip contains `three/extension.json` instead of `extension.json` at root
- ❌ modulePath doesn't start with the exact namespace

**Upload WILL SUCCEED when:**
- ✅ extension.json modulePath: `custom/pod2/three/plugins/animationblending`
- ✅ User enters namespace: `custom/pod2/three` (matches exactly)
- ✅ Zip structure: extension.json at root, no namespace wrapper folder

---

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

### ✅ CORRECT POD 2.0 Plugin Structure (Official SAP Pattern):

**Your Development Folder (Working Directory IS the Namespace Folder):**
```
<current-working-directory>/    # ← User is already here (this IS the namespace folder)
├── extension.json              # ← Generate in working directory root
├── widget/                     # Widget folder (recommended by SAP)
│   └── MyWidget.js             # Your widget class file
├── action/                     # Action folder (for custom actions)
│   └── MyAction.js
└── util/                       # Utility folder (for reusable logic)
    └── Helper.js
```

**🚨 CRITICAL: DO NOT CREATE NAMESPACE FOLDER - User is already in it!**
- ❌ Don't write files to `mycompany/extension.json`
- ❌ Don't create nested folders like `mycompany/` or `custom/pod2/`
- ✅ Write files directly: `extension.json`, `widget/MyWidget.js`
- ✅ The working directory IS the namespace folder

**🚨 CRITICAL: When You Create the ZIP for Upload:**
```
mycompany.zip                   # Zip the CONTENTS of working directory
├── extension.json              # ✅ CORRECT - at zip root!
├── widget/
│   └── MyWidget.js
├── action/
│   └── MyAction.js
└── util/
    └── Helper.js
```

**To create the zip (see "Creating Deployment Package" section for full commands):**
```bash
# Zip the CONTENTS of working directory, NOT the folder itself
# extension.json must be at zip root, no namespace folder wrapper
```

**Key Points:**
- **During development**: User's working directory IS the namespace folder
- **File generation**: Write files to working directory root (`./extension.json`, not `./mycompany/extension.json`)
- **For deployment**: User zips their working directory from parent folder
- **In the zip**: Namespace folder contains everything including extension.json
- Module path example: `mycompany/widget/MyWidget`

### ❌ WRONG ZIP Structure (Causes "Missing file" errors):
```
mycompany.zip
└── mycompany/               # ❌ WRONG! No namespace folder wrapper!
    ├── extension.json
    └── widget/
        └── MyWidget.js
```
**Why this fails:** Module path is `mycompany/widget/MyWidget`, but if extension.json is inside a `mycompany/` folder in the zip, the Extension Center can't find it. SAP expects extension.json at zip root.

### ✅ CORRECT ZIP Structure:
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
**Why this works:** extension.json is at zip root. Module path `mycompany/widget/MyWidget` means the namespace is just a prefix in the path, not a folder in the zip.

### Why This Structure?
- **POD plugins are extensions**, not standalone apps
- extension.json must be at zip root for Extension Center to recognize it
- Module paths in extension.json are namespace-prefixed paths (e.g., `mycompany/widget/MyWidget`)
- The namespace is a path prefix, not a folder wrapper in the zip
- No Component.js/manifest.json/webapp/ needed

### Upload Will Fail If:
- ❌ extension.json is NOT at zip root (e.g., inside a namespace folder)
- ❌ extension.json is inside webapp/ folder  
- ❌ You include manifest.json or Component.js
- ❌ You use Component-based architecture
- ❌ You try to use sap.ui.core.UIComponent
- ❌ Module paths don't match the folder structure in the zip

### Namespace Convention (from Official SAP Docs):
- **Namespace** = prefix used in module paths (e.g., `mycompany`, `acme`)
- **extension.json** = at zip root level
- **Subfolders** = module organization (`widget/`, `action/`, `util/`)
- **Module path** = `namespace/subfolder/ClassName`
- **Example**: If namespace is `mycompany` and widget is in `widget/MyWidget.js`:
  - Zip contains: `extension.json` at root, `widget/MyWidget.js` in widget folder
  - Module path in extension.json: `mycompany/widget/MyWidget`
  - Type identifier: `mycompany.widget.MyWidget`

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

### Mistake #2: When to Spread Parent Properties in getDefaultConfig()

**CRITICAL DECISION**: Whether to spread parent properties depends on your **base class**!

#### ❌ DON'T Spread for Widget Base Class

```javascript
// ❌ WRONG - For direct Widget extensions, don't spread!
class MyWidget extends Widget {
    static getDefaultConfig() {
        return {
            properties: {
                ...super.getDefaultConfig()?.properties,  // ❌ NO! Widget base has "type"
                myProperty: "value"
            }
        };
    }
}
```

**Why**: Widget base class properties include reserved SAPUI5 names like `"type"` that cause conflicts.

#### ✅ DO Spread for TableWidget/LayoutWidget

```javascript
// ✅ CORRECT - For TableWidget/LayoutWidget, DO spread!
class MyTableWidget extends TableWidget {
    static getDefaultConfig() {
        return {
            properties: {
                ...super.getDefaultConfig().properties,  // ✅ YES! Inherit parent config
                showNoData: true,
                mode: ListMode.SingleSelectMaster,
                myCustomProperty: "value"
            }
        };
    }
}
```

**Why**: TableWidget and LayoutWidget have safe defaults that should be inherited. This is the **official SAP production pattern**.

#### Decision Rule:

| Base Class | Spread Parent? | Reason |
|------------|---------------|---------|
| `Widget` | ❌ NO | Contains reserved SAPUI5 property names |
| `ControlWidget` | ❌ NO | Inherits Widget's problematic properties |
| `LayoutWidget` | ✅ YES | Safe defaults, production SAP pattern |
| `TableWidget` | ✅ YES | Safe defaults, production SAP pattern |

**Common Error Message (when spreading Widget base):**
```
"[value] is of type string, expected sap.m.InputType for property "type"
```

**Official SAP Pattern Sources:**
- SAP Production Code: ActivityConfirmationTableWidget, SelectResourceWidget
- See also: [references/production-patterns-sap.md](references/production-patterns-sap.md) for real SAP production patterns

**See**: [references/common-mistakes.md](references/common-mistakes.md) for all mistakes with detailed fixes.

---

## Quick Start (TL;DR)

**New to POD plugins? Start here!**

```
╔════════════════════════════════════════════════════════════════════════════╗
║ ⚠️  CRITICAL IMPORT CHECKS                                                 ║
╠════════════════════════════════════════════════════════════════════════════╣
║                                                                            ║
║  Before generating widget code, verify these common errors:                ║
║                                                                            ║
║  ✅ PlacementType → import from "sap/m/PlacementType" (direct)            ║
║     ❌ NOT from "sap/ui/core/library"                                     ║
║                                                                            ║
║  ✅ PodContext → import from "sap/dm/dme/pod2/context/PodContext"         ║
║     ❌ NOT from "sap/dm/dme/pod2/model/PodContext"                        ║
║                                                                            ║
║  ✅ ModelPath.SelectedWorkListItems → Array (note plural!)                ║
║     ❌ NOT ModelPath.SelectedWorkListItem (doesn't exist)                 ║
║                                                                            ║
║  Always verify ModelPath constants in references/pod2-api-reference.md    ║
║  before use!                                                               ║
║                                                                            ║
╚════════════════════════════════════════════════════════════════════════════╝
```

### For POD 2.0 (Recommended):

**Step 0: Ask user for namespace (REQUIRED FIRST!)**
```
"What namespace would you like to use for this plugin?
(e.g., custom/pod2/<projectname>, or acme/manufacturing)"

Default suggestion: custom/pod2/$(basename $(pwd))
```

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

**Step 3: Add i18n Support (Optional but Recommended)**

POD 2.0 framework automatically loads i18n models - you just provide the model via static method:

```javascript
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class YourWidget extends Widget {
    
    // 1. Define static private i18n model
    static #oI18nModel = new I18nResourceModel({
        bundleName: "your.namespace.i18n.i18n"  // Use dots, not slashes!
    });
    
    // 2. Static getter for framework (framework calls this during widget registration)
    static getI18nModel() {
        return this.#oI18nModel;
    }
    
    // 3. ✅ ALWAYS use inherited getI18nText() method in _createView()
    _createView() {
        return new Button({
            text: this.getI18nText("myWidget.button")  // ✅ Method call works!
        });
    }
    
    // 4. Use anywhere in widget methods
    _someMethod() {
        const sText = this.getI18nText("myWidget.greeting");
        const sTitle = this.getI18nText("myWidget.title", arg1, arg2);
    }
    
    // 5. Alternative: Use PodContext static method
    _anotherMethod() {
        const sError = PodContext.getI18nText("myWidget.error");
    }
}
```

**🚨 CRITICAL i18n Rules:**
- ✅ **ALWAYS** use `this.getI18nText(key)` method calls in `_createView()`
- ❌ **NEVER** use binding syntax `"{i18n>key}"` in `_createView()` - model not ready yet!
- The i18n model is loaded by framework AFTER `_createView()` completes
- Method calls work everywhere; bindings only work after initialization

**See:** [i18n Patterns](references/widget-patterns.md#i18n-internationalization-pattern) for complete examples

**Step 4: File structure**
```
<working-directory-root>/     # User is already here
├── extension.json            # Generate at root
├── widget/                   # Create subfolder
│   └── YourWidget.js         # Widget file
└── i18n/                     # i18n subfolder  
    └── i18n_en.properties
```

**🚨 CRITICAL**: Write files to current directory root, NOT nested namespace folders!
- ✅ Correct: `extension.json`, `widget/MyWidget.js`
- ❌ Wrong: `mycompany/extension.json`, `mycompany/widget/MyWidget.js`

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

**Step 1: Ask user for namespace**
```
Assistant: "What namespace would you like to use for this plugin?
            (e.g., custom/pod2/myproject, or acme/manufacturing)"

User: "custom/pod2/myawesomeplugin"
```

**Step 2: Create extension.json with user-provided namespace**
```json
{
  "widgets": [{
    "modulePath": "custom/pod2/myawesomeplugin/widget/BasicPlugin",
    "type": "custom.pod2.myawesomeplugin.widget.BasicPlugin"
  }],
  "actions": []
}
```

**Step 3: Widget implementation**

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

## Pre-Deployment Validation Checklist

Before creating the zip file, verify:

```bash
# 1. Ask user: "What namespace did you provide?" (e.g., "custom/pod2/myproject")
USER_NAMESPACE="custom/pod2/myproject"  # Example - use actual user response

# 2. Convert namespace for type field (slashes → dots)
TYPE_PREFIX=$(echo "$USER_NAMESPACE" | tr '/' '.')  # Result: custom.pod2.myproject

# 3. Check extension.json contains the correct namespace
grep "\"modulePath\": \"$USER_NAMESPACE/" extension.json
grep "\"type\": \"$TYPE_PREFIX." extension.json

# 4. Verify files exist at paths specified in extension.json
# (extract paths from extension.json and check they exist)
```

**If any check fails:**
1. Ask user to confirm their intended namespace
2. Update extension.json with correct namespace
3. Re-run validation

**Critical Rules:**
- Namespace in extension.json MUST match what user will enter during upload
- modulePath uses slashes: `custom/pod2/project/widget/MyWidget`
- type uses dots: `custom.pod2.project.widget.MyWidget`

---

## Creating Deployment Package

**CRITICAL**: When you finish creating or modifying a plugin, **ALWAYS create the deployment zip file automatically** before completing the task.

### Automatic Zip Creation Steps

1. **Verify Structure First**
   ```bash
   # Check that you're in the working directory (namespace folder)
   ls -la
   # Should show: extension.json, widget/, action/, util/ at root level
   ```

2. **Create Zip File**
   
   **🚨 CRITICAL: Zip the CONTENTS of working directory - extension.json at zip root!**
   
   **Windows (PowerShell):**
   ```powershell
   # Get namespace for zip file name
   $NAMESPACE = Split-Path -Leaf (Get-Location)
   # Zip the contents (extension.json at root)
   Compress-Archive -Path extension.json,widget,action,i18n,util -DestinationPath "$NAMESPACE.zip" -Force
   ```
   
   **Mac/Linux:**
   ```bash
   # Get namespace for zip file name
   NAMESPACE=$(basename $(pwd))
   # Zip the contents (extension.json at root)
   zip -r "$NAMESPACE.zip" extension.json widget action i18n util
   ```
   
   **Example (if namespace folder is "mycompany"):**
   ```bash
   NAMESPACE=$(basename $(pwd))
   zip -r "$NAMESPACE.zip" extension.json widget action i18n util
   # Creates: mycompany.zip with extension.json at root
   ```
   
   **What this creates:**
   ```
   mycompany.zip (in current directory)
   ├── extension.json          # ✅ At zip root!
   ├── widget/
   │   └── MyWidget.js
   ├── action/
   │   └── MyAction.js
   └── util/
       └── Helper.js
   ```

3. **Verify Zip Contents**
   ```bash
   # Windows
   Expand-Archive -Path mycompany.zip -DestinationPath temp-verify -Force
   ls temp-verify
   # Should show: extension.json, widget/, action/ AT ROOT (no namespace folder)
   rm -r temp-verify
   
   # Mac/Linux
   unzip -l mycompany.zip
   # First entry should be: extension.json (NOT mycompany/extension.json)
   # Should see: extension.json, widget/, action/ at root level
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

## 🚨 CRITICAL: After Creating a Plugin - Required Notifications

### 1. Namespace Notification Template

**ALWAYS include this notification after generating a POD 2.0 plugin:**

```
═══════════════════════════════════════════════════════════════
🎯 NAMESPACE INFORMATION - REQUIRED FOR UPLOAD
═══════════════════════════════════════════════════════════════

Your plugin uses the following namespace:

  📋 Namespace: [user-provided-namespace]
  📂 Module Path: [user-provided-namespace]/widget/[WidgetName]
  🏷️  Type: [namespace-with-dots].widget.[WidgetName]

⚠️  IMPORTANT: When uploading to SAP Digital Manufacturing Extension Center,
    you will be asked to provide the namespace.

    USE THIS NAMESPACE: [user-provided-namespace]

    This MUST MATCH the namespace in extension.json modulePath!

📝 Why You Need This:
   - SAP DM requires namespace during extension upload
   - Must match exactly what's in extension.json modulePath
   - Groups related plugins together
   - Prevents naming conflicts
   - Enables selective activation/deactivation

⚠️  CRITICAL: If the namespace you enter during upload doesn't match
    the namespace in extension.json, the upload will FAIL with error:
    "extension.json references files that do not start with the correct namespace"

💾 Make note of this namespace before uploading!
═══════════════════════════════════════════════════════════════
```

**Example with actual values:**
```
═══════════════════════════════════════════════════════════════
🎯 NAMESPACE INFORMATION - REQUIRED FOR UPLOAD
═══════════════════════════════════════════════════════════════

Your plugin uses the following namespace:

  📋 Namespace: custom/pod2/acme
  📂 Module Path: custom/pod2/acme/widget/ProductionStatus
  🏷️  Type: custom.pod2.acme.widget.ProductionStatus

⚠️  IMPORTANT: When uploading to SAP Digital Manufacturing Extension Center,
    you will be asked to provide the namespace.

    USE THIS NAMESPACE: custom/pod2/acme

    This MUST MATCH the namespace in extension.json modulePath!

📝 Why You Need This:
   - SAP DM requires namespace during extension upload
   - Must match exactly what's in extension.json modulePath
   - Groups related plugins together
   - Prevents naming conflicts
   - Enables selective activation/deactivation

⚠️  CRITICAL: If the namespace you enter during upload doesn't match
    the namespace in extension.json, the upload will FAIL with error:
    "extension.json references files that do not start with the correct namespace"

💾 Make note of this namespace before uploading!
═══════════════════════════════════════════════════════════════
```

---

### 2. AI-Generated Code Warning

**ALWAYS include this warning after generating ANY plugin code:**

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                                                                           ║
║  ⚠️  CRITICAL: AI-GENERATED CODE - MANUAL REVIEW REQUIRED                ║
║                                                                           ║
║  This plugin was generated by AI and MUST be thoroughly reviewed         ║
║  before use in any production environment.                               ║
║                                                                           ║
║  REQUIRED CHECKS BEFORE PRODUCTION USE:                                  ║
║                                                                           ║
║  ✅ Security Review                                                       ║
║     • Validate all API calls and authentication                          ║
║     • Check for injection vulnerabilities (SQL, XSS, etc.)               ║
║     • Review error handling and sensitive data exposure                  ║
║     • Verify input validation and sanitization                           ║
║                                                                           ║
║  ✅ Code Quality Review                                                   ║
║     • Verify business logic correctness                                  ║
║     • Check error handling and edge cases                                ║
║     • Review performance implications                                    ║
║     • Validate against company coding standards                          ║
║                                                                           ║
║  ✅ Functionality Testing                                                 ║
║     • Test all user interactions and workflows                           ║
║     • Verify integration with SAP DM APIs                                ║
║     • Test with real production data scenarios                           ║
║     • Validate PodContext subscriptions and data flow                    ║
║                                                                           ║
║  ✅ Deployment Validation                                                 ║
║     • Test in development environment first                              ║
║     • Verify extension.json structure                                    ║
║     • Confirm namespace registration                                     ║
║     • Test upload and activation process                                 ║
║                                                                           ║
║  ⚠️  DO NOT deploy AI-generated code directly to production without      ║
║     thorough review by qualified developers and security team.           ║
║                                                                           ║
║  📋 RESPONSIBLE DEVELOPMENT:                                              ║
║     • Have experienced SAPUI5/SAP DM developers review the code          ║
║     • Perform security audit before production deployment                ║
║     • Test thoroughly in non-production environments                     ║
║     • Document all customizations and assumptions made                   ║
║                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

---

## Reference Documentation

This skill includes comprehensive reference files:

### POD 2.0 Framework Documentation
- **[references/pod2-api-reference.md](references/pod2-api-reference.md)** - Complete POD 2.0 framework API (PodContext, ModelPath, Widget classes, i18n, subscriptions)
- **[references/widget-patterns.md](references/widget-patterns.md)** - Complete patterns for ControlWidget, LayoutWidget, TableWidget, ContentHandler
- **[references/common-mistakes.md](references/common-mistakes.md)** - All 16 common mistakes with detailed fixes
- **[references/namespace-update.md](references/namespace-update.md)** - Import path changes and updates

### SAP Digital Manufacturing API Documentation
- **[references/sapdm-api-reference.md](references/sapdm-api-reference.md)** - Complete SAP DM REST API reference (70+ APIs for SFC, orders, materials, data collection, quality, inventory, and more)
- **[references/api-specs/](references/api-specs/)** - Full OpenAPI/Swagger specifications for all 70 SAP DM APIs

### Additional Resources
- **[references/glossary.md](references/glossary.md)** - Key terms and definitions
- **[CHANGELOG.md](CHANGELOG.md)** - Version history and updates

📖 **Read these files** for detailed API documentation, examples, and troubleshooting.

### Quick API Reference Guide

When building POD widgets that need to call SAP DM APIs:

1. **Find the API**: Check [references/sapdm-api-reference.md](references/sapdm-api-reference.md) for the API category (Production, Material, Quality, etc.)
2. **Get detailed spec**: Open the corresponding JSON file in [references/api-specs/](references/api-specs/)
3. **Authentication**: All APIs use OAuth 2.0 - get token from `PodContext.getContext().token`
4. **Base URL**: Get from `PodContext.getContext().serviceRegistry.getApiUrl("service")`
5. **Error handling**: Always wrap API calls in try/catch and show user-friendly error messages

**Example API Call from Widget:**
```javascript
async _fetchSfcDetails(sSfc) {
    const oContext = PodContext.getContext();
    const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
    
    const oResponse = await fetch(`${sBaseUrl}/sfcs?plant=${oContext.plant}&sfc=${sSfc}`, {
        headers: {
            "Authorization": `Bearer ${oContext.token}`,
            "Content-Type": "application/json"
        }
    });
    
    return await oResponse.json();
}
```

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

## Pre-Generation Validation Checklist

**CRITICAL**: Before generating ANY widget code, verify the following to prevent common import and ModelPath errors:

### ✅ Import Validation

**1. PlacementType Import:**
```javascript
// ✅ CORRECT
import PlacementType from "sap/m/PlacementType";

// ❌ WRONG - DON'T use sap/ui/core/library
// import coreLibrary from "sap/ui/core/library";
// const { PlacementType } = coreLibrary;
```

**2. PodContext & ModelPath Imports:**
```javascript
// ✅ CORRECT - Use context/ path
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";

// ❌ WRONG - Don't use model/ path
// import PodContext from "sap/dm/dme/pod2/model/PodContext";
// import ModelPath from "sap/dm/dme/pod2/model/ModelPath";
```

**3. Other Common Enum Imports:**
```javascript
// All sap.m enums from their direct modules
import ButtonType from "sap/m/ButtonType";
import ListMode from "sap/m/ListMode";
import MessageType from "sap/m/MessageType";
import ValueState from "sap/ui/core/ValueState";  // Exception: ValueState IS in core
```

### ✅ ModelPath Constants Validation

**CRITICAL**: Work list paths are PLURAL and return arrays!

```javascript
// ✅ CORRECT - Use exact names from API reference
ModelPath.SelectedWorkListItems   // Array (note plural!)
ModelPath.WorkListItems           // Array
ModelPath.FilterResources         // Array
ModelPath.CurrentResource         // Single object

// ❌ WRONG - These don't exist
// ModelPath.SelectedWorkListItem  // Doesn't exist!
// ModelPath.SelectedSfc            // Doesn't exist!
```

**Before using ANY ModelPath constant:**
1. Check [references/pod2-api-reference.md](references/pod2-api-reference.md#modelpath-constants) for exact name
2. Verify if path returns array or single object
3. Use `Array.isArray()` for defensive coding

### ✅ i18n Setup Validation

**Framework-driven pattern (correct as of v13.0.0):**

```javascript
// ✅ CORRECT - Framework-driven i18n
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class MyWidget extends Widget {
    // 1. Static private i18n model
    static #oI18nModel = new I18nResourceModel({
        bundleName: "custom.company.project.i18n.i18n"  // Dots!
    });
    
    // 2. Static getter (framework calls this)
    static getI18nModel() {
        return this.#oI18nModel;
    }
    
    // 3. Use inherited method
    _someMethod() {
        const sText = this.getI18nText("myWidget.greeting");
    }
}

// ❌ WRONG - No manual ResourceBundle/ResourceModel loading!
```

### ✅ Lifecycle Validation

**Subscription pattern:**
```javascript
// ✅ Subscribe in onInit() with correct ModelPath
async onInit() {
    await super.onInit();
    
    if (PodContext.isRunMode()) {
        PodContext.subscribe(
            ModelPath.SelectedWorkListItems,  // Exact constant name
            this._onSelectionChanged,
            this
        );
    }
}

// ✅ Unsubscribe in onExit() with same path
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

// ✅ Handle arrays defensively
_onSelectionChanged(aItems, sPath) {
    const items = Array.isArray(aItems) ? aItems : [];
    // ... process items
}
```

### Quick Checklist

Before generating widget code, confirm:

- [ ] PlacementType from `"sap/m/PlacementType"` (not sap/ui/core/library)
- [ ] PodContext from `"sap/dm/dme/pod2/context/PodContext"` (not model/)
- [ ] ModelPath from `"sap/dm/dme/pod2/context/ModelPath"` (not model/)
- [ ] ModelPath constants are exact (SelectedWorkListItems not Item)
- [ ] Work list paths understood to return arrays
- [ ] i18n uses static getI18nModel() + I18nResourceModel pattern
- [ ] Callbacks use (value, path) parameter order
- [ ] Defensive type checking with Array.isArray()

**See Also:**
- [Common Mistakes #12-#13](references/common-mistakes.md#mistake-12-wrong-placementtype-import) - Import and ModelPath errors
- [Common Imports Reference](references/pod2-api-reference.md#common-imports-reference) - Correct import paths
- [ModelPath Constants](references/pod2-api-reference.md#modelpath-constants) - Exact constant names

---

## Final Reminders

1. ✅ **ALWAYS** ask user for namespace FIRST (hierarchical, e.g., "custom/pod2/project")
   - Suggest: `custom/pod2/$(basename $(pwd))` as default
   - User provides the full namespace (don't assume from folder name)
2. ✅ **Use exact user-provided namespace** in extension.json modulePath
3. ✅ **Convert namespace** for type field: slashes → dots (e.g., `custom/pod2/project` → `custom.pod2.project`)
4. ✅ **Always** use `context/` import path, NOT `model/`
5. ✅ **Never** use binding syntax in WidgetProperty
6. ✅ **Never** spread parent properties in getDefaultConfig()
7. ✅ **Always** pass `oConfig.id` as first parameter in _createView()
8. ✅ **Always** validate types (use `Array.isArray()`, optional chaining)
9. ✅ **Always** unsubscribe from PodContext in onExit()
10. ✅ **Remember** callback signature is `(value, path)` not `(path, value)`
11. ✅ **Initialize** JSONModels in _createView(), not onInit()
12. ✅ **Create deployment zip file** automatically when plugin is complete
13. ✅ **Display namespace notification** after creating plugin (show exact namespace to use)
14. ✅ **Display AI-generated code warning** after creating ANY plugin code
15. ✅ **Remind user**: Namespace entered during upload MUST match extension.json modulePath

📖 **For detailed help**, consult the reference documentation files above.
