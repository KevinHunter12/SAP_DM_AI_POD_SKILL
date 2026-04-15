# SAP Digital Manufacturing POD 2.0 ApiClient Complete Reference

> **Last Updated:** 2026-04-13
> 
> **Critical Discovery:** All ApiClient methods accept a **SINGLE OBJECT parameter**, not multiple parameters.

---

## Table of Contents

1. [Overview](#overview)
2. [Critical Parameter Pattern](#critical-parameter-pattern)
3. [Public API Namespaces](#public-api-namespaces)
4. [Common Usage Patterns](#common-usage-patterns)
5. [Internal APIs (Not for Custom Plugins)](#internal-apis-not-for-custom-plugins)
6. [RestClient (Underlying HTTP Client)](#restclient-underlying-http-client)
7. [Error Handling](#error-handling)
8. [Complete Working Examples](#complete-working-examples)

---

## Overview

The `ApiClient` in POD 2.0 is the primary interface for calling SAP Digital Manufacturing APIs from custom widgets. It provides:

- **Type-safe wrappers** around SAP DM REST APIs
- **Automatic authentication** and CSRF token handling
- **Organized namespaces** (sfc, material, order, etc.)
- **Promise-based async/await** pattern
- **Automatic plant context** injection

**Import Path:**
```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
```

**Location in SAP Code:**
- Main file: `sap/dm/dme/pod2/api/ApiClient.js`
- Individual clients: `sap/dm/dme/pod2/api/public/*PublicApiClient.js`

---

## Critical Parameter Pattern

### ⚠️ MOST IMPORTANT RULE ⚠️

**ALL ApiClient methods accept a SINGLE OBJECT parameter, NOT multiple parameters.**

```javascript
// ✅ CORRECT - Single object with all parameters
await ApiClient.sfc.getSfcDetail({
    plant: sPlant,
    sfc: sSfc
});

// ❌ WRONG - Multiple parameters (will fail)
await ApiClient.sfc.getSfcDetail(sPlant, sSfc);

// ❌ WRONG - Separate plant and object (will fail)
await ApiClient.sfc.getSfcDetail(sPlant, { sfc: sSfc });
```

**Why This Pattern?**
- The underlying implementation destructures a single object: `{ plant, sfc, ...otherParams }`
- Allows optional parameters without positional argument problems
- Consistent across all ApiClient namespaces
- Matches SAP's REST API JSON body structure

---

## Public API Namespaces

These are **stable, documented APIs** for custom plugin development.

### 1. ApiClient.sfc ⭐ (Most Commonly Used)

**Type:** `SfcPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_sfc/resource/SFC_Processing

#### Methods:

##### getSfcDetail(params, options)

Gets detailed information about an SFC.

**Method Signature:**
```javascript
async getSfcDetail(c, n)
// c = parameters object
// n = options object (optional)
```

**Parameters:**
```javascript
{
    plant: string,      // Plant code (required)
    sfc: string        // SFC number (required)
}
```

**Options (optional second parameter):**
```javascript
{
    headers: { /* custom headers */ },
    // other RestClient options
}
```

**Response Structure:**
```javascript
{
    sfc: "KEVH132",
    material: {
        material: "MATERIAL1",
        version: "1",
        description: "Material 1"
    },
    status: {
        code: "402",
        description: "IN_QUEUE"
    },
    quantity: 1,
    bom: {
        bom: "BOM1",
        version: "1",
        type: "USERBOM"
    },
    routing: {
        routing: "KEVH_ROUTER1",
        version: "1",
        type: "PRODUCTION"
    },
    order: {
        order: "TRAINING123",
        orderPlannedStartDateTime: "2024-02-05T14:14:06.000+00:00",
        orderPlannedStartDateTime_Z: "2024-02-05T14:14:06Z",
        orderPlannedCompleteDateTime: "2024-02-12T14:14:12.000+00:00",
        orderPlannedCompleteDateTime_Z: "2024-02-12T14:14:12Z"
    },
    operation: {
        operation: "OP10"
    },
    workCenter: {
        workCenter: "WC_001"
    },
    steps: [
        // Array of step objects
    ],
    defaultBatchId: null
}
```

**Example Usage:**
```javascript
const oSfcDetails = await ApiClient.sfc.getSfcDetail({
    plant: "PLANT1",
    sfc: "SFC123"
});
```

---

##### getSfcs(params, options)

Gets a list of SFCs for the worklist.

**Parameters:**
```javascript
{
    plant: string,
    // Additional filter parameters
}
```

**Example:**
```javascript
const oResult = await ApiClient.sfc.getSfcs({
    plant: "PLANT1",
    material: "MAT1",
    operation: "OP10"
});
```

---

##### getSfcsCount(params, options)

Gets count of SFCs matching criteria.

**Parameters:**
```javascript
{
    plant: string,
    // Additional filter parameters
}
```

**Example:**
```javascript
const nCount = await ApiClient.sfc.getSfcsCount({
    plant: "PLANT1",
    status: "402"
});
```

---

##### sfcStart(params)

Starts SFC at an operation.

**Parameters:**
```javascript
{
    plant: string,
    sfc: string,
    operation: string,
    // Additional start parameters
}
```

**Example:**
```javascript
const oResult = await ApiClient.sfc.sfcStart({
    plant: "PLANT1",
    sfc: "SFC123",
    operation: "OP10",
    resource: "RES1"
});
```

---

##### sfcSignoff(params)

Signs off SFC from an operation.

**Parameters:**
```javascript
{
    plant: string,
    sfc: string,
    // Additional signoff parameters
}
```

**Example:**
```javascript
const oResult = await ApiClient.sfc.sfcSignoff({
    plant: "PLANT1",
    sfc: "SFC123"
});
```

---

##### sfcComplete(params)

Completes (yields) SFC.

**Parameters:**
```javascript
{
    plant: string,
    sfc: string,
    quantity: number,
    // Additional complete parameters
}
```

**Example:**
```javascript
const oResult = await ApiClient.sfc.sfcComplete({
    plant: "PLANT1",
    sfc: "SFC123",
    quantity: 10
});
```

---

##### updateSfcBatch(params)

Updates the batch ID for an SFC.

**Parameters:**
```javascript
{
    plant: string,
    sfc: string,
    batchId: string
}
```

**Example:**
```javascript
const oResult = await ApiClient.sfc.updateSfcBatch({
    plant: "PLANT1",
    sfc: "SFC123",
    batchId: "BATCH001"
});
```

---

### 2. ApiClient.assembly

**Type:** `AssemblyPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_assembly/resource/Assembly

#### Methods:

##### getAssembledComponents(params, options)

Gets components that have been assembled to an SFC.

**Example:**
```javascript
const oComponents = await ApiClient.assembly.getAssembledComponents({
    plant: "PLANT1",
    sfc: "SFC123"
});
```

---

##### getPlannedComponents(params, options)

Gets planned components for an SFC (from BOM).

**Example:**
```javascript
const oComponents = await ApiClient.assembly.getPlannedComponents({
    plant: "PLANT1",
    sfc: "SFC123",
    operation: "OP10"
});
```

---

##### assembleComponent(params, options)

Assembles a component to an SFC.

**Example:**
```javascript
const oResult = await ApiClient.assembly.assembleComponent({
    plant: "PLANT1",
    sfc: "SFC123",
    component: {
        material: "COMP1",
        quantity: 1
    }
});
```

---

### 3. ApiClient.bom

**Type:** `BomPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_bom/resource/BOM

#### Methods:

##### getBoms(params, options)

Gets BOM details.

**Parameters:**
```javascript
{
    plant: string,
    bom: string,
    version: string,
    type: string,
    readReservationsAndBatch: boolean
}
```

**Example:**
```javascript
const oBom = await ApiClient.bom.getBoms({
    plant: "PLANT1",
    bom: "BOM1",
    version: "1",
    type: "USERBOM"
});
```

---

### 4. ApiClient.datacollection

**Type:** `DataCollectionPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_datacollection/resource/Data_Collection

**Note:** Lazy-loaded module

#### Methods:

##### getDataCollectionGroups(params, options)

Gets data collection groups for an SFC.

**Example:**
```javascript
await ApiClient.ready(); // Ensure datacollection is loaded

const oGroups = await ApiClient.datacollection.getDataCollectionGroups({
    plant: "PLANT1",
    sfc: "SFC123",
    operation: "OP10"
});
```

---

##### logDataCollectionGroup(params, options)

Records data collection values.

**Example:**
```javascript
const oResult = await ApiClient.datacollection.logDataCollectionGroup({
    plant: "PLANT1",
    sfc: "SFC123",
    dataFields: [
        {
            fieldName: "TEMPERATURE",
            value: "25.5"
        }
    ]
});
```

---

### 5. ApiClient.execution

**Type:** `ExecutionPublicApiClient`

#### Methods:

##### sfcStart(params)

Starts SFC at an operation (v2 API).

**Example:**
```javascript
const oResult = await ApiClient.execution.sfcStart({
    plant: "PLANT1",
    sfcs: ["SFC123"],
    operation: "OP10",
    resource: "RES1"
});
```

---

##### sfcSignoff(params)

Signs off SFC (v2 API).

**Example:**
```javascript
const oResult = await ApiClient.execution.sfcSignoff({
    plant: "PLANT1",
    sfcs: ["SFC123"]
});
```

---

##### sfcComplete(params)

Completes/yields SFC (v2 API).

**Example:**
```javascript
const oResult = await ApiClient.execution.sfcComplete({
    plant: "PLANT1",
    sfcs: [{
        sfc: "SFC123",
        quantity: 10
    }]
});
```

---

### 6. ApiClient.inventory

**Type:** `InventoryPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_inventory/resource/Inventory

#### Methods:

##### getInventories(params, options)

Gets inventory records.

**Returns:** `[content, totalElements]` (array with data and count)

**Example:**
```javascript
const [aInventory, nTotal] = await ApiClient.inventory.getInventories({
    plant: "PLANT1",
    material: "MAT1",
    storageLocation: "LOC1"
});
```

---

##### updateInventory(params, options)

Updates inventory record.

**Example:**
```javascript
const oResult = await ApiClient.inventory.updateInventory({
    inventoryId: "INV123",
    quantity: 100
});
```

---

### 7. ApiClient.material

**Type:** `MaterialPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_material/resource/Material

#### Methods:

##### getMaterials(params, options)

Gets material details with pagination.

**Returns:** `[content, totalElements]` (array with data and count)

**Parameters:**
```javascript
{
    plant: string,
    material: string,
    version: string,
    page: number,
    size: number
}
```

**Example:**
```javascript
const [aMaterials, nTotal] = await ApiClient.material.getMaterials({
    plant: "PLANT1",
    material: "MAT*",
    page: 0,
    size: 20
});
```

---

### 8. ApiClient.order

**Type:** `OrderPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_order/resource/Order

#### Methods:

##### getOrder(params, options)

Gets shop order details.

**Example:**
```javascript
const oOrder = await ApiClient.order.getOrder({
    plant: "PLANT1",
    order: "ORDER123"
});
```

---

### 9. ApiClient.operationactivity

**Type:** `OperationActivityPublicApiClient`

#### Methods:

##### getOperationActivities(params, options)

Gets operation activities with pagination and sorting.

**Returns:** `[content, totalElements]` (array with data and count)

**Parameters:**
```javascript
{
    plant: string,
    operationActivity: string,
    version: string,
    currentVersion: boolean,
    type: string,
    resourceType: string,
    status: string,
    page: number,
    pageSize: number,
    sortBy: string,
    sortDescending: boolean
}
```

**Example:**
```javascript
const [aActivities, nTotal] = await ApiClient.operationactivity.getOperationActivities({
    plant: "PLANT1",
    operationActivity: "ACT*",
    currentVersion: true,
    page: 0,
    pageSize: 20,
    sortBy: "operation",
    sortDescending: false
});
```

---

### 10. ApiClient.processorder

**Type:** `ProcessOrderPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_processorder/resource/ProcessOrder

#### Methods:

##### goodsIssue(params, options)

Issues goods for a process order.

**Example:**
```javascript
const oResult = await ApiClient.processorder.goodsIssue({
    plant: "PLANT1",
    processOrder: "PO123",
    components: [...]
});
```

---

##### getGoodsIssueSummary(params, options)

Gets goods issue summary for a process order.

**Example:**
```javascript
const oSummary = await ApiClient.processorder.getGoodsIssueSummary({
    plant: "PLANT1",
    processOrder: "PO123"
});
```

---

### 11. ApiClient.resource

**Type:** `ResourcePublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_plant_resource_v2/resource/Resource

#### Methods:

##### getResources(params, options)

Gets plant resources.

**Example:**
```javascript
const oResources = await ApiClient.resource.getResources({
    plant: "PLANT1",
    resource: "RES*"
});
```

---

##### getResourceTypes(params, options)

Gets resource types.

**Example:**
```javascript
const oTypes = await ApiClient.resource.getResourceTypes({
    plant: "PLANT1"
});
```

---

### 12. ApiClient.uom

**Type:** `UomPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_uom/resource/UOM

#### Methods:

##### getUom(params, options)

Gets unit of measure details.

**Example:**
```javascript
const oUom = await ApiClient.uom.getUom({
    unitCode: "EA"
});
```

---

### 13. ApiClient.workcenter

**Type:** `WorkCenterPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_plant_workcenter_v2/resource/Work_Center

#### Methods:

##### getWorkCenters(params, options)

Gets work center details.

**Parameters:**
```javascript
{
    plant: string,
    workCenter: string,
    assignedUser: string,
    resourceMembers: string[] // Will be joined with comma
}
```

**Example:**
```javascript
const oWorkCenters = await ApiClient.workcenter.getWorkCenters({
    plant: "PLANT1",
    workCenter: "WC*",
    assignedUser: "USER1"
});
```

---

### 14. ApiClient.workinstruction

**Type:** `WorkInstructionPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_workinstruction/resource/Work_Instructions

#### Methods:

##### getWorkInstructions(params, options)

Gets work instructions attached to a context.

**Example:**
```javascript
const oInstructions = await ApiClient.workinstruction.getWorkInstructions({
    plant: "PLANT1",
    operation: "OP10",
    material: "MAT1"
});
```

---

### 15. ApiClient.alert

**Type:** `AlertPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_alerts/resource/Alerts

#### Methods:

##### createAlert(params, options)

Creates an alert.

**Note:** Requires `Alert-Type` header in options.

**Example:**
```javascript
const oResult = await ApiClient.alert.createAlert({
    plant: "PLANT1",
    message: "Critical issue detected"
}, {
    headers: {
        "Alert-Type": "CRITICAL"
    }
});
```

---

### 16. ApiClient.mdo

**Type:** `MdoApiClient`

MDO (Manufacturing Data Object) APIs for direct OData access.

#### Methods:

##### getWorkCentersMDO(params)

Gets work centers via MDO OData.

**Example:**
```javascript
const aWorkCenters = await ApiClient.mdo.getWorkCentersMDO({
    plant: "PLANT1",
    workCenter: "WC*"
});
```

---

##### getResourcesMDO(params)

Gets resources via MDO OData.

**Example:**
```javascript
const aResources = await ApiClient.mdo.getResourcesMDO({
    plant: "PLANT1",
    resource: "RES*"
});
```

---

##### getOperationActivitiesMDO(params)

Gets operation activities via MDO OData with pagination.

**Parameters:**
```javascript
{
    plant: string,
    operationActivity: string,
    skip: number,
    top: number
}
```

**Example:**
```javascript
const aActivities = await ApiClient.mdo.getOperationActivitiesMDO({
    plant: "PLANT1",
    operationActivity: "ACT*",
    skip: 0,
    top: 100
});
```

---

### 17. ApiClient.user

**Type:** `UserPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_user/resource/User

---

### 18. ApiClient.datafields

**Type:** `DataFieldsPublicApiClient`  
**SAP API Hub:** https://api.sap.com/api/sapdme_datafields/path/getDataFields

---

## Common Usage Patterns

### Pattern 1: Get Context, Call API, Display Result

```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import MessageToast from "sap/m/MessageToast";

async _onButtonPress() {
    const oButton = this.getView();

    try {
        // Disable button during API call
        oButton.setEnabled(false);

        // Step 1: Get context
        const sPlant = PodContext.getPlant();
        const oWorkListItem = PodContext.getLastSelectedWorkListItem();

        if (!oWorkListItem || !oWorkListItem.sfc) {
            MessageToast.show("No SFC selected");
            return;
        }

        // Step 2: Call API
        const oSfcDetails = await ApiClient.sfc.getSfcDetail({
            plant: sPlant,
            sfc: oWorkListItem.sfc
        });

        // Step 3: Display result
        MessageToast.show(`SFC Status: ${oSfcDetails.status.description}`);

    } catch (oError) {
        console.error("Error calling API", oError);
        MessageToast.show(`Error: ${oError.message}`);
    } finally {
        // Always re-enable button
        oButton.setEnabled(true);
    }
}
```

---

### Pattern 2: Handling Nested Object Properties

API responses often contain nested objects. Always check types before accessing:

```javascript
// Helper function to safely extract nested properties
const extractValue = (obj, propName) => {
    if (!obj) return null;
    if (typeof obj === 'object') {
        return obj[propName] || null;
    }
    return obj;
};

// Usage
const sMaterial = extractValue(oSfcDetails.material, 'material');
const sDescription = extractValue(oSfcDetails.material, 'description');
const sMaterialText = sMaterial
    ? (sDescription ? `${sMaterial} - ${sDescription}` : sMaterial)
    : 'Unknown';
```

---

### Pattern 3: Display Results in Popover

```javascript
import Popover from "sap/m/Popover";
import VBox from "sap/m/VBox";
import Text from "sap/m/Text";

_createDetailsPopover(oSfcDetails) {
    const aTexts = [];

    // SFC
    if (oSfcDetails.sfc) {
        aTexts.push(new Text({
            text: `SFC: ${oSfcDetails.sfc}`
        }).addStyleClass("sapUiTinyMarginBottom"));
    }

    // Material with description
    if (oSfcDetails.material) {
        const sMaterial = oSfcDetails.material.material || "Unknown";
        const sDesc = oSfcDetails.material.description || "";
        const sMaterialText = sDesc ? `${sMaterial} - ${sDesc}` : sMaterial;
        aTexts.push(new Text({
            text: `Material: ${sMaterialText}`
        }).addStyleClass("sapUiTinyMarginBottom"));
    }

    // Status with description
    if (oSfcDetails.status) {
        const sCode = oSfcDetails.status.code || "";
        const sDesc = oSfcDetails.status.description || "";
        const sStatusText = sCode && sDesc ? `${sCode} - ${sDesc}` : (sCode || sDesc || "Unknown");
        aTexts.push(new Text({
            text: `Status: ${sStatusText}`
        }).addStyleClass("sapUiTinyMarginBottom"));
    }

    const oContent = new VBox({
        items: aTexts
    }).addStyleClass("sapUiSmallMargin");

    return new Popover({
        title: "SFC Details",
        contentWidth: "500px",
        content: [oContent],
        placement: "Bottom"
    });
}

// Usage
const oPopover = this._createDetailsPopover(oSfcDetails);
oPopover.openBy(oButton);
```

---

### Pattern 4: Lazy-Loaded Modules

Some ApiClient namespaces (like `datacollection`) are lazy-loaded. Use `ApiClient.ready()`:

```javascript
async _initialize() {
    // Wait for all API clients to load
    await ApiClient.ready();

    // Now safe to use datacollection
    const oResult = await ApiClient.datacollection.recordData({
        plant: sPlant,
        sfc: sSfc,
        dataFields: [...]
    });
}
```

---

## Internal APIs (Not for Custom Plugins)

⚠️ **Warning:** Internal APIs are for SAP standard widgets only and can change without notice. **Do not use in custom plugins.**

### Internal Namespaces:

1. `ApiClient.internal.activityconfirmation`
2. `ApiClient.internal.alert`
3. `ApiClient.internal.assembly`
4. `ApiClient.internal.datacollection`
5. `ApiClient.internal.datascan`
6. `ApiClient.internal.demand`
7. `ApiClient.internal.document`
8. `ApiClient.internal.inventory`
9. `ApiClient.internal.oee`
10. `ApiClient.internal.plant`
11. `ApiClient.internal.pod`
12. `ApiClient.internal.portal`
13. `ApiClient.internal.processengine`
14. `ApiClient.internal.product`
15. `ApiClient.internal.qualityInspection`
16. `ApiClient.internal.serviceregistry`
17. `ApiClient.internal.sfc`
18. `ApiClient.internal.signature`
19. `ApiClient.internal.timetracking`
20. `ApiClient.internal.workinstruction`
21. `ApiClient.internal.worklist`

---

## RestClient (Underlying HTTP Client)

ApiClient uses `RestClient` internally. You can also use RestClient directly for custom endpoints:

**Import:**
```javascript
import RestClient from "sap/dm/dme/pod2/api/RestClient";
```

**Methods:**
```javascript
// GET request
const oResponse = await RestClient.get(
    "/dm/api/v1/custom/endpoint",
    { param1: "value1" }, // Query params
    { /* options */ }
);

// POST request
const oResponse = await RestClient.post(
    "/dm/api/v1/custom/endpoint",
    { key: "value" }, // Request body
    { /* options */ }
);

// PUT request
const oResponse = await RestClient.put(url, body, options);

// PATCH request
const oResponse = await RestClient.patch(url, body, options);

// DELETE request
const oResponse = await RestClient.delete(url, options);

// Custom fetch
const oResponse = await RestClient.fetch(url, {
    method: "GET",
    headers: { "Custom-Header": "value" }
});
```

**Features:**
- Automatic CSRF token handling
- Session timeout detection
- Custom SAP DM headers (`x-dme-plant`, `x-dme-industry-type`)
- Automatic JSON parsing
- Error response handling with `ApiError` class

**When to Use RestClient vs ApiClient:**
- ✅ Use **ApiClient** for standard SAP DM APIs (recommended)
- ✅ Use **RestClient** for custom backend endpoints or Process Engine APIs
- ❌ Don't use RestClient for standard SAP DM APIs (use ApiClient instead)

---

## Error Handling

### ApiError Structure

```javascript
class ApiError extends Error {
    constructor(message, response, status, requestUrl) {
        this.message = message;
        this.response = response; // Full response object
        this.status = status;     // HTTP status code
        this.requestUrl = requestUrl;
    }
}
```

### Best Practices

```javascript
async _callApi() {
    try {
        const oResult = await ApiClient.sfc.getSfcDetail({
            plant: sPlant,
            sfc: sSfc
        });

        return oResult;

    } catch (oError) {
        // Log full error for debugging
        console.error("API call failed", oError);

        // Check error type
        if (oError.status === 404) {
            MessageToast.show("SFC not found");
        } else if (oError.status === 403) {
            MessageToast.show("Permission denied");
        } else {
            // Generic error message
            const sMessage = oError.message || "Unknown error occurred";
            MessageToast.show(`Error: ${sMessage}`);
        }

        // Optionally re-throw
        throw oError;
    }
}
```

---

## Complete Working Examples

### Example 1: Simple SFC Details Button

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/api/ApiClient",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/m/Button",
    "sap/m/MessageToast"
], (
    ApiClient,
    PodContext,
    ControlWidget,
    Button,
    MessageToast
) => {
    "use strict";

    class SfcDetailsWidget extends ControlWidget {

        constructor(oConfig) {
            super(Button, oConfig);
        }

        _createView() {
            const oButton = super._createView();
            oButton.setText("Show SFC Details");
            oButton.attachPress(this._onButtonPress.bind(this));
            return oButton;
        }

        async _onButtonPress(oEvent) {
            const oButton = this.getView();

            try {
                oButton.setEnabled(false);

                const sPlant = PodContext.getPlant();
                const oWorkListItem = PodContext.getLastSelectedWorkListItem();

                if (!oWorkListItem || !oWorkListItem.sfc) {
                    MessageToast.show("No SFC selected");
                    return;
                }

                const oSfcDetails = await ApiClient.sfc.getSfcDetail({
                    plant: sPlant,
                    sfc: oWorkListItem.sfc
                });

                const sMessage = `SFC: ${oSfcDetails.sfc}\n` +
                    `Material: ${oSfcDetails.material.material}\n` +
                    `Status: ${oSfcDetails.status.description}\n` +
                    `Quantity: ${oSfcDetails.quantity}`;

                MessageToast.show(sMessage);

            } catch (oError) {
                console.error("Error getting SFC details", oError);
                MessageToast.show(`Error: ${oError.message}`);
            } finally {
                oButton.setEnabled(true);
            }
        }
    }

    return SfcDetailsWidget;
});
```

---

### Example 2: SFC Details with Popover (Full Implementation)

See: `C:\VSCodeProjects\newskilltest_proper\plugins\NewSkillTestWidget.js`

This is a complete, production-ready example showing:
- Proper error handling
- Button state management
- Nested object property handling
- Formatted popover display
- All SAP UI5 best practices

---

## Key Takeaways

1. **ALWAYS use single object parameter** format for all ApiClient methods
2. **Use ApiClient** (not RestClient) for standard SAP DM APIs
3. **Get plant from PodContext**, not from user input
4. **Handle nested object properties** defensively with type checks
5. **Disable buttons during API calls** to prevent double-clicks
6. **Use try/catch/finally** for proper error and state management
7. **Consult SAP API Hub** for detailed API documentation
8. **Use PUBLIC APIs only** - internal APIs can change without notice
9. **Use await ApiClient.ready()** before lazy-loaded modules like datacollection
10. **Log errors** to console for debugging, show user-friendly messages to users

---

## Additional Resources

- **SAP API Hub:** https://api.sap.com (search for "sapdme")
- **ApiClient Source:** `sap/dm/dme/pod2/api/ApiClient.js`
- **RestClient Source:** `sap/dm/dme/pod2/api/RestClient.js`
- **Working Example:** `C:\VSCodeProjects\newskilltest_proper\plugins\NewSkillTestWidget.js`
- **Browser Inspector:** `C:\VSCodeProjects\newskilltest_proper\inspect_apiclient.js`
