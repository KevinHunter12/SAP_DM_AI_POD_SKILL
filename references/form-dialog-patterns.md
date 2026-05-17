# Form & Dialog Patterns

Comprehensive production patterns for ContentHandler, forms, dialogs, validation, and error handling.

---

## Standalone Dialog Handler Pattern

### Overview
Dialog classes that are **NOT widgets** but provide reusable form/dialog functionality. This is a major architectural pattern used in production SAP DM code.

### When to Use
- Forms with validation (create/edit dialogs)
- Reusable selection dialogs (reason codes, resources)
- Multi-step wizards
- Forms that need to be used from multiple widgets
- Simple message boxes (use MessageBox instead)
- UI that needs to be part of POD Designer canvas

### Key Characteristics
- Constructor takes options object with callbacks (confirm, cancel, error)
- Not a Widget subclass - standalone class
- Lifecycle: constructor open() close() destroy()
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
 */
class DowntimeDialog {
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
     * @param {Function} oOptions.confirm - Callback invoked when user confirms
     * @param {Function} [oOptions.cancel] - Callback invoked when user cancels
     * @param {Function} [oOptions.error] - Callback invoked when error occurs
     * @param {Object} [oOptions.editData] - Data to edit. If not provided, create mode
     */
    constructor(oOptions) {
        this.#oEditData = oOptions.editData || null;
        this.#oLog = Logger.getLogger("sap.dm.dme.pod2.widget.oee.DowntimeDialog");
        
        if (typeof oOptions.confirm !== "function") {
            throw new Error("DowntimeDialog requires a confirm callback function");
        }
        this.#fnConfirm = oOptions.confirm;
        this.#fnCancel = typeof oOptions.cancel === "function" ? oOptions.cancel : undefined;
        this.#fnError = typeof oOptions.error === "function" ? oOptions.error : undefined;
    }
    
    open() {
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
    
    close() {
        if (this.#oMainDialog && this.#oMainDialog.isOpen()) {
            this.#oMainDialog.close();
        }
    }
    
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
    
    async _onDialogSave() {
        if (!this._validateDialogFields()) {
            return;
        }
        
        try {
            await this._performSave();
            
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
}

// USAGE FROM WIDGET:
class MyWidget extends Widget {
    async _handleCreateRecord() {
        const oDialog = new DowntimeDialog({
            editData: null,
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
            editData: oData,
            confirm: () => this._refreshData()
        });
        await oDialog.open();
    }
}
```

---

## Create vs. Edit Mode Pattern

### Mode Determination
Mode is determined by the presence of `editData` in constructor options:
- `editData === null` **Create Mode**
- `editData !== null` **Edit Mode**

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
        this.#oEditData = oOptions.editData || null;
    }
    
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
    
    _createDialog() {
        const sTitle = !this.#oEditData ? 
            "Create Record" : 
            "Edit Record";
        
        return new Dialog({
            title: sTitle,
            buttons: this._createButtons()
        });
    }
    
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
        await ApiClient.internal.updateRecord(
            this.#oEditData.id, 
            oRecord
        );
    }
}
```

---

## Complex Form Validation

```javascript
class MyDialog {
    #clearAllValueStates() {
        [this.#oResourceInput, this.#oStartTimePicker, this.#oDurationInput].forEach(oControl => {
            if (oControl) {
                oControl.setValueState(ValueState.None);
                oControl.setValueStateText("");
            }
        });
    }
    
    _validateDialogFields() {
        this.#clearAllValueStates();
        let bValid = true;
        
        // Required field
        if (this.#oResourceInput.getTokens().length === 0) {
            this.#oResourceInput.setValueState(ValueState.Error);
            this.#oResourceInput.setValueStateText(PodContext.getI18nText("error.resourceRequired"));
            bValid = false;
        }
        
        // Range validation
        const fDuration = parseFloat(this.#oDurationInput.getValue());
        if (!fDuration || fDuration < 0.01) {
            this.#oDurationInput.setValueState(ValueState.Error);
            this.#oDurationInput.setValueStateText(PodContext.getI18nText("error.durationMin"));
            bValid = false;
        }
        
        // Cross-field validation
        const oStart = this.#oModel.getProperty("/startDate");
        const oEnd = this.#oModel.getProperty("/endDate");
        if (oStart && oEnd && oStart.getTime() > oEnd.getTime()) {
            this.#oStartTimePicker.setValueState(ValueState.Error);
            this.#oStartTimePicker.setValueStateText(PodContext.getI18nText("error.startAfterEnd"));
            bValid = false;
        }
        
        return bValid;
    }
    
    async _onDialogSave() {
        if (!this._validateDialogFields()) return;
        await this._performSave();
    }
}
```

---

## Token-Based MultiInput

```javascript
_createResourceField() {
    this.#oResourceInput = new MultiInput({
        tokens: {
            path: "/resourceTokens",
            template: new Token({ text: "{}" })
        },
        showValueHelp: true,
        valueHelpOnly: true,
        valueHelpRequest: this._openResourceDialog.bind(this),
        tokenUpdate: this._onResourceTokenUpdate.bind(this),
        required: true,
        enabled: !this.#oEditData  // Disable in edit mode
    });
    return [new Label({ text: "Resources", required: true }), this.#oResourceInput];
}

async _onResourceTokenUpdate(oEvent) {
    const sType = oEvent.getParameter("type");
    const oModel = this._getModel();
    
    if (sType === "removed") {
        const aRemoved = oEvent.getParameter("removedTokens").map(t => t.getProperty("text"));
        const aTokens = oModel.getProperty("/resourceTokens").filter(s => !aRemoved.includes(s));
        oModel.setProperty("/resourceTokens", aTokens);
    } else {
        const aTokens = oEvent.getSource().getTokens().map(t => t.getProperty("text"));
        oModel.setProperty("/resourceTokens", aTokens);
        if (aTokens.length > 0) oEvent.getSource().setValueState(ValueState.None);
    }
}
```

---

## Bidirectional Field Dependencies

```javascript
// Model structure
{ record: { startDate: null, endDate: null }, startEnabled: true, endEnabled: true, durationEnabled: true }

_createDurationField() {
    this.#oDurationInput = new Input({
        value: {
            parts: ["/record/startDate", "/record/endDate"],
            formatter: () => {
                const oRec = this.#oModel.getProperty("/record");
                const iMs = oRec.endDate?.getTime() - oRec.startDate?.getTime();
                return iMs > 0 ? (iMs / 60000).toFixed(2) : "";
            }
        },
        change: this._onDurationChange.bind(this),
        type: InputType.Number,
        enabled: { path: "/durationEnabled" }
    });
}

_onDurationChange(oEvent) {
    const oModel = this._getModel();
    const fMinutes = parseFloat(oEvent.getParameter("value"));
    const oStart = oModel.getProperty("/record/startDate");
    const oEnd = oModel.getProperty("/record/endDate");
    
    if (!fMinutes) {
        oModel.setProperty("/startEnabled", true);
        oModel.setProperty("/endEnabled", true);
        return;
    }
    
    const iMs = fMinutes * 60000;
    if (oStart && !oEnd) {
        oModel.setProperty("/record/endDate", new Date(oStart.getTime() + iMs));
        oModel.setProperty("/endEnabled", false);
    } else if (oEnd && !oStart) {
        oModel.setProperty("/record/startDate", new Date(oEnd.getTime() - iMs));
        oModel.setProperty("/startEnabled", false);
    }
}
```

---

## Helper Dialog Pattern

Dialogs that provide supporting functionality but don't handle full CRUD operations (e.g., selection dialogs, search dialogs).

```javascript
/**
 * Resource selection dialog with tree hierarchy
 */
class ResourceHierarchyDialog {
    #fnConfirm;
    #aSelectedResources;
    #oDialog;
    #oModel;
    
    /**
     * @param {Object} oOptions
     * @param {Function} oOptions.confirm - Receives Array<string> of selected resources
     * @param {Array<string>} [oOptions.selectedResources] Pre-selected resources
     */
    constructor(oOptions) {
        this.#fnConfirm = oOptions.confirm;
        this.#aSelectedResources = oOptions.selectedResources || [];
    }
    
    async open() {
        await this._loadResourceHierarchy();
        this._preselectResources();
        
        if (!this.#oDialog) {
            this._createDialog();
        }
        this.#oDialog.open();
    }
    
    _onApply() {
        const aSelected = this.#oModel.getProperty("/selectedResources");
        this.#fnConfirm(aSelected);
        this.#oDialog.close();
    }
}
```

### When to Use Helper Dialogs
- Selection dialogs (reason codes, resources, materials)
- Search/filter dialogs
- Quick view popovers
- Confirmation dialogs with custom logic
- Full CRUD forms (use Standalone Dialog Handler instead)

---

## #static Private Field Pattern

Access static fields from instance methods:

```javascript
class MyWidget extends Widget {
    static DEFAULT_COLOR = "blue";
    #static = /** @type {typeof MyWidget} */(this.constructor);
    
    _createView() {
        return new Text({ color: this.#static.DEFAULT_COLOR });
    }
}
```

---

## Dynamic Property Removal

```javascript
getProperties() {
    const aProperties = super.getProperties();
    const iIndex = aProperties.findIndex(oProp => oProp.getId() === "tooltip");
    if (iIndex !== -1) aProperties.splice(iIndex, 1);
    return aProperties;
}
```

---

## Alternative Subscriptions

```javascript
// WebSocket
onInit() {
    PodNotificationWebSocket.attachStateChange(this._onStateChange, this);
}
onExit() {
    PodNotificationWebSocket.detachStateChange(this._onStateChange, this);
}

// Event Bus
sap.ui.getCore().getEventBus().subscribe("Channel", "Event", this._handler, this);

// Timers
this.#intervalId = setInterval(() => this._fetch(), 30000);
clearInterval(this.#intervalId);
```

---

## Error Handling & Retry

```javascript
try {
    const oResponse = await ApiClient.post(oPayload);
    const oItem = oResponse.lineItems?.[0];
    
    if (oItem.error) {
        MessageHistory.showError(oItem.errorMessage);
        return;
    }
    
    MessageHistory.toast({ message: "Success", type: MessageHistory.Success });
} catch (oError) {
    const oErr = oError.body?.lineItems?.[0] || oError;
    
    if (oErr.errorCode === "warning.tolerance") {
        MessageHistory.showWarning(oErr.errorMessage, {
            actions: [MessageBox.Action.YES, MessageBox.Action.NO],
            onClose: (sAction) => {
                if (sAction === MessageBox.Action.YES) {
                    this.#model.setProperty("/ignoreWarnings", true);
                    this._onConfirmPost(); // RETRY
                }
            }
        });
    } else {
        MessageHistory.showError(oErr.errorMessage || "Error");
    }
}
```

---

## Static Configuration

```javascript
class StatusWidget extends IconWidget {
    static ACTIVE_ICON = "sap-icon://status-positive";
    static ACTIVE_COLOR = IconColor.Positive;
    static ERROR_THRESHOLD = 95;
    
    #static = /** @type {typeof StatusWidget} */(this.constructor);
    
    _update(sStatus) {
        this.getView().setSrc(this.#static.ACTIVE_ICON);
    }
}
```

---

## Design vs Run Mode

```javascript
_createView() {
    const oIcon = new Icon();
    if (PodContext.isDesignMode()) {
        this._setView(oIcon);
        this._updateIcon(true); // Show preview
    }
    return oIcon;
}

async onInit() {
    await super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(ModelPath.CurrentResource, this._onChange, this);
        await this._load();
    }
}
```

---

## JSDoc Type Casting

```javascript
const oButton = /** @type {sap.m.Button} */ (super._createView());
const aItems = /** @type {Array<sap.m.Control>} */ (oView.getItems());
```

---

**See Also**: [advanced-patterns.md](advanced-patterns.md), [widget-patterns.md](widget-patterns.md), [binding-patterns.md](binding-patterns.md)
