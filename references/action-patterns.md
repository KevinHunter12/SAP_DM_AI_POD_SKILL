# POD 2.0 Action Patterns

Reference guide for POD 2.0 Action development. Actions are the POD 2.0 equivalent of button-triggered operations — stateless execution components.

---

## Actions vs Widgets

| Aspect | Action | Widget |
|--------|--------|--------|
| **Purpose** | Execute operations | Display information |
| **Trigger** | Button click, menu selection | Automatic (lifecycle) |
| **State** | Stateless (execute and done) | Stateful (subscriptions) |
| **UI** | None or minimal (dialogs only) | Full UI rendering |
| **Lifecycle** | `getProperties()` + `execute()` only | `onInit()`, subscriptions, `onExit()` |

**Use Action when**: User clicks a button to trigger a stateless operation (start, complete, signoff).  
**Use Widget when**: Displays data continuously, reacts to context changes, has complex UI.  
**Use Both when**: Execution AND display both needed (Action updates context → Widget reacts).

---

## Base Class Hierarchy

```
Action (abstract)
├── SfcExecutionAction          # ⭐ Use for ALL SFC operations
│   ├── StartAction
│   ├── CompleteAction
│   ├── SignoffAction
│   ├── SplitAction
│   └── RelabelAction
├── PhaseExecutionAction
│   ├── StartPhaseAction
│   └── CompletePhaseAction
└── [Custom]Action              # Direct extension for non-SFC operations
```

**⭐ ALWAYS extend `SfcExecutionAction` for SFC operations**, not the base `Action` class. It provides pre-built helper methods (see below).

---

## SfcExecutionAction Helper Methods

```javascript
// Location: v2/src/sap/dm/dme/pod2/action/sfc/SfcExecutionAction.js

class SfcExecutionAction extends Action {
    #getResource()    // Returns currently selected resource (or null)
    #getSFCs()        // Returns selected SFC identifiers (string[])
    #getStepDetail()  // Returns operation/step information (StepDetail|null)
    #getQuantity()    // Returns execution quantity (number)
    #getWorkCenter()  // Returns selected work center (WorkCenter|null)
}
```

These are **private** helper methods — use them inside your SfcExecutionAction subclass.

---

## Complete Action Template

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/Logger",
    "sap/dm/dme/pod2/action/metadata/ActionProperty",
    "sap/dm/dme/pod2/action/sfc/SfcExecutionAction",
    "sap/dm/dme/pod2/api/ApiClient",
    "sap/dm/dme/pod2/context/MessageHistory",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/propertyeditor/BooleanPropertyEditor",
    "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor"
], (
    Logger,
    ActionProperty,
    SfcExecutionAction,
    ApiClient,
    MessageHistory,
    PodContext,
    BooleanPropertyEditor,
    StringPropertyEditor
) => {
    "use strict";

    /**
     * @typedef {object} StartRequest
     * @property {string} plant
     * @property {string[]} sfcs
     * @property {string} operation
     * @property {string} [resource]
     */

    /**
     * Action to start SFCs at the current operation.
     *
     * @alias namespace.action.sfc.StartAction
     * @extends sap.dm.dme.pod2.action.sfc.SfcExecutionAction
     */
    class StartAction extends SfcExecutionAction {
        /** @type {sap.dm.dme.pod2.Logger} */
        #logger = Logger.getLogger("namespace.action.sfc.StartAction");

        /** Static property identifiers */
        static PropertyId = Object.freeze({
            AutoComplete: "autoComplete",
            DefaultQuantity: "defaultQuantity"
        });

        /**
         * @returns {sap.dm.dme.pod2.action.metadata.ActionProperty[]}
         * @public
         */
        getProperties() {
            return [
                new ActionProperty({
                    propertyEditor: new BooleanPropertyEditor(
                        this,
                        StartAction.PropertyId.AutoComplete,
                        false
                    ),
                    displayName: this.getI18nText("property.autoComplete"),
                    tooltip: this.getI18nText("property.autoComplete.tooltip")
                }),
                new ActionProperty({
                    propertyEditor: new StringPropertyEditor(
                        this,
                        StartAction.PropertyId.DefaultQuantity
                    ),
                    displayName: this.getI18nText("property.defaultQuantity")
                })
            ];
        }

        /**
         * Execute the start operation.
         *
         * @returns {Promise<void>}
         * @throws {Error}
         * @public
         */
        async execute() {
            this.#validatePreconditions();
            const request = this.#buildRequest();
            await this.#executeStartOperation(request);
        }

        /**
         * @throws {Error} If preconditions not met
         * @private
         */
        #validatePreconditions() {
            if (!this.#hasSelection()) {
                throw new Error(this.getI18nText("error.noSelection"));
            }
            if (!this.#hasOperation()) {
                throw new Error(this.getI18nText("error.noOperation"));
            }
        }

        /**
         * @returns {boolean}
         * @private
         */
        #hasSelection() {
            const selectedItems = PodContext.getSelectedWorkListItems();
            return Array.isArray(selectedItems) && selectedItems.length > 0;
        }

        /**
         * @returns {boolean}
         * @private
         */
        #hasOperation() {
            const operation = PodContext.getLastSelectedOperationActivity();
            const workListItem = PodContext.getLastSelectedWorkListItem();
            return !!(operation?.operationActivity || workListItem?.operationActivity);
        }

        /**
         * @returns {StartRequest}
         * @private
         */
        #buildRequest() {
            const filterResources = PodContext.getFilterResources();
            const resource = filterResources?.[0]?.resource || null;

            const operation = PodContext.getLastSelectedOperationActivity();
            const workListItem = PodContext.getLastSelectedWorkListItem();

            const sfcs = PodContext.getSelectedWorkListItems().map(item => item.sfc);
            const operationActivity = operation?.operationActivity || workListItem?.operationActivity;

            const request = {
                plant: PodContext.getPlant(),
                sfcs,
                operation: operationActivity
            };

            if (resource) request.resource = resource;
            return request;
        }

        /**
         * @param {StartRequest} request
         * @private
         */
        async #executeStartOperation(request) {
            try {
                const response = await ApiClient.execution.sfcStart(request);
                this.#handleSuccess(response, request.sfcs.length);
            } catch (error) {
                this.#handleError(error);
            }
        }

        /**
         * @param {object} response
         * @param {number} count
         * @private
         */
        #handleSuccess(response, count) {
            this.#logger.info(`Started ${count} SFC(s)`);
            MessageHistory.showSuccess(
                this.getI18nText("success.started", [count])
            );
        }

        /**
         * @param {unknown} error
         * @throws {Error}
         * @private
         */
        #handleError(error) {
            if (error instanceof Error) {
                this.#logger.error(`Start failed: ${error.message}`, error);
                MessageHistory.showError(error.message);
            } else {
                this.#logger.error("Unknown error during start", error);
                MessageHistory.showError(this.getI18nText("error.unknown"));
            }
            throw error;
        }
    }

    return StartAction;
});
```

---

## Action Lifecycle

```
Constructor → getProperties() → execute() → [complete]
```

- `getProperties()` — called by POD Designer to build the properties panel
- `execute()` — called when the user triggers the action (button press)
- No `onInit()` / `onExit()` — actions are stateless

---

## Property Editor Catalog

### BooleanPropertyEditor
```javascript
new ActionProperty({
    propertyEditor: new BooleanPropertyEditor(
        this,           // Action instance
        "autoComplete", // Property ID
        false           // Default value
    ),
    displayName: this.getI18nText("property.autoComplete"),
    tooltip: this.getI18nText("property.autoComplete.tooltip")
})
```

### StringPropertyEditor
```javascript
new ActionProperty({
    propertyEditor: new StringPropertyEditor(
        this,
        "targetOperation"
        // No default — use getDefaultConfig() for defaults
    ),
    displayName: this.getI18nText("property.targetOperation")
})
```

### IntegerPropertyEditor
```javascript
new ActionProperty({
    propertyEditor: new IntegerPropertyEditor(
        this,
        "maxQuantity",
        100
    ),
    displayName: this.getI18nText("property.maxQuantity")
})
```

### DropdownPropertyEditor
```javascript
new ActionProperty({
    propertyEditor: new DropdownPropertyEditor(
        this,
        "mode",
        "default",
        [
            { key: "default", text: "Default" },
            { key: "advanced", text: "Advanced" }
        ]
    ),
    displayName: this.getI18nText("property.mode")
})
```

---

## Action Patterns Cookbook

### Pattern 1: Simple Execution Action

```javascript
async execute() {
    this.#validatePreconditions();
    const request = this.#buildRequest();
    await this.#callApi(request);
}

#validatePreconditions() {
    if (!this.#hasSelection()) {
        throw new Error(this.getI18nText("error.noSelection"));
    }
}

async #callApi(request) {
    try {
        const response = await ApiClient.execution.sfcStart(request);
        this.#handleSuccess(response);
    } catch (error) {
        this.#handleError(error);
    }
}
```

### Pattern 2: Action with Confirmation Dialog

```javascript
async execute() {
    if (!this.#shouldShowDialog()) {
        return this.#executeDirectly();
    }
    return this.#showConfirmationDialog();
}

#shouldShowDialog() {
    return this.getConfiguration().showDialog !== false;
}

async #showConfirmationDialog() {
    const confirmed = await DialogService.confirm({
        title: this.getI18nText("dialog.title"),
        message: this.getI18nText("dialog.message")
    });
    if (confirmed) {
        await this.#executeDirectly();
    }
}
```

### Pattern 3: Action with Partial Success

```javascript
async #callApi(request) {
    try {
        const response = await ApiClient.execution.sfcComplete(request);
        this.#processPartialResponse(response);
    } catch (error) {
        this.#handleError(error);
    }
}

#processPartialResponse(response) {
    const successes = response.items.filter(item => item.success);
    const failures = response.items.filter(item => !item.success);

    if (successes.length > 0) {
        MessageHistory.showSuccess(
            this.getI18nText("success.completed", [successes.length])
        );
    }
    if (failures.length > 0) {
        MessageHistory.showWarning(
            this.getI18nText("warning.partial", [failures.length])
        );
    }
}
```

---

## i18n for Actions

### File Location
```
i18n/i18n_en.properties      # (in your plugin folder)
```

### Key Naming Convention

```properties
# Button/trigger labels
button.label = Start
button.tooltip = Start selected SFCs at current operation

# Property editor labels
property.autoComplete = Auto Complete
property.autoComplete.tooltip = Automatically complete after starting
property.maxQuantity = Max Quantity

# Error messages (specific)
error.noSelection = Select at least one item first
error.noOperation = No operation selected
error.noResource = No resource selected
error.unknown = An unexpected error occurred

# Success messages (with count placeholders)
success.started = Started {0} item(s) successfully
success.completed = Completed {0} of {1} items

# Dialog messages
dialog.confirm.title = Confirm Action
dialog.confirm.message = Are you sure you want to proceed?
```

---

## extension.json for Actions

```json
{
  "widgets": [],
  "actions": [
    {
      "modulePath": "namespace/action/StartAction",
      "type": "namespace.action.StartAction"
    }
  ]
}
```

**Combined widget + action:**
```json
{
  "widgets": [
    {
      "modulePath": "namespace/widget/StatusWidget",
      "type": "namespace.widget.StatusWidget"
    }
  ],
  "actions": [
    {
      "modulePath": "namespace/action/StartAction",
      "type": "namespace.action.StartAction"
    }
  ]
}
```

---

## File Structure for Actions

```
<working-directory>/
├── extension.json
├── action/
│   └── StartAction.js        # Action class
├── widget/                   # If combined with widget
│   └── StatusWidget.js
└── i18n/
    ├── i18n_en.properties
    ├── i18n_de.properties
    ├── i18n_zh.properties
    └── i18n_ja.properties
```

---

## Code Generation Guidelines (MANDATORY)

These rules apply to ALL generated action code.

### 1. Separation of Concerns

```javascript
// ✅ GOOD: execute() orchestrates only
async execute() {
    this.#validatePreconditions();
    const request = this.#buildRequest();
    await this.#callApi(request);
}

// ❌ BAD: everything in execute()
async execute() {
    // 40 lines of validation + request building + API + error handling
}
```

### 2. Max 20 Lines Per Function

Break long methods into helpers. If a method needs more than 20 lines, extract helper methods.

### 3. Guard Clauses First

```javascript
// ✅ GOOD: early exit, happy path at lowest indentation
async execute() {
    if (!this.#hasOperation()) return;
    if (!this.#hasSelection()) return;

    await this.#performExecution();  // Happy path
}

// ❌ BAD: nested conditions
async execute() {
    if (this.#hasOperation()) {
        if (this.#hasSelection()) {
            // Deep nesting
        }
    }
}
```

### 4. Max 2 Nesting Levels

```javascript
// ✅ GOOD: filter + map
const sfcs = items
    .filter(item => item.sfc && item.quantity > 0)
    .map(item => item.sfc);

// ❌ BAD: 3+ levels
for (const item of items) {
    if (item.sfc) {
        if (item.quantity > 0) {
            sfcs.push(item.sfc);  // Too deep!
        }
    }
}
```

### 5. No Abbreviations

```javascript
// ✅ GOOD
const selectedWorkListItems = PodContext.getSelectedWorkListItems();
const operationActivity = PodContext.getLastSelectedOperationActivity();

// ❌ BAD
const selWlItems = PodContext.getSelectedWorkListItems();
const opAct = PodContext.getLastSelectedOperationActivity();
```

### 6. Private Methods Use `#`

```javascript
class MyAction extends SfcExecutionAction {
    #logger = Logger.getLogger("...");

    async execute() { ... }          // public

    #validatePreconditions() { ... } // private - use # prefix
    #buildRequest() { ... }          // private
    #handleError(error) { ... }      // private
}
```

### 7. Type Safety — No `any`, Use `unknown` + instanceof

```javascript
// ✅ GOOD
#handleError(error) {
    if (error instanceof Error) {
        this.#logger.error(error.message, error);
        MessageHistory.showError(error.message);
    } else {
        this.#logger.error("Unknown error", error);
        MessageHistory.showError(this.getI18nText("error.unknown"));
    }
    throw error;
}

// ❌ BAD
#handleError(error: any) {
    this.#logger.error(error.message);  // Unsafe!
}
```

---

## Production Readiness Checklist

Before finalising any Action:

### Architecture
- [ ] `execute()` is orchestration only — delegates to private methods
- [ ] One-way data flow (PodContext → Action → ApiClient → MessageHistory)
- [ ] Business logic separated from API calls

### Code Quality
- [ ] Functions < 20 lines each
- [ ] Max 2 nesting levels
- [ ] Guard clauses before happy path
- [ ] No abbreviations in variable/method names
- [ ] All private methods use `#` prefix

### Documentation
- [ ] JSDoc on `execute()` and `getProperties()`
- [ ] `@typedef` for request/response objects
- [ ] Parameter and return types documented

### Type Safety
- [ ] No `any` type used
- [ ] Error handlers use `unknown` + `instanceof Error`

### POD 2.0 Compliance
- [ ] Extends `SfcExecutionAction` (not base `Action`) for SFC operations
- [ ] Uses `ApiClient` (not `AjaxUtil` or raw `fetch()`)
- [ ] Uses `PodContext` (not `getPodSelectionModel()`)
- [ ] Uses `MessageHistory` (not `MessageToast`)
- [ ] Uses `Logger`
- [ ] Correct file location: `action/` subfolder

### Testing
- [ ] >80% code coverage
- [ ] Success paths tested
- [ ] Error paths tested (API failure, validation failure)
- [ ] Edge cases covered (empty selection, null operation)
- [ ] Sinon stubs cleaned up in afterEach

---

## Testing Actions

```javascript
sap.ui.define([
    "your/namespace/action/StartAction",
    "sap/dm/dme/pod2/api/ApiClient",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/MessageHistory",
    "sap/ui/thirdparty/sinon"
], (StartAction, ApiClient, PodContext, MessageHistory, sinon) => {
    "use strict";

    QUnit.module("StartAction", {
        beforeEach: function() {
            this.sandbox = sinon.createSandbox();
            this.action = new StartAction();

            // Default stubs
            this.sandbox.stub(PodContext, "getPlant").returns("PLANT_1");
            this.sandbox.stub(PodContext, "getSelectedWorkListItems").returns([
                { sfc: "SFC001" }
            ]);
            this.sandbox.stub(PodContext, "getLastSelectedOperationActivity").returns({
                operationActivity: "OP10"
            });
            this.sandbox.stub(PodContext, "getFilterResources").returns([
                { resource: "RESOURCE1" }
            ]);

            this.apiStub = this.sandbox.stub(ApiClient.execution, "sfcStart")
                .resolves({ success: true, items: [{ sfc: "SFC001", success: true }] });

            this.successStub = this.sandbox.stub(MessageHistory, "showSuccess");
            this.errorStub = this.sandbox.stub(MessageHistory, "showError");
        },

        afterEach: function() {
            this.sandbox.restore();
            this.action.destroy();
        }
    });

    QUnit.test("execute — calls API with correct request", async function(assert) {
        await this.action.execute();

        assert.ok(this.apiStub.calledOnce, "API called once");
        const request = this.apiStub.firstCall.args[0];
        assert.strictEqual(request.plant, "PLANT_1", "Plant correct");
        assert.deepEqual(request.sfcs, ["SFC001"], "SFCs array correct");
        assert.strictEqual(request.operation, "OP10", "Operation correct");
    });

    QUnit.test("execute — shows success message", async function(assert) {
        await this.action.execute();
        assert.ok(this.successStub.calledOnce, "Success message shown");
    });

    QUnit.test("execute — throws when no selection", async function(assert) {
        this.sandbox.restore();
        this.sandbox.stub(PodContext, "getSelectedWorkListItems").returns([]);

        await assert.rejects(this.action.execute(), /noSelection/);
        assert.ok(this.apiStub.notCalled, "API not called");
    });

    QUnit.test("execute — shows error on API failure", async function(assert) {
        this.apiStub.rejects(new Error("Network error"));

        await assert.rejects(this.action.execute());
        assert.ok(this.errorStub.calledOnce, "Error message shown");
    });
});
```

---

## MessageHistory API (Quick Reference)

```javascript
import MessageHistory from "sap/dm/dme/pod2/context/MessageHistory";

MessageHistory.showSuccess(sMsg)    // ✅ Persistent success (use after API success)
MessageHistory.showError(sMsg)      // ✅ Persistent error (use after API failure or validation)
MessageHistory.showWarning(sMsg)    // ✅ Persistent warning (use for partial results)
MessageHistory.showInfo(sMsg)       // ✅ Persistent info
MessageHistory.toast(sMsg)          // ✅ Temporary (auto-dismisses — for guidance only)
MessageHistory.push(oMessage)       // ✅ Push to history popover without displaying
MessageHistory.messageBox(oOpts)    // ✅ Returns Promise — for blocking confirmations

// ❌ NEVER use MessageToast.show() — use MessageHistory methods above
```

**Decision rule:**
| Scenario | Method |
|----------|--------|
| API success | `showSuccess()` |
| API failure | `showError()` |
| Validation failure | `showError()` |
| Partial result (some failed) | `showWarning()` |
| "Nothing selected" guidance | `toast()` |
| Blocking confirmation required | `messageBox()` |
