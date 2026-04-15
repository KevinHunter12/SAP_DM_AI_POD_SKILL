# POD 2.0 Troubleshooting Flowchart

This guide helps you systematically debug POD plugin issues.

---

## Upload Failed

```
Plugin Upload Failed
  │
  ├─ Error: "Failed to create custom extensions"
  │   │
  │   ├─ Check extension.json structure
  │   │   └─ ✅ ONLY widgets and actions arrays
  │   │   └─ ❌ Remove: name, description, version, provider
  │   │
  │   ├─ Check modulePath format
  │   │   └─ ✅ "custom/pod2/example/plugins/widget"
  │   │   └─ ❌ No .js extension
  │   │
  │   └─ Check type field
  │       └─ ✅ "custom.pod2.example.plugins.widget" (dots not slashes)
  │
  ├─ Error: "Module not found"
  │   │
  │   ├─ Check file location
  │   │   └─ modulePath: custom/pod2/example/plugins/widget
  │   │   └─ File should be: plugins/widget.js
  │   │   └─ NOT: plugins/custom/pod2/example/widget.js
  │   │
  │   ├─ Check file name matches
  │   │   └─ Case-sensitive!
  │   │
  │   └─ Check zip structure
  │       └─ extension.json at root
  │       └─ plugins/ folder at root
  │       └─ NOT nested in another folder
  │
  └─ No error but widget not appearing
      │
      ├─ Check browser console (F12)
      │   └─ Look for JavaScript errors
      │   └─ Look for 404 errors (wrong paths)
      │
      ├─ Verify widget class
      │   └─ Extends Widget base class?
      │   └─ All static methods implemented?
      │   └─ getCategory() returns valid category?
      │
      └─ Clear browser cache
          └─ Hard reload (Ctrl+Shift+R)
```

---

## Widget Not Updating

```
Widget Appears but Doesn't Update
  │
  ├─ Controls not updating when context changes
  │   │
  │   ├─ Check: Are you subscribing to context?
  │   │   ```javascript
  │   │   onInit() {
  │   │       super.onInit();
  │   │       if (PodContext.isRunMode()) {  // ← Must check!
  │   │           PodContext.subscribe(...);
  │   │       }
  │   │   }
  │   │   ```
  │   │
  │   ├─ Check: Storing control references?
  │   │   ```javascript
  │   │   _createView() {
  │   │       this._oText = new Text({ text: "..." });  // ← Store reference!
  │   │       return new VBox(this.getConfig().id, {
  │   │           items: [this._oText]
  │   │       });
  │   │   }
  │   │
  │   │   _updateDisplay() {
  │   │       if (this._oText) {  // ← Use stored reference
  │   │           this._oText.setText("New value");
  │   │       }
  │   │   }
  │   │   ```
  │   │
  │   └─ Check: Subscription callback correct?
  │       └─ Parameters are (data, path) NOT (path, data)!
  │       ```javascript
  │       // ❌ WRONG
  │       PodContext.subscribe(path, (sPath, aData) => { ... });
  │
  │       // ✅ CORRECT
  │       PodContext.subscribe(path, (aData, sPath) => { ... });
  │       ```
  │
  ├─ TypeError in subscription callback
  │   │
  │   ├─ Error: "Cannot read property 'map' of undefined"
  │   │   └─ Add defensive type checking:
  │   │   ```javascript
  │   │   _handler(aData, sPath) {
  │   │       // Coerce to expected type
  │   │       const data = Array.isArray(aData) ? aData : [];
  │   │       // Use optional chaining
  │   │       const safe = data.map(item => item?.property || "default");
  │   │   }
  │   │   ```
  │   │
  │   ├─ Error: "aData.map is not a function"
  │   │   └─ Wrong parameter order!
  │   │   └─ Data comes FIRST, path comes SECOND
  │   │   ```javascript
  │   │   // ❌ WRONG order
  │   │   _handler(sPath, aData) { ... }
  │   │
  │   │   // ✅ CORRECT order
  │   │   _handler(aData, sPath) { ... }
  │   │   ```
  │   │
  │   └─ Error: "Cannot set property of null"
  │       └─ Control reference not stored or already destroyed
  │       ```javascript
  │       _updateDisplay() {
  │           if (this._oText) {  // ← Always check!
  │               this._oText.setText("...");
  │           }
  │       }
  │       ```
  │
  └─ Memory leak / widget keeps getting slower
      │
      └─ Check: Are you unsubscribing on exit?
          ```javascript
          onExit() {
              super.onExit();
              if (PodContext.isRunMode()) {
                  PodContext.unsubscribe(path, this._handler, this);
              }
              this._oText = null;  // Clean up references
          }
          ```
```

---

## API Calls Failing

```
API Calls Not Working
  │
  ├─ Error: 404 - Module not found
  │   │
  │   └─ Using wrong module?
  │       ```javascript
  │       // ❌ WRONG - POD 1.0 module
  │       import AjaxUtil from "sap/dm/dme/model/AjaxUtil";
  │
  │       // ✅ CORRECT - POD 2.0 modules
  │       import ApiClient from "sap/dm/dme/pod2/api/ApiClient";
  │       import RestClient from "sap/dm/dme/pod2/api/RestClient";
  │       ```
  │
  ├─ API call never completes / hangs
  │   │
  │   ├─ Missing await?
  │   │   ```javascript
  │   │   async _loadData() {  // ← Function must be async
  │   │       const data = await ApiClient.custom.get(...);  // ← await!
  │   │       return data;
  │   │   }
  │   │   ```
  │   │
  │   └─ Add timeout protection
  │       ```javascript
  │       const data = await Promise.race([
  │           ApiClient.custom.get("/endpoint"),
  │           new Promise((_, reject) =>
  │               setTimeout(() => reject(new Error("Timeout")), 30000)
  │           )
  │       ]);
  │       ```
  │
  ├─ Error: 401 Unauthorized / 403 Forbidden
  │   │
  │   ├─ Check endpoint requires authentication
  │   ├─ Check user has permissions
  │   └─ Check CSRF token if required
  │
  ├─ Error: 500 Internal Server Error
  │   │
  │   ├─ Check Network tab → Preview response
  │   ├─ Check server logs
  │   └─ Validate request payload structure
  │
  └─ Success but data not showing
      │
      ├─ Check response structure
      │   └─ Log response to console
      │   └─ Verify field names match
      │
      └─ Check error handling
          ```javascript
          try {
              const data = await ApiClient.custom.get(...);
              console.log("Response:", data);  // ← Debug output
              this._updateDisplay(data);
          } catch (error) {
              console.error("API failed:", error);  // ← See actual error
              MessageHistory.showError("Failed to load data");
          }
          ```
```

---

## Runtime Errors

```
JavaScript Errors in Console
  │
  ├─ Error: 404 - Failed to load PodContext.js
  │   │
  │   └─ Wrong import path!
  │       ```javascript
  │       // ❌ WRONG
  │       "sap/dm/dme/pod2/model/PodContext"
  │       "sap/dm/dme/pod2/model/ModelPath"
  │
  │       // ✅ CORRECT
  │       "sap/dm/dme/pod2/context/PodContext"
  │       "sap/dm/dme/pod2/context/ModelPath"
  │       ```
  │
  ├─ Error: "getView returned view with different ID"
  │   │
  │   └─ Not passing config ID to view!
  │       ```javascript
  │       // ❌ WRONG
  │       _createView() {
  │           return new VBox({ items: [...] });
  │       }
  │
  │       // ✅ CORRECT
  │       _createView() {
  │           const oConfig = this.getConfig();
  │           return new VBox(oConfig.id, { items: [...] });
  │       }
  │       ```
  │
  ├─ Error: "this.createId is not a function"
  │   │
  │   └─ Using Controller methods in Widget!
  │       ```javascript
  │       // ❌ WRONG - Don't use createId/byId
  │       const sId = this.createId("myControl");
  │       const oControl = this.byId("myControl");
  │
  │       // ✅ CORRECT - Store direct references
  │       this._oControl = new Text({ text: "..." });
  │       // Later: this._oControl.setText("...");
  │       ```
  │
  ├─ Error: "Cannot read property of undefined"
  │   │
  │   ├─ Missing null/undefined check
  │   │   ```javascript
  │   │   // ❌ WRONG
  │   │   const name = item.data.name;
  │   │
  │   │   // ✅ CORRECT
  │   │   const name = item?.data?.name || "Unknown";
  │   │   ```
  │   │
  │   └─ PodContext returned undefined
  │       ```javascript
  │       const oItem = PodContext.getLastSelectedWorkListItem();
  │       if (!oItem) {
  │           Logger.warn("No item selected");
  │           return;  // ← Handle gracefully
  │       }
  │       // Use oItem...
  │       ```
  │
  └─ Widget throws error and disappears
      │
      └─ Wrap _createView() in try-catch
          ```javascript
          _createView() {
              try {
                  const oConfig = this.getConfig();
                  // ... normal code ...
                  return oView;
              } catch (error) {
                  Logger.error("Failed to create view", error);
                  return new VBox({
                      items: [
                          new Text({
                              text: "Error loading widget: " + error.message
                          })
                      ]
                  });
              }
          }
          ```
```

---

## Debugging Steps

### 1. Check Browser Console First
```
Press F12 → Console tab
Look for:
- Red errors (JavaScript exceptions)
- Yellow warnings
- 404 errors (missing files/wrong paths)
- Network errors (failed API calls)
```

### 2. Enable Verbose Logging
```javascript
// Add to your widget
#oLog = Logger.getLogger("custom.MyWidget");

onInit() {
    this.#oLog.debug("Widget initializing...");
    super.onInit();
    this.#oLog.info("Widget initialized");
}

_onResourceChanged(aResources, sPath) {
    this.#oLog.debug("Resource changed:", { aResources, sPath });
    // ...
}
```

### 3. Use Network Tab
```
F12 → Network tab
Look for:
- Failed requests (red, 404, 500)
- Slow requests (> 2s)
- Request/response payloads
- Headers (auth tokens, content-type)
```

### 4. Check Element Inspector
```
F12 → Elements tab
- Find your widget in DOM
- Check if controls exist
- Check CSS classes/styles
- Verify IDs match config
```

### 5. Use Debugger
```javascript
_onButtonPress() {
    debugger;  // ← Browser will pause here
    const sPlant = PodContext.getPlant();
    console.log("Plant:", sPlant);
}
```

---

## Quick Diagnostic Checklist

Before asking for help, verify:

- [ ] Browser console shows no errors
- [ ] extension.json has ONLY widgets/actions arrays
- [ ] modulePath matches file location
- [ ] All imports use correct paths (pod2/context/ not pod2/model/)
- [ ] _createView() passes oConfig.id to root view
- [ ] Subscription callbacks use (data, path) order
- [ ] Defensive type checking on all context data
- [ ] ApiClient/RestClient used (not AjaxUtil)
- [ ] All async functions use await
- [ ] Controls stored as instance properties (this._oControl)
- [ ] Unsubscribe in onExit()
- [ ] super.onInit() and super.onExit() called

---

## Still Stuck?

1. **Simplify**: Create minimal widget that just shows text
2. **Isolate**: Remove features until it works, then add back one by one
3. **Compare**: Look at working SAP example widgets
4. **Ask**: Provide error message, code snippet, and what you've tried
