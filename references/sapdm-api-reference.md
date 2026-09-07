# SAP Digital Manufacturing API Reference

**Version**: 1.1.0  
**Last Updated**: 2026-05-17  
**API Documentation**: [SAP API Business Hub](https://api.sap.com/package/SAPDigitalManufacturingCloud)

This document provides a comprehensive reference to all SAP Digital Manufacturing REST APIs available for POD plugin integration.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Authentication & Base URLs](#authentication--base-urls)
3. [Core Production APIs](#core-production-apis)
4. [Material & BOM APIs](#material--bom-apis)
5. [Data Collection & Quality APIs](#data-collection--quality-apis)
6. [Inventory & Logistics APIs](#inventory--logistics-apis)
7. [Process Manufacturing APIs](#process-manufacturing-apis)
8. [Configuration & Master Data APIs](#configuration--master-data-apis)
9. [Integration & Document APIs](#integration--document-apis)
10. [Common Patterns & Examples](#common-patterns--examples)

---

## Overview

SAP Digital Manufacturing provides 70+ REST APIs covering all aspects of manufacturing execution. These APIs use OAuth 2.0 authentication and follow consistent patterns for requests and responses.

### API Categories

**Production Execution** (15 APIs)
- Shop Floor Control (SFC), Orders, Operations, Assembly, Activity/Quantity Confirmation

**Material Management** (12 APIs)
- Materials, BOMs, Routings, Batches, Inventory, Staging

**Quality & Data Collection** (8 APIs)
- Data Collection, Quality Inspection, Nonconformance, Electronic Batch Records (EBR)

**Process Manufacturing** (6 APIs)
- Process Orders, Process Lots, Recipes, Setpoints

**Configuration** (15 APIs)
- Resources, Work Centers, Tools, Shifts, Users, POD Configuration

**Integration** (14 APIs)
- Documents, Printing, Work Instructions, Notifications, Integration Messages

---

## Authentication & Base URLs

### OAuth 2.0 Authentication

All APIs use OAuth 2.0 Client Credentials flow:

```http
POST https://{subdomain}.authentication.{tokenHost}/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id={clientId}&client_secret={clientSecret}
```

### Region Hosts

**Production Environments:**
- `eu10.dmc.cloud.sap` - Europe (Frankfurt)
- `eu20.dmc.cloud.sap` - Europe (Netherlands)
- `us10.dmc.cloud.sap` - US East (Virginia)
- `us20.dmc.cloud.sap` - US West (Washington)

**Test Environments:**
- `test.eu10.dmc.cloud.sap`
- `test.eu20.dmc.cloud.sap`
- `test.us10.dmc.cloud.sap`
- `test.us20.dmc.cloud.sap`

### Token Hosts

- `eu10.hana.ondemand.com`
- `eu20.hana.ondemand.com`
- `us10.hana.ondemand.com`
- `us20.hana.ondemand.com`

### Base URL Pattern

```
https://api.{regionHost}/{service}/v{version}
```

**Example:**
```
https://api.eu10.dmc.cloud.sap/sfc/v1
https://api.us10.dmc.cloud.sap/material/v1
```

---

## Core Production APIs

### 1. Shop Floor Control (SFC) API

**Spec**: `sapdme_sfc.json`, `sapdme_sfc_v2.json`  
**Base URL**: `https://api.{regionHost}/sfc/v1`  
**Description**: Track materials throughout production. Start, complete, serialize, relabel, split, merge, scrap SFCs.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/sfcs/start` | Start one or more SFCs at operation/resource |
| POST | `/sfcs/complete` | Complete SFCs at operation |
| POST | `/sfcs/serialize` | Serialize SFCs with serial numbers |
| POST | `/sfcs/relabel` | Relabel SFCs |
| POST | `/sfcs/split` | Split SFC into multiple SFCs |
| POST | `/sfcs/merge` | Merge multiple SFCs into one |
| POST | `/sfcs/scrap` | Scrap SFCs |
| POST | `/sfcs/quantity` | Update SFC quantity |
| GET | `/sfcs` | Get SFC details |

**Common Request Pattern (ApiClient.sfc.sfcStart):**
```json
{
  "plant": "PLANT_1",
  "sfcs": ["SFC001"],
  "operation": "OPER_1,1",
  "resource": "RESOURCE_1",
  "autoAssembleEnabled": true
}
```

**Note**: 
- The `sfcs` property is an **array of strings**, not an array of objects
- Use `operation` (not `operationActivity`) - ApiClient differs from REST API
- `plant` IS required (use `PodContext.getPlant()`)
- `autoAssembleEnabled` defaults to true

---

### 2. Order API

**Spec**: `sapdme_order.json`, `sapdme_order_v2.json`  
**Base URL**: `https://api.{regionHost}/order/v1`  
**Description**: Define material to produce, plant location, production dates, quantities, components, and operation sequences.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v1/orders` | Find order by plant and order number |
| GET | `/v1/orders/list` | Retrieve order list with filters |
| POST | `/v1/orders/release` | Release orders for production |
| PATCH | `/v1/orders/customValues` | Update order custom values |

**Example - Find Order:**
```http
GET /v1/orders?plant=PLANT_1&order=ORDER001
```

---

### 3. Activity Confirmation API

**Spec**: `sapdme_activityConfirmation.json`  
**Base URL**: `https://api.{regionHost}/activity/v1`  
**Description**: Confirm activities performed on SFCs (labor, yield, scrap, rework).

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/confirm` | Confirm activity for SFCs |
| GET | `/activityConfirmation` | Get activity confirmation details |
| DELETE | `/activityConfirmation` | Delete activity confirmation |

---

### 4. Quantity Confirmation API

**Spec**: `sapdme_quantityConfirmation.json`  
**Base URL**: `https://api.{regionHost}/quantityConfirmation/v1`  
**Description**: Confirm quantities (yield, scrap, rework) at operations.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/confirm` | Confirm quantities for operation |
| GET | `/quantityConfirmation` | Get quantity confirmation details |

---

### 5. Assembly API

**Spec**: `sapdme_assembly.json`  
**Base URL**: `https://api.{regionHost}/assembly/v1`  
**Description**: Assemble components to SFCs during production.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/assemble` | Assemble components to SFC |
| POST | `/unassemble` | Remove assembled components |
| GET | `/assembled` | Get assembled components |

---

### 6. Operation API

**Spec**: `sapdme_operation.json`, `sapdme_operationactivity.json`  
**Base URL**: `https://api.{regionHost}/operation/v1`  
**Description**: Define manufacturing operations and activities.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/operations` | Find operation by plant and name |
| GET | `/operations/list` | List operations with filters |

---

### 7. Production API v2

**Spec**: `sapdme_production_v2.json`  
**Base URL**: `https://api.{regionHost}/production/v2`  
**Description**: Comprehensive production operations endpoint.

---

## Material & BOM APIs

### 8. Material API

**Spec**: `sapdme_material.json`  
**Base URL**: `https://api.{regionHost}/material/v1`  
**Description**: Create, search, and update materials with routing, BOM, storage location, lot size, UOM, custom values.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v1/materials` | Find material by plant, name, version |
| POST | `/v1/materials` | Create material |
| PATCH | `/v1/materials` | Update material |
| DELETE | `/v1/materials` | Delete material |
| GET | `/v1/materials/list` | List materials with filters |

**Example - Find Material:**
```http
GET /v1/materials?plant=PLANT_1&material=MAT001&version=A
```

**Response:**
```json
{
  "plant": "PLANT_1",
  "material": "MAT001",
  "version": "A",
  "description": "Material Description",
  "materialType": "FERT",
  "baseUnitOfMeasure": "EA",
  "routing": {
    "routing": "ROUTING_1",
    "routingType": "PRODUCTION",
    "version": "1"
  }
}
```

---

### 9. Bill of Material (BOM) API

**Spec**: `sapdme_bom.json`  
**Base URL**: `https://api.{regionHost}/bom/v2`  
**Description**: Define material components required for production.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/boms` | Find BOM by plant, name, version |
| POST | `/boms` | Create BOM |
| PATCH | `/boms` | Update BOM |
| DELETE | `/boms` | Delete BOM |

---

### 10. Routing API

**Spec**: `sapdme_routing.json`  
**Base URL**: `https://api.{regionHost}/routing/v1`  
**Description**: Define operation sequences for production.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/routings` | Find routing by plant, name, version |
| GET | `/routings/list` | List routings with filters |

---

### 11. Batch API

**Spec**: `sapdme_batch.json`, `sapdme_batch_v2.json`  
**Base URL**: `https://api.{regionHost}/batch/v1`  
**Description**: Manage material batches and batch traceability.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/batches` | Find batch by plant, material, batch number |
| POST | `/batches` | Create batch |
| PATCH | `/batches` | Update batch |

---

### 12. Material Group API

**Spec**: `sapdme_materialgroup.json`  
**Base URL**: `https://api.{regionHost}/materialgroup/v1`  
**Description**: Group materials for easier management.

---

## Data Collection & Quality APIs

### 13. Data Collection API

**Spec**: `sapdme_datacollection.json`  
**Base URL**: `https://api.{regionHost}/datacollection/v1`  
**Description**: Collect data values at various manufacturing process points. Create, retrieve, update data collection groups with parameters and attachments.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/log` | Log data collection parameter values |
| POST | `/standalone/log` | Log standalone/non-WIP data collection |
| GET | `/dataCollectionGroups` | Get data collection groups |
| POST | `/dataCollectionGroups` | Create data collection group |
| PATCH | `/dataCollectionGroups` | Update data collection group |

**Example - Log Data:**
```json
{
  "plant": "PLANT_1",
  "sfc": "SFC001",
  "operation": "OPER_1",
  "resource": "RESOURCE_1",
  "dataCollectionGroup": "DCG_001",
  "parameters": [
    {
      "parameterName": "TEMPERATURE",
      "value": "150"
    }
  ]
}
```

---

### 14. Quality Inspection API

**Spec**: `sapdme_qualityinspection.json`, `sapdme_qualityinspection_v2.json`  
**Base URL**: `https://api.{regionHost}/qualityinspection/v1`  
**Description**: Create and manage quality inspections for SFCs.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/inspections` | Create quality inspection |
| GET | `/inspections` | Get inspection details |
| PATCH | `/inspections` | Update inspection |

---

### 15. Nonconformance API

**Spec**: `sapdme_nonconformance.json`  
**Base URL**: `https://api.{regionHost}/nonconformance/v1`  
**Description**: Report and manage nonconformances (defects, issues).

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/nonconformances` | Create nonconformance |
| GET | `/nonconformances` | Get nonconformance details |
| PATCH | `/nonconformances` | Update nonconformance |

---

### 16. Nonconformance Code/Group APIs

**Specs**: `sapdme_nonconformancecode.json`, `sapdme_nonconformancegroup.json`  
**Description**: Define and manage nonconformance codes and groups.

---

### 17. Electronic Batch Record (EBR) API

**Spec**: `sapdme_ebr.json`  
**Base URL**: `https://api.{regionHost}/ebr/v1`  
**Description**: Create and manage electronic batch records for regulated industries.

---

### 18. Classification API

**Spec**: `sapdme_classification.json`  
**Base URL**: `https://api.{regionHost}/classification/v1`  
**Description**: Classify materials and objects with characteristics.

---

### 19. Data Fields API

**Spec**: `sapdme_datafields.json`  
**Base URL**: `https://api.{regionHost}/datafield/v1`  
**Description**: Define custom data fields for various entities.

---

### 20. Data Type API

**Spec**: `sapdme_datatype.json`  
**Base URL**: `https://api.{regionHost}/datatype/v1`  
**Description**: Define custom data types.

---

## Inventory & Logistics APIs

### 21. Inventory API

**Spec**: `sapdme_inventory.json`, `sapdme_inventory_v2.json`  
**Base URL**: `https://api.{regionHost}/inventory/v1`  
**Description**: Manage inventory levels, locations, and movements.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/inventory` | Get inventory details |
| POST | `/inventory/consume` | Consume inventory |
| POST | `/inventory/produce` | Produce inventory |
| POST | `/inventory/transfer` | Transfer inventory |

---

### 22. Staging API

**Spec**: `sapdme_staging.json`, `sapdme_staging_v2.json`  
**Base URL**: `https://api.{regionHost}/staging/v1`  
**Description**: Stage materials for production operations.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/stage` | Stage materials |
| POST | `/unstage` | Remove staged materials |
| GET | `/staged` | Get staged materials |

---

### 23. Logistics API

**Spec**: `sapdme_logistics.json`  
**Base URL**: `https://api.{regionHost}/logistics/v1`  
**Description**: Manage logistics operations and material movements.

---

### 24. Packing Unit API

**Spec**: `sapdme_packingunit.json`  
**Base URL**: `https://api.{regionHost}/packingunit/v1`  
**Description**: Create and manage packing units for materials.

---

### 25. WIP (Work in Process) API

**Spec**: `sapdme_wip.json`  
**Base URL**: `https://api.{regionHost}/wip/v1`  
**Description**: Track work in process inventory.

---

## Process Manufacturing APIs

### 26. Process Order API

**Spec**: `sapdme_processorder.json`, `sapdme_processorder_v2.json`  
**Base URL**: `https://api.{regionHost}/processorder/v1`  
**Description**: Manage process manufacturing orders.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/processOrders` | Find process order |
| POST | `/processOrders/release` | Release process order |

---

### 27. Process Lot API

**Spec**: `sapdme_processlot.json`, `sapdme_processlot_v2.json`  
**Base URL**: `https://api.{regionHost}/processlot/v1`  
**Description**: Manage process manufacturing lots.

---

### 28. Recipe API

**Spec**: `sapdme_recipe.json`  
**Base URL**: `https://api.{regionHost}/recipe/v1`  
**Description**: Define and manage manufacturing recipes.

---

### 29. Setpoint API v3

**Spec**: `sapdme_setpoint_v3.json`  
**Base URL**: `https://api.{regionHost}/setpoint/v3`  
**Description**: Manage equipment setpoints for process control.

---

### 30. Process Manufacturing API

**Spec**: `sapdme_process_manufacturing.json`  
**Base URL**: `https://api.{regionHost}/process-manufacturing/v1`  
**Description**: General process manufacturing operations.

---

### 31. REO (Recipe Execution Order) API

**Spec**: `sapdme_reo.json`  
**Base URL**: `https://api.{regionHost}/reo/v1`  
**Description**: Execute recipes in specific order.

---

## Configuration & Master Data APIs

### 32. Plant API

**Spec**: `sapdme_plant.json`  
**Base URL**: `https://api.{regionHost}/plant/v1`  
**Description**: Manage plant master data.

---

### 33. Plant Resource API v2

**Spec**: `sapdme_plant_resource_v2.json`  
**Base URL**: `https://api.{regionHost}/resource/v2`  
**Description**: Define and manage plant resources (machines, equipment).

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/resources` | Find resource by plant and name |
| GET | `/resources/list` | List resources with filters |

---

### 34. Work Center API

**Spec**: `sapdme_plant_workcenter_v2.json`, `sapdme_plant_workcenter_v3 (1).json`  
**Base URL**: `https://api.{regionHost}/workcenter/v2`  
**Description**: Define and manage work centers.

---

### 35. Resource Type API

**Spec**: `sapdme_resourcetype.json`  
**Base URL**: `https://api.{regionHost}/resourcetype/v1`  
**Description**: Define resource types for classification.

---

### 36. Tool API

**Spec**: `sapdme_tool.json`, `sapdme_tool_v2.json`  
**Base URL**: `https://api.{regionHost}/tool/v1`  
**Description**: Manage tools used in production.

---

### 37. Shift API

**Spec**: `sapdme_shift.json`  
**Base URL**: `https://api.{regionHost}/shift/v1`  
**Description**: Define production shifts and calendars.

---

### 38. User API

**Spec**: `sapdme_user.json`  
**Base URL**: `https://api.{regionHost}/user/v1`  
**Description**: Manage user accounts and permissions.

---

### 39. POD Configuration API

**Spec**: `sapdme_pod.json`  
**Base URL**: `https://api.{regionHost}/pod/v1`  
**Description**: Create, import, export POD configurations.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/configurations` | Create POD configuration |
| GET | `/export` | Export POD configuration |
| POST | `/import` | Import POD configuration |

---

### 40. Unit of Measure (UOM) API

**Spec**: `sapdme_uom.json`  
**Base URL**: `https://api.{regionHost}/uom/v1`  
**Description**: Manage units of measure.

---

### 41. Numbering API

**Spec**: `sapdme_numbering.json`, `sapdme_numbering_identifier_config.json`  
**Base URL**: `https://api.{regionHost}/numbering/v1`  
**Description**: Configure number ranges for entities.

---

### 42. Standard Rate/Value APIs

**Specs**: `sapdme_standardrate.json`, `sapdme_standardvalue.json`  
**Description**: Define standard rates and values for costing.

---

### 43. Plant Certification API

**Spec**: `sapdme_plant_certification.json`  
**Description**: Manage plant certifications.

---

### 44. Asset Model API

**Spec**: `sapdme_asset_model.json`  
**Base URL**: `https://api.{regionHost}/asset-model/v1`  
**Description**: Define asset models for equipment.

---

### 45. Labor API

**Spec**: `sapdme_labor.json`  
**Base URL**: `https://api.{regionHost}/labor/v1`  
**Description**: Track labor time and activities.

---

### 46. Time Tracking API

**Spec**: `sapdme_timetracking.json`  
**Base URL**: `https://api.{regionHost}/timetracking/v1`  
**Description**: Track time for operations and activities.

---

### 47. Last Indicator API

**Spec**: `sapdme_lastindicator.json`  
**Base URL**: `https://api.{regionHost}/lastindicator/v1`  
**Description**: Mark items as last in sequence.

---

### 48. OEE (Overall Equipment Effectiveness) APIs

**Specs**: `sapdme_oee.json`, `sapdme_oee_resourcereasoncode.json`  
**Base URL**: `https://api.{regionHost}/oee/v1`  
**Description**: Calculate and track OEE metrics and reason codes.

---

## Integration & Document APIs

### 49. Document API v2

**Spec**: `sapfnd_document_v2.json`  
**Base URL**: `https://api.{regionHost}/document/v2`  
**Description**: Attach and manage documents (files) to entities.

**Key Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/documents` | Upload document |
| GET | `/documents` | Get document |
| DELETE | `/documents` | Delete document |

---

### 50. Work Instruction API

**Spec**: `sapdme_workinstruction.json`, `sapdme_workinstruction_file.json`  
**Base URL**: `https://api.{regionHost}/workinstruction/v1`  
**Description**: Create and attach work instructions to operations.

---

### 51. Print API

**Spec**: `sapfnd_print.json`  
**Base URL**: `https://api.{regionHost}/print/v1`  
**Description**: Print labels and documents.

---

### 52. Printer API

**Spec**: `sapfnd_printer.json`  
**Base URL**: `https://api.{regionHost}/printer/v1`  
**Description**: Manage printer configurations.

---

### 53. Notification API

**Spec**: `sapdme_notification.json`  
**Base URL**: `https://api.{regionHost}/notification/v1`  
**Description**: Send and manage notifications to users.

---

### 54. Integration Message API

**Spec**: `sapdme_integrationMessage.json`  
**Base URL**: `https://api.{regionHost}/integration/v1`  
**Description**: Send and receive integration messages.

---

## Common Patterns & Examples

### Error Response Structure

All APIs return consistent error responses:

```json
{
  "code": "400",
  "message": "The HTTP request is bad or invalid",
  "causeMessage": "Material MAT001 not found",
  "correlationId": "12345-67890-abcde"
}
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 500 | Internal Server Error |

---

### Example: Calling API from POD Widget

```javascript
import PodContext from "sap/dm/dme/pod2/context/PodContext";

class MyWidget extends Widget {
    
    async _fetchSfcDetails(sSfc) {
        const oContext = PodContext.getContext();
        const sPlant = oContext.plant;
        const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
        
        try {
            const oResponse = await fetch(`${sBaseUrl}/sfcs?plant=${sPlant}&sfc=${sSfc}`, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                    "Authorization": `Bearer ${oContext.token}`
                }
            });
            
            if (!oResponse.ok) {
                throw new Error(`HTTP ${oResponse.status}: ${oResponse.statusText}`);
            }
            
            return await oResponse.json();
        } catch (oError) {
            console.error("Failed to fetch SFC:", oError);
            throw oError;
        }
    }
}
```

---

### Example: Using jQuery Ajax (SAPUI5 Pattern)

```javascript
_logDataCollection(oData) {
    const oContext = PodContext.getContext();
    const sUrl = `${oContext.serviceRegistry.getApiUrl("datacollection")}/log`;
    
    return new Promise((resolve, reject) => {
        jQuery.ajax({
            url: sUrl,
            method: "POST",
            contentType: "application/json",
            data: JSON.stringify(oData),
            headers: {
                "Authorization": `Bearer ${oContext.token}`
            },
            success: (oResponse) => resolve(oResponse),
            error: (oError) => reject(oError)
        });
    });
}
```

---

### Example: Async Endpoints

Some APIs support async processing (SFC operations, data collection):

```javascript
// Request with async=true
const oResponse = await fetch(`${sBaseUrl}/sfcs/start?async=true`, {
    method: "POST",
    body: JSON.stringify(oStartRequest)
});

// Response includes asyncExecutionId
const { asyncExecutionId } = await oResponse.json();

// Poll for results
const oResult = await this._pollAsyncResult(asyncExecutionId);
```

---

### Example: Pagination

APIs with list endpoints support pagination:

```http
GET /v1/materials/list?plant=PLANT_1&page=0&size=20
```

Response includes:
```json
{
  "content": [...],
  "page": 0,
  "size": 20,
  "totalElements": 150,
  "totalPages": 8
}
```

---

## SAP API Business Hub Reference

All SAP Digital Manufacturing API specifications are available on the SAP API Business Hub:

### REST APIs
**URL**: [https://api.sap.com/package/SAPDigitalManufacturingCloud/rest](https://api.sap.com/package/SAPDigitalManufacturingCloud/rest)

| Category | APIs |
|----------|------|
| **Production** | SFC, Order, Operation, Activity Confirmation, Quantity Confirmation |
| **Material** | Material, BOM, Routing, Batch, Inventory |
| **Quality** | Data Collection, Quality Inspection, Nonconformance, EBR |
| **Process Mfg** | Process Order, Process Lot, Recipe, Setpoint |
| **Configuration** | Plant, Resource, Work Center, Shift, Tool, User |
| **Integration** | Document, Print, Work Instruction, Notification |

### OData v4 APIs
**URL**: [https://api.sap.com/package/SAPDigitalManufacturingCloud/odatav4](https://api.sap.com/package/SAPDigitalManufacturingCloud/odatav4)

### Benefits of SAP API Hub
- Always up-to-date specifications
- Interactive API testing ("Try Out")
- Sandbox environment available
- Download OpenAPI/Swagger specs on demand
- Authentication documentation included

---

## Best Practices

### 1. Authentication

✅ **DO:**
- Cache OAuth tokens until expiry
- Use token refresh before expiry
- Handle 401 responses by refreshing token

❌ **DON'T:**
- Request new token for every API call
- Hardcode credentials in widget code

---

### 2. Error Handling

✅ **DO:**
```javascript
try {
    const oResponse = await this._callApi(sUrl, oData);
    return oResponse;
} catch (oError) {
    console.error("API call failed:", oError);
    MessageToast.show(this.getI18nText("api.error"));
    throw oError;
}
```

❌ **DON'T:**
```javascript
// Silent failures
const oResponse = await this._callApi(sUrl, oData).catch(() => {});
```

---

### 3. Performance

✅ **DO:**
- Use list endpoints with pagination
- Request only needed fields (use `expand` parameter)
- Cache frequently accessed data
- Use async endpoints for long operations

❌ **DON'T:**
- Fetch all records without pagination
- Make redundant API calls
- Poll sync endpoints in tight loops

---

### 4. Data Validation

✅ **DO:**
- Validate input before API calls
- Check required fields
- Handle null/undefined values
- Validate response structure

❌ **DON'T:**
- Trust user input without validation
- Assume API responses are always complete

---

## See Also

- [POD 2.0 API Reference](pod2-api-reference.md) - POD framework APIs
- [Widget Patterns](widget-patterns.md) - Widget implementation patterns
- [Common Mistakes](common-mistakes.md) - Common POD plugin errors
- [SAP Digital Manufacturing API Documentation](https://help.sap.com/docs/sap-digital-manufacturing/operations-guide/prepare-for-api-integration)

---

**Document Version**: 1.0.0  
**Last Updated**: 2026-04-17  
**Maintained by**: POD Plugin Skill Team
