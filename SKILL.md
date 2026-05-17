---
name: pod-plugin
description: "Create SAP Digital Manufacturing POD 2.0 plugins. Trigger when users mention: POD plugins, POD widgets, POD 2.0, SAP DM customization, TableWidget, ControlWidget, LayoutWidget, PodContext, extension.json, work center plugins. Expert in ES6 class-based POD 2.0 development. CRITICAL: Never creates webapp/ folder. Files generated at working directory root."
version: 28.0.0
author: Claude
tags: [sap, digital-manufacturing, pod, plugin, pod2, widget-architecture, production-patterns, delegate-patterns, no-webapp-folder, memory-leak-prevention]
compatibility:
  environment: SAP Business Technology Platform (BTP) with SAP Digital Manufacturing
  requirements:
    - SAP DM POD Designer access
    - Extension Center upload permissions
---

# POD 2.0 Plugin Development

Expert guide for SAP Digital Manufacturing POD 2.0 plugin development.

## Documentation

| Topic | File | Description |
|-------|------|-------------|
| **Pattern Index** | [PATTERN-INDEX.md](references/PATTERN-INDEX.md) | Quick reference by widget type and complexity |
| **Widget Patterns** | [widget-patterns.md](references/widget-patterns.md) | Complete widget templates |
| **TableWidget** | [tablewidget-complete.md](references/tablewidget-complete.md) | IGNORE_TABLE_PROPERTIES, selection sync |
| **Table Cells** | [tablecell-patterns.md](references/tablecell-patterns.md) | 13 cell types |
| **Delegates** | [delegate-architecture.md](references/delegate-architecture.md) | 8 official delegates |
| **Common Mistakes** | [common-mistakes.md](references/common-mistakes.md) | 28 mistakes with fixes |
| **Error Handling** | [error-handling.md](references/error-handling.md) | Error decision matrix |
| **API Reference** | [pod2-api-reference.md](references/pod2-api-reference.md) | PodContext, ModelPath |
| **SAP DM APIs** | [sapdm-api-reference.md](references/sapdm-api-reference.md) | 70+ REST APIs |

---

## Critical Rules

### 1. Required Information (Ask First!)

Before creating a plugin, ask:
1. **Namespace** - e.g., `custom/pod2/myproject`
2. **Plugin Name** - e.g., `ProductionStatus`
3. **Category** - Default: `Custom`

### 2. Base Class Selection

| Need | Base Class | Spread Parent Config? |
|------|------------|----------------------|
| Single control (button, input) | `ControlWidget` | ❌ NO |
| Container for widgets | `LayoutWidget` | ✅ YES |
| Data table | `TableWidget` | ✅ YES |
| Business logic only | `ContentHandler` | N/A |

### 3. Correct Import Paths

```javascript
// ✅ CORRECT - Verified against official SAP plugins
"sap/dm/dme/pod2/context/PodContext"
"sap/dm/dme/pod2/context/ModelPath"
"sap/dm/dme/pod2/context/data/WorkListDelegate"  // All delegates use context/data/
"sap/dm/dme/pod2/model/I18nResourceModel"
"sap/dm/dme/pod2/api/ApiClient"
"sap/dm/dme/pod2/Logger"

// ❌ WRONG - Common mistakes
"sap/dm/dme/pod2/model/PodContext"      // Wrong: model/ instead of context/
"sap/dm/dme/pod2/delegate/..."          // Wrong: delegate/ instead of context/data/
```

### 4. ModelPath Constants (ALL PLURAL)

```javascript
ModelPath.SelectedWorkListItems       // ✅ PLURAL (array)
ModelPath.SelectedOperationActivities // ✅ PLURAL (array)
ModelPath.FilterResources             // ✅ PLURAL
ModelPath.LastSelectedWorkListItem    // ✅ Singular (single item getter only)
```

### 5. i18n Pattern

```javascript
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class YourWidget extends Widget {
    static #oI18nModel = new I18nResourceModel({
        bundleName: "your.namespace.i18n.i18n"  // Dots, not slashes!
    });
    
    static getI18nModel() { return this.#oI18nModel; }
    
    _createView() {
        return new Button({
            text: this.getI18nText("key")  // ✅ Method call, NOT binding!
        });
    }
}
```

**CRITICAL**: Use `this.getI18nText()` in `_createView()`. Never use `{i18n>key}` bindings.

### 6. Memory Leak Prevention

```javascript
onInit() {
    PodContext.subscribe(ModelPath.SelectedWorkListItems, this._onSelectionChange, this);
}

onExit() {
    // REQUIRED: Clean up subscriptions
    PodContext.unsubscribe(ModelPath.SelectedWorkListItems, this._onSelectionChange, this);
    // Or use: PodContext.unsubscribeAll(this);
    super.onExit();
}
```

See [Mistake #3](references/common-mistakes.md#mistake-3) for complete patterns.

---

## File Structure

```
<working-directory>/     # User is already in namespace folder
├── extension.json       # Widget registration
├── widget/
│   └── YourWidget.js
└── i18n/
    └── i18n_en.properties
```

**CRITICAL**: Never create namespace folders. Generate files at root.

---

## Pre-Generation Checklist

- [ ] Import paths use `context/` not `model/` for PodContext/ModelPath
- [ ] Delegate paths use `context/data/` not `delegate/`
- [ ] ModelPath constants are plural (SelectedWorkListItems)
- [ ] Root control ID is exactly `oConfig.id` (no suffix)
- [ ] `onExit()` exists if `onInit()` subscribes
- [ ] i18n uses method calls, not bindings

---

## Migration Warning

If user asks to convert POD 1.0 to POD 2.0:

**Recommend RE-ARCHITECTING, not direct conversion.** POD 1.0 (XML views, controllers) and POD 2.0 (ES6 classes, programmatic views) are fundamentally different. Reuse business logic, redesign UI using POD 2.0 patterns.

---

## Post-Generation

After creating plugin:
1. Display namespace notification
2. Display AI-generated code warning
3. Create deployment zip if requested
