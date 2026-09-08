---
name: pod-plugin
description: "Create SAP Digital Manufacturing POD 2.0 plugins. Trigger when users mention: POD plugins, POD widgets, POD 2.0, SAP DM customization, TableWidget, ControlWidget, LayoutWidget, PodContext, extension.json, work center plugins. Expert in ES6 class-based POD 2.0 development. CRITICAL: Never creates webapp/ folder. Files generated at working directory root."
version: 30.6.0
author: Claude
tags: [sap, digital-manufacturing, pod, plugin, pod2, widget-architecture, production-patterns, delegate-patterns, no-webapp-folder, memory-leak-prevention, api-payload-verification]
compatibility:
  environment: SAP Business Technology Platform (BTP) with SAP Digital Manufacturing
  requirements:
    - SAP DM POD Designer access
    - Extension Center upload permissions
---

# POD 2.0 Plugin Development

Expert guide for SAP Digital Manufacturing POD 2.0 plugin development.

## ⚠️ REQUIRED READING Before Generation

**YOU MUST read these files before generating ANY plugin code:**

1. **FIRST**: [QUICK-REFERENCE.md](references/QUICK-REFERENCE.md) - Essential patterns (PodContext getters, imports, etc.)
2. **SECOND**: [widget-patterns.md](references/widget-patterns.md) - Get the correct widget template
3. **THIRD**: [common-mistakes.md](references/common-mistakes.md) - Get the correct extension.json format (Mistake #14)
4. **FOURTH**: [PATTERN-INDEX.md](references/PATTERN-INDEX.md) - Verify pattern selection

**Additional References** (read as needed):
- [QUICK-REFERENCE.md](references/QUICK-REFERENCE.md) - Quick reference card for common patterns
- [action-patterns.md](references/action-patterns.md) - ⭐ **For Actions**: SfcExecutionAction base class, property editors, execute() patterns, code quality rules, production checklist, testing
- [tablewidget-complete.md](references/tablewidget-complete.md) - For TableWidget only
- [tablecell-patterns.md](references/tablecell-patterns.md) - For custom table cells
- [delegate-architecture.md](references/delegate-architecture.md) - For delegate usage
- [notification-patterns.md](references/notification-patterns.md) - WebSocket subscriptions, `PodNotificationWebSocket`, `ManagedSubscription`, `Filter` (fluent filter API), `SubscriptionContext`
- [error-handling.md](references/error-handling.md) - Error decision matrix
- [pod2-api-reference.md](references/pod2-api-reference.md) - PodContext, ModelPath API (comprehensive)
- [podcontext-api-reference.md](references/podcontext-api-reference.md) - ⭐ PodContext deep reference (source-derived): all getters/setters, subscribe/unsubscribe, custom paths, WorkListItem/OperationActivity object shapes, common mistakes
- [delegates-reference.md](references/delegates-reference.md) - ⭐ All data delegates (source-derived): WorkListDelegate, OperationActivityDelegate, ActivityConfirmationDelegate, QuantityConfirmationDelegate, WorkInstructionDelegate, QualityInspectionDelegate, ComponentConsumptionDelegate — public APIs, auto-subscribe behaviours, common mistakes
- [utilities-reference.md](references/utilities-reference.md) - ⭐ All utilities (source-derived): DateTimeUtils (locale formatting methods DO exist), Utilities (array/sort/module/DOM helpers), Logger, ValidationUtils, DialogUtils, LanguageUtils
- [sapdm-api-reference.md](references/sapdm-api-reference.md) - SAP DM REST APIs
- [sap-dm-languages.md](references/sap-dm-languages.md) - All 28 supported languages with ISO codes
- [sapui5-control-apis.md](references/sapui5-control-apis.md) - SAPUI5 control quirks and styling patterns

---

## extension.json Template

**CRITICAL**: ONLY `widgets` and `actions` arrays are supported. NO other fields.

```json
{
  "widgets": [
    {
      "modulePath": "namespace/widget/WidgetName",
      "type": "namespace.widget.WidgetName"
    }
  ],
  "actions": []
}
```

**Rules:**
- ✅ ONLY `widgets` array and `actions` array
- ✅ `modulePath` uses forward slashes (e.g., `custom/pod2/myproject/widget/MyWidget`)
- ✅ `type` uses dots (e.g., `custom.pod2.myproject.widget.MyWidget`)
- ❌ NO `id`, `name`, `version`, `vendor`, `description`, `dependencies`, or `content` fields
- ❌ NO nested objects - only arrays with module definitions

**Why**: SAP DM Extension Center only parses `widgets` and `actions` arrays. Any other fields cause parsing errors.

See [Mistake #14](references/common-mistakes.md#mistake-14) for full details and examples.

---

## Critical Rules

### 1. ⚠️ MANDATORY: Required Information (MUST ASK FIRST!)

**🛑 STOP! Before generating ANY files, you MUST ask the user for:**

1. **Namespace** - e.g., `custom/pod2/myproject`
2. **Plugin Name** - e.g., `ProductionStatus`
3. **Category** - Default: `Custom`
4. **i18n Languages** - Which languages to support:
   - **Option A**: Core languages only (English, German, Chinese, Japanese - 4 files) - DEFAULT
   - **Option B**: Extended set (English, German, French, Spanish, Portuguese, Russian, Chinese, Japanese, Korean - 9 files)
   - **Option C**: All 28 SAP DM supported languages (see [sap-dm-languages.md](references/sap-dm-languages.md))
   - **Option D**: Custom selection (user specifies which languages)

**DO NOT generate files until ALL four questions are answered.**

**Even if the user provides detailed requirements in their prompt, you MUST still explicitly ask for these four pieces of information. Do NOT infer or assume plugin name, category, or languages from the user's description.**

### 2. Component Type Selection

**Widget or Action?**

| Scenario | Component | Why |
|----------|-----------|-----|
| User clicks button to trigger SFC start/complete/signoff | **Action** | Stateless execution, no ongoing UI |
| Display production data, react to selection changes | **Widget** | Stateful, needs lifecycle |
| Button-triggered operation AND display of results | **Action + Widget** | Action updates context, Widget reacts |

**Action Base Class:**

| Operation Type | Base Class |
|----------------|-----------|
| Any SFC operation (start, complete, signoff, split) | `SfcExecutionAction` ⭐ |
| Phase operations | `PhaseExecutionAction` |
| Non-SFC custom operations | `Action` |

**Widget Base Class:**

| Need | Base Class | Spread Parent Config? |
|------|------------|----------------------|
| Single control (button, input) | `ControlWidget` | ❌ NO |
| Container for widgets | `LayoutWidget` | ✅ YES |
| Data table | `TableWidget` | ✅ YES |
| Business logic only | `ContentHandler` | N/A |

**See**: [action-patterns.md](references/action-patterns.md) for full Action development guide.

### 3. Correct Import Paths

```javascript
// ✅ CORRECT - Verified against official SAP plugins
"sap/dm/dme/pod2/context/PodContext"
"sap/dm/dme/pod2/context/ModelPath"
"sap/dm/dme/pod2/context/data/WorkListDelegate"  // All delegates use context/data/
"sap/dm/dme/pod2/model/I18nResourceModel"
"sap/dm/dme/pod2/api/ApiClient"
"sap/dm/dme/pod2/Logger"

// ❌ WRONG - Common mistakes
"sap/dm/dme/pod2/model/PodContext"      // Wrong: model/ instead of context/
"sap/dm/dme/pod2/delegate/..."          // Wrong: delegate/ instead of context/data/
```

### 4. ⭐ CRITICAL: Getting Selected Operation, Resource, and SFC Data

**This is the MOST COMMON pattern in POD 2.0 - used by 90% of official SAP widgets.**

**IMPORTANT**: Plugins must be **adaptive** to work in different POD configurations. Some PODs have OperationActivity widgets, some only have WorkList widgets. Your code must handle both!

#### Adaptive Pattern (Recommended - Works in All PODs)

```javascript
// ✅ CORRECT - Adaptive pattern that works in ANY POD configuration
async _onButtonPress() {
    // Get filtered resources (common to all configurations)
    const aFilterResources = PodContext.getFilterResources();
    const sResource = aFilterResources?.[0]?.resource || null;

    // Try to get from OperationActivity widget first (if it exists)
    const oLastSelectedOperation = PodContext.getLastSelectedOperationActivity();
    const oLastSelectedWorkListItem = PodContext.getLastSelectedWorkListItem();

    let sSfc, sOperation, sWorkCenter, sStepId;

    if (oLastSelectedOperation && oLastSelectedWorkListItem) {
        // Pattern 1: POD has both OperationActivity + WorkList (most common in production)
        sSfc = oLastSelectedWorkListItem.sfc;
        sOperation = oLastSelectedOperation.operationActivity;
        sWorkCenter = oLastSelectedOperation.workCenter;
        sStepId = oLastSelectedOperation.stepId;
        
    } else if (oLastSelectedWorkListItem) {
        // Pattern 2: POD has only WorkList widget
        sSfc = oLastSelectedWorkListItem.sfc;
        sOperation = oLastSelectedWorkListItem.operationActivity;
        sWorkCenter = oLastSelectedWorkListItem.workCenter;
        sStepId = oLastSelectedWorkListItem.stepId;
        
    } else {
        // Pattern 3: Fallback to array
        const aSelectedItems = PodContext.getSelectedWorkListItems();
        if (Array.isArray(aSelectedItems) && aSelectedItems.length > 0) {
            const oItem = aSelectedItems[0];
            sSfc = oItem.sfc;
            sOperation = oItem.operationActivity;
            sWorkCenter = oItem.workCenter;
            sStepId = oItem.stepId;
        } else {
            MessageHistory.showError("No selection");
            return;
        }
    }

    // Validate
    if (!sSfc || !sOperation) {
        MessageHistory.showError("Missing SFC or Operation");
        return;
    }

    // Build request
    const oRequest = {
        plant: PodContext.getPlant(),
        sfc: sSfc,
        operation: sOperation
    };
    
    // Add optional fields
    if (sResource) oRequest.resource = sResource;
    if (sWorkCenter) oRequest.workCenter = sWorkCenter;
    if (sStepId) oRequest.stepId = sStepId;
    
    await ApiClient.sfc.sfcStart(oRequest);
}
```

#### POD Configuration Matrix

| POD Configuration | What's Available | Pattern to Use |
|-------------------|------------------|----------------|
| OperationActivity + WorkList | Both widgets present | Get operation from `getLastSelectedOperationActivity()`, SFC from `getLastSelectedWorkListItem()` |
| WorkList Only | Only WorkList widget | Get everything from `getLastSelectedWorkListItem()` or array |
| OperationActivity Only | Only OperationActivity widget | Get everything from `getSelectedOperationActivities()` |

#### Why Adaptive Patterns Matter

```javascript
// ❌ BAD - Only works if POD has OperationActivity widget
const oOp = PodContext.getLastSelectedOperationActivity();
if (!oOp) return;  // 💥 Fails in WorkList-only PODs!

// ✅ GOOD - Works in any POD configuration
const oOp = PodContext.getLastSelectedOperationActivity();
const oWL = PodContext.getLastSelectedWorkListItem();

if (oOp && oWL) {
    // Both widgets present - use operation data from OperationActivity
} else if (oWL) {
    // WorkList only - use operation data from WorkListItem
} else {
    // Fallback to array
}
```

**Key Methods:**
- `PodContext.getLastSelectedOperationActivity()` - Get selected **operation** (if OperationActivity widget exists)
- `PodContext.getLastSelectedWorkListItem()` - Get selected **SFC** (if WorkList widget exists)
- `PodContext.getFilterResources()` - Get filtered **resources** (array, always available)
- `PodContext.getFilterSfcs()` - Get filter bar **SFC** values (`string[]`)
- `PodContext.getFilterProcessLot()` - Get filter bar **process lot** value (`string`) — use in process-industry PODs
- `PodContext.getIndustryType()` - Returns `"DISCRETE"` or `"PROCESS"` — use to branch on industry type
- `PodContext.getWhenAvailable(path)` - Returns a **Promise** that resolves when a model path becomes non-null — use during init to avoid race conditions

**❌ Common Mistakes:**
```javascript
// ❌ WRONG - Assumes OperationActivity widget always exists
const oOp = PodContext.getLastSelectedOperationActivity();
const sOperation = oOp.operationActivity;  // 💥 Fails if no OperationActivity widget!

// ❌ WRONG - Don't use worklist item for operation when OperationActivity exists
const sOperation = oWorkListItem.operationActivity;  // May be stale if OperationActivity widget present!

// ❌ WRONG - Don't use operation.resource (often null)
const sResource = oLastSelectedOperation.resource;

// ✅ CORRECT - Use getFilterResources()
const sResource = PodContext.getFilterResources()?.[0]?.resource;
```

**See**: [pod2-api-reference.md - Getting Selected Data](references/pod2-api-reference.md#critical-getting-selected-operation-resource-and-worklist-data) for complete examples.

### 5. ⚠️ CRITICAL: API Payload Verification

**MANDATORY**: Before using ANY SAP DM API, you MUST verify the correct request payload format by reading [sapdm-api-reference.md](references/sapdm-api-reference.md).

**❌ WRONG - Guessing API payload format:**
```javascript
// This will fail! API expects different format
await ApiClient.sfc.sfcStart({
    plant: PodContext.getPlant(),
    sfc: sSfc,           // ❌ WRONG - API expects array format
    operation: sOperation // ❌ WRONG - API expects "operationActivity"
});
```

**✅ CORRECT - Verify payload in sapdm-api-reference.md first:**
```javascript
// Read sapdm-api-reference.md Section 1: Shop Floor Control (SFC) API
// Found correct format (ApiClient.sfc.sfcStart):
// {
//   "plant": "PLANT_1",
//   "sfcs": ["SFC001"],
//   "operation": "OPER_1,1",
//   "resource": "RESOURCE_1",
//   "autoAssembleEnabled": true
// }

await ApiClient.sfc.sfcStart({
    plant: PodContext.getPlant(), // ✅ Required
    sfcs: [sSfc],                 // ✅ Array of strings
    operation: sOperation,        // ✅ "operation" not "operationActivity"
    resource: sResource,          // ✅ Optional
    autoAssembleEnabled: true     // ✅ Default true
});
```

**CRITICAL Notes:**
- ApiClient wraps the REST API and uses DIFFERENT property names
- REST API may use `operationActivity`, but ApiClient uses `operation`
- Always verify the ACTUAL format by checking real plugin usage or network traffic

**MANDATORY PRE-GENERATION STEP:**

When generating plugin code that calls SAP DM APIs:

1. **STOP** - Do NOT write API call code yet
2. **READ** [sapdm-api-reference.md](references/sapdm-api-reference.md) to find the exact endpoint
3. **VERIFY** the correct request payload structure
4. **ONLY THEN** write the API call code using the verified format

**Why This Matters:**

API errors like `400 Bad Request: sfcOrProcessLot.missing` or `400 Bad Request: parameter.invalid` occur when payload format is wrong. These errors:
- Block functionality completely
- Are hard to debug (cryptic error messages)
- Require ZIP redeployment to fix
- Frustrate users who expected working code

**Common API Format Patterns to Verify:**

| API | Check For |
|-----|-----------|
| SFC APIs | Array format (`sfcs: [{ sfc: "..." }]`) vs single `sfc` property |
| Operation | `operationActivity` vs `operation` property name |
| Date/Time | ISO 8601 format vs timestamp vs date-only |
| Quantity | Decimal vs integer, UOM requirements |
| Plant | Required vs optional, exact property name |

**Pre-Generation API Verification Checklist:**

- [ ] Identified which SAP DM API endpoint will be called
- [ ] Read [sapdm-api-reference.md](references/sapdm-api-reference.md) section for that API
- [ ] Copied exact request payload structure from reference
- [ ] Verified all required vs optional fields
- [ ] Verified property names (e.g., `operationActivity` not `operation`)
- [ ] Verified data types (array vs string, object vs primitive)
- [ ] Only after ALL verification: wrote API call code

**See**: [sapdm-api-reference.md](references/sapdm-api-reference.md) for complete API reference with request/response examples.

### 6. ModelPath Constants (ALL PLURAL)

```javascript
ModelPath.SelectedWorkListItems       // ✅ PLURAL (array)
ModelPath.SelectedOperationActivities // ✅ PLURAL (array)
ModelPath.FilterResources             // ✅ PLURAL
ModelPath.LastSelectedWorkListItem    // ✅ Singular (single item getter only)
```

### 7. i18n Pattern - Language Options

**SAP Digital Manufacturing supports 28 languages** - see [sap-dm-languages.md](references/sap-dm-languages.md) for complete list and ISO codes.

**Language Options (ask user which to create):**

**Option A - Core Languages (DEFAULT):**
- English (en), German (de), Chinese Simplified (zh), Japanese (ja)

**Option B - Extended Set:**
- English (en), German (de), French (fr), Spanish (es), Portuguese (pt), Russian (ru), Chinese Simplified (zh), Japanese (ja), Korean (ko)

**Option C - All 28 SAP DM Languages:**
- Bulgarian (bg), Chinese Simplified (zh), Chinese Traditional (zh_TW), Croatian (hr), Czech (cs)
- Danish (da), Dutch (nl), English (en), French (fr), German (de)
- Hungarian (hu), Italian (it), Japanese (ja), Korean (ko), Lithuanian (lt)
- Polish (pl), Portuguese/Brazilian (pt), Romanian (ro), Russian (ru), Serbian Latin (sr)
- Slovak (sk), Slovenian (sl), Spanish (es), Swedish (sv), Thai (th)
- Turkish (tr), Ukrainian (uk), Vietnamese (vi)

**Option D - Custom Selection:** User specifies exact languages needed

```javascript
import I18nResourceModel from "sap/dm/dme/pod2/model/I18nResourceModel";

class YourWidget extends Widget {
    static #oI18nModel = new I18nResourceModel({
        bundleName: "your.namespace.i18n.i18n"  // Dots, not slashes!
    });
    
    static getI18nModel() { return this.#oI18nModel; }
    
    _createView() {
        return new Button({
            text: this.getI18nText("key")  // ✅ Method call, NOT binding!
        });
    }
}
```

**CRITICAL**: 
- Use `this.getI18nText()` in `_createView()`. Never use `{i18n>key}` bindings.
- **Ask user which languages to create** - default to Core (4 files) if not specified
- See [sap-dm-languages.md](references/sap-dm-languages.md) for all 28 supported languages and file naming conventions

### 8. Memory Leak Prevention

```javascript
onInit() {
    PodContext.subscribe(ModelPath.SelectedWorkListItems, this._onSelectionChange, this);
}

onExit() {
    // REQUIRED: Clean up subscriptions
    PodContext.unsubscribe(ModelPath.SelectedWorkListItems, this._onSelectionChange, this);
    // Or use: PodContext.unsubscribeAll(this);
    super.onExit();
}
```

For **WebSocket / real-time** subscriptions use `PodNotificationWebSocket` or `ManagedSubscription` (not PodContext). Build subscription scope with the `Filter` class fluent API (`.equals()`, `.and()`, `.equalsAny()`, etc.). See [notification-patterns.md](references/notification-patterns.md) for the full pattern including `Filter`, `ManagedSubscription.destroy()`, and `SubscriptionContext.unsubscribe()`.

See [Mistake #3](references/common-mistakes.md#mistake-3) for complete patterns.

---

## File Structure

```
<working-directory>/     # User is already in namespace folder
├── extension.json       # Widget registration
├── widget/
│   └── YourWidget.js
└── i18n/                # Language files (based on user selection)
    ├── i18n_en.properties      # Always include English
    ├── i18n_de.properties      # Core languages
    ├── i18n_zh.properties      # (Default: 4 files)
    ├── i18n_ja.properties
    └── ... (additional languages if requested)
```

**Default Language Set (Core - 4 files):**
- English (en), German (de), Chinese Simplified (zh), Japanese (ja)

**Extended Set (9 files):**
- Add: French (fr), Spanish (es), Portuguese (pt), Russian (ru), Korean (ko)

**All 28 Languages (if requested):**
- See [sap-dm-languages.md](references/sap-dm-languages.md) for complete list

**CRITICAL**: 
- Never create namespace folders. Generate files at root.
- **Ask user which languages to create** - default to Core (4 files)
- Always include English (en) at minimum

---

## ⚠️ MANDATORY Pre-Generation Steps

**🛑 STOP! Before generating ANY files, complete these steps in order:**

### Step 1: ⚠️ REQUIRED - Collect User Information (DO THIS FIRST!)

**YOU MUST ask the user FOUR questions before proceeding:**

- [ ] **Question 1**: What is your namespace? (e.g., `custom/pod2/myproject`)
- [ ] **Question 2**: What is the plugin name? (e.g., `ProductionStatus`, `StartSfcButton`)
- [ ] **Question 3**: What category? (default: `Custom`)
- [ ] **Question 4**: Which languages for i18n? 
  - Core (en, de, zh, ja - 4 files) - **DEFAULT**
  - Extended (en, de, fr, es, pt, ru, zh, ja, ko - 9 files)
  - All 28 SAP DM languages (see [sap-dm-languages.md](references/sap-dm-languages.md))
  - Custom selection (user specifies)

**❌ DO NOT:**
- Infer plugin name from the user's description
- Assume category is "Custom" without asking
- Create all 28 language files by default
- Skip these questions even if the user provides detailed requirements

**✅ DO:**
- Explicitly ask all four questions
- Wait for user answers before proceeding
- Use the exact answers provided by the user
- Default to "Core languages" if user doesn't specify

### Step 2: Read Reference Files
- [ ] **Determine component type**: Widget, Action, or both? (see Component Type Selection above)
- [ ] **IF building an Action**: Read `references/action-patterns.md` for SfcExecutionAction template, property editors, and code quality rules
- [ ] **IF building a Widget**: Read `references/widget-patterns.md` for the complete widget class template
- [ ] Read `references/common-mistakes.md` - specifically Mistake #14 for extension.json format
- [ ] Read `references/PATTERN-INDEX.md` to select the correct pattern
- [ ] **IF plugin calls SAP DM APIs**: Read `references/sapdm-api-reference.md` to verify correct API payload format
- [ ] **Verified correct data types**: Array of strings vs array of objects vs single values

### Step 3: Verify API Payloads (IF USING APIS)

**🛑 CRITICAL: If plugin will call SAP DM APIs (ApiClient.sfc.*, ApiClient.material.*, etc.):**

- [ ] Identified which SAP DM API endpoints will be called
- [ ] Read [sapdm-api-reference.md](references/sapdm-api-reference.md) section for each API
- [ ] Verified exact request payload structure (property names, data types, array vs single)
- [ ] Verified required vs optional fields
- [ ] **Example**: For `ApiClient.sfc.sfcStart()`:
  - ✅ Verified it requires `plant: PodContext.getPlant()` (required)
  - ✅ Verified it requires `sfcs: ["SFC001"]` (array of STRINGS, not objects)
  - ✅ Verified it requires `operation` (not `operationActivity` - ApiClient differs from REST API!)
  - ✅ Verified `resource` is optional
  - ✅ Verified `autoAssembleEnabled` defaults to true

**Skip this step if plugin does NOT call SAP DM APIs** (e.g., display-only widgets).

### Step 4: Verify Critical Patterns
- [ ] extension.json uses ONLY `widgets` and `actions` arrays (no other fields)
- [ ] Widget class template matches the pattern from widget-patterns.md
- [ ] Import paths verified: use `context/` not `model/`, use `context/data/` for delegates
- [ ] ModelPath constants are plural (SelectedWorkListItems, not SelectedWorkListItem)
- [ ] **Correct i18n language files** will be created based on user selection (see [sap-dm-languages.md](references/sap-dm-languages.md))

**DO NOT generate files until ALL checkboxes are completed.**

---

## Pre-Generation Checklist

**Code Patterns:**
- [ ] Import paths use `context/` not `model/` for PodContext/ModelPath
- [ ] Delegate paths use `context/data/` not `delegate/`
- [ ] ModelPath constants are plural (SelectedWorkListItems)
- [ ] **Uses ADAPTIVE pattern**: tries `getLastSelectedOperationActivity()` first, falls back to `getLastSelectedWorkListItem()`, then array
- [ ] **Plugin works in ALL POD configurations**: OperationActivity+WorkList, WorkList-only, OperationActivity-only
- [ ] **Extracts operation from OperationActivity when available**, falls back to WorkListItem when not
- [ ] **Extracts resource from getFilterResources() array**, not operation object
- [ ] Root control ID is exactly `oConfig.id` (no suffix)
- [ ] **Root control is `CustomPanel`** — NOT VBox/HBox/Panel/FlexBox (see Mistake #21)
- [ ] **If using `sap/ui/core/HTML`**: content with `{` is set via setter or `afterRendering` DOM injection, NOT constructor property bag (see Mistake #32)
- [ ] `onExit()` exists if `onInit()` subscribes
- [ ] i18n uses method calls, not bindings
- [ ] **User-selected languages** confirmed (default: Core 4 languages if not specified)

**API Verification (IF USING APIS):**
- [ ] **Read sapdm-api-reference.md** for each API endpoint used
- [ ] **Verified request payload structure** (property names match exactly)
- [ ] **Verified data types** (array vs string, object vs primitive)
- [ ] **Verified required vs optional fields**
- [ ] **Example verified**: `sfcs: ["SFC001"]` not `sfcs: [{ sfc: "..." }]`
- [ ] **Example verified**: `operation` not `operationActivity` (ApiClient differs from REST API)

---

## Code Generation Guidelines (MANDATORY)

Apply these rules to ALL generated code — widgets AND actions.

### Functions Max 20 Lines

Break long methods into focused private helpers. If a method exceeds 20 lines, extract.

### Guard Clauses First

```javascript
// ✅ GOOD: early exit, happy path at lowest indentation
async execute() {
    if (!this.#hasOperation()) return;
    if (!this.#hasSelection()) return;
    await this.#performExecution();
}

// ❌ BAD: nested conditions
async execute() {
    if (this.#hasOperation()) {
        if (this.#hasSelection()) { /* Deep nesting */ }
    }
}
```

### Max 2 Nesting Levels

Use filter/map chains instead of nested loops.

### No Abbreviations

```javascript
// ✅ GOOD
const selectedWorkListItems = PodContext.getSelectedWorkListItems();

// ❌ BAD
const selWlItems = PodContext.getSelectedWorkListItems();
```

### Private Methods Use `#`

```javascript
class MyAction extends SfcExecutionAction {
    #logger = Logger.getLogger("...");
    async execute() { ... }           // public
    #validatePreconditions() { ... }  // private
    #buildRequest() { ... }           // private
}
```

### Type Safety

```javascript
// ✅ GOOD: use unknown + instanceof
#handleError(error) {
    if (error instanceof Error) {
        MessageHistory.showError(error.message);
    } else {
        MessageHistory.showError(this.getI18nText("error.unknown"));
    }
    throw error;
}
// ❌ BAD: error: any
```

### Separation of Concerns

`execute()` / `onInit()` should be orchestration only — delegate to private helper methods.

---

## Migration Warning

If user asks to convert POD 1.0 to POD 2.0:

**Recommend RE-ARCHITECTING, not direct conversion.** POD 1.0 (XML views, controllers) and POD 2.0 (ES6 classes, programmatic views) are fundamentally different. Reuse business logic, redesign UI using POD 2.0 patterns.

---

## Post-Generation

After creating plugin:
1. Display namespace notification
2. Display AI-generated code warning
3. **Verify correct i18n language files were created** (based on user selection)
4. Create deployment zip if requested

---

## POST-Generation Verification

**After generating files, VERIFY the following:**

### 1. extension.json Validation

```bash
# Must contain ONLY these keys
grep -E '"(widgets|actions)"' extension.json

# Must NOT contain these keys (should return no results)
grep -E '"(id|name|version|vendor|description|dependencies|content)"' extension.json
```

Expected format:
```json
{
  "widgets": [{
    "modulePath": "namespace/widget/WidgetName",
    "type": "namespace.widget.WidgetName"
  }],
  "actions": []
}
```

### 2. Widget Class Validation

- [ ] Import paths use `context/` not `model/` for PodContext/ModelPath
- [ ] Delegate imports use `context/data/` not `delegate/`
- [ ] **Uses ADAPTIVE pattern for getting selected data** (tries OperationActivity first, falls back to WorkList, then array)
- [ ] **Plugin handles all POD configurations** (OperationActivity+WorkList, WorkList-only, OperationActivity-only)
- [ ] Prefers `operationActivity` from OperationActivity object when available, falls back to WorkListItem
- [ ] Uses `getFilterResources()?.[0]?.resource` for resource data (not operation.resource)
- [ ] Root control ID is exactly `oConfig.id` (no suffix like `oConfig.id + "-button"`)
- [ ] **Root control returned from `_createView()` is `CustomPanel`** — if VBox/HBox/Panel is the root, POD Designer will log "not draggable" and the widget cannot be repositioned (Mistake #21)
- [ ] **If `sap/ui/core/HTML` is used**: HTML/CSS content with `{` characters is NOT passed through the constructor property bag — use `oHtml.setContent(s)` setter or inject via `afterRendering` → `getDomRef().innerHTML` (Mistake #32)
- [ ] `onExit()` method exists if `onInit()` has PodContext.subscribe() calls
- [ ] i18n uses `this.getI18nText("key")` not `{i18n>key}` bindings
- [ ] ModelPath constants are plural (e.g., `SelectedWorkListItems`)

### 3. File Structure Validation

```bash
# Verify folder structure
ls -la extension.json widget/ i18n/

# Count i18n language files
ls i18n/*.properties | wc -l
```

Expected structure (default Core languages):
```
<working-directory>/
├── extension.json
├── widget/
│   └── WidgetName.js
└── i18n/
    ├── i18n_en.properties   # English (always include)
    ├── i18n_de.properties   # German
    ├── i18n_zh.properties   # Chinese Simplified
    └── i18n_ja.properties   # Japanese
```

If user requested Extended set (9 files):
```
└── i18n/
    ├── i18n_en.properties
    ├── i18n_de.properties
    ├── i18n_fr.properties
    ├── i18n_es.properties
    ├── i18n_pt.properties
    ├── i18n_ru.properties
    ├── i18n_zh.properties
    ├── i18n_ja.properties
    └── i18n_ko.properties
```

If user requested all 28 languages: See [sap-dm-languages.md](references/sap-dm-languages.md) for complete list.

### 4. i18n Validation

- [ ] **Requested language files created** (based on user selection)
- [ ] All files have identical keys
- [ ] No empty translation files (all files have content)
- [ ] Traditional Chinese uses `zh_TW` (underscore, not hyphen) if included
- [ ] English (en) always included at minimum

**If ANY validation fails, regenerate the affected file with corrections.**

---

## Deployment ZIP Creation

When creating a deployment ZIP file:

**Windows (PowerShell):**
```bash
powershell -Command "Compress-Archive -Path extension.json,widget,i18n -DestinationPath <PluginName>_deployment.zip -Force"
```

**Unix/Linux/Mac:**
```bash
zip -r <PluginName>_deployment.zip extension.json widget/ i18n/
```

**CRITICAL**: 
- Always use PowerShell `Compress-Archive` on Windows platform
- Never attempt `tar` command for ZIP creation
- ZIP file should contain: extension.json, widget/ folder, i18n/ folder
- Name format: `<PluginName>_deployment.zip`
