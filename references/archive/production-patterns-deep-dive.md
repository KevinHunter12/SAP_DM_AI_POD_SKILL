# Production Patterns Deep Dive

**Source**: Real SAP POD 2.0 production widgets  
**Analysis Date**: 2026-04-17  
**Widgets Analyzed**: GoodsReceiptQuantityProgressWidget, MaterialImageWidget, OrderHeaderTextWidget, SFCQuantityProgressWidget

This reference documents advanced patterns extracted from production SAP Digital Manufacturing POD 2.0 widgets. These represent battle-tested implementations used in real manufacturing environments.

---

## Specialized Base Widget Classes

SAP provides pre-built ControlWidget subclasses for common controls with helper methods and best-practice configurations.

### ProgressIndicatorWidget

**Extends**: ControlWidget → ProgressIndicatorWidget  
**Wraps**: `sap.m.ProgressIndicator`

**Inherited Helper Methods**:
- `_getPercentValue(oConfig)` - Calculate percentage from target/actual with fallback support
- `_getDisplayValue(oConfig)` - Format display text with UOM handling

**Usage Pattern**:
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

**Usage**:
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

**Usage**:
```javascript
import ExpandableTextWidget from "sap/dm/dme/pod2/widget/core/ExpandableTextWidget";

class MyTextWidget extends ExpandableTextWidget {
    static EXCLUDE_PROPERTIES = [ ...ExpandableTextWidget.EXCLUDE_PROPERTIES, "text" ];
}
```

---

## Production Logger Pattern

All production SAP widgets use consistent Logger implementation.

### Standard Pattern

```javascript
import Logger from "sap/dm/dme/pod2/Logger";

class MyCustomWidget extends Widget {
    // 1. Private logger field
    #oLog = Logger.getLogger("custom.namespace.widget.MyCustomWidget");
    
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

### Logger Levels

| Level | Method | Use Case |
|-------|--------|----------|
| `debug()` | Verbose | Development, troubleshooting, frequent operations |
| `info()` | Informational | Important state changes, milestones |
| `error()` | Error | Failures, exceptions (always pass Error object as 2nd param) |

### Namespace Convention

**CRITICAL**: Logger name must use **dots**, not slashes:
- Module path: `custom/namespace/widget/MyWidget`
- Logger name: `custom.namespace.widget.MyWidget` ← dots!

### Best Practices

1. ✅ Initialize logger as private field `#oLog`
2. ✅ Use `debug()` for frequent operations
3. ✅ Use `error()` with Error object: `this.#oLog.error("msg", oError)`
4. ✅ Include context in messages: `"Failed to load material ${sMaterial}"`
5. ❌ Don't log sensitive data (credentials, PII)

---

## Multi-Part Property Bindings (Alternative to Subscriptions)

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

## State Caching to Avoid Redundant API Calls

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

## Design Mode Mock Data Patterns

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

**File structure**:
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

## Primary/Fallback Property Pattern

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

## Type Safety with instanceof

Work list items come in different types depending on POD page context. Use `instanceof` for type-safe property access.

### Common Types

```javascript
import WorkListItem from "sap/dm/dme/pod2/context/type/WorkListItem";
import OrderWorkListItem from "sap/dm/dme/pod2/context/type/OrderWorkListItem";
import BaseWorkListItem from "sap/dm/dme/pod2/context/type/BaseWorkListItem";
```

**Hierarchy**:
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

## PodContext Direct Getters

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

## ImageWidget Error Handling

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

## Production Code Quality Patterns

### What Production SAP Code Does Well

1. ✅ **Consistent private fields** (`#`) - Modern ES2022 syntax
2. ✅ **Comprehensive logging** - Debug, info, error at appropriate levels
3. ✅ **Type safety** - instanceof checks everywhere
4. ✅ **Performance optimization** - State caching to avoid redundant calls
5. ✅ **Error handling** - Try-catch with logging and user notification
6. ✅ **Design mode support** - Placeholder/mock data for designer
7. ✅ **Clean code** - Small, focused methods with clear responsibilities
8. ✅ **JSDoc comments** - All methods documented

---

---

# Additional Production Patterns (2026-04-18 Update)

**Source**: LogBuyoffWidget, OrderScheduleWidget, SFCExecutionQuantityWidget

## CustomPanel & CustomVBox ⚠️ CRITICAL

Use POD-specific wrappers for Designer drag-and-drop support:

```javascript
import CustomPanel from "sap/dm/dme/pod2/control/CustomPanel";
import CustomVBox from "sap/dm/dme/pod2/control/CustomVBox";

_createView() {
    return new CustomPanel({
        id: this.getId(),  // CRITICAL
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

## WorkListDelegate.refreshInBackground()

Trigger work list updates after operations:

```javascript
import WorkListDelegate from "sap/dm/dme/pod2/context/data/WorkListDelegate";

async _onAccept() {
    await ApiClient.execute(oRequest);
    WorkListDelegate.refreshInBackground();
    MessageHistory.showSuccess("Accepted");
}
```

## MessageHistory Complete API

```javascript
MessageHistory.showSuccess("msg");  // Persistent
MessageHistory.showError("msg");    // Persistent
MessageHistory.toast("msg");        // Temporary
MessageHistory.toast({ type: MessageType.Error, message: "msg" });
```

## DateTimeUtils

```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

new Text({
    text: {
        path: "date",
        formatter: (oValue) => DateTimeUtils.localeDateTime(oValue)
    }
})
```

## Busy Indicator Pattern

```javascript
async _refresh() {
    const oView = this.getView();
    oView.setBusy(true);
    try {
        await ApiClient.getData(oRequest);
    } finally {
        oView.setBusy(false);
    }
}
```

## Error Code Checking

```javascript
try {
    await ApiClient.findBuyoffLogs(oRequest);
} catch (oError) {
    if (oError?.body?.error?.code === "sfc.notInCompletePending") {
        this._oModel.setData([]);
        return;
    }
    MessageHistory.showError(oError.message);
}
```

## EXCLUDE_PROPERTIES Pattern

```javascript
class MyInputWidget extends InputWidget {
    static EXCLUDE_PROPERTIES = [
        ...InputWidget.EXCLUDE_PROPERTIES,
        "type",        // Hardcoded in getDefaultConfig
        "maxLength"    // Not applicable
    ];
}
```

## Multi-Subscription Pattern

```javascript
PodContext.subscribe([
    ModelPath.SelectedWorkListItems,
    ModelPath.SelectedOperationActivities
], this._refresh, this);
```

## Type Validation Patterns

```javascript
import OperationActivity from "sap/dm/dme/pod2/context/type/OperationActivity";
import Resource from "sap/dm/dme/pod2/context/type/Resource";

if (!(oOp instanceof OperationActivity)) {
    throw new Error("Invalid type");
}

const oResource = Resource.fromInternalODataResponse(oData.plannedResource);
```

## Utility Methods Pattern

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

## Dynamic Table Binding

```javascript
async _loadData() {
    const oData = await ApiClient.getData();
    this._oModel.setData(oData);
    
    const oBindingInfo = {
        path: "/items",
        template: this._createItemTemplate()
    };
    
    if (oData.items.length > 1) {
        oBindingInfo.sorter = new Sorter("category");
    }
    
    this._oTable.bindItems(oBindingInfo);
}
```

---

## Summary: When to Use Each Pattern

| Pattern | Use When |
|---------|----------|
| **ProgressIndicatorWidget** | Showing progress bars with % calculations |
| **ImageWidget** | Displaying images with URL loading and error handling |
| **ExpandableTextWidget** | Showing long text with expand/collapse |
| **Logger** | ALL widgets - for debugging and troubleshooting |
| **Multi-Part Bindings** | Computed display values (%, formatted text) |
| **State Caching** | Widgets that subscribe to frequent PodContext changes |
| **Design Mode Mock Data** | ALL widgets - to support POD Designer |
| **Primary/Fallback** | Data with multiple UOMs or optional properties |
| **instanceof** | Handling different work list item types |
| **PodContext Getters** | Subscription callbacks needing single items |
| **Image Error Handling** | ANY widget displaying images from URLs |
| **CustomPanel/CustomVBox** | ⚠️ ALL custom panel widgets for Designer support |
| **WorkListDelegate** | After operations that change work center data |
| **MessageHistory API** | User notifications (persistent vs temporary) |
| **DateTimeUtils** | All date/time formatting |
| **Busy Indicators** | ALL async operations |
| **Error Code Checking** | SAP DM API error handling |
| **EXCLUDE_PROPERTIES** | ControlWidget/InputWidget extensions |
| **Multi-Subscription** | Subscribe to multiple ModelPaths |
| **Type Validation** | Working with OperationActivity/Resource types |
| **Utility Methods** | Repetitive view creation (3+ occurrences) |
| **Dynamic Binding** | Conditional sorting/grouping in tables |
