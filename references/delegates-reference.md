# POD 2.0 Data Delegates Reference

Authoritative reference derived from source code in `sap/dm/dme/pod2/context/data/`.

All delegates are **static classes** — call methods on the class, never on an instance.

---

## Overview

Delegates are the data layer between POD widgets and the backend. They:
- Fetch data from SAP DM APIs and write results into `PodContext` at specific `ModelPath` paths
- Handle request deduplication, in-flight request management, and minimum refresh intervals
- Subscribe to `PodContext` path changes to auto-refresh when selection changes
- Subscribe to real-time notification events (SFC_START, SFC_COMPLETE, etc.) where relevant
- Load design-time preview data from JSON fixture files in design mode
- Reset themselves on `PodContext.attachInit()` (POD re-initialization)

### Common Pattern for ALL Delegates

```javascript
// 1. Always await init before first use (delegates auto-init on first refresh, but explicit is cleaner)
await SomeDelegate.init();   // or refresh() — they self-init

// 2. Call refresh to populate PodContext
await SomeDelegate.refresh();

// 3. Subscribe to the PodContext path the delegate writes to
PodContext.subscribe(ModelPath.SomePath, this._onDataChanged, this);

// 4. Read the data directly from PodContext
const aItems = PodContext.get(ModelPath.SomePath);
```

---

## WorkListDelegate

**Import:** `sap/dm/dme/pod2/context/data/WorkListDelegate`  
**Writes to:** `ModelPath.WorkListItems`, `ModelPath.WorkListCount`, `ModelPath.WorkListLoading`, `ModelPath.SelectedWorkListItems`

### Purpose
Fetches and manages the work list. Handles three work list types: `WorkCenter`, `OperationActivity`, and order-based (`ProcessOrder`/`ProductionOrder`). Auto-refreshes on SFC_START, SFC_SIGNOFF, SFC_COMPLETE notifications.

### Public Methods

```javascript
// Full foreground refresh — clears list, shows busy indicator, uses current filter/sort
await WorkListDelegate.refresh(oOptions);
// oOptions: { filter?, ignoreRefreshInterval?, abortPendingRequest? }

// Background refresh — does NOT clear list, no busy indicator, same filter/sort as last refresh
await WorkListDelegate.refreshInBackground(oOptions);
// oOptions: { ignoreRefreshInterval?, abortPendingRequest? }

// Fetch next page — appends to current list
await WorkListDelegate.fetchNextPage();

// Selective refresh — only refreshes items in the list that match given SFCs
await WorkListDelegate.refreshIfContainsSfc(["SFC001", "SFC002"]);

// Surgical refresh — updates only stale items by SFC, preferred over refreshIfContainsSfc
await WorkListDelegate.refreshStaleSfcs("SFC001");
await WorkListDelegate.refreshStaleSfcs(["SFC001", "SFC002"]);

// Get the filter object used in the most recent refresh
const oFilter = WorkListDelegate.getLatestFilter();
```

### Key Behaviours
- **1000ms minimum refresh interval** — duplicate calls within 1s are silently skipped (override with `ignoreRefreshInterval: true`)
- **Pending request guard** — if a request is in-flight, new calls are skipped by default (override with `abortPendingRequest: true`)
- **Selection reconciliation** — after every refresh, `SelectedWorkListItems` is automatically reconciled so selections for SFCs no longer in the list are cleared
- **Duplicate deduplication** — duplicate work list items from pagination are automatically removed
- **`refresh()` requires a filter** — if no filter has been set yet, the refresh is skipped silently
- **Design mode** — loads `workList.json` fixture, auto-selects first item

### WorkList Types

| Type | Required Filter Fields |
|------|----------------------|
| `WorkListType.WorkCenter` | `workCenters[]` required; `sfc`, `resource` optional |
| `WorkListType.OperationActivity` | `resource` + `operationActivity` required |
| `WorkListType.ProcessOrder` / `ProductionOrder` | `workCenters[]` + `plannedStartDateRange` required |

---

## OperationActivityDelegate

**Import:** `sap/dm/dme/pod2/context/data/OperationActivityDelegate`  
**Writes to:** `ModelPath.OperationActivities`, `ModelPath.SelectedOperationActivities`, `ModelPath.OperationActivitiesLoading`

### Purpose
Fetches operation activities (discrete) or phases (order-based) for the selected work list item. **Auto-subscribes** to `SelectedWorkListItems` on init and refreshes automatically when the selection changes. Also responds to SFC and operation notification events.

### Public Methods

```javascript
// Refresh operation activities for the current selected work list item
await OperationActivityDelegate.refresh({ force: false });
// force: true bypasses the 1000ms minimum interval

// Enable/disable auto-selection of the next available operation activity after each refresh
OperationActivityDelegate.setAutoSelect(true);

// Manually select the next eligible operation activity (non-completed, first in routing order)
OperationActivityDelegate.selectNextOperationActivity();
```

### Key Behaviours
- **Auto-subscribes on init** — you do NOT need to subscribe `SelectedWorkListItems` yourself if you use this delegate; it does it internally
- **1000ms minimum refresh interval** — bypass with `{ force: true }`
- **Single pending request** — if a refresh is in-flight, new calls await the in-flight one
- **Auto-select** — when `setAutoSelect(true)`, selects the first `NewOrInQueue`, `InWork`, or `CompletePending` operation after each refresh
- **Selection pruning** — after refresh, `SelectedOperationActivities` is pruned so selections for operations no longer in the list are removed; previously selected operations that are still present are re-selected
- **Routing-based sort** — operation activities are sorted by routing step order, not alphabetically
- **Notification events subscribed:** `SFC_START`, `SFC_SIGNOFF`, `SFC_COMPLETE`, `OPERATION_START`, `OPERATION_COMPLETE`
- **Design mode** — loads `operationActivities.json` fixture, auto-selects first item

### What Triggers a Refresh
1. Manual call to `refresh()`
2. `SelectedWorkListItems` changes (auto-subscribed in `#init()`)
3. Notification event received for a relevant SFC or operation

---

## ActivityConfirmationDelegate

**Import:** `sap/dm/dme/pod2/context/data/ActivityConfirmationDelegate`  
**Writes to:** `ModelPath.ActivityConfirmationSummaryList`

### Purpose
Fetches activity confirmation summaries for the current operation and work list selection. Auto-subscribes to `SelectedOperationActivities`.

### Public Methods

```javascript
// Refresh activity summaries for current selection
await ActivityConfirmationDelegate.refreshActivitySummaries(oOptions);
// oOptions: { force?: boolean, clear?: boolean }
// clear: true clears existing data before fetching (avoids showing stale data during load)
```

### Request Parameters (built automatically from PodContext)
- `shopOrder` ← `getLastSelectedWorkListItem().order`
- `batchId` ← `getLastSelectedWorkListItem().sfc`
- `operationActivity` ← `getLastSelectedOperationActivity().operationActivity`
- `workCenter` ← `getLastSelectedOperationActivity().workCenter` (falls back to WorkListItem.workCenter)
- `stepId` ← `getLastSelectedOperationActivity().stepId`

### Key Behaviours
- **Requires BOTH** operation and work list item to be selected — clears list if either is missing
- **Deduplication** — skips if request matches the previous request (unless `force: true`)
- **Single pending request** — concurrent refresh calls are dropped
- **Results sorted** by `sequence` then `activityId` ascending
- **Auto-subscribes** to `SelectedOperationActivities` on init
- **Design mode** — loads `activityConfirmationSummaries.json` fixture

---

## QuantityConfirmationDelegate

**Import:** `sap/dm/dme/pod2/context/data/QuantityConfirmationDelegate`  
**Writes to:** `ModelPath.ReportedQuantityItems`, `ModelPath.ReportedQuantityCount`, `ModelPath.ReportedQuantityLoading`

### Purpose
Fetches reported quantities for the current operation and work list selection. Supports pagination. Auto-subscribes to `SelectedWorkListItems` and `SelectedOperationActivities`.

### Public Methods

```javascript
// Refresh reported quantities (page 0, 20 per page)
await QuantityConfirmationDelegate.refresh(oOptions);
// oOptions: { abortPendingRequest?, page?, size?, force? }

// Append next page of results
await QuantityConfirmationDelegate.fetchNextPage();
```

### Request Parameters (built automatically from PodContext)
- `shopOrder` ← `getLastSelectedWorkListItem().order`
- `batchId` ← `getLastSelectedWorkListItem().sfc`
- `phase` ← `getLastSelectedOperationActivity().operationActivity`
- `page` / `size` — pagination (default size: 20)

### Key Behaviours
- **Requires BOTH** work list item AND operation activity — skips if either is missing
- **Default page size: 20**
- **`fetchNextPage()`** appends to existing items (does not replace)
- **`refresh()`** replaces items (page 0)
- **Abort support** — `abortPendingRequest: true` cancels in-flight request
- **Auto-subscribes** to both `SelectedWorkListItems` and `SelectedOperationActivities` on init
- **Design mode** — loads `reportedQuantities.json` fixture

---

## WorkInstructionDelegate

**Import:** `sap/dm/dme/pod2/context/data/WorkInstructionDelegate`  
**Writes to:** `ModelPath.WorkInstructions`, `ModelPath.WorkInstructionsLoading`, `ModelPath.SelectedWorkInstruction`

### Purpose
Fetches work instructions for the current selection. Uses a **two-step loading strategy**: metadata first (fast), elements on-demand (avoids 5MB+ upfront transfers). Auto-subscribes to selection and filter changes.

### Public Methods

```javascript
// Must be called explicitly before first use (unlike other delegates)
await WorkInstructionDelegate.init();

// Refresh work instructions
await WorkInstructionDelegate.refresh(oOptions);
// oOptions: { request?, force? }
// request: override the auto-built request object
// force: refresh even if request hasn't changed

// Load elements for a specific work instruction (call when user selects one to view)
const aElements = await WorkInstructionDelegate.loadWorkInstructionElements(sWorkInstructionId);
// Returns WorkInstructionElement[] or [] on error
```

### Request Parameters (built automatically from PodContext)
- `sfcs[]` ← `getSelectedWorkListItems()` (all selected, or just last for order-based)
- `operations[]` ← `getSelectedOperationActivities()` (with version) OR `getFilterOperationActivities()`
- `resource` ← `getLastSelectedOperationActivity().resource` OR `getFilterResources()[0]`
- `skipWorkInstructionElementsReading: true` (always — elements loaded on-demand)

### Key Behaviours
- **`init()` must be called explicitly** — unlike most delegates, it does not auto-init on `refresh()`
- **"Last-write-wins" request handling** — new requests cancel in-flight ones via `AbortController`
- **Request ID tracking** — stale responses received after a newer request started are discarded
- **Auto-subscribes** to `[SelectedWorkListItems, SelectedOperationActivities, FilterResources]` (multi-path subscription)
- **Auto-selects** the first non-`HEADER_TEXT` work instruction after each refresh; preserves previous selection if still present
- **Returns `null` if selection is cleared** — calls `setWorkInstructions([])` and `setSelectedWorkInstruction(null)`
- **Design mode** — loads `workInstructions.json`, auto-selects first item

---

## QualityInspectionDelegate

**Import:** `sap/dm/dme/pod2/context/data/QualityInspectionDelegate`  
**Writes to:** `ModelPath.InspectionPoints`, `ModelPath.InspectionCharacteristics`, `ModelPath.IsEnablePoint`, `ModelPath.InspectionFieldCombinations`

### Purpose
Fetches quality inspection points for the current operation activity selection. Auto-subscribes to `SelectedOperationActivities`.

### Public Methods

```javascript
// Refresh inspection points (requires explicit request object)
await QualityInspectionDelegate.refresh(oOptions);
// oOptions: { request?, force? }
// ⚠️ request is NOT auto-built — caller must provide it

// Utility: fetch JSON from a URL
const oData = await QualityInspectionDelegate.fetchData(sUrl);
```

### ⚠️ Important Difference
Unlike other delegates, `QualityInspectionDelegate.refresh()` requires the caller to **explicitly provide the request object** in `oOptions.request`. If no request is provided, it clears `InspectionPoints` and returns. The delegate does NOT auto-build the request from PodContext.

### Key Behaviours
- **Requires explicit `request`** — does not auto-build from PodContext
- **Deduplication** — skips if request matches previous (unless `force: true`)
- **Single pending request guard** — concurrent calls are dropped
- **Auto-subscribes** to `SelectedOperationActivities` on `#init()`
- **Design mode** — loads `inspectionPoints.json` and `fieldCombinations.json`, sets `IsEnablePoint: true`

---

## DataCollectionDelegate

**Import:** `sap/dm/dme/pod2/context/data/DataCollectionDelegate`  
Source file was empty at time of documentation — delegate exists but has no public API documented.

---

## Usage Patterns

### Pattern 1: Widget Using a Delegate (Recommended)

```javascript
async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        // Initialize the delegate
        await WorkInstructionDelegate.init();  // WorkInstructionDelegate needs explicit init
        // OR for others: first refresh call triggers auto-init

        // Subscribe BEFORE calling refresh so you don't miss the first notification
        PodContext.subscribe(ModelPath.WorkInstructions, this._onWorkInstructionsChanged, this);

        // Trigger first load
        await WorkInstructionDelegate.refresh();
    }
}

_onWorkInstructionsChanged(aInstructions, sPath) {
    const aList = Array.isArray(aInstructions) ? aInstructions : [];
    this._updateUI(aList);
}

onExit() {
    PodContext.unsubscribeAll(this);
    super.onExit();
}
```

### Pattern 2: Using OperationActivityDelegate with Auto-Select

```javascript
async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        // Enable auto-selection before init so it applies from the start
        OperationActivityDelegate.setAutoSelect(true);

        // Delegate auto-subscribes to SelectedWorkListItems internally
        await OperationActivityDelegate.refresh();

        // Subscribe to see the results
        PodContext.subscribe(ModelPath.SelectedOperationActivities,
            this._onOperationSelected, this);
    }
}
```

### Pattern 3: Selective WorkList Refresh After SFC Action

```javascript
async _onSfcStart(sSfc) {
    try {
        await ApiClient.sfc.sfcStart({ plant: PodContext.getPlant(), sfcs: [sSfc], ... });

        // Surgical refresh — only update the stale item, don't clear the whole list
        await WorkListDelegate.refreshStaleSfcs(sSfc);
    } catch (oError) {
        MessageHistory.showError(oError.message);
    }
}
```

### Pattern 4: Paginated Reported Quantities

```javascript
async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        // Delegate auto-subscribes to SelectedWorkListItems + SelectedOperationActivities
        PodContext.subscribe(ModelPath.ReportedQuantityItems, this._onQuantitiesLoaded, this);
        await QuantityConfirmationDelegate.refresh();
    }
}

async _onLoadMore() {
    await QuantityConfirmationDelegate.fetchNextPage();
}
```

---

## Delegate Comparison Table

| Delegate | Import Path Suffix | Writes To | Auto-Subscribes | Needs Explicit init()? | Design Mode Fixture |
|----------|-------------------|-----------|-----------------|----------------------|---------------------|
| WorkListDelegate | `context/data/WorkListDelegate` | WorkListItems, WorkListCount, SelectedWorkListItems | ❌ (notification-driven) | ❌ | workList.json |
| OperationActivityDelegate | `context/data/OperationActivityDelegate` | OperationActivities, SelectedOperationActivities | ✅ SelectedWorkListItems | ❌ | operationActivities.json |
| ActivityConfirmationDelegate | `context/data/ActivityConfirmationDelegate` | ActivityConfirmationSummaryList | ✅ SelectedOperationActivities | ❌ | activityConfirmationSummaries.json |
| QuantityConfirmationDelegate | `context/data/QuantityConfirmationDelegate` | ReportedQuantityItems, ReportedQuantityCount | ✅ SelectedWorkListItems + SelectedOperationActivities | ❌ | reportedQuantities.json |
| WorkInstructionDelegate | `context/data/WorkInstructionDelegate` | WorkInstructions, SelectedWorkInstruction | ✅ SelectedWorkListItems + SelectedOperationActivities + FilterResources | ✅ YES | workInstructions.json |
| QualityInspectionDelegate | `context/data/QualityInspectionDelegate` | InspectionPoints | ✅ SelectedOperationActivities | ❌ | inspectionPoints.json |
| ComponentConsumptionDelegate | `sfccomponents/consumption/context/data/ComponentConsumptionDelegate` | componentConsumption/list/items | ❌ (manual) | ✅ YES (`init()` then `refreshHeaderMaterial()` then `refreshConsumptionList(true)`) | consumptionList.json |

---

## Import Paths (Full)

```javascript
import WorkListDelegate          from "sap/dm/dme/pod2/context/data/WorkListDelegate";
import OperationActivityDelegate from "sap/dm/dme/pod2/context/data/OperationActivityDelegate";
import ActivityConfirmationDelegate from "sap/dm/dme/pod2/context/data/ActivityConfirmationDelegate";
import QuantityConfirmationDelegate from "sap/dm/dme/pod2/context/data/QuantityConfirmationDelegate";
import WorkInstructionDelegate   from "sap/dm/dme/pod2/context/data/WorkInstructionDelegate";
import QualityInspectionDelegate from "sap/dm/dme/pod2/context/data/QualityInspectionDelegate";

// Component consumption (different path)
import ComponentConsumptionDelegate from "sap/dm/dme/pod2/sfccomponents/consumption/context/data/ComponentConsumptionDelegate";
```

---

## Common Mistakes

### ❌ Subscribing AFTER the first refresh
```javascript
// WRONG — you miss the first notification
await WorkListDelegate.refresh();
PodContext.subscribe(ModelPath.WorkListItems, this._handler, this); // too late!

// CORRECT — subscribe first
PodContext.subscribe(ModelPath.WorkListItems, this._handler, this);
await WorkListDelegate.refresh();
```

### ❌ Forgetting WorkInstructionDelegate.init()
```javascript
// WRONG — refresh() won't auto-init like other delegates
await WorkInstructionDelegate.refresh();  // silently does nothing!

// CORRECT
await WorkInstructionDelegate.init();
await WorkInstructionDelegate.refresh();
```

### ❌ Double-subscribing to SelectedWorkListItems when using OperationActivityDelegate
```javascript
// OperationActivityDelegate ALREADY subscribes to SelectedWorkListItems internally.
// Adding your own subscription is fine, but calling refresh() from it causes duplicate fetches.

// WRONG
PodContext.subscribe(ModelPath.SelectedWorkListItems, async () => {
    await OperationActivityDelegate.refresh(); // delegate already does this!
}, this);

// CORRECT — just subscribe to the output path, not the trigger
PodContext.subscribe(ModelPath.OperationActivities, this._onOpsLoaded, this);
```

### ❌ Passing wrong request type to QualityInspectionDelegate
```javascript
// WRONG — QualityInspectionDelegate does NOT auto-build the request
await QualityInspectionDelegate.refresh(); // clears InspectionPoints and returns!

// CORRECT — explicitly provide the request
await QualityInspectionDelegate.refresh({
    request: {
        plant: PodContext.getPlant(),
        sfc: PodContext.getLastSelectedWorkListItem().sfc,
        // ... other required fields
    }
});
```

### ❌ Not using refreshStaleSfcs when you only need to update one item
```javascript
// WRONG — clears entire list, expensive, causes UI flicker
await WorkListDelegate.refresh();

// CORRECT — surgical update, preserves rest of list
await WorkListDelegate.refreshStaleSfcs(sSfc);
```

---

**Source**: SAP Digital Manufacturing POD 2.0 delegate source files  
**Last Updated**: 2026-09-07
