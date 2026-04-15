# Complete POD 2.0 Widget Patterns

This reference provides complete, production-ready widget patterns you can copy and customize.

---

## Table of Contents

1. [Minimal Widget](#minimal-widget)
2. [Widget with PodContext Subscription](#widget-with-podcontext-subscription)
3. [Widget with Custom Properties](#widget-with-custom-properties)
4. [Widget with API Calls](#widget-with-api-calls)
5. [TableWidget Complete](#tablewidget-complete)
6. [ControlWidget Complete](#controlwidget-complete)
7. [LayoutWidget Complete](#layoutwidget-complete)
8. [ContentHandler Complete](#contenthandler-complete)

---

## Minimal Widget

**Use when**: You need the simplest possible widget to get started.

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/m/VBox",
    "sap/m/Text"
], (Widget, WidgetCategory, VBox, Text) => {
    "use strict";

    class MinimalWidget extends Widget {

        static getDisplayName() {
            return "Minimal Widget";
        }

        static getIcon() {
            return "sap-icon://hello-world";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        _createView() {
            const oConfig = this.getConfig();

            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [new Text({ text: "Configuration error" })]
                });
            }

            return new VBox(oConfig.id, {
                items: [
                    new Text({ text: "Hello from POD 2.0!" })
                ]
            });
        }
    }

    return MinimalWidget;
});
```

---

## Widget with PodContext Subscription

**Use when**: You need to react to POD context changes (resource selection, work list selection, etc.).

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/Logger",
    "sap/m/VBox",
    "sap/m/Text",
    "sap/m/Label"
], (Widget, PodContext, ModelPath, WidgetCategory, Logger, VBox, Text, Label) => {
    "use strict";

    class ContextAwareWidget extends Widget {
        #oLog = Logger.getLogger("custom.ContextAwareWidget");

        static getDisplayName() {
            return "Context Aware Widget";
        }

        static getIcon() {
            return "sap-icon://synchronize";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        onInit() {
            super.onInit();

            // Only subscribe in run mode (not design mode)
            if (PodContext.isRunMode()) {
                this.#oLog.info("Subscribing to context changes");
                PodContext.subscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }
        }

        _createView() {
            const oConfig = this.getConfig();

            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [new Text({ text: "Configuration error" })]
                });
            }

            // Store control references for updates
            this._oPlantText = new Text({
                text: PodContext.getPlant() || "Unknown"
            });

            this._oResourceText = new Text({
                text: "No resource selected"
            });

            return new VBox(oConfig.id, {
                items: [
                    new Label({ text: "Plant:" }),
                    this._oPlantText,
                    new Label({ text: "Resource:" }),
                    this._oResourceText
                ]
            });
        }

        /**
         * CRITICAL: Callback signature is (newValue, path) NOT (path, newValue)!
         * @param {Array} aResources - The new resources (FIRST parameter)
         * @param {string} sPath - The model path (SECOND parameter)
         */
        _onResourceChanged(aResources, sPath) {
            this.#oLog.debug("Resource changed", { aResources, sPath });

            // ALWAYS validate data types!
            const resources = Array.isArray(aResources) ? aResources : [];

            this._updateResourceDisplay(resources);
        }

        _updateResourceDisplay(aResources) {
            if (!this._oResourceText) {
                return; // Widget not yet created or already destroyed
            }

            // Double-check type and use optional chaining
            const sText = Array.isArray(aResources) && aResources.length > 0
                ? aResources.map(r => r?.resource || "Unknown").join(", ")
                : "No resource selected";

            this._oResourceText.setText(sText);
        }

        onExit() {
            super.onExit();

            // ALWAYS unsubscribe to prevent memory leaks
            if (PodContext.isRunMode()) {
                this.#oLog.info("Unsubscribing from context changes");
                PodContext.unsubscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }

            // Clean up control references
            this._oPlantText = null;
            this._oResourceText = null;
        }
    }

    return ContextAwareWidget;
});
```

---

## Widget with Custom Properties

**Use when**: You need configurable widget properties in POD Designer.

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/widget/metadata/WidgetProperty",
    "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor",
    "sap/dm/dme/pod2/propertyeditor/IntegerPropertyEditor",
    "sap/dm/dme/pod2/propertyeditor/BooleanPropertyEditor",
    "sap/dm/dme/pod2/propertyeditor/SelectPropertyEditor",
    "sap/m/VBox",
    "sap/m/Text"
], (
    Widget,
    WidgetCategory,
    WidgetProperty,
    StringPropertyEditor,
    IntegerPropertyEditor,
    BooleanPropertyEditor,
    SelectPropertyEditor,
    VBox,
    Text
) => {
    "use strict";

    class ConfigurableWidget extends Widget {

        // Define property IDs as frozen object for type safety
        static PropertyId = Object.freeze({
            Title: "title",
            MaxItems: "maxItems",
            ShowHeader: "showHeader",
            DisplayMode: "displayMode"
        });

        static getDisplayName() {
            return "Configurable Widget";
        }

        static getIcon() {
            return "sap-icon://settings";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        static getDefaultConfig() {
            const { PropertyId } = this;
            return {
                properties: {
                    [PropertyId.Title]: "My Widget",
                    [PropertyId.MaxItems]: 10,
                    [PropertyId.ShowHeader]: true,
                    [PropertyId.DisplayMode]: "compact"
                }
            };
        }

        /**
         * Define configurable properties for POD Designer
         */
        getProperties() {
            const { PropertyId } = this.constructor;

            return [
                new WidgetProperty({
                    displayName: "Widget Title",
                    description: "Title displayed at the top of the widget",
                    category: "Main",
                    propertyEditor: new StringPropertyEditor(
                        this,
                        PropertyId.Title,
                        "My Widget"  // default value
                    )
                }),
                new WidgetProperty({
                    displayName: "Maximum Items",
                    description: "Maximum number of items to display",
                    category: "Main",
                    propertyEditor: new IntegerPropertyEditor(
                        this,
                        PropertyId.MaxItems,
                        10  // default value
                    )
                }),
                new WidgetProperty({
                    displayName: "Show Header",
                    description: "Show or hide the widget header",
                    category: "Appearance",
                    propertyEditor: new BooleanPropertyEditor(
                        this,
                        PropertyId.ShowHeader,
                        true  // default value
                    )
                }),
                new WidgetProperty({
                    displayName: "Display Mode",
                    description: "How to display the items",
                    category: "Appearance",
                    propertyEditor: new SelectPropertyEditor(
                        this,
                        PropertyId.DisplayMode,
                        ["compact", "comfortable", "detailed"],  // options
                        "compact"  // default value
                    )
                })
            ];
        }

        _createView() {
            const oConfig = this.getConfig();

            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [new Text({ text: "Configuration error" })]
                });
            }

            // Access property values
            const sTitle = this.getPropertyValue(this.constructor.PropertyId.Title);
            const iMaxItems = this.getPropertyValue(this.constructor.PropertyId.MaxItems);
            const bShowHeader = this.getPropertyValue(this.constructor.PropertyId.ShowHeader);
            const sDisplayMode = this.getPropertyValue(this.constructor.PropertyId.DisplayMode);

            return new VBox(oConfig.id, {
                items: [
                    new Text({
                        text: `Title: ${sTitle || "Not set"}`,
                        visible: bShowHeader
                    }),
                    new Text({ text: `Max Items: ${iMaxItems}` }),
                    new Text({ text: `Display Mode: ${sDisplayMode}` })
                ]
            });
        }
    }

    return ConfigurableWidget;
});
```

---

## Widget with API Calls

**Use when**: You need to load data from custom APIs.

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/api/ApiClient",
    "sap/dm/dme/pod2/Logger",
    "sap/dm/dme/pod2/context/MessageHistory",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/m/VBox",
    "sap/m/Text",
    "sap/m/Button",
    "sap/m/BusyIndicator"
], (
    Widget,
    PodContext,
    ApiClient,
    Logger,
    MessageHistory,
    WidgetCategory,
    VBox,
    Text,
    Button,
    BusyIndicator
) => {
    "use strict";

    const REQUEST_TIMEOUT_MS = 30000;

    class ApiWidget extends Widget {
        #oLog = Logger.getLogger("custom.ApiWidget");

        static getDisplayName() {
            return "API Widget";
        }

        static getIcon() {
            return "sap-icon://chain-link";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        _createView() {
            const oConfig = this.getConfig();

            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [new Text({ text: "Configuration error" })]
                });
            }

            this._oDataText = new Text({
                text: "No data loaded"
            });

            this._oBusyIndicator = new BusyIndicator({
                size: "1rem",
                visible: false
            });

            return new VBox(oConfig.id, {
                items: [
                    new Button({
                        text: "Load Data",
                        press: () => this._onLoadData()
                    }),
                    this._oBusyIndicator,
                    this._oDataText
                ]
            });
        }

        async _onLoadData() {
            this.#oLog.info("Loading data...");

            this._setBusy(true);

            try {
                // Get current POD context
                const sPlant = PodContext.getPlant();
                const aResources = PodContext.get(ModelPath.FilterResources);
                const sResource = Array.isArray(aResources) && aResources.length > 0
                    ? aResources[0]?.resource
                    : null;

                if (!sResource) {
                    MessageHistory.showWarning("Please select a resource first");
                    return;
                }

                // Call custom API with timeout protection
                const oData = await Promise.race([
                    ApiClient.custom.post("/myEndpoint", {
                        plant: sPlant,
                        resource: sResource
                    }),
                    this._createTimeoutPromise(REQUEST_TIMEOUT_MS)
                ]);

                this.#oLog.info("Data loaded successfully", oData);

                // Update UI with data
                this._displayData(oData);

                // Show success message
                MessageHistory.toast({
                    message: "Data loaded successfully",
                    type: MessageHistory.Success
                });

            } catch (oError) {
                this.#oLog.error("Failed to load data", oError);

                const sMessage = oError.message === "Request timeout"
                    ? "Request timed out. Please try again."
                    : "Failed to load data. Please try again.";

                MessageHistory.showError(sMessage);

            } finally {
                this._setBusy(false);
            }
        }

        _displayData(oData) {
            if (!this._oDataText) {
                return;
            }

            // Safely access nested data
            const sStatus = oData?.status || "Unknown";
            const iCount = oData?.items?.length || 0;

            this._oDataText.setText(`Status: ${sStatus}, Count: ${iCount}`);
        }

        _setBusy(bBusy) {
            if (this._oBusyIndicator) {
                this._oBusyIndicator.setVisible(bBusy);
            }
        }

        _createTimeoutPromise(iTimeout) {
            return new Promise((_, reject) => {
                setTimeout(() => reject(new Error("Request timeout")), iTimeout);
            });
        }

        onExit() {
            super.onExit();
            this._oDataText = null;
            this._oBusyIndicator = null;
        }
    }

    return ApiWidget;
});
```

---

## More Complex Patterns

For complete TableWidget, ControlWidget, LayoutWidget, and ContentHandler patterns, see:
- TableWidget Pattern section in main SKILL.md (line ~1143)
- ControlWidget Pattern section (line ~991)
- LayoutWidget Pattern section (line ~1091)
- ContentHandler Pattern section (line ~1352)

Or refer to SAP's official samples:
https://github.com/SAP-samples/digital-manufacturing-extension-samples/tree/main/dm-podplugin-extensions/custom-pod2-examples
