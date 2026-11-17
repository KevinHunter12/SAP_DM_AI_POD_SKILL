# POD 2.0 Line Monitor Widget Patterns

Reference for building widgets that integrate with the SAP DM Line Monitor (`sap.dm.dme.pod2.lm`). LM widgets are standard POD 2.0 widgets — they extend `Widget` using the same ES6 class patterns — but they use LM-specific APIs for indicator data and context.

---

## LM-Specific Context: LineMonitorModelPath

Line Monitor widgets read work center, time range, and shift data from dedicated paths in `PodContext`, separate from the standard execution paths.

**Import:** `sap/dm/dme/pod2/lm/context/LineMonitorModelPath`

```javascript
const LineMonitorModelPath = {
    WorkCenter:    "/lineMonitor/workCenter",   // sap.dm.dme.pod2.context.type.WorkCenter
    TimeRange:     "/lineMonitor/timeRange",    // LineMonitorTimeRange (extends TimeRange)
    CurrentShifts: "/lineMonitor/currentShifts" // Shift[]
};
```

These paths are set by the `WorkCenterSelection` and `TimeRangeSelection` LM widgets. Subscribe to changes with standard `PodContext.subscribe()`:

```javascript
import PodContext from "sap/dm/dme/pod2/context/PodContext";
import LineMonitorModelPath from "sap/dm/dme/pod2/lm/context/LineMonitorModelPath";

// In onInit():
PodContext.subscribe([LineMonitorModelPath.WorkCenter, LineMonitorModelPath.TimeRange],
    this._onSelectionChange, this);

// In onExit():
PodContext.unsubscribe([LineMonitorModelPath.WorkCenter, LineMonitorModelPath.TimeRange],
    this._onSelectionChange, this);

_onSelectionChange() {
    const oWorkCenter = PodContext.get(LineMonitorModelPath.WorkCenter);   // WorkCenter object
    const oTimeRange  = PodContext.get(LineMonitorModelPath.TimeRange);    // LineMonitorTimeRange object
    // oWorkCenter.workCenter — string
    // oTimeRange.startDate, oTimeRange.endDate — UI5Date
}
```

---

## Real-Time Indicator Data: IndicatorSubscriptionManager

The central class for receiving live sensor/indicator values via WebSocket. **Static singleton** — never instantiate it.

**Import:** `sap/dm/dme/pod2/lm/utils/IndicatorSubscriptionManager`

**WebSocket endpoint:** `/wss/dm/dm-linemonitor-indicator/api/v1/lmplugin/indicator?x-dme-plant=<plant>`

### Lifecycle

```javascript
// 1. onInit() — call init() once per POD (idempotent)
onInit() {
    if (!PodContext.isDesignMode()) {
        IndicatorSubscriptionManager.init();  // opens WS after PodRuntime.ready()
    }
}

// 2. _createView() — subscribe after init, before returning the view
_createView() {
    const oIndicator = this.getPropertyValue("indicator"); // IndicatorApiEntry object
    if (oIndicator && !PodContext.isDesignMode()) {
        IndicatorSubscriptionManager.subscribe(
            [oIndicator],   // array of IndicatorApiEntry objects
            this,           // widget instance — required for lifecycle management
            this._onNewValue.bind(this)  // optional callback
        );
    }
    // build and return view...
}

// 3. Bind your control to the shared model (named "lmIndicators")
// Model shape: { indicators: { [indicatorId]: IndicatorValue[] } }
// Values are newest-first. Index 0 = latest value.
myControl.setModel(IndicatorSubscriptionManager.getModel(), "lmIndicators");
myControl.bindProperty("text", { path: "lmIndicators>/indicators/MY_INDICATOR_ID/0/value" });

// 4. onExit() — REQUIRED cleanup
onExit() {
    IndicatorSubscriptionManager.unsubscribeWidget(this);
    super.onExit();
}
```

### API

```javascript
IndicatorSubscriptionManager.init()
// Initialises the WebSocket. Idempotent — safe to call from every widget's onInit().
// Must be called before PodRuntime.ready() resolves (i.e. from onInit(), not _createView()).

IndicatorSubscriptionManager.subscribe(vIndicators, oWidget, fnCallback?)
// vIndicators: IndicatorApiEntry | IndicatorApiEntry[]
// oWidget:     Widget instance (required)
// fnCallback:  optional (sIndicatorId: string, oValue: IndicatorValue) => void
// Deduplicates by indicator id — backend subscribed only once per indicator.

IndicatorSubscriptionManager.getModel()
// Returns the shared JSONModel (name it "lmIndicators" when setting on controls).
// Shape: { indicators: { [indicatorId: string]: IndicatorValue[] } }

IndicatorSubscriptionManager.unsubscribeWidget(oWidget)
// Detaches all callbacks for this widget. Call from onExit(). Does NOT close the WS
// unless this was the last widget — other widgets sharing the same indicators are unaffected.

IndicatorSubscriptionManager.isConnected()  // boolean — WS open?
```

### IndicatorValue model shape

Each entry in `indicators[id]` (newest first, max 1800 entries / 30 min at 1/sec):

```javascript
{
    indicator: {          // Indicator object
        id, indicatorName, structurePath, structure, workCenter, equipment
    },
    value:          string,   // the raw value
    uom:            string,   // unit of measure
    valueTimestamp: UI5Date
}
```

Bind in XML views:
```xml
<!-- Latest value -->
<Text text="{lmIndicators>/indicators/MY_ID/0/value}" />
<!-- Latest with UOM -->
<Text text="{parts: [{path: 'lmIndicators>/indicators/MY_ID/0/value'},
                     {path: 'lmIndicators>/indicators/MY_ID/0/uom'}],
             formatter: '.formatValueUom'}" />
```

---

## Fetching Available Indicators: IndicatorApiClient

Use at design time to populate an indicator picker, or at runtime to resolve indicator metadata.

**Import:** `sap/dm/dme/pod2/lm/api/IndicatorApiClient`

```javascript
// Returns IndicatorApiEntry[] for the current plant
const aIndicators = await IndicatorApiClient.getIndicators();

// IndicatorApiEntry shape:
{
    id:            string,  // composite key: resourceRef + structurePath + structure + name
    workCenter:    string,
    resourceRef:   string,  // resource BO ref
    equipment:     string,  // equipmentId
    equipmentName: string,
    description:   string,
    dataType:      IndicatorDataType,  // "NUMERIC" | "STRING" | "BOOLEAN" | "DATE"
    indicatorName: string,
    structurePath: string,
    structure:     string
}
```

---

## WidgetCategory.LineMonitor

LM widgets should declare their category as `WidgetCategory.LineMonitor` so they appear in the correct section of the POD Designer palette:

```javascript
static getCategory() {
    return WidgetCategory.LineMonitor;  // "lineMonitor"
}
```

Import: `sap/dm/dme/pod2/widget/metadata/WidgetCategory`

---

## Property Editor: IndicatorValueHelpPropertyEditor

A custom property editor that provides a value-help dialog for selecting an indicator at design time. Stores the full `IndicatorApiEntry` object as the property value.

**Import:** `sap/dm/dme/pod2/lm/propertyeditor/IndicatorValueHelpPropertyEditor`

```javascript
new WidgetProperty({
    displayName: "Indicator",
    category: PropertyCategory.Main,
    propertyEditor: new IndicatorValueHelpPropertyEditor(
        this,
        "indicator",   // propertyId — stores full IndicatorApiEntry
        {
            workCenterPropertyId: "workCenter",   // sibling property to also set
            resourcePropertyId:   "resource",     // sibling property to also set
            indicatorPropertyId:  "indicatorName", // sibling display-name property
            allowedDataTypes: [IndicatorDataType.NUMERIC]  // optional filter
        }
    )
})
```

When the user picks an indicator in the dialog, the editor also writes the work center and resource name into the sibling properties automatically.

---

## Complete Minimal Widget Example

```javascript
sap.ui.define([
    "sap/dm/dme/pod2/context/PodContext",
    "sap/dm/dme/pod2/control/CustomText",
    "sap/dm/dme/pod2/model/I18nResourceModel",
    "sap/dm/dme/pod2/widget/Widget",
    "sap/dm/dme/pod2/widget/metadata/WidgetCategory",
    "sap/dm/dme/pod2/widget/metadata/WidgetProperty",
    "sap/dm/dme/pod2/lm/propertyeditor/IndicatorValueHelpPropertyEditor",
    "sap/dm/dme/pod2/lm/utils/IndicatorSubscriptionManager",
    "sap/dm/dme/pod2/propertyeditor/PropertyCategory"
], (PodContext, CustomText, I18nResourceModel, Widget, WidgetCategory, WidgetProperty,
    IndicatorValueHelpPropertyEditor, IndicatorSubscriptionManager, PropertyCategory) => {
    "use strict";

    class MyIndicatorWidget extends Widget {

        static #oI18nModel = new I18nResourceModel({ bundleName: "my.namespace.i18n.i18n" });

        static PropertyId = Object.freeze({
            Indicator:     "indicator",
            WorkCenter:    "workCenter",
            Resource:      "resource",
            IndicatorName: "indicatorName"
        });

        static getI18nModel()    { return this.#oI18nModel; }
        static getCategory()     { return WidgetCategory.LineMonitor; }
        static getDefaultConfig() {
            return { properties: {
                indicator: null, workCenter: "", resource: "", indicatorName: ""
            }};
        }

        #oDisplay;

        onInit() {
            if (!PodContext.isDesignMode()) {
                IndicatorSubscriptionManager.init();
            }
        }

        _createView() {
            const oIndicator = this.getPropertyValue(MyIndicatorWidget.PropertyId.Indicator);

            if (oIndicator && !PodContext.isDesignMode()) {
                IndicatorSubscriptionManager.subscribe([oIndicator], this);
            }

            this.#oDisplay = new CustomText({ width: "100%" });

            if (oIndicator && !PodContext.isDesignMode()) {
                this.#oDisplay.setModel(IndicatorSubscriptionManager.getModel(), "lmIndicators");
                this.#oDisplay.bindProperty("text", {
                    path: `lmIndicators>/indicators/${oIndicator.id}/0/value`
                });
            }

            return this.#oDisplay;
        }

        onExit() {
            IndicatorSubscriptionManager.unsubscribeWidget(this);
            super.onExit();
        }

        getProperties() {
            return [
                new WidgetProperty({
                    displayName: "Indicator",
                    category: PropertyCategory.Main,
                    propertyEditor: new IndicatorValueHelpPropertyEditor(
                        this, MyIndicatorWidget.PropertyId.Indicator, {
                            workCenterPropertyId: MyIndicatorWidget.PropertyId.WorkCenter,
                            resourcePropertyId:   MyIndicatorWidget.PropertyId.Resource,
                            indicatorPropertyId:  MyIndicatorWidget.PropertyId.IndicatorName
                        }
                    )
                })
            ];
        }
    }

    return MyIndicatorWidget;
});
```

---

## Key Differences vs Standard POD 2.0 Widgets

| Aspect | Standard POD 2.0 | Line Monitor |
|---|---|---|
| Work center data | `PodContext.getFilterWorkCenters()` | `PodContext.get(LineMonitorModelPath.WorkCenter)` |
| Time range data | N/A (execution widgets don't use time ranges) | `PodContext.get(LineMonitorModelPath.TimeRange)` |
| Real-time data | `PodNotificationWebSocket` / `ManagedSubscription` | `IndicatorSubscriptionManager` |
| Context subscription | `PodContext.subscribe(ModelPath.*, cb)` | `PodContext.subscribe(LineMonitorModelPath.*, cb)` |
| Widget category | `WidgetCategory.Custom` or others | `WidgetCategory.LineMonitor` |
| Indicator selection | N/A | `IndicatorValueHelpPropertyEditor` |
