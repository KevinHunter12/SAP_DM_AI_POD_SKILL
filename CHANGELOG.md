# POD Plugin Skill - Update Changelog

## Version 14.0.0 - 2026-04-17 🚨 **BREAKING CHANGE**

### 🔥 NEW FEATURE: Import & ModelPath Validation Documentation

**Added comprehensive validation to prevent the most common POD 2.0 plugin errors**: wrong PlacementType import and incorrect ModelPath constants.

---

## ⚠️ NEW DOCUMENTATION SECTIONS

### 1. Common Mistakes #12-#13 (NEW)

**Mistake #12: Wrong PlacementType Import**

Many developers incorrectly import PlacementType from `sap/ui/core/library` when it's actually in `sap/m/PlacementType`. This causes undefined errors when using Popover, Dialog, and other controls.

✅ **CORRECT:**
```javascript
import PlacementType from "sap/m/PlacementType";
```

❌ **WRONG:**
```javascript
import coreLibrary from "sap/ui/core/library";
const { PlacementType } = coreLibrary;  // PlacementType is undefined!
```

**Mistake #13: Wrong ModelPath Constants**

Developers frequently use singular forms like `ModelPath.SelectedWorkListItem` when the correct constant is **plural** `ModelPath.SelectedWorkListItems` and returns an array.

✅ **CORRECT:**
```javascript
ModelPath.SelectedWorkListItems   // Array (note plural!)
ModelPath.WorkListItems           // Array
```

❌ **WRONG:**
```javascript
ModelPath.SelectedWorkListItem    // Doesn't exist!
ModelPath.SelectedSfc              // Doesn't exist!
```

---

### 2. Common Imports Reference Section (NEW)

Added to [references/pod2-api-reference.md](references/pod2-api-reference.md):

**New section after table of contents** providing quick reference for:
- Correct PlacementType import path
- Other common enum imports (ButtonType, ListMode, MessageType, ValueState)
- POD 2.0 core imports (PodContext, ModelPath from context/ NOT model/)
- Property editor imports (correct paths)
- ModelPath constant examples with array vs single object clarification

**Key highlights:**
- PlacementType from `"sap/m/PlacementType"` (not sap/ui/core/library)
- PodContext/ModelPath from `context/` path (not `model/`)
- Work list paths are PLURAL and return arrays
- Links to full ModelPath constants section

---

### 3. Pre-Generation Validation Checklist (NEW)

Added to [SKILL.md](SKILL.md) before "Final Reminders" section:

**Comprehensive checklist for validating code before generation:**

✅ **Import Validation:**
- PlacementType from correct path
- PodContext/ModelPath from context/ not model/
- Other common enum imports

✅ **ModelPath Constants Validation:**
- Work list paths are PLURAL
- Check exact names in API reference before use
- Understand array vs single object returns

✅ **i18n Setup Validation:**
- Framework-driven pattern with static getI18nModel()
- No manual ResourceBundle/ResourceModel loading

✅ **Lifecycle Validation:**
- Subscribe/unsubscribe with correct paths
- Defensive array handling with Array.isArray()
- Correct callback parameter order (value, path)

**Quick checklist format** with checkboxes for rapid validation.

---

### 4. Quick Start Warning Box (NEW)

Added prominent warning box at start of "Quick Start" section in [SKILL.md](SKILL.md):

```
╔════════════════════════════════════════════════════════════════════════════╗
║ ⚠️  CRITICAL IMPORT CHECKS                                                 ║
║  PlacementType → "sap/m/PlacementType" (direct import)                    ║
║  ModelPath.SelectedWorkListItems → Array (note plural!)                   ║
║  Always verify ModelPath constants in references/pod2-api-reference.md    ║
╚════════════════════════════════════════════════════════════════════════════╝
```

Visual warning ensures developers see critical validation rules immediately.

---

## 📋 FILES CHANGED

### 1. references/common-mistakes.md
- **Lines 678-856**: Added Mistake #12 (Wrong PlacementType Import)
- **Lines 857-1034**: Added Mistake #13 (Wrong ModelPath Constants)
- Renumbered subsequent mistakes (#13 → #14, #14 → #15, #15 → #16)
- Complete examples showing ✅ correct vs ❌ wrong patterns
- Explanation of why errors happen
- Prevention strategies
- Links to API reference

**Key additions:**
- PlacementType import from sap.m, not sap.ui.core
- Other common enum imports (ButtonType, ListMode, MessageType)
- ModelPath constant examples (plural vs singular)
- Array validation patterns
- How to find correct constants in API reference

### 2. references/pod2-api-reference.md
- **Lines 9-115**: Added "Common Imports Reference" section after TOC
- Updated TOC numbering (added item #1)
- Control & Enum Imports subsection
- POD 2.0 Core Imports subsection
- Property Editors & Metadata subsection
- Correct ModelPath Constants subsection with warnings

**Subsections:**
1. **Control & Enum Imports** - PlacementType, ButtonType, ListMode, MessageType, ValueState
2. **POD 2.0 Core Imports** - PodContext, ModelPath, Widget base classes, i18n
3. **Property Editors** - Correct import paths for editors and metadata
4. **Correct ModelPath Constants** - Plural names, array returns, common mistakes

### 3. SKILL.md
- **Lines 521-547**: Added warning box to "Quick Start" section
- **Lines 1220-1323**: Added "Pre-Generation Validation Checklist" before "Final Reminders"
- Updated "Final Reminders" section references

**Pre-Generation Checklist sections:**
1. Import Validation (PlacementType, PodContext, enums)
2. ModelPath Constants Validation (plural names, arrays)
3. i18n Setup Validation (framework-driven pattern)
4. Lifecycle Validation (subscriptions, defensive coding)
5. Quick Checklist (checkbox format)

---

## 📊 STATISTICS

**New content added:**
- common-mistakes.md: +177 lines (2 new mistakes with complete examples)
- pod2-api-reference.md: +106 lines (new Common Imports Reference section)
- SKILL.md: +130 lines (validation checklist + warning box)
- **Total**: +413 lines of validation documentation

**Purpose:**
Prevent the two most common POD 2.0 plugin errors:
1. Wrong PlacementType import path (causes undefined enum values)
2. Wrong ModelPath constants (singular vs plural, non-existent constants)

---

## 🎯 IMPACT

### Who benefits:
- **All POD 2.0 plugin developers** - prevents common import/ModelPath errors
- **New developers** - clear guidance on correct patterns from the start
- **Experienced developers** - quick reference for correct imports

### Error prevention:
1. ✅ PlacementType import from correct module
2. ✅ ModelPath constants use exact names (plural for arrays)
3. ✅ Defensive type checking with Array.isArray()
4. ✅ Pre-generation validation checklist
5. ✅ Visual warning box in Quick Start

### Why this is important:
- Wrong PlacementType import causes undefined enum values → runtime errors
- Wrong ModelPath constants cause subscriptions to never fire → silent failures
- These are the #1 and #2 most common errors in POD 2.0 plugin development
- Both errors are preventable with correct documentation

---

## ✅ VERIFICATION

Documentation structure:
1. ✅ Common mistakes section updated with #12 and #13
2. ✅ API reference has new "Common Imports" section at top
3. ✅ SKILL.md has validation checklist before "Final Reminders"
4. ✅ Quick Start has prominent warning box
5. ✅ All cross-references link correctly
6. ✅ Code examples show ✅ correct and ❌ wrong patterns
7. ✅ Explanations include "why this happens"

---

## 📚 SEE ALSO

- [references/common-mistakes.md](references/common-mistakes.md) - Complete mistake #12 and #13 documentation
- [references/pod2-api-reference.md](references/pod2-api-reference.md) - Common Imports Reference section
- [SKILL.md - Pre-Generation Validation](SKILL.md#pre-generation-validation-checklist) - Complete checklist
- [SKILL.md - Quick Start](SKILL.md#quick-start-tldr) - Warning box at top

---

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
