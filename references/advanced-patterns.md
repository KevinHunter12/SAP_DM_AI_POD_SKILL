# Advanced Production Patterns for POD 2.0 Plugins

**Source**: Real SAP Digital Manufacturing production code patterns  
**Focus**: Advanced/specialized patterns beyond core foundations

> **Note**: For core patterns (JSDoc, private fields, logger, enums, subscriptions, error handling, delegates, type safety, ContentHandler basics), see [production-patterns-unified.md](production-patterns-unified.md).

This reference covers **truly advanced** patterns for complex UI scenarios and enterprise-scale features.

---

## Table of Contents

1. [Custom Toolbars and HeaderBar Integration](#1-custom-toolbars-and-headerbar-integration)
2. [Authorization-Based UI State Management](#2-authorization-based-ui-state-management)
3. [Dynamic Column Creation](#3-dynamic-column-creation)
4. [Advanced Error Handling with Retry Logic](#4-advanced-error-handling-with-retry-logic)
5. [Async Popover with Lifecycle Management](#5-async-popover-with-lifecycle-management)
6. [Optimistic UI Updates](#6-optimistic-ui-updates)
7. [NavContainer Master-Detail Navigation](#7-navcontainer-master-detail-navigation)
8. [Warning Dialogs with Custom Actions](#8-warning-dialogs-with-custom-actions)
9. [Contextual No-Data Messages](#9-contextual-no-data-messages)
10. [Programmatic Table Selection and Focus](#10-programmatic-table-selection-and-focus)
11. [JSDoc @extensible Markers](#11-jsdoc-extensible-markers)

---

## 1. Custom Toolbars and HeaderBar Integration

**Pattern**: Override `_createToolbar()` to add custom toolbar with action buttons, counters, and dynamic model bindings.

```javascript
import Toolbar from "sap/m/Toolbar";
import ToolbarSpacer from "sap/m/ToolbarSpacer";
import Title from "sap/m/Title";
import Button from "sap/m/Button";

class ActivityConfirmationTableWidget extends TableWidget {
    #oTable;
    #oReportButton;
    
    // Override to create custom toolbar
    _createToolbar() {
        this.#oReportButton = new Button({
            text: this.getI18nText("report.btn"),
            press: () => this.onReportButtonPress(),
            enabled: "{/reportButtonEnabled}"  // Bind to model property
        });
        
        return new Toolbar({
            content: [
                new Title({
                    text: {
                        // Dynamic title with count
                        parts: [`${ModelPath.ActivitySummaryList}/length`],
                        formatter: (iLength) => {
                            return this.getI18nText("toolbarTitle", iLength || 0);
                        }
                    }
                }),
                new ToolbarSpacer(),
                this.#oReportButton,
                new Button({
                    icon: "sap-icon://refresh",
                    press: () => this._onRefreshPress()
                })
            ]
        });
    }
    
    // Inject toolbar into table
    _createTable(oTableConfig) {
        this.#oTable = super._createTable(oTableConfig);
        this.#oTable.setHeaderToolbar(this._createToolbar());
        return this.#oTable;
    }
}
```

**Key Points:**
- Store button references as private fields for later enable/disable
- Use model binding for button `enabled` state
- Dynamic title with formatters showing counts
- ToolbarSpacer for flexible layout

---

## 2. Authorization-Based UI State Management

**Pattern**: Check user authorization dynamically to enable/disable actions based on permissions and business logic.

```javascript
class ActivityConfirmationTableWidget extends TableWidget {
    
    async onInit() {
        await super.onInit();
        
        if (PodContext.isRunMode()) {
            // Subscribe to selection changes
            PodContext.subscribe(
                ModelPath.SelectedOperationActivities,
                () => this._updateReportButtonEnabled(),
                this
            );
            
            // Initial check
            await this._updateReportButtonEnabled();
        }
    }
    
    async _updateReportButtonEnabled() {
        let bEnabled = false;
        
        try {
            const oOperation = PodContext.getLastSelectedOperationActivity();
            if (!oOperation) {
                this.getView().getModel().setProperty("/reportButtonEnabled", false);
                return;
            }
            
            // 1. Check authorization
            const bAuthorized = await ApiClient.internal.plant.isUserAssignedToWorkCenter(
                PodContext.getPlant(),
                PodContext.getUserId(),
                oOperation.workCenter
            );
            
            // 2. Check business logic
            const oSummaries = PodContext.getActivitySummaries();
            const bHasData = oSummaries && oSummaries.summaries.length > 0;
            const bNotComplete = !oOperation.statusComplete;
            
            // 3. Combine conditions
            bEnabled = bAuthorized && bHasData && bNotComplete;
            
        } catch (oError) {
            this.#oLog.error("Error checking button enabled state", oError);
        }
        
        // Update model
        this.getView().getModel().setProperty("/reportButtonEnabled", bEnabled);
    }
}
```

**Key Points:**
- Async authorization checks via API
- Combine authorization + business logic
- Update model property (button bound to it)
- Error handling with logger
- Check on selection changes

---

## 3. Dynamic Column Creation

**Pattern**: Add/remove table columns conditionally based on configuration, data presence, or permissions.

### Production Example: Custom Field Column

```javascript
class GoodsReceiptPostingsDialog extends Dialog {
    #oTable;
    #oDialogParams;
    
    _bindTableItems(aData) {
        // 1. Build base columns
        const aColumns = [
            { text: "{i18n>columns.material}" },
            { text: "{i18n>columns.quantity}" },
            { text: "{i18n>columns.status}" },
            { text: "{i18n>columns.comments}" }
        ];
        
        const sCustomFieldLabel = this.#oDialogParams.customFieldLabel;
        const sCustomFieldId = this.#oDialogParams.customFieldId;
        
        // 2. Check if ANY item has custom field data
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
        
        // 3. Insert custom field column BEFORE comments column
        const iCommentsColIdx = aColumns.length - 1;
        if (sCustomFieldLabel && bCustomFieldDataPresent) {
            aColumns.splice(iCommentsColIdx, 0, { text: sCustomFieldLabel });
        }
        
        // 4. Build cells array (MUST match columns!)
        const aCells = [
            new Text({ text: "{material}" }),
            new Text({ text: "{quantity/value}" }),
            new Text({ text: "{status}" }),
            new Text({ text: "{comments}" })
        ];
        
        // 5. Insert custom field cell at SAME position
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
        
        // 6. Create table with dynamic columns
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

**Critical Rules:**
1. Column and cell arrays MUST have matching indices
2. Use `Array.splice(index, 0, item)` to insert at specific position
3. Check data presence before adding column (don't show empty columns)
4. JSON parse with try-catch for custom field data

---

## 4. Advanced Error Handling with Retry Logic

**Pattern**: Handle SAP DM API errors with specific error code checks and retry logic for tolerance warnings.

### SAP Error Response Structures

SAP APIs can return errors in multiple ways:
1. HTTP error (catch block) with error in body
2. Success (200) but with error flag in response
3. Success with warning message

```javascript
class GoodsReceiptPostingDialog extends Dialog {
    #oDialog;
    #oPostModel;
    
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
            
            // ⚠️ Check for business errors in SUCCESS response
            if (oLineItem.error) {
                const sErrorMsg = oLineItem.errorMessage || 
                                 PodContext.getI18nText("goodsReceiptPosting.error");
                MessageHistory.showError(sErrorMsg);
                this.#oDialog.setBusy(false);
                return;
            }
            
            // Success handling
            this.#oLog.info("Goods receipt posted successfully");
            
            let sSuccessMsg = PodContext.getI18nText("postDialog.message.success", 
                                                     oLineItem.inventoryId);
            
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
            
            // Extract error from nested structure or use error itself
            const oErr = (oError.body?.lineItems && oError.body.lineItems[0]) || oError;
            const sErrorMsg = oErr.errorMessage || oErr.message || 
                             PodContext.getI18nText("postDialog.message.error");
            
            // ⚠️ Handle specific error code with RETRY logic
            if (oErr.error && oErr.errorCode === "gr.warning.quantity.overtolerance") {
                const oMessage = MessageHistory.showWarning(sErrorMsg, {
                    actions: [MessageBox.Action.YES, MessageBox.Action.NO],
                    onClose: (sAction) => {
                        if (sAction === MessageBox.Action.YES) {
                            // Modify request and RETRY
                            this.#oDialog.setBusy(true);
                            this.#oPostModel.setProperty("/quantityToleranceCheck", false);
                            this._onConfirmPost(); // ← RECURSIVE RETRY!
                        }
                        MessageHistory.dismissMessage(oMessage);
                    }
                });
            } else {
                // Generic error - no retry
                MessageHistory.showError(sErrorMsg);
            }
            
            this.#oDialog.setBusy(false);
        }
    }
    
    // Helper: Extract error from nested structures
    _extractError(oError) {
        const oErr = oError.body?.lineItems?.[0] || oError;
        
        return {
            message: oErr.errorMessage || oErr.message || "Unknown error",
            code: oErr.errorCode || null,
            isError: !!oErr.error
        };
    }
}
```

**Best Practices:**
1. Check `oLineItem.error` even in 200 response
2. Extract error from `oError.body.lineItems[0]` vs `oError` itself
3. Use specific error codes for retry logic
4. Modify model property and call same method recursively
5. Always clear busy state in all paths
6. Log errors with context

---

## 5. Async Popover with Lifecycle Management

**Pattern**: Create popovers that load data asynchronously while showing busy indicator, with proper cleanup.

```javascript
import Popover from "sap/m/Popover";
import PlacementType from "sap/m/PlacementType";
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
                // ⚠️ CRITICAL: Destroy popover after close to prevent memory leaks
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
        
        // Show busy BEFORE loading data
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
            this.#oLog.error("Failed to load reason code details", error);
            MessageHistory.showError("Failed to load details");
        } finally {
            this.#oReasonCodePopover.setBusy(false);
        }
    }
}
```

**Key Points:**
- `afterClose` with `destroy()` call (memory leak prevention)
- `openBy()` for positioning
- Set busy BEFORE opening
- Clear busy in finally block
- Reuse detection (close if same item)
- Private field for popover reference

---

## 6. Optimistic UI Updates

**Pattern**: Update UI immediately for instant feedback, then call API. Rollback on error.

```javascript
class ReportedQuantityTableWidget extends TableWidget {
    
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
        
        // 1. OPTIMISTIC UPDATE: Update UI first (instant feedback)
        const oModel = oBindingContext.getModel();
        const sPath = oBindingContext.getPath();
        const sOriginalValue = oModel.getProperty(`${sPath}/reasonCodes`);
        oModel.setProperty(`${sPath}/reasonCodes`, [oReasonCode.id]);
        
        // 2. Then call API (async)
        try {
            const sScrapActivityLogId = oBindingContext.getProperty("scrapActivityLogId");
            await ApiClient.internal.sfc.updateReportedScrapReasonCode(
                sScrapActivityLogId,
                oReasonCode
            );
            
            // 3. Refresh to get authoritative data
            QuantityConfirmationDelegate.refresh({ force: true });
            
            MessageToast.show(this.getI18nText("message.reasonCodeUpdated"));
            
        } catch (error) {
            // 4. ROLLBACK: Restore original value on error
            oModel.setProperty(`${sPath}/reasonCodes`, sOriginalValue);
            this.#oLog.error("Failed to update reason code", error);
            MessageBox.error(this.getI18nText("error.reasonCodeUpdateFailed"));
        }
    }
}
```

**Benefits:**
- Instant UI feedback (no spinner wait)
- Smoother user experience
- Rollback on error maintains data integrity
- Refresh after success ensures authoritative data

**When to Use:**
- Simple property updates
- Non-critical data changes
- User expects instant feedback

**When NOT to Use:**
- Financial transactions
- Critical business operations
- Operations that cannot be easily rolled back

---

## 7. NavContainer Master-Detail Navigation

**Pattern**: Use NavContainer for wizard-like flows or master-detail views within dialogs.

```javascript
import NavContainer from "sap/m/NavContainer";
import Page from "sap/m/Page";

class MultiStepDialog extends Dialog {
    #oNavContainer;
    #oMasterPage;
    #oDetailPage;
    
    _createContent() {
        this.#oMasterPage = new Page({
            title: "Select Item",
            content: [
                new List({
                    mode: ListMode.SingleSelectMaster,
                    items: {
                        path: "/items",
                        template: new StandardListItem({
                            title: "{name}",
                            type: ListType.Navigation,
                            press: (oEvent) => this._onItemPress(oEvent)
                        })
                    }
                })
            ]
        });
        
        this.#oDetailPage = new Page({
            title: "Item Details",
            showNavButton: true,
            navButtonPress: () => this.#oNavContainer.back(),
            content: [
                new VBox({
                    items: [
                        new Text({ text: "{/selectedItem/name}" }),
                        new Text({ text: "{/selectedItem/description}" })
                    ]
                })
            ]
        });
        
        this.#oNavContainer = new NavContainer({
            pages: [this.#oMasterPage, this.#oDetailPage],
            initialPage: this.#oMasterPage
        });
        
        return this.#oNavContainer;
    }
    
    _onItemPress(oEvent) {
        const oItem = oEvent.getSource().getBindingContext().getObject();
        this.getModel().setProperty("/selectedItem", oItem);
        
        // Navigate to detail page
        this.#oNavContainer.to(this.#oDetailPage);
    }
}
```

**Key Points:**
- Store NavContainer and Page references
- Use `showNavButton: true` on detail pages
- `navButtonPress: () => oNavContainer.back()`
- Navigate with `to(oPage)`, back with `back()`

---

## 8. Warning Dialogs with Custom Actions

**Pattern**: Show warnings with custom proceed/cancel actions before executing operations.

```javascript
class GoodsReceiptWidget extends Widget {
    
    _showOverToleranceWarning(fnProceed) {
        const oMessage = MessageHistory.showWarning(
            PodContext.getI18nText("warning.quantityOverTolerance"),
            {
                actions: [
                    PodContext.getI18nText("proceed"),
                    MessageBox.Action.CANCEL
                ],
                onClose: (sAction) => {
                    if (sAction === PodContext.getI18nText("proceed")) {
                        fnProceed(); // Execute callback
                    }
                    MessageHistory.dismissMessage(oMessage);
                }
            }
        );
    }
    
    async _onPostButtonPress() {
        const bOverTolerance = this._checkToleranceExceeded();
        
        if (bOverTolerance) {
            // Show warning with callback
            this._showWarningDialog(() => {
                this._executePost({ ignoreToleranceWarnings: true });
            });
        } else {
            // Direct execution
            await this._executePost({ ignoreToleranceWarnings: false });
        }
    }
}
```

**Key Points:**
- Pass callback function to proceed action
- Use `MessageHistory.dismissMessage()` to clean up
- Modify request parameters in callback (e.g., `ignoreWarnings: true`)

---

## 9. Contextual No-Data Messages

**Pattern**: Show specific error messages based on POD state to guide users.

```javascript
class ActivityListWidget extends TableWidget {
    
    async onInit() {
        await super.onInit();
        
        if (PodContext.isRunMode()) {
            PodContext.subscribe([
                ModelPath.FilterResources,
                ModelPath.SelectedOperationActivities
            ], this._updateNoDataText, this);
            
            this._updateNoDataText();
        }
    }
    
    _updateNoDataText() {
        const oTable = this.getTable();
        
        // Check prerequisites in order
        if (!PodContext.getFilterResources()?.length) {
            oTable.setNoDataText(this.getI18nText("error.noResource"));
            return;
        }
        
        if (!PodContext.getSelectedOperationActivities()?.length) {
            oTable.setNoDataText(this.getI18nText("error.noOperation"));
            return;
        }
        
        // Default no-data message
        oTable.setNoDataText(this.getI18nText("table.noData"));
    }
}
```

**Key Points:**
- Progressive validation (check prerequisites in order)
- Specific error messages guide user to action
- Subscribe to all relevant context paths
- Update on context changes

---

## 10. Programmatic Table Selection and Focus

**Pattern**: Programmatically select and focus table rows for keyboard navigation and accessibility.

```javascript
class WorkInstructionTableWidget extends TableWidget {
    
    _selectItem(oTarget) {
        const oTable = this.getTable();
        
        // Clear selection if no target
        if (!oTarget) {
            oTable.removeSelections();
            return;
        }
        
        // Find matching list item
        const oNew = oTable.getItems().find(oItem => {
            return oItem.getBindingContext().getObject() === oTarget;
        });
        
        // Select and focus if found and different
        if (oNew && oNew !== oTable.getSelectedItem()) {
            oTable.removeSelections();
            oNew.setSelected(true);
            oNew.focus(); // ← Accessibility!
        }
    }
    
    syncTableSelectionWithPodContext() {
        const oSelectedWorkInstruction = PodContext.getSelectedWorkInstruction();
        const oTable = this.getTable();
        const oSelectedListItem = oTable.getSelectedItem();
        
        // Early exit - already in sync
        if (oSelectedListItem && 
            oSelectedListItem.getBindingContext().getObject() === oSelectedWorkInstruction) {
            return;
        }
        
        this._selectItem(oSelectedWorkInstruction);
    }
}
```

**Key Points:**
- `setSelected(true)` for visual selection
- `focus()` for keyboard navigation and accessibility
- Early exit optimization (avoid redundant work)
- `removeSelections()` before selecting new item

---

## 11. JSDoc @extensible Markers

**Pattern**: Mark methods designed for subclass override using `@extensible` JSDoc tag.

### Why Use @extensible

In enterprise environments, widgets may be subclassed. The `@extensible` marker:
- Documents extension points for developers
- Indicates SAP-approved override methods
- Guides safe customization
- Follows SAP framework conventions

### Pattern

```javascript
class BaseWidget extends Widget {
    /**
     * Gets the display name for the widget
     * @override
     * @extensible
     * @returns {string}
     */
    static getDisplayName() {
        return "Base Widget";
    }
    
    /**
     * Formats a value for display
     * @extensible
     * @param {number} nValue The value to format
     * @returns {string} The formatted value
     */
    _formatValue(nValue) {
        return nValue.toString();
    }
    
    /**
     * Internal helper - NOT extensible
     * @private
     * @param {object} oData Data object
     * @returns {boolean}
     */
    _validateData(oData) {
        return oData !== null;
    }
}
```

### Usage Example: Subclass Override

```javascript
class CustomTimerWidget extends BaseWidget {
    /**
     * @override
     * @extensible
     */
    static getDisplayName() {
        return "Custom Timer";
    }
    
    /**
     * Custom formatting for time values
     * @override
     * @extensible
     */
    _formatValue(nValue) {
        // Custom HH:MM:SS format
        const nHours = Math.floor(nValue / 3600);
        const nMinutes = Math.floor((nValue % 3600) / 60);
        const nSeconds = nValue % 60;
        
        return `${nHours}:${nMinutes.toString().padStart(2, "0")}:${nSeconds.toString().padStart(2, "0")}`;
    }
}
```

### Guidelines

**Mark these as @extensible:**
- Static metadata methods (`getDisplayName`, `getIcon`, `getCategory`)
- Static config methods (`getDefaultConfig`)
- Formatting/transformation methods
- Validation methods that subclasses may customize
- Hook methods (`_beforeLoad`, `_afterLoad`)

**Don't mark these:**
- Private implementation details (use `@private`)
- Framework lifecycle methods (`onInit`, `onExit`) - use `@override` only
- Final methods that should never be changed

---

## Pattern Summary

| Pattern | Use Case | Impact |
|---------|----------|--------|
| Custom Toolbars | Add action buttons, counters, filters to tables | High |
| Authorization Checks | Enable/disable actions based on permissions | Critical |
| Dynamic Columns | Show/hide columns based on config/data | High |
| Error Handling + Retry | Tolerance warnings with retry logic | Critical |
| Async Popover | Load data while showing dialog | High |
| Optimistic Updates | Instant feedback with rollback | Medium |
| NavContainer | Multi-step wizards, master-detail | Medium |
| Warning Dialogs | Proceed/cancel for risky operations | Medium |
| Contextual No-Data | Guide users to fix prerequisites | Medium |
| Table Selection | Accessibility, keyboard navigation | High |
| @extensible | Document extension points | Low |

---

## 12. Static Cache Pattern ⭐⭐⭐⭐⭐

Static class with cached data shared across widget instances, preventing redundant API calls.

### When to Use
- Large datasets used by multiple widgets (UOMs, resources, reason codes)
- Data that doesn't change frequently
- Expensive API calls (hierarchy data, configuration)
- Master data (plants, work centers)
- Real-time transactional data (avoid caching)
- User-specific data that varies per context (avoid caching)

### Production Example

```javascript
import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import MessageBox from "sap/m/MessageBox";

class ResourceHierarchyCache {
    static #aCachedData = null;
    
    static async getResourceHierarchy() {
        if (this.#aCachedData) {
            return Promise.resolve(this.#aCachedData);
        }
        return await this._loadResourceHierarchy();
    }
    
    static async _loadResourceHierarchy() {
        try {
            const sPlant = PodContext.getPlant();
            const aWorkCenters = PodContext.getFilterWorkCenters();
            const sWorkCenter = aWorkCenters?.[0]?.workCenter || null;
            
            const oData = await ApiClient.internal.plant
                .getResourceHierarchyData(sPlant, sWorkCenter);
            
            const oHierarchyData = this._prepareWorkcenterData(oData);
            this.#aCachedData = oHierarchyData;
            
            return oHierarchyData;
        } catch (oError) {
            MessageBox.error(
                PodContext.getI18nText("error.resourceHierarchyFailed", oError.message)
            );
            throw oError;
        }
    }
    
    static _prepareWorkcenterData(oWorkCenterData) {
        const aFinalData = [];
        
        if (oWorkCenterData.oeeHierarchyNodes?.length > 0) {
            const aTransformed = this._transformHierarchy(oWorkCenterData.oeeHierarchyNodes);
            aFinalData.push(...aTransformed);
            
            const oResourceSet = this._getUniqueResources(aFinalData);
            const aMembers = this._transformMembers(oWorkCenterData.members);
            aFinalData.push(...aMembers.filter(o => !oResourceSet.has(o.resourceName)));
        } else if (oWorkCenterData.members?.length > 0) {
            aFinalData.push(...this._transformMembers(oWorkCenterData.members));
        }
        
        return aFinalData;
    }
    
    static clearCache() {
        this.#aCachedData = null;
    }
}

// USAGE FROM WIDGET:
class MyWidget extends Widget {
    async _loadResourceHierarchy() {
        const oData = await ResourceHierarchyCache.getResourceHierarchy();
        this.#oModel.setProperty("/resourceHierarchy", oData);
    }
}
```

### Cache Invalidation Strategies

```javascript
// 1. No Invalidation (Simplest)
static #aCachedData = null; // Cache lasts for session lifetime

// 2. Manual Invalidation
static clearCache() {
    this.#aCachedData = null;
}
await ApiClient.internal.plant.updateResource(...);
ResourceHierarchyCache.clearCache();

// 3. Time-Based Expiration
static #aCachedData = null;
static #cacheTimestamp = null;
static #CACHE_TTL = 5 * 60 * 1000; // 5 minutes

static async getResourceHierarchy() {
    const now = Date.now();
    if (this.#aCachedData && this.#cacheTimestamp && 
        (now - this.#cacheTimestamp) < this.#CACHE_TTL) {
        return Promise.resolve(this.#aCachedData);
    }
    return await this._loadResourceHierarchy();
}

// 4. PodContext Subscription
static init() {
    PodContext.subscribe(
        ModelPath.FilterWorkCenters,
        () => this.clearCache(),
        this
    );
}
```

### Best Practices
- Use static private field for cached data
- Single-flight pattern (prevent concurrent loads)
- Transform data once, cache transformed result
- Provide clearCache() method for manual invalidation
- Handle errors gracefully (don't cache errors)

---

## Cross-References

**For core patterns, see [production-patterns-unified.md](production-patterns-unified.md):**
- JSDoc Documentation Standards (#1)
- Private Fields and Encapsulation (#2)
- Logger Pattern (#3)
- Enum Patterns with Object.freeze (#4)
- Property Spreading (#5)
- Design Mode vs Run Mode (#6)
- Subscription Patterns (#7)
- Error Handling Patterns (#8)
- Delegate Patterns (#9)
- EXCLUDE_PROPERTIES (#10)
- Specialized Base Classes (#11)
- Multi-Part Bindings (#12)
- State Caching (#13)
- Type Safety with instanceof (#16)
- PodContext Direct Getters (#17)
- JSDoc Type Casting (#18)
- CustomPanel/CustomVBox (#19)
- ContentHandler Production Pattern (#20)
- ImageWidget Error Handling (#21)
- Bidirectional Sync (#22)
- Custom Binding Info (#23)
- Composite Bindings (#24)
- Icon Cells (#25)
- Expression Binding (#31)
- MessageHistory API (#28)
- DateTimeUtils (#29)
- Busy Indicators (#30)
- flatMap (#33)

---

## Document Metadata

**Optimization Stats:**
- Original lines: 1,588
- Optimized lines: ~800
- Reduction: ~788 lines (50%)
- Patterns removed: 16 (now in production-patterns-unified.md)
- Unique patterns kept: 11

**Consolidation Date**: 2026-04-19

---

**See Also**:
- [production-patterns-unified.md](production-patterns-unified.md) - Core foundation patterns
- [common-mistakes.md](common-mistakes.md) - 28 mistakes with fixes
- [widget-patterns.md](widget-patterns.md) - Widget templates
- [form-dialog-patterns.md](form-dialog-patterns.md) - Form and dialog patterns
