# PodContext API Reference

Authoritative reference derived from `sap/dm/dme/pod2/context/PodContext.js` source.

---

## Overview

`PodContext` is the single shared application state store for a POD 2.0 session. It is a **static class** — all methods are called on the class itself, never on an instance.

```javascript
import PodContext from "sap/dm/dme/pod2/context/PodContext";
```

It provides:
- A **JSON model** (`PodContextModel`) holding all shared state
- **Typed getters/setters** for standard paths (validated against `PodContextMetadata`)
- **`get(path)` / `set(path, value)`** for custom paths (e.g. `"/flex5/slot1/material"`)
- **`subscribe` / `unsubscribe` / `unsubscribeAll`** for reactive change notifications
- **`getWhenAvailable(path)`** — Promise that resolves when a path becomes non-null

---

## Mode Guards

Always check mode before subscribing or fetching data. Design mode has mock data; subscriptions should only run in run mode.

```javascript
PodContext.isRunMode()     // true in the live POD player
PodContext.isDesignMode()  // true in POD Designer
PodContext.isEdge()        // true on edge deployments (reads from runtime properties at init)
PodContext.isCloud()       // true when NOT edge (i.e. standard cloud/BTP deployment)
```

**Note**: `isDesignMode()` is simply `Boolean(#oPodDesigner)` — the designer reference is truthy only when running inside the POD Designer. `isRunMode()` is `!isDesignMode()`.

---

## Core Getters

### Plant & Session

```javascript
PodContext.getPlant()           // → string  e.g. "1010"
PodContext.getPlantTimeZone()   // → string  e.g. "Europe/Berlin"
PodContext.getIndustryType()    // → "DISCRETE" | "PROCESS"
PodContext.isDiscreteIndustry() // → boolean
PodContext.isProcessIndustry()  // → boolean
PodContext.getPodId()           // → string  e.g. "MY_POD"
PodContext.getUserId()          // → string  e.g. "john.doe@company.com"
PodContext.getTenantId()        // → string
PodContext.getPodRuntime()      // → PodRuntime (navigation, widget access)
```

### Selection — ADAPTIVE PATTERN (use all three together)

```javascript
// ⚠️ ALWAYS use the adaptive pattern — not all PODs have all widgets
const oOp = PodContext.getLastSelectedOperationActivity(); // null if no OperationActivity widget
const oWL = PodContext.getLastSelectedWorkListItem();      // null if no WorkList widget
const aRes = PodContext.getFilterResources();              // array (may be empty)

if (oOp && oWL) {
    // Both widgets present — preferred source for each:
    sSfc = oWL.sfc;
    sOperation = oOp.operationActivity;
    sWorkCenter = oOp.workCenter;
} else if (oWL) {
    sSfc = oWL.sfc;
    sOperation = oWL.operationActivity;
} else {
    const aItems = PodContext.getSelectedWorkListItems(); // array fallback
    if (aItems?.length) { sSfc = aItems[0].sfc; }
}

const sResource = aRes?.[0]?.resource || null; // ALWAYS use getFilterResources() for resource
```

### Full Selection Getters

| Method | Returns | Notes |
|--------|---------|-------|
| `getLastSelectedWorkListItem()` | `WorkListItem \| null` | Last item in selection |
| `getLastSelectedOperationActivity()` | `OperationActivity \| null` | Last item in selection |
| `getSelectedWorkListItems()` | `WorkListItem[]` | All selected |
| `getSelectedOperationActivities()` | `OperationActivity[]` | All selected |
| `getFilterResources()` | `Resource[]` | Filter bar resources |
| `getFilterWorkCenters()` | `WorkCenter[]` | Filter bar work centers |
| `getFilterMaterials()` | `Material[]` | Filter bar materials |
| `getFilterOperationActivities()` | `OperationActivityMaster[]` | Filter bar ops |
| `getFilterSfcs()` | `string[]` | Filter bar SFC values |
| `getFilterProcessLot()` | `string` | Filter bar process lot |

### WorkList

```javascript
PodContext.getWorkListItems()     // → BaseWorkListItem[]  (all loaded items)
PodContext.getWorkListCount()     // → number
PodContext.getWorkListLoading()   // → boolean
PodContext.getWorkListPageSize()  // → number
PodContext.getWorkListSorting()   // → Sorting[]
PodContext.getWorkListType()      // → WorkListType enum
PodContext.clearWorkList()        // resets items=[], count=0, loading=false
```

### Operation Activities

```javascript
PodContext.getOperationActivities()        // → BaseOperationWorkItem[]
PodContext.getOperationActivitiesLoading() // → boolean (no direct getter — read via PodContext.get(ModelPath.OperationActivitiesLoading))
```

### Reported Quantities

```javascript
PodContext.getReportedQuantityItems()   // → ReportedQuantity[]
PodContext.getReportedQuantityCount()   // → number
PodContext.getReportedQuantityLoading() // → boolean
```

### Work Instructions

```javascript
PodContext.getWorkInstructions()          // → WorkInstruction[]
PodContext.getWorkInstructionsLoading()   // → boolean
PodContext.getSelectedWorkInstruction()   // → WorkInstruction
PodContext.setSelectedWorkInstruction(o)  // sets selected work instruction
```

### Activity Confirmation

```javascript
PodContext.getActivityConfirmationSummaryList()              // → ActivityConfirmationSummary[]
PodContext.setActivityConfirmationSummaryList(aList)
```

### Execution

```javascript
PodContext.getExecutionSFCQuantity()       // → number|undefined  (quantity for start/complete)
PodContext.setExecutionSFCQuantity(iQty)   // pass null to clear
```

### Miscellaneous

```javascript
PodContext.getMessageHistory()              // → UserMessage[]
PodContext.setMessageHistory(aMessages)     // auto-sorted descending by timestamp
PodContext.getSignatureHistory(sWidgetId)   // → Signature[]  (path: /signature/history/<widgetId>)
PodContext.isFeatureFlagEnabled(sFlag)      // → boolean  (internal feature flags from backend)
PodContext.getExecutionApiVersion()         // → ExecutionApiVersion  (internal, not in model)
PodContext.getUserId()                      // → string  (read-only, no setter)
PodContext.getTenantId()                    // → string  (read-only, no setter)
```

---

## Generic get / set (Custom Paths)

For any path not covered by a typed getter/setter — including custom plugin data and third-party extensions:

```javascript
// READ any path
const vValue = PodContext.get("/myNamespace/someKey");
const sMaterial = PodContext.get("/flex5/slot1/material");

// WRITE any path (path MUST start with "/")
PodContext.set("/myNamespace/someKey", "value");
PodContext.set("/flex5/slot1/material", "PM989-600D-HON");

// Wait until a path is populated (e.g. set by another plugin)
const vValue = await PodContext.getWhenAvailable("/flex5/slot1/material");
```

**Rules:**
- Path must begin with `"/"` — throws if falsy
- Standard `ModelPath` paths should use their typed setters, not `set()`
- Custom paths like `/flex5/...` are safe to use freely
- The model is a flat JSON store — nested paths work: `/a/b/c`

---

## WorkListItem Object Structure

```javascript
const oItem = PodContext.getLastSelectedWorkListItem();
// {
//   sfc: "SFC_12345",
//   material: "MATERIAL1",
//   materialVersion: "A",
//   order: "SHOP_ORDER_001",
//   workCenter: "WC-001",
//   operationActivity: "OP10-ASSEMBLY",  // ⚠️ may be stale if OperationActivity widget exists
//   stepId: "10",
//   sfcStatusCode: "402",
//   sfcQuantity: 10,
//   customFields: {                       // OBJECT not array!
//     "SHOP_ORDER.MY_FIELD": "value",
//     "ITEM.OTHER_FIELD": "123"
//   },
//   // ... 45+ total properties
// }
```

## OperationActivity Object Structure

```javascript
const oOp = PodContext.getLastSelectedOperationActivity();
// {
//   sfc: "SFC_12345",
//   operationActivity: "OP10-ASSEMBLY",  // ← use this for operation
//   workCenter: "WC-001",
//   stepId: "10",
//   resource: null,                       // ⚠️ often null — use getFilterResources() instead
//   statusComplete: false,
//   statusInWork: true
// }
```

## Resource Object Structure

```javascript
const aRes = PodContext.getFilterResources();
const sResource = aRes?.[0]?.resource; // "RESOURCE_001"
// Resource object: { resource: "RESOURCE_001", resourceType: "EQUIPMENT", description: "..." }
```

---

## Subscription

### subscribe (single path)

Callback receives `(newValue, sPath)`:

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

_onSelectionChanged(aItems, sPath) {
    // aItems = new value, sPath = the model path string
    const aList = Array.isArray(aItems) ? aItems : [];
    if (aList.length > 0) { ... }
}
```

### subscribe (multiple paths)

Callback receives `(Map<path,value>, Map<path,boolean>)`:

```javascript
PodContext.subscribe(
    [ModelPath.SelectedWorkListItems, ModelPath.FilterResources],
    (oValues, oChanged) => {
        const aItems = oValues.get(ModelPath.SelectedWorkListItems);
        const bItemsChanged = oChanged.get(ModelPath.SelectedWorkListItems);
    },
    this
);
```

### Unsubscribe

```javascript
// Specific subscription (all 3 args must match subscribe call exactly)
PodContext.unsubscribe(ModelPath.SelectedWorkListItems, this._onSelectionChanged, this);

// All subscriptions for this context — use in onExit()
PodContext.unsubscribeAll(this);
```

### CRITICAL: Always unsubscribe in onExit()

```javascript
onExit() {
    PodContext.unsubscribeAll(this); // cleans up ALL subscriptions at once
    super.onExit();
}
```

**Notification batching**: Multiple `set()` calls in the same microtask are batched — subscribers are called once with the final value, not once per `set()`.

**Parent path propagation**: Setting `/a/b/c` also notifies subscribers of `/a/b` and `/a`.

---

## Lifecycle Hooks (Static — use sparingly)

For static/shared components that must hook into the POD lifecycle rather than `onInit`/`onExit`:

```javascript
// Called after PodContext finishes init (before config/plugins load)
PodContext.attachInit(fnCallback);
PodContext.detachInit(fnCallback);

// Called when PodContext is destroyed (FLP navigation away)
PodContext.attachDestroy(fnCallback);
PodContext.detachDestroy(fnCallback);
```

---

## i18n

```javascript
// Get the global POD i18n model
PodContext.getI18nModel() // → I18nResourceModel

// Translate a key using the global POD i18n bundle
PodContext.getI18nText("someKey")           // → string
PodContext.getI18nText("key.with.param", 5) // → string with {0} substituted

// Apply all core models to a detached control (dialogs etc.)
PodContext.applyCoreModelsTo(oDialog);  // sets default model + "pod" + "i18n" + "i18n-global"
```

---

## resolveBinding

Resolves binding expressions against PodContext model at runtime:

```javascript
// Simple path
PodContext.resolveBinding("{/plant/plant}") // → "1010"

// Expression
PodContext.resolveBinding("{= ${/workList/count} > 0 ? 'Has items' : 'Empty'}")

// With formatter
PodContext.resolveBinding({
    path: "/workList/lastSelected/orderScheduledStartDate",
    type: "sap.ui.model.type.Date",
    formatOptions: { pattern: "yyyy/MM/dd" }
})
```

---

## ModelPath Constants

Import separately:
```javascript
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";
```

**ALL constants are PLURAL** for array paths. The table below shows exact path strings from source — use these when subscribing with `PodContext.subscribe()` or when reading via `PodContext.get()`.

| Constant | Actual Path String | Type |
|----------|--------------------|------|
| `ModelPath.ActivityConfirmationSummaryList` | `/activityConfirmation/summaries/list` | `ActivityConfirmationSummary[]` |
| `ModelPath.ExecutionSFCQuantity` | `/execution/sfcQuantity` | `number` |
| `ModelPath.FilterInputType` | `/filter/inputType` | `WorkListFilterInputType` |
| `ModelPath.FilterMaterials` | `/filter/materials` | `Material[]` |
| `ModelPath.FilterOperationActivities` | `/filter/operationActivities` | `OperationActivityMaster[]` |
| `ModelPath.FilterProcessLot` | `/filter/processLot` | `string` |
| `ModelPath.FilterResources` | `/filter/resources` | `Resource[]` |
| `ModelPath.FilterSfcs` | `/filter/sfcs` | `string[]` |
| `ModelPath.FilterWorkCenters` | `/filter/workCenters` | `WorkCenter[]` |
| `ModelPath.InspectionCharacteristics` | `/qualityInspection/inspectionCharacteristics` | see QI delegate |
| `ModelPath.InspectionCharacteristicsResults` | `/qualityInspection/inspectionCharacteristicsResults` | see QI delegate |
| `ModelPath.InspectionFieldCombinations` | `/qualityInspection/fieldCombinations` | see QI delegate |
| `ModelPath.InspectionPointLot` | `/qualityInspection/inspectionPointLot` | see QI delegate |
| `ModelPath.InspectionPoints` | `/qualityInspection/inspectionPoints` | see QI delegate |
| `ModelPath.IsEnablePoint` | `/qualityInspection/isEnablePoint` | `boolean` |
| `ModelPath.LastSelectedOperationActivity` | `/execution/operationActivity/lastSelected` | `BaseOperationWorkItem \| null` |
| `ModelPath.LastSelectedWorkListItem` | `/workList/lastSelected` | `BaseWorkListItem \| null` |
| `ModelPath.MessageHistory` | `/messageHistory` | `UserMessage[]` |
| `ModelPath.OperationActivities` | `/execution/operationActivity/list` | `BaseOperationWorkItem[]` |
| `ModelPath.OperationActivitiesLoading` | `/execution/operationActivity/loading` | `boolean` |
| `ModelPath.Plant` | `/plant` | `Plant` |
| `ModelPath.Pod` | `/pod` | `Pod` |
| `ModelPath.ReportedQuantityCount` | `/quantityConfirmation/count` | `number` |
| `ModelPath.ReportedQuantityItems` | `/quantityConfirmation/list` | `ReportedQuantity[]` |
| `ModelPath.ReportedQuantityLoading` | `/quantityConfirmation/loading` | `boolean` |
| `ModelPath.SelectedOperationActivities` | `/execution/operationActivity/selected` | `BaseOperationWorkItem[]` |
| `ModelPath.SelectedWorkInstruction` | `/workInstruction/selected` | `WorkInstruction` |
| `ModelPath.SelectedWorkListItems` | `/workList/selected` | `BaseWorkListItem[]` |
| `ModelPath.TenantId` | `/tenant/tenantId` | `string` |
| `ModelPath.UserId` | `/user/id` | `string` |
| `ModelPath.UserLanguage` | `/user/language` | `string` |
| `ModelPath.WorkInstructions` | `/workInstruction/list` | `WorkInstruction[]` |
| `ModelPath.WorkInstructionsLoading` | `/workInstruction/loading` | `boolean` |
| `ModelPath.WorkListCount` | `/workList/count` | `number` |
| `ModelPath.WorkListItems` | `/workList/list` | `BaseWorkListItem[]` |
| `ModelPath.WorkListLoading` | `/workList/loading` | `boolean` |
| `ModelPath.WorkListPageSize` | `/workList/pageSize` | `number` |
| `ModelPath.WorkListSorting` | `/workList/sorting` | `Sorting[]` |
| `ModelPath.WorkListType` | `/workList/type` | `WorkListType` |

**⚠️ Common path mistakes** (the paths below are WRONG — do not use them):

| ❌ Wrong path | ✅ Correct path |
|---|---|
| `/workList/selectedItems` | `/workList/selected` |
| `/workList/items` | `/workList/list` |
| `/execution/operationActivities/selected` | `/execution/operationActivity/selected` |
| `/execution/operationActivities/items` | `/execution/operationActivity/list` |
| `/reportedQuantity/items` | `/quantityConfirmation/list` |
| `/workInstructions/items` | `/workInstruction/list` |
| `/userId` | `/user/id` |
| `/tenantId` | `/tenant/tenantId` |
| `/userLanguage` | `/user/language` |
| `/activityConfirmation/summaryList` | `/activityConfirmation/summaries/list` |

---

## Common Mistakes

### ❌ Reading resource from operation object
```javascript
// WRONG — resource is often null on the operation object
const sResource = PodContext.getLastSelectedOperationActivity().resource;

// CORRECT
const sResource = PodContext.getFilterResources()?.[0]?.resource;
```

### ❌ Assuming OperationActivity widget always exists
```javascript
// WRONG — crashes in WorkList-only PODs
const sOperation = PodContext.getLastSelectedOperationActivity().operationActivity;

// CORRECT — adaptive pattern
const oOp = PodContext.getLastSelectedOperationActivity();
const sOperation = oOp ? oOp.operationActivity : PodContext.getLastSelectedWorkListItem()?.operationActivity;
```

### ❌ Subscribing without unsubscribing
```javascript
// Memory leak — always unsubscribe in onExit()
onExit() {
    PodContext.unsubscribeAll(this); // ✅
    super.onExit();
}
```

### ❌ Using PodContext.set() for standard paths
```javascript
// WRONG — bypasses type validation
PodContext.set(ModelPath.FilterResources, aResources);

// CORRECT
PodContext.setFilterResources(aResources);
```

### ❌ Calling setters that don't exist (read-only paths)
```javascript
// WRONG — no public setter exists for these paths
PodContext.set(ModelPath.UserId, "someUser");    // UserId is read-only
PodContext.set(ModelPath.TenantId, "t1");        // TenantId is read-only
PodContext.set(ModelPath.Plant, oPlant);         // Plant is read-only (set internally from backend)

// CORRECT — use the getter only, value is populated during PodContext.init()
const sUser = PodContext.getUserId();
const sPlant = PodContext.getPlant();
```

### ❌ Treating customFields as an array
```javascript
// WRONG
oItem.customFields.forEach(cf => ...); // TypeError

// CORRECT — it's an object
Object.entries(oItem.customFields || {}).forEach(([sKey, sValue]) => ...);
```

### ❌ Calling PodContext.get() without leading slash
```javascript
PodContext.get("flex5/slot1/material")  // ❌ wrong — undefined
PodContext.get("/flex5/slot1/material") // ✅ correct
```

---

## Custom Data Patterns

### Writing custom state (e.g. from a plugin that owns data)
```javascript
// Write
PodContext.set("/myPlugin/selectedItem", oItem);
PodContext.set("/flex5/slot1/material", "PM989-600D-HON");

// Subscribe to it from another plugin
PodContext.subscribe("/flex5/slot1/material", (sMaterial, sPath) => {
    // sMaterial = new value
}, this);
```

### Waiting for data set by another plugin
```javascript
async onInit() {
    await super.onInit();
    const sMaterial = await PodContext.getWhenAvailable("/flex5/slot1/material");
    // sMaterial is guaranteed non-null here
}
```

---

**Source**: `sap/dm/dme/pod2/context/PodContext.js` + `sap/dm/dme/pod2/context/ModelPath.js` (SAP Digital Manufacturing POD 2.0)  
**Last Updated**: 2026-09-07 — ModelPath string values verified against source; Quality Inspection paths added; read-only path mistakes added
