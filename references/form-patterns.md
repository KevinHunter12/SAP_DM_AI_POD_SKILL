# Form & Dialog Patterns

Concise production patterns for ContentHandler, forms, validation, and error handling.

## #static Private Field ⭐⭐⭐⭐⭐

```javascript
class MyWidget extends Widget {
    static DEFAULT_COLOR = "blue";
    #static = /** @type {typeof MyWidget} */(this.constructor);
    
    _createView() {
        return new Text({ color: this.#static.DEFAULT_COLOR });
    }
}
```

## Dynamic Property Removal ⭐⭐⭐

```javascript
getProperties() {
    const aProperties = super.getProperties();
    const iIndex = aProperties.findIndex(oProp => oProp.getId() === "tooltip");
    if (iIndex !== -1) aProperties.splice(iIndex, 1);
    return aProperties;
}
```

## Alternative Subscriptions ⭐⭐⭐⭐⭐

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

## Error Handling & Retry ⭐⭐⭐⭐⭐

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

## Static Configuration ⭐⭐⭐⭐

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

## Design vs Run Mode ⭐⭐⭐⭐⭐

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

## JSDoc Type Casting ⭐⭐

```javascript
const oButton = /** @type {sap.m.Button} */ (super._createView());
const aItems = /** @type {Array<sap.m.Control>} */ (oView.getItems());
```

---

## Complex Form Validation ⭐⭐⭐⭐⭐

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

## Token-Based MultiInput ⭐⭐⭐⭐

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

## Bidirectional Field Dependencies ⭐⭐⭐⭐

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

**See**: [dialog-patterns.md](dialog-patterns.md), [advanced-patterns.md](advanced-patterns.md), [widget-patterns.md](widget-patterns.md)
