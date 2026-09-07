# SAPUI5 Control API Reference

Quick reference for common SAPUI5 control quirks, property/method mismatches, and styling patterns relevant to POD 2.0 plugin development.

---

## Table of Contents

1. [Button Control](#button-control)
2. [Text Controls](#text-controls)
3. [Input Controls](#input-controls)
4. [Container Controls](#container-controls)
5. [Common Patterns](#common-patterns)
6. [Property vs Setter Reference](#property-vs-setter-reference)

---

## Button Control

### ❌ Common Mistake: setHeight() doesn't exist

**Error**: `TypeError: oButton.setHeight is not a function`

```javascript
// ❌ WRONG - Button has no setHeight() method
import Button from "sap/m/Button";

const oButton = new Button({
    text: "Click Me",
    width: "200px"
});
oButton.setHeight("80px");  // 💥 ERROR!
```

### ✅ Solution: Use inline styles via onAfterRendering

```javascript
// ✅ CORRECT - Set height via inline style
import Button from "sap/m/Button";

const oButton = new Button({
    text: "Click Me",
    width: "200px"  // ✅ width has setter
});

oButton.addEventDelegate({
    onAfterRendering: () => {
        const oDomRef = oButton.getDomRef();
        if (oDomRef) {
            oDomRef.style.height = "80px";  // ✅ Set via DOM
            oDomRef.style.backgroundColor = "#00FF00";
            oDomRef.style.color = "#FF0000";
        }
    }
});
```

### Button Properties Reference

| Property | Has Setter? | Alternative |
|----------|-------------|-------------|
| `text` | ✅ `setText()` | Constructor property |
| `width` | ✅ `setWidth()` | Constructor property |
| `height` | ❌ NO | Inline style only |
| `enabled` | ✅ `setEnabled()` | Constructor property |
| `visible` | ✅ `setVisible()` | Constructor property |
| `type` | ✅ `setType()` | Constructor property |
| `icon` | ✅ `setIcon()` | Constructor property |
| `backgroundColor` | ❌ NO | Inline style only |
| `color` | ❌ NO | Inline style only |

### Custom Button Styling Pattern

**⚠️ CRITICAL: When styling buttons in ControlWidget, you MUST:**
1. Use `!important` flag on all styles (SAPUI5 CSS has high specificity)
2. Target `.sapMBtnInner` for background colors and layout
3. Target `.sapMBtnContent` for text color and typography
4. Use flexbox (`display: flex`, `align-items: center`, `justify-content: center`) for vertical centering
5. Remove ControlWidget container styles (transparent background, no border)

```javascript
import Button from "sap/m/Button";
import ButtonType from "sap/m/ButtonType";

class CustomButton {
    createStyledButton(oConfig) {
        const oButton = new Button(oConfig.id, {
            text: "Custom Button",
            type: ButtonType.Emphasized,
            width: "200px"
        });

        // Apply custom styles after rendering
        oButton.addEventDelegate({
            onAfterRendering: () => {
                const oDomRef = oButton.getDomRef();
                if (oDomRef) {
                    // Set button height
                    oDomRef.style.setProperty("height", "80px", "important");
                    oDomRef.style.setProperty("min-height", "80px", "important");
                    
                    // Button outer colors
                    oDomRef.style.setProperty("background-color", "#00FF00", "important");
                    oDomRef.style.setProperty("background", "#00FF00", "important");
                    oDomRef.style.setProperty("border", "2px solid #00CC00", "important");

                    // Target .sapMBtnInner - the actual button background and layout
                    const oInnerButton = oDomRef.querySelector(".sapMBtnInner");
                    if (oInnerButton) {
                        oInnerButton.style.setProperty("height", "100%", "important");
                        oInnerButton.style.setProperty("background-color", "#00FF00", "important");
                        oInnerButton.style.setProperty("background", "#00FF00", "important");
                        oInnerButton.style.setProperty("border-color", "#00CC00", "important");
                        oInnerButton.style.setProperty("padding", "0 20px", "important");
                        
                        // CRITICAL: Use flexbox for vertical centering
                        oInnerButton.style.setProperty("display", "flex", "important");
                        oInnerButton.style.setProperty("align-items", "center", "important");
                        oInnerButton.style.setProperty("justify-content", "center", "important");
                    }

                    // Target .sapMBtnContent - the button text
                    const oButtonText = oDomRef.querySelector(".sapMBtnContent");
                    if (oButtonText) {
                        oButtonText.style.setProperty("color", "#FF0000", "important");
                        oButtonText.style.setProperty("font-size", "18px", "important");
                        oButtonText.style.setProperty("font-weight", "bold", "important");
                        oButtonText.style.setProperty("line-height", "normal", "important");
                    }
                }

                // CRITICAL: Remove ControlWidget container styling
                const oView = this.getView();
                if (oView) {
                    const oContainerDom = oView.getDomRef();
                    if (oContainerDom) {
                        oContainerDom.style.setProperty("background-color", "transparent", "important");
                        oContainerDom.style.setProperty("background", "transparent", "important");
                        oContainerDom.style.setProperty("border", "none", "important");
                    }
                }
            }
        });

        return oButton;
    }
}
```

### ❌ Common Mistakes When Styling Buttons in ControlWidget

**Mistake 1: Not using `!important`**
```javascript
// ❌ WRONG - SAPUI5 CSS overrides this
oDomRef.style.backgroundColor = "#00FF00";

// ✅ CORRECT - Use setProperty with "important"
oDomRef.style.setProperty("background-color", "#00FF00", "important");
```

**Mistake 2: Not targeting inner elements**
```javascript
// ❌ WRONG - Only styles outer button wrapper
oDomRef.style.setProperty("color", "#FF0000", "important");

// ✅ CORRECT - Target .sapMBtnContent for text color
const oButtonText = oDomRef.querySelector(".sapMBtnContent");
oButtonText.style.setProperty("color", "#FF0000", "important");
```

**Mistake 3: Text not vertically centered**
```javascript
// ❌ WRONG - Using line-height pushes text to top/bottom
oDomRef.style.setProperty("line-height", "80px", "important");

// ✅ CORRECT - Use flexbox on .sapMBtnInner
const oInnerButton = oDomRef.querySelector(".sapMBtnInner");
oInnerButton.style.setProperty("display", "flex", "important");
oInnerButton.style.setProperty("align-items", "center", "important");
oInnerButton.style.setProperty("justify-content", "center", "important");
```

**Mistake 4: Not removing ControlWidget container styling**
```javascript
// ❌ WRONG - Container inherits button's green background/border
// Result: Green box around green button

// ✅ CORRECT - Always clean up container styling
const oView = this.getView();
if (oView) {
    const oContainerDom = oView.getDomRef();
    if (oContainerDom) {
        oContainerDom.style.setProperty("background-color", "transparent", "important");
        oContainerDom.style.setProperty("background", "transparent", "important");
        oContainerDom.style.setProperty("border", "none", "important");
    }
}
```

---

## Text Controls

### Text (sap.m.Text)

```javascript
import Text from "sap/m/Text";

// ✅ Properties with setters
const oText = new Text();
oText.setText("Hello World");          // ✅
oText.setMaxLines(3);                  // ✅
oText.setTextAlign("Center");          // ✅
oText.setWidth("100%");                // ✅

// ❌ No setter for height
oText.setHeight("50px");               // ❌ NO - use inline style
```

### Title (sap.m.Title)

```javascript
import Title from "sap/m/Title";
import TitleLevel from "sap/ui/core/TitleLevel";

const oTitle = new Title({
    text: "My Title",
    level: TitleLevel.H2,  // H1, H2, H3, H4, H5, H6
    width: "100%"
});

// ✅ Has setters
oTitle.setText("New Title");           // ✅
oTitle.setLevel(TitleLevel.H3);        // ✅

// ❌ No height setter
oTitle.setHeight("60px");              // ❌ NO
```

### Label (sap.m.Label)

```javascript
import Label from "sap/m/Label";

const oLabel = new Label({
    text: "Field Name:",
    required: true,
    width: "120px"
});

// ✅ Has setters
oLabel.setText("New Label:");          // ✅
oLabel.setRequired(false);             // ✅
oLabel.setWidth("150px");              // ✅

// ❌ No height setter
oLabel.setHeight("40px");              // ❌ NO
```

---

## Input Controls

### Input (sap.m.Input)

```javascript
import Input from "sap/m/Input";

const oInput = new Input({
    value: "Initial Value",
    placeholder: "Enter text...",
    width: "300px"
});

// ✅ Has setters
oInput.setValue("New Value");          // ✅
oInput.setPlaceholder("Type here");    // ✅
oInput.setWidth("350px");              // ✅
oInput.setEnabled(false);              // ✅

// ❌ No height setter - but Input respects line-height
oInput.setHeight("50px");              // ❌ NO
```

**Input Height Workaround:**

```javascript
oInput.addEventDelegate({
    onAfterRendering: () => {
        const oDom = oInput.getDomRef();
        if (oDom) {
            oDom.style.height = "50px";
            // Also adjust inner input element
            const oInner = oDom.querySelector("input");
            if (oInner) {
                oInner.style.height = "50px";
                oInner.style.lineHeight = "50px";
            }
        }
    }
});
```

### ComboBox (sap.m.ComboBox)

```javascript
import ComboBox from "sap/m/ComboBox";
import Item from "sap/ui/core/Item";

const oComboBox = new ComboBox({
    width: "250px",
    items: [
        new Item({ key: "1", text: "Option 1" }),
        new Item({ key: "2", text: "Option 2" })
    ]
});

// ✅ Has setters
oComboBox.setSelectedKey("1");         // ✅
oComboBox.setWidth("300px");           // ✅
oComboBox.setEnabled(true);            // ✅

// ❌ No height setter
oComboBox.setHeight("50px");           // ❌ NO
```

---

## Container Controls

### VBox / HBox

```javascript
import VBox from "sap/m/VBox";
import HBox from "sap/m/HBox";

const oVBox = new VBox(oConfig.id, {
    width: "100%",
    height: "100%",  // ✅ height works in constructor
    items: [/* controls */]
});

// ✅ Both width and height have setters
oVBox.setWidth("500px");               // ✅
oVBox.setHeight("400px");              // ✅ Works for VBox/HBox!
```

**Note**: Unlike Button/Text, container controls (VBox, HBox, Panel) **DO** have `setHeight()` methods.

### Panel

```javascript
import Panel from "sap/m/Panel";

const oPanel = new Panel(oConfig.id, {
    headerText: "My Panel",
    width: "100%",
    height: "100%",
    expandable: true,
    expanded: true
});

// ✅ Panel has height setter
oPanel.setWidth("600px");              // ✅
oPanel.setHeight("500px");             // ✅ Works for Panel!
oPanel.setHeaderText("New Title");     // ✅
```

---

## Common Patterns

### Pattern 1: Styling Non-Container Controls

For controls **without** `setHeight()` (Button, Text, Label, Input):

```javascript
function applyCustomStyles(oControl, mStyles) {
    oControl.addEventDelegate({
        onAfterRendering: () => {
            const oDom = oControl.getDomRef();
            if (oDom) {
                Object.keys(mStyles).forEach(sKey => {
                    oDom.style[sKey] = mStyles[sKey];
                });
            }
        }
    });
}

// Usage
const oButton = new Button({ text: "Click Me" });
applyCustomStyles(oButton, {
    height: "80px",
    backgroundColor: "#00FF00",
    color: "#FF0000",
    fontSize: "18px",
    fontWeight: "bold"
});
```

### Pattern 2: Reusable Style Delegate

```javascript
class StyleHelper {
    static applyStyles(oControl, mStyles) {
        oControl.addEventDelegate({
            onAfterRendering: () => {
                const oDom = oControl.getDomRef();
                if (!oDom) return;
                
                Object.entries(mStyles).forEach(([key, value]) => {
                    oDom.style[key] = value;
                });
            }
        });
    }
    
    static applyClass(oControl, sClassName) {
        oControl.addStyleClass(sClassName);
    }
    
    static makeGreenButton(oButton) {
        this.applyStyles(oButton, {
            height: "80px",
            backgroundColor: "#00FF00",
            color: "#FF0000",
            border: "2px solid #00CC00",
            fontSize: "18px",
            fontWeight: "bold"
        });
    }
}

// Usage
const oBtn = new Button({ text: "Start" });
StyleHelper.makeGreenButton(oBtn);
```

### Pattern 3: CSS Classes (Preferred for Reusable Styles)

**Better approach for consistent styling:**

```javascript
// Create CSS in your plugin
// (Note: In POD 2.0, you'd need to inject this via a style tag)

const sCustomCSS = `
.custom-green-button {
    height: 80px !important;
    background-color: #00FF00 !important;
    color: #FF0000 !important;
    border: 2px solid #00CC00 !important;
    font-size: 18px !important;
    font-weight: bold !important;
}
`;

// Inject CSS once
function injectCustomCSS() {
    if (!document.getElementById("custom-pod-styles")) {
        const oStyle = document.createElement("style");
        oStyle.id = "custom-pod-styles";
        oStyle.textContent = sCustomCSS;
        document.head.appendChild(oStyle);
    }
}

// Usage
injectCustomCSS();
const oButton = new Button({ text: "Start SFC" });
oButton.addStyleClass("custom-green-button");  // ✅ Clean and reusable!
```

---

## Property vs Setter Reference

### Quick Lookup Table

| Control | width | height | text | enabled | visible | Custom Colors |
|---------|-------|--------|------|---------|---------|---------------|
| Button | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ (inline only) |
| Text | ✅ | ❌ | ✅ | N/A | ✅ | ❌ (inline only) |
| Label | ✅ | ❌ | ✅ | N/A | ✅ | ❌ (inline only) |
| Input | ✅ | ❌ | ✅ (value) | ✅ | ✅ | ❌ (inline only) |
| Title | ✅ | ❌ | ✅ | N/A | ✅ | ❌ (inline only) |
| VBox | ✅ | ✅ | N/A | N/A | ✅ | ❌ (inline only) |
| HBox | ✅ | ✅ | N/A | N/A | ✅ | ❌ (inline only) |
| Panel | ✅ | ✅ | ✅ (headerText) | N/A | ✅ | ❌ (inline only) |
| Table | ✅ | ✅ | N/A | N/A | ✅ | N/A |

**Legend:**
- ✅ = Has setter method (e.g., `setWidth()`)
- ❌ = No setter method (use inline styles)
- N/A = Not applicable

### General Rules

1. **width**: Most controls have `setWidth()` ✅
2. **height**: Only containers (VBox, HBox, Panel, Table) have `setHeight()` ✅
3. **Custom colors**: Always use inline styles ❌
4. **enabled/visible**: Most interactive controls have setters ✅
5. **text/value**: Content controls have setters ✅

---

## When to Use Each Approach

### Use Constructor Properties When:
- Setting initial values
- Values won't change after creation
- Standard SAPUI5 properties

```javascript
const oButton = new Button({
    text: "Click Me",
    type: ButtonType.Emphasized,
    width: "200px",
    enabled: true
});
```

### Use Setter Methods When:
- Values need to change dynamically
- Responding to user actions or data updates
- Properties have corresponding setters

```javascript
oButton.setText("New Text");
oButton.setEnabled(false);
oButton.setWidth("250px");
```

### Use Inline Styles When:
- Property has NO setter (height, colors, fonts)
- Applying custom CSS that SAPUI5 doesn't support
- Fine-grained visual control

```javascript
oButton.addEventDelegate({
    onAfterRendering: () => {
        const oDom = oButton.getDomRef();
        if (oDom) {
            oDom.style.height = "80px";
            oDom.style.backgroundColor = "#00FF00";
        }
    }
});
```

### Use CSS Classes When:
- Styles are reusable across multiple controls
- Need consistent theming
- Want to respect user's theme preferences

```javascript
oButton.addStyleClass("custom-large-button");
```

---

## Debugging Tips

### Check if a setter exists:

```javascript
const oButton = new Button();
console.log(typeof oButton.setHeight);  // "undefined" = no setter
console.log(typeof oButton.setWidth);   // "function" = has setter
```

### Inspect available methods:

```javascript
const oButton = new Button();
console.log(Object.getOwnPropertyNames(Object.getPrototypeOf(oButton))
    .filter(m => m.startsWith("set")));
// Output: ["setText", "setWidth", "setEnabled", "setType", ...]
```

### View all properties:

```javascript
const oButton = new Button();
console.log(oButton.getMetadata().getAllProperties());
```

---

## Common Errors and Solutions

### Error 1: `setHeight is not a function`

```javascript
// ❌ WRONG
oButton.setHeight("80px");

// ✅ CORRECT
oButton.addEventDelegate({
    onAfterRendering: () => {
        oButton.getDomRef().style.height = "80px";
    }
});
```

### Error 2: `setBackgroundColor is not a function`

```javascript
// ❌ WRONG
oButton.setBackgroundColor("#00FF00");

// ✅ CORRECT
oButton.addEventDelegate({
    onAfterRendering: () => {
        oButton.getDomRef().style.backgroundColor = "#00FF00";
    }
});
```

### Error 3: Styles not applying

**Problem**: Inline styles applied too early (before rendering)

```javascript
// ❌ WRONG - DOM doesn't exist yet
const oButton = new Button({ text: "Click" });
oButton.getDomRef().style.height = "80px";  // 💥 getDomRef() returns null!

// ✅ CORRECT - Wait for rendering
oButton.addEventDelegate({
    onAfterRendering: () => {
        const oDom = oButton.getDomRef();
        if (oDom) {
            oDom.style.height = "80px";  // ✅ DOM exists now
        }
    }
});
```

---

## Deprecated Pseudo-Module Imports ⭐ CRITICAL

**Problem**: Importing SAPUI5 enum/type constants as direct pseudo-modules produces deprecation warnings:

```
Importing the pseudo module 'sap/m/ButtonType' is deprecated. To access the type
'sap.m.ButtonType', please import 'sap/m/library'.
```

### ❌ WRONG — Direct pseudo-module imports (deprecated)

```javascript
sap.ui.define([
    "sap/m/ButtonType",          // ❌ Deprecated pseudo-module
    "sap/m/FlexAlignItems",      // ❌ Deprecated pseudo-module
    "sap/ui/core/ValueState"     // ❌ Deprecated pseudo-module
], (ButtonType, FlexAlignItems, ValueState) => { ... });
```

### ✅ CORRECT — Import from parent library module

```javascript
sap.ui.define([
    "sap/m/library",             // ✅ One import covers all sap/m enum types
    "sap/ui/core/library"        // ✅ One import covers all sap/ui/core enum types
], (mobileLibrary, coreLibrary) => {
    "use strict";

    // Extract enum types from library objects
    var ButtonType         = mobileLibrary.ButtonType;
    var FlexAlignItems     = mobileLibrary.FlexAlignItems;
    var FlexJustifyContent = mobileLibrary.FlexJustifyContent;
    var FlexWrap           = mobileLibrary.FlexWrap;
    var PlacementType      = mobileLibrary.PlacementType;
    var ListMode           = mobileLibrary.ListMode;
    var ListSeparators     = mobileLibrary.ListSeparators;
    var MessageType        = mobileLibrary.MessageType;
    var ValueState         = coreLibrary.ValueState;
    var TextAlign          = coreLibrary.TextAlign;
    var TextDirection      = coreLibrary.TextDirection;
});
```

### Complete Replacement Table

| ❌ Deprecated pseudo-module | ✅ Library | ✅ Extraction |
|-----------------------------|-----------|--------------|
| `"sap/m/ButtonType"` | `"sap/m/library"` | `var ButtonType = mobileLibrary.ButtonType` |
| `"sap/m/FlexAlignItems"` | `"sap/m/library"` | `var FlexAlignItems = mobileLibrary.FlexAlignItems` |
| `"sap/m/FlexJustifyContent"` | `"sap/m/library"` | `var FlexJustifyContent = mobileLibrary.FlexJustifyContent` |
| `"sap/m/FlexWrap"` | `"sap/m/library"` | `var FlexWrap = mobileLibrary.FlexWrap` |
| `"sap/m/PlacementType"` | `"sap/m/library"` | `var PlacementType = mobileLibrary.PlacementType` |
| `"sap/m/ListMode"` | `"sap/m/library"` | `var ListMode = mobileLibrary.ListMode` |
| `"sap/m/ListSeparators"` | `"sap/m/library"` | `var ListSeparators = mobileLibrary.ListSeparators` |
| `"sap/m/MessageType"` | `"sap/m/library"` | `var MessageType = mobileLibrary.MessageType` |
| `"sap/ui/core/ValueState"` | `"sap/ui/core/library"` | `var ValueState = coreLibrary.ValueState` |
| `"sap/ui/core/TextAlign"` | `"sap/ui/core/library"` | `var TextAlign = coreLibrary.TextAlign` |
| `"sap/ui/core/TextDirection"` | `"sap/ui/core/library"` | `var TextDirection = coreLibrary.TextDirection` |

**Rule of thumb**: If the path is `sap/m/<Name>` or `sap/ui/core/<Name>` and it's a constant/enum (not a control class like `Button`, `VBox`, `Input`), it's a deprecated pseudo-module — replace it with the library pattern above.

---

## References

- [SAPUI5 SDK - sap.m.Button](https://sapui5.hana.ondemand.com/sdk/#/api/sap.m.Button)
- [SAPUI5 SDK - Control API](https://sapui5.hana.ondemand.com/sdk/#/api/sap.ui.core.Control)
- [SAPUI5 Custom Styling Guide](https://sapui5.hana.ondemand.com/sdk/#/topic/91f087396f4d1014b6dd926db0e91070)

---

**Last Updated**: 2026-09-01  
**Version**: 1.0.0
