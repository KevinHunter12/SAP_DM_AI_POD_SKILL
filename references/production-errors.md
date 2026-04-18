# Common Production Errors and Fixes

**Source**: Issues found in real SAP production POD code analysis

These are actual bugs and anti-patterns found in production widgets that you must avoid.

---

## Error #1: Missing onExit() Interval Cleanup → Memory Leak

**Symptom**: Widget destroyed while timer running → interval continues forever → memory leak

**Production Bug**: StopWatchWidget.js missing interval cleanup in onExit()

### ❌ WRONG - Missing cleanup
```javascript
class StopWatchWidget extends Widget {
    #nIntervalId;
    
    _startTimer() {
        this.#nIntervalId = setInterval(() => this._updateText(), 1000);
    }
    
    _stopTimer() {
        clearInterval(this.#nIntervalId);
        this.#nIntervalId = undefined;
    }
    
    // ❌ BUG: Missing interval cleanup!
    onExit() {
        super.onExit();
        // Forgot to stop timer!
        this.#oTimeText = null;
        this.#oStartStopButton = null;
    }
}
```

### ✅ CORRECT - Always cleanup in onExit
```javascript
onExit() {
    super.onExit();
    
    // ✅ CRITICAL: Stop timer if running
    if (this._isRunning()) {
        this._stopTimer();
    }
    
    // Clean up references
    this.#oTimeText = null;
    this.#oStartStopButton = null;
    this.#nIntervalId = null;
}
```

**Impact**: Memory leak if widget destroyed while timer running. Interval continues forever.

**Prevention**: Always check for running intervals in onExit() and stop them.

---

## Error #2: Time Formatting Bug - Missing Modulo

**Symptom**: Time displays incorrectly (1:75:135 instead of 2:15:15)

**Production Bug**: StopWatchWidget.js time formatting

### ❌ WRONG - Missing modulo in formatting
```javascript
_formatTime(nTime) {
    const nSeconds = Math.floor(nTime / 1000);
    const nMinutes = Math.floor(nSeconds / 60);
    const nHours = Math.floor(nMinutes / 60);
    
    // ❌ BUG: Shows total minutes and seconds, not remainder
    return `${nHours.toString().padStart(2, "0")}:${nMinutes.toString().padStart(2, "0")}:${nSeconds.toString().padStart(2, "0")}`;
}
```

**Example Output**: After 135 seconds (2 minutes 15 seconds):
- nSeconds = 135
- nMinutes = 2
- nHours = 0
- Display: `00:02:135` ❌ WRONG!

### ✅ CORRECT - Use modulo for remainder
```javascript
_formatTime(nTime) {
    const nSeconds = Math.floor(nTime / 1000);
    const nMinutes = Math.floor(nSeconds / 60);
    const nHours = Math.floor(nMinutes / 60);
    
    // ✅ CORRECT: Use modulo to get remainder values
    return `${nHours.toString().padStart(2, "0")}:${(nMinutes % 60).toString().padStart(2, "0")}:${(nSeconds % 60).toString().padStart(2, "0")}`;
}
```

**Example Output**: After 135 seconds:
- nSeconds = 135, nSeconds % 60 = 15 ✅
- nMinutes = 2, nMinutes % 60 = 2 ✅
- nHours = 0
- Display: `00:02:15` ✅ CORRECT!

**Impact**: Time displays incorrectly for all values. Basic math error in production code.

**Prevention**: Always use modulo (%) when displaying time components.

---

## Error #3: Cannot Read Property 'id' of undefined

**Symptom**: "Cannot read property 'id' of undefined" error in console

**Cause**: Missing config validation in _createView()

### ❌ WRONG - No validation
```javascript
_createView() {
    const oConfig = this.getConfig();
    
    // ❌ Accessing oConfig.id without checking if oConfig exists
    return new VBox(oConfig.id, {
        items: [new Text({ text: "Hello" })]
    });
}
```

### ✅ CORRECT - Validate and handle gracefully
```javascript
_createView() {
    const oConfig = this.getConfig();
    
    // ✅ CRITICAL: Validate config before using
    if (!oConfig || !oConfig.id) {
        console.error("Widget configuration is invalid");
        return new VBox({
            items: [
                new Text({ 
                    text: "Configuration Error: Unable to load widget" 
                })
            ]
        });
    }
    
    // Safe to use oConfig.id now
    return new VBox(oConfig.id, {
        items: [new Text({ text: "Hello" })]
    });
}
```

**Impact**: Widget crashes during initialization. Cryptic error message.

**Prevention**: Always validate config at start of _createView().

---

## Error #4: Property Changes Don't Update Widget

**Symptom**: Changing properties in POD Designer doesn't update widget display

**Cause**: Missing setPropertyValue() override

### ❌ WRONG - No property update handling
```javascript
class MyWidget extends Widget {
    #oText;
    
    _createView() {
        this.#oText = new Text({ text: this.getPropertyValue("displayText") });
        return new VBox(this.getConfig().id, { items: [this.#oText] });
    }
    
    // ❌ Missing setPropertyValue override
    // Property changes in Designer don't update display
}
```

### ✅ CORRECT - Override setPropertyValue
```javascript
class MyWidget extends Widget {
    PropertyId = Object.freeze({
        DisplayText: "displayText"
    });
    
    #oText;
    
    _createView() {
        this.#oText = new Text({ text: this.getPropertyValue("displayText") });
        return new VBox(this.getConfig().id, { items: [this.#oText] });
    }
    
    // ✅ Override to handle live updates
    setPropertyValue(sName, vValue) {
        if (sName === this.PropertyId.DisplayText && this.#oText) {
            this.#oText.setText(vValue);
        }
        super.setPropertyValue(sName, vValue);  // CRITICAL: Call parent!
    }
}
```

**Impact**: Designer UX broken. Users must refresh to see property changes.

**Prevention**: Override setPropertyValue() for all visual properties.

---

## Error #5: Multiple Timers Running Simultaneously

**Symptom**: Timer speeds up or multiple instances run at once

**Cause**: No guard in _startTimer()

### ❌ WRONG - No guard against multiple intervals
```javascript
_startTimer() {
    // ❌ No check if already running
    this.#nStartTime = Date.now();
    this.#nIntervalId = setInterval(() => this._updateText(), 1000);
}
```

**Result**: Calling _startTimer() twice creates two intervals running simultaneously.

### ✅ CORRECT - Guard against multiple intervals
```javascript
_startTimer() {
    // ✅ Guard: Prevent multiple intervals
    if (this._isRunning()) {
        return;
    }
    
    this.#nStartTime = Date.now();
    this.#nIntervalId = setInterval(() => this._updateText(), 1000);
}

_isRunning() {
    return this.#nIntervalId !== undefined;
}
```

**Impact**: Multiple timers run simultaneously, causing race conditions.

**Prevention**: Always check _isRunning() before starting interval.

---

## Error #6: Cannot Set Property of Null

**Symptom**: "Cannot set property of null" error after widget destroyed

**Cause**: Control reference accessed after onExit()

### ❌ WRONG - No null check before using control
```javascript
setPropertyValue(sName, vValue) {
    switch (sName) {
        case this.PropertyId.Text:
            // ❌ What if this.#oText is null?
            this.#oText.setText(vValue);
            break;
    }
    super.setPropertyValue(sName, vValue);
}
```

### ✅ CORRECT - Check control exists before using
```javascript
setPropertyValue(sName, vValue) {
    switch (sName) {
        case this.PropertyId.Text:
            // ✅ Check control exists first
            if (this.#oText) {
                this.#oText.setText(vValue);
            }
            break;
    }
    super.setPropertyValue(sName, vValue);
}
```

**Impact**: Error if property changed after widget destroyed or before created.

**Prevention**: Always check control exists: `if (this.#oControl)` before using.

---

## Error #7: Direct Property Access in _createView()

**Symptom**: Properties not applied correctly during initialization

**Production Pattern**: Inconsistent with framework patterns

### ⚠️ Production code pattern (works but not recommended)
```javascript
_createView() {
    const oConfig = this.getConfig();
    
    this.#oText = new CustomText(oConfig.id + "-text", {
        backgroundColor: oConfig.properties.backgroundColor,  // Direct access
        fontColor: oConfig.properties.fontColor
    });
}
```

### ✅ Better pattern (more consistent)
```javascript
_createView() {
    const oConfig = this.getConfig();
    
    this.#oText = new CustomText(oConfig.id + "-text", {
        backgroundColor: this.getPropertyValue(this.PropertyId.BackgroundColor),
        fontColor: this.getPropertyValue(this.PropertyId.FontColor)
    });
}
```

**Impact**: Minor - both work, but getPropertyValue() is more consistent with framework.

**Recommendation**: Use getPropertyValue() for consistency, though direct access works.

---

## Error #8: No Error Handling for setInterval

**Symptom**: Uncaught errors break widget silently

**Cause**: No error handling in interval callback

### ❌ WRONG - No error handling
```javascript
_startTimer() {
    this.#nStartTime = Date.now();
    
    // ❌ Uncaught errors will break everything
    this.#nIntervalId = setInterval(() => {
        this._updateText();  // What if this throws?
    }, 1000);
}
```

### ✅ CORRECT - Handle errors gracefully
```javascript
_startTimer() {
    try {
        this.#nStartTime = Date.now();
        
        this.#nIntervalId = setInterval(() => {
            try {
                this._updateText();
            } catch (oError) {
                console.error("Error updating timer display:", oError);
                this._stopTimer();  // Stop on error
            }
        }, 1000);
        
    } catch (oError) {
        console.error("Error starting timer:", oError);
        MessageToast.show("Failed to start timer");
    }
}
```

**Impact**: Uncaught errors break widget; better to handle gracefully.

**Prevention**: Wrap interval callback in try-catch, stop timer on error.

---

## Production Error Checklist

Before deploying widgets, verify:

### Timer Widgets
- [ ] Interval ID stored in private field
- [ ] `_isRunning()` check method exists
- [ ] `_startTimer()` guards against multiple intervals
- [ ] `_stopTimer()` clears interval and sets ID to undefined
- [ ] `onExit()` stops timer if running
- [ ] Error handling inside interval callback
- [ ] Time formatting uses modulo (%) for display values

### All Widgets
- [ ] Config validation at start of `_createView()`
- [ ] `setPropertyValue()` overridden for visual properties
- [ ] Control null checks before using (`if (this.#oControl)`)
- [ ] `super.setPropertyValue()` called in override
- [ ] All subscriptions unsubscribed in `onExit()`
- [ ] Control references cleared in `onExit()`

### Custom Controls
- [ ] Import from `sap/dm/dme/pod2/control/Custom*`
- [ ] Properties match custom control API
- [ ] `setPropertyValue()` updates custom control properties
- [ ] Default config includes all custom control properties

---

## Quick Reference: Production Bugs

| Bug | Symptom | Fix |
|-----|---------|-----|
| Missing interval cleanup | Memory leak | Stop timer in `onExit()` |
| Time formatting error | Wrong time display | Use modulo (%) for minutes/seconds |
| No config validation | Crash on load | Validate config in `_createView()` |
| No setPropertyValue | Properties don't update | Override `setPropertyValue()` |
| Multiple intervals | Timer speeds up | Guard with `_isRunning()` check |
| Null control access | Cannot set property of null | Check `if (this.#oControl)` |
| No error handling | Silent failures | Wrap interval in try-catch |

---

## See Also

- [widget-patterns.md](widget-patterns.md) - Timer and interval management pattern
- [common-mistakes.md](common-mistakes.md) - All common mistakes
- [error-handling.md](error-handling.md) - Complete error handling patterns
