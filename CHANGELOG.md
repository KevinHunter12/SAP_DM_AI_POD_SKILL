# POD Plugin Skill - Update Changelog

## Version 13.0.0 - 2026-04-17 🚨 **BREAKING CHANGE**

### 🔥 CRITICAL FIX: i18n Implementation Pattern

**Research Finding**: Versions 12.0.0-12.1.0 documented **INCORRECT** i18n patterns. After researching actual SAP production code (`C:\VSCodeProjects\v2`), discovered POD 2.0 framework automatically loads i18n models - widgets should NOT manually load ResourceBundle or ResourceModel!

---

## ⚠️ BREAKING CHANGES

### ❌ REMOVED INCORRECT PATTERNS:
1. **Manual ResourceBundle loading** in `onInit()` - ❌ NOT used in SAP code
2. **Manual ResourceModel loading** in constructor - ❌ NOT used in SAP code  
3. **Custom `_loadI18n()` methods** - ❌ NOT found in production
4. **Storing `_oResourceBundle` instance variables** - ❌ NOT used in SAP widgets

### ✅ ADDED CORRECT PATTERN (From SAP Production Code):

**Static `getI18nModel()` with framework loading:**

```javascript
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class MyWidget extends Widget {
    // 1. Static private i18n model
    static #oI18nModel = new I18nResourceModel({
        bundleName: "your.namespace.i18n.i18n"  // Dots!
    });
    
    // 2. Static getter for framework (auto-called during registration)
    static getI18nModel() {
        return this.#oI18nModel;
    }
    
    // 3. Use inherited getI18nText() method - no manual loading!
    _someMethod() {
        const sText = this.getI18nText("myWidget.greeting");
    }
}
```

**Three ways to use i18n:**
- `this.getI18nText(key, ...args)` - Inherited method
- `PodContext.getI18nText(key, ...args)` - Static method
- `"{i18n>key}"` - Binding syntax in controls

---

## 📋 FILES CHANGED

### 1. SKILL.md
- **Line 558-608**: Replaced incorrect manual loading approaches
- **Version**: 12.1.0 → 13.0.0
- **Tags**: Removed `i18n-resourcemodel`, `i18n-resourcebundle`, added `I18nResourceModel`, `framework-driven-i18n`
- **Description**: Updated to mention framework-driven pattern

**OLD (v12.1.0 - WRONG):**
```javascript
// Approach A: ResourceModel
constructor(oConfig) {
    super(BaseClass, oConfig);
    this._oResourceBundle = null;
    this._loadI18n();
}

// Approach B: ResourceBundle
async onInit() {
    this._oResourceBundle = await ResourceBundle.create({...});
}
```

**NEW (v13.0.0 - CORRECT):**
```javascript
static #oI18nModel = new I18nResourceModel({
    bundleName: "namespace.i18n.i18n"
});

static getI18nModel() {
    return this.#oI18nModel;
}

// Use inherited method
const sText = this.getI18nText("key");
```

### 2. references/widget-patterns.md
- **Lines 716-1313**: Complete rewrite of i18n section (~600 lines)
- Removed manual ResourceBundle/ResourceModel loading examples
- Added framework-driven pattern from SAP production code
- Added three usage methods (inherited, PodContext, binding)
- Updated troubleshooting section

### 3. NEW: FINDINGS-I18N-RESEARCH.md
- Complete research documentation
- Evidence from actual SAP production widgets
- Examples from ComponentWidget, ActivityConfirmationTableWidget, ControlWidget
- Framework loading mechanism (WidgetRegistry.js)
- Explanation of PodObject inheritance

---

## 📊 RESEARCH EVIDENCE

**Sources analyzed:**
- `C:\VSCodeProjects\v2\Widget.js` - Base class (extends PodObject)
- `C:\VSCodeProjects\v2\ComponentWidget.js` - Shows getI18nModel() pattern in JSDoc
- `C:\VSCodeProjects\v2\activityconfirmation/ActivityConfirmationTableWidget.js` - Production usage
- `C:\VSCodeProjects\v2\ControlWidget.js` - Shows this.getI18nText() usage
- `C:\VSCodeProjects\v2\WidgetRegistry.js` - Shows `await oWidgetClass.getI18nModel()?.ready()`

**Key Findings:**
1. Framework calls `getI18nModel()` during widget registration
2. Widgets use inherited `this.getI18nText()` from PodObject
3. Alternative: `PodContext.getI18nText()` static method
4. Binding syntax `"{i18n>key}"` works in controls
5. **ZERO manual ResourceBundle/ResourceModel loading found in production**

---

## 🎯 IMPACT

### Who is affected:
- **All users who implemented i18n using v12.0.0 or v12.1.0 documentation**
- Widgets using manual ResourceBundle.create() or ResourceModel loading
- Any code following the previous "two approaches" pattern

### Migration required:
1. Remove manual ResourceBundle/ResourceModel loading code
2. Add static `#oI18nModel` field with I18nResourceModel
3. Add static `getI18nModel()` getter
4. Replace `this._getI18nText()` calls with `this.getI18nText()` (inherited method)
5. Remove `_oResourceBundle` instance variables
6. Remove `_loadI18n()` custom methods

### Why this is critical:
- Previous documentation taught patterns NOT used in SAP production code
- Manual loading creates unnecessary complexity
- Framework provides automatic loading - widgets just need to declare the model
- Inherited methods are simpler and more maintainable

---

## ✅ VERIFICATION

Research method:
1. Searched C:\VSCodeProjects\v2 for actual SAP production widget code
2. Found NO manual ResourceBundle.create() in any widget
3. Found NO ResourceModel instantiation in widget constructors
4. Found ComponentWidget JSDoc example showing correct pattern
5. Found WidgetRegistry calling getI18nModel() during registration
6. Found production widgets using inherited this.getI18nText() method

---

## 📚 SEE ALSO

- [FINDINGS-I18N-RESEARCH.md](FINDINGS-I18N-RESEARCH.md) - Complete research documentation
- [SKILL.md - Step 3](SKILL.md#quick-start-guide) - Updated i18n pattern
- [widget-patterns.md - i18n Section](references/widget-patterns.md#i18n-internationalization-pattern) - Complete examples

---

## Version 12.1.0 - 2026-04-17 (SUPERSEDED - INCORRECT)

❌ **This version contained INCORRECT i18n documentation**

See Version 13.0.0 above for correct implementation.

---

## Version 12.0.0 - 2026-04-17 (SUPERSEDED - INCORRECT)

❌ **This version contained INCORRECT i18n documentation**

See Version 13.0.0 above for correct implementation.

Previous changelog entries moved to [CHANGELOG-ARCHIVE.md](CHANGELOG-ARCHIVE.md).
