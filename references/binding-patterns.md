# Binding Patterns

Comprehensive guide to data binding in POD 2.0 widgets with production patterns.

## Overview

POD 2.0 supports multiple binding approaches. Understanding when to use each is critical for production code.

## Basic Binding

### Simple Property Binding

**Syntax**: `{path}`

```javascript
new Text({
    text: "{orderNumber}"
})
```

### Absolute vs Relative Paths

```javascript
// Relative (from current binding context)
text: "{orderNumber}"

// Absolute (from root of model)
text: "{/orders/0/orderNumber}"

// Named model
text: "{myModel>/data/value}"
```

## Multi-Part Binding

### With Formatter Function

**Use Case**: Combine multiple model paths in single binding

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

**Key Points**:
- Formatter receives all parts as parameters
- Reactive: updates when ANY part changes
- Clean separation of display logic
- ALWAYS validate parameters (null checks)

### Date Range Pattern

**Production Pattern**:
```javascript
_createPlannedDateRangeCell() {
    return new Text({
        text: {
            parts: [
                { path: "orderPlannedStartDate" },
                { path: "orderPlannedCompleteDate" }
            ],
            formatter: (oStartDate, oEndDate) => {
                // CRITICAL: Defensive null checking
                if (!oStartDate || !oEndDate) {
                    return "";
                }
                return `${DateTimeUtils.localeDate(oStartDate)} – ${DateTimeUtils.localeDate(oEndDate)}`;
            }
        }
    });
}
```

**Required Import**:
```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";
```

### Quantity with UOM Pattern

```javascript
_createQuantityCell() {
    return new Text({
        text: {
            parts: [
                { path: "quantity" },
                { path: "uom" }
            ],
            formatter: (fQty, sUom) => {
                if (fQty == null) return "";
                return `${fQty} ${sUom || ""}`.trim();
            }
        }
    });
}
```

### Actual/Planned Pattern

```javascript
_createProgressCell() {
    return new Text({
        text: {
            parts: [
                { path: "actualQty" },
                { path: "plannedQty" },
                { path: "uom" }
            ],
            formatter: (fActual, fPlanned, sUom) => {
                if (fActual == null || fPlanned == null) return "";
                return `${fActual} / ${fPlanned} ${sUom || ""}`.trim();
            }
        }
    });
}
```

## Composite Binding

### String Template Binding

```javascript
new Text({
    text: {
        parts: [
            { path: "materialNumber" },
            { path: "materialDescription" }
        ],
        formatter: (sMaterial, sDesc) => {
            return sMaterial ? `${sMaterial} - ${sDesc || ""}` : "";
        }
    }
})
```

### Conditional Text

```javascript
new Text({
    text: {
        parts: [
            { path: "status" },
            { path: "errorMessage" }
        ],
        formatter: (sStatus, sError) => {
            if (sStatus === "ERROR") {
                return sError || this.getI18nText("unknownError");
            }
            return StatusFormatter.getStatusText(sStatus);
        }
    }
})
```

## Expression Binding

### Conditional Visibility

```javascript
new Button({
    visible: {
        parts: [
            { path: "status" },
            { path: "authorized" }
        ],
        formatter: (sStatus, bAuthorized) => {
            return sStatus === "ACTIVE" && bAuthorized === true;
        }
    }
})
```

### State-Based Rendering

**Production Pattern**:
```javascript
_createActionCell() {
    return new HBox({
        items: [
            new CustomButton({
                text: this.getI18nText("PhaseTableWidget.phase.start.text"),
                type: ButtonType.Success,
                visible: {
                    path: "",  // Current context
                    formatter: (oPhase) => {
                        return !oPhase.isActive() && !oPhase.isComplete();
                    }
                },
                enabled: "{userAuthorizedForWorkCenter}",
                press: this._startPhase.bind(this)
            }),
            new CustomButton({
                text: this.getI18nText("PhaseTableWidget.phase.complete.text"),
                type: ButtonType.Emphasized,
                visible: {
                    path: "",
                    formatter: (oPhase) => {
                        return oPhase.isActive() && !oPhase.isComplete();
                    }
                },
                enabled: "{userAuthorizedForWorkCenter}",
                press: this._completePhase.bind(this)
            }),
            new Text({
                text: this.getI18nText("PhaseTableWidget.phase.completed.text"),
                visible: {
                    path: "",
                    formatter: (oPhase) => oPhase.isComplete()
                }
            })
        ]
    });
}
```

**Key Points**:
- Empty path (`""`) binds to entire context object
- Formatter receives full object
- Different controls for different states
- Clean business logic separation

## Dynamic Enabling/Disabling

### Based on Selection

```javascript
new MenuButton({
    enabled: {
        path: ModelPath.SelectedWorkListItems,
        formatter: (aSelectedWorkListItems) => {
            return Array.isArray(aSelectedWorkListItems) && 
                   aSelectedWorkListItems.length !== 0;
        }
    }
})
```

### Based on Authorization

```javascript
new CustomButton({
    enabled: {
        parts: [
            { path: "userAuthorizedForWorkCenter" },
            { path: "status" }
        ],
        formatter: (bAuthorized, sStatus) => {
            return bAuthorized === true && sStatus === "ACTIVE";
        }
    }
})
```

## Two-Way Binding

### Form Inputs

```javascript
new Input({
    value: {
        path: "printingModel>/customFields/batchNumber",
        mode: BindingMode.TwoWay
    },
    valueLiveUpdate: true,
    width: "100%"
})
```

**Required Import**:
```javascript
import BindingMode from "sap/ui/model/BindingMode";
```

### Named Model Binding

```javascript
// Create model in _createView()
const oModel = new JSONModel({
    customFields: {
        field1: "",
        field2: ""
    }
});
this.setModel(oModel, "printingModel");

// Bind with two-way
new Input({
    value: {
        path: "printingModel>/customFields/field1",
        mode: BindingMode.TwoWay
    }
})
```

## Binding Context

### Accessing Row Data

```javascript
_onButtonPress(oEvent) {
    const oButton = oEvent.getSource();
    const oContext = oButton.getBindingContext();
    const oRowData = oContext.getObject();
    
    console.log(oRowData.orderNumber);
}
```

### Setting Context

```javascript
const oContext = oTable.getItems()[0].getBindingContext();
oDialog.setBindingContext(oContext);
```

## PodContext Binding

### Direct Binding to PodContext

```javascript
new Text({
    text: {
        path: ModelPath.Plant,
        formatter: (sPlant) => {
            return this.getI18nText("currentPlant", sPlant || "");
        }
    }
})
```

### Multiple PodContext Paths

```javascript
new Text({
    text: {
        parts: [
            ModelPath.Plant,
            ModelPath.FilterResources
        ],
        formatter: (sPlant, aResources) => {
            const iCount = Array.isArray(aResources) ? aResources.length : 0;
            return this.getI18nText("plantResourceCount", sPlant, iCount);
        }
    }
})
```

## Formatter Utility Classes

### StatusFormatter

**Production Pattern**:
```javascript
import StatusFormatter from "sap/dm/dme/pod2/formatter/StatusFormatter";

_createStatusCell(oColumnConfig) {
    return this._createTextCell(oColumnConfig, {
        path: oColumnConfig.field,
        formatter: (sValue) => (sValue ? StatusFormatter.getStatusText(sValue) : "")
    });
}
```

### DateTimeUtils

**Production Patterns**:
```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

// Locale date
formatter: (oDate) => oDate ? DateTimeUtils.localeDate(oDate) : ""

// Locale time
formatter: (oTime) => oTime ? DateTimeUtils.localeTime(oTime) : ""

// Locale date and time
formatter: (oDateTime) => oDateTime ? DateTimeUtils.localeDateTime(oDateTime) : ""
```

## i18n Binding

### ⚠️ CRITICAL Rules

**DO NOT use binding syntax in `_createView()`**:
```javascript
// ❌ WRONG - model not available yet
_createView() {
    return new Text({
        text: "{i18n>title}"  // ❌ Will fail!
    });
}
```

**ALWAYS use method calls**:
```javascript
// ✅ CORRECT
_createView() {
    return new Text({
        text: this.getI18nText("title")  // ✅ Works!
    });
}
```

### Where Bindings ARE Allowed

**In control properties after initialization**:
```javascript
_createTable(oConfig, mSettings = {}) {
    return super._createTable(oConfig, {
        noDataText: "{i18n>noData}",  // ✅ OK here
        tooltip: "{i18n>tableTooltip}",  // ✅ OK here
        ...mSettings
    });
}
```

**In getDefaultConfig()**:
```javascript
static getDefaultConfig() {
    return {
        ...TableWidget.getDefaultConfig(),
        properties: {
            noDataText: "{i18n>noData}"  // ✅ OK here
        }
    };
}
```

### i18n with Placeholders

```javascript
// i18n.properties
# resourceCount=Found {0} resources in {1}

// Widget code
this.getI18nText("resourceCount", iCount, sPlant)
```

## Binding in TableWidget

### Item Binding Info

```javascript
_getItemBindingInfo(oTable, oTemplate) {
    return {
        ...super._getItemBindingInfo(oTable, oTemplate),
        path: "/items",
        sorter: new Sorter({
            path: "orderNumber",
            descending: false
        }),
        filters: [
            new Filter({
                path: "status",
                operator: FilterOperator.NE,
                value1: "DELETED"
            })
        ]
    };
}
```

### Factory Function

```javascript
_getItemBindingInfo(oTable, oTemplate) {
    return {
        path: "/items",
        factory: (sId, oContext) => {
            const oData = oContext.getObject();
            return this._createRowItem(sId, oData);
        }
    };
}

_createRowItem(sId, oData) {
    return new ColumnListItem(sId, {
        cells: this.getVisibleColumns().map((oCol) => {
            return this._createCellForData(oCol, oData);
        })
    });
}
```

## Performance Best Practices

### ✅ DO:
- Use formatters for business logic
- Validate all formatter parameters
- Use multi-part binding instead of nested bindings
- Cache formatter results when expensive
- Use binding modes appropriately
- Unsubscribe from PodContext in onExit()

### ❌ DON'T:
- Use bindings in `_createView()` for i18n
- Forget null checks in formatters
- Create circular binding dependencies
- Put complex logic in templates
- Bind to functions that return Promises
- Create memory leaks with subscriptions

## Complete Example

```javascript
_createView() {
    // Create named model
    const oFormModel = new JSONModel({
        orderNumber: "",
        quantity: 0,
        uom: "EA"
    });
    this.setModel(oFormModel, "formModel");
    
    return new VBox(this.getConfig().id, {
        items: [
            // Static text (method call)
            new Text({
                text: this.getI18nText("title")
            }),
            
            // PodContext binding
            new Text({
                text: {
                    path: ModelPath.Plant,
                    formatter: (sPlant) => {
                        return this.getI18nText("currentPlant", sPlant);
                    }
                }
            }),
            
            // Multi-part PodContext binding
            new Text({
                text: {
                    parts: [
                        ModelPath.FilterResources,
                        ModelPath.Plant
                    ],
                    formatter: (aResources, sPlant) => {
                        const iCount = Array.isArray(aResources) ? aResources.length : 0;
                        return this.getI18nText("resourcesInPlant", iCount, sPlant);
                    }
                }
            }),
            
            // Two-way binding to named model
            new Input({
                value: {
                    path: "formModel>/orderNumber",
                    mode: BindingMode.TwoWay
                },
                valueLiveUpdate: true
            }),
            
            // Composite binding in named model
            new Text({
                text: {
                    parts: [
                        { path: "formModel>/quantity" },
                        { path: "formModel>/uom" }
                    ],
                    formatter: (fQty, sUom) => {
                        if (fQty == null) return "";
                        return `${fQty} ${sUom}`.trim();
                    }
                }
            }),
            
            // Conditional visibility
            new Button({
                text: this.getI18nText("submit"),
                visible: {
                    path: "formModel>/orderNumber",
                    formatter: (sOrder) => {
                        return sOrder && sOrder.length > 0;
                    }
                },
                press: () => this._onSubmit()
            })
        ]
    });
}
```

## Quick Reference

| Binding Type | Syntax | Use When |
|--------------|--------|----------|
| Simple | `{path}` | Single property |
| Multi-part | `{ parts: [...], formatter: ... }` | Multiple properties |
| Expression | `{ path: "", formatter: (obj) => ... }` | Entire context |
| Two-way | `{ path: "...", mode: BindingMode.TwoWay }` | Form inputs |
| Named model | `{modelName>/path}` | Multiple models |
| PodContext | `ModelPath.Plant` | Global state |
| i18n (method) | `this.getI18nText("key")` | In _createView() |
| i18n (binding) | `{i18n>key}` | After initialization |

---

**Related References**:
- [tablecell-patterns.md](tablecell-patterns.md) - Cell-specific binding examples
- [tablewidget-complete.md](tablewidget-complete.md) - TableWidget binding patterns
- [widget-patterns.md](widget-patterns.md) - Complete widget examples
