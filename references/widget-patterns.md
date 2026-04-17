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
        // CRITICAL: ALWAYS use method calls in _createView() - binding syntax doesn't work!
        _createView() {
            const oConfig = this.getConfig();

            return new VBox(oConfig.id, {
                items: [
                    // ✅ CORRECT: Use inherited getI18nText() method
                    new Label({ 
                        text: this.getI18nText("label.welcome") 
                    }),
                    
                    // ✅ CORRECT: Use method call, not binding
                    new Button({
                        text: this.getI18nText("button.submit"),  // Method call!
                        press: () => this._onButtonPress()
                    }),
                    
                    // ✅ CORRECT: With parameters
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

### How to Use i18n - CRITICAL Rules

**🚨 CRITICAL: NEVER use binding syntax `"{i18n>key}"` during `_createView()`!**

The i18n model is NOT available during view creation, so binding syntax will fail silently or cause errors.

#### ✅ CORRECT: Use Method Calls (ALWAYS)

POD 2.0 provides two methods for using internationalized text:

#### Method 1: Inherited `this.getI18nText()` (Recommended for widgets)

```javascript
// ✅ CORRECT: Available anywhere in widget instance
const sText = this.getI18nText("myWidget.greeting");
const sWithParams = this.getI18nText("myWidget.title", [arg1, arg2]);

// ✅ CORRECT: In _createView()
_createView() {
    return new Button({
        text: this.getI18nText("button.submit")  // Method call works!
    });
}
```

**Use when:** You're inside widget instance methods (including `_createView()`)

#### Method 2: Static `PodContext.getI18nText()`

```javascript
// ✅ CORRECT: Static method, works anywhere
const sText = PodContext.getI18nText("myWidget.error");
const sWithParams = PodContext.getI18nText("myWidget.message", [count]);
```

**Use when:** You need i18n in static methods or outside widget context

#### ❌ WRONG: Binding Syntax During View Creation

```javascript
// ❌ WRONG: This does NOT work in _createView()!
_createView() {
    return new Button({
        text: "{i18n>button.submit}"  // Model not available yet!
    });
}
```

**Why this fails:** The i18n model isn't registered until AFTER `_createView()` completes, so bindings fail to resolve.

#### ⚠️ Binding Syntax MAY Work After onInit() (Not Recommended)

Binding syntax `"{i18n>key}"` might work for controls created dynamically AFTER `onInit()`, but:
- **Method calls are safer and more consistent**
- **Always works, regardless of timing**
- **No dependency on model availability**

**Bottom line:** Always use method calls (`this.getI18nText()` or `PodContext.getI18nText()`)

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

## SAP DM API Integration Pattern

POD widgets can call any of the 70+ SAP Digital Manufacturing REST APIs for production operations, material management, quality, inventory, and more.

### API Reference

📖 **Complete API Reference**: [references/sapdm-api-reference.md](sapdm-api-reference.md)  
📖 **API Specifications**: [references/api-specs/](api-specs/)

### Authentication & Base URL Pattern

All SAP DM APIs use OAuth 2.0. PodContext provides token and service registry:

```javascript
const oContext = PodContext.getContext();
const sToken = oContext.token;                           // OAuth 2.0 Bearer token
const sPlant = oContext.plant;                           // Current plant
const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc"); // Get API base URL
```

### Common API Categories

**Production APIs:**
- **SFC (Shop Floor Control)**: Start, complete, serialize, split, merge SFCs
- **Order**: Find orders, release for production, update custom values
- **Activity/Quantity Confirmation**: Confirm labor, yield, scrap, rework
- **Assembly**: Assemble/unassemble components to SFCs

**Material & BOM APIs:**
- **Material**: Create, search, update materials with routing, BOM, storage locations
- **BOM**: Define material components required for production
- **Batch**: Manage material batches and traceability

**Quality & Data Collection APIs:**
- **Data Collection**: Log parameter values at manufacturing process points
- **Quality Inspection**: Create and manage quality inspections for SFCs
- **Nonconformance**: Report and manage defects/issues

**Inventory & Logistics APIs:**
- **Inventory**: Manage inventory levels, locations, movements
- **Staging**: Stage materials for production operations
- **WIP**: Track work in process inventory

### API Call Pattern (Fetch)

```javascript
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import MessageToast from "sap/m/MessageToast";

class MyApiWidget extends Widget {
    
    /**
     * Fetch SFC details from SAP DM SFC API
     * @param {string} sSfc - SFC number
     * @returns {Promise<Object>} SFC details
     */
    async _fetchSfcDetails(sSfc) {
        const oContext = PodContext.getContext();
        const sPlant = oContext.plant;
        const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
        const sUrl = `${sBaseUrl}/sfcs?plant=${sPlant}&sfc=${sSfc}`;
        
        try {
            const oResponse = await fetch(sUrl, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                    "Authorization": `Bearer ${oContext.token}`
                }
            });
            
            if (!oResponse.ok) {
                const oError = await oResponse.json();
                throw new Error(oError.message || `HTTP ${oResponse.status}`);
            }
            
            return await oResponse.json();
            
        } catch (oError) {
            console.error("Failed to fetch SFC details:", oError);
            MessageToast.show(this.getI18nText("api.error.sfc"));
            throw oError;
        }
    }
    
    /**
     * Start SFCs at operation
     * @param {Array<string>} aSfcs - SFC numbers to start
     * @param {string} sOperation - Operation activity
     * @param {string} sResource - Resource name
     * @returns {Promise<Object>} Start response
     */
    async _startSfcs(aSfcs, sOperation, sResource) {
        const oContext = PodContext.getContext();
        const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
        
        const oRequest = {
            plant: oContext.plant,
            sfcs: aSfcs.map(sSfc => ({ sfc: sSfc })),
            operationActivity: sOperation,
            resource: sResource
        };
        
        try {
            const oResponse = await fetch(`${sBaseUrl}/sfcs/start`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    "Authorization": `Bearer ${oContext.token}`
                },
                body: JSON.stringify(oRequest)
            });
            
            if (!oResponse.ok) {
                throw new Error(`Start failed: HTTP ${oResponse.status}`);
            }
            
            return await oResponse.json();
            
        } catch (oError) {
            console.error("Failed to start SFCs:", oError);
            throw oError;
        }
    }
}
```

### API Call Pattern (jQuery Ajax - SAPUI5 Standard)

```javascript
import PodContext from "sap/dm/dme/pod2/context/PodContext";

class MyApiWidget extends Widget {
    
    /**
     * Log data collection values
     * @param {Object} oData - Data collection request
     * @returns {Promise<Object>} Log response
     */
    _logDataCollection(oData) {
        const oContext = PodContext.getContext();
        const sUrl = `${oContext.serviceRegistry.getApiUrl("datacollection")}/log`;
        
        return new Promise((resolve, reject) => {
            jQuery.ajax({
                url: sUrl,
                method: "POST",
                contentType: "application/json",
                data: JSON.stringify(oData),
                headers: {
                    "Authorization": `Bearer ${oContext.token}`
                },
                success: (oResponse) => {
                    console.log("Data collection logged:", oResponse);
                    resolve(oResponse);
                },
                error: (oError) => {
                    console.error("Data collection failed:", oError);
                    reject(oError);
                }
            });
        });
    }
}
```

### Complete API Widget Example

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/context/ModelPath",
    "sap/m/VBox",
    "sap/m/Input",
    "sap/m/Button",
    "sap/m/Text",
    "sap/m/MessageToast",
    "sap/dm/dme/pod2/model/I18nResourceModel"
], (Widget, PodContext, ModelPath, VBox, Input, Button, Text, MessageToast, I18nResourceModel) => {
    "use strict";
    
    /**
     * Widget that fetches SFC details from SAP DM API
     */
    class SfcDetailsWidget extends Widget {
        
        // Static i18n model
        static #oI18nModel = new I18nResourceModel({
            bundleName: "custom.pod2.sfcdetails.i18n.i18n"
        });
        
        static getI18nModel() {
            return this.#oI18nModel;
        }
        
        static getDisplayName() {
            return "SFC Details Widget";
        }
        
        static getIcon() {
            return "sap-icon://product";
        }
        
        constructor(oConfig) {
            super(Widget, oConfig);
            this._oSfcInput = null;
            this._oResultText = null;
        }
        
        /**
         * Create widget view
         */
        _createView() {
            this._oSfcInput = new Input({
                placeholder: this.getI18nText("sfcDetails.input.placeholder"),
                width: "15rem"
            });
            
            const oFetchButton = new Button({
                text: this.getI18nText("sfcDetails.button.fetch"),
                press: () => this._onFetchSfc()
            });
            
            this._oResultText = new Text({
                text: ""
            });
            
            return new VBox({
                items: [
                    this._oSfcInput,
                    oFetchButton,
                    this._oResultText
                ],
                class: "sapUiSmallMargin"
            });
        }
        
        /**
         * Initialize widget - subscribe to context
         */
        async onInit() {
            await super.onInit();
            
            // Subscribe to selected SFC changes
            this.subscribe(ModelPath.SelectedSfc, this._onSfcSelected.bind(this));
        }
        
        /**
         * Handle selected SFC change
         */
        _onSfcSelected(sSfc, sPath) {
            if (sSfc) {
                this._oSfcInput.setValue(sSfc);
                this._onFetchSfc();
            }
        }
        
        /**
         * Fetch SFC details from API
         */
        async _onFetchSfc() {
            const sSfc = this._oSfcInput.getValue();
            
            if (!sSfc) {
                MessageToast.show(this.getI18nText("sfcDetails.error.noSfc"));
                return;
            }
            
            try {
                const oData = await this._fetchSfcDetails(sSfc);
                
                const sResult = `SFC: ${oData.sfc}\n` +
                               `Material: ${oData.material}\n` +
                               `Status: ${oData.status}\n` +
                               `Quantity: ${oData.quantity}`;
                
                this._oResultText.setText(sResult);
                
            } catch (oError) {
                this._oResultText.setText(this.getI18nText("sfcDetails.error.fetch"));
            }
        }
        
        /**
         * Call SAP DM SFC API to get SFC details
         */
        async _fetchSfcDetails(sSfc) {
            const oContext = PodContext.getContext();
            const sPlant = oContext.plant;
            const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
            const sUrl = `${sBaseUrl}/sfcs?plant=${sPlant}&sfc=${sSfc}`;
            
            const oResponse = await fetch(sUrl, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                    "Authorization": `Bearer ${oContext.token}`
                }
            });
            
            if (!oResponse.ok) {
                throw new Error(`HTTP ${oResponse.status}: ${oResponse.statusText}`);
            }
            
            return await oResponse.json();
        }
        
        /**
         * Cleanup on destroy
         */
        onExit() {
            this.unsubscribe();
            super.onExit();
        }
    }
    
    return SfcDetailsWidget;
});
```

### API Best Practices

✅ **DO:**
- Cache OAuth tokens from PodContext (already cached by framework)
- Use `serviceRegistry.getApiUrl()` for base URLs
- Handle errors gracefully with user-friendly messages
- Show loading indicators for long API calls
- Validate input before making API calls
- Use async/await for cleaner code
- Log errors for debugging
- Check HTTP status codes

❌ **DON'T:**
- Request new OAuth token for every API call
- Hardcode API base URLs
- Show raw error messages to users
- Make synchronous API calls (blocks UI)
- Trust user input without validation
- Ignore error responses
- Make redundant API calls (cache when appropriate)

### Error Handling Pattern

```javascript
async _callApi(sEndpoint, oData) {
    const oContext = PodContext.getContext();
    const sUrl = `${oContext.serviceRegistry.getApiUrl("service")}${sEndpoint}`;
    
    try {
        const oResponse = await fetch(sUrl, {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${oContext.token}`
            },
            body: JSON.stringify(oData)
        });
        
        if (!oResponse.ok) {
            const oError = await oResponse.json();
            throw new Error(oError.message || `HTTP ${oResponse.status}`);
        }
        
        return await oResponse.json();
        
    } catch (oError) {
        console.error(`API call failed (${sEndpoint}):`, oError);
        
        // Show user-friendly error message
        const sErrorKey = oError.code ? `api.error.${oError.code}` : "api.error.generic";
        MessageToast.show(this.getI18nText(sErrorKey));
        
        throw oError;
    }
}
```

### Pagination Pattern

Many list APIs support pagination:

```javascript
async _fetchMaterialList(sSearchTerm, iPage = 0, iSize = 20) {
    const oContext = PodContext.getContext();
    const sBaseUrl = oContext.serviceRegistry.getApiUrl("material");
    const sUrl = `${sBaseUrl}/v1/materials/list?plant=${oContext.plant}&page=${iPage}&size=${iSize}&search=${sSearchTerm}`;
    
    const oResponse = await fetch(sUrl, {
        headers: {
            "Authorization": `Bearer ${oContext.token}`
        }
    });
    
    const oData = await oResponse.json();
    
    return {
        items: oData.content,
        totalItems: oData.totalElements,
        totalPages: oData.totalPages,
        currentPage: oData.page
    };
}
```

### Async Operations Pattern

Some APIs support async processing:

```javascript
async _startSfcsAsync(aSfcs) {
    const oContext = PodContext.getContext();
    const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
    
    // Start async operation
    const oResponse = await fetch(`${sBaseUrl}/sfcs/start?async=true`, {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "Authorization": `Bearer ${oContext.token}`
        },
        body: JSON.stringify({
            plant: oContext.plant,
            sfcs: aSfcs.map(sSfc => ({ sfc: sSfc })),
            operationActivity: "OPER_1,1",
            resource: "RESOURCE_1"
        })
    });
    
    const { asyncExecutionId } = await oResponse.json();
    
    // Poll for completion
    return await this._pollAsyncResult(asyncExecutionId);
}

async _pollAsyncResult(sExecutionId, iMaxAttempts = 30) {
    const oContext = PodContext.getContext();
    const sBaseUrl = oContext.serviceRegistry.getApiUrl("sfc");
    
    for (let i = 0; i < iMaxAttempts; i++) {
        await new Promise(resolve => setTimeout(resolve, 2000)); // Wait 2s
        
        const oResponse = await fetch(`${sBaseUrl}/async/${sExecutionId}`, {
            headers: {
                "Authorization": `Bearer ${oContext.token}`
            }
        });
        
        const oResult = await oResponse.json();
        
        if (oResult.status === "COMPLETED") {
            return oResult.data;
        } else if (oResult.status === "FAILED") {
            throw new Error(oResult.error || "Async operation failed");
        }
    }
    
    throw new Error("Async operation timeout");
}
```

### API Documentation

📖 **See Also:**
- **[sapdm-api-reference.md](sapdm-api-reference.md)** - Complete reference for all 70+ SAP DM APIs
- **[api-specs/](api-specs/)** - Full OpenAPI/Swagger specifications
- **SAP Help Portal**: [API Integration Guide](https://help.sap.com/docs/sap-digital-manufacturing/operations-guide/prepare-for-api-integration)

**Common APIs:**
- SFC API: `api-specs/sapdme_sfc.json` - Shop floor control operations
- Order API: `api-specs/sapdme_order.json` - Production order management
- Material API: `api-specs/sapdme_material.json` - Material master data
- Data Collection API: `api-specs/sapdme_datacollection.json` - Parameter logging
- Quality Inspection API: `api-specs/sapdme_qualityinspection.json` - Quality checks
- Inventory API: `api-specs/sapdme_inventory.json` - Inventory management

---

## Navigation

📖 **Back to main skill**: [SKILL.md](../SKILL.md)

**Other references**:
- [Common Mistakes](common-mistakes.md) - All mistakes with fixes
- [SAP DM API Reference](sapdm-api-reference.md) - Complete API documentation
- [Glossary](glossary.md) - Key terms & definitions
