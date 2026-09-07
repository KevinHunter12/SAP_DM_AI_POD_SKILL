# POD 2.0 Delegate Architecture

## Overview

POD 2.0 provides **8 official data delegates** that manage different aspects of manufacturing execution. This document covers delegate architecture, patterns, and integration with PodContext.

---

## Official POD 2.0 Delegates

| Delegate | Purpose | Key Features |
|----------|---------|--------------|
| **ActivityConfirmationDelegate** | Activity confirmations and summaries | Simple deduplication, sorting by sequence |
| **DataCollectionDelegate** | Data collection (placeholder) | Currently empty/no implementation |
| **GoodsReceiptDelegate** | Goods receipt summaries and line items | Single API, dual PodContext updates |
| **OperationActivityDelegate** | Operation activities and phases | WebSocket notifications, auto-select |
| **QualityInspectionDelegate** | Quality inspection points | Manual refresh, multi-property updates |
| **QuantityConfirmationDelegate** | Reported quantities | Pagination, AbortController |
| **WorkInstructionDelegate** | Work instructions and elements | On-demand loading, request ID tracking |
| **WorkListDelegate** | Work list items (SFC/order-based) | Most complex, multi-type, notifications |

---

## Core Delegate Pattern

All delegates follow a consistent architecture with static class pattern:

```javascript
import Logger from "sap/dm/dme/pod2/Logger";
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
import Utilities from "sap/dm/dme/pod2/Utilities";

class DelegateName {
    static #oLog = Logger.getLogger("sap.dm.dme.pod2.context.data.DelegateName");
    static #bInitialized = false;
    static #oPendingRequest; // Promise for preventing duplicate requests
    static #oPreviousRequest; // For deduplication
    
    // Static initializer - attaches to PodContext.attachInit()
    static {
        PodContext.attachInit(() => {
            this.#bInitialized = false;
            this.#oPreviousRequest = undefined;
            // Reset other state...
        });
    }
    
    // Private initialization method
    static async #init() {
        if (this.#bInitialized) return;
        this.#bInitialized = true;
        await PodContext.getPodRuntime().ready();
        
        if (PodContext.isRunMode()) {
            // Subscribe to PodContext changes
            PodContext.subscribe(ModelPath.SelectedWorkListItems, 
                () => this.refresh(), this);
            
            // Subscribe to notifications (if needed)
            // this.#subscribeToNotifications();
        } else {
            // Load design-time sample data
            const oResponse = await fetch(
                sap.ui.require.toUrl("sap/dm/dme/pod2/designer/data/sampleData.json")
            );
            const aData = await oResponse.json();
            PodContext.setData(aData.map(item => Type.fromInternalApiResponse(item)));
            this.#oLog.info("Sample data loaded.");
        }
    }
    
    // Public refresh method - main entry point
    static async refresh(oOptions = {}) {
        if (!this.#bInitialized) await this.#init();
        if (PodContext.isDesignMode()) return;
        
        // Build request
        const oRequest = this.#buildRequest();
        if (!oRequest) {
            this.#oPreviousRequest = null;
            PodContext.setData([]);
            return;
        }
        
        // Deduplication check
        if (!oOptions.force && this.#oPreviousRequest &&
            Utilities.shallowEqual(oRequest, this.#oPreviousRequest)) {
            this.#oLog.debug("Request skipped (duplicate).");
            return;
        }
        
        // Prevent concurrent requests
        if (!this.#oPendingRequest) {
            this.#oPreviousRequest = oRequest;
            this.#oPendingRequest = ApiClient.internal.xxx.getData(oRequest)
                .finally(() => { this.#oPendingRequest = null; });
        } else {
            this.#oLog.debug("Request skipped. Already pending.");
            return;
        }
        
        try {
            const aData = await this.#oPendingRequest;
            const aTyped = aData.map(o => Type.fromInternalApiResponse(o));
            PodContext.setData(aTyped);
            this.#oLog.info(`Fetched ${aTyped.length} items`);
        } catch (oError) {
            if (oError.name === "AbortError") return;
            this.#oLog.error(`Failed: ${oError.message}`);
            PodContext.setData([]);
        }
    }
    
    static #buildRequest() {
        const oItem = PodContext.getLastSelectedWorkListItem();
        if (!oItem) return null;
        
        return {
            plant: PodContext.getPlant(),
            sfc: oItem.sfc,
            // ... other params
        };
    }
}

export default DelegateName;
```

### Key Patterns

- **Static class with private fields (#)** - Modern JavaScript encapsulation
- **Lazy initialization via #init()** - Only initializes when first called
- **Separate run mode vs design mode** - Different behavior for runtime vs designer
- **Promise-based request management** - Prevents duplicate/concurrent requests
- **PodContext subscription** - Reactive updates when context changes
- **Lifecycle management** - Reset state on POD re-initialization

---

## Initialization Lifecycle

**Critical Pattern:** All delegates attach to `PodContext.attachInit()` to reset state:

```javascript
static {
    PodContext.attachInit(() => {
        this.#bInitialized = false;
        this.#oPreviousRequest = undefined;
        this.#oPendingRequest = null;
        this.#oAbortController = null;
        // Reset other state...
    });
}
```

**Why this matters:**
- POD can reinitialize (e.g., user switches plant/resource)
- Without proper reset, stale data persists
- Subscriptions need to be re-established
- Request state must be cleared

---

## PodContext Integration

### PodContext Subscribe Pattern

Delegates use `PodContext.subscribe()` to react to context changes:

```javascript
// Single path subscription
PodContext.subscribe(
    ModelPath.SelectedWorkListItems, 
    () => this.refresh(), 
    this
);

// Multiple paths subscription
PodContext.subscribe([
    ModelPath.FilterResources,
    ModelPath.FilterWorkCenters,
    ModelPath.FilterOperationActivities
], () => {
    // Update filters or refresh data
}, this);
```

**Common ModelPath subscriptions:**

| ModelPath | Type | Description |
|-----------|------|-------------|
| `ModelPath.SelectedWorkListItems` | Array | When work list selection changes |
| `ModelPath.SelectedOperationActivities` | Array | When operation activity selection changes |
| `ModelPath.FilterResources` | Array | When resource filter changes |
| `ModelPath.FilterWorkCenters` | Array | When work center filter changes |
| `ModelPath.FilterOperationActivities` | Array | When operation activity filter changes |
| `ModelPath.FilterMaterials` | Array | When material filter changes |

### PodContext Data Access Methods

**Getters (Read Data):**
```javascript
// Selection
PodContext.getLastSelectedWorkListItem()      // Single WorkListItem
PodContext.getLastSelectedOperationActivity() // Single OperationActivity
PodContext.getSelectedWorkListItems()         // Array
PodContext.getSelectedOperationActivities()   // Array

// Filters
PodContext.getFilterResources()               // Array
PodContext.getFilterWorkCenters()             // Array
PodContext.getFilterOperationActivities()     // Array
PodContext.getFilterMaterials()               // Array

// Context
PodContext.getPlant()                         // String
PodContext.getCurrentResource()               // Resource object

// Data Collections
PodContext.getWorkListItems()                 // Array
PodContext.getOperationActivities()           // Array
```

**Setters (Update Context):**
```javascript
// Work List
PodContext.setWorkListItems(aItems)
PodContext.setWorkListCount(iCount)
PodContext.setWorkListLoading(bLoading)
PodContext.setSelectedWorkListItems(aItems)

// Operation Activities
PodContext.setOperationActivities(aItems)
PodContext.setOperationActivitiesLoading(bLoading)
PodContext.setSelectedOperationActivities(aItems)

// Activity Confirmations
PodContext.setActivityConfirmationSummaryList(aItems)

// Quantity Confirmations
PodContext.setReportedQuantityItems(aItems)
PodContext.setReportedQuantityCount(iCount)
PodContext.setReportedQuantityLoading(bLoading)

// Quality Inspections
PodContext.setInspectionPoints(aItems)
PodContext.setFieldCombinations(aItems)
PodContext.setIsEnablePoint(bEnabled)

// Work Instructions
PodContext.setWorkInstructions(aItems)
PodContext.setSelectedWorkInstruction(oItem)
PodContext.setWorkInstructionsLoading(bLoading)

// Goods Receipt
PodContext.setGoodsReceiptSummary(oSummary)
PodContext.setGoodsReceiptLineItems(aItems)
```

---

## WebSocket Notification Pattern

Several delegates (WorkListDelegate, OperationActivityDelegate) subscribe to real-time notifications:

```javascript
import PodNotificationWebSocket from "sap/dm/dme/pod2/notification/PodNotificationWebSocket";
import EventType from "sap/dm/dme/pod2/notification/EventType";
import Filter from "sap/dm/dme/pod2/notification/Filter";

// Subscribe to notifications
this.#oSubscriptionContextSfcStart = PodNotificationWebSocket.subscribe({
    eventType: EventType.SFC_START,
    onMessage: (oPodNotificationPayload) => 
        this.#onSfcStartNotificationMessage(
            EventType.SFC_START, 
            oPodNotificationPayload.data
        ),
    filter: oFilter,
    description: "DelegateName"
});

// Update subscription filter dynamically
this.#oSubscriptionContextSfcStart.updateFilter(oNewFilter);
```

**Common Event Types:**
- `EventType.SFC_START` - SFC started on operation
- `EventType.SFC_SIGNOFF` - SFC signed off from operation
- `EventType.SFC_COMPLETE` - SFC completed
- `EventType.OPERATION_START` - Operation started
- `EventType.OPERATION_COMPLETE` - Operation completed

**Filter Construction:**
```javascript
const oFilter = Filter.and([
    Filter.equals("plant", plant),
    Filter.equalsAny("workCenter.workCenter", workCenters),
    Filter.equalsAny("resource.resource", resources),
    Filter.equalsAny("routingStep.operationActivity.operationActivity", operationActivities),
    Filter.equalsAny("material.material", materials)
]);
```

---

## Request Management Patterns

### Basic Request Deduplication

```javascript
// Prevent duplicate requests
if (!oOptions.force && this.#oPreviousRequest &&
    Utilities.shallowEqual(oRequest, this.#oPreviousRequest)) {
    return;
}

// Prevent concurrent requests
if (!this.#oPendingRequest) {
    this.#oPreviousRequest = oRequest;
    this.#oPendingRequest = ApiClient.internal.xxx.getData(oRequest)
        .finally(() => { this.#oPendingRequest = null; });
} else {
    this.#oLog.debug("Refresh skipped. A request is already pending.");
    return;
}

await this.#oPendingRequest;
```

### Advanced Request Cancellation (AbortController)

Used in WorkListDelegate, QuantityConfirmationDelegate, WorkInstructionDelegate:

```javascript
static #oAbortController;

// Cancel previous request if needed
if (this.#oPendingRequest) {
    if (oOptions.abortPendingRequest) {
        this.#oAbortController.abort("Refreshed while pending.");
        await this.#oPendingRequest.catch(() => {});
    } else {
        return; // Skip new request
    }
}

// Create new AbortController
this.#oAbortController = new AbortController();

// Send request with signal
this.#oPendingRequest = ApiClient.internal.xxx.getData(oRequest, 
    { signal: this.#oAbortController.signal })
    .finally(() => {
        this.#oPendingRequest = null;
        this.#oAbortController = null;
    });
```

### Request ID Pattern (Race Condition Prevention)

Used in WorkInstructionDelegate:

```javascript
static #iRequestId = 0;

// Increment request ID
const iCurrentRequestId = ++this.#iRequestId;

// Send request
const oPendingRequest = ApiClient.internal.xxx.getData(oRequest)
    .finally(() => {
        if (iCurrentRequestId === this.#iRequestId) {
            this.#oPendingRequest = null;
        }
    });

this.#oPendingRequest = oPendingRequest;

try {
    const aResults = await oPendingRequest;
    
    // Check if still latest request
    if (iCurrentRequestId !== this.#iRequestId) {
        this.#oLog.info("Discarding stale response.");
        return;
    }
    
    // Process results
} catch (oError) {
    if (oError.name === "AbortError") return;
    // Handle error only if still latest
    if (iCurrentRequestId === this.#iRequestId) {
        // Error handling
    }
}
```

### Minimum Refresh Interval

```javascript
const MIN_REFRESH_INTERVAL = 1000; // Milliseconds

static #oLastRefreshTime = 0;

const iElapsed = Date.now() - this.#oLastRefreshTime;
if (!oOptions.force && iElapsed < MIN_REFRESH_INTERVAL) {
    this.#oLog.debug(`Refresh skipped (${iElapsed}ms < ${MIN_REFRESH_INTERVAL}ms).`);
    return;
}

this.#oLastRefreshTime = Date.now();
```

---

## Design Mode (Preview Data) Pattern

All delegates load sample JSON data in design mode:

```javascript
if (PodContext.isDesignMode()) {
    return; // Skip actual API calls
}

// In #init() for design mode:
if (!PodContext.isRunMode()) {
    const oResponse = await fetch(
        sap.ui.require.toUrl("sap/dm/dme/pod2/designer/data/sampleData.json")
    );
    const aData = await oResponse.json();
    PodContext.setData(aData.map(item => Type.fromInternalApiResponse(item)));
    this.#oLog.info("Sample data loaded.");
}
```

**Sample Data Files:**
- `sap/dm/dme/pod2/designer/data/workList.json`
- `sap/dm/dme/pod2/designer/data/operationActivities.json`
- `sap/dm/dme/pod2/designer/data/activityConfirmationSummaries.json`
- `sap/dm/dme/pod2/designer/data/reportedQuantities.json`
- `sap/dm/dme/pod2/designer/data/workInstructions.json`
- `sap/dm/dme/pod2/designer/data/inspectionPoints.json`
- `sap/dm/dme/pod2/designer/data/fieldCombinations.json`

---

## API Client Patterns

### Common API Namespaces

```javascript
// Work List
ApiClient.internal.worklist.getWorkCenterWorkList(oRequest)
ApiClient.internal.worklist.getWorkCenterWorkListCount(oRequest)
ApiClient.internal.worklist.getWorkCenterWorkListItem(oRequest)
ApiClient.internal.worklist.getOperationActivityWorkList(oRequest)
ApiClient.internal.worklist.getOperationWorkListCount(oRequest)
ApiClient.internal.worklist.getOrderWorkList(oRequest)
ApiClient.internal.worklist.getOrderWorkListCount(oRequest)
ApiClient.internal.worklist.getOperationActivities(oRequest)
ApiClient.internal.worklist.getPhases(oRequest)

// Activity Confirmations
ApiClient.internal.activityconfirmation.getSummaries(oRequest)

// Quantity Confirmations
ApiClient.internal.sfc.getReportedQuantityDetails(oRequest)

// Quality Inspection
ApiClient.internal.qualityInspection.getInspectionPoints(oRequest)

// Work Instructions
ApiClient.internal.workinstruction.findByContext(oRequest, oRequestInit)
ApiClient.internal.workinstruction.findWorkInstructionElements(oRequest)

// Goods Receipt
ApiClient.internal.inventory.getGoodsReceiptSummary(oRequest)
```

### Type Conversion Pattern

All API responses are converted to typed objects:

```javascript
const aTypedItems = aApiResponse.map((oRecord) => 
    TypedClass.fromInternalApiResponse(oRecord)
);
PodContext.setItems(aTypedItems);
```

**Common Types:**
- `WorkListItem.fromInternalApiResponse()`
- `OrderWorkListItem.fromInternalApiResponse()`
- `OperationActivity.fromInternalApiResponse()`
- `Phase.fromInternalApiResponse()`
- `ActivityConfirmationSummary.fromInternalApiResponse()`
- `ReportedQuantity.fromInternalApiResponse()`
- `WorkInstruction.fromInternalApiResponse()`
- `WorkInstructionElement.fromApiResponse()`
- `GoodsReceiptSummary.fromInternalApiResponse()`
- `GoodsReceiptLineItem.fromInternalApiResponse()`

---

## Pagination Pattern

Used in WorkListDelegate and QuantityConfirmationDelegate:

```javascript
// Refresh (page 0)
static async refresh(oOptions = {}) {
    await this.#fetch({
        offset: 0,
        pageSize: PodContext.getWorkListPageSize(),
        clear: true,
        busy: true,
        ...oOptions
    });
}

// Fetch next page
static async fetchNextPage() {
    await this.#fetch({
        offset: PodContext.getWorkListItems().length,
        pageSize: PodContext.getWorkListPageSize(),
        clear: false,
        busy: true
    });
}

// Update context
if (bRefreshing) {
    PodContext.setItems(aNewItems);
} else {
    const aExisting = PodContext.getItems();
    PodContext.setItems(aExisting.concat(aNewItems));
}
```

---

## Loading State Pattern

```javascript
// Set loading before request
PodContext.setWorkListLoading(true);

try {
    const aData = await ApiClient.internal.xxx.getData(oRequest);
    PodContext.setData(aData);
} catch (oError) {
    this.#oLog.error(`Failed: ${oError.message}`);
} finally {
    PodContext.setWorkListLoading(false);
}
```

**Loading State Methods:**
- `PodContext.setWorkListLoading(bLoading)`
- `PodContext.setOperationActivitiesLoading(bLoading)`
- `PodContext.setReportedQuantityLoading(bLoading)`
- `PodContext.setWorkInstructionsLoading(bLoading)`

---

## Selection Management Patterns

### Auto-Select First Item

```javascript
// Select first item by default
if (!oPreviousSelection && aItems.length > 0) {
    oInitialSelection = aItems[0];
}
PodContext.setSelectedWorkInstruction(oInitialSelection);
```

### Preserve Selection After Refresh

```javascript
static #refreshSelectedWorkListItems() {
    const aSelected = PodContext.getSelectedWorkListItems();
    const aWorkList = PodContext.getWorkListItems();
    
    const oSelectedSfcs = new Set(aSelected.map(o => o.sfc));
    if (aWorkList.length && oSelectedSfcs.size) {
        PodContext.setSelectedWorkListItems(
            aWorkList.filter(o => oSelectedSfcs.has(o.sfc))
        );
    } else {
        PodContext.setSelectedWorkListItems([]);
    }
}
```

### Prune Stale Selections

```javascript
static #pruneSelectedItems(aNewOperationActivities) {
    const aNewSelectedItems = [];
    const aPreviousSelected = PodContext.getSelectedOperationActivities();
    
    if (aNewOperationActivities?.length && aPreviousSelected?.length) {
        const oSelectedIds = new Set(aPreviousSelected.map(o => o.getIdentifier()));
        for (const oItem of aNewOperationActivities) {
            if (oSelectedIds.has(oItem.getIdentifier())) {
                aNewSelectedItems.push(oItem);
            }
        }
    }
    
    PodContext.setSelectedOperationActivities(aNewSelectedItems);
}
```

---

## Performance Optimization Patterns

### On-Demand Loading (WorkInstructionDelegate)

```javascript
// Initial fetch: Load metadata only, skip large element data
const oRequest = {
    sfcs: aSfcs,
    skipWorkInstructionElementsReading: true // Performance optimization
};

// Load elements on-demand when user selects a work instruction
static async loadWorkInstructionElements(sWorkInstructionId) {
    const aElements = await ApiClient.internal.workinstruction
        .findWorkInstructionElements({
            sfcs: aSfcs,
            workInstructionId: sWorkInstructionId
        });
    return aElements.map(o => WorkInstructionElement.fromApiResponse(o));
}
```

### Duplicate Filtering

```javascript
static #filterDuplicateWorkListItems(aWorkListItems) {
    const oSeen = new Set();
    const aFiltered = [];
    
    // Iterate backwards to keep latest items
    for (let i = aWorkListItems.length - 1; i >= 0; i--) {
        const sIdentifier = aWorkListItems[i].getIdentifier();
        if (!oSeen.has(sIdentifier)) {
            aFiltered.push(aWorkListItems[i]);
            oSeen.add(sIdentifier);
        }
    }
    
    return aFiltered.reverse();
}
```

---

## Error Handling Patterns

```javascript
try {
    const aData = await this.#oPendingRequest;
    PodContext.setData(aData);
} catch (oError) {
    // Don't log AbortError (expected when cancelled)
    if (oError.name === "AbortError") {
        this.#oLog.info("Request was cancelled.");
        return;
    }
    
    this.#oLog.error(`Failed to fetch: ${oError.message}`);
    
    // Show error to user
    MessageHistory.showError(oError.message);
    
    // Clear data on error
    PodContext.setData([]);
}
```

---

## Delegate-Specific Insights

### WorkListDelegate (Most Complex)

**Features:**
- Multi-type work list (WorkCenter, OperationActivity, ProcessOrder, ProductionOrder)
- Pagination with `offset` and `size`
- Sorting support with denormalization for different APIs
- Background refresh (no UI blocking)
- Foreground refresh (clear + show busy)
- AbortController for request cancellation
- Notification subscriptions (SFC_START, SFC_SIGNOFF, SFC_COMPLETE)
- Dynamic filter updates
- Duplicate item filtering
- Conditional labored operators fetch based on widget config

**Public Methods:**
```javascript
// Main refresh - clears data, shows busy indicator
static async refresh(oOptions = {})

// Background refresh - no clearing, no busy indicator  
static async refreshInBackground(oOptions = {})

// Pagination - fetch next page and append
static async fetchNextPage()

// Selective refresh - only if list contains specific SFCs
static async refreshIfContainsSfc(aSfcs)

// Refresh only specific stale SFCs — does NOT clear non-matching items
// vSfcs: string | Array<string>
static async refreshStaleSfcs(vSfcs)

// Get last filter used
static getLatestFilter()
```

**Work List Types:**
```javascript
import WorkListType from "sap/dm/dme/pod2/enumeration/WorkListType";

switch (oWorkListFilter.workListType) {
    case WorkListType.WorkCenter:
        return fetchWorkCenterItems();
    case WorkListType.OperationActivity:
        return fetchOperationActivityItems();
    case WorkListType.ProcessOrder:
    case WorkListType.ProductionOrder:
        return fetchOrderItems();
}
```

### OperationActivityDelegate

**Features:**
- Handles both OperationActivity and Phase types
- Notification subscriptions (SFC_START, SFC_SIGNOFF, SFC_COMPLETE, OPERATION_START, OPERATION_COMPLETE)
- Auto-select next incomplete operation activity
- Prunes selected items after refresh
- Sorting by stepId
- MIN_REFRESH_INTERVAL throttling

**Public Methods:**
```javascript
// Main refresh
static async refresh(oOptions = {})

// Auto-select first incomplete operation activity
static selectNextOperationActivity()
```

### WorkInstructionDelegate

**Features:**
- Request ID tracking for race condition prevention
- AbortController for cancellation
- On-demand element loading (`skipWorkInstructionElementsReading`)
- Filter non-HEADER_TEXT work instructions
- Preserve previous selection if still valid
- Subscribe to multiple ModelPaths (SelectedWorkListItems, SelectedOperationActivities, FilterResources)

**Public Methods:**
```javascript
// Manual initialization — called automatically on first refresh if not yet initialized
static async init()

// Main refresh — fetches work instructions, shows loading state
static async refresh(oOptions = {})

// Load work instruction elements on-demand for a specific instruction
// Returns array of WorkInstructionElement
static async loadWorkInstructionElements(sWorkInstructionId)
```

### QuantityConfirmationDelegate

**Features:**
- Pagination with page/size
- AbortController support
- Subscribe to both SelectedWorkListItems and SelectedOperationActivities
- `fetchNextPage()` for infinite scrolling

**Public Methods:**
```javascript
// Main refresh
static async refresh(oOptions = {})

// Pagination - fetch next page
static async fetchNextPage()
```

### ActivityConfirmationDelegate

**Features:**
- Simple request deduplication
- Sorting by sequence and activityId
- Subscribe to SelectedOperationActivities with `force: true, clear: true`

**Public Methods:**
```javascript
// Main refresh - NOTE: actual method name is refreshActivitySummaries
static async refreshActivitySummaries(oOptions = {})
```

### QualityInspectionDelegate

**Features:**
- Minimal implementation
- Manual refresh via options
- Sets multiple PodContext properties (InspectionPoints, FieldCombinations, IsEnablePoint)

**Public Methods:**
```javascript
// Main refresh
static async refresh(oOptions = {})

// Fetch JSON data from URL
static async fetchData(sJSONUrl)
```

### GoodsReceiptDelegate

**Features:**
- Single API call returns both summary and line items
- Updates two PodContext properties
- Subscribes only to SelectedWorkListItems

**Public Methods:**
```javascript
// Main refresh - fetches summary and line items
static async refreshSummary()
```

---

## Logging Pattern

```javascript
static #oLog = Logger.getLogger("sap.dm.dme.pod2.context.data.DelegateName");

// Debug - detailed flow information
this.#oLog.debug("Request skipped. Already pending.");

// Info - successful operations
this.#oLog.info(`Fetched ${aItems.length} items`);

// Error - failures
this.#oLog.error(`Failed to fetch data: ${oError.message}`);
```

---

## Complete Delegate Template

```javascript
import Logger from "sap/dm/dme/pod2/Logger";
import Utilities from "sap/dm/dme/pod2/Utilities";
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";
import PodContext from "sap/dm/dme/pod2/context/PodContext";

class CustomDelegate {
    static #oLog = Logger.getLogger("com.company.CustomDelegate");
    static #bInitialized = false;
    static #oPendingRequest;
    static #oPreviousRequest;

    static {
        PodContext.attachInit(() => {
            this.#bInitialized = false;
            this.#oPreviousRequest = undefined;
        });
    }

    static async #init() {
        if (this.#bInitialized) return;
        this.#bInitialized = true;
        await PodContext.getPodRuntime().ready();

        if (PodContext.isRunMode()) {
            PodContext.subscribe(ModelPath.SelectedWorkListItems, 
                () => this.refresh(), this);
        } else {
            // Load design mode sample data
            const oResponse = await fetch(
                sap.ui.require.toUrl("path/to/sampleData.json")
            );
            const aData = await oResponse.json();
            PodContext.setCustomData(aData);
            this.#oLog.info("Sample data loaded.");
        }
    }

    static async refresh(oOptions = {}) {
        if (!this.#bInitialized) await this.#init();
        if (PodContext.isDesignMode()) return;

        const oRequest = this.#buildRequest();
        if (!oRequest) {
            this.#oPreviousRequest = null;
            PodContext.setCustomData([]);
            return;
        }

        // Deduplication
        if (!oOptions.force && this.#oPreviousRequest &&
            Utilities.shallowEqual(oRequest, this.#oPreviousRequest)) {
            return;
        }

        // Prevent concurrent requests
        if (!this.#oPendingRequest) {
            this.#oPreviousRequest = oRequest;
            this.#oPendingRequest = ApiClient.internal.xxx.getData(oRequest)
                .finally(() => { this.#oPendingRequest = null; });
        } else {
            this.#oLog.debug("Refresh skipped. Already pending.");
            return;
        }

        try {
            const aData = await this.#oPendingRequest;
            PodContext.setCustomData(aData);
            this.#oLog.info(`Fetched ${aData.length} items`);
        } catch (oError) {
            this.#oLog.error(`Failed: ${oError.message}`);
            PodContext.setCustomData([]);
        }
    }

    static #buildRequest() {
        const oWorkListItem = PodContext.getLastSelectedWorkListItem();
        if (!oWorkListItem) return null;

        return {
            plant: PodContext.getPlant(),
            sfc: oWorkListItem.sfc,
            shopOrder: oWorkListItem.order
        };
    }
}

export default CustomDelegate;
```

---

## Common Imports Reference

```javascript
// Core POD 2.0
import Logger from "sap/dm/dme/pod2/Logger";
import Utilities from "sap/dm/dme/pod2/Utilities";
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
import MessageHistory from "sap/dm/dme/pod2/context/MessageHistory";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";
import PodContext from "sap/dm/dme/pod2/context/PodContext";

// Notifications
import EventType from "sap/dm/dme/pod2/notification/EventType";
import Filter from "sap/dm/dme/pod2/notification/Filter";
import PodNotificationWebSocket from "sap/dm/dme/pod2/notification/PodNotificationWebSocket";

// Enumerations
import WorkListType from "sap/dm/dme/pod2/enumeration/WorkListType";
```

---

## Type System Documentation

**Typed classes used in delegates:**

- `WorkListItem` - Work list item (SFC-based)
- `OrderWorkListItem` - Order work list item
- `OperationActivity` - Active operation activity
- `Phase` - Order phase
- `ActivityConfirmationSummary` - Activity confirmation summary
- `ReportedQuantity` - Quantity confirmation entry
- `WorkInstruction` - Work instruction metadata
- `WorkInstructionElement` - Work instruction element
- `GoodsReceiptSummary` - Goods receipt summary
- `GoodsReceiptLineItem` - Goods receipt line item
- `Sorting` - Sorting criteria

---

## Key Takeaways

1. **Consistent Patterns:** All delegates follow the same lifecycle and structure
2. **Reactive Updates:** PodContext subscriptions enable automatic data refresh
3. **Performance:** Request deduplication, cancellation, and throttling prevent waste
4. **Real-Time:** WebSocket notifications keep data current
5. **Design Mode:** Preview data enables plugin development without backend
6. **Type Safety:** All API responses converted to typed objects
7. **Error Resilience:** Proper error handling and graceful degradation

---

**See Also**:
- [production-patterns-unified.md](production-patterns-unified.md) - Foundation patterns
- [widget-patterns.md](widget-patterns.md) - Widget templates with subscription patterns
- [pod2-api-reference.md](pod2-api-reference.md) - Complete API reference
