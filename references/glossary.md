# POD Plugin Glossary

Key terms and definitions for SAP Digital Manufacturing POD plugin development.

---

## Core Concepts

**POD (Production Operator Dashboard)**
- User interface for shop floor operators in SAP Digital Manufacturing
- Provides real-time production data, work lists, and operational controls
- Two versions: POD 1.0 (legacy) and POD 2.0 (modern)

**Widget**
- A custom UI component that can be added to a POD page
- Base class for all POD 2.0 plugins
- Provides lifecycle management, configuration, and event handling

**Plugin**
- Generic term for custom extensions to POD
- Can be widgets (with UI) or content handlers (business logic only)
- Registered via extension.json file

---

## Widget Base Classes

**ControlWidget**
- Widget that wraps a single SAPUI5 control
- Examples: ButtonWidget, InputWidget, TextWidget
- Constructor pattern: `super(ControlClass, oConfig)`

**LayoutWidget**
- Widget that wraps a layout container
- Examples: VBoxWidget, HBoxWidget, PanelWidget
- Constructor pattern: `super(LayoutClass, oConfig)`

**TableWidget**
- Widget for displaying tabular data with columns, sorting, pagination
- Most complex base class
- Requires: `getFields()`, `_createCell()`, `_getModelPath()`

**ContentHandler**
- Business logic without visual representation
- Used for dialogs, form processing, workflows
- Not shown in POD Designer widget palette

---

## State Management

**PodContext**
- Central state management system for POD 2.0
- Provides access to current plant, resources, operations, work lists
- Supports publish/subscribe pattern for reactive updates
- Import path: `sap/dm/dme/pod2/context/PodContext`

**ModelPath**
- Constants for subscribing to POD state changes
- Examples: `ModelPath.FilterResources`, `ModelPath.WorkListItems`
- Used with `PodContext.subscribe(ModelPath.X, callback, context)`
- Import path: `sap/dm/dme/pod2/context/ModelPath`

---

## API Clients

**RestClient**
- Client for calling custom or external REST APIs
- Supports GET, POST, PUT, DELETE
- Usage: `RestClient.post(url, data, headers)`
- Import path: `sap/dm/dme/pod2/api/RestClient`

**ApiClient**
- Client for calling SAP DM public operations
- Type-safe methods for specific operations
- Examples: `ApiClient.sfc.start()`, `ApiClient.order.release()`
- Import path: `sap/dm/dme/pod2/api/ApiClient`

---

## Configuration

**extension.json**
- Registration file for POD 2.0 plugins
- Contains only `widgets` and `actions` arrays
- No other metadata fields (name, version, description) are supported
- Located at root of plugin ZIP file

**getDefaultConfig()**
- Static method that defines default widget configuration
- Returns object with `properties` key
- Never spread parent properties - define directly

**WidgetProperty**
- Metadata definition for widget configuration properties
- Used with property editors (StringPropertyEditor, IntegerPropertyEditor, etc.)
- Never use binding syntax in displayName or description

---

## Lifecycle

**POD 2.0 Widget Lifecycle Order:**
```
1. constructor(oConfig)     ← Widget instantiated
2. _createView()            ← UI created - MODEL MUST EXIST HERE!
3. onInit()                 ← Async initialization, subscriptions
4. [widget is rendered]
5. onExit()                 ← Cleanup when destroyed
```

**_createView()**
- Required method that creates and returns the widget's UI
- Must pass `oConfig.id` as first parameter to root control
- Called BEFORE `onInit()` - initialize models here, not in onInit()

**onInit()**
- Async initialization method
- Called after _createView()
- Use for PodContext subscriptions and data loading

**onExit()**
- Cleanup method called when widget is destroyed
- Unsubscribe from all PodContext subscriptions
- Clean up control references to prevent memory leaks

---

## Categories

**WidgetCategory**
- Determines where widget appears in POD Designer palette
- Options: Elements, Layout, WorkList, Order, SFC, DataCollection, Hidden
- Import path: `sap/dm/dme/pod2/widget/metadata/WidgetCategory`

**PropertyCategory**
- Organizes widget properties in configuration panel
- Options: Main, Appearance, Dimension, Data, Events
- Import path: `sap/dm/dme/pod2/propertyeditor/PropertyCategory`

---

## Common Terms

**Plant**
- Manufacturing site/facility identifier
- Retrieved via `PodContext.getPlant()`

**Resource**
- Work center, machine, or production line
- Current resource(s): `PodContext.get(ModelPath.FilterResources)`

**SFC (Shop Floor Control)**
- Production order or work order identifier
- Core entity in SAP DM representing a unit of production

**Operation**
- Manufacturing step or routing operation
- Retrieved via `PodContext.get(ModelPath.CurrentOperation)`

**Work List**
- List of available work items (SFCs, orders) for a resource
- Data: `PodContext.get(ModelPath.WorkListItems)`
- Selection: `PodContext.get(ModelPath.SelectedWorkListItems)`

---

## File Structure

**CRITICAL**: POD plugins do NOT use webapp/ folder! Files are generated in working directory root!

### 🚨 IMPORTANT: File Generation Approach
- **User is already IN their namespace folder** (working directory IS the namespace folder)
- **Generate files directly in working directory root**, NOT nested namespace subfolders
- ❌ Don't create `mycompany/`, `custom/pod2/`, or any namespace folders
- ✅ Create files at root: `extension.json`, `widget/MyWidget.js`

### ✅ Correct Structure (Official SAP Pattern):
```
<working-directory>/             # ← User is already here (this IS the namespace folder)
├── extension.json               # ← Generate at working directory root
├── widget/                      # Widgets folder (SAP recommended)
│   ├── MyWidget.js              # Widget implementation
│   └── i18n/
│       └── i18n.properties      # Translations
├── action/                      # Actions folder (SAP recommended)
│   └── MyAction.js
└── util/                        # Utilities folder (SAP recommended)
    └── Helper.js
```

**File Path Convention When Generating:**
- `extension.json` (NOT `mycompany/extension.json`)
- `widget/MyWidget.js` (NOT `mycompany/widget/MyWidget.js`)
- `action/MyAction.js` (NOT `mycompany/action/MyAction.js`)

**Module Path Convention (for extension.json content):**
- Namespace folder = working directory name (e.g., if user is in `/home/user/mycompany`, namespace is `mycompany`)
- extension.json = at working directory root
- Subfolders = module organization (`widget/`, `action/`, `util/`)
- Full module path: `mycompany/widget/MyWidget` (namespace prefix + relative path)
- Module paths in extension.json are relative to extension.json location

### ❌ WRONG Structure (SAPUI5 App - Don't do this!):
```
my-custom-plugin/
└── webapp/                      # ❌ Breaks POD plugin upload!
    ├── manifest.json           # ❌ Not needed
    ├── Component.js            # ❌ Not needed
    └── extension.json          # ❌ Wrong location!
```

### ❌ WRONG Structure (extension.json outside namespace):
```
├── extension.json               # ❌ WRONG! Causes "Missing file" error!
└── mycompany/
    └── widget/
        └── MyWidget.js
```
**Why this fails:** Module path is `mycompany/widget/MyWidget`, but extension.json is outside the namespace folder. Relative paths don't resolve correctly.

**Remember**: POD plugins ≠ SAPUI5 applications

**Alternative Simple Structure (for single widget):**
```
simpleplugin/
├── extension.json              # At root of namespace folder
└── MyWidget.js                 # Widget directly at root
```

**ZIP File Structure for Upload:**
```
mycompany.zip
└── mycompany/                  # ← Namespace folder IS the zip content
    ├── extension.json          # ← Inside namespace folder
    ├── widget/
    │   ├── MyWidget.js         # Widget class file
    │   └── i18n/
    │       ├── i18n_en.properties  # English translations
    │       └── i18n_de.properties  # German translations (optional)
    ├── action/                 # Optional
    │   └── MyAction.js
    └── util/                   # Optional
        └── thirdPartyLib/      # Third-party libraries (optional)
            ├── moment.min.js
            └── lodash.min.js
```

**How to Create Correct Zip:**
```bash
# User must cd to PARENT directory first, then zip their working directory
cd ..
zip -r mycompany.zip mycompany/
# or PowerShell:
cd ..
Compress-Archive -Path mycompany -DestinationPath mycompany.zip
```

**Why This Matters:**
- **During development**: Working directory IS the namespace folder (generate files at root level)
- **For deployment**: Zip the working directory from parent folder
- **In the zip**: Namespace folder contains extension.json and all files
- **Module resolution**: Paths are relative to extension.json location

---

## Error Messages

**Common Errors:**

1. `"is of type string, expected sap.m.InputType for property 'type'"`
   - Cause: Using binding syntax in WidgetProperty or spreading parent properties
   - Fix: Use method calls for i18n, define properties directly

2. `"404 - Failed to load PodContext.js"`
   - Cause: Wrong import path (model/ instead of context/)
   - Fix: Use `sap/dm/dme/pod2/context/PodContext`

3. `"getView method returned a view with a different ID than configuration"`
   - Cause: Missing view ID in _createView()
   - Fix: Pass `oConfig.id` as first parameter to root control

4. `"TypeError: aResources.map is not a function"`
   - Cause: Wrong callback parameter order
   - Fix: Signature is `(newValue, path)` not `(path, newValue)`

---

## Navigation

📖 **Back to main skill**: [SKILL.md](../SKILL.md)

**Reference Files**:
- [Common Mistakes](common-mistakes.md) - All 28 categorized mistakes with fixes
- [Widget Patterns](widget-patterns.md) - Complete widget code patterns
- [TableWidget Complete](tablewidget-complete.md) - Full TableWidget implementation guide
- [Table Cell Patterns](tablecell-patterns.md) - Column and cell creation patterns
- [Binding Patterns](binding-patterns.md) - Data binding and formatter patterns
- [Production Patterns](production-patterns-unified.md) - 35 patterns from SAP production code
- [Advanced Patterns](advanced-patterns.md) - Complex scenarios and optimizations
- [POD 2.0 API Reference](pod2-api-reference.md) - Framework API documentation
- [SAP API Business Hub](https://api.sap.com/package/SAPDigitalManufacturingCloud) - SAP Digital Manufacturing API specifications
