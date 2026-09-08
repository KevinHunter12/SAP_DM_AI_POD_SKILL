# POD 2.0 API Complete Reference

## Overview

This document provides a comprehensive API reference for SAP Digital Manufacturing POD 2.0 development, extracted from official SAP JSDoc documentation.

---

## TABLE OF CONTENTS

1. [Common Imports Reference](#common-imports-reference)
2. [Widget Base Class](#widget-base-class)
3. [Widget Subclasses](#widget-subclasses)
4. [Property and Event Filtering](#property-and-event-filtering)
5. [PodContext](#podcontext)
6. [ModelPath Constants](#modelpath-constants)
7. [Action Base Class](#action-base-class)
8. [Core Action Classes](#core-action-classes)
9. [Property Editors](#property-editors)
10. [REST Client](#rest-client)
11. [OData Clients](#odata-clients)
12. [Public API Clients](#public-api-clients)
13. [Logger](#logger)
14. [DateTimeUtils](#datetimeutils)
15. [Widget Registry](#widget-registry)
16. [Action Registry](#action-registry)

---

## COMMON IMPORTS REFERENCE

### Control & Enum Imports

**CRITICAL**: SAPUI5 enum pseudo-module imports (`"sap/m/ButtonType"`, `"sap/m/PlacementType"`, etc.) are **deprecated** and produce console warnings. Always import from the parent library:

```javascript
// ❌ WRONG — deprecated pseudo-module imports (cause console warnings)
import PlacementType from "sap/m/PlacementType";
import ButtonType    from "sap/m/ButtonType";
import ListMode      from "sap/m/ListMode";
import MessageType   from "sap/m/MessageType";
import ValueState    from "sap/ui/core/ValueState";

// ✅ CORRECT — import library, extract types as vars
sap.ui.define([
    "sap/m/library",
    "sap/ui/core/library"
], (mobileLibrary, coreLibrary) => {
    "use strict";

    var PlacementType      = mobileLibrary.PlacementType;
    var ButtonType         = mobileLibrary.ButtonType;
    var ListMode           = mobileLibrary.ListMode;
    var ListSeparators     = mobileLibrary.ListSeparators;
    var MessageType        = mobileLibrary.MessageType;
    var FlexAlignItems     = mobileLibrary.FlexAlignItems;
    var FlexJustifyContent = mobileLibrary.FlexJustifyContent;
    var FlexWrap           = mobileLibrary.FlexWrap;
    var ValueState         = coreLibrary.ValueState;   // ValueState IS in core
    var TextAlign          = coreLibrary.TextAlign;
    var TextDirection      = coreLibrary.TextDirection;
    // ... use vars here
});
```

See [sapui5-control-apis.md](sapui5-control-apis.md#deprecated-pseudo-module-imports) for the full replacement table.

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

// ✅ Custom Controls (POD 2.0 styling-aware)
import CustomText     from "sap/dm/dme/pod2/control/CustomText";
import CustomTextArea from "sap/dm/dme/pod2/control/CustomTextArea";
import CustomHBox     from "sap/dm/dme/pod2/control/CustomHBox";
import CustomVBox     from "sap/dm/dme/pod2/control/CustomVBox";

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

---

## PROPERTY AND EVENT FILTERING

Control which properties and events are exposed in POD Designer:

```javascript
class MyWidget extends ControlWidget {
    // Only these properties available in POD Designer
    static INCLUDE_PROPERTIES = ["title", "visible", "enabled"];
    
    // Only these events available
    static INCLUDE_EVENTS = ["press"];
    
    getProperties() {
        const aAllProperties = super.getProperties();
        // Framework automatically filters based on INCLUDE_PROPERTIES
        return aAllProperties;
    }
}
```

**How it works**:
- If `INCLUDE_PROPERTIES` defined, only listed properties exposed
- If `INCLUDE_EVENTS` defined, only listed events exposed
- Empty array = NO properties/events
- Undefined = ALL properties/events

**Examples**:

```javascript
// Minimal configuration
class ReadOnlyWidget extends TextWidget {
    static INCLUDE_PROPERTIES = ["size"];
    static INCLUDE_EVENTS = [];
}

// Selective exposure
class CustomButton extends ButtonWidget {
    static INCLUDE_PROPERTIES = ["text", "icon", "type", "enabled"];
    static INCLUDE_EVENTS = ["press"];
}
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
 * REQUIRED - Get widget category for POD Designer palette
 * @returns {string}
 */
static getCategory(): string

/**
 * Get display name shown in POD Designer
 * @returns {string}
 */
static getDisplayName(): string

/**
 * Get description shown as tooltip/subtitle in POD Designer
 * @returns {string}
 */
static getDescription(): string

/**
 * Get URL for help documentation link in POD Designer.
 * If returned, a help icon link is shown next to the widget.
 * @returns {string | null}
 */
static getHelpUrl(): string | null

/**
 * Shorthand for providing a type-specific i18n model.
 * Return either a bundle name string, or a settings object.
 * The framework creates an I18nResourceModel from this automatically.
 * Prefer this over getI18nModel() for simpler setups.
 * @returns {string | $I18nResourceModelSettings}
 */
static getI18nModelSettings(): string | object

/**
 * Get default widget configuration
 * @param {string} sId - Widget ID
 * @returns {DefaultWidgetConfig}
 */
static getDefaultConfig(sId): DefaultWidgetConfig

/**
 * Get path to optional CSS stylesheet for this widget.
 * The framework loads this stylesheet before the widget is rendered.
 * @returns {string | null}  e.g. "namespace/widget/MyWidget.css"
 */
static getStyleSheet(): string | null

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

#### getStyleSheet() Example

```javascript
class MyWidget extends Widget {
    static getStyleSheet() {
        // Framework loads this CSS before rendering
        return "custom/pod2/myproject/widget/MyWidget.css";
    }
}
```

#### getHelpUrl() Example

```javascript
class MyWidget extends Widget {
    static getHelpUrl() {
        return "https://help.sap.com/docs/my-docs-page";
    }
}
```

#### getI18nModelSettings() Shorthand

```javascript
class MyWidget extends Widget {
    // Shorthand: return bundle name string
    static getI18nModelSettings() {
        return "custom.pod2.myproject.i18n.i18n";
    }

    // OR: return settings object
    static getI18nModelSettings() {
        return {
            bundleName: "custom.pod2.myproject.i18n.i18n",
            // additional I18nResourceModel settings...
        };
    }
    // NOTE: Either method works — use getI18nModel() for I18nResourceModel instance access,
    // use getI18nModelSettings() for a simpler one-liner declaration.
}
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
 * Wait for a model path value to become defined (not undefined).
 * Returns a Promise that resolves immediately if the value is already set,
 * or waits until it is set. Useful in onInit() when another widget may
 * not have populated data yet.
 * @param {string} sModelPath - Model path
 * @returns {Promise<any>}
 */
static getWhenAvailable(sModelPath): Promise<any>

/**
 * Set context value (custom properties only)
 * @param {string} sModelPath - Model path
 * @param {any} vValue - Value to set
 * @returns {boolean}
 */
static set(sModelPath, vValue): boolean

/**
 * Resolve a binding expression or object
 * @param {any} vBinding - Binding expression or binding info object
 * @param {sap.ui.core.Control} [oControl] - Optional context control
 * @returns {any} Resolved value
 */
static resolveBinding(vBinding, oControl?): any
```

#### getWhenAvailable() Example

```javascript
// Use when a required delegate value may not be loaded yet at onInit() time
async onInit() {
    await super.onInit();

    if (PodContext.isRunMode()) {
        // Wait for worklist to be populated before acting
        const aItems = await PodContext.getWhenAvailable(ModelPath.WorkListItems);
        this._updateDisplay(aItems);
    }
}
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
 * Get SFC filter values (from worklist filter bar)
 * @returns {Array<string>}
 */
static getFilterSfcs(): Array<string>

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
 * Set SFC filter values
 * @param {Array<string>} aSfcs
 */
static setFilterSfcs(aSfcs): void

/**
 * Set process lot filter
 * @param {string} sProcessLot
 */
static setFilterProcessLot(sProcessLot): void

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
 * Get plant time zone (IANA timezone string, e.g. "America/Chicago")
 * @returns {string}
 */
static getPlantTimeZone(): string

/**
 * Get current POD ID
 * @returns {string}
 */
static getPodId(): string

/**
 * Get industry type ("DISCRETE" or "PROCESS")
 * @returns {string}
 */
static getIndustryType(): string

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

/**
 * Get POD runtime object (navigation, widget access, etc.)
 * @returns {PodRuntime}
 */
static getPodRuntime(): PodRuntime
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
 * Get activity confirmation summary list
 * @returns {Array<ActivityConfirmationSummary>}
 */
static getActivityConfirmationSummaryList(): Array<ActivityConfirmationSummary>

/**
 * Get reported quantity items
 * @returns {Array<ReportedQuantity>}
 */
static getReportedQuantityItems(): Array<ReportedQuantity>

/**
 * Get reported quantity count
 * @returns {number}
 */
static getReportedQuantityCount(): number

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
 * Get execution SFC quantity (quantity set for start/complete actions)
 * @returns {number}
 */
static getExecutionSFCQuantity(): number

/**
 * Get signature history for a signature widget
 * @param {string} sSignatureWidgetId - Widget ID
 * @returns {Array<Signature>}
 */
static getSignatureHistory(sSignatureWidgetId): Array<Signature>

/**
 * Set execution SFC quantity (used by quantity widgets before SFC actions)
 * @param {number} iQuantity
 */
static setExecutionSFCQuantity(iQuantity): void

/**
 * Set selected work instruction
 * @param {WorkInstruction} oWorkInstruction
 */
static setSelectedWorkInstruction(oWorkInstruction): void

/**
 * Set activity confirmation summary list
 * @param {Array} oActivityConfirmationSummaryList
 */
static setActivityConfirmationSummaryList(oActivityConfirmationSummaryList): void

/**
 * Set reported quantity count
 * @param {number} iCount
 */
static setReportedQuantityCount(iCount): void

/**
 * Set reported quantity items
 * @param {Array} aItems
 */
static setReportedQuantityItems(aItems): void
```

---

## MODELPATH CONSTANTS

**Class:** `sap.dm.dme.pod2.context.ModelPath`

Common model paths for use with PodContext.subscribe() and PodContext.get():

```javascript
// Filter Paths
ModelPath.FilterResources
ModelPath.FilterWorkCenters
ModelPath.FilterMaterials
ModelPath.FilterOperationActivities
ModelPath.FilterSfcs          // Array<string> — SFC filter bar values
ModelPath.FilterInputType

// Work List Paths (note: plural for arrays!)
ModelPath.SelectedWorkListItems   // Array — the selected items
ModelPath.WorkListItems           // Array — all visible items
ModelPath.WorkListCount           // number — total count
ModelPath.WorkListLoading         // boolean
ModelPath.WorkListPageSize        // number
ModelPath.WorkListSorting         // Array<Sorting>
ModelPath.WorkListType            // WorkListType enum

// Operation & Activity Paths
ModelPath.OperationActivities
ModelPath.SelectedOperationActivities
ModelPath.OperationActivitiesLoading   // boolean

// Work Instructions
ModelPath.WorkInstructions
ModelPath.SelectedWorkInstruction
ModelPath.WorkInstructionsLoading      // boolean

// Reported Quantities
ModelPath.ReportedQuantityItems
ModelPath.ReportedQuantityCount        // number
ModelPath.ReportedQuantityLoading      // boolean

// Activity Confirmation
ModelPath.ActivityConfirmationSummaryList

// Goods Receipt
ModelPath.GoodsReceiptSummary
ModelPath.GoodsReceiptLineItems

// Data Collection Paths
ModelPath.DataCollectionGroups
ModelPath.DataCollectionSelectedGroup
ModelPath.DataCollectionLog

// Other
ModelPath.ExecutionSFCQuantity    // number — set by quantity widgets
ModelPath.PodId                   // string — current POD ID
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
 * OPTIONAL lifecycle hook — called before the FIRST execute() in a sequence.
 * All actions in the sequence are initialized (onInit called) in order and
 * AWAITED before any action's execute() is called.
 * @returns {void | Promise<void>}
 */
onInit?(): void | Promise<void>

/**
 * OPTIONAL lifecycle hook — called after ALL actions in the sequence have executed.
 * @returns {void | Promise<void>}
 */
onExit?(): void | Promise<void>

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

/**
 * STATIC — Return property name(s) that hold references to widget IDs.
 * When a referenced widget is renamed in POD Designer, the action's
 * property value will be automatically updated to match.
 * @returns {string | Array<string> | undefined}
 */
static getWidgetReferenceProperties(): string | Array<string> | undefined
```

### Action Lifecycle Sequence

When multiple actions are assigned to a widget event, they run as a sequence:

1. `onInit()` called on **all** actions in order (awaited sequentially)
2. `execute()` called on each action in order
   - Any action can call `oActionContext.abort()` to skip remaining actions
3. `onExit()` called on **all** actions after the sequence completes

### Action Context

**Class:** `sap.dm.dme.pod2.action.ActionContext`

```javascript
// Properties
widget: Widget         // Widget that triggered action
event: Event          // UI event that triggered action

/**
 * Abort — skip all subsequent actions in the sequence
 */
abort(): void

/**
 * Check if a previous action in the sequence called abort()
 * IMPORTANT: Call this at the start of execute() to respect abort requests
 * @returns {boolean}
 */
isAborted(): boolean
```

### Correct Action Pattern with Abort Handling

```javascript
class MyAction extends Action {
    async onInit() {
        // Setup code that runs before execute() — e.g. validate prerequisites
        const oItem = PodContext.getLastSelectedWorkListItem();
        if (!oItem) {
            this._abortReason = "No item selected";
        }
    }

    async execute(oActionContext) {
        // ALWAYS check isAborted() first to respect prior action decisions
        if (oActionContext.isAborted()) {
            return;
        }

        // Abort if our own init found a problem
        if (this._abortReason) {
            MessageHistory.showError(this._abortReason);
            oActionContext.abort();
            return;
        }

        // Actual logic
        await ApiClient.sfc.sfcStart({ ... });
    }
}
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
new SelectPropertyEditor(
    oPropertyAccessor,  // Property accessor (pass `this`)
    sPropertyId,        // Property ID string (e.g. "myProp")
    vItems?,            // Plain object: { key: "Display Text", key2: "Text 2" }
                        // OR array of strings (each used as both key and text)
                        // NOT Item instances, NOT [{ key, text }] arrays
    sDefaultKey?        // Key to select by default if no value already saved
)
```

**CRITICAL: `vItems` must be a plain `{ key: "text" }` object.**

```javascript
// ✅ CORRECT — plain object, keys are option keys, values are display text
getProperties() {
    return [
        new WidgetProperty({
            displayName: this.getI18nText("property.layout"),
            category: "Main",
            propertyEditor: new SelectPropertyEditor(this, "layout", {
                layout1: this.getI18nText("layout.option1"),
                layout2: this.getI18nText("layout.option2"),
                layout3: this.getI18nText("layout.option3")
            }, "layout1")
        })
    ];
}

// ✅ ALSO CORRECT — array of strings (key === text)
new SelectPropertyEditor(this, "size", ["small", "medium", "large"], "medium")

// ❌ WRONG — array of objects: shows "[object Object]"
new SelectPropertyEditor(this, "layout", [{ key: "layout1", text: "Layout 1" }])

// ❌ WRONG — Item instances: shows "Element sap.ui.core.Item#__item79"
new SelectPropertyEditor(this, "layout", [new Item({ key: "layout1", text: "Layout 1" })])
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

### TimeTracking API

**Client:** `ApiClient.timeTracking`  
**Source:** `sap/dm/dme/pod2/api/timetracking/TimeTrackingPublicApiClient`

```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";

// Clock operators in/out for a shift
await ApiClient.timeTracking.clockIn(oRequest)
await ApiClient.timeTracking.clockOut(oRequest)

// Labor tracking (direct labor on/off)
await ApiClient.timeTracking.laborOn(oRequest)
await ApiClient.timeTracking.laborOff(oRequest)

// Indirect labor (non-production activities)
await ApiClient.timeTracking.indirectLaborOn(oRequest)
await ApiClient.timeTracking.indirectLaborOff(oRequest)

// Production time tracking
await ApiClient.timeTracking.productionTimeOn(oRequest)
await ApiClient.timeTracking.productionTimeOff(oRequest)

// Resource usage tracking
await ApiClient.timeTracking.resourceUsageOn(oRequest)
await ApiClient.timeTracking.resourceUsageOff(oRequest)
```

---

### Notable Internal Clients

> **Warning:** Internal APIs power SAP's own widgets and may change without notice between versions. Prefer public APIs where available. Test thoroughly after upgrades.
>
> **Note on `ApiClient.internal.*` sub-namespaces:** The `ApiClient.internal.plant`, `ApiClient.internal.oee`, etc. paths shown below reflect the logical structure of the internal clients. The exact wiring under `ApiClient.internal` depends on the version of `ApiClient.js`. When in doubt, import the client class directly (import path shown for each) rather than going through `ApiClient.internal`.

#### PlantInternalApiClient — Resource, Work Center & Plant Data

The most useful internal client for plugin development. Provides resource and work center lookups not available through the public `ResourcePublicApiClient`.

**Import:** `sap/dm/dme/pod2/api/internal/plant/PlantInternalApiClient`
Accessed via `ApiClient.internal.plant` or imported directly.

```javascript
// Resource lookups
await ApiClient.internal.plant.getResourceByKey({ plant, resource, version })
await ApiClient.internal.plant.getResourcesOData({ plant, ...filters })
await ApiClient.internal.plant.getResourcesForWorkCenters({ plant, workCenters })
await ApiClient.internal.plant.getResourceHierarchyData({ plant, resource })
await ApiClient.internal.plant.getDowntimeResourceStatuses({ plant, resources })
await ApiClient.internal.plant.getResourceStatusesOData({ plant, ...filters })

// Work center lookups
await ApiClient.internal.plant.getWorkCenterByKey({ plant, workCenter, version })
await ApiClient.internal.plant.getWorkCentersForUser({ plant, userId })
await ApiClient.internal.plant.isUserAssignedToWorkCenter({ plant, workCenter, userId })

// Users
await ApiClient.internal.plant.getUsers({ plant, ...filters })
await ApiClient.internal.plant.getUsersPage({ plant, page, pageSize })
await ApiClient.internal.plant.getUsersByIds({ plant, userIds })

// Reason codes (for OEE/downtime)
await ApiClient.internal.plant.getReasonCodeObject({ plant, reasonCode })
await ApiClient.internal.plant.getReasonCodeObjectByRef({ plant, ref })
await ApiClient.internal.plant.getReasonCodeObjectsByTimeElement({ plant, timeElementRef })
await ApiClient.internal.plant.findAllAssignedReasonCodesToResource({ plant, resource })
await ApiClient.internal.plant.findAllResourceReasonCodeChildren({ plant, parentRef })
await ApiClient.internal.plant.findTimeElementsByType({ plant, type })

// Custom fields
await ApiClient.internal.plant.getCustomFieldDefinitions({ plant, businessObject })
await ApiClient.internal.plant.getResourceCustomFieldDefinitions({ plant })
```

#### OeeInternalApiClient — Downtime & Speed Loss

**Import:** `sap/dm/dme/pod2/api/internal/oee/OeeInternalApiClient`

```javascript
// Downtime management
await ApiClient.internal.oee.createDowntimes(oRequest)
await ApiClient.internal.oee.updateDowntime(oRequest)
await ApiClient.internal.oee.deleteDowntimes(oRequest)
await ApiClient.internal.oee.splitDowntime(oRequest)
await ApiClient.internal.oee.getDowntimesForWorkCenter(oRequest)

// Speed loss
await ApiClient.internal.oee.createTaggedSpeedLoss(oRequest)
await ApiClient.internal.oee.updateTaggedSpeedLoss(oRequest)
await ApiClient.internal.oee.deleteTaggedSpeedLoss(oRequest)
await ApiClient.internal.oee.getTaggedSpeedLoss(oRequest)
await ApiClient.internal.oee.getSpeedLossSummary(oRequest)
```

#### SignatureInternalApiClient — Digital Signature / Buyoff

**Import:** `sap/dm/dme/pod2/api/internal/signature/SignatureInternalApiClient`  
Use `PodContext.getSignatureHistory(sWidgetId)` to read stored signatures.

```javascript
await ApiClient.internal.signature.sign(oRequest)          // record a digital signature
await ApiClient.internal.signature.getSignatureLogs(oRequest) // retrieve signature history
```

#### DemandInternalApiClient — Order Header Text & Custom Fields

**Import:** `sap/dm/dme/pod2/api/internal/demand/DemandInternalApiClient`

```javascript
await ApiClient.internal.demand.findByPlantAndShopOrder({ plant, shopOrder })
await ApiClient.internal.demand.getOrderHeaderText({ plant, shopOrder })   // rich text header
await ApiClient.internal.demand.getCustomFieldDefinitions({ plant })
```

#### DataScanInternalApiClient — Barcode / Scan Entry

**Import:** `sap/dm/dme/pod2/api/internal/datascan/DataScanInternalApiClient`

```javascript
await ApiClient.internal.datascan.scanData(oRequest)              // process a scanned value
await ApiClient.internal.datascan.getDataScanConfig(oRequest)     // scan routing config
await ApiClient.internal.datascan.getConsumerMappings(oRequest)   // field → consumer map
```

#### ProcessEngineInternalApiClient — Process Manufacturing Workflows

**Import:** `sap/dm/dme/pod2/api/internal/processengine/ProcessEngineInternalApiClient`

```javascript
await ApiClient.internal.processengine.start(oRequest)         // start a process engine task
await ApiClient.internal.processengine.completeTask(oRequest)  // complete a task step
```

---

### Other Available APIs

```javascript
ApiClient.alert          // Alert APIs
ApiClient.inventory      // Inventory operations
ApiClient.uom            // Unit of measure
ApiClient.user           // User APIs
ApiClient.workinstruction // Work instructions
ApiClient.processorder   // Process orders
ApiClient.execution      // Execution APIs
ApiClient.mdo            // Master data object APIs (MDO — distinct from standard REST)
ApiClient.numbering      // SFC/entity number generation
ApiClient.routing        // Routing lookups
ApiClient.operationactivity // Operation activity lookups
ApiClient.datatype       // Custom data types
ApiClient.datafields     // Custom data field definitions
ApiClient.timeTracking   // Time tracking (clockIn/Out, laborOn/Off — see above)
ApiClient.internal       // Internal APIs (SAP widgets only — may change without notice)
```

---

## DATETIMEUTILS

**Class:** `sap.dm.dme.pod2.DateTimeUtils`

**ALL METHODS ARE STATIC**

### Date/Time Methods

```javascript
/**
 * Get current date/time in plant timezone
 * @returns {Date | UI5Date}
 * @static
 */
DateTimeUtils.now(): Date

/**
 * Get start of day (00:00:00.000) in plant timezone
 * @param {Date} [oDate] - Date (defaults to today)
 * @returns {Date | UI5Date}
 * @static
 */
DateTimeUtils.startOfDay(oDate?): Date

/**
 * Get end of day (23:59:59.999) in plant timezone
 * @param {Date} [oDate] - Date (defaults to today)
 * @returns {Date | UI5Date}
 * @static
 */
DateTimeUtils.endOfDay(oDate?): Date

/**
 * Parse OData date string "/Date(timestamp)/"
 * @param {string} sDate - OData date string
 * @returns {Date | UI5Date | null} null if parsing fails
 * @static
 */
DateTimeUtils.fromODataDateString(sDate): Date | null
```

### Usage Example

```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

// Get current time in plant timezone
const now = DateTimeUtils.now();

// Get start/end of today in plant timezone
const startOfDay = DateTimeUtils.startOfDay();
const endOfDay = DateTimeUtils.endOfDay();

// For a specific date
const startOfSpecificDay = DateTimeUtils.startOfDay(someDate);

// Parse OData date from API response
const oDate = DateTimeUtils.fromODataDateString("/Date(1623668012060)/");
if (oDate) {
    // Valid date
}
```

> **Note:** `DateTimeUtils` does NOT have `localeDate()`, `localeTime()`, or `localeDateTime()` methods. For locale-formatted display, use SAPUI5's `DateFormat` directly or bind dates to UI5 controls with format options.

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

## PODRUNTIME

**Class:** `sap.dm.dme.pod2.runtime.PodRuntime`

Access via `PodContext.getPodRuntime()` or `this.getPodRuntime()` inside any widget or action.

### Navigation

```javascript
/**
 * Navigate to a page by ID. Protected against nested navigation.
 * @param {string} sPageId - Page ID as configured in POD Designer
 * @returns {Promise<void>}
 */
async navigateToPage(sPageId): Promise<void>

/**
 * Navigate back to the previous page.
 * @returns {Promise<void>}
 */
async navigateBack(): Promise<void>

/**
 * Navigate to a widget by ID (handles page navigation and tab switching).
 * @param {string} sWidgetId - Widget ID
 * @returns {Promise<void>}
 */
async navigateToWidget(sWidgetId): Promise<void>

/**
 * Navigate to a widget by type string. Only navigates if exact match found.
 * @param {string} sWidgetType - Widget type (e.g., "sap.dm.dme.pod2.widget.WorkListTableWidget")
 * @returns {Promise<void>}
 */
async navigateToWidgetByType(sWidgetType): Promise<void>

/**
 * Open a dialog widget by ID.
 * @param {string} sDialogId - Dialog widget ID
 * @returns {Promise<void>}
 */
async showDialog(sDialogId): Promise<void>
```

### Widget Access

```javascript
/**
 * Get a widget instance by its ID
 * @param {string} sId - Widget ID
 * @returns {Widget}
 */
getWidget(sId): Widget

/**
 * Get the Widget for a given SAPUI5 control
 * @param {sap.ui.core.Element} oView - The SAPUI5 control
 * @returns {Widget}
 */
getWidgetForView(oView): Widget

/**
 * Get Widgets for an array of SAPUI5 controls (order matches input)
 * @param {Array<sap.ui.core.Element>} aViews
 * @returns {Array<Widget>}
 */
getWidgetsForViews(aViews): Array<Widget>

/**
 * Execute callback for each instantiated widget across all pages/dialogs
 * @param {Function} fnCallback - (oWidget: Widget) => void
 */
forEachWidget(fnCallback): void

/**
 * Find a widget config matching a predicate. Stops at first match.
 * @param {Function} fnCompare - (oWidgetConfig: WidgetConfig) => boolean
 * @returns {WidgetConfig}
 */
findWidgetConfig(fnCompare): WidgetConfig

/**
 * Execute callback for each widget configuration (regardless of instantiation)
 * @param {Function} fnCallback - (oWidgetConfig: WidgetConfig) => void
 */
forEachWidgetConfig(fnCallback): void

/**
 * Execute callback for each action configuration
 * @param {Function} fnCallback - (oActionConfig, { widgetConfig, event }) => void
 */
forEachActionConfig(fnCallback): void
```

### Page / View Access

```javascript
/**
 * Get the current page control
 * @returns {sap.ui.core.Control}
 */
getCurrentPage(): sap.ui.core.Control

/**
 * Get the current page widget
 * @returns {PageWidget}
 */
getCurrentPageWidget(): PageWidget

/**
 * Get the main app view
 * @returns {sap.m.App}
 */
getView(): sap.m.App

/**
 * Get the full POD configuration object
 * @returns {PodConfig}
 */
getPodConfig(): PodConfig
```

### Usage Examples

```javascript
// Programmatic navigation from a button press
async _onNavigateToDetailPage() {
    const oRuntime = PodContext.getPodRuntime();
    await oRuntime.navigateToPage("detailPage");
}

// Open a dialog programmatically
async _onOpenDialog() {
    const oRuntime = this.getPodRuntime();
    await oRuntime.showDialog("myCustomDialog");
}

// Find another widget and call a method on it
_onRefreshOtherWidget() {
    const oRuntime = this.getPodRuntime();
    const oOtherWidget = oRuntime.getWidget("workListTable");
    if (oOtherWidget && typeof oOtherWidget.refresh === "function") {
        oOtherWidget.refresh();
    }
}

// Find a widget config by type
_findWorkListConfig() {
    const oRuntime = this.getPodRuntime();
    return oRuntime.findWidgetConfig(oConfig =>
        oConfig.type === "sap.dm.dme.pod2.widget.WorkListTableWidget"
    );
}
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

## DATA TYPES

### Type Hierarchy

```
BaseWorkListItem
  └── WorkListItem      (discrete industry, SFC-based)
  └── OrderWorkListItem (process industry, order-based)

BaseOperationWorkItem
  └── OperationActivity (discrete industry)
  └── Phase             (process industry)
```

---

### BaseWorkListItem

**Class:** `sap.dm.dme.pod2.context.type.BaseWorkListItem`

Base type for all work list rows. Methods are defined here; properties on subclasses.

```javascript
/**
 * Get unique identifier (SFC or order number depending on subtype)
 * @returns {string}
 */
getIdentifier(): string

/**
 * Validate the item. Throws Error if required fields missing.
 */
validate(): void
```

**Common Properties** (on both WorkListItem and OrderWorkListItem):

| Property | Type | Description |
|----------|------|-------------|
| `sfc` | `string` | SFC number |
| `material` | `string` | Material number |
| `materialDescription` | `string` | |
| `materialVersion` | `string` | |
| `routing` | `string` | |
| `routingVersion` | `string` | |
| `order` | `string` | Shop/process order number |
| `sfcStatusCode` | `SFCStatusCode` | |
| `sfcStatusDescription` | `string` | |
| `sfcQuantity` | `number` | |
| `operationActivity` | `string` | Current/last operation |
| `workCenter` | `string` | |
| `customFields` | `Object.<string, string>` | Custom data — **OBJECT, not array!** |

---

### WorkListItem (Discrete Industry)

**Class:** `sap.dm.dme.pod2.context.type.WorkListItem`

Extends `BaseWorkListItem`. Full properties:

| Property | Type |
|----------|------|
| `sfcBatchNumber` | `string` |
| `orderBatchNumber` | `string` |
| `sfcQuantityInQueue` | `number` |
| `sfcQuantityInWork` | `number` |
| `sfcQuantityCompletePending` | `number` |
| `sfcCompletePending` | `boolean` |
| `sfcStartDate` | `UI5Date` |
| `sfcDateQueued` | `UI5Date` |
| `sfcDueDate` | `UI5Date` |
| `orderPlannedStartDate` | `UI5Date` |
| `orderScheduledStartDate` | `UI5Date` |
| `orderScheduledCompletionDate` | `UI5Date` |
| `operationActivityDescription` | `string` |
| `operationActivityGroup` | `string` |
| `stepId` | `string` |
| `resource` | `string` |
| `priority` | `string` |
| `processLot` | `string` |
| `customer` | `string` |
| `customerOrder` | `string` |
| `rmaNumber` | `string` |
| `materialGroup` | `string` |

---

### OrderWorkListItem (Process Industry)

**Class:** `sap.dm.dme.pod2.context.type.OrderWorkListItem`

Extends `BaseWorkListItem`. Additional properties for process industry:

| Property | Type |
|----------|------|
| `orderExecutionStatus` | `ExecutionStatus` |
| `orderReleaseStatus` | `ReleaseStatus` |
| `orderQuantityPlanned` | `number` |
| `orderQuantityCompleted` | `number` |
| `orderQuantityPlannedInProductionUom` | `number` |
| `sfcQuantityInProductionUom` | `number` |
| `sfcQuantityCompleted` | `number` |
| `baseCommercialUom` | `string` |
| `productionCommercialUom` | `string` |
| `erpAutoGRStatus` | `boolean` |
| `coAndByProductsIndicator` | `string` |
| `bom` | `string` |
| `bomType` | `string` |
| `bomVersion` | `string` |

---

### BaseOperationWorkItem

**Class:** `sap.dm.dme.pod2.context.type.BaseOperationWorkItem`

Base for OperationActivity and Phase.

```javascript
isActive(): boolean    // True if status is In Queue or In Work
isComplete(): boolean  // True if statusComplete = true
isInQueue(): boolean   // True if statusInQueue = true
```

**Common Properties:**

| Property | Type |
|----------|------|
| `operationActivity` | `string` |
| `operationActivityGroup` | `string` |
| `stepId` | `string` |
| `workCenter` | `string` |
| `resource` | `string` (often null — use `getFilterResources()`) |
| `quantity` | `number` |
| `quantityComplete` | `number` |
| `quantityInQueue` | `number` |
| `quantityInWork` | `number` |
| `scheduleStartDate` | `UI5Date` |
| `scheduleEndDate` | `UI5Date` |
| `statusNew` | `boolean` |
| `statusBypassed` | `boolean` |
| `statusInQueue` | `boolean` |
| `statusInWork` | `boolean` |
| `statusComplete` | `boolean` |
| `statusCompletePending` | `boolean` |

---

### OperationActivity (Discrete Industry)

**Class:** `sap.dm.dme.pod2.context.type.OperationActivity`

Extends `BaseOperationWorkItem`. Additional properties:

| Property | Type |
|----------|------|
| `sfc` | `string` |
| `material` | `string` |
| `routing` | `string` |
| `routingVersion` | `string` |
| `previouslyStarted` | `boolean` |
| `priority` | `string` |
| `quantityReject` | `number` |
| `quantityCompletePending` | `number` |
| `dueDate` | `UI5Date` |
| `plannedStartDate` | `UI5Date` |
| `plannedEndDate` | `UI5Date` |
| `opSplitId` | `number` |
| `splitQuantity` | `number` |
| `laboredOperators` | `Array<string>` |

---

### Phase (Process Industry)

**Class:** `sap.dm.dme.pod2.context.type.Phase`

Extends `BaseOperationWorkItem`. Additional properties for phases:

| Property | Type |
|----------|------|
| `description` | `string` |
| `actualStartDate` | `UI5Date` |
| `actualEndDate` | `UI5Date` |
| `userAuthorizedForWorkCenter` | `boolean` |

---

### Sfc

**Class:** `sap.dm.dme.pod2.context.type.Sfc`

Simple SFC data type (different from WorkListItem — this is the SFC object itself):

| Property | Type |
|----------|------|
| `sfc` | `string` |
| `status` | `SfcStatus` (enum) |
| `quantity` | `number` |
| `order` | `string` |
| `material` | `string` |
| `materialVersion` | `string` |
| `createdAtDate` | `UI5Date` |

---

### ReportedQuantity

**Class:** `sap.dm.dme.pod2.context.type.ReportedQuantity`

Access via `PodContext.getReportedQuantityItems()`.

| Property | Type |
|----------|------|
| `yieldQuantity` | `number` |
| `yieldUnitOfMeasure` | `UnitOfMeasure` |
| `yieldActivityLogId` | `string` |
| `scrapQuantity` | `number` |
| `scrapUnitOfMeasure` | `UnitOfMeasure` |
| `scrapActivityLogId` | `string` |
| `scrapReasonCode` | `string` |
| `scrapReasonCodes` | `Array<string>` |
| `scrapReasonCodeDescription` | `string` |
| `resource` | `string` |
| `resourceDescription` | `string` |
| `userId` | `string` |
| `status` | `string` |
| `postingDate` | `UI5Date` |
| `createdDate` | `UI5Date` |

---

### Resource

**Class:** `sap.dm.dme.pod2.context.type.Resource`

| Property | Type |
|----------|------|
| `plant` | `string` |
| `resource` | `string` |
| `description` | `string` |
| `status` | (enum) |
| `workCenter` | `string` |
| `efficiency` | `number` |
| `sfcLimit` | `number` |
| `resourceTypes` | (array) |
| `processResource` | `boolean` |
| `customValues` | `Object.<string, string>` |

---

### WorkCenter

**Class:** `sap.dm.dme.pod2.context.type.WorkCenter`

| Property | Type |
|----------|------|
| `plant` | `string` |
| `workCenter` | `string` |
| `description` | `string` |
| `status` | `"ENABLED"` |
| `maxPeople` | `number` |
| `minPeople` | `number` |
| `isErp` | `boolean` |
| `customValues` | `Object.<string, string>` |

---

### Material

**Class:** `sap.dm.dme.pod2.context.type.Material`

| Property | Type |
|----------|------|
| `material` | `string` |
| `version` | `string` |
| `currentVersion` | `boolean` |
| `description` | `string` |
| `materialType` | (enum) |
| `status` | (enum) |
| `unitOfMeasure` | `string` |
| `lotSize` | `number` |
| `erpBackflushing` | `boolean` |
| `customValues` | `Object.<string, string>` |

---

### Plant

**Class:** `sap.dm.dme.pod2.context.type.Plant`

Available via `ApiClient` queries; `industryType` determines discrete vs. process.

| Property | Type |
|----------|------|
| `plant` | `string` |
| `description` | `string` |
| `timeZone` | `string` (IANA) |
| `industryType` | `"DISCRETE"` or `"PROCESS"` |
| `integrationMode` | `string` |
| `erpDestination` | `string` |
| `erpLanguage` | `string` |
| `isLocal` | `boolean` |

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



## PodContext Direct Getters

Direct getters provide one-time access to context without subscription.

### When to Use Direct Getters vs Subscription

| Use Direct Getter | Use Subscription |
|-------------------|------------------|
| One-time read during onInit() | React to context changes |
| Building request objects | Update UI when context changes |
| Button click handlers | Data refresh on selection |
| Validation checks | Live filtering/updates |

### Usage Examples

**Example 1: Building request objects**
```javascript
_getRequest() {
    const aSelectedOps = PodContext.getSelectedOperationActivities();
    if (!aSelectedOps || aSelectedOps.length === 0) return null;

    return {
        plant: PodContext.getPlant(),
        sfcs: aSelectedOps.map(op => op.sfc),
        operations: aSelectedOps.map(op => op.operationActivity)
    };
}
```

**Example 2: Production Pattern - Subscribe + initial getter call**

This is the pattern used by SAP production widgets (MaterialImageWidget, OrderHeaderTextWidget):

```javascript
async onInit() {
    await super.onInit();
    PodContext.subscribe(
        ModelPath.SelectedWorkListItems,
        this._onSelectionChanged,
        this
    );
    this._onSelectionChanged(PodContext.getSelectedWorkListItems());
}

_onSelectionChanged(aItems) {
    if (aItems && aItems.length > 0) {
        this._loadData(aItems[0]);
    }
}
```

---

### ⭐ CRITICAL: Getting Selected Operation, Resource, and Worklist Data

**This is the most common pattern in POD 2.0 plugins. Used by 90% of official SAP widgets.**

**⚠️ IMPORTANT: Plugins must be ADAPTIVE to work in different POD configurations!**

Some PODs have OperationActivity widgets, some only have WorkList widgets, some have both. Your plugin must detect which widgets are available and adapt accordingly.

#### Adaptive Pattern (REQUIRED - Works in All POD Configurations)

```javascript
// ✅ CORRECT - Adaptive pattern that handles all POD configurations
async _onButtonPress() {
    // Get filtered resources (available in all configurations)
    const aFilterResources = PodContext.getFilterResources();
    const sResource = aFilterResources?.[0]?.resource || null;

    // Try to get from both widget types
    const oLastSelectedOperation = PodContext.getLastSelectedOperationActivity();
    const oLastSelectedWorkListItem = PodContext.getLastSelectedWorkListItem();

    let sSfc, sOperation, sWorkCenter, sStepId;

    if (oLastSelectedOperation && oLastSelectedWorkListItem) {
        // Pattern 1: POD has BOTH OperationActivity widget AND WorkList widget
        // This is the most common configuration in production PODs
        sSfc = oLastSelectedWorkListItem.sfc;
        sOperation = oLastSelectedOperation.operationActivity;
        sWorkCenter = oLastSelectedOperation.workCenter;
        sStepId = oLastSelectedOperation.stepId;

        this.#oLog.info("Using OperationActivity + WorkList pattern");

    } else if (oLastSelectedWorkListItem) {
        // Pattern 2: POD has ONLY WorkList widget (no OperationActivity widget)
        // Get operation data from worklist item
        sSfc = oLastSelectedWorkListItem.sfc;
        sOperation = oLastSelectedWorkListItem.operationActivity;
        sWorkCenter = oLastSelectedWorkListItem.workCenter;
        sStepId = oLastSelectedWorkListItem.stepId;

        this.#oLog.info("Using WorkList-only pattern");

    } else {
        // Pattern 3: Fallback to array if getLastSelected* returns null
        const aSelectedItems = PodContext.getSelectedWorkListItems();

        if (Array.isArray(aSelectedItems) && aSelectedItems.length > 0) {
            const oWorkListItem = aSelectedItems[0];
            sSfc = oWorkListItem.sfc;
            sOperation = oWorkListItem.operationActivity;
            sWorkCenter = oWorkListItem.workCenter;
            sStepId = oWorkListItem.stepId;

            this.#oLog.info("Using WorkList array fallback pattern");
        } else {
            MessageHistory.showError(this.getI18nText("error.noSelection"));
            this.#oLog.error("No worklist item or operation selected");
            return;
        }
    }

    // Validate required data
    if (!sSfc || !sOperation) {
        MessageHistory.showError(this.getI18nText("error.invalidSelection"));
        this.#oLog.error("Missing SFC or Operation", { sSfc, sOperation });
        return;
    }

    // Build API request with required fields
    const oRequest = {
        plant: PodContext.getPlant(),
        sfc: sSfc,
        operation: sOperation,
        startDateTime: new Date().toISOString()
    };

    // Add optional fields if available
    if (sResource) {
        oRequest.resource = sResource;
    }
    if (sWorkCenter) {
        oRequest.workCenter = sWorkCenter;
    }
    if (sStepId) {
        oRequest.stepId = sStepId;
    }

    // Call API
    await ApiClient.sfc.sfcStart(oRequest);
}
```

#### POD Configuration Matrix

| POD Configuration | Widgets Present | getLastSelectedOperationActivity() | getLastSelectedWorkListItem() | Pattern to Use |
|-------------------|-----------------|-----------------------------------|-------------------------------|----------------|
| **OperationActivity + WorkList** | Both | ✅ Returns data | ✅ Returns data | Use operation from OperationActivity, SFC from WorkList |
| **WorkList Only** | WorkList | ❌ Returns null | ✅ Returns data | Use everything from WorkList |
| **OperationActivity Only** | OperationActivity | ✅ Returns data | ❌ Returns null | Use everything from OperationActivity array |
| **Neither** | Other widgets | ❌ Returns null | ❌ Returns null | Fall back to array getters |

#### Why Adaptive Patterns Are Required

**Non-adaptive plugins will fail in certain POD configurations:**

```javascript
// ❌ BAD - Only works if POD has OperationActivity widget
async _onButtonPress() {
    const oOp = PodContext.getLastSelectedOperationActivity();
    
    if (!oOp) {
        MessageHistory.showError("No operation selected");
        return;  // 💥 FAILS in WorkList-only PODs!
    }
    
    const sOperation = oOp.operationActivity;
    // ...
}

// ✅ GOOD - Works in any POD configuration
async _onButtonPress() {
    const oOp = PodContext.getLastSelectedOperationActivity();
    const oWL = PodContext.getLastSelectedWorkListItem();

    let sOperation, sSfc;

    if (oOp && oWL) {
        // Both widgets present - prefer OperationActivity for operation
        sOperation = oOp.operationActivity;
        sSfc = oWL.sfc;
    } else if (oWL) {
        // WorkList only - use it for both
        sOperation = oWL.operationActivity;
        sSfc = oWL.sfc;
    } else if (oOp) {
        // OperationActivity only
        sOperation = oOp.operationActivity;
        sSfc = oOp.sfc;
    } else {
        MessageHistory.showError("No selection");
        return;
    }
    // ...
}
```

#### Real-World POD Configurations

**Configuration 1: Standard Production POD**
```
Widgets: OperationActivity Table + WorkList + Resource Filter
Result: 
  - getLastSelectedOperationActivity() ✅ Returns data
  - getLastSelectedWorkListItem() ✅ Returns data
  - getFilterResources() ✅ Returns array
```

**Configuration 2: Simple Execution POD**
```
Widgets: WorkList Only
Result:
  - getLastSelectedOperationActivity() ❌ Returns null
  - getLastSelectedWorkListItem() ✅ Returns data
  - getFilterResources() ⚠️ Returns empty array
```

**Configuration 3: Phase-Based POD**
```
Widgets: OperationActivity Table (phases) Only
Result:
  - getLastSelectedOperationActivity() ✅ Returns data
  - getLastSelectedWorkListItem() ❌ Returns null
  - getFilterResources() ✅ Returns array
```

#### Key Getters for Operation/SFC Data

| Method | Returns | Use Case |
|--------|---------|----------|
| `getLastSelectedOperationActivity()` | Single `OperationActivity` object | Get **operation** for current action |
| `getLastSelectedWorkListItem()` | Single `WorkListItem` object | Get **SFC** for current action |
| `getFilterResources()` | Array of `Resource` objects | Get currently filtered **resources** |
| `getSelectedOperationActivities()` | Array of `OperationActivity` objects | Multi-select operations |
| `getSelectedWorkListItems()` | Array of `WorkListItem` objects | Multi-select SFCs |

#### OperationActivity Object Structure

```javascript
{
    sfc: "SFC_12345",
    operationActivity: "OP10-ASSEMBLY",  // ← Use this for operation!
    operationActivityDescription: "Assembly Operation",
    workCenter: "WC-001",
    stepId: "10",
    resource: null,  // Often null - use getFilterResources() instead
    statusComplete: false,
    statusInWork: true
}
```

#### WorkListItem Object Structure

```javascript
{
    sfc: "SFC_12345",               // ← Use this for SFC!
    material: "MATERIAL1",
    order: "SHOP_ORDER_001",
    workCenter: "WC-001",
    operationActivity: "OP10-ASSEMBLY",
    sfcStatusCode: "402",
    sfcStatusDescription: "In Queue",
    sfcQuantity: 10
}
```

#### Resource Object Structure

```javascript
{
    resource: "RESOURCE_001",       // ← Use this for resource!
    resourceType: "EQUIPMENT",
    description: "Assembly Station 1"
}
```

#### ❌ Common Mistakes

```javascript
// ❌ WRONG - Using worklist item for operation
const oWorkListItem = PodContext.getLastSelectedWorkListItem();
const sOperation = oWorkListItem.operationActivity;  // This may be stale!

// ✅ CORRECT - Use OperationActivity for operation
const oOperation = PodContext.getLastSelectedOperationActivity();
const sOperation = oOperation.operationActivity;

// ❌ WRONG - Using operation object for resource
const oOperation = PodContext.getLastSelectedOperationActivity();
const sResource = oOperation.resource;  // Often null!

// ✅ CORRECT - Use getFilterResources() for resource
const aResources = PodContext.getFilterResources();
const sResource = aResources?.[0]?.resource || null;

// ❌ WRONG - Using getResource() (doesn't exist!)
const sResource = PodContext.getResource();  // TypeError!

// ✅ CORRECT - Use getFilterResources() array
const aResources = PodContext.getFilterResources();
const sResource = aResources?.[0]?.resource;
```

#### Real-World Examples from SAP Official Plugins

**Activity Confirmation Plugin:**
```javascript
// From: ActivityConfirmationTableWidget.js
const oLastSelectedOperation = PodContext.getLastSelectedOperationActivity();
const oLastSelectedWorkListItem = PodContext.getLastSelectedWorkListItem();

let sWorkCenter = oLastSelectedOperation.workCenter;
if (!sWorkCenter && oLastSelectedWorkListItem instanceof WorkListItem) {
    sWorkCenter = oLastSelectedWorkListItem.workCenter;
}

const oRequest = {
    shopOrder: oLastSelectedWorkListItem.order,
    batchId: oLastSelectedWorkListItem.sfc,
    operationActivity: oLastSelectedOperation.operationActivity,
    workCenter: sWorkCenter,
    stepId: oLastSelectedOperation.stepId
};
```

**Quantity Confirmation Plugin:**
```javascript
// From: QuantityConfirmationDelegate.js
const oOperationActivityItem = PodContext.getLastSelectedOperationActivity();
const oWorkListItem = PodContext.getLastSelectedWorkListItem();

const oRequest = {
    shopOrder: oWorkListItem.order,
    batchId: oWorkListItem.sfc,
    phase: oOperationActivityItem.operationActivity
};
```

**Downtime Widget:**
```javascript
// From: DowntimeWidget.js
const sWorkCenter = PodContext.getFilterWorkCenters()[0].workCenter;
const sResource = PodContext.getFilterResources()?.[0]?.resource || "";

const oRequest = {
    workcenter: sWorkCenter,
    resource: sResource
};
```

#### When to Use Each Getter

| Scenario | Use |
|----------|-----|
| Starting an SFC | `getLastSelectedOperationActivity()` + `getLastSelectedWorkListItem()` + `getFilterResources()` |
| Reporting quantity | `getLastSelectedOperationActivity()` + `getLastSelectedWorkListItem()` |
| Loading SFC details | `getLastSelectedWorkListItem()` |
| Enabling/disabling button based on selection | Subscribe to `ModelPath.SelectedWorkListItems` or `ModelPath.SelectedOperationActivities` |
| Getting current resource for filtering | `getFilterResources()?.[0]?.resource` |
| Multi-select operations | `getSelectedOperationActivities()` + `getSelectedWorkListItems()` |

---

---

## Utility Classes Reference

### GrowingJSONModel

**Import:** `sap/dm/dme/pod2/model/GrowingJSONModel`

**Use Case:** Tables with pagination/lazy loading

**Usage:**
```javascript
import GrowingJSONModel from "sap/dm/dme/pod2/model/GrowingJSONModel";

#oModel = new GrowingJSONModel();
#iPage = 0;
#iPageSize = 20;

// Binding parameters
this.#oTable.bindItems({
    path: "/items",
    template: oTemplate,
    parameters: {
        countPath: "/totalCount",      // Path to total count
        listControl: this.#oTable,     // Table reference
        onGrowing: () => this._fetch() // Next page callback
    }
});

// Fetch method
async _fetch() {
    const iPage = this.#iPage++;  // Increment page!
    const oResponse = await API.get({ page: iPage, size: this.#iPageSize });
    return [oResponse.items, oResponse.totalCount];
}
```

**See also:** [widget-patterns.md - GrowingJSONModel](widget-patterns.md#tablewidget-with-growingjsonmodel-pagination-pattern)

---

### MessageHistory - User Notifications

**Import:** `sap/dm/dme/pod2/context/MessageHistory`

```javascript
// Persistent messages (in message popover)
MessageHistory.push({
    message: this.getI18nText("error.criticalFailure"),
    type: MessageHistory.Error
});

// Transient toast messages (auto-dismiss)
MessageHistory.toast({
    message: this.getI18nText("info.dataSaved"),
    type: MessageHistory.Success
});

// Message types
MessageHistory.Error        // Red - failures, errors
MessageHistory.Warning      // Orange - warnings, cautions
MessageHistory.Success      // Green - success confirmations
MessageHistory.Information  // Blue - informational
```

**Use `push()` for:** Errors, validation failures, important info that users should review.
**Use `toast()` for:** Success confirmations, minor transient info.

---

### I18nResourceModel

**Import:** `sap/dm/dme/pod2/model/I18nResourceModel`

**Usage (Framework-driven pattern):**
```javascript
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class MyWidget extends Widget {
    // Static private i18n model
    static #oI18nModel = new I18nResourceModel({
        bundleName: "custom.company.project.i18n.i18n"  // Dots, not slashes!
    });
    
    // Static getter (framework calls this)
    static getI18nModel() {
        return this.#oI18nModel;
    }
    
    // Use inherited method
    _someMethod() {
        const sText = this.getI18nText("myWidget.greeting");
        const sTitle = this.getI18nText("myWidget.title", arg1, arg2);
    }
}
```

#### I18nResourceModel.getText()

```javascript
// Direct access to translation string (synchronous)
const sText = oI18nModel.getText("key");
const sWithArgs = oI18nModel.getText("key", [arg1, arg2]);
```

#### I18nResourceModel.enhanceForProcessIndustry()

For process industry PODs, SAP DM uses different terminology (e.g. "Phase" instead of "Operation"). This method enhances the model with overrides from a process-industry-specific bundle.

```javascript
// Call in onInit() AFTER the model is created, ONLY if process industry relevant
async onInit() {
    await super.onInit();
    if (PodContext.isProcessIndustry()) {
        await MyWidget.#oI18nModel.enhanceForProcessIndustry(
            "custom.company.project.i18n.i18n"  // Usually same bundle name
        );
    }
}
```

> Note: This only applies if the plant's `industryType` is `"PROCESS"`. For discrete-only plugins this is not needed.

---

## ApiClient.internal - Internal SAP DM APIs

### Overview

`ApiClient.internal` provides access to internal SAP Digital Manufacturing APIs not documented in public API reference. These APIs power SAP's own widgets.

**Warning:** Internal APIs may change between versions without notice. Use with caution and test thoroughly after upgrades.

### Common Internal API Namespaces

```javascript
import ApiClient from "sap/dm/dme/pod2/context/ApiClient";

// SFC internal APIs
await ApiClient.internal.sfc.getReportedQuantitySummary(sOrder, sSfc, sOperation);
await ApiClient.internal.sfc.getReportedQuantities(oRequest);
await ApiClient.internal.sfc.getSfcDetails({ plant, sfc });

// Plant/Resource/Operation internal APIs
await ApiClient.internal.plant.getReasonCodeObject(sPlant, sReasonCode);
await ApiClient.internal.resource.getResourceDetails(sResource);
await ApiClient.internal.operation.getOperationData(sOperation);
```

### When to Use Internal APIs

**Use when:** Public APIs don't provide needed functionality, or replicating behavior from SAP standard widgets.

**Avoid when:** Public API exists for the same purpose, or building long-term production-critical features.

---

## PodContext Direct Getters

### getLastSelectedWorkListItem()

Get currently selected work list item without subscription.

```javascript
const oWorkListItem = PodContext.getLastSelectedWorkListItem();
// Returns: { order, sfc, material, quantity, operation, resource, ... }

const oOperationActivity = PodContext.getLastSelectedOperationActivity();
// Returns: { operationActivity, operation, stepId, ... }

// Use in subscriptions for current values:
this.subscribe(ModelPath.ReportedQuantityItems, async () => {
    const oWorkListItem = PodContext.getLastSelectedWorkListItem();
    const oOperationActivity = PodContext.getLastSelectedOperationActivity();
    
    if (!oWorkListItem || !oOperationActivity) return;
    
    const oData = await ApiClient.internal.sfc.getReportedQuantitySummary(
        oWorkListItem.order,
        oWorkListItem.sfc,
        oOperationActivity.operationActivity
    );
    this.#oModel.setData([oData]);
}, this);
```

---

**END OF API REFERENCE**
