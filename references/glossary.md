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

**CRITICAL**: POD plugins do NOT use webapp/ folder!

### ✅ Correct Structure:
```
my-custom-plugin/
├── extension.json              # Widget registration (REQUIRED, at ROOT!)
└── custom/                     # Namespace (your choice)
    └── plugin/
        ├── MyWidget.js         # Widget implementation
        └── i18n/
            └── i18n.properties
```

### ❌ WRONG Structure (SAPUI5 App - Don't do this!):
```
my-custom-plugin/
└── webapp/                      # ❌ Breaks POD plugin upload!
    ├── manifest.json           # ❌ Not needed
    ├── Component.js            # ❌ Not needed
    └── extension.json          # ❌ Wrong location!
```

**Remember**: POD plugins ≠ SAPUI5 applications

**Typical POD 2.0 Plugin Structure:**
```
plugin-name.zip
├── extension.json              # Widget registration (REQUIRED)
├── plugins/
│   ├── MyWidget.js             # Widget class file
│   ├── i18n/
│   │   ├── i18n_en.properties  # English translations
│   │   └── i18n_de.properties  # German translations (optional)
│   └── thirdPartyLib/          # Third-party libraries (optional)
│       ├── moment.min.js
│       └── lodash.min.js
└── README.md                    # Documentation (optional)
```

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

**Other references**:
- [Common Mistakes](common-mistakes.md) - All 11 mistakes with fixes
- [Widget Patterns](widget-patterns.md) - Complete code patterns
