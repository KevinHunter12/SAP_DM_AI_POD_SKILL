# Production Widget Patterns (WorkInstruction Analysis)

**Source:** WorkInstructionHeaderTextWidget.js, WorkInstructionTableWidget.js  
**Analysis Date:** 2026-04-18  
**Location:** C:\VSCodeProjects\v2\workinstruction  
**Status:** Real production SAP Digital Manufacturing code

---

## Pattern #1: Specialized Base Widget Classes

**Current Hierarchy Extended:**
```
Widget (abstract)
├── ControlWidget
│   ├── ButtonWidget, InputWidget, TextWidget
│   └── ExpandableTextWidget ← Specialized with expand/collapse
│       └── WorkInstructionHeaderTextWidget (production)
├── LayoutWidget
├── TableWidget
│   └── WorkInstructionTableWidget (production)
└── ContentHandler
```

**When to use:** Check SAP-provided specialized classes before extending base tiers.

**Example:**
```javascript
import ExpandableTextWidget from "sap/m/ExpandableText";

class WorkInstructionHeaderTextWidget extends ExpandableTextWidget {
    // Inherits expand/collapse behavior automatically
}
```

---

## Pattern #2: EXCLUDE_PROPERTIES for Property Editor Control

**Purpose:** Hide properties from POD Designer property panel (but still usable in code).

**Pattern:**
```javascript
class MyWidget extends SomeBaseWidget {
    // Hide "text" and "value" from property editor
    static EXCLUDE_PROPERTIES = [ ...SomeBaseWidget.EXCLUDE_PROPERTIES, "text", "value" ];
    
    // Properties won't appear in POD Designer but can be set programmatically
}
```

**Production Example:**
```javascript
// WorkInstructionHeaderTextWidget.js:49
static EXCLUDE_PROPERTIES = [ ...ExpandableTextWidget.EXCLUDE_PROPERTIES, "text" ];
// ↑ "text" set dynamically from work instructions, not user-configurable
```

**Critical:**
- ✅ Always spread parent EXCLUDE_PROPERTIES
- ✅ Document WHY each property excluded (comment)
- ❌ Don't exclude properties users need to configure

---

## Pattern #3: Data Delegate Pattern (Shared Data Loading)

**Problem:** Multiple widgets need same data  
**Solution:** Centralized Delegate loads once, broadcasts to all subscribers

**Architecture:**
```
WorkInstructionDelegate (singleton)
    ↓ refresh() loads once
PodContext.WorkInstructions
    ↓ subscribe()
WorkInstructionTableWidget, WorkInstructionHeaderTextWidget
```

**Implementation:**
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

---

## Pattern #4: Bidirectional Sync (Table Selection ↔ PodContext)

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

## Pattern #5: Custom _getItemBindingInfo() with Filters

**Pattern:** Override to add custom client-side filters

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:134-148
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

## Pattern #6: Composite Bindings (Multiple Fields in One Cell)

**Pattern:** Combine multiple model properties into one display value

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:209-221
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

## Pattern #7: Icon Cells with Conditional Formatting

**Pattern:** Display icons that change based on data

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:230-249
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

## Pattern #8: _createIdentifierCell() Helper

**Pattern:** Display composite identifier (e.g., "SFC/version", "operation/activity")

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:205
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

## Pattern #9: Early Exit Optimization

**Pattern:** Check if operation needed before doing expensive work

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:176
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

## Pattern #10: JSDoc Type Casting for IDE Support

**Pattern:** Use JSDoc annotations for better IDE support and type safety

**Production Examples:**
```javascript
// Line 75 - Cast getView() return type
const oText = /** @type {sap.m.ExpandableText} */ (this.getView());

// Line 186 - Cast getObject() return type
const oObject = /** @type {sap.dm.dme.pod2.context.type.WorkInstruction} */
    (oListItem.getBindingContext().getObject());

// Line 267 - Cast event type
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

## Pattern #11: Expression Binding in getDefaultConfig()

**Pattern:** Combine static i18n text with dynamic model values

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:65
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

## Pattern #12: Library Destructuring for Enums

**Pattern:** Import library once, destructure needed enums

**Production Example:**
```javascript
// WorkInstructionTableWidget.js:24-26
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

## Pattern #13: flatMap for Nested Array Processing

**Pattern:** Process nested arrays in one step instead of nested loops

**Production Example:**
```javascript
// WorkInstructionHeaderTextWidget.js:82
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

## 🚨 Critical Issue Found: Missing onExit()

**File:** WorkInstructionHeaderTextWidget.js  
**Line:** 55-68

**Problem:**
```javascript
// ❌ PRODUCTION CODE ISSUE - Memory leak potential
onInit() {
    super.onInit();
    this._updateText();
    PodContext.subscribe(ModelPath.WorkInstructions, this._updateText, this);  // ← Subscribes
    if (!PodContext.getWorkInstructions()) {
        WorkInstructionDelegate.refresh();
    }
}
// ❌ Missing onExit() - should unsubscribe!
```

**Should be:**
```javascript
onExit() {
    super.onExit();
    PodContext.unsubscribe(ModelPath.WorkInstructions, this._updateText, this);
}
```

**Validation Rule:**
If onInit() has subscriptions → onExit() must unsubscribe.

---

## Pattern Summary Table

| # | Pattern | Impact | Use Case |
|---|---------|--------|----------|
| 1 | Specialized Base Classes | High | Check SAP-provided widgets first |
| 2 | EXCLUDE_PROPERTIES | Medium | Hide properties from Designer |
| 3 | Data Delegate | High | Shared data loading across widgets |
| 4 | Bidirectional Sync | High | Table selection ↔ PodContext sync |
| 5 | Custom _getItemBindingInfo() | Medium | Client-side table filtering |
| 6 | Composite Bindings | Medium | Multi-field cell display |
| 7 | Icon Cells | Medium | Conditional icon/color display |
| 8 | _createIdentifierCell() | Low | Composite identifier display |
| 9 | Early Exit Optimization | High | Performance & loop prevention |
| 10 | JSDoc Type Casting | Medium | IDE support & type safety |
| 11 | Expression Binding | Medium | Dynamic config properties |
| 12 | Library Destructuring | Low | Cleaner code |
| 13 | flatMap | Low | Nested array processing |

---

## Next Steps

1. Add these patterns to main SKILL.md selectively (high priority only)
2. Update common-mistakes.md with missing onExit() issue
3. Add pre-generation validation checklist for subscriptions
4. Test patterns in new widget generation

**Production Code Confidence:** High - All patterns extracted from real, deployed SAP code.
