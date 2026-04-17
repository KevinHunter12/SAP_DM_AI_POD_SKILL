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

POD 2.0 framework automatically loads i18n models - widgets provide static `getI18nModel()` and use inherited methods.

**CRITICAL**: Do NOT manually load ResourceBundle/ResourceModel - framework handles this!

### Complete i18n Implementation

**Pattern from SAP Production Code:**

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/ControlWidget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/widget/metadata/WidgetProperty",
    "sap/dm/dme/pod2/propertyeditor/StringPropertyEditor",
    "sap/dm/dme/pod2/model/I18nResourceModel",  // ← Framework i18n model
    "sap/dm/dme/pod2/context/PodContext",
    "sap/m/Button",
    "sap/m/VBox",
    "sap/m/Label",
    "sap/m/MessageToast"
], (
    ControlWidget,
    WidgetCategory,
    WidgetProperty,
    StringPropertyEditor,
    I18nResourceModel,
    PodContext,
    Button,
    VBox,
    Label,
    MessageToast
) => {
    "use strict";

    /**
     * Example widget with framework-driven i18n
     * @alias mycompany.widget.I18nExampleWidget
     * @extends sap.dm.dme.pod2.widget.ControlWidget
     */
    class I18nExampleWidget extends ControlWidget {

        // ✅ STEP 1: Define static private i18n model
        static #oI18nModel = new I18nResourceModel({
            bundleName: "mycompany.widget.i18n.i18n"  // Namespace with dots!
        });

        // ✅ STEP 2: Static getter for framework
        // Framework calls this during widget registration and loads the model
        static getI18nModel() {
            return this.#oI18nModel;
        }

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
            super(VBox, oConfig);
        }

        async onInit() {
            await super.onInit();
            // No manual i18n loading needed - framework already loaded it!
            console.log("Widget initialized - i18n ready");
        }

        // ✅ STEP 3: Use inherited getI18nText() method
        getProperties() {
            return [
                new WidgetProperty({
                    // Use inherited method - no manual loading!
                    displayName: this.getI18nText("property.customMessage"),
                    description: this.getI18nText("property.customMessage.description"),
                    category: "Main",
                    propertyEditor: new StringPropertyEditor(this, "customMessage")
                })
            ];
        }

        // ✅ STEP 4: Use inherited getI18nText() in UI creation
        _createView() {
            const oConfig = this.getConfig();

            return new VBox(oConfig.id, {
                items: [
                    // Method 1: Use inherited getI18nText()
                    new Label({ 
                        text: this.getI18nText("label.welcome") 
                    }),
                    
                    // Method 2: Use binding syntax for controls
                    new Button({
                        text: "{i18n>button.submit}",
                        press: () => this._onButtonPress()
                    }),
                    
                    // Method 3: With parameters
                    new Label({ 
                        text: this.getI18nText("label.itemCount", [5]) 
                    })
                ]
            });
        }

        _onButtonPress() {
            // Use inherited method
            const sSuccess = this.getI18nText("message.success");
            MessageToast.show(sSuccess);
            
            // Alternative: Use PodContext static method
            const sAlternative = PodContext.getI18nText("message.alternative");
            console.log(sAlternative);
        }
    }

    return I18nExampleWidget;
});
```

### Three Ways to Use i18n

POD 2.0 provides three methods for using internationalized text:

#### Method 1: Inherited `this.getI18nText()` (Recommended)

```javascript
// Available anywhere in widget instance
const sText = this.getI18nText("myWidget.greeting");
const sWithParams = this.getI18nText("myWidget.title", [arg1, arg2]);
```

**Use when:** You're inside widget instance methods

#### Method 2: Static `PodContext.getI18nText()`

```javascript
// Static method, works anywhere
const sText = PodContext.getI18nText("myWidget.error");
const sWithParams = PodContext.getI18nText("myWidget.message", [count]);
```

**Use when:** You need i18n in static methods or outside widget context

#### Method 3: Binding Syntax `"{i18n>key}"`

```javascript
// In control properties
new Button({
    text: "{i18n>myWidget.button.label}",
    tooltip: "{i18n>myWidget.button.tooltip}"
});
```

**Use when:** Creating SAPUI5 controls with property bindings

### File Structure

**Required folder structure:**

```
mycompany/                        # Your namespace folder
├── extension.json
├── widget/
│   └── I18nExampleWidget.js
└── i18n/                         # ✅ i18n folder at namespace root
    ├── i18n.properties           # Default (English)
    ├── i18n_en.properties        # English (explicit)
    ├── i18n_de.properties        # German
    └── i18n_fr.properties        # French
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

# Labels
label.welcome=Welcome to the Widget
label.itemCount=You have {0} items

# Button Labels
button.submit=Submit
button.cancel=Cancel
button.delete=Delete

# Messages
message.success=Operation completed successfully
message.error=An error occurred
message.alternative=Alternative message text
```

**i18n_de.properties** (German):

```properties
# Widget Display
widget.title=Mein Widget
widget.description=Beispiel-Widget mit Internationalisierung

# Properties
property.customMessage=Benutzerdefinierte Nachricht
property.customMessage.description=Geben Sie eine benutzerdefinierte Nachricht ein

# Labels
label.welcome=Willkommen zum Widget
label.itemCount=Sie haben {0} Elemente

# Button Labels
button.submit=Absenden
button.cancel=Abbrechen
button.delete=Löschen

# Messages
message.success=Vorgang erfolgreich abgeschlossen
message.error=Ein Fehler ist aufgetreten
message.alternative=Alternativer Meldungstext
```

### Parameters in i18n Text

Use `{0}`, `{1}`, etc. for dynamic values:

```javascript
// In i18n.properties:
// message.itemsSelected={0} item(s) selected out of {1} total

_showSelectionCount(iSelected, iTotal) {
    const sMessage = this.getI18nText("message.itemsSelected", [iSelected, iTotal]);
    // Result: "5 item(s) selected out of 20 total"
    MessageToast.show(sMessage);
}
```

### Common Usage Patterns

**Pattern 1: Property Editor Labels**

```javascript
getProperties() {
    return [
        new WidgetProperty({
            displayName: this.getI18nText("property.apiEndpoint"),
            description: this.getI18nText("property.apiEndpoint.description"),
            category: "Main",
            propertyEditor: new StringPropertyEditor(this, "apiEndpoint")
        })
    ];
}
```

**Pattern 2: Dynamic Messages**

```javascript
async _loadData() {
    try {
        const oData = await ApiClient.custom.get("/data");
        const sSuccess = this.getI18nText("message.success");
        MessageToast.show(sSuccess);
    } catch (error) {
        const sError = this.getI18nText("message.error");
        MessageBox.error(sError);
    }
}
```

**Pattern 3: Binding in Controls**

```javascript
_createView() {
    return new VBox(this.getConfig().id, {
        items: [
            new Label({ text: "{i18n>label.username}" }),
            new Input({ placeholder: "{i18n>label.placeholder}" }),
            new Button({
                text: "{i18n>button.save}",
                press: () => this._onSave()
            })
        ]
    });
}
```

### Locale Detection

`I18nResourceModel` automatically detects user locale from browser settings. The loading order is:

1. `i18n_de_DE.properties` (exact locale match)
2. `i18n_de.properties` (language match)
3. `i18n.properties` (fallback/default)

No additional configuration needed - framework handles this automatically!

### Critical Rules

✅ **DO:**
- Define static `getI18nModel()` returning `I18nResourceModel`
- Use inherited `this.getI18nText(key, params)` method
- Use `PodContext.getI18nText(key, params)` for static contexts
- Use binding syntax `"{i18n>key}"` in controls
- Place i18n folder at namespace root
- Use dot notation for bundleName: `"namespace.i18n.i18n"`

❌ **DON'T:**
- Manually load ResourceBundle or ResourceModel
- Create custom `_loadI18n()` methods
- Store `_oResourceBundle` instance variables
- Load in `onInit()` - framework already loaded it!
- Use slash notation: `"namespace/i18n/i18n"`

### Troubleshooting

**Issue**: "this.getI18nText is not a function"
**Solution**: Ensure you defined static `getI18nModel()` returning `I18nResourceModel`. Framework calls this during registration.

**Issue**: Resource bundle not found (404 error)
**Solution**: Verify bundleName matches namespace structure with dots: `"mycompany.widget.i18n.i18n"` and i18n folder exists at namespace root.

**Issue**: Text shows as key (e.g., "button.text" instead of "Click Me")
**Solution**: Check i18n.properties file syntax and ensure key exists. Use browser console to verify bundle loaded.

### See Also

- [Production Patterns: i18n from SAP Code](production-patterns-sap.md)
- [SKILL.md: Step 3 - i18n Implementation](../SKILL.md)
- **SAPUI5 Documentation**: I18nResourceModel API Reference

---

## Navigation

📖 **Back to main skill**: [SKILL.md](../SKILL.md)

**Other references**:
- [Common Mistakes](common-mistakes.md) - All mistakes with fixes
- [Glossary](glossary.md) - Key terms & definitions
