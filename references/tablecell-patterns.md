# Table Cell Patterns

Complete guide to creating table cells in POD 2.0 TableWidget with production-tested patterns.

## Overview

TableWidget cells are created in the `_createCell()` method using a switch statement on the field name. Each cell type serves specific use cases.

## Basic Cell Types

### 1. Text Cell (Simple)

**Use Case**: Plain text display

```javascript
_createCell(oColumnConfig) {
    switch (oColumnConfig.field) {
        case Field.OrderNumber:
            return this._createTextCell(oColumnConfig, oColumnConfig.field);
    }
}
```

### 2. Text Cell with Formatter

**Use Case**: Transformed or computed text

```javascript
case Field.Status:
    return this._createTextCell(oColumnConfig, {
        path: oColumnConfig.field,
        formatter: (sValue) => (sValue ? StatusFormatter.getStatusText(sValue) : "")
    });
```

**Production Import**:
```javascript
import StatusFormatter from "sap/dm/dme/pod2/formatter/StatusFormatter";
```

### 3. Date Cell

**Use Case**: Formatted dates

```javascript
case Field.PlannedStartDate:
    return new Text({
        text: {
            path: oColumnConfig.field,
            formatter: (oDate) => oDate ? DateTimeUtils.localeDate(oDate) : ""
        }
    });
```

**Production Import**:
```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";
```

## Advanced Cell Types

### 4. Date Range Cell

**Use Case**: Display date ranges (planned start → end)

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
                // CRITICAL: Always validate both dates
                if (!oStartDate || !oEndDate) {
                    return "";
                }
                return `${DateTimeUtils.localeDate(oStartDate)} – ${DateTimeUtils.localeDate(oEndDate)}`;
            }
        }
    });
}
```

**Key Points**:
- Uses multi-part binding
- Defensive null checking (CRITICAL)
- Proper locale formatting
- En-dash (–) separator for ranges

### 5. Identifier Cell (Clickable)

**Use Case**: Interactive links (SFC, Order, Material numbers)

**Production Pattern**:
```javascript
_createIdentifierCell(oColumnConfig) {
    return this._createIdentifierCell(oColumnConfig, {
        press: (oEvent) => this._onIdentifierPress(oEvent)
    });
}

_onIdentifierPress(oEvent) {
    const oLink = oEvent.getSource();
    const sIdentifier = oLink.getText();
    // Navigate or open details
}
```

**Alternative** (if not using parent method):
```javascript
case Field.SfcName:
    return new Link({
        text: `{${oColumnConfig.field}}`,
        emphasized: true,
        press: (oEvent) => this._onSfcPress(oEvent)
    });
```

### 6. Status Cell (Icon + Text)

**Use Case**: Visual status indicators

**Production Pattern**:
```javascript
_createSfcStatusCell(oColumnConfig) {
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
            this._createTextCell(oColumnConfig, oColumnConfig.field)
        ]
    });
}
```

**Required Imports**:
```javascript
import HBox from "sap/m/HBox";
import FlexRendertype from "sap/m/FlexRendertype";
import FlexAlignItems from "sap/m/FlexAlignItems";
import StatusIcon from "sap/dm/dme/pod2/component/StatusIcon";
```

**Key Points**:
- `FlexRendertype.Bare` prevents extra div wrappers
- `decorative: true` for screen readers
- Fixed icon size for consistency
- Proper spacing with `gap`

### 7. Bullet Chart Cell (Progress)

**Use Case**: Visual progress (actual vs. target)

**Production Pattern**:
```javascript
_createQuantityBulletChartCell() {
    return this._createQuantityBulletChartCell({
        scale: "productionCommercialUom",
        targetValue: "orderQuantityPlanned",
        actualValue: "orderQuantityCompleted",
        targetLabel: "i18n>OrderListTableWidget.column.orderQuantityPlanned.targetLabel"
    });
}
```

**What It Creates**:
- Mini bar chart in table cell
- Shows actual quantity vs. planned quantity
- Color-coded (green when complete, yellow in progress)
- Includes UOM label

**Use When**:
- Order completion tracking
- Operation progress
- Material consumption

### 8. Action Cell (Buttons)

**Use Case**: Row-specific actions

**Production Pattern**:
```javascript
_createActionCell() {
    return new HBox({
        items: [
            new CustomButton({
                text: this.getI18nText("PhaseTableWidget.phase.start.text"),
                type: ButtonType.Success,
                visible: {
                    path: "",
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
- State-based visibility (show correct button for state)
- Authorization-aware (enabled binding)
- Different buttons for different states
- Text message when completed

**Pre-Selection Pattern**:
```javascript
async _startPhase(oEvent) {
    const oButton = oEvent.getSource();
    const oPhase = oButton.getBindingContext().getObject();
    
    // CRITICAL: Ensure phase is selected before action
    if (oPhase.getIdentifier() !== PodContext.getLastSelectedOperationActivity()?.getIdentifier()) {
        PodContext.setSelectedOperationActivities([oPhase]);
    }
    
    await this._handleEvent(PhaseTableWidget.EventId.StartPhase, oEvent);
}
```

### 9. Composite Cell (Multiple Controls)

**Use Case**: Complex layouts with multiple elements

**Production Pattern**:
```javascript
_createOrderInfoCell() {
    return new VBox({
        renderType: FlexRendertype.Bare,
        items: [
            new Text({
                text: "{orderNumber}",
                wrapping: false
            }),
            new Text({
                text: {
                    parts: [
                        { path: "materialNumber" },
                        { path: "materialDescription" }
                    ],
                    formatter: (sMaterial, sDesc) => {
                        return sMaterial ? `${sMaterial} - ${sDesc || ""}` : "";
                    }
                },
                wrapping: false
            })
        ]
    });
}
```

**Use When**:
- Displaying hierarchical data in one column
- Combining multiple related fields
- Creating master/detail views in cell

## Specialized Patterns

### 10. Icon Cell

**Use Case**: Simple icons without text

**Production Pattern**:
```javascript
_createIconCell(sIcon, sTooltip) {
    return new Icon({
        src: sIcon,
        size: "1rem",
        tooltip: sTooltip || "",
        decorative: !sTooltip
    });
}
```

### 11. Image Cell

**Use Case**: Thumbnails, product images

```javascript
_createImageCell(oColumnConfig) {
    return new Image({
        src: `{${oColumnConfig.field}}`,
        width: "3rem",
        height: "3rem",
        densityAware: false,
        decorative: true
    });
}
```

### 12. UOM Quantity Cell

**Use Case**: Quantities with units of measure

**Production Pattern**:
```javascript
_createQuantityCell(oColumnConfig) {
    return new Text({
        text: {
            parts: [
                { path: "quantity" },
                { path: "uom" }
            ],
            formatter: (fQuantity, sUom) => {
                if (fQuantity == null) return "";
                return `${fQuantity} ${sUom || ""}`.trim();
            }
        }
    });
}
```

### 13. Conditional Cell (Different Types Based on Data)

**Production Pattern**:
```javascript
_createCell(oColumnConfig) {
    const oTemplate = new TemplateCell({
        content: {
            path: "type",
            formatter: (sType) => {
                switch (sType) {
                    case "NORMAL":
                        return this._createTextCell(oColumnConfig, "value");
                    case "LINK":
                        return this._createIdentifierCell(oColumnConfig);
                    case "STATUS":
                        return this._createStatusCell(oColumnConfig);
                    default:
                        return new Text({ text: "" });
                }
            }
        }
    });
    return oTemplate;
}
```

## Performance Patterns

### Cell Factory Pattern

For very large tables, use factory functions:

```javascript
_getItemBindingInfo(oTable, oTemplate) {
    return {
        ...super._getItemBindingInfo(oTable, oTemplate),
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

## Cell Creation Best Practices

### ✅ DO:
- Always validate data before formatting (null/undefined checks)
- Use `FlexRendertype.Bare` for HBox/VBox in cells
- Import proper enum values (don't use strings)
- Use `this.getI18nText()` for button labels
- Bind `enabled` for authorization
- Use formatters for business logic
- Pre-select rows before actions

### ❌ DON'T:
- Forget null checks in multi-part formatters
- Use bindings in `_createView()` (use method calls)
- Create controls without proper cleanup
- Hardcode text (use i18n)
- Put business logic in templates
- Forget to import required classes

## Complete Example: Production Table Cell

```javascript
_createCell(oColumnConfig) {
    switch (oColumnConfig.field) {
        case Field.SfcName:
            return this._createIdentifierCell(oColumnConfig, {
                press: (oEvent) => this._onSfcPress(oEvent)
            });
            
        case Field.SfcStatus:
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
            
        case Field.DateRange:
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
            
        case Field.Quantity:
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
            
        case Field.Actions:
            return new HBox({
                items: [
                    new CustomButton({
                        text: this.getI18nText("execute"),
                        type: ButtonType.Success,
                        enabled: "{authorized}",
                        press: this._onExecute.bind(this)
                    })
                ]
            });
            
        default:
            return this._createTextCell(oColumnConfig, oColumnConfig.field);
    }
}
```

## Quick Reference: Cell Type Matrix

| Cell Type | Complexity | Use Case | Key Classes |
|-----------|------------|----------|-------------|
| Text | ⭐ | Simple display | Text |
| Date | ⭐⭐ | Formatted dates | Text + DateTimeUtils |
| Identifier | ⭐⭐ | Clickable links | Link |
| Status | ⭐⭐⭐ | Icon + text | HBox + StatusIcon + Text |
| Date Range | ⭐⭐⭐ | Start → end | Text + multi-part binding |
| Bullet Chart | ⭐⭐⭐⭐ | Progress | BulletChart (via helper) |
| Actions | ⭐⭐⭐⭐ | Buttons | HBox + CustomButton |
| Composite | ⭐⭐⭐⭐⭐ | Complex layouts | VBox/HBox + multiple |

---

**Related References**:
- [tablewidget-complete.md](tablewidget-complete.md) - Complete TableWidget patterns
- [binding-patterns.md](binding-patterns.md) - Data binding techniques
- [widget-patterns.md](widget-patterns.md) - Full widget templates
