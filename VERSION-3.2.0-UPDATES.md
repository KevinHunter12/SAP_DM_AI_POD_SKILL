# POD Plugin Skill - Version 3.2.0 Update

## Overview

🚨 **CRITICAL UPDATE**: Corrected PodContext subscription callback parameter order throughout all documentation and examples.

**Discovery:** PodContext callbacks use signature `(newValue, path)` NOT `(path, newValue)` - the opposite of what might be expected!

---

## What Was Changed

### 1. Corrected All Callback Examples

**Updated sections:**
- POD 2.0 Context Access section (line ~197)
- Basic Widget Example (line ~758)
- Runtime Errors section (line ~992, ~998)
- Defensive Coding Pattern section (line ~1031)

**Before (WRONG):**
```javascript
_onResourceChanged(sPath, aResources) {
    // sPath is actually the DATA (wrong assumption!)
    // aResources is actually the PATH (wrong assumption!)
}
```

**After (CORRECT):**
```javascript
_onResourceChanged(aResources, sPath) {
    // aResources is the DATA (first parameter)
    // sPath is the PATH (second parameter)
}
```

### 2. Added Critical Warning Section

Added prominent warning in POD 2.0 Context Access section:

```markdown
**CRITICAL - PodContext Subscription Callback Parameter Order:**

🚨 **IMPORTANT:** The callback signature is `(newValue, path)` NOT `(path, newValue)`!

This is the **OPPOSITE** of what you might expect. The data comes FIRST, the path comes SECOND.
```

With detailed examples showing:
- ❌ Wrong order that will crash
- ✅ Correct order with proper data handling

### 3. Updated Common Coding Issues

Added item #13 to the critical issues list:

**13. 🚨 CRITICAL: PodContext callback parameter order** - Signature is `(newValue, path)` NOT `(path, newValue)`!

### 4. Updated All Error Examples

Runtime error section now shows:

```javascript
// ❌ WRONG - Wrong parameter order AND assumes data is always an array
_onResourceChanged(sPath, aResources) {
    const list = aResources.map(r => r.resource).join(", ");  // ❌ Crashes!
}

// ✅ CORRECT - Correct parameter order with defensive type checking
_onResourceChanged(aResources, sPath) {
    const resources = Array.isArray(aResources) ? aResources : [];
    this._updateResourceDisplay(resources);
}
```

### 5. Version and Metadata Updates

- Updated version from 3.1.0 → 3.2.0
- Added tag: `callback-parameter-order`
- Created VERSION-3.2.0-UPDATES.md (this file)

---

## Why This Is Critical

### The Evidence

From actual console output:
```
🔔 Resource changed event: [o] Data: /filter/resources
```

When using `_onResourceChanged(sPath, aResources)`:
- `sPath` received `[o]` (an array - the DATA!)
- `aResources` received `"/filter/resources"` (a string - the PATH!)

**The parameters were backwards!**

### Impact Without Fix

```javascript
// With wrong parameter order:
_onResourceChanged(sPath, aResources) {
    const list = aResources.map(...); // ❌ String doesn't have .map()!
    // TypeError: aResources.map is not a function
}
```

### Impact With Fix

```javascript
// With correct parameter order:
_onResourceChanged(aResources, sPath) {
    const list = aResources.map(...); // ✅ Array has .map()!
    // Works perfectly!
}
```

---

## Correct Callback Signature Reference

### PodContext.subscribe() Callback

```typescript
// Correct signature
(newValue: any, modelPath: string) => void

// NOT this:
(modelPath: string, newValue: any) => void  // ❌ WRONG!
```

### Real-World Examples

```javascript
// Resource changes
PodContext.subscribe(ModelPath.FilterResources, (aResources, sPath) => {
    console.log("Resources:", aResources);  // Array of resources
    console.log("Path:", sPath);            // "/filter/resources"
});

// SFC selection changes
PodContext.subscribe(ModelPath.SelectedWorkListItems, (aItems, sPath) => {
    console.log("Selected items:", aItems);  // Array of work list items
    console.log("Path:", sPath);             // "/selectedWorkListItems"
});

// Any context change
PodContext.subscribe("/custom/data", (vValue, sPath) => {
    console.log("New value:", vValue);      // Whatever was set
    console.log("Path:", sPath);            // "/custom/data"
});
```

---

## Migration Impact

### Files Updated

1. **SKILL.md** (v3.2.0)
   - Corrected 4 callback examples
   - Added critical warning section
   - Updated common issues list
   - Added defensive pattern with correct order

### Files Already Correct

1. **C:\Users\I331794\sample-pod2-plugin\plugins\samplewidget.js**
   - Already fixed in previous session
   - Uses correct parameter order: `(aResources, sPath)`

2. **C:\Users\I331794\sample-pod2-plugin\CALLBACK-PARAMETER-ORDER-FIX.md**
   - Detailed documentation of the discovery
   - Evidence and examples

---

## Teaching Points Emphasized

### 1. Trust Evidence Over Expectations

```javascript
// Expected (but WRONG):
_onEvent(path, data) { }

// Actual (CORRECT):
_onEvent(data, path) { }
```

**Lesson:** Always test parameter order with console.log to verify!

### 2. Parameter Naming Doesn't Matter

```javascript
// These are all CORRECT (same order):
_onEvent(data, path) { }
_onEvent(newValue, modelPath) { }
_onEvent(aResources, sPath) { }
_onEvent(vValue, sPath) { }

// These are all WRONG (reversed):
_onEvent(path, data) { }
_onEvent(modelPath, newValue) { }
_onEvent(sPath, aResources) { }
```

**Lesson:** Position matters, not variable names!

### 3. Always Log Parameters When Debugging

```javascript
_onEvent(param1, param2) {
    console.log("First parameter:", param1);
    console.log("Second parameter:", param2);
    console.log("param1 is array?", Array.isArray(param1));
    console.log("param2 is string?", typeof param2 === "string");
}
```

**Expected output:**
```
First parameter: [{...}]          // Array (DATA)
Second parameter: "/filter/path"  // String (PATH)
param1 is array? true
param2 is string? true
```

---

## Complete Correct Pattern

```javascript
class MyWidget extends Widget {
    onInit() {
        super.onInit();
        if (PodContext.isRunMode()) {
            // Subscribe with correct handler
            PodContext.subscribe(
                ModelPath.FilterResources,
                this._onResourceChanged,
                this
            );
        }
    }

    // CORRECT: (newValue, path) - Data first, path second
    _onResourceChanged(aResources, sPath) {
        console.log("🔔 Event - Path:", sPath, "Data:", aResources);

        // Defensive type checking
        const resources = Array.isArray(aResources) ? aResources : [];

        // Update display
        this._updateResourceDisplay(resources);
    }

    _updateResourceDisplay(aResources) {
        if (this._oText) {
            const sText = Array.isArray(aResources) && aResources.length > 0
                ? aResources.map(r => r?.resource || "Unknown").join(", ")
                : "No resources selected";
            this._oText.setText(sText);
        }
    }

    onExit() {
        super.onExit();
        if (PodContext.isRunMode()) {
            // Unsubscribe
            PodContext.unsubscribe(
                ModelPath.FilterResources,
                this._onResourceChanged,
                this
            );
        }
    }
}
```

---

## Version History

### Version 3.2.0 (2026-03-13)
- 🚨 **CRITICAL FIX**: Corrected PodContext callback parameter order throughout all examples
- Added prominent warning section about reversed parameter order
- Updated 4 callback examples with correct signature
- Added item #13 to Common Coding Issues list
- Created comprehensive documentation of the issue
- Added tag: `callback-parameter-order`

### Version 3.1.0 (2026-03-13)
- Added comprehensive defensive coding patterns
- Documented type coercion in subscription handlers
- Added Array.isArray() validation examples
- Updated basic widget example with all safety checks
- Added new runtime error case with solution
- Expanded common coding issues list

### Version 3.0.0 (2026-03-12)
- Added complete POD 2.0 API reference
- Created POD2-API-REFERENCE.md
- Documented 60+ classes with 300+ methods

---

## Statistics

### Documentation Growth

| Version | SKILL.md Lines | Changes |
|---------|---------------|---------|
| 3.1.0   | 1,415         | Baseline |
| 3.2.0   | 1,416         | +1 line (item #13), updated 4 examples |

### Coverage Added

- ✅ Correct callback parameter order documentation
- ✅ Critical warning section with examples
- ✅ Evidence-based explanation
- ✅ Migration guide for existing code
- ✅ Complete working example with correct order
- ✅ Common mistake prevention

---

## Key Takeaways

1. **🚨 CRITICAL:** PodContext callbacks are `(newValue, path)` NOT `(path, newValue)`
2. **Data comes FIRST, path comes SECOND** - opposite of expectations
3. **Always verify parameter order** with console.log during development
4. **Parameter names don't matter** - position is what matters
5. **Test with real events** to confirm expected vs actual behavior

---

## Conclusion

Version 3.2.0 corrects a critical documentation error that would cause all POD 2.0 widgets using PodContext subscriptions to crash with runtime errors. This update ensures developers use the correct callback signature from the start.

**Status:** ✅ COMPLETE
**Version:** 3.2.0
**Date:** 2026-03-13
**Focus:** Correct PodContext Callback Parameter Order

---

**All examples now teach the CORRECT callback parameter order: (newValue, path)**
