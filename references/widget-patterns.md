# Widget Patterns Reference

Detailed patterns for ControlWidget, LayoutWidget, TableWidget, and ContentHandler in POD 2.0.

---

## ControlWidget Pattern (For Single Controls)

ControlWidget wraps single SAPUI5 controls. This is the most common pattern for simple widgets.

### Basic ControlWidget Pattern

```javascript
sap.ui.define([
    "sap/m/Button",
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/propertyeditor/PropertyCategory"
], (Button, ControlWidget, WidgetCategory, PropertyCategory) => {
    "use strict";

    /**
     * @alias sap.dm.dme.pod2.widget.core.ButtonWidget
     * @extends sap.dm.dme.pod2.widget.ControlWidget
     */
    class ButtonWidget extends ControlWidget {
        static getDisplayName() {
            return "Button Widget";
        }

        static getIcon() {
            return "sap-icon://iphone-2";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        static getDefaultConfig() {
            return {
                properties: {
                    text: this.getDisplayName(),
                    type: "Ghost"  // ButtonType.Ghost
                }
            };
        }

        // Static configuration arrays (optional but recommended)
        static BINDABLE_PROPERTIES = ["text", "enabled", "visible"];
        static INCLUDE_EVENTS = ["press"];
        static EXCLUDE_PROPERTIES = ["somePropertyToHide"];
        static PROPERTY_CATEGORY_OVERRIDE = {
            text: PropertyCategory.Main,
            icon: PropertyCategory.Appearance,
            width: PropertyCategory.Dimension
        };

        /**
         * Constructor - pass SAPUI5 control class to super
         * @param {Object} oConfig
         */
        constructor(oConfig) {
            super(Button, oConfig);  // Pass the control class!
        }

        /**
         * Override for custom initialization
         * @override
         */
        _createView() {
            // Add custom logic before/after
            const oControl = super._createView();
            // Additional setup...
            return oControl;
        }
    }

    return ButtonWidget;
});
```

### Common ControlWidget Types

**Text Display:**
- `TextWidget` - Simple text
- `LabelWidget` - Form labels
- `TitleWidget` - Section titles
- `ExpandableTextWidget` - Collapsible text

**Input:**
- `InputWidget` - Text input
- `TextAreaWidget` - Multi-line input
- `DateTimeTextWidget` - Date/time display

**Buttons:**
- `ButtonWidget` - Standard button
- `MenuButtonWidget` - Button with dropdown

**Visual:**
- `IconWidget` - SAP icons
- `ImageWidget` - Images
- `HTMLWidget` - HTML content
- `IFrameWidget` - Embedded content

---

## LayoutWidget Pattern (For Containers)

LayoutWidget wraps layout containers that hold other widgets.

```javascript
sap.ui.define([
    "sap/m/VBox",
    "sap/dm/dme/pod2/widget/LayoutWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory"
], (VBox, LayoutWidget, WidgetCategory) => {
    "use strict";

    class VBoxWidget extends LayoutWidget {
        static getDisplayName() {
            return "Vertical Box";
        }

        static getIcon() {
            return "sap-icon://vertical-grip";
        }

        static getCategory() {
            return WidgetCategory.Layout;
        }

        constructor(oConfig) {
            super(VBox, oConfig);  // Pass container class
        }
    }

    return VBoxWidget;
});
```

### Common LayoutWidget Types

- `VBoxWidget`, `HBoxWidget` - Basic boxes
- `FlexBoxWidget` - Flexible layout
- `PanelWidget` - Panel with header
- `DialogWidget` - Modal dialog (Hidden category)
- `ToolbarWidget` - Toolbar
- `SplitterWidget` - Resizable split panes
- `ResponsiveSplitterWidget` - Responsive split panes

---

## TableWidget Pattern (For Complex Tables)

TableWidget is the most complex base class, used for displaying tabular data with columns, sorting, and pagination.

### Complete TableWidget Example

```javascript
sap.ui.define([
    "sap/m/library",
    "sap/ui/core/library",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/widget/core/TableWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory"
], (
    SapMLibrary,
    SapUiCoreLibrary,
    ModelPath,
    PodContext,
    TableWidget,
    WidgetCategory
) => {
    "use strict";

    const { ListMode, Text, ObjectIdentifier } = SapMLibrary;
    const { Priority, TextAlign } = SapUiCoreLibrary;

    /**
     * @alias sap.dm.dme.pod2.widget.custom.MyTableWidget
     * @extends sap.dm.dme.pod2.widget.core.TableWidget
     */
    class MyTableWidget extends TableWidget {

        /**
         * Define field names as enum for type safety
         * @enum {string}
         */
        static Field = Object.freeze({
            SFC: "sfc",
            Material: "material",
            Quantity: "quantity",
            Status: "status"
        });

        static getDisplayName() {
            return "My Table Widget";
        }

        static getIcon() {
            return "sap-icon://table-view";
        }

        /**
         * Define available columns with metadata
         * @override
         * @returns {Array}
         */
        static getFields() {
            const { Field } = this;
            return [
                {
                    field: Field.SFC,
                    text: "{i18n>sfc}",           // i18n key
                    importance: Priority.High,
                    width: "150px",
                    sortable: true
                },
                {
                    field: Field.Material,
                    text: "{i18n>material}",
                    width: "180px",
                    sortable: true
                },
                {
                    field: Field.Quantity,
                    text: "{i18n>quantity}",
                    width: "80px",
                    sortable: true,
                    hAlign: TextAlign.End      // Right-align numbers
                },
                {
                    field: Field.Status,
                    text: "{i18n>status}",
                    width: "100px",
                    sortable: false
                }
            ];
        }

        /**
         * Define which fields to show by default
         * @override
         * @returns {Array<string>}
         */
        static getDefaultFields() {
            const { Field } = this;
            return [Field.SFC, Field.Material, Field.Quantity];
        }

        /**
         * Define default configuration
         * @override
         */
        static getDefaultConfig() {
            // CRITICAL: Defensive null check - base Widget returns null!
            const oParentConfig = super.getDefaultConfig();
            const oParentProperties = oParentConfig?.properties || {};

            return {
                properties: {
                    ...oParentProperties,
                    mode: ListMode.SingleSelectMaster,
                    growingScrollToLoad: true,
                    pageSize: 100,
                    defaultSorting: [{
                        sortBy: this.Field.SFC,
                        descending: false
                    }]
                }
            };
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        static EXCLUDE_PROPERTIES = [
            ...TableWidget.EXCLUDE_PROPERTIES,
            "headerText"  // Hide specific properties
        ];

        constructor(oConfig) {
            super(oConfig);
        }

        /**
         * REQUIRED: Specify model path for table data
         * @override
         * @returns {string}
         */
        _getModelPath() {
            return ModelPath.WorkListItems;  // Or custom path like "/materials"
        }

        /**
         * Optional: Specify count path for pagination
         * @override
         * @returns {string}
         */
        _getCountPath() {
            return ModelPath.WorkListCount;
        }

        /**
         * REQUIRED: Create cell controls for each column
         * @override
         * @param {Object} oColumnConfig
         * @returns {sap.ui.core.Control}
         */
        _createCell(oColumnConfig) {
            switch (oColumnConfig.field) {
                case MyTableWidget.Field.SFC:
                    // Clickable identifier
                    return this._createIdentifierCell(oColumnConfig, "sfc");

                case MyTableWidget.Field.Material:
                case MyTableWidget.Field.Quantity:
                    // Simple text
                    return this._createTextCell(oColumnConfig, oColumnConfig.field);

                case MyTableWidget.Field.Status:
                    // Custom control
                    return new Text({
                        text: "{statusCode}"
                    });

                default:
                    // Handle custom fields
                    if (oColumnConfig.field.startsWith("customFields/")) {
                        return this._createTextCell(oColumnConfig, oColumnConfig.field);
                    }
                    throw new Error(`Unsupported field: ${oColumnConfig.field}`);
            }
        }

        /**
         * Optional: Handle sorting changes
         * @override
         */
        _onSort(aSorting) {
            PodContext.setWorkListSorting(aSorting);
            // Trigger data refresh
        }
    }

    return MyTableWidget;
});
```

### TableWidget Helper Methods

TableWidget provides these helper methods for creating cells:

```javascript
// Text cell with binding
_createTextCell(oColumnConfig, vBindPath)

// Identifier cell (clickable object name)
_createIdentifierCell(oColumnConfig, vBindPath)

// Date cell with formatting
_createDateCell(oColumnConfig, vBinding, fnFormatter)

// Quantity bullet chart
_createQuantityBulletChartCell(oBindPaths)
```

---

## ContentHandler Pattern (Business Logic)

ContentHandlers encapsulate complex business logic without UI. Used for form processing, dialogs, and workflows.

```javascript
sap.ui.define([
    "sap/m/Dialog",
    "sap/m/Button",
    "sap/m/ButtonType",
    "sap/ui/model/json/JSONModel",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/api/ApiClient",
    "sap/dm/dme/pod2/Logger",
    "sap/dm/dme/pod2/context/MessageHistory"
], (Dialog, Button, ButtonType, JSONModel, PodContext, ApiClient, Logger, MessageHistory) => {
    "use strict";

    /**
     * @alias sap.dm.dme.pod2.widget.custom.MyContentHandler
     */
    class MyContentHandler {
        #oModel = new JSONModel();
        #oDialog;
        #oLog = Logger.getLogger("sap.dm.dme.pod2.widget.custom.MyContentHandler");

        /**
         * Opens the content as a dialog
         * @param {Object} oData
         */
        async openAsDialog(oData) {
            this.#oModel.setData(oData);

            const oDialog = new Dialog({
                title: "My Dialog",
                content: [this._createForm()],
                buttons: [
                    new Button({
                        text: "Confirm",
                        type: ButtonType.Emphasized,
                        press: () => this._onConfirm()
                    }),
                    new Button({
                        text: "Cancel",
                        press: () => oDialog.close()
                    })
                ],
                afterClose: () => oDialog.destroy()
            });

            this.#oDialog = oDialog;
            oDialog.setModel(this.#oModel);
            oDialog.open();
        }

        _createForm() {
            // Create form controls
        }

        async _onConfirm() {
            const oData = this.#oModel.getData();

            try {
                await ApiClient.custom.post("/myEndpoint", oData);
                MessageHistory.toast({
                    message: "Success",
                    type: MessageHistory.Success
                });
            } catch (oError) {
                this.#oLog.error("Error", oError);
                MessageHistory.showError("Operation failed");
            }

            this.#oDialog.close();
        }
    }

    return MyContentHandler;
});
```

---

## Widget Categories

Available categories from `WidgetCategory`:
- `Elements` - Basic UI elements (buttons, inputs, text)
- `Layout` - Layout containers (panels, boxes, dialogs)
- `WorkList` - Work list related widgets
- `Order` - Order-related widgets
- `SFC` - Shop Floor Control widgets
- `DataCollection` - Data collection widgets
- `QuantityConfirmation` - Quantity reporting
- `ActivityConfirmation` - Activity confirmation
- `Assembly` - Assembly/component widgets
- `GoodsReceipt` - Goods receipt widgets
- `Hidden` - Not shown in widget palette (for DialogWidget, etc.)

---

## Common Patterns

### Quick Reference - Copy & Paste Examples

**Pattern 1: Display Current Resource**
```javascript
_createView() {
    this._oText = new Text({ text: "No resource" });
    return new VBox(this.getConfig().id, { items: [this._oText] });
}

onInit() {
    super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(ModelPath.FilterResources, (aRes, sPath) => {
            const resources = Array.isArray(aRes) ? aRes : [];
            this._oText.setText(resources[0]?.resource || "None");
        }, this);
    }
}
```

**Pattern 2: Button that Calls Custom API**
```javascript
_createView() {
    return new VBox(this.getConfig().id, {
        items: [
            new Button({
                text: "Load Data",
                press: () => this._onLoadData()
            })
        ]
    });
}

async _onLoadData() {
    try {
        const oData = await ApiClient.custom.post("/myEndpoint", {
            plant: PodContext.getPlant(),
            resource: PodContext.get(ModelPath.FilterResources)?.[0]?.resource
        });
        MessageHistory.toast({ message: "Success", type: MessageHistory.Success });
    } catch (oError) {
        Logger.error("Load failed", oError);
        MessageHistory.showError("Failed to load data");
    }
}
```

**Pattern 3: Simple Table Display**
```javascript
_createView() {
    this._oTable = new Table({
        columns: [
            new Column({ header: new Label({ text: "Item" }) }),
            new Column({ header: new Label({ text: "Status" }) })
        ]
    });
    return new VBox(this.getConfig().id, { items: [this._oTable] });
}

_updateTable(aItems) {
    this._oTable.removeAllItems();
    aItems.forEach(item => {
        this._oTable.addItem(new ColumnListItem({
            cells: [
                new Text({ text: item.name }),
                new Text({ text: item.status })
            ]
        }));
    });
}
```

**Pattern 4: Read Custom Property**
```javascript
static PropertyId = Object.freeze({
    ApiEndpoint: "apiEndpoint",
    RefreshInterval: "refreshInterval"
});

getProperties() {
    return [
        new WidgetProperty({
            displayName: "API Endpoint",
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "apiEndpoint", "/default")
        }),
        new WidgetProperty({
            displayName: "Refresh Interval (seconds)",
            category: "Main",
            propertyEditor: new IntegerPropertyEditor(this, "refreshInterval", 30)
        })
    ];
}

_someMethod() {
    const sEndpoint = this.getPropertyValue("apiEndpoint");
    const iInterval = this.getPropertyValue("refreshInterval");
    // Use values...
}
```

**Pattern 5: Subscribe to Work List Selection**
```javascript
onInit() {
    super.onInit();
    if (PodContext.isRunMode()) {
        PodContext.subscribe(ModelPath.SelectedWorkListItems, (aItems, sPath) => {
            const items = Array.isArray(aItems) ? aItems : [];
            if (items.length > 0) {
                this._handleSelection(items[0]);
            }
        }, this);
    }
}

_handleSelection(oItem) {
    const sSfc = oItem?.sfc;
    const sOrder = oItem?.order;
    const sMaterial = oItem?.material;
    // Use data...
}
```

---

### Basic Widget Example (POD 2.0)

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/m/VBox",
    "sap/m/Text",
    "sap/m/Button",
    "sap/m/MessageToast"
], (Widget, PodContext, ModelPath, VBox, Text, Button, MessageToast) => {
    "use strict";

    class BasicPlugin extends Widget {

        static getDisplayName() {
            return "Basic Plugin";
        }

        static getIcon() {
            return "sap-icon://factory";
        }

        static getCategory() {
            return "Custom Widgets";
        }

        static getDescription() {
            return "Basic example plugin with event subscription";
        }

        onInit() {
            super.onInit();

            // Subscribe to resource changes
            if (PodContext.isRunMode()) {
                PodContext.subscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }
        }

        _createView() {
            const oConfig = this.getConfig();

            // Validate config
            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [new Text({ text: "Configuration error" })]
                });
            }

            // Store control reference for updates
            this._oResourceText = new Text({
                text: "No resources selected"
            });

            // Pass oConfig.id as first parameter
            return new VBox(oConfig.id, {
                items: [
                    new Text({ text: "Hello from POD 2.0" }),
                    this._oResourceText,
                    new Button({
                        text: "Show Plant",
                        press: () => this._onButtonPress()
                    })
                ]
            });
        }

        // CRITICAL: Callback signature is (newValue, path) NOT (path, newValue)!
        _onResourceChanged(aResources, sPath) {
            // Coerce to array to prevent crashes
            const resources = Array.isArray(aResources) ? aResources : [];
            this._updateResourceDisplay(resources);
        }

        _updateResourceDisplay(aResources) {
            if (this._oResourceText) {
                // Double-check type and use optional chaining
                const sText = Array.isArray(aResources) && aResources.length > 0
                    ? aResources.map(r => r?.resource || "Unknown").join(", ")
                    : "No resources selected";
                this._oResourceText.setText(sText);
            }
        }

        _onButtonPress() {
            const sPlant = PodContext.getPlant();
            MessageToast.show(`Plant: ${sPlant}`);
        }

        onExit() {
            super.onExit();

            // Always unsubscribe to prevent memory leaks
            if (PodContext.isRunMode()) {
                PodContext.unsubscribe(
                    ModelPath.FilterResources,
                    this._onResourceChanged,
                    this
                );
            }

            // Clean up control references
            this._oResourceText = null;
        }
    }

    return BasicPlugin;
});
```

---

## i18n (Internationalization) Pattern

POD 2.0 widgets require manual resource bundle loading for internationalization.

**CRITICAL**: `getResourceBundle()` is NOT available on Widget base classes!

### Complete i18n Implementation

```javascript
sap.ui.define([
    "sap/m/Button",
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/widget/metadata/WidgetProperty",
    "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor",
    "sap/base/i18n/ResourceBundle",  // ✅ REQUIRED for i18n
    "sap/m/VBox",
    "sap/m/Text"
], (
    Button,
    ControlWidget,
    WidgetCategory,
    WidgetProperty,
    StringPropertyEditor,
    ResourceBundle,  // ✅ Import ResourceBundle
    VBox,
    Text
) => {
    "use strict";

    /**
     * Example widget with proper i18n implementation
     * @alias mycompany.widget.I18nExampleWidget
     * @extends sap.dm.dme.pod2.widget.ControlWidget
     */
    class I18nExampleWidget extends ControlWidget {
        #oResourceBundle = null;  // ✅ Store resource bundle instance

        static getDisplayName() {
            return "i18n Example Widget";
        }

        static getIcon() {
            return "sap-icon://globe";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        static getDefaultConfig() {
            return {
                properties: {
                    customMessage: "Hello World"
                }
            };
        }

        constructor(oConfig) {
            super(Button, oConfig);
        }

        /**
         * STEP 1: Load resource bundle in onInit()
         * @override
         */
        async onInit() {
            await super.onInit();

            // Load i18n resource bundle
            try {
                // Get module path for this widget
                const sModulePath = sap.ui.require.toUrl("mycompany/widget");
                
                // Create resource bundle from i18n.properties file
                this.#oResourceBundle = await ResourceBundle.create({
                    url: `${sModulePath}/i18n/i18n.properties`,
                    async: true
                });
                
                console.log("Resource bundle loaded successfully");
            } catch (error) {
                console.error("Failed to load resource bundle:", error);
                // Widget continues to work with fallback values
            }

            // Continue with other initialization...
        }

        /**
         * STEP 2: Use resource bundle in getProperties()
         * @override
         */
        getProperties() {
            return [
                new WidgetProperty({
                    displayName: this._getI18nText("property.customMessage"),  // ✅ Method call, not binding!
                    description: this._getI18nText("property.customMessage.description"),
                    category: "Main",
                    propertyEditor: new StringPropertyEditor(this, "customMessage")
                })
            ];
        }

        /**
         * STEP 3: Create helper method for text retrieval
         * Includes fallback values in case bundle fails to load
         */
        _getI18nText(sKey, aParams) {
            // Try to get text from loaded bundle
            if (this.#oResourceBundle) {
                const sText = this.#oResourceBundle.getText(sKey, aParams);
                if (sText) {
                    return sText;
                }
            }

            // Fallback to hardcoded defaults
            const fallbacks = {
                "property.customMessage": "Custom Message",
                "property.customMessage.description": "Enter a custom message to display",
                "button.text": "Click Me",
                "button.text.description": "Button text",
                "message.success": "Operation completed successfully",
                "message.error": "An error occurred",
                "label.placeholder": "Enter text here"
            };

            return fallbacks[sKey] || sKey;  // Return key if no fallback
        }

        /**
         * Use i18n text in UI creation
         * @override
         */
        _createView() {
            const oConfig = this.getConfig();

            if (!oConfig || !oConfig.id) {
                return new VBox({
                    items: [
                        new Text({ text: this._getI18nText("message.error") })
                    ]
                });
            }

            // Create button with i18n text
            const oButton = new Button(oConfig.id, {
                text: this._getI18nText("button.text"),
                press: () => this._onButtonPress()
            });

            return oButton;
        }

        _onButtonPress() {
            const sMessage = this._getI18nText("message.success");
            sap.m.MessageToast.show(sMessage);
        }

        /**
         * STEP 4: Clean up in onExit()
         * @override
         */
        onExit() {
            super.onExit();
            this.#oResourceBundle = null;  // ✅ Clean up reference
        }
    }

    return I18nExampleWidget;
});
```

### File Structure

**Required folder structure:**

```
mycompany/
├── extension.json
├── widget/
│   ├── I18nExampleWidget.js
│   └── i18n/                      # ✅ i18n folder inside widget folder
│       ├── i18n.properties        # Default (English)
│       ├── i18n_en.properties     # English (explicit)
│       ├── i18n_de.properties     # German
│       ├── i18n_fr.properties     # French
│       └── i18n_es.properties     # Spanish
└── util/
```

**Alternative structure (i18n at namespace root):**

```
mycompany/
├── extension.json
├── i18n/                          # ✅ i18n folder at root
│   ├── i18n.properties
│   ├── i18n_de.properties
│   └── i18n_fr.properties
└── widget/
    └── I18nExampleWidget.js
```

**Adjust module path accordingly:**

```javascript
// For widget/i18n/ structure:
const sModulePath = sap.ui.require.toUrl("mycompany/widget");
this.#oResourceBundle = await ResourceBundle.create({
    url: `${sModulePath}/i18n/i18n.properties`,
    async: true
});

// For root i18n/ structure:
const sModulePath = sap.ui.require.toUrl("mycompany");
this.#oResourceBundle = await ResourceBundle.create({
    url: `${sModulePath}/i18n/i18n.properties`,
    async: true
});
```

### i18n.properties File Example

**i18n.properties** (Default - English):

```properties
# Widget Display
widget.title=My Widget
widget.description=Example widget with internationalization

# Properties
property.customMessage=Custom Message
property.customMessage.description=Enter a custom message to display
property.apiEndpoint=API Endpoint
property.apiEndpoint.description=Enter the API endpoint URL

# Button Labels
button.text=Click Me
button.save=Save
button.cancel=Cancel
button.delete=Delete
button.refresh=Refresh

# Messages
message.success=Operation completed successfully
message.error=An error occurred
message.loading=Loading data...
message.noData=No data available

# Labels
label.placeholder=Enter text here
label.required=Required field
label.optional=Optional
```

**i18n_de.properties** (German):

```properties
# Widget Display
widget.title=Mein Widget
widget.description=Beispiel-Widget mit Internationalisierung

# Properties
property.customMessage=Benutzerdefinierte Nachricht
property.customMessage.description=Geben Sie eine benutzerdefinierte Nachricht ein
property.apiEndpoint=API-Endpunkt
property.apiEndpoint.description=Geben Sie die API-Endpunkt-URL ein

# Button Labels
button.text=Klicke mich
button.save=Speichern
button.cancel=Abbrechen
button.delete=Löschen
button.refresh=Aktualisieren

# Messages
message.success=Vorgang erfolgreich abgeschlossen
message.error=Ein Fehler ist aufgetreten
message.loading=Daten werden geladen...
message.noData=Keine Daten verfügbar

# Labels
label.placeholder=Text hier eingeben
label.required=Pflichtfeld
label.optional=Optional
```

### Advanced: i18n with Parameters

Use `getText()` with parameters for dynamic values:

```javascript
// In i18n.properties:
// message.itemsSelected={0} item(s) selected

_showSelectionCount(iCount) {
    const sMessage = this._getI18nText("message.itemsSelected", [iCount]);
    // Result: "5 item(s) selected"
    sap.m.MessageToast.show(sMessage);
}
```

### Advanced: Locale Detection

ResourceBundle automatically detects user locale from browser settings. The loading order is:

1. `i18n_de_DE.properties` (exact locale match)
2. `i18n_de.properties` (language match)
3. `i18n.properties` (fallback/default)

No additional configuration needed - SAPUI5 handles this automatically!

### Common Patterns

**Pattern 1: Using i18n in WidgetProperty (CORRECT)**

```javascript
getProperties() {
    return [
        new WidgetProperty({
            displayName: this._getI18nText("property.apiEndpoint"),  // ✅ Method call!
            description: this._getI18nText("property.apiEndpoint.description"),
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "apiEndpoint")
        })
    ];
}
```

**Pattern 2: Using i18n in Control Creation**

```javascript
_createView() {
    return new VBox(this.getConfig().id, {
        items: [
            new Label({ text: this._getI18nText("label.username") }),
            new Input({ placeholder: this._getI18nText("label.placeholder") }),
            new Button({
                text: this._getI18nText("button.save"),
                press: () => this._onSave()
            })
        ]
    });
}
```

**Pattern 3: Dynamic Messages**

```javascript
async _loadData() {
    try {
        const oData = await ApiClient.custom.get("/data");
        const sMessage = this._getI18nText("message.success");
        sap.m.MessageToast.show(sMessage);
    } catch (error) {
        const sError = this._getI18nText("message.error");
        sap.m.MessageBox.error(sError);
    }
}
```

### Critical Rules

✅ **DO:**
- Load ResourceBundle manually in `onInit()`
- Use `sap.ui.require.toUrl()` to get module path
- Store bundle in instance variable
- Provide fallback defaults
- Clean up in `onExit()`
- Use method calls in `getProperties()`: `this._getI18nText("key")`

❌ **DON'T:**
- Assume `getResourceBundle()` exists
- Use binding syntax in `WidgetProperty`: `"{i18n>key}"`
- Load bundle synchronously
- Forget fallback values
- Hard-code user-visible text

### Troubleshooting

**Issue**: "TypeError: this.getResourceBundle is not a function"
**Solution**: Widget classes don't have this method. Load ResourceBundle manually.

**Issue**: Resource bundle not found (404 error)
**Solution**: Check module path with `console.log(sap.ui.require.toUrl("mycompany/widget"))`. Verify i18n folder location.

**Issue**: Binding syntax errors in property editor
**Solution**: Never use `"{i18n>key}"` in WidgetProperty. Use `this._getI18nText("key")` instead.

**Issue**: Text shows as key (e.g., "button.text" instead of "Click Me")
**Solution**: Check fallback object includes the key, or verify i18n.properties file is loaded correctly.

### See Also

- [Mistake #2: Using Binding Syntax in WidgetProperty](common-mistakes.md#mistake-2-using-binding-syntax-in-widgetproperty-metadata)
- [Mistake #12: Assuming getResourceBundle() Exists](common-mistakes.md#mistake-12-assuming-getresourcebundle-exists-on-widget)
- **SAPUI5 Documentation**: ResourceBundle API Reference

---

## Alternative: ResourceModel-Based i18n Pattern (Synchronous)

### Overview

An alternative to the async `ResourceBundle.create()` approach is using `ResourceModel`, which loads synchronously in the constructor. This approach is simpler but doesn't use async/await.

### When to Use This Approach

- ✅ Simple widgets that don't need async initialization
- ✅ Want i18n available immediately in constructor
- ✅ Prefer synchronous loading pattern
- ❌ Avoid if you need fine-grained control over bundle loading timing

### Complete Implementation

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/m/Button",
    "sap/m/MessageToast",
    "sap/ui/model/resource/ResourceModel"  // ← Use ResourceModel instead
], (ControlWidget, WidgetCategory, Button, MessageToast, ResourceModel) => {
    "use strict";

    class MultilingualWidget extends ControlWidget {
        constructor(oConfig) {
            super(Button, oConfig);
            this._oResourceBundle = null;
            this._loadI18n();  // ✅ Load synchronously in constructor
        }

        static getDisplayName() {
            return "Multilingual Widget";
        }

        static getIcon() {
            return "sap-icon://world";
        }

        static getCategory() {
            return WidgetCategory.Elements;
        }

        /**
         * Load i18n using ResourceModel (synchronous)
         */
        _loadI18n() {
            // Replace with your actual namespace
            const sModulePath = "custom/pod2/yournamespace";
            const oResourceModel = new ResourceModel({
                bundleName: sModulePath + ".i18n.i18n"  // ← Dot notation!
            });
            this._oResourceBundle = oResourceModel.getResourceBundle();
        }

        /**
         * Get translated text with optional parameters
         * @param {string} sKey - i18n key
         * @param {Array} aParams - Optional parameters for placeholders
         * @returns {string} Translated text or key as fallback
         */
        _getI18nText(sKey, aParams) {
            if (!this._oResourceBundle) {
                return sKey;  // Fallback to key if bundle not loaded
            }
            return this._oResourceBundle.getText(sKey, aParams);
        }

        _createView() {
            const oConfig = this.getConfig();
            return new Button(oConfig.id, {
                text: this._getI18nText("button.submit"),  // ✅ Available immediately!
                press: () => this._onPress()
            });
        }

        _onPress() {
            const sMessage = this._getI18nText("message.success");
            MessageToast.show(sMessage);
        }

        onExit() {
            super.onExit();
            this._oResourceBundle = null;
        }
    }

    return MultilingualWidget;
});
```

### Key Differences: ResourceBundle vs ResourceModel

| Feature | ResourceBundle.create() | ResourceModel |
|---------|------------------------|---------------|
| Loading | Async (await in onInit) | Sync (in constructor) |
| Import | `sap/base/i18n/ResourceBundle` | `sap/ui/model/resource/ResourceModel` |
| Bundle Name | URL path: `"${path}/i18n/i18n.properties"` | Dot notation: `"namespace.i18n.i18n"` |
| Available When | After onInit() completes | Immediately in constructor |
| Best For | Complex widgets with async setup | Simple widgets, immediate use |

### Namespace Convention (CRITICAL!)

The `bundleName` must match your namespace structure:

```javascript
// If namespace is: custom/pod2/myproject
// Then bundleName is: "custom/pod2/myproject.i18n.i18n"
//                      ^^^^ Slashes become dots ^^^^

const oResourceModel = new ResourceModel({
    bundleName: "custom/pod2/myproject.i18n.i18n"
    //          ^namespace^  ^folder^ ^basename^
});
```

### File Structure (Same for Both Approaches)

```
<namespace-folder>/
├── extension.json
├── widget/
│   └── MyWidget.js
└── i18n/
    ├── i18n.properties        (fallback/default)
    ├── i18n_en.properties     (English)
    ├── i18n_de.properties     (German)
    └── i18n_fr.properties     (French)
```

### Example: Dynamic Messages with Parameters

```javascript
// In i18n/i18n.properties:
// message.itemsSelected=You have selected {0} items
// message.error=Error {0}: {1}

_showSelection(iCount) {
    const sMsg = this._getI18nText("message.itemsSelected", [5]);
    // Result: "You have selected 5 items"
    MessageToast.show(sMsg);
}

_showError(sCode, sMessage) {
    const sError = this._getI18nText("message.error", ["404", "Not Found"]);
    // Result: "Error 404: Not Found"
    MessageToast.show(sError);
}
```

### Common Mistakes with ResourceModel Approach

❌ **WRONG: Using slash notation for bundleName**

```javascript
// ❌ This will fail!
new ResourceModel({
    bundleName: "custom/pod2/myproject/i18n/i18n"  // Slashes don't work!
});
```

❌ **WRONG: Using URL path**

```javascript
// ❌ ResourceModel doesn't accept URLs!
new ResourceModel({
    bundleName: `${sap.ui.require.toUrl("custom/pod2/myproject")}/i18n/i18n.properties`
});
```

✅ **CORRECT: Dot notation for namespace**

```javascript
// ✅ Convert slashes to dots!
new ResourceModel({
    bundleName: "custom.pod2.myproject.i18n.i18n"  // Dots work!
});
```

### Comparison Summary

**Use ResourceBundle.create() (Async) when:**
- You need async initialization
- You prefer explicit URL control
- You want to handle loading errors with try/catch in onInit()
- You're loading from non-standard locations

**Use ResourceModel (Sync) when:**
- You want simplest possible implementation
- You need i18n available in constructor/before onInit()
- Your i18n follows standard structure
- You prefer synchronous loading

Both approaches work correctly - choose based on your widget's needs!

---

## Navigation

📖 **Back to main skill**: [SKILL.md](../SKILL.md)

**Other references**:
- [Common Mistakes](common-mistakes.md) - All mistakes with fixes
- [Glossary](glossary.md) - Key terms & definitions
