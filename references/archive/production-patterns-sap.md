# Production Patterns from SAP POD 2.0 Code

**Source**: Extracted from SAP Digital Manufacturing v2 production code (real SAP widgets)

This reference contains actual patterns used in SAP's production POD 2.0 widgets, not theoretical examples.

---

## Table of Contents

1. [JSDoc Documentation Patterns](#jsdoc-documentation-patterns)
2. [Private Fields and Encapsulation](#private-fields-and-encapsulation)
3. [Enum Patterns with Object.freeze](#enum-patterns-with-objectfreeze)
4. [Property Spreading Patterns](#property-spreading-patterns)
5. [Design Mode vs Run Mode](#design-mode-vs-run-mode)
6. [ContentHandler Production Pattern](#contenthandler-production-pattern)
7. [Subscription Patterns](#subscription-patterns)
8. [Property Editor Patterns](#property-editor-patterns)
9. [Delegate Patterns](#delegate-patterns)
10. [Error Handling Patterns](#error-handling-patterns)

---

## JSDoc Documentation Patterns

### Pattern from SAP Production Code

```javascript
/**
 * Hello World Widget - Simple POD 2.0 plugin demonstration
 *
 * @alias custom.kevh.demo123.plugins.helloworld
 * @extends sap.dm.dme.pod2.widget.Widget
 */
class HelloWorldWidget extends Widget {

    /**
     * Display name shown in POD Designer
     * @override
     * @extensible
     * @returns {string}
     */
    static getDisplayName() {
        return "Hello World Widget";
    }

    /**
     * Icon for widget palette (SAP icon name)
     * @override
     * @returns {string}
     */
    static getIcon() {
        return "sap-icon://hello-world";
    }

    /**
     * Constructor
     * @param {Object} oConfig - Widget configuration
     */
    constructor(oConfig) {
        super(oConfig);
    }

    /**
     * Lifecycle method - called after widget initialization
     * @override
     */
    async onInit() {
        await super.onInit();
        // Additional initialization logic
    }

    /**
     * Create and return the widget view
     * @override
     * @returns {sap.ui.core.Control}
     */
    _createView() {
        // Implementation
    }
}
```

**Key Points:**
- Use `@alias` for the full class path
- Use `@extends` to show inheritance
- Use `@override` for overridden methods
- Use `@extensible` for methods that subclasses can override
- Use `@param` and `@returns` with types

---

## Private Fields and Encapsulation

### Pattern from TableWidget

```javascript
class MyTableWidget extends TableWidget {
    // Private field with # prefix (truly private)
    #oLog = Logger.getLogger("sap.dm.dme.pod2.widget.custom.MyTableWidget");

    // Typed accessor for static class reference
    #static = /** @type {typeof sap.dm.dme.pod2.widget.core.TableWidget} */(this.constructor);

    // Private table reference
    #oTable;

    // Private dialog reference
    #oViewPostingsDialog;

    // Protected field with _ prefix (convention, not enforced)
    _sCustomFieldProperty;
    _bAllowUomFlag;
    _bEnablePostingDate;

    onInit() {
        super.onInit();
        this.#oLog.info("Widget initializing");

        // Access private field
        if (!this.#oTable) {
            this.#oLog.warn("Table not initialized");
        }
    }
}
```

**Key Points:**
- Use `#field` for truly private fields (ES2019+)
- Use `_field` for protected fields (convention)
- Logger should always be private (`#oLog`)
- Store control references as private fields

---

## Enum Patterns with Object.freeze

### Pattern from ActivityConfirmationTableWidget

```javascript
class ActivityConfirmationTableWidget extends TableWidget {

    /**
     * @enum {string}
     * @property {"customField"} customField
     * @property {"allowOnlyBaseUoM"} allowOnlyBaseUoM
     * @property {"enablePostingDate"} enablePostingDate
     */
    static PropertyId = Object.freeze({
        ...super.PropertyId,  // Spread parent properties!
        customField: "customField",
        allowOnlyBaseUoM: "allowOnlyBaseUoM",
        enablePostingDate: "enablePostingDate"
    });

    /**
     * @enum {string}
     * @property {"parameter"} parameter
     * @property {"standardValue"} standardValue
     * @property {"reported"} reported
     * @property {"posting"} posting
     */
    static Field = Object.freeze({
        parameter: "parameter",
        standardValue: "standardValue",
        reported: "reported",
        posting: "posting"
    });

    // Usage in code
    getProperties() {
        return [
            new WidgetProperty({
                propertyEditor: new StringPropertyEditor(
                    this,
                    this.constructor.PropertyId.customField  // ← Type-safe!
                )
            })
        ];
    }

    _someMethod() {
        const sField = this.constructor.Field.parameter;  // ← No typos!
    }
}
```

**Key Points:**
- Use `Object.freeze()` to prevent modification
- Spread parent enums when extending: `...super.PropertyId`
- Document with `@enum` JSDoc
- Access via `this.constructor.PropertyId.X` (not `MyClass.PropertyId.X`)
- Prevents typos and enables autocomplete

---

## Property Spreading Patterns

### Pattern from Multiple SAP Widgets

```javascript
class MyWidget extends TableWidget {

    static getDefaultConfig() {
        return {
            properties: {
                // Spread parent default properties
                ...super.getDefaultConfig().properties,
                // Add/override specific properties
                showNoData: true,
                mode: ListMode.SingleSelectMaster,
                growingScrollToLoad: true
            }
        };
    }

    static EXCLUDE_PROPERTIES = [
        // Spread parent exclusions
        ...TableWidget.EXCLUDE_PROPERTIES,
        // Add specific exclusions
        "headerText"
    ];
}
```

**Key Points:**
- Always spread parent defaults: `...super.getDefaultConfig().properties`
- Override specific properties after spreading
- Same pattern for EXCLUDE_PROPERTIES, INCLUDE_EVENTS, etc.

---

## Design Mode vs Run Mode

### Pattern from SelectResourceWidget

```javascript
class SelectResourceWidget extends ControlWidget {

    constructor(oConfig) {
        // Use different control based on mode
        if (PodContext.isDesignMode()) {
            // Simplified preview control for designer
            super(CustomInput, oConfig);
        } else {
            // Full-featured control for runtime
            super(ResourceSingleInput, oConfig);
        }
    }

    onInit() {
        super.onInit();

        if (PodContext.isDesignMode()) {
            // Design-time setup
            const oPreviewInput = /** @type {sap.m.Input} */(this.getView());
            oPreviewInput.setEditable(false);
            oPreviewInput.setValue(this.getPropertyValue(PropertyId.DefaultResource));

        } else {
            // Runtime setup
            const oControl = /** @type {sap.dm.dme.pod2.valuehelp.ResourceSingleInput} */(this.getView());

            // Set initial value
            const aResources = PodContext.getFilterResources();
            if (Array.isArray(aResources) && aResources.length !== 0) {
                oControl.setSelection(aResources[0]);
            }

            // Subscribe to changes (ONLY in run mode!)
            PodContext.subscribe(ModelPath.FilterResources, (aResources) => {
                if (Array.isArray(aResources) && aResources.length !== 0) {
                    oControl.setSelection(aResources[0]);
                } else {
                    oControl.setSelection(null);
                }
            }, this);
        }
    }
}
```

**Key Points:**
- Check `PodContext.isDesignMode()` for preview behavior
- Check `PodContext.isRunMode()` before subscribing
- Use simpler controls in design mode for performance
- Type-cast controls after getView(): `/** @type {ControlType} */(this.getView())`

---

## ContentHandler Production Pattern

### Pattern from ReportActivityContentHandler

```javascript
/**
 * @alias sap.dm.dme.pod2.widget.activityconfirmation.ReportActivityContentHandler
 * @extensible
 */
class ReportActivityContentHandler {
    /** @type {sap.ui.model.json.JSONModel} */
    _oModel;

    /** @type {sap.m.Dialog} */
    _oDialog;

    /** @type {sap.m.Button} */
    _oConfirmButton;

    /** @type {sap.ui.layout.form.SimpleForm} */
    _oForm;

    /** @type {sap.dm.dme.pod2.Logger} */
    #oLog = Logger.getLogger("sap.dm.dme.pod2.widget.activityconfirmation.ReportActivityContentHandler");

    /** @type {Record<string, sap.dm.dme.pod2.api.internal.product.UnitOfMeasure>} */
    #mUomMap = {};

    constructor() {
        this._oModel = new JSONModel();
    }

    /**
     * Opens the content as a dialog.
     *
     * @extensible
     * @param {Object} oData - Data to populate the model.
     */
    async openAsDialog(oData) {
        this.#oLog.info("Opening dialog with data", oData);

        // Set model data
        this._oModel.setData(oData);

        // Create form content
        await this._createForm();
        this._oForm.setModel(this._oModel);

        // Create dialog
        const oDialog = new Dialog({
            title: PodContext.getI18nText("reportActivityDialog.title"),
            contentWidth: "30%",
            resizable: true,
            draggable: true,
            busyIndicatorDelay: 0,
            content: [ this._oForm ],
            buttons: [
                new Button({
                    text: PodContext.getI18nText("confirm.btn"),
                    type: ButtonType.Emphasized,
                    press: () => this._onConfirmButtonPress()
                }),
                new Button({
                    text: PodContext.getI18nText("cancel.btn"),
                    press: () => oDialog.close()
                })
            ],
            afterClose: () => oDialog.destroy()  // ← Always destroy!
        });

        // Store references
        this._oDialog = oDialog;
        this._oConfirmButton = oDialog.getButtons()[0];

        // Update button state
        this._updateConfirmButtonStatus();

        // Open dialog
        oDialog.open();
    }

    async _createForm() {
        // Create form fields...
        this._oForm = new SimpleForm({
            // ...
        });
    }

    async _onConfirmButtonPress() {
        this.#oLog.info("Confirm button pressed");

        const oData = this._oModel.getData();

        try {
            // Call API
            await ApiClient.custom.post("/confirm", oData);

            MessageHistory.toast({
                message: "Success",
                type: MessageHistory.Success
            });

            this._oDialog.close();

        } catch (oError) {
            this.#oLog.error("Confirmation failed", oError);
            MessageHistory.showError("Operation failed");
        }
    }
}
```

**Key Points:**
- ContentHandlers don't extend Widget
- Store `_oModel`, `_oDialog`, `_oForm` as instance fields
- Use `async openAsDialog(oData)` as entry point
- Always `afterClose: () => oDialog.destroy()` to prevent memory leaks
- Store button references for enabling/disabling

---

## Subscription Patterns

### Pattern from ActivityConfirmationTableWidget

```javascript
class ActivityConfirmationTableWidget extends TableWidget {

    async onInit() {
        super.onInit();

        // Only subscribe in run mode
        if (PodContext.isRunMode()) {
            // Subscribe to context changes
            PodContext.subscribe(
                ModelPath.ActivitySummaries,
                () => this._updateReportButtonEnabled(),
                this  // ← Context binding!
            );

            // Initialize delegates
            await ActivityConfirmationDelegate.init();
        }
    }

    onExit() {
        super.onExit();

        // Always unsubscribe
        if (PodContext.isRunMode()) {
            PodContext.unsubscribe(
                ModelPath.ActivitySummaries,
                this._updateReportButtonEnabled,
                this
            );
        }
    }
}
```

### Subscription with Inline Handler

```javascript
PodContext.subscribe(ModelPath.FilterResources, (aResources) => {
    // Defensive type checking
    if (Array.isArray(aResources) && aResources.length !== 0) {
        oControl.setSelection(aResources[0]);
    } else {
        oControl.setSelection(null);
    }
}, this);
```

**Key Points:**
- Always check `PodContext.isRunMode()` before subscribing
- Pass `this` as third parameter for context binding
- Use defensive type checking (`Array.isArray()`)
- Handle both empty arrays and null/undefined
- Always unsubscribe in `onExit()`

---

## Property Editor Patterns

### Excluding Properties from ControlWidget

```javascript
class SelectResourceWidget extends ControlWidget {

    static EXCLUDE_PROPERTIES = [
        "autocomplete", "dateFormat", "enableSuggestionsHighlighting",
        "enableTableAutoPopinMode", "fieldWidth", "filterSuggests",
        "forceUpperCase", "maxLength", "maxSuggestionWidth", "name",
        "selectedKey", "showSuggestion", "showTableSuggestionValueHelp",
        "showValueHelp", "showValueStateMessage", "startSuggestion",
        "suggestionRowValidator", "textFormatMode", "textFormatter",
        "type", "value", "valueHelpIconSrc", "valueHelpOnly",
        "valueLiveUpdate", "valueState", "valueStateText"
    ];

    static INCLUDE_EVENTS = [ ];  // No events exposed

    static PROPERTY_CATEGORY_OVERRIDE = {
        description: PropertyCategory.Appearance,
        placeholder: PropertyCategory.Appearance,
        required: PropertyCategory.Behavior,
        showClearIcon: PropertyCategory.Behavior
    };
}
```

---

## Delegate Patterns

### Using Delegates for Data Management

```javascript
import ActivityConfirmationDelegate from "sap/dm/dme/pod2/context/data/ActivityConfirmationDelegate";

class MyWidget extends TableWidget {

    async onInit() {
        super.onInit();

        if (PodContext.isRunMode()) {
            // Initialize delegate (manages data loading/caching)
            await ActivityConfirmationDelegate.init();

            // Subscribe to changes
            // Delegate automatically updates ModelPath.ActivitySummaries
            PodContext.subscribe(
                ModelPath.ActivitySummaries,
                this._onActivitiesChanged,
                this
            );
        }
    }
}
```

**Key Points:**
- Delegates handle data fetching and caching
- Initialize delegates in `onInit()`
- Subscribe to the ModelPath that the delegate manages
- Common delegates:
  - `ActivityConfirmationDelegate`
  - `WorkListDelegate`
  - `DataCollectionDelegate`

---

## Error Handling Patterns

### API Call Error Handling

```javascript
async _loadData() {
    this.#oLog.info("Loading data...");

    try {
        const oData = await ApiClient.custom.post("/endpoint", {
            plant: PodContext.getPlant()
        });

        this.#oLog.debug("Data loaded successfully", oData);
        this._updateDisplay(oData);

        MessageHistory.toast({
            message: "Data loaded successfully",
            type: MessageHistory.Success
        });

    } catch (oError) {
        this.#oLog.error("Failed to load data", oError);

        MessageHistory.showError(
            PodContext.getI18nText("error.loadFailed")
        );

        // Re-throw if caller needs to handle
        throw oError;
    }
}
```

**Key Points:**
- Always log at appropriate level (`info`, `debug`, `error`)
- Use `MessageHistory.showError()` for user-facing errors
- Use `MessageHistory.toast()` for success messages
- Re-throw errors if caller needs to handle them
- Use i18n keys for error messages

---

## Summary: Key Takeaways

1. **Use JSDoc** - Document all public methods with `@override`, `@extensible`, types
2. **Use private fields** - `#oLog`, `#oTable` for true privacy
3. **Use Object.freeze** - For PropertyId and Field enums
4. **Spread parent config** - `...super.getDefaultConfig().properties`
5. **Check run mode** - Before subscribing: `if (PodContext.isRunMode())`
6. **Type cast controls** - After getView(): `/** @type {Type} */(this.getView())`
7. **Destroy dialogs** - `afterClose: () => oDialog.destroy()`
8. **Defensive arrays** - `Array.isArray(aData) && aData.length !== 0`
9. **Always unsubscribe** - In `onExit()` to prevent memory leaks
10. **Use delegates** - For complex data management patterns
