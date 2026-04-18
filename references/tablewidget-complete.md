# TableWidget Complete Reference

Comprehensive guide to POD 2.0 TableWidget with all production patterns from real SAP code.

## Overview

TableWidget is the most complex POD 2.0 base class. This guide covers all patterns from basic to enterprise-grade production code.

## Property Control Patterns

### EXCLUDE_PROPERTIES (Static)

**Purpose**: Blacklist properties from POD Designer property panel

**When to Use**: You want to hide specific properties but expose most others

```javascript
class MyTableWidget extends TableWidget {
    static EXCLUDE_PROPERTIES = [
        ...TableWidget.EXCLUDE_PROPERTIES,  // Inherit parent exclusions
        "growing",
        "growingThreshold",
        "growingTriggerText",
        "rememberSelections"
    ];
}
```

**What It Does**:
- Properties in this array won't appear in POD Designer property panel
- User can't configure these properties
- Properties can still be set programmatically

### INCLUDE_PROPERTIES (Static)

**Purpose**: Whitelist properties for POD Designer property panel

**When to Use**: You want tight control and only expose specific properties

```javascript
class PhaseTableWidget extends TableWidget {
    static INCLUDE_PROPERTIES = [
        "alternateRowColors",
        "backgroundDesign",
        "inset",
        "showOverlay",
        "showSeparators",
        "sticky",
        "visible",
        "width"
    ];
}
```

**What It Does**:
- ONLY these properties appear in POD Designer
- More restrictive than EXCLUDE_PROPERTIES
- Prevents accidental exposure of internal properties

**⚠️ IMPORTANT**: Use either EXCLUDE_PROPERTIES OR INCLUDE_PROPERTIES, not both!

### IGNORE_TABLE_PROPERTIES (Instance)

**Purpose**: Exclude properties from table constructor

**When to Use**: Property is for widget logic but shouldn't be passed to sap.m.Table

```javascript
class OrderListTableWidget extends TableWidget {
    // Instance property (NOT static!)
    IGNORE_TABLE_PROPERTIES = [
        "pageSize",
        "printButtonVisible",
        "printConfigOrder",
        "printConfigLabel"
    ];
    
    // Still exclude from designer if needed
    static EXCLUDE_PROPERTIES = [
        ...TableWidget.EXCLUDE_PROPERTIES,
        "growing",
        "growingThreshold"
    ];
}
```

**What It Does**:
- Properties in this array won't be passed to `new sap.m.Table(config)`
- Prevents "unknown property" errors from sap.m.Table
- Widget can still use these properties internally

### Decision Matrix

| Pattern | Type | Purpose | Use When |
|---------|------|---------|----------|
| EXCLUDE_PROPERTIES | static | Hide from designer | Blacklist approach |
| INCLUDE_PROPERTIES | static | Show in designer | Whitelist approach |
| IGNORE_TABLE_PROPERTIES | instance | Skip in constructor | TableWidget-specific config |

**Example - All Three**:
```javascript
class ComplexTableWidget extends TableWidget {
    // Whitelist: only these appear in designer
    static INCLUDE_PROPERTIES = [
        "alternateRowColors",
        "visible",
        "width"
    ];
    
    // Don't pass these to sap.m.Table constructor
    IGNORE_TABLE_PROPERTIES = [
        "pageSize",
        "customToolbarConfig"
    ];
}
```

## Sorting and Grouping

### Custom Sorter with Comparator

**Use Case**: Complex sort logic beyond simple field comparison

```javascript
_getItemBindingInfo(oTable, oTemplate) {
    return {
        ...super._getItemBindingInfo(oTable, oTemplate),
        sorter: new Sorter({
            path: "operationActivity",
            comparator: (sA, sB) => {
                // Sort by last 4 characters (numeric suffix)
                return Number(sA.slice(-4)) - Number(sB.slice(-4));
            },
            descending: false,
            group: (oContext) => oContext.getObject().operationActivityGroup
        })
    };
}
```

**Required Import**:
```javascript
import Sorter from "sap/ui/model/Sorter";
```

### Grouping with Custom Headers

**Use Case**: Hierarchical data display (by work center, operation, etc.)

```javascript
_getGroupHeader(oGroup) {
    return new GroupHeaderListItem({
        title: oGroup.key,
        upperCase: false
    });
}

_getItemBindingInfo(oTable, oTemplate) {
    return {
        ...super._getItemBindingInfo(oTable, oTemplate),
        sorter: new Sorter({
            path: "operationActivity",
            group: (oContext) => oContext.getObject().operationActivityGroup
        }),
        groupHeaderFactory: this._getGroupHeader.bind(this)
    };
}
```

**Required Imports**:
```javascript
import Sorter from "sap/ui/model/Sorter";
import GroupHeaderListItem from "sap/m/GroupHeaderListItem";
```

**Key Points**:
- `group` function returns grouping key
- `groupHeaderFactory` creates header items
- Must bind the factory function

### Hidden Sortable Columns

**Use Case**: Sort by fields not shown in table

```javascript
_getSortableColumns() {
    return [
        ...super._getSortableColumns(),
        {
            field: "plannedStartDate",
            text: "{i18n>OrderListTableWidget.field.plannedStartDate}"
        },
        {
            field: "completedQty",
            text: "{i18n>OrderListTableWidget.field.completedQty}"
        },
        {
            field: "sfcCompletedQty",
            text: "{i18n>OrderListTableWidget.field.sfcCompletedQty}"
        }
    ];
}
```

**Why Useful**:
- Show date range in cell, sort by start date
- Show composite cell, sort by hidden numeric value
- Enhance table flexibility without cluttering columns

## Advanced Filtering

### Dynamic Filtering in Binding

**Use Case**: Property-driven conditional filters

```javascript
_getItemBindingInfo(oTable, oTemplate) {
    return {
        ...super._getItemBindingInfo(oTable, oTemplate),
        filters: this.getPropertyValue(PhaseTableWidget.PropertyId.FilterToAssignedWorkCenters) === true ?
            new Filter({
                path: "userAuthorizedForWorkCenter",
                operator: FilterOperator.EQ,
                value1: true
            }) :
            []
    };
}
```

**Required Imports**:
```javascript
import Filter from "sap/ui/model/Filter";
import FilterOperator from "sap/ui/model/FilterOperator";
```

**Key Points**:
- Declarative filtering at binding level
- More efficient than programmatic filtering
- User configurable via property

**Multiple Filters**:
```javascript
filters: [
    new Filter({
        path: "status",
        operator: FilterOperator.NE,
        value1: "DELETED"
    }),
    new Filter({
        path: "authorized",
        operator: FilterOperator.EQ,
        value1: true
    })
]
```

## Performance Patterns

### Growing Tables with Pagination

**Use Case**: Large datasets (100+ rows)

```javascript
_createTable(oConfig, mSettings = {}) {
    return super._createTable(oConfig, {
        growing: true,
        growingThreshold: this.getPropertyValue(OrderListTableWidget.PropertyId.PageSize),
        updateStarted: (oEvent) => {
            if (oEvent.getParameter("reason") === "Growing") {
                // Fire and forget, do not return a Promise
                WorkListDelegate.fetchNextPage();
            }
        },
        ...mSettings
    });
}

async onInit() {
    const iPageSize = this.getPropertyValue(OrderListTableWidget.PropertyId.PageSize);
    PodContext.setWorkListPageSize(iPageSize);
}
```

**Required Import**:
```javascript
import WorkListDelegate from "sap/dm/dme/pod2/delegate/WorkListDelegate";
```

**Key Points**:
- `growing: true` enables "load more"
- `growingThreshold` sets page size
- Coordinate with data delegate
- Set page size in PodContext

**Property Definition**:
```javascript
static PropertyId = Object.freeze({
    ...TableWidget.PropertyId,
    PageSize: "pageSize"
});

static getDefaultConfig() {
    return {
        ...TableWidget.getDefaultConfig(),
        properties: {
            ...TableWidget.getDefaultConfig().properties,
            [OrderListTableWidget.PropertyId.PageSize]: 50
        }
    };
}
```

## Selection Patterns

### Bidirectional Selection Synchronization

**Use Case**: Table selections must sync with PodContext

**Complete Pattern**:
```javascript
async onInit() {
    // Subscribe to changes in selected items
    PodContext.subscribe(ModelPath.SelectedWorkListItems, () => {
        this._syncSelectionsWithPodContext();
    }, this);
    
    // Initial sync
    this._syncSelectionsWithPodContext();
}

_syncSelectionsWithPodContext() {
    const oTable = this.getTable();
    const aSelectedWorkListItems = PodContext.getSelectedWorkListItems();
    const aSelectedIdentifiers = Array.isArray(aSelectedWorkListItems) ?
        aSelectedWorkListItems.map((oWorkListItem) => oWorkListItem.getIdentifier()) :
        [];

    oTable.getItems().forEach((oListItem) => {
        const oWorkListItem = oListItem.getBindingContext().getObject();
        oListItem.setSelected(aSelectedIdentifiers.includes(oWorkListItem.getIdentifier()));
    });
}

_onSelectionChange(oEvent) {
    const aCurrentPodContextSelection = PodContext.getSelectedWorkListItems() || [];
    const aCurrentTableSelection = this.getTable().getSelectedContexts().map((oContext) => {
        return oContext.getObject();
    });

    const aNewSelection = [];
    const oSelectedIdentifiers = new Set(aCurrentTableSelection.map((oItem) => oItem.getIdentifier()));
    
    // Keep previously selected items that are still selected
    for (const oWorkListItem of aCurrentPodContextSelection) {
        if (oSelectedIdentifiers.has(oWorkListItem.getIdentifier())) {
            aNewSelection.push(oWorkListItem);
        }
    }

    // Add newly selected items
    if (oEvent.getParameter("selected")) {
        const aModifiedListItems = oEvent.getParameter("listItems").map((oListItem) =>
            oListItem.getBindingContext().getObject());
        aNewSelection.push(...aModifiedListItems);
    }

    PodContext.setSelectedWorkListItems(aNewSelection);
}
```

**Key Points**:
- Single source of truth (PodContext)
- Bidirectional sync
- Handles multi-select correctly
- Uses Set for efficient lookup

### Item Press with Selection Preservation

**Use Case**: Click item without losing multi-selection

```javascript
_onItemPress(oEvent) {
    const oListItem = oEvent.getParameter("listItem");
    const oPressedWorkListItem = oListItem.getBindingContext().getObject();

    // Rather than selecting only the clicked item, keep other selections
    // and move clicked item to end of array (most recent selected)
    const aSelectedWorkListItems = PodContext.getSelectedWorkListItems() ?? [];
    const sSelectedIdentifier = oPressedWorkListItem.getIdentifier();
    const aNewSelections = [];

    for (const oSelectedWorkListItem of aSelectedWorkListItems) {
        if (oSelectedWorkListItem.getIdentifier() !== sSelectedIdentifier) {
            aNewSelections.push(oSelectedWorkListItem);
        }
    }
    aNewSelections.push(oPressedWorkListItem);
    PodContext.setSelectedWorkListItems(aNewSelections);
}
```

**Why This Pattern**:
- Better UX for multi-select scenarios
- Preserves existing selections
- Clicked item becomes "last selected"

## Toolbar Customization

### Custom Toolbar Title with Binding

```javascript
_createToolbarTitle() {
    return new Title({
        text: {
            parts: [
                ModelPath.WorkListCount,
                ModelPath.WorkListType
            ],
            formatter: (iCount, eWorkListType) => {
                const sI18nKey = eWorkListType === WorkListType.ProductionOrder ?
                    "OrderListTableWidget.toolBarTitle.production" :
                    "OrderListTableWidget.toolBarTitle.process";
                return this.getI18nText(sI18nKey, iCount || 0);
            }
        }
    });
}
```

**Required Import**:
```javascript
import Title from "sap/m/Title";
```

### MenuButton with Dynamic Enable

```javascript
#oPrintMenuButton;  // Private field

_createTable(oConfig, mSettings = {}) {
    this.#oPrintMenuButton = new MenuButton({
        icon: "sap-icon://print",
        tooltip: "{i18n>OrderListTableWidget.menuButton.tooltip}",
        visible: Boolean(this.getPropertyValue(OrderListTableWidget.PropertyId.PrintButtonVisible)),
        enabled: {
            path: ModelPath.SelectedWorkListItems,
            formatter: (aSelectedWorkListItems) => {
                return Array.isArray(aSelectedWorkListItems) && aSelectedWorkListItems.length !== 0;
            }
        },
        menu: new Menu({
            items: [
                new MenuItem({
                    text: "{i18n>OrderListTableWidget.menuButton.item.printOrder}",
                    press: () => this._onPrintMenuItemPress(OrderListTablePrintDialog.PrintMode.Order)
                }),
                new MenuItem({
                    text: "{i18n>OrderListTableWidget.menuButton.item.printLabel}",
                    press: () => this._onPrintMenuItemPress(OrderListTablePrintDialog.PrintMode.Label)
                })
            ]
        })
    });
    
    return super._createTable(oConfig, {
        toolbar: this._createCustomToolbar(),
        ...mSettings
    });
}

_createCustomToolbar() {
    return new OverflowToolbar({
        content: [
            this._createToolbarTitle(),
            new ToolbarSpacer(),
            this.#oPrintMenuButton
        ]
    });
}
```

**Required Imports**:
```javascript
import MenuButton from "sap/m/MenuButton";
import Menu from "sap/m/Menu";
import MenuItem from "sap/m/MenuItem";
import OverflowToolbar from "sap/m/OverflowToolbar";
import ToolbarSpacer from "sap/m/ToolbarSpacer";
```

## Property Management Patterns

### getPropertyValue with Coalescing

**Use Case**: Provide complex default values dynamically

```javascript
getPropertyValue(sName) {
    const vValue = super.getPropertyValue(sName);
    
    if (sName === OrderListTableWidget.PropertyId.PrintConfigOrder && !vValue) {
        return {
            documentNumber: "",
            documentVersion: ""
        };
    } else if (sName === OrderListTableWidget.PropertyId.PrintConfigLabel && !vValue) {
        return {
            documentNumber: "",
            documentVersion: "",
            customFieldsMetaData: [
                { fieldName: "", fieldDescription: "" },
                { fieldName: "", fieldDescription: "" },
                { fieldName: "", fieldDescription: "" },
                { fieldName: "", fieldDescription: "" }
            ]
        };
    }
    
    return vValue;
}
```

**Why Useful**:
- Complex object properties need structure
- Prevents null reference errors
- Cleaner than checking everywhere

### setPropertyValue with UI Sync

**Use Case**: Live preview in POD Designer

```javascript
setPropertyValue(sName, vValue) {
    if (sName === OrderListTableWidget.PropertyId.PrintButtonVisible) {
        if (this.#oPrintMenuButton) {
            this.#oPrintMenuButton.setVisible(vValue);
        }
    }
    super.setPropertyValue(sName, vValue);
}
```

**Key Points**:
- Update UI when properties change in designer
- User sees changes immediately
- Check if control exists (may not be created yet)

## Complete Production Example

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/core/TableWidget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/dm/dme/pod2/delegate/WorkListDelegate",
    "sap/dm/dme/pod2/formatter/StatusFormatter",
    "sap/dm/dme/pod2/DateTimeUtils",
    "sap/m/HBox",
    "sap/m/FlexRendertype",
    "sap/m/FlexAlignItems",
    "sap/dm/dme/pod2/component/StatusIcon",
    "sap/ui/model/Sorter",
    "sap/ui/model/Filter",
    "sap/ui/model/FilterOperator",
    "sap/m/MenuButton",
    "sap/m/Menu",
    "sap/m/MenuItem",
    "sap/m/OverflowToolbar",
    "sap/m/ToolbarSpacer",
    "sap/m/Title"
], (
    TableWidget, PodContext, ModelPath, WorkListDelegate, StatusFormatter, DateTimeUtils,
    HBox, FlexRendertype, FlexAlignItems, StatusIcon, Sorter, Filter, FilterOperator,
    MenuButton, Menu, MenuItem, OverflowToolbar, ToolbarSpacer, Title
) => {
    "use strict";

    class ProductionTableWidget extends TableWidget {
        
        static PropertyId = Object.freeze({
            ...TableWidget.PropertyId,
            PageSize: "pageSize",
            FilterToAuthorized: "filterToAuthorized",
            PrintButtonVisible: "printButtonVisible"
        });
        
        static Field = Object.freeze({
            SfcName: "sfcName",
            SfcStatus: "sfcStatus",
            DateRange: "dateRange",
            Quantity: "quantity"
        });
        
        // Exclude from property panel
        static EXCLUDE_PROPERTIES = [
            ...TableWidget.EXCLUDE_PROPERTIES,
            "growing",
            "growingThreshold"
        ];
        
        // Don't pass to table constructor
        IGNORE_TABLE_PROPERTIES = [
            "pageSize",
            "printButtonVisible"
        ];
        
        #oPrintMenuButton;
        
        static getDefaultConfig() {
            return {
                ...TableWidget.getDefaultConfig(),
                properties: {
                    ...TableWidget.getDefaultConfig().properties,
                    [ProductionTableWidget.PropertyId.PageSize]: 50,
                    [ProductionTableWidget.PropertyId.FilterToAuthorized]: true,
                    [ProductionTableWidget.PropertyId.PrintButtonVisible]: true
                }
            };
        }
        
        getFields() {
            return [
                {
                    field: ProductionTableWidget.Field.SfcName,
                    text: "{i18n>sfcName}",
                    width: "10rem",
                    sortable: true
                },
                {
                    field: ProductionTableWidget.Field.SfcStatus,
                    text: "{i18n>status}",
                    width: "10rem",
                    sortable: true
                },
                {
                    field: ProductionTableWidget.Field.DateRange,
                    text: "{i18n>dateRange}",
                    width: "15rem",
                    sortable: false
                },
                {
                    field: ProductionTableWidget.Field.Quantity,
                    text: "{i18n>quantity}",
                    width: "8rem",
                    sortable: true
                }
            ];
        }
        
        getDefaultFields() {
            return [
                ProductionTableWidget.Field.SfcName,
                ProductionTableWidget.Field.SfcStatus,
                ProductionTableWidget.Field.Quantity
            ];
        }
        
        _createCell(oColumnConfig) {
            switch (oColumnConfig.field) {
                case ProductionTableWidget.Field.SfcName:
                    return this._createIdentifierCell(oColumnConfig, {
                        press: (oEvent) => this._onSfcPress(oEvent)
                    });
                    
                case ProductionTableWidget.Field.SfcStatus:
                    return new HBox({
                        renderType: FlexRendertype.Bare,
                        alignItems: FlexAlignItems.Center,
                        gap: "0.25rem",
                        items: [
                            new StatusIcon({
                                size: "1.25rem",
                                width: "1.25rem",
                                bindPath: "sfcStatusCode",
                                decorative: true
                            }),
                            this._createTextCell(oColumnConfig, {
                                path: oColumnConfig.field,
                                formatter: (sValue) => StatusFormatter.getStatusText(sValue)
                            })
                        ]
                    });
                    
                case ProductionTableWidget.Field.DateRange:
                    return new Text({
                        text: {
                            parts: [
                                { path: "startDate" },
                                { path: "endDate" }
                            ],
                            formatter: (oStart, oEnd) => {
                                if (!oStart || !oEnd) return "";
                                return `${DateTimeUtils.localeDate(oStart)} – ${DateTimeUtils.localeDate(oEnd)}`;
                            }
                        }
                    });
                    
                default:
                    return this._createTextCell(oColumnConfig, oColumnConfig.field);
            }
        }
        
        _createTable(oConfig, mSettings = {}) {
            this.#oPrintMenuButton = new MenuButton({
                icon: "sap-icon://print",
                tooltip: "{i18n>print}",
                visible: Boolean(this.getPropertyValue(ProductionTableWidget.PropertyId.PrintButtonVisible)),
                enabled: {
                    path: ModelPath.SelectedWorkListItems,
                    formatter: (aSelected) => Array.isArray(aSelected) && aSelected.length !== 0
                },
                menu: new Menu({
                    items: [
                        new MenuItem({
                            text: "{i18n>printOrder}",
                            press: () => this._onPrintOrder()
                        })
                    ]
                })
            });
            
            return super._createTable(oConfig, {
                growing: true,
                growingThreshold: this.getPropertyValue(ProductionTableWidget.PropertyId.PageSize),
                updateStarted: (oEvent) => {
                    if (oEvent.getParameter("reason") === "Growing") {
                        WorkListDelegate.fetchNextPage();
                    }
                },
                toolbar: new OverflowToolbar({
                    content: [
                        new Title({ text: "{i18n>title}" }),
                        new ToolbarSpacer(),
                        this.#oPrintMenuButton
                    ]
                }),
                ...mSettings
            });
        }
        
        _getItemBindingInfo(oTable, oTemplate) {
            return {
                ...super._getItemBindingInfo(oTable, oTemplate),
                filters: this.getPropertyValue(ProductionTableWidget.PropertyId.FilterToAuthorized) === true ?
                    new Filter({
                        path: "userAuthorized",
                        operator: FilterOperator.EQ,
                        value1: true
                    }) :
                    [],
                sorter: new Sorter({
                    path: "sfcName",
                    descending: false
                })
            };
        }
        
        async onInit() {
            await super.onInit();
            
            const iPageSize = this.getPropertyValue(ProductionTableWidget.PropertyId.PageSize);
            PodContext.setWorkListPageSize(iPageSize);
            
            PodContext.subscribe(ModelPath.SelectedWorkListItems, () => {
                this._syncSelectionsWithPodContext();
            }, this);
            
            this._syncSelectionsWithPodContext();
        }
        
        _syncSelectionsWithPodContext() {
            const oTable = this.getTable();
            const aSelectedItems = PodContext.getSelectedWorkListItems();
            const aSelectedIds = Array.isArray(aSelectedItems) ?
                aSelectedItems.map((oItem) => oItem.getIdentifier()) :
                [];

            oTable.getItems().forEach((oListItem) => {
                const oItem = oListItem.getBindingContext().getObject();
                oListItem.setSelected(aSelectedIds.includes(oItem.getIdentifier()));
            });
        }
        
        setPropertyValue(sName, vValue) {
            if (sName === ProductionTableWidget.PropertyId.PrintButtonVisible) {
                if (this.#oPrintMenuButton) {
                    this.#oPrintMenuButton.setVisible(vValue);
                }
            }
            super.setPropertyValue(sName, vValue);
        }
        
        onExit() {
            super.onExit();
            
            PodContext.unsubscribe(
                ModelPath.SelectedWorkListItems,
                this._syncSelectionsWithPodContext,
                this
            );
        }
    }
    
    return ProductionTableWidget;
});
```

## Best Practices Checklist

### ✅ DO:
- Spread parent PropertyId for TableWidget/LayoutWidget
- Use IGNORE_TABLE_PROPERTIES for widget-specific config
- Implement bidirectional selection sync
- Use growing tables for >50 rows
- Add custom sorters for complex logic
- Validate data in multi-part formatters
- Unsubscribe in onExit()
- Use private fields (#) for internal state
- Pre-select rows before actions

### ❌ DON'T:
- Mix EXCLUDE_PROPERTIES and INCLUDE_PROPERTIES
- Forget IGNORE_TABLE_PROPERTIES for custom properties
- Skip selection synchronization
- Load all rows at once for large datasets
- Put business logic in templates
- Forget to bind toolbar buttons to PodContext
- Create memory leaks (missing unsubscribe)

---

**Related References**:
- [tablecell-patterns.md](tablecell-patterns.md) - All cell types
- [binding-patterns.md](binding-patterns.md) - Data binding techniques
- [widget-patterns.md](widget-patterns.md) - Complete templates
