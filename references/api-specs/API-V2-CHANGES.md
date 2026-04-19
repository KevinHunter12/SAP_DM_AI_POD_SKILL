# SAP DM API v1 → v2 Changes Reference

This document provides guidance on API version differences for SAP Digital Manufacturing REST APIs. The pod-plugin skill maintains **v1 API specifications** as the baseline reference. This file documents key differences when working with v2 APIs.

## Overview

- **Baseline**: All API specs in this directory are v1 (except where explicitly marked v2/v3)
- **v2 APIs Removed**: To reduce skill size, v2 JSON specifications have been removed
- **v2 Access**: For v2 API details, consult the [SAP API Business Hub](https://api.sap.com/package/SAPDigitalManufacturing/rest)

## Deleted v2 API Specifications

The following v2 API specs were removed (use v1 specs as baseline):

1. `sapdme_sfc_v2.json` - Shop Floor Control v2
2. `sapdme_order_v2.json` - Order Management v2
3. `sapdme_batch_v2.json` - Batch Management v2
4. `sapdme_inventory_v2.json` - Inventory Management v2
5. `sapdme_production_v2.json` - Production v2
6. `sapdme_plant_resource_v2.json` - Plant Resource v2
7. `sapdme_plant_workcenter_v2.json` - Plant Work Center v2
8. `sapdme_processlot_v2.json` - Process Lot v2
9. `sapdme_processorder_v2.json` - Process Order v2
10. `sapdme_qualityinspection_v2.json` - Quality Inspection v2
11. `sapdme_staging_v2.json` - Staging v2
12. `sapdme_tool_v2.json` - Tool Management v2
13. `sapfnd_document_v2.json` - Document Management v2

## General v1 → v2 Differences

### Common Changes Across APIs

1. **Expanded Request Bodies**
   - v2 APIs often include additional optional fields
   - New filtering and sorting capabilities
   - Enhanced pagination parameters

2. **Response Structure Enhancements**
   - Additional metadata in responses
   - Improved error details with sub-codes
   - New fields for tracking and auditing

3. **Query Parameter Improvements**
   - More flexible filtering syntax
   - Case-insensitive search options
   - Additional sort parameters

4. **Backward Compatibility**
   - v1 endpoints remain supported
   - v2 is primarily additive (new fields, not breaking changes)
   - Existing v1 integrations continue to work

### Error Handling Differences

**v1 Error Format:**
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Error description"
  }
}
```

**v2 Error Format:**
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Error description",
    "details": [
      {
        "code": "SUB_ERROR_CODE",
        "message": "Detailed error information",
        "target": "fieldName"
      }
    ]
  }
}
```

### Pagination Differences

**v1 Pagination:**
```
GET /api/v1/resources?$top=50&$skip=0
```

**v2 Pagination:**
```
GET /api/v2/resources?$top=50&$skip=0&$count=true
```
- v2 adds optional `$count` parameter for total count
- v2 responses include `@odata.count` when requested

## API-Specific Notable Changes

### Shop Floor Control (SFC)

**v2 Additions:**
- Enhanced SFC status transitions
- Additional custom fields support
- Improved batch SFC operations

**Recommendation**: Use v1 for standard SFC operations. Upgrade to v2 only if you need:
- Bulk SFC status updates
- Extended custom data fields
- Advanced filtering by multiple criteria

### Order Management

**v2 Additions:**
- Order scheduling enhancements
- Priority management improvements
- Material substitution tracking

**Recommendation**: v1 is sufficient for most POD plugin use cases.

### Inventory Management

**v2 Additions:**
- Multi-location inventory queries
- Stock reservation tracking
- Enhanced material movement history

**Recommendation**: Use v1 unless you need multi-location inventory visibility.

### Quality Inspection

**v2 Additions:**
- Digital signature support
- Advanced defect tracking
- Inspection plan versioning

**Recommendation**: v1 covers standard inspection operations. Use v2 for advanced quality workflows.

## Migration Guidance

### When to Use v1 (Recommended Default)

Use v1 APIs when:
- ✅ Standard CRUD operations are sufficient
- ✅ You need proven, stable API contracts
- ✅ Your POD plugin doesn't require v2-specific features
- ✅ You want maximum compatibility across SAP DM versions

### When to Consider v2

Consider v2 APIs when:
- ⚠️ You need specific v2-only features (check SAP API Hub for details)
- ⚠️ Enhanced error details are critical for your use case
- ⚠️ You require advanced filtering/sorting capabilities
- ⚠️ Your SAP DM instance is on latest version with v2 fully supported

### Upgrading from v1 to v2

**Step 1**: Verify Feature Need
- Check SAP API Business Hub for v2 API documentation
- Confirm the specific v2 feature you need exists
- Evaluate if v1 workaround is simpler

**Step 2**: Update API Calls
```javascript
// v1
const response = await fetch('/plant/services/odata/v1/sfcs/...');

// v2
const response = await fetch('/plant/services/odata/v2/sfcs/...');
```

**Step 3**: Handle Response Differences
- Check for new optional fields in responses
- Update error handling for enhanced error structure
- Test pagination if using `$count` parameter

**Step 4**: Backward Compatibility
- Keep v1 fallback for non-critical features
- Use feature detection rather than version detection
- Document v2 dependency in plugin README

## Testing Recommendations

When using v2 APIs:

1. **Dual Testing**: Test against both v1 and v2 endpoints during development
2. **Version Detection**: Add logic to detect supported API version at runtime
3. **Graceful Degradation**: Fall back to v1 if v2 is unavailable
4. **Error Handling**: Handle both v1 and v2 error response formats

## External Resources

- [SAP API Business Hub - Digital Manufacturing](https://api.sap.com/package/SAPDigitalManufacturing/rest)
- [SAP Help Portal - API Documentation](https://help.sap.com/docs/sap-digital-manufacturing)
- [SAP Digital Manufacturing - What's New](https://help.sap.com/docs/sap-digital-manufacturing/whats-new)

## Quick Reference: v1 API Specs Available

The following v1 API specifications are available in this directory:

- **Shop Floor Control**: `sapdme_sfc.json`
- **Order Management**: `sapdme_order.json`
- **Material**: `sapdme_material.json`
- **Operation Activity**: `sapdme_operationactivity.json`
- **Work Instruction**: `sapdme_workinstruction.json`
- **Data Collection**: `sapdme_datacollection.json`
- **Inventory**: `sapdme_inventory.json`
- **Plant Resource**: `sapdme_plant_resource.json`
- **Plant Work Center**: `sapdme_plant_workcenter.json`
- **BOM**: `sapdme_bom.json`
- **Quality Inspection**: `sapdme_qualityinspection.json`
- **Activity Confirmation**: `sapdme_activityconfirmation.json`
- **Quantity Confirmation**: `sapdme_quantityconfirmation.json`
- **Labor**: `sapdme_labor.json`
- **OEE**: `sapdme_oee.json`
- And 40+ more...

---

**Last Updated**: 2026-04-18  
**Skill Version**: 26.0.0 (Optimization Release)
