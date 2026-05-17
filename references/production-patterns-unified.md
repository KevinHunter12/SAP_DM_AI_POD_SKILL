# Production Patterns Unified Reference

**Source**: Consolidated from SAP POD 2.0 production code analysis  
**Date**: 2026-04-18  
**Analysis Scope**: 15+ production widgets from SAP Digital Manufacturing v2  

This reference consolidates battle-tested patterns from real SAP production code, eliminating redundancy while preserving all unique implementation patterns.

---

## Table of Contents

1. [Core Foundation Patterns](#core-foundation-patterns)
2. [Advanced Implementation Patterns](#advanced-implementation-patterns)
3. [Widget-Specific Patterns](#widget-specific-patterns)
4. [Enterprise Production Patterns](#enterprise-production-patterns)

---

# Core Foundation Patterns

These patterns appear consistently across ALL production SAP widgets and represent the foundation of POD 2.0 development.

## 1. JSDoc Documentation Standards

**Pattern from Production Code:**
```javascript
/**
 * Custom POD 2.0 Widget
 *
 * @alias custom.namespace.plugins.MyWidget
 * @extends sap.dm.dme.pod2.widget.Widget
 */
class MyWidget extends Widget {

    /**
     * Display name shown in POD Designer
     * @override
     * @extensible
     * @returns {string}
     */
    static getDisplayName() {
        return "My Widget";
    }

    /**
     * Icon for widget palette (SAP icon name)
     * @override
     * @returns {string}
     */
    static getIcon() {
        return "sap-icon://my-icon";
    }

    /**
     * Constructor
     * @param {Object} oConfig - Widget configuration
     */
    constructor(oConfig) {
        super(oConfig);
    }

    /**
     * Lifecycle method - called after widget initialization
     * @override
     */
    async onInit() {
        await super.onInit();
    }

    /**
     * Create and return the widget view
     * @override
     * @returns {sap.ui.core.Control}
     */
    _createView() {
        // Implementation
    }
}
```

**Key Points:**
- Use `@alias` for the full class path
- Use `@extends` to show inheritance
- Use `@override` for overridden methods
- Use `@extensible` for methods that subclasses can override
- Use `@param` and `@returns` with types

---

## 2. Private Fields and Encapsulation

**Modern ES2022 Pattern:**
```javascript
class MyWidget extends TableWidget {
    // Private field with # prefix (truly private)
    #oLog = Logger.getLogger("custom.namespace.MyWidget");

    // Typed accessor for static class reference
    #static = /** @type {typeof sap.dm.dme.pod2.widget.core.TableWidget} */(this.constructor);

    // Private control references
    #oTable;
    #oDialog;

    // Protected field with _ prefix (convention, not enforced)
    _sCustomFieldProperty;
    _bAllowUomFlag;

    onInit() {
        super.onInit();
        this.#oLog.info("Widget initializing");

        if (!this.#oTable) {
            this.#oLog.warn("Table not initialized");
        }
    }
}
```

**Rules:**
- Use `#field` for truly private fields (ES2022+)
- Use `_field` for protected fields (convention)
- Logger should always be private (`#oLog`)
- Store control references as private fields

---

## 3. Logger Pattern (Consistent Across All Production Code)

**Standard Implementation:**
```javascript
import Logger from "sap/dm/dme/pod2/Logger";

class MyWidget extends Widget {
    // 1. Private logger field - use dots, not slashes
    #oLog = Logger.getLogger("custom.namespace.widget.MyWidget");
    
    async _someMethod() {
        // 2. Debug for development/troubleshooting
        this.#oLog.debug("Method started with param:", sValue);
        
        try {
            // Business logic
            this.#oLog.info("Operation completed successfully");
        } catch (oError) {
            // 3. Error with exception object
            this.#oLog.error("Operation failed", oError);
        }
    }
}
```

**Logger Levels:**

| Level | Method | Use Case |
|-------|--------|----------|
| `debug()` | Verbose | Development, troubleshooting, frequent operations |
| `info()` | Informational | Important state changes, milestones |
| `error()` | Error | Failures, exceptions (always pass Error object as 2nd param) |

**Namespace Convention:**
- Module path: `custom/namespace/widget/MyWidget`
- Logger name: `custom.namespace.widget.MyWidget` ← **dots, not slashes!**

**Best Practices:**
1. ✅ Initialize logger as private field `#oLog`
2. ✅ Use `debug()` for frequent operations
3. ✅ Use `error()` with Error object: `this.#oLog.error("msg", oError)`
4. ✅ Include context in messages: `"Failed to load material ${sMaterial}"`
5. ❌ Don't log sensitive data (credentials, PII)

---

## 4. Enum Patterns with Object.freeze

**Production Pattern from ActivityConfirmationTableWidget:**
```javascript
class MyWidget extends TableWidget {

    /**
     * @enum {string}
     * @property {"customField"} customField
     * @property {"allowOnlyBaseUoM"} allowOnlyBaseUoM
     * @property {"enablePostingDate"} enablePostingDate
     */
    static PropertyId = Object.freeze({
        ...super.PropertyId,  // Spread parent properties!
        customField: "customField",
        allowOnlyBaseUoM: "allowOnlyBaseUoM",
        enablePostingDate: "enablePostingDate"
    });

    /**
     * @enum {string}
     * @property {"parameter"} parameter
     * @property {"standardValue"} standardValue
     */
    static Field = Object.freeze({
        parameter: "parameter",
        standardValue: "standardValue",
        reported: "reported",
        posting: "posting"
    });

    // Usage in code
    getProperties() {
        return [
            new WidgetProperty({
                propertyEditor: new StringPropertyEditor(
                    this,
                    this.constructor.PropertyId.customField  // ← Type-safe!
                )
            })
        ];
    }

    _someMethod() {
        const sField = this.constructor.Field.parameter;  // ← No typos!
    }
}
```

**Key Points:**
- Use `Object.freeze()` to prevent modification
- Spread parent enums when extending: `...super.PropertyId`
- Document with `@enum` JSDoc
- Access via `this.constructor.PropertyId.X` (not `MyClass.PropertyId.X`)
- Prevents typos and enables autocomplete

---

## 5. Property Spreading Patterns

**Pattern from Multiple SAP Widgets:**
```javascript
class MyWidget extends TableWidget {

    static getDefaultConfig() {
        return {
            properties: {
                // Spread parent default properties
                ...super.getDefaultConfig().properties,
                // Add/override specific properties
                showNoData: true,
                mode: ListMode.SingleSelectMaster,
                growingScrollToLoad: true
            }
        };
    }

    static EXCLUDE_PROPERTIES = [
        // Spread parent exclusions
        ...TableWidget.EXCLUDE_PROPERTIES,
        // Add specific exclusions
        "headerText"
    ];
}
```

**Key Points:**
- Always spread parent defaults: `...super.getDefaultConfig().properties`
- Override specific properties after spreading
- Same pattern for EXCLUDE_PROPERTIES, INCLUDE_EVENTS, etc.

---

## 6. Design Mode vs Run Mode

**Pattern from SelectResourceWidget:**
```javascript
class SelectResourceWidget extends ControlWidget {

    constructor(oConfig) {
        // Use different control based on mode
        if (PodContext.isDesignMode()) {
            // Simplified preview control for designer
            super(CustomInput, oConfig);
        } else {
            // Full-featured control for runtime
            super(ResourceSingleInput, oConfig);
        }
    }

    onInit() {
        super.onInit();

        if (PodContext.isDesignMode()) {
            // Design-time setup
            const oPreviewInput = /** @type {sap.m.Input} */(this.getView());
            oPreviewInput.setEditable(false);
            oPreviewInput.setValue(this.getPropertyValue(PropertyId.DefaultResource));

        } else {
            // Runtime setup
            const oControl = /** @type {sap.dm.dme.pod2.valuehelp.ResourceSingleInput} */(this.getView());

            // Set initial value
            const aResources = PodContext.getFilterResources();
            if (Array.isArray(aResources) && aResources.length !== 0) {
                oControl.setSelection(aResources[0]);
            }

            // Subscribe to changes (ONLY in run mode!)
            PodContext.subscribe(ModelPath.FilterResources, (aResources) => {
                if (Array.isArray(aResources) && aResources.length !== 0) {
                    oControl.setSelection(aResources[0]);
                } else {
                    oControl.setSelection(null);
                }
            }, this);
        }
    }
}
```

**Key Points:**
- Check `PodContext.isDesignMode()` for preview behavior
- Check `PodContext.isRunMode()` before subscribing
- Use simpler controls in design mode for performance
- Type-cast controls after getView(): `/** @type {ControlType} */(this.getView())`

---

## 7. Subscription Patterns (⚠️ CRITICAL)

**Pattern from ActivityConfirmationTableWidget:**
```javascript
class ActivityConfirmationTableWidget extends TableWidget {

    async onInit() {
        super.onInit();

        // Only subscribe in run mode
        if (PodContext.isRunMode()) {
            // Subscribe to context changes
            PodContext.subscribe(
                ModelPath.ActivitySummaries,
                () => this._updateReportButtonEnabled(),
                this  // ← Context binding!
            );

            // Initialize delegates
            await ActivityConfirmationDelegate.init();
        }
    }

    onExit() {
        super.onExit();

        // Always unsubscribe
        if (PodContext.isRunMode()) {
            PodContext.unsubscribe(
                ModelPath.ActivitySummaries,
                this._updateReportButtonEnabled,
                this
            );
        }
    }
}
```

**Subscription with Inline Handler:**
```javascript
PodContext.subscribe(ModelPath.FilterResources, (aResources) => {
    // Defensive type checking
    if (Array.isArray(aResources) && aResources.length !== 0) {
        oControl.setSelection(aResources[0]);
    } else {
        oControl.setSelection(null);
    }
}, this);
```

**Multi-Subscription Pattern:**
```javascript
// Subscribe to multiple paths at once
PodContext.subscribe([
    ModelPath.SelectedWorkListItems,
    ModelPath.SelectedOperationActivities
], this._refresh, this);
```

**⚠️ CRITICAL RULES:**
1. ✅ Always check `PodContext.isRunMode()` before subscribing
2. ✅ Pass `this` as third parameter for context binding
3. ✅ Use defensive type checking (`Array.isArray()`)
4. ✅ Handle both empty arrays and null/undefined
5. ✅ **ALWAYS unsubscribe in `onExit()`** (memory leak prevention)

**🚨 Production Bug Found:**
```javascript
// ❌ MISSING onExit() - Memory leak!
onInit() {
    PodContext.subscribe(ModelPath.WorkInstructions, this._updateText, this);
}
// Should have:
onExit() {
    super.onExit();
    PodContext.unsubscribe(ModelPath.WorkInstructions, this._updateText, this);
}
```

**Validation Rule:** If `onInit()` has subscriptions → `onExit()` MUST unsubscribe.

---

## 8. Error Handling Patterns

**API Call Error Handling:**
```javascript
async _loadData() {
    this.#oLog.info("Loading data...");

    try {
        const oData = await ApiClient.custom.post("/endpoint", {
            plant: PodContext.getPlant()
        });

        this.#oLog.debug("Data loaded successfully", oData);
        this._updateDisplay(oData);

        MessageHistory.toast({
            message: "Data loaded successfully",
            type: MessageHistory.Success
        });

    } catch (oError) {
        this.#oLog.error("Failed to load data", oError);

        MessageHistory.showError(
            PodContext.getI18nText("error.loadFailed")
        );

        // Re-throw if caller needs to handle
        throw oError;
    }
}
```

**Error Code Checking Pattern:**
```javascript
try {
    await ApiClient.findBuyoffLogs(oRequest);
} catch (oError) {
    // Check specific error codes
    if (oError?.body?.error?.code === "sfc.notInCompletePending") {
        this._oModel.setData([]);
        return;
    }
    MessageHistory.showError(oError.message);
}
```

**Key Points:**
- Always log at appropriate level (`info`, `debug`, `error`)
- Use `MessageHistory.showError()` for user-facing errors
- Use `MessageHistory.toast()` for success messages
- Re-throw errors if caller needs to handle them
- Use i18n keys for error messages
- Check error codes for specific handling

---

## 9. Delegate Patterns

**Using Delegates for Shared Data Management:**
```javascript
import ActivityConfirmationDelegate from "sap/dm/dme/pod2/context/data/ActivityConfirmationDelegate";
import WorkListDelegate from "sap/dm/dme/pod2/context/data/WorkListDelegate";

class MyWidget extends TableWidget {

    async onInit() {
        super.onInit();

        if (PodContext.isRunMode()) {
            // Initialize delegate (manages data loading/caching)
            await ActivityConfirmationDelegate.init();

            // Subscribe to changes
            // Delegate automatically updates ModelPath.ActivitySummaries
            PodContext.subscribe(
                ModelPath.ActivitySummaries,
                this._onActivitiesChanged,
                this
            );
        }
    }
    
    async _onAccept() {
        await ApiClient.execute(oRequest);
        
        // Trigger work list update after operation
        WorkListDelegate.refreshInBackground();
        
        MessageHistory.showSuccess("Accepted");
    }
}
```

**Data Delegate Pattern (Shared Loading):**
```javascript
class MyWidget extends Widget {
    async onInit() {
        await super.onInit();
        
        // 1. Subscribe to data changes
        PodContext.subscribe(ModelPath.WorkInstructions, this._onDataChanged, this);
        
        // 2. Check if data already loaded
        if (!PodContext.getWorkInstructions()) {
            // 3. Trigger load via delegate (shared across widgets)
            WorkInstructionDelegate.refresh();
        }
    }
    
    _onDataChanged(aWorkInstructions, sPath) {
        // Handle data update
    }
    
    onExit() {
        super.onExit();
        PodContext.unsubscribe(ModelPath.WorkInstructions, this._onDataChanged, this);
    }
}
```

**Benefits:**
- ✅ No duplicate API calls
- ✅ Consistent data across widgets
- ✅ Centralized error handling

**Key Points:**
- Delegates handle data fetching and caching
- Initialize delegates in `onInit()`
- Subscribe to the ModelPath that the delegate manages
- Common delegates:
  - `ActivityConfirmationDelegate`
  - `WorkListDelegate`
  - `DataCollectionDelegate`
  - `WorkInstructionDelegate`

---

## 10. EXCLUDE_PROPERTIES Pattern

**Purpose:** Hide properties from POD Designer property panel (but still usable in code).

**Pattern:**
```javascript
class MyInputWidget extends InputWidget {
    static EXCLUDE_PROPERTIES = [
        ...InputWidget.EXCLUDE_PROPERTIES,  // ✅ Always spread parent
        "type",        // Hardcoded in getDefaultConfig
        "maxLength"    // Not applicable
    ];
}
```

**ControlWidget Example:**
```javascript
class SelectResourceWidget extends ControlWidget {

    static EXCLUDE_PROPERTIES = [
        "autocomplete", "dateFormat", "enableSuggestionsHighlighting",
        "enableTableAutoPopinMode", "fieldWidth", "filterSuggests",
        "forceUpperCase", "maxLength", "maxSuggestionWidth", "name",
        "selectedKey", "showSuggestion", "showTableSuggestionValueHelp",
        "showValueHelp", "showValueStateMessage", "startSuggestion",
        "suggestionRowValidator", "textFormatMode", "textFormatter",
        "type", "value", "valueHelpIconSrc", "valueHelpOnly",
        "valueLiveUpdate", "valueState", "valueStateText"
    ];

    static INCLUDE_EVENTS = [ ];  // No events exposed

    static PROPERTY_CATEGORY_OVERRIDE = {
        description: PropertyCategory.Appearance,
        placeholder: PropertyCategory.Appearance,
        required: PropertyCategory.Behavior,
        showClearIcon: PropertyCategory.Behavior
    };
}
```

**Production Example:**
```javascript
// WorkInstructionHeaderTextWidget.js
static EXCLUDE_PROPERTIES = [ ...ExpandableTextWidget.EXCLUDE_PROPERTIES, "text" ];
// ↑ "text" set dynamically from work instructions, not user-configurable
```

**Critical:**
- ✅ Always spread parent EXCLUDE_PROPERTIES
- ✅ Document WHY each property excluded (comment)
- ❌ Don't exclude properties users need to configure

---

# Advanced Implementation Patterns

These patterns handle complex scenarios and performance optimization in production widgets.

## 11. Specialized Base Widget Classes

SAP provides pre-built ControlWidget subclasses for common controls with helper methods and best-practice configurations.

### ProgressIndicatorWidget

**Extends**: ControlWidget → ProgressIndicatorWidget  
**Wraps**: `sap.m.ProgressIndicator`

**Inherited Helper Methods**:
- `_getPercentValue(oConfig)` - Calculate percentage from target/actual with fallback support
- `_getDisplayValue(oConfig)` - Format display text with UOM handling

**Usage Pattern:**
```javascript
import ProgressIndicatorWidget from "sap/dm/dme/pod2/widget/core/ProgressIndicatorWidget";

class MyProgressWidget extends ProgressIndicatorWidget {
    static EXCLUDE_PROPERTIES = [ "percentValue", "displayValue" ];
    
    onInit() {
        super.onInit();
        this.#updateBindings();
    }
    
    #updateBindings() {
        const oIndicator = this.getView();
        oIndicator.bindProperty("percentValue", {
            parts: ["", "planned", "completed"],
            formatter: (oItem) => {
                return this._getPercentValue({
                    target: oItem.planned,
                    actual: oItem.completed
                });
            }
        });
    }
}
```

### ImageWidget

**Extends**: ControlWidget → ImageWidget  
**Wraps**: `sap.m.Image`

**Static Properties**:
- `ImageWidget.PLACEHOLDER_IMAGE_URL` - Default placeholder image

**Usage:**
```javascript
import ImageWidget from "sap/dm/dme/pod2/widget/core/ImageWidget";

class MyImageWidget extends ImageWidget {
    static EXCLUDE_PROPERTIES = [ ...ImageWidget.EXCLUDE_PROPERTIES, "src", "alt" ];
    
    async _updateImage() {
        const oImage = this.getView();
        try {
            const sUrl = await this._getImageUrl();
            oImage.setSrc(sUrl);
        } catch (e) {
            oImage.setSrc(ImageWidget.PLACEHOLDER_IMAGE_URL);
        }
    }
}
```

### ExpandableTextWidget

**Extends**: ControlWidget → ExpandableTextWidget  
**Wraps**: `sap.m.ExpandableText` for collapsible long text

**Usage:**
```javascript
import ExpandableTextWidget from "sap/dm/dme/pod2/widget/core/ExpandableTextWidget";

class MyTextWidget extends ExpandableTextWidget {
    static EXCLUDE_PROPERTIES = [ ...ExpandableTextWidget.EXCLUDE_PROPERTIES, "text" ];
}
```

**Current Hierarchy:**
```
Widget (abstract)
├── ControlWidget
│   ├── ButtonWidget, InputWidget, TextWidget
│   ├── ProgressIndicatorWidget ← Helper methods for progress calculations
│   ├── ImageWidget ← Placeholder constants and error handling
│   └── ExpandableTextWidget ← Expand/collapse behavior
├── LayoutWidget
├── TableWidget
└── ContentHandler
```

**When to use:** Check SAP-provided specialized classes before extending base tiers.

---

## 12. Multi-Part Property Bindings (Alternative to Subscriptions)

For **computed properties** derived from model data, use multi-part bindings instead of PodContext subscriptions. This is the pattern used by `ProgressIndicatorWidget` and similar widgets.

### When to Use

- ✅ Computing derived values from model properties (%, formatted text)
- ✅ Values that update automatically with model changes
- ✅ Display-only properties (no user interaction needed)
- ❌ Don't use for: async operations, API calls, complex logic

### Pattern

```javascript
class MyProgressWidget extends ProgressIndicatorWidget {
    onInit() {
        super.onInit();
        this.#updateBindings();
    }
    
    #updateBindings() {
        const oControl = this.getView();
        
        // 1. Bind the context
        oControl.bindObject(ModelPath.LastSelectedWorkListItem);
        
        // 2. Bind property with multiple parts
        oControl.bindProperty("percentValue", {
            parts: [
                "",              // "" = whole object (for instanceof check)
                "planned",       // Individual properties that trigger updates
                "completed",
                "uom"
            ],
            formatter: (oItem) => {
                // 3. Type-safe check
                if (oItem instanceof OrderWorkListItem) {
                    // 4. Compute value
                    return this._calculatePercent(oItem.planned, oItem.completed);
                }
                return 0;
            }
        });
    }
}
```

### Why List All Properties?

```javascript
parts: [
    "",           // Whole object - enables instanceof check
    "field1",     // Triggers update when field1 changes
    "field2"      // Triggers update when field2 changes
]
```

The formatter runs when **ANY** listed property changes. Listing all relevant fields ensures the binding updates correctly.

### Comparison: Binding vs Subscription

| Aspect | Multi-Part Binding | PodContext Subscription |
|--------|-------------------|------------------------|
| **Use Case** | Computed display values | Async logic, API calls |
| **Lifecycle** | Automatic (framework) | Manual subscribe/unsubscribe |
| **Update Trigger** | Any bound property changes | Specific ModelPath changes |
| **onExit()** | Not needed | ⚠️ Required (unsubscribe) |
| **Type Checking** | instanceof in formatter | Check in callback |
| **Performance** | Efficient (framework) | Manual optimization needed |

---

## 13. State Caching to Avoid Redundant API Calls

When widgets subscribe to PodContext changes, the callback fires on **every change** - even if relevant data hasn't changed. Cache previous values to avoid redundant API calls.

### Pattern

```javascript
class MyWidget extends Widget {
    // Private cache
    #oCurrentItem = {
        id: null,
        version: null
    };
    
    async onInit() {
        await super.onInit();
        PodContext.subscribe(ModelPath.SelectedWorkListItems, this._onSelectionChanged, this);
        await this._onSelectionChanged();  // Initial load
    }
    
    async _onSelectionChanged() {
        const oItem = PodContext.getLastSelectedWorkListItem();
        
        // 1. Check if relevant data changed
        if (oItem.id === this.#oCurrentItem.id &&
            oItem.version === this.#oCurrentItem.version) {
            this.#oLog.debug("Selection unchanged, skipping API call");
            return;
        }
        
        // 2. Fetch data (expensive operation)
        const oData = await this._fetchData(oItem.id, oItem.version);
        
        // 3. Update cache AFTER successful fetch
        this.#oCurrentItem = {
            id: oItem.id,
            version: oItem.version
        };
        
        // 4. Update UI
        this._updateView(oData);
    }
}
```

### Why This Matters

Without caching:
- ❌ API called every time user selects same item
- ❌ Network overhead, slower UI response
- ❌ Higher backend load

With caching:
- ✅ API called only when data actually changes
- ✅ Fast response for repeated selections
- ✅ Reduced backend load

### Cache Invalidation

```javascript
_onRefresh() {
    // Clear cache to force re-fetch
    this.#oCurrentItem = { id: null, version: null };
    this._onSelectionChanged();
}
```

---

## 14. Design Mode Mock Data Patterns

Widgets should provide sample data during design mode so users can configure layout/styling without connecting to a real system.

### Pattern 1: Simple Placeholder

```javascript
async _fetchData() {
    if (PodContext.isDesignMode()) {
        return "Sample Data";  // Simple static value
    }
    // Real API call
    return await ApiClient.getData();
}
```

### Pattern 2: Load from File

For complex mock data (tables, long text), store in a file:

```javascript
async _fetchData() {
    if (PodContext.isDesignMode()) {
        // Load from your plugin's mock data file
        const sUrl = sap.ui.require.toUrl("custom/namespace/data/mockData.json");
        const oResponse = await fetch(sUrl);
        return await oResponse.json();
    }
    // Real API call
    return await ApiClient.getData();
}
```

**File structure:**
```
your-plugin/
├── extension.json
├── widget/
│   └── MyWidget.js
└── data/                    # Mock data folder
    ├── mockData.json
    └── sampleText.txt
```

### Pattern 3: Placeholder from Base Class

```javascript
if (PodContext.isDesignMode()) {
    return ImageWidget.PLACEHOLDER_IMAGE_URL;  // Inherited constant
}
```

### Best Practices

1. ✅ Check `PodContext.isDesignMode()` FIRST
2. ✅ Return synchronously when possible
3. ✅ Provide realistic sample data
4. ❌ Don't make real API calls in design mode
5. ❌ Don't show errors in design mode

---

## 15. Primary/Fallback Property Pattern

When data may exist in multiple forms (e.g., production UOM vs base UOM), use primary/fallback pattern for robustness.

### Common Scenario: UOM Handling

Work list items may have quantities in:
1. **Production UOM** - Preferred (orderQuantityPlannedInProductionUom)
2. **Base UOM** - Fallback (orderQuantityPlanned)

### Pattern

```javascript
_calculatePercent(oItem) {
    return this._getPercentValue({
        // Try production UOM first
        target: oItem.orderQuantityPlannedInProductionUom,
        actual: oItem.orderQuantityCompletedInProductionUom,
        // Fall back to base UOM
        targetFallback: oItem.orderQuantityPlanned,
        actualFallback: oItem.orderQuantityCompleted
    });
}

_formatDisplay(oItem) {
    return this._getDisplayValue({
        // Primary: production UOM
        target: oItem.orderQuantityPlannedInProductionUom,
        actual: oItem.orderQuantityCompletedInProductionUom,
        uom: oItem.productionCommercialUom,
        // Fallback: base UOM
        targetFallback: oItem.orderQuantityPlanned,
        actualFallback: oItem.orderQuantityCompleted,
        uomFallback: oItem.baseCommercialUom
    });
}
```

### Implementation (ProgressIndicatorWidget)

`ProgressIndicatorWidget` base class provides helper methods:

```javascript
class MyProgressWidget extends ProgressIndicatorWidget {
    #updateBindings() {
        const oIndicator = this.getView();
        
        oIndicator.bindProperty("percentValue", {
            parts: ["", "planned", "completed", "plannedUom", "completedUom"],
            formatter: (oItem) => {
                // Base class method handles primary/fallback logic
                return this._getPercentValue({
                    target: oItem.plannedUom,      // Try this first
                    actual: oItem.completedUom,
                    targetFallback: oItem.planned, // Use if primary is null
                    actualFallback: oItem.completed
                });
            }
        });
    }
}
```

### When to Use

- ✅ Data may exist in multiple UOMs (production vs base)
- ✅ Properties may be null/undefined in some contexts
- ✅ Need robust handling across different POD pages

Common cases: Quantity, Status text, Dates

---

## 16. Type Safety with instanceof

Work list items come in different types depending on POD page context. Use `instanceof` for type-safe property access.

### Common Types

```javascript
import WorkListItem from "sap/dm/dme/pod2/context/type/WorkListItem";
import OrderWorkListItem from "sap/dm/dme/pod2/context/type/OrderWorkListItem";
import BaseWorkListItem from "sap/dm/dme/pod2/context/type/BaseWorkListItem";
import OperationActivity from "sap/dm/dme/pod2/context/type/OperationActivity";
import Resource from "sap/dm/dme/pod2/context/type/Resource";
```

**Hierarchy:**
```
BaseWorkListItem (abstract)
├── WorkListItem (Operation Activity context)
└── OrderWorkListItem (Order context)
```

### Pattern

```javascript
_onSelectionChanged(aItems) {
    const oItem = aItems[0];
    
    if (oItem instanceof OrderWorkListItem) {
        // Order-specific properties
        const sOrder = oItem.order;
        const nQty = oItem.orderQuantityPlanned;
    } else if (oItem instanceof WorkListItem) {
        // Operation Activity-specific properties
        const sSfc = oItem.sfc;
        const nQty = oItem.sfcQuantity;
    } else {
        // Fallback for unknown types
        this.#oLog.warning("Unknown work list item type");
    }
}
```

### In Formatters

```javascript
oControl.bindProperty("value", {
    parts: ["", "field1"],
    formatter: (oItem) => {
        if (oItem instanceof OrderWorkListItem) {
            return oItem.orderQuantityPlanned;
        } else if (oItem instanceof WorkListItem) {
            return oItem.sfcQuantity;
        }
        return 0;  // Fallback
    }
});
```

### Type Validation Pattern

```javascript
if (!(oOp instanceof OperationActivity)) {
    throw new Error("Invalid type");
}

const oResource = Resource.fromInternalODataResponse(oData.plannedResource);
```

### Type-Specific Property Examples

| Property | OrderWorkListItem | WorkListItem |
|----------|------------------|--------------|
| `order` | ✅ | ❌ |
| `sfc` | ❌ | ✅ |
| `orderQuantityPlanned` | ✅ | ❌ |
| `sfcQuantity` | ❌ | ✅ |
| `material` | ✅ | ✅ (common) |
| `resource` | ✅ | ✅ (common) |

---

## 17. PodContext Direct Getters

For convenience, PodContext provides direct getter methods useful in subscription callbacks or one-time reads.

### Common Getters

```javascript
// Get single item from selection
const oItem = PodContext.getLastSelectedWorkListItem();

// Get all selected items
const aItems = PodContext.getSelectedWorkListItems();

// Get current plant
const sPlant = PodContext.getPlant();

// Get current resource
const oResource = PodContext.getCurrentResource();

// Get selected operations
const aOps = PodContext.getSelectedOperationActivities();

// Get work instructions
const aWI = PodContext.getWorkInstructions();

// Get selected work instruction
const oWI = PodContext.getSelectedWorkInstruction();
```

### Usage Pattern: Subscribe + Getter

Production pattern: Subscribe to changes, then use getter to read value.

```javascript
async onInit() {
    await super.onInit();
    // Subscribe to trigger callback on changes
    PodContext.subscribe(
        ModelPath.SelectedWorkListItems,  // Array path
        this._onSelectionChanged,
        this
    );
    // Initial load
    await this._onSelectionChanged();
}

async _onSelectionChanged() {
    // Use getter for convenience (returns single item)
    const oItem = PodContext.getLastSelectedWorkListItem();
    
    if (!oItem) {
        this.#oLog.error("No item selected");
        return;
    }
    
    // Process single item
    await this._processItem(oItem);
}
```

### Why This Pattern?

Using the getter is cleaner when you only care about the last selected item:

```javascript
// ❌ With parameter - more code
_onSelectionChanged(aItems) {
    if (!aItems || aItems.length === 0) return;
    const oItem = aItems[aItems.length - 1];
    // ...
}

// ✅ With getter - cleaner
_onSelectionChanged() {
    const oItem = PodContext.getLastSelectedWorkListItem();
    if (!oItem) return;
    // ...
}
```

---

## 18. JSDoc Type Casting for IDE Support

**Pattern:** Use JSDoc annotations for better IDE support and type safety.

**Production Examples:**
```javascript
// Cast getView() return type
const oText = /** @type {sap.m.ExpandableText} */ (this.getView());

// Cast getObject() return type
const oObject = /** @type {sap.dm.dme.pod2.context.type.WorkInstruction} */
    (oListItem.getBindingContext().getObject());

// Cast event type
const oListItem = /** @type {sap.m.ListBase$ItemPressEvent} */(oEvent).getParameter("listItem");
```

**Common Use Cases:**

**1. Cast getView() Return Type:**
```javascript
_updateDisplay() {
    const oText = /** @type {sap.m.ExpandableText} */ (this.getView());
    oText.setText("New text");  // ✅ IDE suggests setText()
}
```

**2. Cast Binding Context Objects:**
```javascript
_onItemPress(oEvent) {
    const oListItem = oEvent.getParameter("listItem");
    const oWorkInstruction = /** @type {sap.dm.dme.pod2.context.type.WorkInstruction} */
        (oListItem.getBindingContext().getObject());
    
    const sId = oWorkInstruction.workInstruction;  // ✅ IDE suggests properties
}
```

**3. Cast Event Types:**
```javascript
_handleEvent(sEventId, oEvent) {
    if (sEventId === "itemPress") {
        const oListItem = /** @type {sap.m.ListBase$ItemPressEvent} */(oEvent)
            .getParameter("listItem");
    }
}
```

**Benefits:**
- ✅ IntelliSense/autocomplete in IDE
- ✅ Type checking in supporting IDEs
- ✅ Documentation of expected types
- ✅ Catches type errors during development

---

# Widget-Specific Patterns

Patterns for specific widget types and scenarios.

## 19. CustomPanel & CustomVBox (⚠️ CRITICAL for Designer)

Use POD-specific wrappers for Designer drag-and-drop support:

```javascript
import CustomPanel from "sap/dm/dme/pod2/control/CustomPanel";
import CustomVBox from "sap/dm/dme/pod2/control/CustomVBox";

_createView() {
    return new CustomPanel({
        id: this.getId(),  // CRITICAL - enables Designer functionality
        width: "100%",
        content: [
            new CustomVBox({
                paddingTop: "Small",
                items: [/* controls */]
            })
        ]
    });
}
```

**Why This Matters:**
- ✅ Enables drag-and-drop in POD Designer
- ✅ Proper widget lifecycle management
- ✅ Consistent behavior across POD
- ❌ Regular `Panel`/`VBox` won't work correctly

---

## 20. ContentHandler Production Pattern

**Pattern from ReportActivityContentHandler:**

```javascript
/**
 * @alias sap.dm.dme.pod2.widget.activityconfirmation.ReportActivityContentHandler
 * @extensible
 */
class ReportActivityContentHandler {
    /** @type {sap.ui.model.json.JSONModel} */
    _oModel;

    /** @type {sap.m.Dialog} */
    _oDialog;

    /** @type {sap.m.Button} */
    _oConfirmButton;

    /** @type {sap.ui.layout.form.SimpleForm} */
    _oForm;

    /** @type {sap.dm.dme.pod2.Logger} */
    #oLog = Logger.getLogger("sap.dm.dme.pod2.widget.activityconfirmation.ReportActivityContentHandler");

    /** @type {Record<string, sap.dm.dme.pod2.api.internal.product.UnitOfMeasure>} */
    #mUomMap = {};

    constructor() {
        this._oModel = new JSONModel();
    }

    /**
     * Opens the content as a dialog.
     *
     * @extensible
     * @param {Object} oData - Data to populate the model.
     */
    async openAsDialog(oData) {
        this.#oLog.info("Opening dialog with data", oData);

        // Set model data
        this._oModel.setData(oData);

        // Create form content
        await this._createForm();
        this._oForm.setModel(this._oModel);

        // Create dialog
        const oDialog = new Dialog({
            title: PodContext.getI18nText("reportActivityDialog.title"),
            contentWidth: "30%",
            resizable: true,
            draggable: true,
            busyIndicatorDelay: 0,
            content: [ this._oForm ],
            buttons: [
                new Button({
                    text: PodContext.getI18nText("confirm.btn"),
                    type: ButtonType.Emphasized,
                    press: () => this._onConfirmButtonPress()
                }),
                new Button({
                    text: PodContext.getI18nText("cancel.btn"),
                    press: () => oDialog.close()
                })
            ],
            afterClose: () => oDialog.destroy()  // ← Always destroy!
        });

        // Store references
        this._oDialog = oDialog;
        this._oConfirmButton = oDialog.getButtons()[0];

        // Update button state
        this._updateConfirmButtonStatus();

        // Open dialog
        oDialog.open();
    }

    async _createForm() {
        // Create form fields...
        this._oForm = new SimpleForm({
            // ...
        });
    }

    async _onConfirmButtonPress() {
        this.#oLog.info("Confirm button pressed");

        const oData = this._oModel.getData();

        try {
            // Call API
            await ApiClient.custom.post("/confirm", oData);

            MessageHistory.toast({
                message: "Success",
                type: MessageHistory.Success
            });

            this._oDialog.close();

        } catch (oError) {
            this.#oLog.error("Confirmation failed", oError);
            MessageHistory.showError("Operation failed");
        }
    }
}
```

**Key Points:**
- ContentHandlers don't extend Widget
- Store `_oModel`, `_oDialog`, `_oForm` as instance fields
- Use `async openAsDialog(oData)` as entry point
- Always `afterClose: () => oDialog.destroy()` to prevent memory leaks
- Store button references for enabling/disabling

---

## 21. ImageWidget Error Handling

Images from URLs may fail to load. Handle errors gracefully with fallback.

### Pattern

```javascript
import ImageWidget from "sap/dm/dme/pod2/widget/core/ImageWidget";
import MessageHistory from "sap/dm/dme/pod2/context/MessageHistory";

class MyImageWidget extends ImageWidget {
    async onInit() {
        super.onInit();
        
        // 1. Attach error handler
        const oImage = this.getView();
        oImage.attachError(this._onImageLoadError.bind(this));
        
        // 2. Load initial image
        await this._loadImage();
    }
    
    _onImageLoadError(oEvent) {
        const oImage = this.getView();
        
        // Only notify if image was expected (not blank)
        if (oImage.getSrc()) {
            MessageHistory.push({
                message: this.getI18nText("error.imageLoadFailed"),
                type: MessageHistory.Error
            });
        }
        
        // Show placeholder
        oImage.setSrc(ImageWidget.PLACEHOLDER_IMAGE_URL);
    }
    
    async _loadImage() {
        const oImage = this.getView();
        try {
            const sUrl = await this._getImageUrl();
            oImage.setSrc(sUrl);
        } catch (oError) {
            // API error - use placeholder immediately
            this.#oLog.error("Failed to get image URL", oError);
            oImage.setSrc(ImageWidget.PLACEHOLDER_IMAGE_URL);
        }
    }
}
```

### Two Error Scenarios

| Scenario | When | Handle |
|----------|------|--------|
| **API Error** | `_getImageUrl()` fails | Catch, log, set placeholder |
| **Load Error** | Browser can't load image URL | `attachError`, notify user, set placeholder |

### Why Check getSrc()?

```javascript
_onImageLoadError(oEvent) {
    // Only notify if src was set (not blank/initial state)
    if (this.#getImageControl().getSrc()) {
        MessageHistory.push({ /* ... */ });
    }
}
```

Prevents showing error when image is intentionally blank.

### Best Practices

1. ✅ Always attach error handler in `onInit()`
2. ✅ Use `ImageWidget.PLACEHOLDER_IMAGE_URL` for fallback
3. ✅ Log API errors separately from load errors
4. ✅ Only show user notification for unexpected failures
5. ✅ Check `getSrc()` before showing error

---

## 22. Bidirectional Sync (Table Selection ↔ PodContext)

**Use Case:** Table selection stays in sync with PodContext selection

**Flow:**
```
User clicks table row
    → _handleEvent() updates PodContext
        → PodContext broadcasts change
            → syncTableSelectionWithPodContext() updates table UI
                → Early exit (already selected)
```

**Complete Pattern:**
```javascript
// Part 1: Subscribe in onInit()
onInit() {
    super.onInit();
    
    if (PodContext.isRunMode()) {
        PodContext.subscribe(ModelPath.SelectedWorkInstruction,
            this.syncTableSelectionWithPodContext.bind(this), this);
    }
}

// Part 2: Sync table when PodContext changes (external update)
syncTableSelectionWithPodContext() {
    const oSelectedWorkInstruction = PodContext.getSelectedWorkInstruction();
    const oTable = this.getTable();
    const oSelectedListItem = oTable.getSelectedItem();
    
    // ✅ Early exit - already in sync
    if (oSelectedListItem && 
        oSelectedListItem.getBindingContext().getObject() === oSelectedWorkInstruction) {
        return;
    }

    oTable.removeSelections();
    if (!oSelectedWorkInstruction) {
        return;
    }

    // Find matching item and select
    const oListItem = oTable.getItems().find((oListItem) => {
        const oObject = oListItem.getBindingContext().getObject();
        return oObject.workInstruction === oSelectedWorkInstruction.workInstruction &&
            oObject.version === oSelectedWorkInstruction.version;
    });
    if (oListItem) {
        oListItem.setSelected(true);
    }
}

// Part 3: Update PodContext when user clicks table (internal update)
_handleEvent(sEventId, oEvent) {
    if (sEventId === "itemPress") {
        const oListItem = oEvent.getParameter("listItem");
        const oWorkInstruction = oListItem.getBindingContext().getObject();
        PodContext.setSelectedWorkInstruction(oWorkInstruction);  // ← Update context
    }
    return super._handleEvent(sEventId, oEvent);
}
```

**Critical Points:**
1. **Early exit optimization** - Check if already in sync
2. **isRunMode() guard** - Only sync in run mode, not designer
3. **Object comparison** - Match by unique identifiers, not reference
4. **Unsubscribe** - Must unsubscribe in onExit()

---

## 23. Custom _getItemBindingInfo() with Filters

**Pattern:** Override to add custom client-side filters

**Production Example:**
```javascript
_getItemBindingInfo(oTable, oTemplate) {
    const oBindingInfo = super._getItemBindingInfo(oTable, oTemplate);

    // Filter out work instructions that only contain HEADER_TEXT
    oBindingInfo.filters = [
        new Filter({
            path: "types",
            test: function(aTypes) {
                return aTypes && aTypes.some((sType) => sType !== WorkInstructionType.HEADER_TEXT);
            }
        })
    ];

    return oBindingInfo;
}
```

**Array Property Filter:**
```javascript
new Filter({
    path: "types",
    test: function(aTypes) {
        // Test array property
        return aTypes && aTypes.some(sType => sType !== "EXCLUDED_TYPE");
    }
})
```

**Critical:**
- ✅ Call super._getItemBindingInfo() first
- ✅ Return modified binding info
- ❌ Don't replace entire binding info

---

## 24. Composite Bindings (Multiple Fields in One Cell)

**Pattern:** Combine multiple model properties into one display value

**Production Example:**
```javascript
_createCell(oColumnConfig) {
    switch (oColumnConfig.field) {
        case WorkInstructionTableWidget.Field.OperationActivityStepId:
            return new Text({
                text: {
                    parts: [ "operationActivity", "stepId" ],  // ← Multiple bindings
                    formatter: (sOperationActivity, sStepId) => {
                        if (sOperationActivity === "MULTIPLE") {
                            return this.getI18nText("multiple");  // ← i18n in formatter
                        }
                        return sStepId ? `${sOperationActivity}/${sStepId}` : sOperationActivity;
                    }
                }
            });
    }
}
```

**Benefits:**
- ✅ Combine multiple fields into single display
- ✅ Complex formatting logic
- ✅ Conditional display based on data
- ✅ i18n support within formatter

---

## 25. Icon Cells with Conditional Formatting

**Pattern:** Display icons that change based on data

**Production Example:**
```javascript
_createCell(oColumnConfig) {
    case WorkInstructionTableWidget.Field.CurrentVersion:
        return new Icon({
            src: {
                path: oColumnConfig.field,
                formatter: (bCurrentVersion) => {
                    return bCurrentVersion ? "sap-icon://accept" : "sap-icon://decline";
                }
            },
            color: {
                path: oColumnConfig.field,
                formatter: (bCurrentVersion) => {
                    return bCurrentVersion ? IconColor.Positive : IconColor.Negative;
                }
            },
            tooltip: {
                path: oColumnConfig.field,
                formatter: (bCurrentVersion) => {
                    return bCurrentVersion ? this.getI18nText("yes") : this.getI18nText("no");
                }
            }
        });
}
```

**Status Icon Pattern:**
```javascript
new Icon({
    src: {
        path: "status",
        formatter: (sStatus) => {
            const iconMap = {
                "SUCCESS": "sap-icon://message-success",
                "WARNING": "sap-icon://message-warning",
                "ERROR": "sap-icon://message-error",
                "INFO": "sap-icon://message-information"
            };
            return iconMap[sStatus] || "sap-icon://question-mark";
        }
    },
    color: {
        path: "status",
        formatter: (sStatus) => {
            const colorMap = {
                "SUCCESS": IconColor.Positive,
                "WARNING": IconColor.Critical,
                "ERROR": IconColor.Negative,
                "INFO": IconColor.Neutral
            };
            return colorMap[sStatus] || IconColor.Default;
        }
    }
});
```

**Critical:**
- ✅ Same binding path for all properties (src, color, tooltip)
- ✅ Consistent logic across formatters
- ✅ Use IconColor enum for semantic colors
- ✅ Provide accessible tooltip

---

## 26. _createIdentifierCell() Helper

**Pattern:** Display composite identifier (e.g., "SFC/version", "operation/activity")

**Production Example:**
```javascript
case WorkInstructionTableWidget.Field.WorkInstructionVersion:
    return this._createIdentifierCell(oColumnConfig, "workInstruction}/{version");
    // ↑ Displays as "INSTR001/v3"
```

**Usage:**
```javascript
_createCell(oColumnConfig) {
    case MyWidget.Field.SfcVersion:
        // Binds to "sfc" and "version", displays as "SFC123/v2"
        return this._createIdentifierCell(oColumnConfig, "sfc}/{version");
        
    case MyWidget.Field.OperationActivity:
        return this._createIdentifierCell(oColumnConfig, "operation}/{activity");
}
```

**Pattern:** Uses `}/{` as separator in binding path to create composite display.

---

## 27. Early Exit Optimization

**Pattern:** Check if operation needed before doing expensive work

**Production Example:**
```javascript
syncTableSelectionWithPodContext() {
    const oSelectedWorkInstruction = PodContext.getSelectedWorkInstruction();
    const oTable = this.getTable();
    const oSelectedListItem = oTable.getSelectedItem();
    
    // ✅ Early exit - already in sync, no action needed
    if (oSelectedListItem && 
        oSelectedListItem.getBindingContext().getObject() === oSelectedWorkInstruction) {
        return;
    }
    
    // ... expensive sync logic only if needed
}
```

**Benefits:**
- ✅ Reduces unnecessary DOM operations
- ✅ Prevents infinite loops in bidirectional sync
- ✅ Improves performance
- ✅ Reduces flicker/re-render

**Use Cases:**
- Sync operations (already in sync? exit)
- Validation (already valid? exit)
- Data refresh (data unchanged? exit)
- State updates (state same? exit)

---

# Enterprise Production Patterns

Advanced patterns for enterprise-scale deployments.

## 28. MessageHistory Complete API

```javascript
// Persistent messages (appear in message popover)
MessageHistory.showSuccess("msg");
MessageHistory.showError("msg");

// Temporary toast (auto-dismiss)
MessageHistory.toast("msg");
MessageHistory.toast({ 
    type: MessageType.Error, 
    message: "msg" 
});

// Push to message history
MessageHistory.push({
    message: "msg",
    type: MessageHistory.Error  // or Success, Warning, Info
});
```

**When to Use:**
- `showSuccess()` - User actions succeeded (persistent)
- `showError()` - User actions failed (persistent)
- `toast()` - Temporary notifications (auto-dismiss)
- `push()` - Add to message history manually

---

## 29. DateTimeUtils

```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

// In binding formatter
new Text({
    text: {
        path: "date",
        formatter: (oValue) => DateTimeUtils.localeDateTime(oValue)
    }
})

// Direct use
const sFormatted = DateTimeUtils.localeDateTime(oDate);
```

**Methods:**
- `localeDateTime(oDate)` - Format date/time per user locale
- `localeDate(oDate)` - Format date only
- `localeTime(oDate)` - Format time only

---

## 30. Busy Indicator Pattern

```javascript
async _refresh() {
    const oView = this.getView();
    oView.setBusy(true);
    try {
        await ApiClient.getData(oRequest);
    } finally {
        oView.setBusy(false);  // ← Always in finally
    }
}
```

**Critical:**
- ✅ Always use try/finally
- ✅ Set busy before async operation
- ✅ Clear busy in finally block (even if error)
- ✅ Apply to correct control (view, table, dialog)

---

## 31. Expression Binding in getDefaultConfig()

**Pattern:** Combine static i18n text with dynamic model values

**Production Example:**
```javascript
static getDefaultConfig() {
    return {
        properties: {
            ...super.getDefaultConfig().properties,
            // Result: "Work Instructions (3)" where count updates reactively
            headerText: `{i18n>WorkInstructionTableWidget.headerText} ({${ModelPath.WorkInstructions}/length})`
        }
    };
}
```

**More Examples:**
```javascript
static getDefaultConfig() {
    return {
        properties: {
            // Display: "My Items (5)" where 5 updates automatically
            headerText: `{i18n>title} ({${ModelPath.Items}/length})`,
            
            // Display: "Plant: 1000"
            title: `{i18n>plant.label}: {${ModelPath.Plant}}`,
            
            // Conditional visibility
            visible: `{= \${${ModelPath.Items}/length} > 0 }`,
            
            // Conditional text
            text: `{= \${status} === 'ACTIVE' ? 'Running' : 'Stopped' }`
        }
    };
}
```

**Critical Rules:**
- ✅ Expression binding works in getDefaultConfig() properties
- ✅ Template strings with `${}` work for concatenation
- ✅ i18n bindings work: `{i18n>key}`
- ✅ ModelPath bindings work: `{${ModelPath.Something}}`
- ❌ NO binding syntax in getProperties() WidgetProperty definitions

---

## 32. Library Destructuring for Enums

**Pattern:** Import library once, destructure needed enums

**Production Example:**
```javascript
import SapMLibrary from "sap/m/library";
import SapUiCoreLibrary from "sap/ui/core/library";

const { ListMode, ListType } = SapMLibrary;
const { Icon, IconColor, Priority, TextAlign } = SapUiCoreLibrary;

// Clean usage throughout class
new Icon({ color: IconColor.Positive })  // Not: SapUiCoreLibrary.IconColor.Positive
```

**vs. Verbose Alternative:**
```javascript
// ❌ VERBOSE - Repeat library name every time
import SapMLibrary from "sap/m/library";

new Button({ type: SapMLibrary.ButtonType.Emphasized });
```

**Common Library Imports:**

| Library | Common Enums |
|---------|-------------|
| `sap/m/library` | ListMode, ListType, MessageType, ButtonType, InputType, FlexAlignItems, FlexJustifyContent |
| `sap/ui/core/library` | IconColor, Priority, TextAlign, ValueState, TitleLevel, MessageType |

---

## 33. flatMap for Nested Array Processing

**Pattern:** Process nested arrays in one step instead of nested loops

**Production Example:**
```javascript
_updateText() {
    const aWorkInstructions = PodContext.getWorkInstructions();
    if (!aWorkInstructions?.length) {
        oText.setText("");
        return;
    }
    
    // ✅ flatMap flattens nested arrays in one operation
    const aHeaderTextElements = aWorkInstructions
        .flatMap((oWorkInstruction) => oWorkInstruction.workInstructionElements)
        .filter((oElement) => oElement.type === WorkInstructionType.HEADER_TEXT);
    
    const sText = aHeaderTextElements.map((oElement) => oElement.text).join("\n");
    oText.setText(sText);
}
```

**Data Structure:**
```
[
  { name: "WI1", elements: [{text: "A"}, {text: "B"}] },
  { name: "WI2", elements: [{text: "C"}] }
]

Want: ["A", "B", "C"]
```

**Comparison:**

❌ **OLD WAY (nested loops):**
```javascript
const allElements = [];
for (const wi of workInstructions) {
    for (const element of wi.elements) {
        allElements.push(element);
    }
}
```

✅ **NEW WAY (flatMap):**
```javascript
const allElements = workInstructions.flatMap(wi => wi.elements);
```

**With Chaining:**
```javascript
// Get all HEADER_TEXT elements from all work instructions
const headerTexts = workInstructions
    .flatMap(wi => wi.workInstructionElements)  // Flatten
    .filter(el => el.type === WorkInstructionType.HEADER_TEXT)  // Filter
    .map(el => el.text)  // Extract
    .join("\n");  // Combine
```

**Benefits:**
- ✅ More readable than nested loops
- ✅ Chainable with filter/map/reduce
- ✅ Flattens one level automatically
- ✅ Modern JavaScript (ES2019+)

---

## 34. Utility Methods Pattern

**Pattern:** Extract repetitive view creation logic into helper methods

**Production Example:**
```javascript
_createFormContainer(aConfig) {
    return new FormContainer({
        formElements: aConfig.map(c => 
            this._createFormElement(c.label, c.text)
        )
    });
}

_createFormElement(sLabel, vText) {
    return new FormElement({
        label: new Label({ text: sLabel }),
        fields: [new Text({ text: vText })]
    });
}
```

**When to Use:**
- ✅ Code repeated 3+ times
- ✅ Clear, single responsibility
- ✅ Improves readability

---

## 35. Dynamic Table Binding with Conditional Sorting

**Pattern:** Apply sorting only when needed

**Production Example:**
```javascript
async _loadData() {
    const oData = await ApiClient.getData();
    this._oModel.setData(oData);
    
    const oBindingInfo = {
        path: "/items",
        template: this._createItemTemplate()
    };
    
    // Only sort if multiple items
    if (oData.items.length > 1) {
        oBindingInfo.sorter = new Sorter("category");
    }
    
    this._oTable.bindItems(oBindingInfo);
}
```

**Benefits:**
- ✅ Conditional sorting based on data
- ✅ Performance optimization
- ✅ Flexible binding configuration

---

## Pattern Summary Tables

### Core Foundation Patterns (Must Know)

| # | Pattern | Impact | Mandatory |
|---|---------|--------|-----------|
| 1 | JSDoc Documentation | High | ✅ |
| 2 | Private Fields | High | ✅ |
| 3 | Logger Pattern | High | ✅ |
| 4 | Object.freeze Enums | High | ✅ |
| 5 | Property Spreading | High | ✅ |
| 6 | Design/Run Mode | High | ✅ |
| 7 | Subscription Pattern | **Critical** | ✅ |
| 8 | Error Handling | High | ✅ |
| 9 | Delegate Patterns | High | Recommended |
| 10 | EXCLUDE_PROPERTIES | Medium | As needed |

### Advanced Implementation Patterns

| # | Pattern | Impact | Use Case |
|---|---------|--------|----------|
| 11 | Specialized Base Classes | High | Check SAP widgets first |
| 12 | Multi-Part Bindings | High | Computed display values |
| 13 | State Caching | High | Performance optimization |
| 14 | Design Mode Mock Data | High | Designer support |
| 15 | Primary/Fallback | High | UOM handling |
| 16 | instanceof Type Safety | High | Type-safe property access |
| 17 | PodContext Getters | Medium | Cleaner code |
| 18 | JSDoc Type Casting | Medium | IDE support |

### Widget-Specific Patterns

| # | Pattern | Impact | Widget Type |
|---|---------|--------|-------------|
| 19 | CustomPanel/VBox | **Critical** | Panel widgets |
| 20 | ContentHandler | High | Dialogs |
| 21 | ImageWidget Error Handling | High | Image widgets |
| 22 | Bidirectional Sync | High | Table widgets |
| 23 | Custom Binding Info | Medium | Table widgets |
| 24 | Composite Bindings | Medium | Table cells |
| 25 | Icon Cells | Medium | Table cells |
| 26 | _createIdentifierCell | Low | Table cells |
| 27 | Early Exit | High | Performance |

### Enterprise Production Patterns

| # | Pattern | Impact | Use Case |
|---|---------|--------|----------|
| 28 | MessageHistory API | High | User notifications |
| 29 | DateTimeUtils | Medium | Date formatting |
| 30 | Busy Indicators | High | ALL async operations |
| 31 | Expression Binding | Medium | Dynamic config |
| 32 | Library Destructuring | Low | Cleaner code |
| 33 | flatMap | Low | Nested arrays |
| 34 | Utility Methods | Low | Code reuse |
| 35 | Dynamic Binding | Medium | Conditional sorting |

---

## Critical Validation Checklist

Before deploying any widget, verify:

### Subscriptions
- [ ] All subscriptions wrapped in `PodContext.isRunMode()` check
- [ ] All subscriptions have corresponding unsubscribe in `onExit()`
- [ ] Context binding (`this`) passed as third parameter

### Memory Management
- [ ] Dialogs have `afterClose: () => oDialog.destroy()`
- [ ] No subscriptions without unsubscribe
- [ ] No event handlers without detach

### Type Safety
- [ ] instanceof checks before accessing type-specific properties
- [ ] Defensive array checks (`Array.isArray() && length !== 0`)
- [ ] JSDoc type casts for getView() and binding contexts

### Design Mode Support
- [ ] Mock data provided for design mode
- [ ] No API calls in design mode
- [ ] Subscriptions only in run mode

### Error Handling
- [ ] Try-catch around all API calls
- [ ] Errors logged with Logger
- [ ] User-facing errors shown via MessageHistory
- [ ] Busy indicators in finally blocks

### Performance
- [ ] State caching to avoid redundant API calls
- [ ] Early exit checks in frequently-called methods
- [ ] Multi-part bindings for computed values

---

## When to Use Each Pattern

| Scenario | Pattern to Use |
|----------|---------------|
| **Progress bar with %** | ProgressIndicatorWidget + Multi-Part Binding |
| **Display image** | ImageWidget + Error Handling |
| **Long text** | ExpandableTextWidget |
| **Table widget** | TableWidget + Bidirectional Sync |
| **Dialog** | ContentHandler + Dialog Destroy |
| **Subscribe to context** | Subscription Pattern + onExit Unsubscribe |
| **Computed display** | Multi-Part Binding |
| **API calls** | Error Handling + Busy Indicator |
| **Multiple UOMs** | Primary/Fallback Pattern |
| **Type-specific logic** | instanceof Type Safety |
| **Panel layout** | CustomPanel + CustomVBox |
| **Date display** | DateTimeUtils |
| **Status icons** | Icon Cells + Conditional Formatting |

---

## Production Code Quality Summary

### What Production SAP Code Does Well

1. ✅ **Consistent private fields** (`#`) - Modern ES2022 syntax
2. ✅ **Comprehensive logging** - Debug, info, error at appropriate levels
3. ✅ **Type safety** - instanceof checks everywhere
4. ✅ **Performance optimization** - State caching to avoid redundant calls
5. ✅ **Error handling** - Try-catch with logging and user notification
6. ✅ **Design mode support** - Placeholder/mock data for designer
7. ✅ **Clean code** - Small, focused methods with clear responsibilities
8. ✅ **JSDoc comments** - All methods documented
9. ✅ **Memory management** - Proper cleanup in onExit()
10. ✅ **Specialized widgets** - Leverage SAP base classes

### Critical Rules (Never Violate)

1. ⚠️ **ALWAYS unsubscribe** - Memory leaks are production incidents
2. ⚠️ **Check isRunMode()** - Before ALL subscriptions
3. ⚠️ **Destroy dialogs** - In afterClose callback
4. ⚠️ **Use CustomPanel** - For Designer drag-and-drop support
5. ⚠️ **Type-safe access** - instanceof before type-specific properties
6. ⚠️ **Busy indicators** - For ALL async operations
7. ⚠️ **Error handling** - Try-catch around ALL API calls
8. ⚠️ **Defensive arrays** - Check isArray && length before access

---

## Document Metadata

**Consolidation Stats:**
- Original files: 3 (2,113 total lines)
- Unified file: 1 (1,396 lines)
- Reduction: 717 lines (34% elimination)
- Unique patterns preserved: 35
- Production widgets analyzed: 15+

**Files Consolidated:**
1. `/c/Users/I331794/.claude/skills/pod-plugin/references/production-patterns-sap.md` (597 lines)
2. `/c/Users/I331794/.claude/skills/pod-plugin/references/production-patterns-deep-dive.md` (855 lines)
3. `/c/Users/I331794/.claude/skills/pod-plugin/references/production-patterns-wi.md` (661 lines)

**Key Achievements:**
- Eliminated 60-70% redundancy in core patterns
- Preserved ALL unique patterns
- Organized by complexity (Foundation → Advanced → Specific → Enterprise)
- Added cross-references and decision tables
- Maintained all code examples and production references
- Enhanced with validation checklist and usage guidelines

---

**See Also**:
- [advanced-patterns.md](advanced-patterns.md) - Complex UI patterns
- [common-mistakes.md](common-mistakes.md) - 28 mistakes with fixes
- [widget-patterns.md](widget-patterns.md) - Widget templates
- [delegate-architecture.md](delegate-architecture.md) - Delegate patterns
