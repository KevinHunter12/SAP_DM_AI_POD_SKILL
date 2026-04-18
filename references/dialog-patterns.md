# Dialog Patterns for POD 2.0 Plugins

## Standalone Dialog Handler Pattern

### Overview
Dialog classes that are **NOT widgets** but provide reusable form/dialog functionality. This is a major architectural pattern used in production SAP DM code.

### When to Use
- ✅ Complex forms with validation (create/edit dialogs)
- ✅ Reusable selection dialogs (reason codes, resources)
- ✅ Multi-step wizards
- ✅ Forms that need to be used from multiple widgets
- ❌ Simple message boxes (use MessageBox instead)
- ❌ UI that needs to be part of POD Designer canvas

### Key Characteristics
- Constructor takes options object with callbacks (confirm, cancel, error)
- Not a Widget subclass - standalone class
- Lifecycle: constructor → open() → close() → destroy()
- Model initialization in open() or constructor
- Destruction in afterClose handler
- Callback pattern for communication with parent

### Production Example

```javascript
import Dialog from "sap/m/Dialog";
import VBox from "sap/m/VBox";
import Button from "sap/m/Button";
import ButtonType from "sap/m/ButtonType";
import JSONModel from "sap/ui/model/json/JSONModel";
import MessageHistory from "sap/dm/dme/pod2/core/util/MessageHistory";
import Logger from "sap/dm/dme/pod2/Logger";

/**
 * Standalone dialog handler for downtime records
 * @alias sap.dm.dme.pod2.widget.oee.DowntimeDialog
 */
class DowntimeDialog {
    /**
     * @callback ConfirmHandler
     * @memberof sap.dm.dme.pod2.widget.oee.DowntimeDialog
     */
    
    /**
     * @callback CancelHandler
     * @memberof sap.dm.dme.pod2.widget.oee.DowntimeDialog
     */
    
    /**
     * @callback ErrorHandler
     * @param {string} sErrorMessage The error message
     * @memberof sap.dm.dme.pod2.widget.oee.DowntimeDialog
     */
    
    #fnConfirm;
    #fnCancel;
    #fnError;
    #oMainDialog;
    #oModel;
    #oLog;
    #oEditData;
    
    /**
     * Creates a new DowntimeDialog instance
     * @param {Object} oOptions
     * @param {ConfirmHandler} oOptions.confirm - Callback invoked when user confirms
     * @param {CancelHandler} [oOptions.cancel] - Callback invoked when user cancels
     * @param {ErrorHandler} [oOptions.error] - Callback invoked when error occurs
     * @param {Object} [oOptions.editData] - Data to edit. If not provided, create mode
     */
    constructor(oOptions) {
        this.#oEditData = oOptions.editData || null;
        this.#oLog = Logger.getLogger("sap.dm.dme.pod2.widget.oee.DowntimeDialog");
        
        // Validate and store callbacks
        if (typeof oOptions.confirm !== "function") {
            throw new Error("DowntimeDialog requires a confirm callback function");
        }
        this.#fnConfirm = oOptions.confirm;
        this.#fnCancel = typeof oOptions.cancel === "function" ? oOptions.cancel : undefined;
        this.#fnError = typeof oOptions.error === "function" ? oOptions.error : undefined;
    }
    
    /**
     * Opens the dialog
     */
    open() {
        // Initialize model with default values
        this.#oModel = new JSONModel({
            downtime: null,
            resourceTokens: [],
            statusList: []
        });
        
        this._resetAllFields();
        
        if (!this.#oMainDialog) {
            this._createDowntimeDialog();
        }
        this.#oMainDialog.open();
    }
    
    /**
     * Closes the dialog
     */
    close() {
        if (this.#oMainDialog && this.#oMainDialog.isOpen()) {
            this.#oMainDialog.close();
        }
    }
    
    /**
     * Creates the UI5 Dialog instance
     */
    _createDowntimeDialog() {
        const sTitle = !this.#oEditData ? "Create Downtime" : "Edit Downtime";
        
        const oDialog = new Dialog({
            title: sTitle,
            contentWidth: "56rem",
            resizable: true,
            draggable: true,
            content: new VBox({
                items: this._createDialogContent()
            }),
            buttons: this._createDialogButtons(),
            afterClose: (oEvent) => {
                oEvent.getSource().destroy();
            }
        });
        
        this.#oMainDialog = oDialog;
        this.#oMainDialog.setModel(this.#oModel);
    }
    
    /**
     * Creates dialog buttons
     * @returns {Array<sap.m.Button>}
     */
    _createDialogButtons() {
        return [
            new Button({
                text: "Save",
                type: ButtonType.Emphasized,
                press: this._onDialogSave.bind(this)
            }),
            new Button({
                text: "Cancel",
                press: () => {
                    if (this.#fnCancel) {
                        this.#fnCancel();
                    }
                    this.close();
                }
            })
        ];
    }
    
    /**
     * Handles save button press with validation
     */
    async _onDialogSave() {
        if (!this._validateDialogFields()) {
            return; // Don't proceed if validation fails
        }
        
        try {
            // Perform save logic
            await this._performSave();
            
            // Call success callback
            if (this.#fnConfirm && typeof this.#fnConfirm === "function") {
                this.#fnConfirm();
            }
            this.close();
        } catch (oError) {
            this.#oLog.error("Save failed:", oError);
            if (this.#fnError) {
                this.#fnError(oError.message);
            } else {
                MessageHistory.showError(oError.message);
            }
        }
    }
    
    /**
     * Validates all dialog fields
     * @returns {boolean} True if all validations pass
     */
    _validateDialogFields() {
        // See form-patterns.md for comprehensive validation examples
        return true;
    }
    
    /**
     * Resets all fields based on create/edit mode
     */
    _resetAllFields() {
        // See "Create vs. Edit Mode Pattern" below
    }
}

// USAGE FROM WIDGET:
class MyWidget extends Widget {
    async _handleCreateRecord() {
        const oDialog = new DowntimeDialog({
            editData: null, // Create mode
            confirm: () => this._refreshData(),
            cancel: () => console.log("Cancelled"),
            error: (sError) => MessageHistory.showError(sError)
        });
        await oDialog.open();
    }
    
    async _handleEditRecord() {
        const oSelectedItem = this.getTable().getSelectedItem();
        const oData = oSelectedItem.getBindingContext().getObject();
        
        const oDialog = new DowntimeDialog({
            editData: oData, // Edit mode
            confirm: () => this._refreshData()
        });
        await oDialog.open();
    }
}
```

### Benefits
- **Reusable** across multiple widgets
- **Clear separation** of concerns
- **Testable** in isolation
- **Callback-based** communication (no tight coupling)
- **Own lifecycle** management

---

## Create vs. Edit Mode Pattern

### Overview
Single dialog class that handles both create and edit modes with different behaviors based on presence of `editData`.

### Mode Determination
Mode is determined by the presence of `editData` in constructor options:
- `editData === null` → **Create Mode**
- `editData !== null` → **Edit Mode**

### Key Differences

| Aspect | Create Mode | Edit Mode |
|--------|------------|-----------|
| **Data Source** | `null` or defaults | Existing record |
| **Field Enablement** | All editable | Key fields locked |
| **Validation** | Full validation | Partial validation |
| **Button Text** | "Create" | "Save" / "Update" |
| **API Endpoint** | POST /create | PUT /update |

### Implementation Pattern

```javascript
class MyDialog {
    #oEditData;
    
    constructor(oOptions) {
        // Store edit data to determine mode
        this.#oEditData = oOptions.editData || null;
    }
    
    /**
     * Resets fields based on create/edit mode
     */
    _resetAllFields() {
        const oModel = this._getModel();
        
        if (this.#oEditData) {
            // EDIT MODE: Load existing data
            oModel.setProperty("/record", new Record(this.#oEditData));
            oModel.setProperty("/resourceTokens", [this.#oEditData.resource]);
        } else {
            // CREATE MODE: Set defaults
            oModel.setProperty("/record", new Record({
                id: "",
                plant: PodContext.getPlant(),
                status: 0,
                startDate: new Date()
            }));
            oModel.setProperty("/resourceTokens", []);
        }
    }
    
    /**
     * Create field with mode-dependent enablement
     */
    _createResourceField() {
        return new MultiInput({
            tokens: {
                path: "/resourceTokens",
                template: new Token({ text: "{}" })
            },
            showValueHelp: true,
            valueHelpOnly: true,
            valueHelpRequest: this._openResourceDialog.bind(this),
            required: true,
            enabled: !this.#oEditData  // Disabled in edit mode
        });
    }
    
    /**
     * Create dialog with mode-dependent title
     */
    _createDialog() {
        const sTitle = !this.#oEditData ? 
            "Create Record" : 
            "Edit Record";
        
        return new Dialog({
            title: sTitle,
            buttons: this._createButtons()
        });
    }
    
    /**
     * Save logic branches by mode
     */
    async _onDialogSave() {
        if (!this._validateDialogFields()) {
            return;
        }
        
        if (!this.#oEditData) {
            await this._handleCreateSave();
        } else {
            await this._handleEditSave();
        }
        
        this.#fnConfirm();
        this.close();
    }
    
    async _handleCreateSave() {
        const oRecord = this.#oModel.getProperty("/record");
        await ApiClient.internal.createRecord(oRecord);
    }
    
    async _handleEditSave() {
        const oRecord = this.#oModel.getProperty("/record");
        // Use original key from editData for update
        await ApiClient.internal.updateRecord(
            this.#oEditData.id, 
            oRecord
        );
    }
}
```

### Best Practices
- ✅ Single dialog class for both modes (don't duplicate)
- ✅ Mode determined by presence of `editData` in constructor
- ✅ Disable key fields in edit mode (ID, resource, plant)
- ✅ Different validation rules per mode
- ✅ Different button text per mode
- ✅ Different success messages per mode
- ✅ Preserve original key for update API call
- ✅ Clear distinction in _resetAllFields() method

---

## Helper Dialog Pattern

### Overview
Dialogs that provide supporting functionality but don't handle full CRUD operations (e.g., selection dialogs, search dialogs).

### Example: Resource Selection Dialog

```javascript
/**
 * Resource selection dialog with tree hierarchy
 * @alias sap.dm.dme.pod2.widget.oee.ResourceHierarchyDialog
 */
class ResourceHierarchyDialog {
    /**
     * @callback ResourceConfirmHandler
     * @param {Array<string>} aSelectedResources Array of selected resource names
     */
    
    #fnConfirm;
    #aSelectedResources;
    #oDialog;
    #oModel;
    
    /**
     * @param {Object} oOptions
     * @param {ResourceConfirmHandler} oOptions.confirm
     * @param {Array<string>} [oOptions.selectedResources] Pre-selected resources
     */
    constructor(oOptions) {
        this.#fnConfirm = oOptions.confirm;
        this.#aSelectedResources = oOptions.selectedResources || [];
    }
    
    async open() {
        // Load resource hierarchy
        await this._loadResourceHierarchy();
        
        // Pre-select resources
        this._preselectResources();
        
        if (!this.#oDialog) {
            this._createDialog();
        }
        this.#oDialog.open();
    }
    
    _onApply() {
        const aSelected = this.#oModel.getProperty("/selectedResources");
        this.#fnConfirm(aSelected); // Pass array of strings
        this.#oDialog.close();
    }
}
```

### When to Use
- ✅ Selection dialogs (reason codes, resources, materials)
- ✅ Search/filter dialogs
- ✅ Quick view popovers
- ✅ Confirmation dialogs with custom logic
- ❌ Full CRUD forms (use Standalone Dialog Handler instead)

---

## Related Patterns
- [Form Validation](form-patterns.md#validation-pattern) - Comprehensive validation for dialogs
- [Token-Based MultiInput](form-patterns.md#token-input) - Multi-select input fields
- [Static Cache](cache-patterns.md) - Sharing data across dialog instances
- [Error Handling](error-handling.md) - Error handling in dialogs
