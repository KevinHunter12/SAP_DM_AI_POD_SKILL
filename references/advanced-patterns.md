# Advanced Production Patterns for POD 2.0 Plugins

**Source**: Real SAP Digital Manufacturing production code patterns

These patterns are essential for building enterprise-grade POD plugins that match SAP's production code standards.

---

## 1. Modern JavaScript Private Fields (#)

**Pattern**: Use `#` prefix for truly private fields (preferred in modern SAP code)

```javascript
class MyWidget extends Widget {
    #oLog;           // Private logger
    #oDialog;        // Private dialog reference
    #oTable;         // Private table reference
    #mUomMap = {};   // Private cache map
    
    constructor(oConfig) {
        super(oConfig);
        this.#oLog = Logger.getLogger("MyWidget");
    }
    
    _someMethod() {
        this.#oLog.info("Method called");
    }
}

// ❌ Old Pattern (still works, but private fields preferred)
class MyWidget extends Widget {
    _oDialog;        // Underscore convention
}
```

---

## 2. Static PropertyId Enum Pattern

**Pattern**: Define property IDs as frozen enum with parent spreading

```javascript
class MyTableWidget extends TableWidget {
    // Define property IDs as frozen enum
    static PropertyId = Object.freeze({
        ...super.PropertyId,        // Inherit parent property IDs
        customField: "customField",
        allowOnlyBaseUoM: "allowOnlyBaseUoM",
        enablePostingDate: "enablePostingDate"
    });
    
    // Use in getProperties()
    getProperties() {
        return [
            ...super.getProperties(),
            new WidgetProperty({
                propertyEditor: new StringPropertyEditor(
                    this,
                    MyTableWidget.PropertyId.customField  // Type-safe reference
                )
            })
        ];
    }
}
```

---

## 3. Static Field Enum Pattern

**Pattern**: Define table column field identifiers as frozen enum

```javascript
class MyTableWidget extends TableWidget {
    // Define field names as enum for type safety
    static Field = Object.freeze({
        parameter: "parameter",
        standardValue: "standardValue",
        reported: "reported",
        posting: "posting"
    });
    
    static getFields() {
        return [
            {
                field: this.Field.parameter,  // Use enum, not string literals
                text: "{i18n>columns.parameter}",
                width: "40%"
            }
        ];
    }
    
    _createCell(oColumnConfig) {
        switch (oColumnConfig.field) {
            case MyTableWidget.Field.parameter:
                return this._createTextCell(...);
            // ...
        }
    }
}
```

---

## 4. Advanced TableWidget - Custom Toolbar

**Pattern**: Override `_createToolbar()` to add custom toolbar with buttons

```javascript
class MyTableWidget extends TableWidget {
    #oTable;
    #oReportButton;
    
    // Override to add custom toolbar
    _createToolbar() {
        this.#oReportButton = new Button({
            text: this.getI18nText("report.btn"),
            press: () => this.onReportButtonPress(),
            enabled: "{/reportButtonEnabled}"  // Bind to model
        });
        
        return new Toolbar({
            content: [
                new Title({
                    text: {
                        parts: [`${ModelPath.ActivitySummaryList}/length`],
                        formatter: (iLength) => {
                            return this.getI18nText("toolbarTitle", iLength || 0);
                        }
                    }
                }),
                new ToolbarSpacer(),
                this.#oReportButton
            ]
        });
    }
    
    // Override to inject toolbar
    _createTable(oTableConfig) {
        this.#oTable = super._createTable(oTableConfig);
        this.#oTable.setHeaderToolbar(this._createToolbar());
        return this.#oTable;
    }
}
```

---

## 5. Advanced TableWidget - Complex Cell Types

**Pattern**: Create different cell types (identifier, text with UoM, buttons)

```javascript
_createCell(oColumnConfig) {
    switch (oColumnConfig.field) {
        case MyTableWidget.Field.parameter:
            // Identifier cell with composite binding
            return this._createIdentifierCell(oColumnConfig, {
                parts: [
                    { path: "activityId" },
                    { path: "activityText" }
                ],
                formatter: Formatter.formatActivityIdWithText
            });
            
        case MyTableWidget.Field.standardValue:
            // Text cell with value + UoM
            return this._createTextCell(oColumnConfig, {
                parts: [
                    { path: "targetQuantity" },
                    { path: "targetQuantityUom/uom" }
                ],
                formatter: Formatter.formatValueWithUom
            });
            
        case MyTableWidget.Field.posting:
            // Button cell with conditional enabling
            return new Button({
                type: ButtonType.Default,
                text: PodContext.getI18nText("viewPostings.btn"),
                enabled: {
                    parts: ["actualQuantity"],
                    formatter: (iValue) => iValue > 0
                },
                press: this.onViewPostingButtonPress.bind(this)
            });
            
        default:
            throw new Error(`Column field ${oColumnConfig.field} not supported`);
    }
}
```

---

## 6. Dynamic Button Enabling with Authorization

**Pattern**: Check user authorization and business logic before enabling actions

```javascript
async _updateReportButtonEnabled() {
    let bEnabled = false;
    try {
        const oOperation = PodContext.getLastSelectedOperationActivity();
        if (!oOperation) {
            this.getView().getModel().setProperty("/reportButtonEnabled", false);
            return;
        }
        
        // Check authorization
        const bAuthorized = await ApiClient.internal.plant.isUserAssignedToWorkCenter(
            PodContext.getPlant(),
            PodContext.getUserId(),
            oOperation.workCenter
        );
        
        // Check other conditions
        const bHasData = PodContext.getActivitySummaries().summaries.length > 0;
        const bNotDone = !oOperation.statusComplete;
        
        bEnabled = bAuthorized && bHasData && bNotDone;
    } catch (e) {
        this.#oLog.error("Error checking button enabled state", e);
    }
    
    this.getView().getModel().setProperty("/reportButtonEnabled", bEnabled);
}
```

---

## 7. ContentHandler with Dialog and Form

**Pattern**: Complex form with validation and API posting

```javascript
class ReportActivityContentHandler {
    #oModel;
    #oDialog;
    #oConfirmButton;
    #oForm;
    #oLog = Logger.getLogger("ReportActivityContentHandler");
    
    constructor() {
        this.#oModel = new JSONModel();
    }
    
    // Entry point - open as dialog
    async openAsDialog(oData) {
        this.#setModelData(oData);
        await this.#createForm();
        this.#oForm.setModel(this.#oModel);
        
        this.#oDialog = new Dialog({
            title: PodContext.getI18nText("dialog.title"),
            contentWidth: "30%",
            resizable: true,
            draggable: true,
            busyIndicatorDelay: 0,
            content: [this.#oForm],
            buttons: [
                new Button({
                    text: PodContext.getI18nText("confirm.btn"),
                    type: ButtonType.Emphasized,
                    press: () => this.#onConfirmButtonPress()
                }),
                new Button({
                    text: PodContext.getI18nText("cancel.btn"),
                    press: () => this.#oDialog.close()
                })
            ],
            afterClose: () => this.#oDialog.destroy()
        });
        
        this.#oConfirmButton = this.#oDialog.getButtons()[0];
        this.#updateConfirmButtonStatus();
        this.#oDialog.open();
    }
    
    // Create form with validation
    async #createForm() {
        const aFormContent = [];
        
        // Input with live validation
        aFormContent.push(
            new Label({ text: PodContext.getI18nText("quantity.label") }),
            new Input({
                textAlign: TextAlign.Right,
                valueLiveUpdate: true,
                change: (oEvent) => this.#onQuantityChange(oEvent)
            })
        );
        
        // Date picker with validation
        aFormContent.push(
            new Label({ text: PodContext.getI18nText("postingDate.label") }),
            new DatePicker({
                valueFormat: "yyyy-MM-dd",
                value: {
                    path: "/postingDate",
                    formatter: DateTimeUtils.localeDate
                },
                maxDate: DateTimeUtils.now(),
                change: (oEvent) => this.#onChangePostingDate(oEvent)
            })
        );
        
        this.#oForm = new SimpleForm({
            editable: true,
            layout: SimpleFormLayout.ResponsiveGridLayout,
            labelSpanXL: 12,
            labelSpanL: 12,
            labelSpanM: 12,
            labelSpanS: 12,
            content: aFormContent
        });
        
        return this.#oForm;
    }
    
    // Validation handlers
    #onQuantityChange(oEvent) {
        const oInput = oEvent.getSource();
        try {
            ValidationUtils.validateQuantity(oInput.getValue());
            oInput.setValueState(ValueState.None);
            oInput.setValueStateText("");
        } catch (oException) {
            oInput.setValueState(ValueState.Error);
            oInput.setValueStateText(oException.message);
        }
        this.#updateConfirmButtonStatus();
    }
    
    // Enable/disable confirm button based on validation
    #updateConfirmButtonStatus() {
        if (!this.#oForm || !this.#oConfirmButton) return;
        
        let bHasErrors = false;
        for (const oControl of this.#oForm.getContent()) {
            if ((oControl instanceof Input || oControl instanceof DatePicker) &&
                oControl.getValueState() === ValueState.Error) {
                bHasErrors = true;
                break;
            }
        }
        
        this.#oConfirmButton.setEnabled(!bHasErrors);
    }
    
    // API posting with error handling
    async #onConfirmButtonPress() {
        const oRequest = this.#buildRequest();
        
        this.#oDialog.setBusy(true);
        try {
            await ApiClient.internal.activityconfirmation.postActivityConfirmation(oRequest);
            MessageHistory.toast({
                message: PodContext.getI18nText("postingSuccess"),
                type: MessageHistory.Success
            });
            await ActivityConfirmationDelegate.refreshActivitySummaries({ force: true });
        } catch (oError) {
            this.#oLog.error("Posting failed", oError);
            MessageHistory.showError(PodContext.getI18nText("postingError"));
        } finally {
            this.#oDialog.close();
        }
    }
}
```

---

## 8. Custom Dialog Extension Pattern

**Pattern**: Extend `sap.m.Dialog` directly (NOT Widget) for custom dialogs

```javascript
// Custom Dialog (extends sap.m.Dialog, NOT Widget)
class ViewPostingsDialog extends Dialog {
    #oModel;
    #oTable;
    
    constructor() {
        super({
            title: PodContext.getI18nText("dialog.title"),
            contentWidth: "800px",
            contentHeight: "400px",
            buttons: [
                new Button({
                    text: PodContext.getI18nText("close.btn"),
                    type: ButtonType.Emphasized,
                    press: () => this.close()
                })
            ],
            afterClose: () => this.destroy()  // Always destroy after close
        });
        
        this.#oModel = new JSONModel();
        this.setModel(this.#oModel);
    }
    
    openDialog(oData) {
        this.#oModel.setData(oData);
        this.#createContent();
        this.open();
    }
    
    #createContent() {
        this.#oTable = new Table({
            columns: [
                new Column({ header: new Text({ text: "Quantity" }) }),
                new Column({ header: new Text({ text: "Posted By" }) })
            ]
        });
        
        this.#oTable.bindItems({
            path: "/items",
            template: new ColumnListItem({
                cells: [
                    new Text({ text: "{quantity}" }),
                    new Text({ text: "{postedBy}" })
                ]
            })
        });
        
        this.addContent(this.#oTable);
    }
}

// Usage from widget
class MyWidget extends Widget {
    #oViewDialog;
    
    _onButtonPress() {
        this.#oViewDialog = new ViewPostingsDialog();
        this.#oViewDialog.openDialog({ items: [...] });
    }
}
```

---

## 9. Formatter Utility Class Pattern

**Pattern**: Static utility class with reusable formatters

```javascript
// Reusable Formatter Utility Class
class ActivityConfirmationWidgetFormatter {
    /**
     * Format value with unit of measure
     * @param {number|string} vValue - The value
     * @param {string} sUom - Unit of measure
     * @returns {string} Formatted string
     */
    static formatValueWithUom(vValue, sUom) {
        if (vValue === null) {
            vValue = 0;
        }
        let nValue = typeof vValue === "string" 
            ? NumberFormatter.parse(vValue) 
            : vValue;
        return NumberFormatter.formatQuantityWithUom(nValue, sUom);
    }
    
    /**
     * Format activity ID with text
     * @param {string} sId - Activity ID
     * @param {string} sText - Activity text
     * @returns {string} Formatted string
     */
    static formatActivityIdWithText(sId, sText) {
        if (sText) {
            return PodContext.getI18nText("activityIdWithText", sId, sText);
        }
        return sId;
    }
}

// Usage in widget with composite binding
_createCell(oColumnConfig) {
    return this._createTextCell(oColumnConfig, {
        parts: [
            { path: "targetQuantity" },
            { path: "targetQuantityUom/uom" }
        ],
        formatter: ActivityConfirmationWidgetFormatter.formatValueWithUom
    });
}
```

---

## 10. UOM (Unit of Measure) Handling Pattern

**Pattern**: Fetching, caching, and Select control binding for UOMs

```javascript
class ReportActivityContentHandler {
    #mUomMap = {};  // Private cache for UOM data
    
    // Populate UOM map for all unique UOMs
    async #populateUomMap(aActivitySummaries) {
        const oUniqueUoms = new Set();
        aActivitySummaries.forEach((oData) => {
            const sUom = oData.targetQuantityUom.internalUom;
            if (sUom && !this.#mUomMap[sUom]) {
                oUniqueUoms.add(sUom);
            }
        });
        
        // Fetch all UOMs in parallel
        await Promise.all(
            Array.from(oUniqueUoms).map((sUom) =>
                ApiClient.internal.product.getUnitOfMeasure(sUom)
                    .then((oResp) => { this.#mUomMap[sUom] = oResp; })
            )
        );
    }
    
    // Create Select control with UOM options
    #createUomSelect(sInternalUom, sActivityId) {
        const oSelect = new Select({
            selectedKey: sInternalUom,
            enabled: true,
            change: (oEvent) => this.#onChangeUom(oEvent)
        })
        .data("activityId", sActivityId)  // Store context data
        .bindAggregation("items", {
            path: "/",
            template: new Item({
                key: "{internalUom}",
                text: "{uom}"
            })
        });
        
        // Set UOM model for this select
        oSelect.setModel(new JSONModel(this.#mUomMap[sInternalUom]));
        
        return oSelect;
    }
}
```

---

## 11. Data Delegate Pattern

**Pattern**: Shared business logic via delegates

```javascript
// Usage: Data delegates provide shared business logic
async onInit() {
    await super.onInit();
    
    if (PodContext.isRunMode()) {
        // Initialize shared delegate (singleton pattern)
        await ActivityConfirmationDelegate.init();
        
        // Subscribe to delegate's managed data
        PodContext.subscribe(
            ModelPath.ActivitySummaries,
            () => this._onActivitySummariesChanged(),
            this
        );
    }
}

// Trigger delegate refresh after operations
async _afterPosting() {
    // Force delegate to refresh data
    await ActivityConfirmationDelegate.refreshActivitySummaries({ force: true });
}

// When to use delegates:
// ✅ Multiple widgets need same data
// ✅ Complex data transformation logic
// ✅ Data caching and refresh management
// ✅ Coordination between widgets
//
// ❌ Don't use for widget-specific logic
// ❌ Don't use for simple PodContext.get() calls
```

---

## 12. Custom Field Extensibility Pattern

**Pattern**: Configurable custom fields with validation

```javascript
class MyWidget extends Widget {
    _sCustomFieldProperty;
    _aCustomFieldJson = [];
    
    async onInit() {
        await super.onInit();
        const oConfig = this.getConfig();
        this._sCustomFieldProperty = oConfig.properties.customField || "";
    }
    
    // Add custom field to form if configured
    _createForm() {
        const aFormContent = [];
        
        // Regular fields
        aFormContent.push(...this._createRegularFields());
        
        // Optional custom field
        if (this._sCustomFieldProperty) {
            aFormContent.push(
                new Label({ text: this._sCustomFieldProperty }),
                new Input({
                    value: "",
                    valueLiveUpdate: true,
                    liveChange: (oEvent) => this._onCustomFieldChange(oEvent)
                })
            );
        }
        
        return new SimpleForm({ content: aFormContent });
    }
    
    // Validate custom field input (A-Z, a-z, 0-9, @, ., space)
    _validateCustomField(sValue) {
        const oValidCharactersRegex = /^[A-Za-z0-9@. ]+$/;
        return oValidCharactersRegex.test(sValue);
    }
    
    // Store as JSON array
    _updateCustomFieldData(sCustomFieldId, sValue) {
        if (!sValue) {
            this._aCustomFieldJson = [];
            return;
        }
        
        const oCustomFieldData = { id: sCustomFieldId, value: sValue };
        const oExisting = this._aCustomFieldJson.find(
            (oField) => oField.id === sCustomFieldId
        );
        
        if (oExisting) {
            oExisting.value = sValue;
        } else {
            this._aCustomFieldJson.push(oCustomFieldData);
        }
    }
    
    // Include in API request
    _buildRequest() {
        const oRequest = {
            // ... other fields
            customFieldData: this._aCustomFieldJson.length > 0 
                ? JSON.stringify(this._aCustomFieldJson) 
                : ""
        };
        return oRequest;
    }
}
```

---

## 13. Warning Dialog Pattern

**Pattern**: Show warning with custom actions before proceeding

```javascript
// Warning dialog pattern
_showWarningDialog(fnProceed) {
    const oMessage = MessageHistory.showWarning(
        PodContext.getI18nText("warningMessage"),
        {
            actions: [
                PodContext.getI18nText("proceed"),
                MessageBox.Action.CANCEL
            ],
            onClose: (sAction) => {
                if (sAction === PodContext.getI18nText("proceed")) {
                    fnProceed();
                }
                MessageHistory.dismissMessage(oMessage);
            }
        }
    );
}
```

---

## 14. Expression Binding (Computed Properties)

**Pattern**: Use `{= expression }` for simple computed properties without formatters

```javascript
// Conditional visibility
new Input({
    visible: "{= ${type} === 'TEXT' }",
    value: "{value}"
})

// Null coalescing
new Label({
    text: "{= ${prompt} || ${name} }",
    required: "{= ${requiredDataEntries} > 0 }"
})

// Use expression when: simple logic, no i18n, no complex formatting
// Use formatter when: multi-parameter logic, UOM handling, i18n required
```

---

## 15. Custom Widget Events

**Pattern**: Inter-widget communication via custom events

```javascript
class MyWidget extends TableWidget {
    static EventId = Object.freeze({ Collect: "collect" });
    
    getEvents() {
        return [
            new WidgetEvent({
                id: MyWidget.EventId.Collect,
                displayName: this.getI18nText("event.collect")
            }),
            ...super.getEvents()
        ];
    }
    
    _onButtonPress(oEvent) {
        this._handleEvent(MyWidget.EventId.Collect, oEvent);
    }
}
```

---

## 16. Async Popover Pattern

**Pattern**: Show busy state while loading data

```javascript
async _onButtonPress(oEvent) {
    if (this._oCurrent?.isOpen()) return;
    
    const oPopover = new Popover({
        busy: true, busyIndicatorDelay: 0,
        afterClose: () => { oPopover.destroy(); this._oCurrent = null; }
    });
    this._oCurrent = oPopover;
    oPopover.openBy(oEvent.getSource());
    
    try {
        const aData = await this._fetchData();
        oPopover.addContent(this._createContent(aData));
    } finally {
        oPopover.setBusy(false);
    }
}
```

---

## 17. Contextual No-Data Messages

**Pattern**: Specific error messages based on POD state

```javascript
_updateNoDataText() {
    const oTable = this.getTable();
    if (!PodContext.getFilterResources()?.length) {
        oTable.setNoDataText(this.getI18nText("error.noResource"));
        return;
    }
    if (!PodContext.getSelectedOperationActivities()?.length) {
        oTable.setNoDataText(this.getI18nText("error.noOperation"));
        return;
    }
    oTable.setNoDataText(this.getI18nText("table.noData"));
}
```

---

## 18. Programmatic Table Selection

```javascript
_selectItem(oTarget) {
    const oTable = this.getTable();
    if (!oTarget) { oTable.removeSelections(); return; }
    
    const oNew = oTable.getItems().find(
        i => i.getBindingContext().getObject() === oTarget
    );
    if (oNew && oNew !== oTable.getSelectedItem()) {
        oTable.removeSelections();
        oNew.setSelected(true);
        oNew.focus();
    }
}
```

---

## 19. EXCLUDE_PROPERTIES

```javascript
class MyWidget extends TableWidget {
    static EXCLUDE_PROPERTIES = [
        ...TableWidget.EXCLUDE_PROPERTIES,
        "noDataText"  // Set programmatically
    ];
}
```

---

## 20. NavContainer Master-Detail

```javascript
const oNav = new NavContainer();
const oMaster = new Page({
    content: [oList],
    // Item press navigates to detail
});
const oDetail = new Page({
    showNavButton: true,
    navButtonPress: () => oNav.back()
});
oNav.addPage(oMaster).addPage(oDetail);
// Navigate: oNav.to(oDetail)
```

---

## Pattern Decision Matrix

| Pattern | Use When |
|---------|----------|
| Private fields (#) | Encapsulating widget state, hiding implementation details |
| Static PropertyId enum | Defining configurable widget properties, avoiding magic strings |
| Static Field enum | Defining table columns, form fields, or any fixed set of identifiers |
| Custom Toolbar | Adding action buttons, filtering, or summary info to tables |
| Complex Cells | Displaying composite data, buttons, or custom formatting in tables |
| Authorization checks | Enabling/disabling actions based on user permissions |

---

## 21. Popover Pattern with Lifecycle Management

Create and manage popovers with proper cleanup to avoid memory leaks.

```javascript
import Popover from "sap/m/Popover";
import PlacementType from "sap/m/PlacementType";
import Button from "sap/m/Button";
import ButtonType from "sap/m/ButtonType";
import Toolbar from "sap/m/Toolbar";
import ToolbarSpacer from "sap/m/ToolbarSpacer";

class ReportedQuantityTableWidget extends TableWidget {
    #oReasonCodePopover = null;
    #oSelectedListItem = null;
    
    _createReasonCodePopover() {
        const oPopover = new Popover({
            busyIndicatorDelay: 0,
            showHeader: false,
            placement: PlacementType.HorizontalPreferredRight,
            footer: new Toolbar({
                content: [
                    new ToolbarSpacer(),
                    new Button({
                        text: this.getI18nText("changeReasonCode.button"),
                        type: ButtonType.Transparent,
                        press: (oEvent) => this._onChangeReasonCodeButtonPress(oEvent)
                    })
                ]
            }),
            afterClose: () => {
                // CRITICAL: Destroy popover after close to prevent memory leaks
                this.#oReasonCodePopover.destroy();
                this.#oReasonCodePopover = null;
                this.#oSelectedListItem = null;
            }
        });
        
        oPopover.addStyleClass("sapUiContentPadding");
        return oPopover;
    }
    
    async _onReasonCodeLinkPress(oEvent) {
        const oLink = oEvent.getSource();
        const oListItem = oLink.getParent().getParent(); // HBox -> ColumnListItem
        
        // Reuse pattern: close if same item clicked
        if (oListItem === this.#oSelectedListItem) {
            if (this.#oReasonCodePopover) {
                this.#oReasonCodePopover.close();
            }
            return;
        }
        
        this.#oSelectedListItem = oListItem;
        
        if (!this.#oReasonCodePopover) {
            this.#oReasonCodePopover = this._createReasonCodePopover();
        }
        
        this.#oReasonCodePopover.setBusy(true);
        this.#oReasonCodePopover.openBy(oLink);
        
        // Load data asynchronously
        try {
            const oBindingContext = oListItem.getBindingContext();
            const sReasonCode = oBindingContext.getProperty("reasonCode");
            const oData = await ApiClient.internal.plant.getReasonCodeDetails(sReasonCode);
            
            this.#oReasonCodePopover.removeAllContent();
            this.#oReasonCodePopover.addContent(new Text({ text: oData.description }));
        } catch (error) {
            console.error("Failed to load reason code details:", error);
        } finally {
            this.#oReasonCodePopover.setBusy(false);
        }
    }
}
```

**Key Points:**
- Private field for popover reference
- `afterClose` handler with `destroy()` call
- `openBy()` for positioning
- Busy indicator while loading
- Reuse detection (close if same item clicked)

---

## 22. Data Delegate Pattern for Shared State

Use delegate classes to manage shared data across multiple widgets (DRY principle).

```javascript
// util/QuantityConfirmationDelegate.js
sap.ui.define([
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath"
], (PodContext, ModelPath) => {
    "use strict";
    
    /**
     * Delegate for managing quantity confirmation data shared across widgets
     */
    class QuantityConfirmationDelegate {
        static #iCurrentPage = 0;
        static #iTotalPages = 0;
        
        /**
         * Refresh all quantity confirmation data
         * @param {Object} options - Refresh options
         * @param {boolean} options.force - Force refresh even if cached
         */
        static async refresh(options = {}) {
            this.#iCurrentPage = 0;
            
            PodContext.set(ModelPath.ReportedQuantityLoading, true);
            
            try {
                const oWorkListItem = PodContext.getLastSelectedWorkListItem();
                const oOperationActivity = PodContext.getLastSelectedOperationActivity();
                
                const oResponse = await ApiClient.internal.sfc.getReportedQuantities({
                    plant: PodContext.getPlant(),
                    order: oWorkListItem.order,
                    sfc: oWorkListItem.sfc,
                    operation: oOperationActivity.operationActivity,
                    page: 0,
                    size: 20
                });
                
                PodContext.set(ModelPath.ReportedQuantityItems, oResponse.content);
                PodContext.set(ModelPath.ReportedQuantityCount, oResponse.totalElements);
                this.#iTotalPages = oResponse.totalPages;
                
            } catch (error) {
                console.error("Failed to refresh quantity data:", error);
                PodContext.set(ModelPath.ReportedQuantityItems, []);
                PodContext.set(ModelPath.ReportedQuantityCount, 0);
            } finally {
                PodContext.set(ModelPath.ReportedQuantityLoading, false);
            }
        }
        
        /**
         * Fetch next page of data
         */
        static async fetchNextPage() {
            if (this.#iCurrentPage >= this.#iTotalPages - 1) {
                return; // No more pages
            }
            
            this.#iCurrentPage++;
            
            try {
                const oWorkListItem = PodContext.getLastSelectedWorkListItem();
                const oOperationActivity = PodContext.getLastSelectedOperationActivity();
                
                const oResponse = await ApiClient.internal.sfc.getReportedQuantities({
                    plant: PodContext.getPlant(),
                    order: oWorkListItem.order,
                    sfc: oWorkListItem.sfc,
                    operation: oOperationActivity.operationActivity,
                    page: this.#iCurrentPage,
                    size: 20
                });
                
                // Append to existing items
                const aCurrentItems = PodContext.get(ModelPath.ReportedQuantityItems);
                PodContext.set(ModelPath.ReportedQuantityItems, [...aCurrentItems, ...oResponse.content]);
                
            } catch (error) {
                console.error("Failed to fetch next page:", error);
            }
        }
    }
    
    return QuantityConfirmationDelegate;
});

// Usage in widget:
import QuantityConfirmationDelegate from "custom/pod2/util/QuantityConfirmationDelegate";

class ReportedQuantityTableWidget extends TableWidget {
    async onInit() {
        await super.onInit();
        
        // Delegate manages data fetching and PodContext updates
        await QuantityConfirmationDelegate.refresh();
        
        if (PodContext.isRunMode()) {
            const oTable = this.getTable();
            
            oTable.attachUpdateStarted(async (oEvent) => {
                if (oEvent.getParameter("reason") === "Growing") {
                    // Delegate handles pagination
                    await QuantityConfirmationDelegate.fetchNextPage();
                }
            });
        }
    }
    
    _onChangeReasonCodeButtonPress(oEvent) {
        // ... make changes ...
        
        // Refresh data through delegate (updates all subscribed widgets)
        QuantityConfirmationDelegate.refresh({ force: true });
    }
}
```

**Key Points:**
- Shared data management across widgets
- Static methods for easy access
- PodContext updates (all widgets react)
- Pagination handling
- Cache and state management

---

## 23. Optimistic UI Update Pattern

Update UI immediately, then call API. Rollback on error.

```javascript
async _onChangeReasonCodeButtonPress(oEvent) {
    const oListItem = this.#oSelectedListItem;
    const oBindingContext = oListItem.getBindingContext();
    
    // Prompt user for reason code
    const oReasonCodeDialog = new ScrapReasonCodeDialog(
        oBindingContext.getProperty("resource")
    );
    const oReasonCode = await oReasonCodeDialog.show();
    
    if (!oReasonCode) {
        return;  // User cancelled
    }
    
    // OPTIMISTIC UPDATE: Update UI first (instant feedback)
    const oModel = oBindingContext.getModel();
    const sPath = oBindingContext.getPath();
    const sOriginalValue = oModel.getProperty(`${sPath}/reasonCodes`);
    oModel.setProperty(`${sPath}/reasonCodes`, [oReasonCode.id]);
    
    // Then call API (async)
    try {
        const sScrapActivityLogId = oBindingContext.getProperty("scrapActivityLogId");
        await ApiClient.internal.sfc.updateReportedScrapReasonCode(
            sScrapActivityLogId,
            oReasonCode
        );
        
        // Refresh to get authoritative data
        QuantityConfirmationDelegate.refresh({ force: true });
        
        MessageToast.show(this.getI18nText("message.reasonCodeUpdated"));
        
    } catch (error) {
        // ROLLBACK: Restore original value on error
        oModel.setProperty(`${sPath}/reasonCodes`, sOriginalValue);
        console.error("Failed to update reason code:", error);
        MessageBox.error(this.getI18nText("error.reasonCodeUpdateFailed"));
    }
}
```

**Key Points:**
- Update UI first (instant feedback)
- Call API asynchronously
- Store original value for rollback
- Refresh after success (authoritative data)
- Rollback on error
- UX benefit: instant vs. spinner
| ContentHandler + Dialog | Complex forms with validation and API posting |
| Custom Dialog | Reusable dialog components (view-only data, confirmations) |
| Formatter class | Reusable formatting logic across multiple widgets |
| UOM handling | Working with quantities and unit conversions |
| Data Delegates | Sharing data/logic between multiple widgets |
| Custom Fields | Making widgets configurable for customer-specific data |
| Warning dialogs | Showing warnings with custom proceed/cancel actions |
| Expression binding | Simple computed properties (visibility, enabled, text) |
| Custom events | Inter-widget communication, triggering actions in other widgets |
| Async popover | Loading data before showing dialog/popover |
| Contextual no-data | Progressive validation, specific error messages |
| Table selection | Programmatic row selection, focus management |
| NavContainer | Master-detail flows in dialogs/popovers |

---

## 14. Dynamic Column Creation Pattern ⭐⭐⭐⭐

**Production Pattern - Essential for Extensible Widgets**

### Use Case: Conditional Table Columns

Add/remove columns based on:
- Configuration properties (custom fields enabled)
- Data presence (has custom field values)
- User permissions (can see sensitive data)

### Production Implementation (From SAP Code)

```javascript
class GoodsReceiptPostingsDialog extends PodDialog {
    #oDialogParams;
    #oTable;
    
    _bindTableItems(aData) {
        // Build base columns
        const aColumns = [
            { text: "{i18n>columns.material}" },
            { text: "{i18n>columns.quantity}" },
            { text: "{i18n>columns.status}" },
            { text: "{i18n>columns.comments}" }
        ];
        
        const sCustomFieldLabel = this.#oDialogParams.customFieldLabel;
        const sCustomFieldId = this.#oDialogParams.customFieldId;
        
        // Check if ANY item has customFieldData
        let bCustomFieldDataPresent = false;
        if (sCustomFieldId && Array.isArray(aData)) {
            bCustomFieldDataPresent = aData.some((oItem) => {
                if (!oItem.customFieldData) return false;
                
                try {
                    const aArr = JSON.parse(oItem.customFieldData);
                    return Array.isArray(aArr) && 
                           aArr.some(oField => oField.id === sCustomFieldId && oField.value);
                } catch (oError) {
                    return false;
                }
            });
        }
        
        // Insert custom field column BEFORE comments column
        const iCommentsColIdx = aColumns.length - 1;
        if (sCustomFieldLabel && bCustomFieldDataPresent) {
            aColumns.splice(iCommentsColIdx, 0, { text: sCustomFieldLabel });
        }
        
        // Build cells array (must match columns!)
        const aCells = [
            new Text({ text: "{material}" }),
            new Text({ text: "{quantity/value}" }),
            new Text({ text: "{status}" }),
            new Text({ text: "{comments}" })
        ];
        
        // Insert custom field cell at SAME position
        if (sCustomFieldId && bCustomFieldDataPresent) {
            const iCommentsCellIdx = aCells.length - 1;
            aCells.splice(iCommentsCellIdx, 0, new Text({
                text: {
                    parts: ["customFieldData"],
                    formatter: (sCustomFieldData) => {
                        if (sCustomFieldData) {
                            try {
                                const aArr = JSON.parse(sCustomFieldData);
                                const oField = aArr.find(oField => oField.id === sCustomFieldId);
                                return oField ? oField.value : "";
                            } catch (oError) {
                                return "";
                            }
                        }
                        return "";
                    }
                }
            }));
        }
        
        // Create table with dynamic columns
        this.#oTable = new Table({
            columns: aColumns.map((oCol, iIdx) => new Column({
                header: new Text({ text: oCol.text }),
                ...(iIdx === 0 ? { mergeDuplicates: true } : {})
            }))
        });
        
        this.#oTable.bindItems({
            path: "/postingsList",
            template: new ColumnListItem({ cells: aCells })
        });
    }
}
```

### Critical Details

**Column/Cell Index Alignment:**
```javascript
// ❌ WRONG - Indices don't match
aColumns.push(customColumn);      // Index 5
aCells.splice(3, 0, customCell);  // Index 3 - MISMATCH!

// ✅ CORRECT - Same index
const iInsertIdx = aColumns.length - 1;
aColumns.splice(iInsertIdx, 0, customColumn);
aCells.splice(iInsertIdx, 0, customCell);
```

**Data Presence Check:**
```javascript
// Check if ANY row has the custom field
const bHasCustomField = aData.some(oItem => {
    try {
        const aFields = JSON.parse(oItem.customFieldData);
        return aFields.some(f => f.id === fieldId && f.value);
    } catch {
        return false;
    }
});
```

---

## 15. Error Handling & Retry Pattern ⭐⭐⭐⭐⭐

**Critical Production Pattern - Comprehensive Error Management**

### Complete Error Handling (From SAP Production Code)

```javascript
async _onConfirmPost() {
    if (this.#oDialog) {
        this.#oDialog.setBusy(true);
    }
    
    const oPayload = this._buildGoodsReceiptPayload();
    
    try {
        const oResponse = await ApiClient.internal.inventory.postGoodsReceipt(oPayload);
        const oLineItem = oResponse.lineItems && oResponse.lineItems[0];
        
        if (!oLineItem) {
            this.#oLog.error("No line item returned in response");
            return;
        }
        
        // Check for business errors in success response
        if (oLineItem.error) {
            const sErrorMsg = oLineItem.errorMessage || PodContext.getI18nText("goodsReceiptPosting.error");
            MessageHistory.showError(sErrorMsg);
            this.#oDialog.setBusy(false);
            return;
        }
        
        // Success
        this.#oLog.info("Goods receipt posted successfully");
        
        let sSuccessMsg = PodContext.getI18nText("postDialog.message.success", oLineItem.inventoryId);
        
        // Handle warnings in success response
        if (oLineItem.batchCharacteristicWarningMessage) {
            sSuccessMsg = PodContext.getI18nText("postDialog.message.warning", 
                oLineItem.batchCharacteristicWarningMessage);
            this.#oDialog.setBusy(false);
        }
        
        MessageHistory.toast({ message: sSuccessMsg, type: MessageHistory.Success });
        await GoodsReceiptDelegate.refreshSummary({ force: true });
        
        if (this.#oDialog) {
            this.#oDialog.close();
        }
        
    } catch (oError) {
        this.#oLog.error("Goods receipt posting failed", oError);
        
        // Extract error from response body or use error itself
        const oErr = (oError.body?.lineItems && oError.body.lineItems[0]) || oError;
        const sErrorMsg = oErr.errorMessage || oErr.message || 
                         PodContext.getI18nText("postDialog.message.error");
        
        // Handle specific error code with retry
        if (oErr.error && oErr.errorCode === "gr.warning.quantity.overtolerance") {
            const oMessage = MessageHistory.showWarning(sErrorMsg, {
                actions: [MessageBox.Action.YES, MessageBox.Action.NO],
                onClose: (sAction) => {
                    if (sAction === MessageBox.Action.YES) {
                        // Modify request and retry
                        this.#oDialog.setBusy(true);
                        this.#postModel.setProperty("/quantityToleranceCheck", false);
                        this._onConfirmPost(); // RETRY!
                    }
                    MessageHistory.dismissMessage(oMessage);
                }
            });
        } else {
            MessageHistory.showError(sErrorMsg);
        }
        
        this.#oDialog.setBusy(false);
    }
}
```

### Error Response Structures

**SAP DM APIs can return errors in multiple ways:**

```javascript
// 1. HTTP error with error in body
catch (oError) {
    // oError.body.lineItems[0].errorMessage
}

// 2. Success (200) but with error flag
const oResponse = await API.call();
if (oResponse.lineItems[0].error) {
    // Business error in success response
}

// 3. Success with warning message
if (oResponse.lineItems[0].warningMessage) {
    // Non-blocking warning
}
```

### Retry Pattern for Tolerance Warnings

```javascript
// User exceeded tolerance but can override
if (oErr.errorCode === "warning.tolerance") {
    MessageHistory.showWarning("Quantity exceeds tolerance. Continue anyway?", {
        actions: [MessageBox.Action.YES, MessageBox.Action.NO],
        onClose: (sAction) => {
            if (sAction === MessageBox.Action.YES) {
                // Set flag and retry
                this.#postModel.setProperty("/ignoreToleranceWarnings", true);
                this._onConfirmPost(); // Recursive retry
            }
        }
    });
}
```

### Error Extraction Helper

```javascript
_extractError(oError) {
    // Check nested body structure
    const oErr = oError.body?.lineItems?.[0] || oError;
    
    return {
        message: oErr.errorMessage || oErr.message || "Unknown error",
        code: oErr.errorCode || null,
        isError: !!oErr.error
    };
}
```

### Error Handling Best Practices

1. **Always set busy state** - Show loading indicator
2. **Try-catch around API calls** - Always
3. **Check response for business errors** - `oLineItem.error` even in 200 response
4. **Log errors** - Use Logger.error() with context
5. **Extract error from nested structures** - Check body.lineItems[0] vs error itself
6. **Default error messages** - Fallback to generic i18n text
7. **Handle warnings in success** - Some APIs return warnings with success
8. **Specific error code handling** - Use errorCode for retry logic
9. **Retry pattern** - Modify request and call same method again
10. **Always clear busy state** - In finally block or after each path

**See also:** [form-patterns.md#error-handling--retry](form-patterns.md#error-handling--retry) for complete forms pattern

---

## Summary

These 15 patterns represent **enterprise-grade POD 2.0 development** based on real SAP production code:

**Core Patterns (1-5):**
- Private fields, static enums, field identifiers, custom toolbars, complex cells

**UI Patterns (6-10):**
- Authorization, ContentHandler, dialogs, formatters, UOM handling

**Advanced Patterns (11-15):**
- Data delegates, custom fields, warnings, dynamic columns, error handling

**Key Takeaway:** These patterns are found in 90%+ of production POD plugins. Master them for enterprise-ready widgets.

**See also:**
- [form-patterns.md](form-patterns.md) - Complete ContentHandler, PodDialog, forms, validation
- [widget-patterns.md](widget-patterns.md) - TableWidget, ControlWidget, LayoutWidget
- [production-patterns-sap.md](production-patterns-sap.md) - JSDoc, error handling, delegates
