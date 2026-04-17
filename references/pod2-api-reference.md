# POD 2.0 API Complete Reference

## Overview

This document provides a comprehensive API reference for SAP Digital Manufacturing POD 2.0 development, extracted from official SAP JSDoc documentation.

---

## TABLE OF CONTENTS

1. [Common Imports Reference](#common-imports-reference)
2. [Widget Base Class](#widget-base-class)
3. [Widget Subclasses](#widget-subclasses)
4. [PodContext](#podcontext)
5. [ModelPath Constants](#modelpath-constants)
6. [Action Base Class](#action-base-class)
7. [Core Action Classes](#core-action-classes)
8. [Property Editors](#property-editors)
9. [REST Client](#rest-client)
10. [OData Clients](#odata-clients)
11. [Public API Clients](#public-api-clients)
12. [Logger](#logger)
13. [DateTimeUtils](#datetimeutils)
14. [Widget Registry](#widget-registry)
15. [Action Registry](#action-registry)

---

## COMMON IMPORTS REFERENCE

### Control & Enum Imports

**CRITICAL**: Many common SAPUI5 enums are frequently imported from wrong modules. Use these correct paths:

```javascript
// ✅ PlacementType (for Popover, Dialog positioning)
import PlacementType from "sap/m/PlacementType";
// NOT from "sap/ui/core/library"!
// Values: PlacementType.Auto, PlacementType.Bottom, PlacementType.Top,
//         PlacementType.Left, PlacementType.Right, etc.

// ✅ Button Types
import ButtonType from "sap/m/ButtonType";
// Values: ButtonType.Default, ButtonType.Accept, ButtonType.Reject,
//         ButtonType.Emphasized, ButtonType.Transparent, etc.

// ✅ List Modes (selection)
import ListMode from "sap/m/ListMode";
// Values: ListMode.None, ListMode.SingleSelect, ListMode.MultiSelect,
//         ListMode.Delete, ListMode.SingleSelectLeft, etc.

// ✅ Message Types
import MessageType from "sap/m/MessageType";
// Values: MessageType.Success, MessageType.Error, MessageType.Warning,
//         MessageType.Information, MessageType.None

// ✅ Value States (input validation)
import ValueState from "sap/ui/core/ValueState";
// This one IS in sap/ui/core (exception to the rule)
// Values: ValueState.None, ValueState.Error, ValueState.Warning,
//         ValueState.Success, ValueState.Information
```

### POD 2.0 Core Imports

**CRITICAL**: Use `context/` NOT `model/` for PodContext and ModelPath!

```javascript
// ✅ CORRECT - Use context/ path
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import ModelPath from "sap/dm/dme/pod2/context/ModelPath";

// ❌ WRONG - Don't use model/ path (old/incorrect)
// import PodContext from "sap/dm/dme/pod2/model/PodContext";
// import ModelPath from "sap/dm/dme/pod2/model/ModelPath";

// ✅ Widget Base Classes
import Widget from "sap/dm/dme/pod2/widget/Widget";
import ControlWidget from "sap/dm/dme/pod2/widget/ControlWidget";
import LayoutWidget from "sap/dm/dme/pod2/widget/LayoutWidget";
import TableWidget from "sap/dm/dme/pod2/widget/core/TableWidget";

// ✅ i18n (Framework-driven pattern)
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";
```

### Property Editors & Metadata

```javascript
// ✅ Widget Property Definition
import WidgetProperty from "sap/dm/dme/pod2/widget/metadata/WidgetProperty";
// NOTE: in widget/metadata/ NOT property/!

// ✅ Property Editors (in propertyeditor/ NOT property/editor/)
import StringPropertyEditor from "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor";
import IntegerPropertyEditor from "sap/dm/dme/pod2/propertyeditor/IntegerPropertyEditor";
import BooleanPropertyEditor from "sap/dm/dme/pod2/propertyeditor/BooleanPropertyEditor";
import SelectPropertyEditor from "sap/dm/dme/pod2/propertyeditor/SelectPropertyEditor";
import ColorPropertyEditor from "sap/dm/dme/pod2/propertyeditor/ColorPropertyEditor";
import IconPropertyEditor from "sap/dm/dme/pod2/propertyeditor/IconPropertyEditor";
```

### Correct ModelPath Constants

**CRITICAL**: Work list paths are PLURAL and return arrays!

```javascript
// ✅ Work List (arrays - note plural!)
ModelPath.SelectedWorkListItems   // Array of selected items (NOT "Item" singular!)
ModelPath.WorkListItems           // Array of all items
ModelPath.WorkListCount           // Number
ModelPath.WorkListLoading         // Boolean

// ✅ Resources (arrays)
ModelPath.FilterResources         // Array of selected resources
ModelPath.CurrentResource         // Single resource object

// ✅ Operations
ModelPath.CurrentOperation        // Single operation object
ModelPath.SelectedOperationActivities // Array (plural!)

// ❌ WRONG - These DON'T EXIST
// ModelPath.SelectedWorkListItem  // ❌ Doesn't exist!
// ModelPath.SelectedSfc            // ❌ Doesn't exist!
```

**Always verify exact constant names in** [ModelPath Constants](#modelpath-constants) **section before use!**

---

## WIDGET BASE CLASS

**Class:** `sap.dm.dme.pod2.widget.Widget`

### Lifecycle Methods

```javascript
/**
 * Called when widget is initialized
 */
onInit(): void

/**
 * Called when widget is destroyed
 */
onExit(): void

/**
 * ABSTRACT - Must be overridden
 * Create and return the widget view
 * @returns {sap.ui.core.Element | Promise<sap.ui.core.Element>}
 */
_createView(): sap.ui.core.Element | Promise<sap.ui.core.Element>

/**
 * Handle widget events asynchronously
 * @param {string} sEventId - Event identifier
 * @param {sap.ui.base.Event} oEvent - Event object
 * @returns {Promise<void>}
 */
_handleEvent(sEventId, oEvent): Promise<void>
```

### Property Methods

```javascript
/**
 * Get property value by name
 * @param {string} sName - Property name
 * @returns {any} Property value
 */
getPropertyValue(sName): any

/**
 * Set property value
 * @param {string} sName - Property name
 * @param {any} vValue - New value
 */
setPropertyValue(sName, vValue): void

/**
 * Get all widget properties
 * @returns {Array<sap.dm.dme.pod2.widget.metadata.WidgetProperty> | null}
 */
getProperties(): Array<WidgetProperty> | null

/**
 * Get property editor for a property
 * @param {WidgetProperty} oProperty - Property object
 * @returns {PropertyEditor | null}
 */
getPropertyEditor(oProperty): PropertyEditor | null
```

### Configuration Methods

```javascript
/**
 * Get widget configuration
 * @returns {sap.dm.dme.pod2.widget.WidgetConfig}
 */
getConfig(): WidgetConfig

/**
 * Get layout data configuration
 * @returns {Object | null}
 */
getLayoutDataConfig(): Object | null

/**
 * Get layout data property value
 * @param {string} sName - Property name
 * @returns {any | null}
 */
getLayoutDataPropertyValue(sName): any | null

/**
 * Set layout data property value
 * @param {string} sProperty - Property name
 * @param {any} vValue - New value
 */
setLayoutDataPropertyValue(sProperty, vValue): void

/**
 * Get layout properties for child widget
 * @param {Widget} oChildWidget - Child widget
 * @returns {Array<LayoutDataProperty> | null}
 */
getLayoutDataPropertiesForChild(oChildWidget): Array<LayoutDataProperty> | null
```

### Child Widget Management

```javascript
/**
 * Add a child widget
 * @param {Widget} oWidget - Widget to add
 * @param {number} [iIndex] - Optional index position
 */
addChildWidget(oWidget, iIndex?): void

/**
 * Get child widgets in aggregation
 * @param {string} sAggregationId - Aggregation ID
 * @returns {Array<Widget>}
 */
getChildWidgets(sAggregationId): Array<Widget>

/**
 * Check if widget contains another widget
 * @param {Widget} oWidget - Widget to check
 * @returns {boolean}
 */
containsWidget(oWidget): boolean

/**
 * Find child widget
 * @param {Widget} oWidget - Widget to find
 * @returns {[string, number]} Tuple of aggregation ID and index
 */
findChildWidget(oWidget): [string, number]

/**
 * Remove widget from view
 * @param {string} sAggregationId - Aggregation ID
 * @param {Widget} oWidget - Widget to remove
 * @private
 */
_removeWidgetFromView(sAggregationId, oWidget): void

/**
 * Add widget to view
 * @param {WidgetAggregation} oAggregation - Aggregation object
 * @param {Widget} oWidget - Widget to add
 * @param {number} [iIndex] - Optional index
 * @private
 */
_addWidgetToView(oAggregation, oWidget, iIndex?): void
```

### Aggregation & Events

```javascript
/**
 * Get all widget aggregations
 * @returns {Array<WidgetAggregation> | null}
 */
getAggregations(): Array<WidgetAggregation> | null

/**
 * Get specific aggregation
 * @param {string} sAggregationId - Aggregation ID
 * @returns {WidgetAggregation | null}
 */
getAggregation(sAggregationId): WidgetAggregation | null

/**
 * Get all widget events
 * @returns {Array<WidgetEvent> | null}
 */
getEvents(): Array<WidgetEvent> | null

/**
 * Check if widget has aggregations
 * @returns {boolean}
 */
hasAggregations(): boolean

/**
 * Check if widget has child widgets
 * @returns {boolean}
 */
hasChildWidgets(): boolean
```

### Getters & Utilities

```javascript
/**
 * Get unique widget ID
 * @returns {string}
 */
getId(): string

/**
 * Get widget type
 * @returns {string}
 */
getType(): string

/**
 * Get rendered view element
 * @returns {sap.ui.core.Element}
 */
getView(): sap.ui.core.Element

/**
 * Get parent widget
 * @returns {Widget}
 */
getParentWidget(): Widget

/**
 * Get root widget in hierarchy
 * @returns {Widget}
 */
getRootWidget(): Widget

/**
 * Get POD runtime instance
 * @returns {PodRuntime}
 */
getPodRuntime(): PodRuntime

/**
 * Check widget visibility
 * @returns {boolean}
 */
isVisible(): boolean

/**
 * Scroll widget into view
 * @param {Object} [oOptions] - Scroll options
 */
scrollIntoView(oOptions?): void

/**
 * Get aggregation ID for dropped item
 * @param {sap.ui.base.Event} oEvent - Drop event
 * @returns {string | null}
 */
getDroppedAggregationId(oEvent): string | null

/**
 * Get localized text
 * @param {string} sKey - i18n key
 * @param {...any} [aArgs] - Format arguments
 * @returns {string}
 */
getI18nText(sKey, ...aArgs?): string
```

### Static Methods

```javascript
/**
 * REQUIRED - Get widget category for POD Designer
 * @returns {string}
 */
static getCategory(): string

/**
 * Get default widget configuration
 * @param {string} sId - Widget ID
 * @returns {DefaultWidgetConfig}
 */
static getDefaultConfig(sId): DefaultWidgetConfig

/**
 * Generate unique widget ID
 * @param {string} sWidgetType - Widget type
 * @returns {string}
 * @private
 */
static _generateWidgetId(sWidgetType): string

/**
 * Generate multiple widget IDs
 * @param {string} sWidgetType - Widget type
 * @param {number} iCount - Number of IDs
 * @returns {Array<string>}
 * @private
 */
static _generateWidgetIds(sWidgetType, iCount): Array<string>
```

---

## WIDGET SUBCLASSES

### IntegrationWidget

**Class:** `sap.dm.dme.pod2.widget.IntegrationWidget`

Extends Widget for integration with external systems.

```javascript
/**
 * Returns promise when widget is ready
 * @returns {Promise<unknown>}
 */
ready(): Promise<unknown>

/**
 * ABSTRACT - Get entry point URL
 * @returns {string | Promise<string>}
 */
getEntryPoint(): string | Promise<string>
```

### LayoutWidget

**Class:** `sap.dm.dme.pod2.widget.LayoutWidget`

Extends Widget for layouts containing child widgets.

```javascript
/**
 * Aggregations to include in layout
 * @type {Array<string>}
 */
static INCLUDE_AGGREGATIONS: Array<string>

/**
 * Aggregations to exclude from layout
 * @type {Array<string>}
 */
static EXCLUDE_AGGREGATIONS: Array<string>
```

### ComponentWidget

**Class:** `sap.dm.dme.pod2.widget.ComponentWidget`

Extends Widget for SAPUI5 component-based widgets.

```javascript
/**
 * ABSTRACT - Get component name
 * @returns {string}
 */
getComponentName(): string

/**
 * Get component models
 * @returns {Object<string, sap.ui.model.Model>}
 */
getComponentModels(): Object<string, Model>
```

### TableWidget

**Class:** `sap.dm.dme.pod2.widget.core.TableWidget`

Extends Widget for table-based displays.

```javascript
/**
 * Get underlying table control
 * @returns {sap.m.Table}
 */
getTable(): sap.m.Table

/**
 * Get table sorting configuration
 * @returns {Array<Sorting>}
 */
getSorting(): Array<Sorting>

/**
 * ABSTRACT - Create table cell
 * @param {Object} oColumnConfig - Column configuration
 * @returns {sap.ui.core.Control}
 */
_createCell(oColumnConfig): sap.ui.core.Control

/**
 * ABSTRACT - Get model path for data binding
 * @returns {string}
 */
_getModelPath(): string

/**
 * Get data model
 * @returns {sap.ui.model.Model}
 */
_getModel(): sap.ui.model.Model
```

---

## PODCONTEXT

**Class:** `sap.dm.dme.pod2.context.PodContext`

**ALL METHODS ARE STATIC** - Call directly on the class.

### Core Data Access

```javascript
/**
 * Get data from context model path
 * @param {string} sModelPath - Model path (use ModelPath constants)
 * @returns {any}
 */
static get(sModelPath): any

/**
 * Wait for data to become available
 * @param {string} sModelPath - Model path
 * @returns {Promise<any>}
 */
static getWhenAvailable(sModelPath): Promise<any>

/**
 * Set context value
 * @param {string} sModelPath - Model path
 * @param {any} vValue - Value to set
 * @returns {boolean}
 */
static set(sModelPath, vValue): boolean
```

### Subscription Management

```javascript
/**
 * Subscribe to model path changes
 * @param {string|Array<string>} vModelPath - Model path or paths
 * @param {Function} fnCallback - Callback function (newValue, oldValue?)
 * @param {Object} oBindContext - Context for callback (usually 'this')
 */
static subscribe(vModelPath, fnCallback, oBindContext): void

/**
 * Unsubscribe from changes
 * @param {string|Array<string>} vModelPath - Model path or paths
 * @param {Function} fnCallback - Same callback used in subscribe
 * @param {Object} oBindContext - Same context used in subscribe
 */
static unsubscribe(vModelPath, fnCallback, oBindContext): void

/**
 * Unsubscribe all listeners for a context
 * @param {Object} oBindContext - Context object
 */
static unsubscribeAll(oBindContext): void
```

### Work List Methods

```javascript
/**
 * Get all work list items
 * @returns {Array<WorkListItem>}
 */
static getWorkListItems(): Array<WorkListItem>

/**
 * Get selected work list items
 * @returns {Array<BaseWorkListItem>}
 */
static getSelectedWorkListItems(): Array<BaseWorkListItem>

/**
 * Get last selected work list item
 * @returns {BaseWorkListItem | null}
 */
static getLastSelectedWorkListItem(): BaseWorkListItem | null

/**
 * Get total work list count
 * @returns {number}
 */
static getWorkListCount(): number

/**
 * Check if work list is loading
 * @returns {boolean}
 */
static getWorkListLoading(): boolean

/**
 * Get work list page size
 * @returns {number}
 */
static getWorkListPageSize(): number

/**
 * Get work list sorting
 * @returns {Array<Sorting>}
 */
static getWorkListSorting(): Array<Sorting>

/**
 * Get work list type
 * @returns {WorkListType}
 */
static getWorkListType(): WorkListType

/**
 * Clear work list
 */
static clearWorkList(): void

/**
 * Set work list items
 * @param {Array} aItems - Work list items
 */
static setWorkListItems(aItems): void

/**
 * Set selected work list items
 * @param {Array} aWorkListItems - Selected items
 */
static setSelectedWorkListItems(aWorkListItems): void

/**
 * Set work list count
 * @param {number} iCount - Total count
 */
static setWorkListCount(iCount): void

/**
 * Set work list loading state
 * @param {boolean} bLoading - Loading state
 */
static setWorkListLoading(bLoading): void

/**
 * Set work list page size
 * @param {number} iPageSize - Page size
 */
static setWorkListPageSize(iPageSize): void

/**
 * Set work list sorting
 * @param {Array<Sorting>} aSorting - Sorting configuration
 */
static setWorkListSorting(aSorting): void

/**
 * Set work list type
 * @param {string} sType - Work list type
 */
static setWorkListType(sType): void
```

### Operation Activity Methods

```javascript
/**
 * Get operation activities
 * @returns {Array<BaseOperationWorkItem>}
 */
static getOperationActivities(): Array<BaseOperationWorkItem>

/**
 * Get selected operation activities
 * @returns {Array<BaseOperationWorkItem>}
 */
static getSelectedOperationActivities(): Array<BaseOperationWorkItem>

/**
 * Get last selected operation activity
 * @returns {BaseOperationWorkItem | null}
 */
static getLastSelectedOperationActivity(): BaseOperationWorkItem | null

/**
 * Set operation activities
 * @param {Array} aOperationActivities - Activities
 */
static setOperationActivities(aOperationActivities): void

/**
 * Set selected operation activities
 * @param {Array} aOperationActivities - Selected activities
 */
static setSelectedOperationActivities(aOperationActivities): void
```

### Data Collection Methods

```javascript
/**
 * Get data collection groups
 * @returns {Array<DataCollectionGroup>}
 */
static getDataCollectionGroups(): Array<DataCollectionGroup>

/**
 * Get selected data collection group
 * @returns {DataCollectionGroup}
 */
static getDataCollectionSelectedGroup(): DataCollectionGroup

/**
 * Get data collection log
 * @returns {Array<DataCollectionParameter>}
 */
static getDataCollectionLog(): Array<DataCollectionParameter>

/**
 * Set data collection groups
 * @param {Array} aDataCollectionGroups - Groups
 */
static setDataCollectionGroups(aDataCollectionGroups): void

/**
 * Set selected data collection group
 * @param {DataCollectionGroup} oDataCollectionGroup - Selected group
 */
static setDataCollectionSelectedGroup(oDataCollectionGroup): void

/**
 * Set data collection log
 * @param {Array} aDataCollectionLog - Log entries
 */
static setDataCollectionLog(aDataCollectionLog): void
```

### Filter Methods

```javascript
/**
 * Get SFC filter
 * @returns {string}
 */
static getFilterSFC(): string

/**
 * Get filter input type
 * @returns {WorkListFilterInputType}
 */
static getFilterInputType(): WorkListFilterInputType

/**
 * Get resource filter
 * @returns {Array<Resource>}
 */
static getFilterResources(): Array<Resource>

/**
 * Get work center filter
 * @returns {Array<WorkCenter>}
 */
static getFilterWorkCenters(): Array<WorkCenter>

/**
 * Get material filter
 * @returns {Array<Material>}
 */
static getFilterMaterials(): Array<Material>

/**
 * Get operation activity filter
 * @returns {Array<OperationActivityMaster>}
 */
static getFilterOperationActivities(): Array<OperationActivityMaster>

/**
 * Set filter input type
 * @param {string} sInputType - Input type
 */
static setFilterInputType(sInputType): void

/**
 * Set resource filter
 * @param {Array} aResources - Resources
 */
static setFilterResources(aResources): void

/**
 * Set work center filter
 * @param {Array} aWorkCenters - Work centers
 */
static setFilterWorkCenters(aWorkCenters): void

/**
 * Set material filter
 * @param {Array} aMaterials - Materials
 */
static setFilterMaterials(aMaterials): void

/**
 * Set operation activity filter
 * @param {Array} aOperationActivities - Operation activities
 */
static setFilterOperationActivities(aOperationActivities): void
```

### User & Environment Methods

```javascript
/**
 * Get badged-in user
 * @returns {BadgedInUser}
 */
static getBadgedInUser(): BadgedInUser

/**
 * Get current user ID
 * @returns {string}
 */
static getUserId(): string

/**
 * Get current plant
 * @returns {string}
 */
static getPlant(): string

/**
 * Get plant time zone
 * @returns {string}
 */
static getPlantTimeZone(): string

/**
 * Get POD ID
 * @returns {string}
 */
static getPodId(): string

/**
 * Get industry type
 * @returns {string}
 */
static getIndustryType(): string

/**
 * Set badged-in user
 * @param {BadgedInUser} oBadgedInUser - User object
 */
static setBadgedInUser(oBadgedInUser): void

/**
 * Check if in design mode
 * @returns {boolean}
 */
static isDesignMode(): boolean

/**
 * Check if in run mode
 * @returns {boolean}
 */
static isRunMode(): boolean

/**
 * Check if discrete industry
 * @returns {boolean}
 */
static isDiscreteIndustry(): boolean

/**
 * Check if process industry
 * @returns {boolean}
 */
static isProcessIndustry(): boolean
```

### I18n & Models

```javascript
/**
 * Apply POD core models to control
 * @param {sap.ui.core.Control} oControl - Control
 */
static applyCoreModelsTo(oControl): void

/**
 * Get i18n resource model
 * @returns {I18nResourceModel}
 */
static getI18nModel(): I18nResourceModel

/**
 * Get localized text
 * @param {string} sKey - i18n key
 * @param {...any} [aArgs] - Format arguments
 * @returns {string}
 */
static getI18nText(sKey, ...aArgs?): string
```

### Other Context Methods

```javascript
/**
 * Get goods receipt summary
 * @returns {GoodsReceiptSummary}
 */
static getGoodsReceiptSummary(): GoodsReceiptSummary

/**
 * Get goods receipt line items
 * @returns {Array<GoodsReceiptLineItem>}
 */
static getGoodsReceiptLineItems(): Array<GoodsReceiptLineItem>

/**
 * Get activity summaries
 * @returns {ActivitySummaries}
 */
static getActivitySummaries(): ActivitySummaries

/**
 * Get reported quantity items
 * @returns {Array<ReportedQuantity>}
 */
static getReportedQuantityItems(): Array<ReportedQuantity>

/**
 * Get work instructions
 * @returns {Array<WorkInstruction>}
 */
static getWorkInstructions(): Array<WorkInstruction>

/**
 * Get selected work instruction
 * @returns {WorkInstruction}
 */
static getSelectedWorkInstruction(): WorkInstruction

/**
 * Get execution SFC quantity
 * @returns {number}
 */
static getExecutionSFCQuantity(): number

/**
 * Get POD runtime
 * @returns {PodRuntime}
 */
static getPodRuntime(): PodRuntime

/**
 * Resolve data binding
 * @param {any} vBinding - Binding expression
 * @param {sap.ui.core.Control} [oControl] - Control context
 * @returns {any}
 */
static resolveBinding(vBinding, oControl?): any
```

---

## MODELPATH CONSTANTS

**Class:** `sap.dm.dme.pod2.context.ModelPath`

Common model paths for use with PodContext.subscribe() and PodContext.get():

```javascript
ModelPath.FilterResources           // Selected resources
ModelPath.FilterWorkCenters          // Selected work centers
ModelPath.FilterMaterials            // Selected materials
ModelPath.FilterOperationActivities  // Selected operation activities
ModelPath.FilterInputType            // Filter input type

ModelPath.SelectedWorkListItems      // Selected work list items
ModelPath.WorkListItems              // All work list items
ModelPath.WorkListCount              // Work list count
ModelPath.WorkListLoading            // Loading state
ModelPath.WorkListPageSize           // Page size
ModelPath.WorkListSorting            // Sorting configuration
ModelPath.WorkListType               // Work list type

ModelPath.OperationActivities        // Operation activities
ModelPath.SelectedOperationActivities // Selected activities

ModelPath.DataCollectionGroups       // Data collection groups
ModelPath.DataCollectionSelectedGroup // Selected DC group
ModelPath.DataCollectionLog          // DC log entries

ModelPath.GoodsReceiptSummary        // Goods receipt summary
ModelPath.GoodsReceiptLineItems      // GR line items

ModelPath.ActivitySummaries          // Activity summaries
ModelPath.ReportedQuantityItems      // Reported quantities
ModelPath.WorkInstructions           // Work instructions
ModelPath.SelectedWorkInstruction    // Selected instruction
ModelPath.ExecutionSFCQuantity       // SFC quantity
```

---

## ACTION BASE CLASS

**Class:** `sap.dm.dme.pod2.action.Action`

### Core Methods

```javascript
/**
 * ABSTRACT - Execute the action (must be overridden)
 * @param {ActionContext} oActionContext - Action context
 * @returns {void | Promise<void>}
 */
execute(oActionContext): void | Promise<void>

/**
 * Get action ID
 * @returns {string}
 */
getId(): string

/**
 * Get action configuration
 * @returns {ActionConfig}
 */
getConfig(): ActionConfig

/**
 * Get action properties
 * @returns {Array<ActionProperty>}
 */
getProperties(): Array<ActionProperty>

/**
 * Get property value
 * @param {string} sPropertyId - Property ID
 * @returns {any}
 */
getPropertyValue(sPropertyId): any

/**
 * Set property value
 * @param {string} sPropertyId - Property ID
 * @param {any} vValue - New value
 */
setPropertyValue(sPropertyId, vValue): void

/**
 * Get POD runtime
 * @returns {PodRuntime}
 */
getPodRuntime(): PodRuntime

/**
 * Get localized text
 * @param {string} sKey - i18n key
 * @param {...any} [aArgs] - Format arguments
 * @returns {string}
 */
getI18nText(sKey, ...aArgs?): string
```

### Action Context

**Class:** `sap.dm.dme.pod2.action.ActionContext`

```javascript
// Properties
widget: Widget         // Widget that triggered action
event: Event          // UI event that triggered action

/**
 * Abort action execution
 */
abort(): void

/**
 * Check if action was aborted
 * @returns {boolean}
 */
isAborted(): boolean
```

---

## CORE ACTION CLASSES

### BusyIndicatorAction

**Class:** `sap.dm.dme.pod2.action.core.BusyIndicatorAction`

Shows/hides busy indicator.

```javascript
execute(oActionContext): void | Promise<void>
onExit(): void
```

### ConfirmAction

**Class:** `sap.dm.dme.pod2.action.core.ConfirmAction`

Shows confirmation dialog.

```javascript
execute(oActionContext): Promise<void>
```

### MessageBoxAction

**Class:** `sap.dm.dme.pod2.action.core.MessageBoxAction`

Shows message box dialog.

```javascript
// Property IDs
static PropertyId = {
    Message: "message",
    MessageBoxType: "messageBoxType"
}

// Message box types
static MessageBoxType = {
    Dialog: "Dialog",
    Error: "Error",
    Information: "Information",
    Success: "Success",
    Warning: "Warning",
    Confirm: "Confirm"
}

execute(oActionContext): Promise<void>
```

### LogoutAction

**Class:** `sap.dm.dme.pod2.action.core.LogoutAction`

Logs out current user.

```javascript
execute(oActionContext): void | Promise<void>
```

### Dialog Actions

```javascript
// Show dialog
sap.dm.dme.pod2.action.dialog.ShowDialogAction
execute(oActionContext): void | Promise<void>

// Close dialog
sap.dm.dme.pod2.action.dialog.CloseDialogAction
execute(oActionContext): void | Promise<void>
```

### Navigation Actions

```javascript
// Navigate to page
sap.dm.dme.pod2.action.navigation.NavigateToPageAction
execute(oActionContext): void | Promise<void>

// Navigate back
sap.dm.dme.pod2.action.navigation.NavigateBackAction
execute(oActionContext): void | Promise<void>

// Navigate to widget
sap.dm.dme.pod2.action.navigation.NavigateToWidgetAction
execute(oActionContext): void | Promise<void>
```

### Badge Actions

```javascript
// Badge in user
sap.dm.dme.pod2.action.badge.BadgeInAction
execute(oActionContext): Promise<void>

// Badge out user
sap.dm.dme.pod2.action.badge.BadgeOutAction
execute(oActionContext): Promise<void>
```

### SFC Actions

```javascript
// Start SFC
sap.dm.dme.pod2.action.sfc.StartAction
execute(oActionContext): Promise<void>
getSfcs(fnFilter?): Array<string>
getResource(): string
getOperationActivity(): string

// Complete SFC
sap.dm.dme.pod2.action.sfc.CompleteAction
execute(oActionContext): Promise<void>
getSfcs(fnFilter?): Array<string>
getQuantity(bQuantityRequired?): number | undefined

// Serialize SFC
sap.dm.dme.pod2.action.sfc.SerializeAction
execute(oActionContext): Promise<void>

// Sign off SFC
sap.dm.dme.pod2.action.sfc.SignoffAction
execute(oActionContext): Promise<void>
```

### Phase Actions

```javascript
// Start phase/operation activity
sap.dm.dme.pod2.action.phase.StartPhaseAction
execute(oActionContext): Promise<void>

// Complete phase
sap.dm.dme.pod2.action.phase.CompletePhaseAction
execute(oActionContext): Promise<void>
onInit(): void
```

### Quantity Actions

```javascript
// Report quantity
sap.dm.dme.pod2.action.quantity.ReportQuantityAction
execute(oActionContext): Promise<void>
onInit(): void
```

---

## PROPERTY EDITORS

### CRITICAL: Correct Import Paths for Properties

```javascript
// WidgetProperty - for defining configurable properties
import WidgetProperty from "sap/dm/dme/pod2/widget/metadata/WidgetProperty";

// Property Editors - for creating property editor controls
import BooleanPropertyEditor from "sap/dm/dme/pod2/propertyeditor/BooleanPropertyEditor";
import StringPropertyEditor from "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor";
import SelectPropertyEditor from "sap/dm/dme/pod2/propertyeditor/SelectPropertyEditor";
import IntegerPropertyEditor from "sap/dm/dme/pod2/propertyeditor/IntegerPropertyEditor";
import ColorPropertyEditor from "sap/dm/dme/pod2/propertyeditor/ColorPropertyEditor";
import IconPropertyEditor from "sap/dm/dme/pod2/propertyeditor/IconPropertyEditor";
```

**IMPORTANT:**
- WidgetProperty is in `widget/metadata/` NOT `property/`
- Property editors are in `propertyeditor/` NOT `property/editor/`
- There is NO `PropertyCategory` class - use string values like `"Main"` for category

### Base PropertyEditor

**Class:** `sap.dm.dme.pod2.propertyeditor.PropertyEditor`

```javascript
/**
 * ABSTRACT - Get UI control for editor
 * @returns {sap.ui.core.Control}
 */
getControl(): sap.ui.core.Control

/**
 * Get label for property
 * @returns {sap.m.Label}
 */
getLabel(): sap.m.Label

/**
 * Get property ID
 * @returns {string}
 */
getPropertyId(): string

/**
 * Set visibility
 * @param {boolean} bVisible - Visible state
 */
setVisible(bVisible): void

/**
 * Get property accessor
 * @returns {$PropertyAccessor}
 * @private
 */
_getPropertyAccessor(): $PropertyAccessor

/**
 * Get current property value
 * @returns {any}
 * @private
 */
_getPropertyValue(): any

/**
 * Set property value
 * @param {any} vValue - New value
 * @private
 */
_setPropertyValue(vValue): void

/**
 * Create label
 * @param {string} sDisplayName - Display name
 * @param {string} [sDescription] - Description
 * @returns {sap.m.Label}
 * @private
 */
_createLabel(sDisplayName, sDescription?): sap.m.Label

/**
 * Get localized text
 * @param {string} sKey - i18n key
 * @param {...any} [aArgs] - Format arguments
 * @returns {string}
 * @private
 */
_getI18nText(sKey, ...aArgs?): string
```

### Available Property Editor Types

#### StringPropertyEditor
For text input.

```javascript
new sap.dm.dme.pod2.propertyeditor.StringPropertyEditor(
    oPropertyAccessor,  // Property accessor
    sProperty,          // Property ID
    eInputType?         // Optional input type
)
```

#### IntegerPropertyEditor
For integer values.

```javascript
new sap.dm.dme.pod2.propertyeditor.IntegerPropertyEditor(
    oPropertyAccessor,  // Property accessor
    sProperty,          // Property ID
    iDefault?           // Optional default value
)
```

#### FloatPropertyEditor
For decimal numbers.

```javascript
new sap.dm.dme.pod2.propertyeditor.FloatPropertyEditor(
    oPropertyAccessor,  // Property accessor
    sProperty           // Property ID
)
```

#### BooleanPropertyEditor
For true/false values.

```javascript
new sap.dm.dme.pod2.propertyeditor.BooleanPropertyEditor(
    oPropertyAccessor,  // Property accessor
    sProperty,          // Property ID
    bDefaultValue?      // Optional default value
)
```

#### SelectPropertyEditor
For dropdown selection.

```javascript
new sap.dm.dme.pod2.propertyeditor.SelectPropertyEditor(
    oPropertyAccessor,  // Property accessor
    sPropertyId,        // Property ID
    vItems?,            // Optional items array or binding
    sDefaultKey?        // Optional default key
)
```

#### EnumPropertyEditor
For enum values.

```javascript
new sap.dm.dme.pod2.propertyeditor.EnumPropertyEditor(
    oPropertyAccessor,  // Property accessor
    sProperty,          // Property ID
    vItems,             // Enum items
    sDefaultValue?      // Optional default value
)
```

#### Other Property Editors

```javascript
// Multiple selection
sap.dm.dme.pod2.propertyeditor.MultiChoicePropertyEditor

// Color picker
sap.dm.dme.pod2.propertyeditor.ColorPropertyEditor

// Icon picker
sap.dm.dme.pod2.propertyeditor.IconPropertyEditor

// CSS size input
sap.dm.dme.pod2.propertyeditor.CssSizePropertyEditor

// Binding expression editor
sap.dm.dme.pod2.propertyeditor.BindStringPropertyEditor

// Resource picker
sap.dm.dme.pod2.propertyeditor.ResourcePropertyEditor

// Work center picker
sap.dm.dme.pod2.propertyeditor.WorkCenterPropertyEditor

// Operation activity picker
sap.dm.dme.pod2.propertyeditor.OperationActivityPropertyEditor
```

---

## REST CLIENT

**Class:** `sap.dm.dme.pod2.api.RestClient`

**ALL METHODS ARE STATIC**

```javascript
/**
 * Generic fetch request
 * @param {string} sUrl - Request URL
 * @param {Object} [oOptions] - Fetch options
 * @returns {Promise<any>}
 */
static fetch(sUrl, oOptions?): Promise<any>

/**
 * GET request
 * @param {string} sUrl - Request URL
 * @param {Object} [oQueryParams] - Query parameters
 * @param {Object} [oOptions] - Fetch options
 * @returns {Promise<any>}
 */
static get(sUrl, oQueryParams?, oOptions?): Promise<any>

/**
 * POST request
 * @param {string} sUrl - Request URL
 * @param {any} vRequestBody - Request body
 * @param {Object} [oOptions] - Fetch options
 * @returns {Promise<any>}
 */
static post(sUrl, vRequestBody, oOptions?): Promise<any>

/**
 * PUT request
 * @param {string} sUrl - Request URL
 * @param {any} vRequestBody - Request body
 * @param {Object} [oOptions] - Fetch options
 * @returns {Promise<any>}
 */
static put(sUrl, vRequestBody, oOptions?): Promise<any>

/**
 * PATCH request
 * @param {string} sUrl - Request URL
 * @param {any} vRequestBody - Request body
 * @param {Object} [oOptions] - Fetch options
 * @returns {Promise<any>}
 */
static patch(sUrl, vRequestBody, oOptions?): Promise<any>

/**
 * DELETE request
 * @param {string} sUrl - Request URL
 * @param {Object} [oOptions] - Fetch options
 * @returns {Promise<any>}
 */
static delete(sUrl, oOptions?): Promise<any>
```

### Usage Example

```javascript
import RestClient from "sap/dm/dme/pod2/api/RestClient";

// GET request
const oData = await RestClient.get("/api/v1/materials", {
    plant: "PLANT1000",
    material: "MAT*"
});

// POST request
const oResult = await RestClient.post("/api/v1/sfc/start", {
    plant: "PLANT1000",
    sfc: "SFC_001",
    resource: "RES_001"
});
```

---

## ODATA CLIENTS

### ODataV4Client

**Class:** `sap.dm.dme.pod2.api.ODataV4Client`

```javascript
/**
 * Constructor
 * @param {string} sServiceUrl - OData service URL
 * @param {number} [iMaxPage] - Max page size (default varies)
 */
constructor(sServiceUrl, iMaxPage?)

/**
 * Get single page of data
 * @param {string} sPath - Entity set path
 * @param {Object} oODataParams - OData parameters ($filter, $select, etc.)
 * @param {Object} [oOptions] - Request options
 * @returns {Promise<[Array, number]>} Tuple of [data array, total count]
 */
getPage(sPath, oODataParams, oOptions?): Promise<[Array, number]>

/**
 * Get all pages of data
 * @param {string} sPath - Entity set path
 * @param {Object} [oODataParams] - OData parameters
 * @param {Object} [oOptions] - Request options
 * @returns {Promise<Array<any>>}
 */
getAllPages(sPath, oODataParams?, oOptions?): Promise<Array<any>>

/**
 * Get entity by key
 * @param {string} sPath - Entity set path
 * @param {Object} oPredicateMap - Key-value map for keys
 * @param {Object} [oQueryParams] - Query parameters
 * @param {Object} [oOptions] - Request options
 * @returns {Promise<any>}
 */
getByKey(sPath, oPredicateMap, oQueryParams?, oOptions?): Promise<any>

/**
 * Get count
 * @param {string} sPath - Entity set path
 * @param {Object} oODataParams - OData parameters
 * @param {Object} [oOptions] - Request options
 * @returns {Promise<number>}
 */
getCount(sPath, oODataParams, oOptions?): Promise<number>

/**
 * Build $orderby clause from sorting array
 * @param {Array<Sorting>} aSorting - Sorting configuration
 * @returns {string}
 * @static
 */
static getOrderBy(aSorting): string
```

### ODataV2Client

**Class:** `sap.dm.dme.pod2.api.ODataV2Client`

Similar interface to ODataV4Client with OData V2 specifics.

```javascript
/**
 * Constructor
 * @param {string} sServiceUrl - OData service URL
 * @param {number} [iMaxPage] - Max page size
 */
constructor(sServiceUrl, iMaxPage?)

getPage(sPath, oODataParams, oOptions?): Promise<[Array, number]>
getAllPages(sPath, oODataParams?, oOptions?): Promise<Array<any>>
getByKey(sPath, oPredicateMap, oQueryParams?, oOptions?): Promise<any>
getCount(sPath, oODataParams, oOptions?): Promise<number>
getMaxPage(): number
getServiceUrl(): string
makeKeyPredicate(sPath, oPredicateMap): void
```

### Usage Example

```javascript
import { ODataV4Client } from "sap/dm/dme/pod2/api/ODataClient";

const oClient = new ODataV4Client("/odata/v4/MaterialService");

// Get page of materials
const [aMaterials, iCount] = await oClient.getPage("Materials", {
    $filter: "plant eq 'PLANT1000'",
    $select: "material,description",
    $top: 20,
    $skip: 0
});

// Get all materials (handles pagination automatically)
const allMaterials = await oClient.getAllPages("Materials", {
    $filter: "plant eq 'PLANT1000'"
});
```

---

## PUBLIC API CLIENTS

**Class:** `sap.dm.dme.pod2.api.ApiClient`

Provides static instances of API clients for various domains.

### SFC API

```javascript
import { ApiClient } from "sap/dm/dme/pod2/api/ApiClient";

/**
 * Get SFCs
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array>}
 */
await ApiClient.sfc.getSfcs(oRequest, oOptions?)

/**
 * Get SFC detail
 * @param {Object} oRequest - Request with plant and sfc
 * @param {Object} [oOptions] - Options
 * @returns {Promise<SfcDetail>}
 */
await ApiClient.sfc.getSfcDetail(oRequest, oOptions?)

/**
 * Get SFCs count
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<object>}
 */
await ApiClient.sfc.getSfcsCount(oRequest, oOptions?)

/**
 * Start SFC
 * @param {Object} oRequest - Start request
 * @returns {Promise<SfcStartResponse>}
 */
await ApiClient.sfc.sfcStart(oRequest)

/**
 * Complete SFC
 * @param {Object} oRequest - Complete request
 * @returns {Promise<SfcCompleteResponse>}
 */
await ApiClient.sfc.sfcComplete(oRequest)

/**
 * Sign off SFC
 * @param {Object} oRequest - Signoff request
 * @returns {Promise<SfcSignoffResponse>}
 */
await ApiClient.sfc.sfcSignoff(oRequest)

/**
 * Update SFC batch
 * @param {Object} oRequest - Update request
 * @returns {Promise<UpdateSfcBatchResponse>}
 */
await ApiClient.sfc.updateSfcBatch(oRequest)
```

### Order API

```javascript
/**
 * Get order details
 * @param {Object} oRequest - Request with plant and order
 * @param {Object} [oOptions] - Options
 * @returns {Promise<object>}
 */
await ApiClient.order.getOrder(oRequest, oOptions?)
```

### Material API

```javascript
/**
 * Get materials
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<[Array<GetMaterialsResponse>, number]>}
 */
await ApiClient.material.getMaterials(oRequest, oOptions?)
```

### Resource API

```javascript
/**
 * Get resources
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Resource>}
 */
await ApiClient.resource.getResources(oRequest, oOptions?)

/**
 * Get resource types
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array<ResourceType>>}
 */
await ApiClient.resource.getResourceTypes(oRequest, oOptions?)
```

### Work Center API

```javascript
/**
 * Get work centers
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array<WorkCenter>>}
 */
await ApiClient.workcenter.getWorkCenters(oRequest, oOptions?)
```

### Operation Activity API

```javascript
/**
 * Get operation activities
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<[Array<OperationActivity>, number]>}
 */
await ApiClient.operationactivity.getOperationActivities(oRequest, oOptions?)
```

### Data Collection API

```javascript
/**
 * Get data collection groups
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array<GetDataCollectionGroupsResponse>>}
 */
await ApiClient.datacollection.getDataCollectionGroups(oRequest, oOptions?)

/**
 * Log data collection group
 * @param {Object} oRequest - Log request
 * @param {Object} [oOptions] - Options
 * @returns {Promise<object>}
 */
await ApiClient.datacollection.logDataCollectionGroup(oRequest, oOptions?)
```

### Assembly API

```javascript
/**
 * Get assembled components
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array<GetAssembledComponentsResponse>>}
 */
await ApiClient.assembly.getAssembledComponents(oRequest, oOptions?)

/**
 * Get planned components
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array<GetPlannedComponentsResponse>>}
 */
await ApiClient.assembly.getPlannedComponents(oRequest, oOptions?)

/**
 * Assemble component
 * @param {Object} oRequest - Assemble request
 * @param {Object} [oOptions] - Options
 * @returns {Promise<object>}
 */
await ApiClient.assembly.assembleComponent(oRequest, oOptions?)
```

### BOM API

```javascript
/**
 * Get BOMs
 * @param {Object} oRequest - Request parameters
 * @param {Object} [oOptions] - Options
 * @returns {Promise<Array<GetBomsResponse>>}
 */
await ApiClient.bom.getBoms(oRequest, oOptions?)
```

### Other Available APIs

```javascript
ApiClient.inventory          // Inventory operations
ApiClient.uom               // Unit of measure
ApiClient.workinstruction   // Work instructions
ApiClient.processorder      // Process orders
ApiClient.mdo               // MDO objects
ApiClient.ebr               // Electronic batch records
ApiClient.internal          // Internal APIs
```

---

## LOGGER

**Class:** `sap.dm.dme.pod2.Logger`

### Get Logger Instance

```javascript
/**
 * Get logger for component
 * @param {string} sComponent - Component name
 * @returns {Logger}
 * @static
 */
const logger = sap.dm.dme.pod2.Logger.getLogger("MyComponent");
```

### Log Levels

```javascript
Logger.Level = {
    TRACE: 5,
    DEBUG: 4,
    INFO: 3,
    WARN: 2,
    ERROR: 1,
    FATAL: 0
}

Logger.LevelNames = {
    5: "TRACE",
    4: "DEBUG",
    3: "INFO",
    2: "WARN",
    1: "ERROR",
    0: "FATAL"
}
```

### Logging Methods

```javascript
/**
 * Trace level logging (most verbose)
 * @param {string} sMessage - Message
 * @param {...any} [aArgs] - Additional arguments
 */
logger.trace(sMessage, ...aArgs?)

/**
 * Debug level logging
 * @param {string} sMessage - Message
 * @param {...any} [aArgs] - Additional arguments
 */
logger.debug(sMessage, ...aArgs?)

/**
 * Info level logging
 * @param {string} sMessage - Message
 * @param {...any} [aArgs] - Additional arguments
 */
logger.info(sMessage, ...aArgs?)

/**
 * Warning level logging
 * @param {string} sMessage - Message
 * @param {...any} [aArgs] - Additional arguments
 */
logger.warn(sMessage, ...aArgs?)

/**
 * Error level logging
 * @param {string} sMessage - Message
 * @param {...any} [aArgs] - Additional arguments
 */
logger.error(sMessage, ...aArgs?)

/**
 * Fatal level logging (most severe)
 * @param {string} sMessage - Message
 * @param {...any} [aArgs] - Additional arguments
 */
logger.fatal(sMessage, ...aArgs?)
```

### Usage Example

```javascript
import Logger from "sap/dm/dme/pod2/Logger";

class MyWidget extends Widget {
    constructor() {
        this._logger = Logger.getLogger("MyWidget");
    }

    _someMethod() {
        this._logger.debug("Method called");

        try {
            // Do something
            this._logger.info("Operation successful");
        } catch (oError) {
            this._logger.error("Operation failed", oError);
        }
    }
}
```

---

## DATETIMEUTILS

**Class:** `sap.dm.dme.pod2.DateTimeUtils`

**ALL METHODS ARE STATIC**

### Date/Time Methods

```javascript
/**
 * Get current date/time
 * @returns {Date | UI5Date}
 * @static
 */
DateTimeUtils.now(): Date

/**
 * Get start of day (00:00:00)
 * @param {Date} [oDate] - Date (defaults to today)
 * @returns {Date | UI5Date}
 * @static
 */
DateTimeUtils.startOfDay(oDate?): Date

/**
 * Get end of day (23:59:59)
 * @param {Date} [oDate] - Date (defaults to today)
 * @returns {Date | UI5Date}
 * @static
 */
DateTimeUtils.endOfDay(oDate?): Date

/**
 * Parse OData date string
 * @param {string} sDate - OData date string
 * @returns {Date | UI5Date | null}
 * @static
 */
DateTimeUtils.fromODataDateString(sDate): Date | null
```

### Formatting Methods

```javascript
/**
 * Format as locale date
 * @param {any} vValue - Date value
 * @returns {string}
 * @static
 */
DateTimeUtils.localeDate(vValue): string

/**
 * Format as locale time
 * @param {any} vValue - Time value
 * @returns {string}
 * @static
 */
DateTimeUtils.localeTime(vValue): string

/**
 * Format as locale date and time
 * @param {any} vValue - DateTime value
 * @returns {string}
 * @static
 */
DateTimeUtils.localeDateTime(vValue): string
```

### Usage Example

```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

// Get current time
const now = DateTimeUtils.now();

// Get start/end of day
const startOfDay = DateTimeUtils.startOfDay();
const endOfDay = DateTimeUtils.endOfDay();

// Format for display
const formattedDate = DateTimeUtils.localeDate(now);
const formattedTime = DateTimeUtils.localeTime(now);
const formattedDateTime = DateTimeUtils.localeDateTime(now);
```

---

## WIDGET REGISTRY

**Class:** `sap.dm.dme.pod2.widget.WidgetRegistry`

**ALL METHODS ARE STATIC**

```javascript
/**
 * Get widget class by type
 * @param {string} sWidgetType - Widget type string
 * @returns {Class<Widget> | null}
 * @static
 */
WidgetRegistry.getWidget(sWidgetType): Class<Widget> | null

/**
 * Get all registered widgets
 * @returns {Object<string, Class<Widget>>}
 * @static
 */
WidgetRegistry.getWidgets(): Object<string, Class<Widget>>

/**
 * Get widget type from class
 * @param {Class<Widget>} oWidgetClass - Widget class
 * @returns {string | null}
 * @static
 */
WidgetRegistry.getType(oWidgetClass): string | null

/**
 * Get widget display name
 * @param {Class<Widget>} oWidgetClass - Widget class
 * @returns {string}
 * @static
 */
WidgetRegistry.getDisplayName(oWidgetClass): string

/**
 * Get widget description
 * @param {Class<Widget>} oWidgetClass - Widget class
 * @returns {string}
 * @static
 */
WidgetRegistry.getDescription(oWidgetClass): string

/**
 * Check if core widget
 * @param {Class<Widget>} oWidgetClass - Widget class
 * @returns {boolean}
 * @static
 */
WidgetRegistry.isCore(oWidgetClass): boolean

/**
 * Check if custom widget
 * @param {Class<Widget>} oWidgetClass - Widget class
 * @returns {boolean}
 * @static
 */
WidgetRegistry.isCustom(oWidgetClass): boolean
```

---

## ACTION REGISTRY

**Class:** `sap.dm.dme.pod2.action.ActionRegistry`

**ALL METHODS ARE STATIC**

```javascript
/**
 * Get action class by type
 * @param {string} sActionType - Action type string
 * @returns {Class<Action> | null}
 * @static
 */
ActionRegistry.getAction(sActionType): Class<Action> | null

/**
 * Get action type from class
 * @param {Class<Action>} oActionClass - Action class
 * @returns {string | null}
 * @static
 */
ActionRegistry.getActionType(oActionClass): string | null

/**
 * Get all registered actions
 * @returns {Object<string, Class<Action>>}
 * @static
 */
ActionRegistry.getActions(): Object<string, Class<Action>>

/**
 * Get action display name
 * @param {Class<Action>} oActionClass - Action class
 * @returns {string}
 * @static
 */
ActionRegistry.getDisplayName(oActionClass): string

/**
 * Get action description
 * @param {Class<Action>} oActionClass - Action class
 * @returns {string}
 * @static
 */
ActionRegistry.getDescription(oActionClass): string

/**
 * Check if core action
 * @param {Class<Action>} oActionClass - Action class
 * @returns {boolean}
 * @static
 */
ActionRegistry.isCore(oActionClass): boolean

/**
 * Check if custom action
 * @param {Class<Action>} oActionClass - Action class
 * @returns {boolean}
 * @static
 */
ActionRegistry.isCustom(oActionClass): boolean
```

---

## COMMON PATTERNS

### Event Subscription Pattern

```javascript
class MyWidget extends Widget {
    onInit() {
        super.onInit();

        // Only subscribe in run mode
        if (PodContext.isRunMode()) {
            PodContext.subscribe(
                ModelPath.FilterResources,
                this._onResourceChanged,
                this  // Context for callback
            );
        }
    }

    _onResourceChanged(sPath, aResources) {
        // Handle resource change
        // Update UI accordingly
    }

    onExit() {
        super.onExit();

        // Always unsubscribe to prevent memory leaks
        if (PodContext.isRunMode()) {
            PodContext.unsubscribe(
                ModelPath.FilterResources,
                this._onResourceChanged,
                this
            );
        }
    }
}
```

### Action Execution Pattern

```javascript
class MyAction extends Action {
    execute(oActionContext) {
        // Get widget and event
        const oWidget = oActionContext.widget;
        const oEvent = oActionContext.event;

        // Check if should abort
        if (someCondition) {
            oActionContext.abort();
            return;
        }

        // Perform action logic
        // Can return Promise for async operations
    }
}
```

### Property Definition Pattern

```javascript
class MyWidget extends Widget {
    static PropertyId = Object.freeze({
        EnableFeature: "enableFeature",
        MaxRows: "maxRows"
    });

    getProperties() {
        return [
            new WidgetProperty({
                id: this.constructor.PropertyId.EnableFeature,
                displayName: "Enable Feature",
                description: "Enable or disable the feature",
                category: PropertyCategory.Main,
                editor: new BooleanPropertyEditor(
                    this,
                    this.constructor.PropertyId.EnableFeature,
                    true  // default value
                )
            }),
            new WidgetProperty({
                id: this.constructor.PropertyId.MaxRows,
                displayName: "Max Rows",
                description: "Maximum number of rows",
                category: PropertyCategory.Main,
                editor: new IntegerPropertyEditor(
                    this,
                    this.constructor.PropertyId.MaxRows,
                    10  // default value
                )
            })
        ];
    }

    _someMethod() {
        // Access property values
        const bEnabled = this.getPropertyValue(
            this.constructor.PropertyId.EnableFeature
        );
        const iMaxRows = this.getPropertyValue(
            this.constructor.PropertyId.MaxRows
        );
    }
}
```

### API Call Pattern with Error Handling

```javascript
import RestClient from "sap/dm/dme/pod2/api/RestClient";
import Logger from "sap/dm/dme/pod2/Logger";

class MyWidget extends Widget {
    constructor() {
        this._logger = Logger.getLogger("MyWidget");
    }

    async _fetchData() {
        const REQUEST_TIMEOUT_MS = 30000;

        try {
            const oResponse = await Promise.race([
                RestClient.post("/api/v1/endpoint", {
                    plant: PodContext.getPlant(),
                    // ... other params
                }),
                this._createTimeoutPromise(REQUEST_TIMEOUT_MS)
            ]);

            this._logger.info("Data fetched successfully");
            return oResponse;

        } catch (oError) {
            this._logger.error("Failed to fetch data", oError);
            MessageToast.show("Failed to load data");
            throw oError;
        }
    }

    _createTimeoutPromise(iTimeout) {
        return new Promise((_, reject) => {
            setTimeout(
                () => reject(new Error("Request timeout")),
                iTimeout
            );
        });
    }
}
```

---

## NOTES

1. **All PodContext methods are static** - Call directly on the class, not on instances
2. **Async patterns** - Many API and action methods return Promises
3. **I18n integration** - All classes support localization via i18n models
4. **Event handling** - Widgets emit events that can trigger actions
5. **Model binding** - Heavy use of SAPUI5 model/binding patterns
6. **Subscription pattern** - Use subscribe/unsubscribe for reactive updates
7. **Property editors** - Extensible system for widget property configuration
8. **REST/OData clients** - Support for both REST and SAP OData protocols
9. **Error handling** - Actions can abort, async methods use Promise rejection
10. **Lifecycle management** - Always call super methods and clean up properly

---

## REFERENCE

- **Source:** Official SAP Digital Manufacturing POD 2.0 JSDoc Documentation
- **Location:** https://github.com/SAP-samples/digital-manufacturing-extension-samples/blob/main/documentation/jsdoc_pod2.zip
- **Version:** Based on documentation dated August 21, 2025
- **Extracted:** March 12, 2026

---

**END OF API REFERENCE**
