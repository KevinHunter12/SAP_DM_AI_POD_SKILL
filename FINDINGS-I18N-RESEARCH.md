# i18n Implementation Research Findings
**Date**: 2026-04-17  
**Source**: C:\VSCodeProjects\v2 (SAP POD 2.0 production code)

---

## 🔍 RESEARCH SUMMARY

### What We Found in **ACTUAL SAP Production Code**:

1. **Widget classes define a static `getI18nModel()` method** (from ComponentWidget.js JSDoc example)
2. **Widgets call `this.getI18nText(key)` inherited method** (found in ControlWidget, ActivityConfirmationTableWidget, ComponentWidget)
3. **Static method `PodContext.getI18nText(key)` is also used** (found in ActivityConfirmationTableWidget)
4. **Binding syntax `"{i18n>key}"` is used in controls** (found in ActivityConfirmationTableWidget)
5. **NO manual ResourceBundle.create() calls** found in any production widget
6. **NO manual ResourceModel instantiation in widget constructors** found

---

## 📋 EVIDENCE FROM PRODUCTION CODE

### Example 1: ComponentWidget.js (Lines 78-88)
**JSDoc documentation example** showing the CORRECT pattern:

```javascript
/**
 * @example
 * class CustomWidget extends ComponentWidget {
 *    static #oI18nModel = new I18nResourceModel({
 *        bundleName: "acme.widget.customWidget.i18n.i18n",
 *        enhanceWith: [{
 *            bundleName: "acme.widget.customWidget.i18n.builder"
 *        }]
 *    })
 *
 *    static getI18nModel() {
 *        return this.#oI18nModel;
 *    }
 *
 *    static getComponentName() {
 *        return "acme.widget.customWidget";
 *    }
 *
 *    // Models to be applied to the ComponentContainer
 *    static getComponentModels() {
 *        return {
 *            i18n: this.#oI18nModel
 *        };
 *    }
 * }
 */
```

**Key Points:**
- Static private field `#oI18nModel` with `I18nResourceModel`
- Static getter `getI18nModel()` returns the model
- Used in `getComponentModels()` to expose to container

---

### Example 2: ActivityConfirmationTableWidget.js Usage

**Using `this.getI18nText()` inherited method:**
```javascript
displayName: this.getI18nText(`ActivityConfirmationTableWidget.properties.${PropertyId.customField}.name`)
text: this.getI18nText("ActivityConfirmationTableWidget.toolbarTitle", iLength || 0)
text: this.getI18nText("ActivityConfirmationTableWidget.report.btn")
```

**Using `PodContext.getI18nText()` static method:**
```javascript
text: PodContext.getI18nText("ActivityConfirmationTableWidget.viewPostings.btn")
```

**Using binding syntax:**
```javascript
text: "{i18n>ActivityConfirmationTableWidget.columns.parameter}"
```

---

### Example 3: ComponentWidget.js Usage (Lines 154-161)

```javascript
getProperties() {
    return [
        new WidgetProperty({
            displayName: this.getI18nText("designer.width"),
            description: this.getI18nText("designer.width.description"),
            category: PropertyCategory.Dimension,
            propertyEditor: new StringPropertyEditor(this, ComponentWidget.#PropertyId.Width)
        }),
        // ...
    ];
}
```

---

### Example 4: ControlWidget.js Usage

**In `getProperties()` method:**
```javascript
const sI18nKey = `designer.properties.${oProperty.name}.label`;
sDisplayName = PodContext.getI18nText(sI18nKey);

// If no translation was found, generate a label from the key.
if (sDisplayName === sI18nKey) {
    sDisplayName = this._camelCaseToWords(oProperty.name);
}
```

**In `getEvents()` method:**
```javascript
const sDisplayNameKey = `designer.event.${oEvent.name}.label`;
let sDisplayName = this.getI18nText(sDisplayNameKey);
if (sDisplayName === sDisplayNameKey) {
    sDisplayName = this._camelCaseToWords(oEvent.name);
}
```

---

## 🏗️ ARCHITECTURE UNDERSTANDING

### Class Hierarchy:
```
PodObject (SAP internal base class)
  └─ Widget extends PodObject
      ├─ ComponentWidget extends Widget
      ├─ ControlWidget extends Widget
      │   └─ TableWidget extends ControlWidget
      │       └─ ActivityConfirmationTableWidget extends TableWidget
      └─ LayoutWidget extends Widget
```

### i18n Method Inheritance:
- `PodObject` (internal SAP class) likely provides `getI18nText()` instance method
- Widgets define static `getI18nModel()` to provide their i18n model
- Framework loads models via `WidgetRegistry.loadCoreWidgets()` which calls `await oWidgetClass.getI18nModel()?.ready()`
- Once loaded, `this.getI18nText(key)` inherited method works automatically
- Binding syntax `"{i18n>key}"` works once model is registered

### Key Framework Code (WidgetRegistry.js:123):
```javascript
// Ensure all i18n resources are loaded before proceeding
await oWidgetClass.getI18nModel()?.ready();
```

This shows the framework automatically loads i18n models during widget registration!

---

## ❌ WHAT WE **DON'T** FIND IN PRODUCTION CODE

### NOT Found:
1. ❌ Manual `ResourceBundle.create()` calls in widget constructors
2. ❌ Manual `ResourceModel` instantiation in widget constructors
3. ❌ Storing `_oResourceBundle` as instance variable
4. ❌ Manual `_loadI18n()` methods in widget classes
5. ❌ Async loading in `onInit()` for i18n

### Why They're Missing:
- Framework handles model loading automatically
- Widgets just provide static `getI18nModel()` 
- Inherited `this.getI18nText()` method handles lookups

---

## ✅ THE CORRECT PATTERN (From Production Code)

### Pattern for Custom Widgets:

```javascript
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class MyCustomWidget extends Widget {
    
    // Static private i18n model
    static #oI18nModel = new I18nResourceModel({
        bundleName: "your.namespace.i18n.i18n"  // Dot notation!
    });
    
    // Static getter for framework
    static getI18nModel() {
        return this.#oI18nModel;
    }
    
    // Use inherited method anywhere
    _someMethod() {
        const sText = this.getI18nText("myWidget.greeting");
        const sTitle = this.getI18nText("myWidget.title", arg1, arg2);
    }
    
    // Or use PodContext static method
    _anotherMethod() {
        const sText = PodContext.getI18nText("myWidget.error");
    }
    
    // Or use binding in controls
    _createView() {
        return new Button({
            text: "{i18n>myWidget.button.label}"
        });
    }
}
```

---

## 🎯 CONCLUSION

### The CORRECT i18n pattern is:

1. **Define static `getI18nModel()` method** returning `I18nResourceModel`
2. **Framework loads it automatically** during widget registration
3. **Use inherited `this.getI18nText(key, ...args)` method** anywhere in widget
4. **OR use `PodContext.getI18nText(key, ...args)` static method**
5. **OR use binding syntax `"{i18n>key}"` in controls**

### What we documented (v12.1.0) is **WRONG**:
- ❌ No need for manual `ResourceBundle.create()` in `onInit()`
- ❌ No need for manual `ResourceModel` in constructor
- ❌ No need for `_oResourceBundle` instance field
- ❌ No need for custom `_loadI18n()` methods

---

## 📝 NEXT STEPS

1. **Remove incorrect i18n documentation** from SKILL.md and widget-patterns.md
2. **Add correct pattern** showing static `getI18nModel()` approach
3. **Update examples** to show `this.getI18nText()` usage
4. **Document all three methods**: inherited method, PodContext static, binding syntax
5. **Update version** to 13.0.0 (major breaking change in documentation)
