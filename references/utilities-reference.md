# POD 2.0 Utilities Reference

Authoritative reference derived from source code in `sap/dm/dme/pod2/`.

All utilities are **static classes** — call methods on the class, never on an instance.

---

## DateTimeUtils

**Import:** `sap/dm/dme/pod2/DateTimeUtils`

Utility class for all date/time operations in the **plant time zone**. Always use `DateTimeUtils.now()` instead of `new Date()` when the timezone matters.

### Full API

```javascript
// ✅ ALWAYS use instead of new Date() when timezone is relevant
DateTimeUtils.now()
// → UI5Date (current date/time in plant timezone)

// Start/end of day in plant timezone
DateTimeUtils.startOfDay(oDate?)   // → UI5Date at 00:00:00.000
DateTimeUtils.endOfDay(oDate?)     // → UI5Date at 23:59:59.999
// oDate: optional UI5Date or JS Date. If omitted, uses today.

// Format dates for display (locale-aware, respects plant timezone)
DateTimeUtils.localeDateTime(vValue, oDateFormatOptions?)  // → string  e.g. "Sep 7, 2026, 10:30:00 AM"
DateTimeUtils.localeDate(vValue, oDateFormatOptions?)      // → string  e.g. "Sep 7, 2026"
DateTimeUtils.localeTime(vValue, oDateFormatOptions?)      // → string  e.g. "10:30:00 AM"
// vValue: UI5Date, JS Date, or ISO date string. Returns "" for invalid dates.

// Parse OData date strings "/Date(1623668012060)/"
DateTimeUtils.fromODataDateString(sDate)  // → UI5Date | null

// Validate a date object
DateTimeUtils.isValidDate(oDate)  // → boolean

// Convert ISO string properties to Date objects (mutates object in place)
DateTimeUtils.parseDateProperties(oObject, "startDate", "endDate")
DateTimeUtils.parseDateProperties(aArray, "createdAt")  // also works on arrays

// Convert Date properties back to ISO strings (mutates in place)
DateTimeUtils.encodeDateProperties(oObject, "startDate", "endDate")
```

### ⚠️ Critical: localeDate/Time/DateTime DO EXIST

**Previous documentation was wrong.** The source code confirms that `localeDate()`, `localeTime()`, and `localeDateTime()` ARE real methods. The QUICK-REFERENCE.md note saying "localeDate/Time/DateTime do NOT exist and will crash" is **incorrect** — these methods exist and are fully implemented.

The correct restricted list is: only `now()`, `startOfDay()`, `endOfDay()`, `fromODataDateString()`, `isValidDate()`, `parseDateProperties()`, and `encodeDateProperties()` existed in the older API. The locale formatting methods were **added in a newer version**.

### Usage Pattern

```javascript
import DateTimeUtils from "sap/dm/dme/pod2/DateTimeUtils";

// Date range for a filter
const oStart = DateTimeUtils.startOfDay();
const oEnd   = DateTimeUtils.endOfDay();

// Display a date from the API
const sDisplay = DateTimeUtils.localeDateTime(oItem.createdAt);

// Parse an API response with date strings
DateTimeUtils.parseDateProperties(oResponse, "startDate", "completionDate");

// Format for API submission
DateTimeUtils.encodeDateProperties(oPayload, "scheduledStart");
```

---

## Utilities

**Import:** `sap/dm/dme/pod2/Utilities`

General-purpose helper functions for arrays, IDs, DOM, modules, and browser APIs.

### Array Helpers

```javascript
Utilities.isNonEmptyArray(aValues)     // → boolean: true if array with ≥1 element
Utilities.isEmptyOrNotArray(aValues)   // → boolean: true if empty or not an array
Utilities.getUniqueArrayElements(aValues) // → new array with duplicates removed (shallow)
```

**These are the standard null-safe array checks used throughout all SAP DM source code.** Use these instead of `Array.isArray(x) && x.length > 0`.

### Sorting

```javascript
Utilities.sortByProperty({
    objects: aItems,
    property: "sequence",
    descending: false   // optional
})
// → new sorted array (original not modified)
// Works with string (localeCompare), number, and Date properties
```

### ID Utilities

```javascript
Utilities.createId("myWidget-table")
// → "myWidget-table" if unused, or "myWidget-table1", "myWidget-table2" etc. if already taken
// Checks Element.getElementById — only considers already-created controls

Utilities.isValidId("myWidget-table")
// → boolean: must start with letter, contain only [A-Za-z0-9-_.]
```

### Object Equality

```javascript
Utilities.shallowEqual(oObjectA, oObjectB)
// → boolean: same keys, same values (===)
// Used internally by delegates for request deduplication
```

### DOM / SAPUI5 Element Helpers

```javascript
Utilities.isVisible(oElement)
// → boolean: true if control is visible AND not hidden in DOM
// Checks both SAPUI5 getVisible() AND actual CSS visibility

Utilities.scrollIntoView(oElement, oOptions?)
// Scrolls the element into view only if it is currently visible
// Default: { block: "nearest" } — minimum scroll, no movement if already in view
```

### Module Loading (Async)

```javascript
// Load a single module as a Promise
const oModule = await Utilities.requireModule("sap/dm/dme/pod2/some/Module");

// Load multiple modules — rejects if ANY fails
const [oA, oB] = await Utilities.requireAllModules(["path/A", "path/B"]);

// Load multiple modules — never rejects, returns Map<path, module|Error>
const oMap = await Utilities.requireSomeModules(["path/A", "path/B"]);
const oModule = oMap.get("path/A");
if (oModule instanceof Error) { /* failed */ }
```

### Browser / Platform Detection

```javascript
Utilities.isStandalone()  // → boolean: true if POD running outside FLP (Fiori Launchpad)
Utilities.isMac()         // → boolean: true if running on macOS
```

### Desktop Notifications

```javascript
// Request browser notification permission
Utilities.requestNotificationPermission();

// Show a desktop notification (requests permission if not yet granted)
await Utilities.showNotification("Scan Complete", {
    body: "SFC-001 has been processed",
    icon: "/path/to/icon.png",
    click: (oNotification, oEvent) => {
        // Handle notification click
        oNotification.close();
    }
});
```

### File Download

```javascript
// Trigger a browser file download
Utilities.downloadFile(sContent, "report.csv", "text/csv");
// sMimeType defaults to "application/octet-stream" if omitted
```

### Content Density

```javascript
// Apply SAP content density class to document body
// ContentDensity.Compact | ContentDensity.Cozy | ContentDensity.System
Utilities.applyContentDensity(ContentDensity.Compact);
// ContentDensity.System → Compact on non-touch, Cozy on touch devices
```

---

## Logger

**Import:** `sap/dm/dme/pod2/Logger`

POD 2.0's custom logger — replaces `sap/base/Log` with colour-coded, component-tagged console output.

### Getting an Instance

```javascript
// Always use getLogger — instances are cached by component path
class MyWidget extends Widget {
    // Private field pattern (preferred in widgets)
    #oLog = Logger.getLogger("custom.pod2.myplugin.widget.MyWidget");

    // OR module-level var (required when using HTML embedding — no # allowed)
    // var _oLog = Logger.getLogger("custom.pod2.myplugin.widget.MyWidget");
}
```

### Log Levels

| Level | Method | Console output |
|-------|--------|---------------|
| `FATAL` (0) | `fatal(msg, ...args)` | `console.error` in red |
| `ERROR` (1) | `error(msg, ...args)` | `console.error` in red |
| `WARN` (2) | `warn(msg, ...args)` | `console.warn` in orange |
| `INFO` (3) | `info(msg, ...args)` | `console.info` in blue |
| `DEBUG` (4) | `debug(msg, ...args)` | `console.debug` in grey |
| `TRACE` (5) | `trace(msg, ...args)` | `console.trace` in grey |

**Default level: DEBUG** — all messages at DEBUG and above are shown.

### Usage

```javascript
this.#oLog.info("Widget initialized");
this.#oLog.warn("No SFC selected, skipping");
this.#oLog.error("API call failed", oError);
this.#oLog.debug("Request payload:", oPayload);

// Additional args are passed to the console call (shown as expandable objects)
this.#oLog.info("Loaded items", aItems);
```

### Controlling Log Level

```javascript
// Per-instance (rarely needed)
oLog.setLevel(Logger.Level.WARN);   // only show WARN and above for this logger
oLog.getLevel();                    // → current level number

// Global default (affects all loggers without a specific level)
Logger.setDefaultLevel(Logger.Level.INFO);
```

### Log Level Constants

```javascript
Logger.Level.NONE   // -1
Logger.Level.FATAL  //  0
Logger.Level.ERROR  //  1
Logger.Level.WARN   //  2
Logger.Level.INFO   //  3
Logger.Level.DEBUG  //  4  ← default
Logger.Level.TRACE  //  5
Logger.Level.ALL    //  6
```

---

## ValidationUtils

**Import:** `sap/dm/dme/pod2/utils/ValidationUtils`

Validates quantity inputs from users against SAP DM business rules.

### API

```javascript
try {
    const nQty = ValidationUtils.validateQuantity(vValue, oOptions);
    // nQty is the parsed, validated number
} catch (oError) {
    // oError.message is a localized user-facing error string
    MessageHistory.showError(oError.message);
}
```

**Parameters:**
- `vValue`: `string | number` — the raw input value
- `oOptions.unitOfMeasure`: string — if a countable unit (EA, PC, etc.), decimals are not allowed
- `oOptions.maxIntegerDigits`: number — default 11
- `oOptions.maxFractionDigits`: number — default from `NumberFormatter` (or 0 for countable units)

**Throws when:**
- Value cannot be parsed (e.g. "abc")
- Value is NaN, Infinity, negative, or zero
- Value exceeds `Number.MAX_SAFE_INTEGER`
- Integer digits exceed `maxIntegerDigits`
- Fraction digits exceed `maxFractionDigits`
- Value has decimals but `maxFractionDigits` is 0 (countable unit)

**Returns:** the validated `number`

### Usage Pattern

```javascript
import ValidationUtils from "sap/dm/dme/pod2/utils/ValidationUtils";

_onQuantityChange(sValue, sUom) {
    try {
        const nQty = ValidationUtils.validateQuantity(sValue, { unitOfMeasure: sUom });
        this._oQuantityInput.setValueState(ValueState.None);
        this._nValidatedQty = nQty;
    } catch (oError) {
        this._oQuantityInput.setValueState(ValueState.Error);
        this._oQuantityInput.setValueStateText(oError.message);
    }
}
```

---

## DialogUtils

**Import:** `sap/dm/dme/pod2/utils/DialogUtils`

Creates pre-built dialogs for common patterns. Currently provides one factory method.

### showInputDialog

```javascript
const sValue = await DialogUtils.showInputDialog({
    title: "Enter Serial Number",
    prompt: "Please scan or enter the serial number:",
    validate: (sInput) => {
        // Throw an Error to show validation error in the dialog
        if (!sInput || sInput.trim() === "") {
            throw new Error("Serial number is required");
        }
        // Optionally use ValidationUtils for numeric inputs
    }
});

if (sValue !== undefined) {
    // User clicked OK and validation passed
    // sValue is the trimmed input string
} else {
    // User cancelled
}
```

**Behaviour:**
- Returns `Promise<string | undefined>` — resolves with value on OK, `undefined` on Cancel
- Pressing Enter in the input fires the OK button
- Validation errors are shown inline as value state messages on the input
- Dialog is automatically destroyed on close
- Uses `PodDialog` (automatically applies core POD models: i18n, pod context)
- OK button text uses `{i18n>ok}`, Cancel uses `{i18n>cancel}` from the global POD i18n bundle

---

## LanguageUtils

**Import:** `sap/dm/dme/pod2/utils/LanguageUtils`

Catalogue of the 28 SAP DM supported locales with their BCP-47 codes and native names.

### API

```javascript
// Get all supported languages
const aLanguages = LanguageUtils.getLanguages();
// → [{ code: "en", nativeName: "English" }, { code: "de", nativeName: "Deutsch" }, ...]

// Look up a specific locale
const oEntry = LanguageUtils.getLanguage("zh-CN");
// → { code: "zh-CN", nativeName: "简体中文" } or undefined if not supported
```

### Supported Locales

| Code | Native Name | Code | Native Name |
|------|------------|------|------------|
| `en` | English | `de` | Deutsch |
| `de-CH` | Schweizerdeutsch | `es` | Español |
| `fr` | Français | `it` | Italiano |
| `ja` | 日本語 | `ko` | 한국어 |
| `pt` | Português | `ru` | Русский |
| `zh-CN` | 简体中文 | `zh-TW` | 繁體中文 |
| `bg` | Български | `cs` | Čeština |
| `da` | Dansk | `hr` | Hrvatski |
| `hu` | Magyar | `lt` | Lietuvių |
| `nl` | Nederlands | `pl` | Polski |
| `ro` | Română | `sh` | Srpski (Serbian) |
| `sk` | Slovenčina | `sl` | Slovenščina |
| `sv` | Svenska | `th` | ไทย |
| `tr` | Türkçe | `uk` | Українська |
| `vi` | Tiếng Việt | | |

**⚠️ Language file naming:** i18n property files use **underscore** format (`i18n_zh_CN.properties`), but LanguageUtils codes use **dash** format (`zh-CN`). SAPUI5 resolves the bundle internally.

**⚠️ Serbian:** The code is `sh` (not `sr`). The `sap-dm-languages.md` reference uses `sr` — use `sh` when working with `LanguageUtils`.

---

## CacheBuster

**Import:** `sap/dm/dme/pod2/CacheBuster`

Configures the SAPUI5 module loader with cache-busting tokens for extension namespaces. Called once by the POD runtime at startup — **custom plugins do not need to call this**.

```javascript
// Internal use only — called by POD runtime
await CacheBuster.configure();
```

---

## DTRUMService

**Import:** `sap/dm/dme/pod2/utils/DTRUMService`

Dynatrace Real User Monitoring integration. Used internally by the POD runtime — **custom plugins do not need to call this** directly.

```javascript
// Internal use only
await DTRUMService.identifyUser();
await DTRUMService.firePodOpened();
```

---

## Quick Reference — Which Utility for What

| Task | Use |
|------|-----|
| Get current date/time (timezone-safe) | `DateTimeUtils.now()` |
| Format a date for display | `DateTimeUtils.localeDateTime/Date/Time()` |
| Start/end of a day | `DateTimeUtils.startOfDay()` / `endOfDay()` |
| Parse OData `/Date(...)/ ` | `DateTimeUtils.fromODataDateString()` |
| Convert ISO strings ↔ Date objects in objects | `DateTimeUtils.parseDateProperties()` / `encodeDateProperties()` |
| Null-safe array check | `Utilities.isNonEmptyArray()` / `isEmptyOrNotArray()` |
| Sort array by property | `Utilities.sortByProperty()` |
| Unique array elements | `Utilities.getUniqueArrayElements()` |
| Validate a UI5 control ID | `Utilities.isValidId()` |
| Generate unique ID (no conflict) | `Utilities.createId()` |
| Check if control is visible | `Utilities.isVisible()` |
| Scroll control into view | `Utilities.scrollIntoView()` |
| Dynamic module loading | `Utilities.requireModule/AllModules/SomeModules()` |
| Validate a quantity input | `ValidationUtils.validateQuantity()` |
| Prompt user for text input | `DialogUtils.showInputDialog()` |
| List/lookup supported locales | `LanguageUtils.getLanguages()` / `getLanguage()` |
| Log messages | `Logger.getLogger(component)` → `.info/warn/error/debug()` |

---

## Import Paths Summary

```javascript
import DateTimeUtils    from "sap/dm/dme/pod2/DateTimeUtils";
import Utilities        from "sap/dm/dme/pod2/Utilities";
import Logger           from "sap/dm/dme/pod2/Logger";
import ValidationUtils  from "sap/dm/dme/pod2/utils/ValidationUtils";
import DialogUtils      from "sap/dm/dme/pod2/utils/DialogUtils";
import LanguageUtils    from "sap/dm/dme/pod2/utils/LanguageUtils";
import CacheBuster      from "sap/dm/dme/pod2/CacheBuster";       // runtime only
import DTRUMService     from "sap/dm/dme/pod2/utils/DTRUMService"; // runtime only
```

---

**Source**: SAP Digital Manufacturing POD 2.0 utility source files  
**Last Updated**: 2026-09-07
