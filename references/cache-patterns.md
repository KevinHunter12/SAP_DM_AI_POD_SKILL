# Cache Patterns for POD 2.0 Plugins

## Static Cache Pattern ⭐⭐⭐⭐⭐

### Overview
Static class with cached data shared across widget instances, preventing redundant API calls.

### When to Use
- ✅ Large datasets used by multiple widgets (UOMs, resources, reason codes)
- ✅ Data that doesn't change frequently
- ✅ Expensive API calls (hierarchy data, configuration)
- ✅ Master data (plants, work centers)
- ❌ Real-time transactional data
- ❌ User-specific data that varies per context

### Production Example

```javascript
import ApiClient from "sap/dm/dme/pod2/ApiClient";
import PodContext from "sap/dm/dme/pod2/PodContext";
import MessageBox from "sap/m/MessageBox";

/**
 * Static cache for resource hierarchy data across widgets
 * @public
 * @alias sap.dm.dme.pod2.widget.oee.ResourceHierarchyCache
 */
class ResourceHierarchyCache {
    static #aCachedData = null;
    
    /**
     * Ensures the resource hierarchy data is loaded.
     * Uses single-flight pattern to prevent multiple simultaneous loads.
     * @returns {Promise<Array>} Promise resolving to the hierarchy data
     */
    static async getResourceHierarchy() {
        // If data already exists, return it immediately
        if (this.#aCachedData) {
            return Promise.resolve(this.#aCachedData);
        }
        return await this._loadResourceHierarchy();
    }
    
    /**
     * Loads the resource hierarchy data from the backend
     * @returns {Promise<Array>} Promise resolving to the hierarchy data
     */
    static async _loadResourceHierarchy() {
        try {
            const sPlant = PodContext.getPlant();
            const aWorkCenters = PodContext.getFilterWorkCenters();
            const sWorkCenter = aWorkCenters?.[0]?.workCenter || null;
            
            // Load data from API
            const oData = await ApiClient.internal.plant
                .getResourceHierarchyData(sPlant, sWorkCenter);
            
            // Transform data
            const oHierarchyData = this._prepareWorkcenterData(oData);
            
            // Cache transformed data
            this.#aCachedData = oHierarchyData;
            
            return oHierarchyData;
        } catch (oError) {
            MessageBox.error(
                PodContext.getI18nText("error.resourceHierarchyFailed", oError.message)
            );
            throw oError;
        }
    }
    
    /**
     * Prepares workcenter data by transforming hierarchy nodes
     * @param {Object} oWorkCenterData - Raw data from API
     * @returns {Array} Transformed hierarchy data
     */
    static _prepareWorkcenterData(oWorkCenterData) {
        const aFinalData = [];
        
        if (oWorkCenterData.oeeHierarchyNodes?.length > 0) {
            const aTransformed = this._transformHierarchy(oWorkCenterData.oeeHierarchyNodes);
            aFinalData.push(...aTransformed);
            
            // Build set of unique resources in hierarchy
            const oResourceSet = this._getUniqueResources(aFinalData);
            
            // Add members not in hierarchy
            const aMembers = this._transformMembers(oWorkCenterData.members);
            aFinalData.push(...aMembers.filter(o => !oResourceSet.has(o.resourceName)));
        } else if (oWorkCenterData.members?.length > 0) {
            aFinalData.push(...this._transformMembers(oWorkCenterData.members));
        }
        
        return aFinalData;
    }
    
    /**
     * Transforms hierarchy nodes recursively
     */
    static _transformHierarchy(aParentNodes) {
        if (!Array.isArray(aParentNodes)) return [];
        
        return aParentNodes.map(oNode => ({
            id: oNode.id,
            resourceName: oNode.resource?.resource,
            isBottleneck: oNode.isBottleneck,
            childNodes: this._transformHierarchy(
                oNode.childNodes?.filter(c => c.nodeStatus === "ENABLED")
            )
        }));
    }
    
    /**
     * Cache invalidation (optional - add if needed)
     */
    static clearCache() {
        this.#aCachedData = null;
    }
}

// USAGE FROM WIDGET:
class MyWidget extends Widget {
    async _loadResourceHierarchy() {
        try {
            // This uses cached data if available, or loads if not
            const oData = await ResourceHierarchyCache.getResourceHierarchy();
            this.#oModel.setProperty("/resourceHierarchy", oData);
        } catch (oError) {
            this.#oLog.error("Failed to load resource hierarchy:", oError);
        }
    }
}

// USAGE FROM DIALOG (preload in background):
class MyDialog {
    _onDialogAfterOpen() {
        // Preload cache in background (fire and forget)
        ResourceHierarchyCache.getResourceHierarchy().catch(oError => {
            this.#oLog.error("Error preloading resource hierarchy:", oError);
        });
    }
}
```

### Cache Invalidation Strategies

#### 1. No Invalidation (Simplest)
```javascript
static #aCachedData = null; // Cache lasts for session lifetime
```

#### 2. Manual Invalidation
```javascript
static clearCache() {
    this.#aCachedData = null;
}

// Call when data changes
await ApiClient.internal.plant.updateResource(...);
ResourceHierarchyCache.clearCache();
```

#### 3. Time-Based Expiration
```javascript
static #aCachedData = null;
static #cacheTimestamp = null;
static #CACHE_TTL = 5 * 60 * 1000; // 5 minutes

static async getResourceHierarchy() {
    const now = Date.now();
    if (this.#aCachedData && this.#cacheTimestamp && 
        (now - this.#cacheTimestamp) < this.#CACHE_TTL) {
        return Promise.resolve(this.#aCachedData);
    }
    return await this._loadResourceHierarchy();
}

static async _loadResourceHierarchy() {
    // ... load data ...
    this.#aCachedData = oHierarchyData;
    this.#cacheTimestamp = Date.now();
    return oHierarchyData;
}
```

#### 4. PodContext Subscription
```javascript
// Invalidate when work center changes
static init() {
    PodContext.subscribe(
        ModelPath.FilterWorkCenters,
        () => this.clearCache(),
        this
    );
}
```

### Best Practices
- ✅ Use static private field for cached data
- ✅ Single-flight pattern (prevent concurrent loads)
- ✅ Transform data once, cache transformed result
- ✅ Provide clearCache() method for manual invalidation
- ✅ Handle errors gracefully (don't cache errors)
- ✅ Document cache lifetime and invalidation strategy
- ❌ Don't cache user-specific data
- ❌ Don't cache real-time data

---

**See**: [dialog-patterns.md](dialog-patterns.md), [advanced-patterns.md](advanced-patterns.md)
