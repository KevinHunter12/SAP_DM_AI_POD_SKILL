# Error Handling for POD 2.0 Plugins

## Error Handling Decision Matrix ⭐⭐⭐⭐⭐

| Scenario | Log | Show User | Action | Tool |
|----------|-----|-----------|--------|------|
| Background data load | ✅ | ❌ | Continue | Logger.error() |
| User action failure | ✅ | ✅ | Rollback | MessageHistory.showError() |
| Critical failure | ✅ | ✅ | Block/Disable | MessageBox.error() |
| Validation error | ❌ | ✅ | Highlight field | ValueState.Error |
| Permission denied | ✅ | ✅ | Hide feature | MessageHistory.showError() |
| Warning (retryable) | ✅ | ✅ | Offer retry | MessageHistory.showWarning() |
| Success | ❌ | ✅ | Continue | MessageToast.show() |

## Production Patterns

### Pattern 1: Background Load (Silent Failure)

```javascript
async #loadResourceStatuses() {
    try {
        const aStatuses = await ApiClient.internal.plant.getDowntimeResourceStatuses();
        this.#aStatusList = aStatuses;
    } catch (oError) {
        // Log technical details
        this.#oLog.error("Error loading resource statuses", oError);
        // Don't show error to user - widget continues to work
    }
}
```

### Pattern 2: User Action with Busy State

```javascript
async _refreshDowntimes() {
    const oTable = this.getTable();
    try {
        oTable.setBusy(true);
        
        const sWorkCenter = PodContext.getFilterWorkCenters()[0].workCenter;
        const aRecords = await ApiClient.internal.oee.getDowntimesForWorkCenter({
            workcenter: sWorkCenter,
            startDateTime: DateTimeUtils.startOfDay(),
            endDateTime: DateTimeUtils.endOfDay()
        });
        
        this._updateDowntimeModel(aRecords);
    } catch (oError) {
        // Reset table
        this._updateDowntimeModel([]);
        
        // Show user-facing error
        MessageHistory.showError(
            this.getI18nText("error.fetchFailed", oError.message)
        );
        
        // Log technical details
        this.#oLog.error("Error fetching downtimes:", oError);
    } finally {
        oTable.setBusy(false);
    }
}
```

### Pattern 3: Warning with Retry

```javascript
try {
    const oResponse = await ApiClient.post(oPayload);
    if (oResponse.lineItems?.[0]?.error) {
        MessageHistory.showError(oResponse.lineItems[0].errorMessage);
        return;
    }
    MessageHistory.toast({ message: "Success", type: MessageHistory.Success });
} catch (oError) {
    const oErr = oError.body?.lineItems?.[0] || oError;
    
    if (oErr.errorCode === "warning.tolerance") {
        MessageHistory.showWarning(oErr.errorMessage, {
            actions: [MessageBox.Action.YES, MessageBox.Action.NO],
            onClose: (sAction) => {
                if (sAction === MessageBox.Action.YES) {
                    this.#model.setProperty("/ignoreWarnings", true);
                    this._onConfirmPost(); // RETRY
                }
            }
        });
    } else {
        MessageHistory.showError(oErr.errorMessage || "Error");
    }
}
```

### Pattern 4: Delete with Confirmation and Success

```javascript
async _deleteDowntimes(oDeleteRequest) {
    const iCount = oDeleteRequest.deleteDowntimes.length;
    
    try {
        await ApiClient.internal.oee.deleteDowntimes(oDeleteRequest);
    } catch (oError) {
        this.#oLog.error("Delete failed:", oError);
        MessageHistory.showError(
            this.getI18nText("error.deleteFailed", oError.message)
        );
        return;
    }
    
    // Success path
    this.getTable().removeSelections(true);
    await this._refreshDowntimes();
    
    MessageToast.show(
        iCount === 1 ? 
        this.getI18nText("success.deleteSingleSucceeded") :
        this.getI18nText("success.deleteSucceeded", iCount)
    );
}
```

## Best Practices

### ✅ DO
- Log all errors with Logger.error()
- Show user-friendly messages (no stack traces)
- Use i18n for all user-facing messages
- Set busy state during async operations
- Clear busy state in finally block
- Provide specific error messages per scenario
- Offer retry for transient errors
- Reset UI state on error

### ❌ DON'T
- Don't show technical details to users
- Don't use alert() or console.log() for errors
- Don't show errors for background operations
- Don't leave controls in busy state
- Don't use generic "An error occurred" messages
- Don't ignore errors silently (always log)
- Don't retry automatically without user consent

---

**See**: [form-dialog-patterns.md](form-dialog-patterns.md), [advanced-patterns.md](advanced-patterns.md)
