# POD 2.0 Notification & WebSocket Patterns

Real-time event subscription for SAP Digital Manufacturing POD 2.0 plugins.

---

## Overview

The notification system provides WebSocket-based pub/sub for real-time events like SFC status changes, resource status changes, and production events. Two classes support this:

- **`PodNotificationWebSocket`** — one-shot subscribe/unsubscribe
- **`ManagedSubscription`** — lifecycle-aware wrapper that auto-manages the subscription

---

## Imports

```javascript
import PodNotificationWebSocket from "sap/dm/dme/pod2/notification/PodNotificationWebSocket";
import ManagedSubscription from "sap/dm/dme/pod2/notification/ManagedSubscription";
// EventType and Filter are sub-namespaces accessed via their full paths
// Check JSDoc for the exact EventType enum values available in your version
```

---

## PodNotificationWebSocket

**Class:** `sap.dm.dme.pod2.notification.PodNotificationWebSocket`  
**All methods static.**

### subscribe()

```javascript
/**
 * Subscribe to a notification topic/event type
 * @param {Object} oOptions
 * @param {string} oOptions.eventType - Event type enum value
 * @param {Function} oOptions.onMessage - Callback: (oPayload) => void
 * @param {AbstractFilter} [oOptions.filter] - Optional filter
 * @param {string} [oOptions.topic] - Optional topic override
 * @param {string} [oOptions.description] - Optional description for debugging
 * @returns {SubscriptionContext} — use to unsubscribe later
 */
static subscribe(oOptions): SubscriptionContext
```

### Example

```javascript
class MyWidget extends Widget {
    #oSubscriptionContext;

    async onInit() {
        await super.onInit();

        if (PodContext.isRunMode()) {
            // Subscribe to SFC status change events for current plant
            this.#oSubscriptionContext = PodNotificationWebSocket.subscribe({
                eventType: "SFC_STATUS_CHANGED",  // Use EventType.* enum in real code
                onMessage: (oPayload) => this._onSfcStatusChanged(oPayload),
                description: "MyWidget SFC status listener"
            });
        }
    }

    _onSfcStatusChanged(oPayload) {
        // oPayload contains the notification data
        // Typically refresh the worklist or update UI
        WorkListDelegate.refreshInBackground();
    }

    onExit() {
        // REQUIRED: unsubscribe to prevent memory leaks
        if (this.#oSubscriptionContext) {
            this.#oSubscriptionContext.unsubscribe();
            this.#oSubscriptionContext = null;
        }
        super.onExit();
    }
}
```

---

## ManagedSubscription

**Class:** `sap.dm.dme.pod2.notification.ManagedSubscription`

A lifecycle-aware wrapper that automatically manages subscription state based on criteria. Use when your subscription filter depends on POD context (e.g., current resource or work center) that can change.

### Constructor

```javascript
/**
 * @param {Object} mSettings
 * @param {Function} mSettings.getFilter - Returns AbstractFilter|null.
 *   Return null when subscription criteria not met (auto-unsubscribes).
 * @param {Function} mSettings.onMessage - Callback: (oPayload) => void
 * @param {string} [mSettings.eventType] - Event type
 * @param {string} [mSettings.description] - Debug description
 */
new ManagedSubscription(mSettings)
// NOTE: Constructor automatically calls update() on creation.
```

### Methods

```javascript
/**
 * Re-evaluate subscription criteria. Call when context changes
 * (e.g. when resource filter changes) to subscribe/unsubscribe accordingly.
 */
update(): void

/**
 * Unsubscribe regardless of criteria. For when you need to force-stop.
 */
unsubscribe(): void

/**
 * Unsubscribe and prevent future reuse. Call in onExit().
 */
destroy(): void
```

### Example

```javascript
class ResourceStatusWidget extends Widget {
    #oManagedSubscription;

    async onInit() {
        await super.onInit();

        if (PodContext.isRunMode()) {
            // Create subscription that adapts to current resource filter
            this.#oManagedSubscription = new ManagedSubscription({
                eventType: "RESOURCE_STATUS_CHANGED",
                description: "ResourceStatusWidget listener",
                getFilter: () => {
                    const aResources = PodContext.getFilterResources();
                    if (!aResources || aResources.length === 0) {
                        return null;  // No resource selected — don't subscribe
                    }
                    // Return a filter scoped to the current resource
                    // (exact Filter class depends on your SAP DM version)
                    return { resource: aResources[0].resource };
                },
                onMessage: (oPayload) => this._onResourceStatusChanged(oPayload)
            });

            // Re-evaluate when resource selection changes
            PodContext.subscribe(
                ModelPath.FilterResources,
                this._onResourceFilterChanged,
                this
            );
        }
    }

    _onResourceFilterChanged() {
        // Tell ManagedSubscription to re-evaluate its filter
        this.#oManagedSubscription?.update();
        this._refreshDisplay();
    }

    _onResourceStatusChanged(oPayload) {
        this._refreshDisplay();
    }

    onExit() {
        // destroy() handles both unsubscribe + cleanup
        this.#oManagedSubscription?.destroy();
        this.#oManagedSubscription = null;

        PodContext.unsubscribe(
            ModelPath.FilterResources,
            this._onResourceFilterChanged,
            this
        );
        super.onExit();
    }
}
```

---

## Filter Class

**Source:** `sap/dm/dme/pod2/notification/Filter`

Use `Filter` to scope a `PodNotificationWebSocket` or `ManagedSubscription` to specific field values. Filters are built with a fluent API.

```javascript
import Filter from "sap/dm/dme/pod2/notification/Filter";
```

### Comparison Methods (return `Filter` instance for chaining)

```javascript
Filter.equals("fieldName", "value")          // field === value
Filter.notEquals("fieldName", "value")       // field !== value
Filter.greaterThan("fieldName", value)       // field > value
Filter.lessThan("fieldName", value)          // field < value
Filter.equalsAny("fieldName", ["a", "b"])    // field is one of these values
Filter.notEqualsAll("fieldName", ["a", "b"]) // field is none of these values
```

### Logical Combinators (return `Filter` instance for chaining)

```javascript
filter.and(anotherFilter)   // both must match
filter.or(anotherFilter)    // either must match
```

### Example — Scoped Resource Subscription

```javascript
import Filter from "sap/dm/dme/pod2/notification/Filter";

getFilter: () => {
    const aResources = PodContext.getFilterResources();
    if (!aResources?.length) return null;  // no resource selected — don't subscribe

    // Only receive events for the currently selected resource
    return Filter.equals("resource", aResources[0].resource);
}
```

### Example — Combined Filter

```javascript
// Receive events for plant 1010 where status is ACTIVE or IN_QUEUE
const oFilter = Filter.equals("plant", "1010")
    .and(Filter.equalsAny("status", ["ACTIVE", "IN_QUEUE"]));
```

---

## SubscriptionContext

The object returned by `PodNotificationWebSocket.subscribe()`. Store it as a private field and call `.unsubscribe()` in `onExit()`.

```javascript
// returned from PodNotificationWebSocket.subscribe(...)
this.#oSubscriptionContext = PodNotificationWebSocket.subscribe({ ... });

// In onExit():
this.#oSubscriptionContext?.unsubscribe();
this.#oSubscriptionContext = null;
```

`ManagedSubscription` manages its own `SubscriptionContext` internally — you never call `.unsubscribe()` on one directly; call `managedSub.destroy()` instead.

---

## Key Differences

| | PodNotificationWebSocket | ManagedSubscription |
|-|--------------------------|---------------------|
| Use when | Simple, static subscription | Filter depends on changing context |
| Lifecycle | Manual subscribe/unsubscribe | Auto-manages via `update()` |
| Cleanup | Call `context.unsubscribe()` | Call `destroy()` |
| Constructor | N/A (static) | Calls `update()` automatically |
| Filter | Pass `Filter` instance in options | Return `Filter` from `getFilter()` |

---

## When to Use Real-Time Notifications

Use notifications when your widget needs to react to **backend events** without user interaction:

- ✅ Refresh worklist when an SFC status changes elsewhere
- ✅ Update resource status icon when a machine goes down
- ✅ Show alerts when a quality inspection result arrives
- ✅ Auto-advance to next operation when current operation completes

For data that updates on **user selection** (clicking a worklist item), use `PodContext.subscribe(ModelPath.SelectedWorkListItems, ...)` instead — that's simpler and more appropriate.

---

## WorkListDelegate Integration

The most common real-time pattern is refreshing the worklist when a background event occurs:

```javascript
import WorkListDelegate from "sap/dm/dme/pod2/context/data/WorkListDelegate";

_onSfcStatusChanged(oPayload) {
    const sSfc = oPayload?.sfc;

    if (sSfc) {
        // Only refresh if the changed SFC is currently visible
        WorkListDelegate.refreshStaleSfcs(sSfc);
    } else {
        // Full background refresh without clearing the list
        WorkListDelegate.refreshInBackground();
    }
}
```
