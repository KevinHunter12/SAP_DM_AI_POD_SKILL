# POD 2.0 Quick Reference Card

Essential patterns and gotchas for POD 2.0 plugin development.

---

## ⭐ MOST COMMON PATTERN: Getting Selected Data (ADAPTIVE)

**90% of POD plugins use this pattern to get selected SFC, operation, and resource:**

**⚠️ CRITICAL: Must be ADAPTIVE to work in different POD configurations!**

```javascript
// ✅ CORRECT - Adaptive pattern (works in ALL POD configurations)
async _onButtonPress() {
    const aFilterResources = PodContext.getFilterResources();
    const sResource = aFilterResources?.[0]?.resource || null;

    // Try both widget types
    const oOp = PodContext.getLastSelectedOperationActivity();
    const oWL = PodContext.getLastSelectedWorkListItem();

    let sSfc, sOperation;

    if (oOp && oWL) {
        // Pattern 1: Both OperationActivity + WorkList present
        sSfc = oWL.sfc;
        sOperation = oOp.operationActivity;
    } else if (oWL) {
        // Pattern 2: WorkList only
        sSfc = oWL.sfc;
        sOperation = oWL.operationActivity;
    } else {
        // Pattern 3: Array fallback
        const aItems = PodContext.getSelectedWorkListItems();
        if (aItems?.length > 0) {
            sSfc = aItems[0].sfc;
            sOperation = aItems[0].operationActivity;
        } else {
            MessageHistory.showError("No selection");
            return;
        }
    }

    if (!sSfc || !sOperation) {
        MessageHistory.showError("Missing SFC or Operation");
        return;
    }

    const oRequest = {
        plant: PodContext.getPlant(),
        sfc: sSfc,
        operation: sOperation
    };

    if (sResource) oRequest.resource = sResource;

    await ApiClient.sfc.sfcStart(oRequest);
}
```

**Why Adaptive?** Different PODs have different widgets:
- Some have OperationActivity + WorkList
- Some have only WorkList
- Some have only OperationActivity
- Your plugin must work in ALL configurations!

**❌ Common Mistakes:**
```javascript
// ❌ DON'T assume OperationActivity widget exists
const oOp = PodContext.getLastSelectedOperationActivity();
const sOperation = oOp.operationActivity;  // 💥 Fails if no OperationActivity widget!

// ❌ DON'T use worklist for operation when OperationActivity exists
const sOperation = oWorkListItem.operationActivity;  // May be stale!

// ❌ DON'T use operation.resource (often null)
const sResource = oLastSelectedOperation.resource;

// ❌ DON'T call getResource() (doesn't exist!)
const sResource = PodContext.getResource();

// ❌ DON'T fetch data that's already in PodContext!
// WorkListItem ALREADY contains customValues - no API call needed!
const oWorkListItem = PodContext.getLastSelectedWorkListItem();
// oWorkListItem already has: sfc, material, order, customValues, etc.
// Just use: oWorkListItem.customValues directly!
```

---

## ⚠️ CRITICAL: Data Already in PodContext (DON'T FETCH WHAT YOU HAVE!)

**Before making ANY API call, check if the data is already in PodContext objects!**

### WorkListItem Object Structure

```javascript
const oWorkListItem = PodContext.getLastSelectedWorkListItem();
// Contains:
{
    sfc: "SFC_12345",
    material: "MATERIAL1",
    order: "SHOP_ORDER_001",
    workCenter: "WC-001",
    operationActivity: "OP10-ASSEMBLY",
    customFields: {                    // ← Custom data is HERE! (OBJECT, not array!)
        "SHOP_ORDER.BREWFATHER_FG": "0.9",
        "ITEM.SAPJAPAN": "Demo custom data",
        "ITEM.MATERIAL_CUSTOM": "Test value"
    },
    sfcStatusCode: "402",
    sfcQuantity: 10,
    // ... more fields (45+ properties total)
}
```

**⚠️ CRITICAL: `customFields` is an OBJECT (key-value pairs), NOT an array!**

**✅ CORRECT - Use data directly from PodContext:**
```javascript
_onWorkListSelectionChanged(aSelectedItems) {
    const oItem = aSelectedItems[0];
    
    // Extract custom data directly - NO API CALL NEEDED!
    // customFields is an OBJECT with key-value pairs
    if (oItem.customFields && typeof oItem.customFields === "object") {
        Object.keys(oItem.customFields).forEach(sKey => {
            const sValue = oItem.customFields[sKey];
            // Process: sKey = attribute name, sValue = value
            console.log(`${sKey}: ${sValue}`);
        });
    }
}
```

**❌ WRONG - Treating customFields as array:**
```javascript
// ❌ DON'T DO THIS - customFields is NOT an array!
if (Array.isArray(oItem.customFields)) {  // Always false!
    oItem.customFields.forEach(cf => { ... });  // Never executes!
}
```

**❌ WRONG - Fetching data you already have:**
```javascript
// ❌ DON'T DO THIS - you already have customFields!
const sSfc = oWorkListItem.sfc;
const oResponse = await fetch(`/sfcs?sfc=${sSfc}`);  // Unnecessary API call!
const oData = await oResponse.json();
// You already have the same data in oWorkListItem.customFields!
```

**When to Actually Call APIs:**
- Starting/completing operations (`ApiClient.sfc.sfcStart`, `sfcComplete`)
- Posting quantities (`ApiClient.production.reportQuantity`)
- Fetching data NOT in PodContext (e.g., material master, BOM, routing)
- Triggering backend actions (create NC, serialize, etc.)

**When NOT to Call APIs:**
- Displaying data already in worklist (`customFields` object, sfc, material, order, etc.)
- Getting current selection (use PodContext getters)
- Getting filtered resources/operations/workcenters (use PodContext.getFilter*)


---

## Import Paths

```javascript
// ✅ ALWAYS use context/ for PodContext
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";

// ✅ ALWAYS use context/data/ for delegates
import WorkListDelegate from "sap/dm/dme/pod2/context/data/WorkListDelegate";

// ✅ i18n model
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

// ✅ API client
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";

// ✅ Logger
import Logger from "sap/dm/dme/pod2/Logger";
```

### ⚠️ SAPUI5 Enum Types — Use Library Imports, NOT Pseudo-Modules

Importing enum/type constants as direct modules (e.g. `"sap/m/ButtonType"`) is **deprecated** and produces console warnings. Always import from the parent library instead:

```javascript
// ❌ WRONG — deprecated pseudo-module imports (cause console warnings)
"sap/m/ButtonType"
"sap/m/FlexAlignItems"
"sap/ui/core/ValueState"

// ✅ CORRECT — import library, extract types as vars
sap.ui.define([
    "sap/m/library",
    "sap/ui/core/library"
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
});
```

**Rule**: If the path is `sap/m/<Name>` or `sap/ui/core/<Name>` and it's an enum/constant (not a control like `Button`, `VBox`), use the library pattern above.

See [sapui5-control-apis.md - Deprecated Pseudo-Module Imports](sapui5-control-apis.md#deprecated-pseudo-module-imports) for the full replacement table.

---

## ModelPath Constants (ALL PLURAL)

```javascript
ModelPath.SelectedWorkListItems       // ✅ PLURAL
ModelPath.SelectedOperationActivities // ✅ PLURAL
ModelPath.FilterResources             // ✅ PLURAL
ModelPath.FilterOperationActivities   // ✅ PLURAL
ModelPath.LastSelectedWorkListItem    // ✅ Singular (getter only)
```

---

## PodContext Getters Reference

### For Single Selection (Most Common) - MUST BE ADAPTIVE

**⚠️ CRITICAL: Different PODs have different widgets. Your code must handle all configurations!**

| Method | Returns | May Return Null? | Use For |
|--------|---------|------------------|---------|
| `getLastSelectedOperationActivity()` | Single OperationActivity | ✅ YES (if no OperationActivity widget) | Get **operation** (preferred) |
| `getLastSelectedWorkListItem()` | Single WorkListItem | ✅ YES (if no WorkList widget) | Get **SFC** (required) |
| `getFilterResources()` | Array of Resources | ✅ YES (returns empty array) | Get **resource** |

**POD Configuration Examples:**

| Configuration | oOp | oWL | What to Do |
|---------------|-----|-----|------------|
| OperationActivity + WorkList | ✅ | ✅ | Use oOp for operation, oWL for SFC |
| WorkList Only | ❌ null | ✅ | Use oWL for both operation and SFC |
| OperationActivity Only | ✅ | ❌ null | Use oOp for both |
| Neither | ❌ null | ❌ null | Use array: `getSelectedWorkListItems()[0]` |

### For Multi-Selection

| Method | Returns | Use For |
|--------|---------|---------|
| `getSelectedOperationActivities()` | Array of OperationActivity | Multi-select operations |
| `getSelectedWorkListItems()` | Array of WorkListItem | Multi-select SFCs |

### Other Common Getters

| Method | Returns |
|--------|---------|
| `getPlant()` | Current plant (string) |
| `getUserId()` | Current user ID (string) |
| `getPodId()` | Current POD ID (string) |
| `getPlantTimeZone()` | IANA timezone string |
| `getIndustryType()` | `"DISCRETE"` or `"PROCESS"` |
| `isDiscreteIndustry()` | boolean |
| `isProcessIndustry()` | boolean |
| `getFilterWorkCenters()` | Array of work centers |
| `getFilterSfcs()` | Array of SFC strings (filter bar values) |
| `getFilterMaterials()` | Array of materials |
| `getFilterOperationActivities()` | Array of filtered operations |
| `getPodRuntime()` | PodRuntime — navigation, widget access |
| `getWhenAvailable(sModelPath)` | Promise resolving when value is defined |

---

## PodRuntime Quick Reference

Access via `PodContext.getPodRuntime()` or `this.getPodRuntime()`:

```javascript
const oRuntime = PodContext.getPodRuntime();

// Navigation
await oRuntime.navigateToPage("myPageId");
await oRuntime.navigateBack();
await oRuntime.navigateToWidget("myWidgetId");
await oRuntime.showDialog("myDialogId");

// Widget access
const oWidget = oRuntime.getWidget("widgetId");
oRuntime.forEachWidget(oWidget => { ... });
const oConfig = oRuntime.findWidgetConfig(c => c.type === "...");
```

---

## Object Structures

### OperationActivity Object

```javascript
{
    sfc: "SFC_12345",
    operationActivity: "OP10-ASSEMBLY",  // ← Use for operation!
    workCenter: "WC-001",
    stepId: "10",
    resource: null,  // Often null! Use getFilterResources()
    statusComplete: false,
    statusInWork: true
}
```

### WorkListItem Object

```javascript
{
    sfc: "SFC_12345",               // ← Use for SFC!
    material: "MATERIAL1",
    order: "SHOP_ORDER_001",
    workCenter: "WC-001",
    operationActivity: "OP10-ASSEMBLY",  // May be stale!
    sfcStatusCode: "402",
    sfcQuantity: 10
}
```

### Resource Object

```javascript
{
    resource: "RESOURCE_001",       // ← Use for resource!
    resourceType: "EQUIPMENT",
    description: "Assembly Station 1"
}
```

---

## Subscription Pattern

```javascript
async onInit() {
    await super.onInit();
    
    // Subscribe to selection changes
    PodContext.subscribe(
        ModelPath.SelectedWorkListItems,
        this._onSelectionChanged,
        this
    );
    
    // Initial load
    this._onSelectionChanged(PodContext.getSelectedWorkListItems());
}

_onSelectionChanged(aItems) {
    // React to changes
    if (Array.isArray(aItems) && aItems.length > 0) {
        this._updateUI(aItems);
    }
}

onExit() {
    // REQUIRED: Unsubscribe to prevent memory leaks!
    PodContext.unsubscribe(
        ModelPath.SelectedWorkListItems,
        this._onSelectionChanged,
        this
    );
    super.onExit();
}
```

---

## i18n Pattern

```javascript
// Static i18n model
static #oI18nModel = new I18nResourceModel({
    bundleName: "your.namespace.i18n.i18n"  // Dots, not slashes!
});

static getI18nModel() {
    return this.#oI18nModel;
}

// Use in _createView()
_createView() {
    return new Button({
        text: this.getI18nText("button.text")  // ✅ Method call
        // NOT: text: "{i18n>button.text}"     // ❌ Binding doesn't work!
    });
}
```

---

## Base Class Selection

| Need | Base Class | Pattern |
|------|------------|---------|
| Single control (Button, Input) | `ControlWidget` | Pass control class to `super(Button, oConfig)` |
| Container (VBox, HBox, Panel) | `LayoutWidget` | Pass container class to `super(VBox, oConfig)` |
| Data table | `TableWidget` | Pass Table class to `super(Table, oConfig)` |
| Custom complex layout | `Widget` | Create controls manually |

---

## ControlWidget Pattern

```javascript
import Button from "sap/m/Button";
import ControlWidget from "sap/dm/dme/pod2/widget/ControlWidget";

class MyButton extends ControlWidget {
    constructor(oConfig) {
        super(Button, oConfig);  // Creates button
    }
    
    _createView() {
        const oButton = super._createView();  // Get button
        
        // Configure it
        oButton.setText("Click Me");
        oButton.setWidth("200px");
        
        // Height via inline style (Button has no setHeight!)
        oButton.addEventDelegate({
            onAfterRendering: () => {
                oButton.getDomRef().style.height = "80px";
            }
        });
        
        return oButton;
    }
}
```

---

## SAPUI5 Control Styling

### Controls WITHOUT setHeight()
- Button, Text, Label, Input, Title, ComboBox, Icon

```javascript
// ❌ WRONG
oButton.setHeight("80px");  // TypeError!

// ✅ CORRECT
oButton.addEventDelegate({
    onAfterRendering: () => {
        oButton.getDomRef().style.height = "80px";
    }
});
```

### Controls WITH setHeight()
- VBox, HBox, Panel, Table

```javascript
// ✅ CORRECT
oPanel.setHeight("400px");
```

---

## extension.json Format

**ONLY `widgets` and `actions` arrays allowed!**

```json
{
  "widgets": [
    {
      "modulePath": "namespace/widget/WidgetName",
      "type": "namespace.widget.WidgetName"
    }
  ],
  "actions": []
}
```

**❌ NO other fields!** No `id`, `name`, `version`, `vendor`, `description`, `dependencies`, or `content`.

---

## Memory Leak Prevention

```javascript
// ✅ ALWAYS unsubscribe in onExit()
onInit() {
    PodContext.subscribe(ModelPath.X, this._handler, this);
}

onExit() {
    PodContext.unsubscribe(ModelPath.X, this._handler, this);
    // OR: PodContext.unsubscribeAll(this);
    super.onExit();
}
```

---

## Error Handling

```javascript
import MessageHistory from "sap/dm/dme/pod2/context/MessageHistory";

// Success message
MessageHistory.showSuccess(this.getI18nText("message.success"));

// Error message
MessageHistory.showError(this.getI18nText("error.failed"));

// Warning message
MessageHistory.showWarning(this.getI18nText("warning.partial"));

// Push to message popover (persistent)
MessageHistory.push({
    message: this.getI18nText("error.critical"),
    type: MessageHistory.Error
});
```

---

## API Client Pattern

**⚠️ CRITICAL: ApiClient is NOT a generic HTTP client!**

```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";

// ✅ CORRECT - Use pre-built methods
await ApiClient.sfc.sfcStart(oRequest);
await ApiClient.sfc.sfcComplete(oRequest);
await ApiClient.production.reportQuantity(oRequest);
await ApiClient.material.getMaterial(oRequest);

// ❌ WRONG - These methods DON'T exist!
await ApiClient.get(sUrl);     // ❌ No .get() method
await ApiClient.post(sUrl);    // ❌ No .post() method
await ApiClient.request(...);  // ❌ No .request() method
```

**If you need custom HTTP calls, use native `fetch()`:**

```javascript
// For custom endpoints not covered by ApiClient
const oContext = PodContext.getContext();
const sUrl = `${oContext.serviceRegistry.getApiUrl("sfc")}/custom-endpoint`;

const oResponse = await fetch(sUrl, {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${oContext.token}`
    },
    body: JSON.stringify(oPayload)
});

const oData = await oResponse.json();
```

**But FIRST - check if data is already in PodContext before making ANY API call!**

---

## When to Use ApiClient vs fetch() vs PodContext

| Scenario | Solution | Why |
|----------|----------|-----|
| Display custom data for selected SFC | Use `PodContext.getLastSelectedWorkListItem().customValues` | Data already loaded |
| Display SFC, material, order, quantity | Use `PodContext.getLastSelectedWorkListItem()` | Data already loaded |
| Get current operation | Use `PodContext.getLastSelectedOperationActivity()` | Data already loaded |
| Get filtered resources | Use `PodContext.getFilterResources()` | Data already loaded |
| Start/complete SFC | Use `ApiClient.sfc.sfcStart/sfcComplete()` | Action required |
| Post production quantity | Use `ApiClient.production.reportQuantity()` | Action required |
| Fetch material master (not in worklist) | Use `ApiClient.material.getMaterial()` | Data not in PodContext |
| Custom/internal endpoint | Use `fetch()` with manual token | ApiClient doesn't cover it |

**Golden Rule: If it's selection or filter data → PodContext. If it's an action or data not loaded → ApiClient or fetch().**

---

## API Client Methods (Not Exhaustive)

```javascript
// SFC operations
await ApiClient.sfc.sfcStart(oRequest);
await ApiClient.sfc.sfcComplete(oRequest);
await ApiClient.sfc.getSfcDetails(oRequest);  // Rarely needed - worklist has this!

// Production operations
await ApiClient.production.reportQuantity(oRequest);

// Material operations
await ApiClient.material.getMaterial(oRequest);

// Internal APIs (use with caution)
await ApiClient.internal.activityconfirmation.getSummaries(oRequest);
```

---

## ⚠️ API Payload Verification (CRITICAL!)

**MANDATORY**: Before calling ANY SAP DM API, verify the exact payload format in `sapdm-api-reference.md`.

### Common API Payload Mistakes

```javascript
// ❌ WRONG - Guessing payload format
await ApiClient.sfc.sfcStart({
    plant: PodContext.getPlant(),
    sfc: sSfc,                // ❌ API expects array format!
    operationActivity: sOperation // ❌ ApiClient expects "operation"!
});
// Result: 400 Bad Request

// ❌ ALSO WRONG - Wrong array format (objects instead of strings)
await ApiClient.sfc.sfcStart({
    sfcs: [{ sfc: sSfc }],    // ❌ API expects array of STRINGS!
    operationActivity: sOperation
});
// Result: 400 Bad Request: Cannot deserialize String from Object

// ✅ CORRECT - Actual ApiClient.sfc.sfcStart format
await ApiClient.sfc.sfcStart({
    plant: PodContext.getPlant(), // ✅ Required
    sfcs: [sSfc],                 // ✅ Array of strings
    operation: sOperation,        // ✅ "operation" (ApiClient differs from REST API!)
    resource: sResource,          // ✅ Optional
    autoAssembleEnabled: true     // ✅ Default true
});
```

**CRITICAL**: `ApiClient` methods use DIFFERENT property names than the REST API:
- REST API: `operationActivity` → ApiClient: `operation`
- Always verify by checking browser network traffic or real plugin usage!

### Pre-API-Call Checklist

Before writing ANY API call:

1. **STOP** - Don't write the code yet
2. **READ** `sapdm-api-reference.md` for the API endpoint
3. **VERIFY** exact payload structure:
   - Property names (e.g., `operationActivity` not `operation`)
   - Data types (array vs string, object vs primitive)
   - Required vs optional fields
4. **ONLY THEN** write the API call code

### Common APIs to Verify

| API | Verify These |
|-----|--------------|
| `sfcStart()` | `sfcs: ["SFC001"]` array of strings, `operation` (not `operationActivity`), `autoAssembleEnabled` |
| `sfcComplete()` | `sfcs: ["SFC001"]` array of strings, verify property names |
| `reportQuantity()` | Quantity structure, UOM requirements |
| `getMaterial()` | Plant + material + version parameters |

**CRITICAL**: `ApiClient` wrapper uses DIFFERENT property names than REST API docs. Always verify actual format via network traffic.

**Why**: Wrong payloads cause `400 Bad Request` errors that block functionality and require redeployment to fix.

---

## Logger Pattern

```javascript
import Logger from "sap/dm/dme/pod2/Logger";

class MyWidget extends Widget {
    #oLog = Logger.getLogger("namespace.widget.MyWidget");
    
    _someMethod() {
        this.#oLog.info("Info message");
        this.#oLog.warn("Warning message");
        this.#oLog.error("Error message", oError);
        this.#oLog.debug("Debug message", oData);
    }
}
```

---

## DateTimeUtils

```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

// Always use instead of new Date() when timezone matters
DateTimeUtils.now()                          // Current time in plant timezone → UI5Date
DateTimeUtils.startOfDay(oDate?)             // 00:00:00.000 in plant timezone
DateTimeUtils.endOfDay(oDate?)               // 23:59:59.999 in plant timezone
DateTimeUtils.fromODataDateString(s)         // Parse "/Date(timestamp)/" → Date | null
DateTimeUtils.isValidDate(oDate)             // → boolean

// Locale-aware display formatting (respects plant timezone)
DateTimeUtils.localeDateTime(vValue, oOpts?) // → string e.g. "Sep 7, 2026, 10:30 AM"
DateTimeUtils.localeDate(vValue, oOpts?)     // → string e.g. "Sep 7, 2026"
DateTimeUtils.localeTime(vValue, oOpts?)     // → string e.g. "10:30:00 AM"
// vValue: UI5Date, JS Date, or ISO string. Returns "" for invalid/null dates.

// Batch convert ISO strings ↔ Date objects in place (mutates object/array)
DateTimeUtils.parseDateProperties(oObj, "startDate", "endDate")
DateTimeUtils.encodeDateProperties(oObj, "startDate", "endDate")
```

---

## File Structure

```
<working-directory>/     # User is already in namespace
├── extension.json       # Widget registration
├── widget/
│   └── WidgetName.js
└── i18n/
    ├── i18n_en.properties  # English (always include)
    ├── i18n_de.properties  # German
    ├── i18n_zh.properties  # Chinese Simplified
    └── i18n_ja.properties  # Japanese (Core 4 languages)
```

**NEVER create namespace folders!** Generate files at working directory root.

---

## Quick Validation Checklist

Before generating plugin code:

- [ ] Read `widget-patterns.md` for widget template
- [ ] Read `common-mistakes.md` Mistake #14 for extension.json format
- [ ] Asked user for: namespace, plugin name, category, languages
- [ ] Using correct PodContext getters (getLastSelected*, getFilter*)
- [ ] Import paths use `context/` not `model/`
- [ ] Delegate paths use `context/data/` not `delegate/`
- [ ] ModelPath constants are PLURAL
- [ ] i18n uses method calls, not bindings
- [ ] onExit() unsubscribes if onInit() subscribes
- [ ] **Enum types use `sap/m/library` / `sap/ui/core/library`** — NOT pseudo-modules like `"sap/m/ButtonType"`, `"sap/m/FlexAlignItems"`, `"sap/ui/core/ValueState"` (see [sapui5-control-apis.md](sapui5-control-apis.md#deprecated-pseudo-module-imports))
- [ ] **IF USING APIs**: Read `sapdm-api-reference.md` and verified payload format

---

## ⚠️ Embedded HTML/CSS Widget Gotchas (READ IF USING `sap.ui.core.HTML`)

If your widget embeds raw HTML/CSS via `sap.ui.core.HTML` (pixel-perfect mockups, custom visualizations, third-party embeds), THREE blocking errors will hit you. See `common-mistakes.md` Mistakes #30, #31, #32 for full details.

### Gotcha 1: NO `#` Characters Anywhere in `.js` File

JSTokenizer pre-scans every widget source file and crashes on `#` — even inside strings.

```javascript
// ❌ JSTokenizer crash: "Unexpected '#'"
class W extends Widget {
    static #oModel = new I18nResourceModel({...});  // 💥 private field
}

_getHtml() {
    return '<style>.box { color: #fff; }</style>';  // 💥 hex color in string
}

// ✅ CORRECT
var _oModel = new I18nResourceModel({...});  // module-scoped var

_getHtml() {
    return '<style>.box { color: rgb(255,255,255); }</style>';  // rgb() instead
}
```

**Pre-deploy check**: `grep -c '#' widget/YourWidget.js` MUST be `0`.

### Gotcha 2: Use ES6 `class extends Widget`, NOT `Widget.extend()`

POD 2.0 `Widget` is an ES6 class — no classic UI5 `.extend()` static method.

```javascript
// ❌ "Widget.extend is not a function"
var W = Widget.extend("ns.W", { constructor: function(c) {...} });

// ✅ CORRECT
class W extends Widget {
    constructor(oConfig) { super(oConfig); }
    static getDisplayName() { return "..."; }
}
return W;
```

### Gotcha 3: `{` in Constructor Property Triggers BindingParser

`ManagedObject` constructor runs every property value through `BindingParser`. If your value contains `{` (e.g. CSS rules `.box { ... }`), UI5 tries to parse it as a binding expression and crashes.

```javascript
// ❌ "JSTokenizer: Unexpected 'r'" (or any char after '{')
// Stack trace contains BindingParser → BindingInfo → ManagedObject constructor
new HTML(id, { content: sHtml });  // 💥 sHtml has '{' from CSS

// ✅ CORRECT — setters bypass binding parsing
var oHtml = new HTML(id);
oHtml.setSanitizeContent(false);
oHtml.setContent(sHtml);  // ✅ Raw value, no parsing
```

**Diagnostic**: If JSTokenizer error stack includes `BindingParser` or `BindingInfo`, it's this gotcha — NOT a source-file syntax issue.

### Combined Pre-Deploy Checklist for HTML-Embedding Widgets

```bash
# MUST be 0 — no double-quoted JS string lines containing #
python3 -c "
content = open('widget/YourWidget.js').read()
bad = [i+1 for i, l in enumerate(content.splitlines())
       if '#' in l and l.strip().startswith('\"') and not l.strip().startswith('//')]
print('Double-quoted # lines (must be 0):', len(bad))
"

grep -n 'class .* extends Widget' widget/*.js     # MUST find match
grep -n 'Widget.extend' widget/*.js               # MUST be empty
grep -n 'new HTML.*content:' widget/*.js          # MUST be empty
grep -n 'setContent' widget/*.js                  # MUST find match
```

**The three `#` hazards and their fixes:**

| Hazard | Bad | Safe |
|--------|-----|------|
| Private field | `static #x = ...` | `var _x = ...` (module-scoped) |
| Hex colour | `"color:#fff"` | `"color:rgb(255,255,255)"` |
| CSS ID selector | `"#panel{...}"` in double-quoted string | `'#panel{...}'` in single-quoted string |

**❌ NEVER use `[id="name"]` as a replacement for `#name` CSS selectors** — the inner `"` breaks the outer JS string, causing `SyntaxError: Unexpected identifier`.

---

**Last Updated**: 2026-06-24  
**Version**: 1.2.0
