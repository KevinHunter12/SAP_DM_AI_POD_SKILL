# Tree & TreeTable Patterns for POD 2.0 Plugins

## TreeTable Pattern ⭐⭐⭐⭐

### When to Use
- ✅ Hierarchical data with columns (reason codes, BOMs, hierarchies)
- ✅ Leaf-only selection needed
- ✅ Tabular display of tree structure
- ❌ Simple lists (use Table instead)
- ❌ Custom item rendering (use Tree instead)

### Production Example

```javascript
import TreeTable from "sap/ui/table/TreeTable";
import Column from "sap/ui/table/Column";
import Label from "sap/m/Label";
import Text from "sap/m/Text";
import SelectionMode from "sap/ui/table/SelectionMode";
import SelectionBehavior from "sap/ui/table/SelectionBehavior";
import Auto from "sap/ui/table/rowmodes/Auto";

#createTreeTable() {
    const oTreeTable = new TreeTable({
        rows: {
            path: "reasonCodeModel>/timeElementReasonCodeTree",
            parameters: {
                arrayNames: ["reasonCodes"]  // Property containing children
            }
        },
        selectionMode: SelectionMode.Single,
        selectionBehavior: SelectionBehavior.RowOnly,
        enableSelectAll: false,
        expandFirstLevel: true,
        columnHeaderVisible: true,
        rowMode: new Auto(),
        rowSelectionChange: () => {
            const aIndices = oTreeTable.getSelectedIndices();
            if (aIndices.length > 0) {
                const oContext = oTreeTable.getContextByIndex(aIndices[0]);
                const oSelected = oContext?.getObject();
                
                // Only allow selection of leaf nodes (no children)
                if (oSelected && !oSelected.reasonCodes && 
                    oSelected.typeOfElement !== "timeElement") {
                    this.#oModel.setProperty("/selectedReasonCode", oSelected);
                } else {
                    this.#oModel.setProperty("/selectedReasonCode", null);
                }
            }
        },
        columns: [
            new Column({
                label: new Label({ text: "Description" }),
                template: new Text({ text: "{reasonCodeModel>description}" })
            }),
            new Column({
                label: new Label({ text: "Code" }),
                template: new Text({ text: "{reasonCodeModel>id}" })
            })
        ]
    });
    
    oTreeTable.setModel(this.#oModel, "reasonCodeModel");
    return oTreeTable;
}
```

### Data Structure

```javascript
{
    id: "parent1",
    description: "Parent Node",
    reasonCodes: [  // Must match arrayNames parameter
        {
            id: "child1",
            description: "Child Node",
            reasonCodes: []  // Leaf nodes have empty array
        }
    ]
}
```

## Tree Pattern (Multi-Select) ⭐⭐⭐

### When to Use
- ✅ Hierarchical data with custom rendering
- ✅ Multi-select needed
- ✅ Rich item content (icons, badges, etc.)
- ❌ Simple tabular data (use TreeTable)

### Production Example

```javascript
import Tree from "sap/m/Tree";
import CustomTreeItem from "sap/m/CustomTreeItem";
import ListMode from "sap/m/ListMode";
import VBox from "sap/m/VBox";
import Text from "sap/m/Text";
import HBox from "sap/m/HBox";
import Label from "sap/m/Label";

_createResourceTree() {
    const oTree = new Tree({
        mode: ListMode.MultiSelect,
        includeItemInSelection: true,
        selectionChange: this._onRowSelectionChange.bind(this),
        rememberSelections: true,
        items: {
            path: "resourceHierarchy>/childNodes",
            template: new CustomTreeItem({
                content: [
                    new VBox({
                        items: [
                            new Text({
                                text: "{resourceHierarchy>resourceName}"
                            }),
                            new HBox({
                                items: [
                                    new Label({ text: "Bottleneck:" }),
                                    new Text({
                                        text: {
                                            parts: ["resourceHierarchy>isBottleneck"],
                                            formatter: (bBottleneck) => bBottleneck ? "Yes" : "No"
                                        }
                                    })
                                ]
                            })
                        ]
                    })
                ]
            })
        }
    });
    
    return oTree;
}

_onRowSelectionChange(oEvent) {
    const oTree = oEvent.getSource();
    const aSelectedItems = oTree.getSelectedItems();
    const aSelectedResources = aSelectedItems.map(oItem => {
        const oContext = oItem.getBindingContext("resourceHierarchy");
        return oContext.getProperty("resourceName");
    });
    this.#oModel.setProperty("/selectedResources", aSelectedResources);
}
```

## Recursive Transformation Pattern ⭐⭐⭐⭐

```javascript
/**
 * Transforms API response to tree structure
 * @param {Array} aResourceReasonCodeNodes Raw API nodes
 * @param {number} iLevel Current depth
 * @returns {Array} Transformed tree nodes
 */
_toReasonCodeTreeNode(aResourceReasonCodeNodes, iLevel = 1) {
    if (!Array.isArray(aResourceReasonCodeNodes)) return [];
    
    return aResourceReasonCodeNodes.map(oReasonCodeNode => {
        const oTreeNode = {
            id: oReasonCodeNode.reasonCode,
            description: oReasonCodeNode.description,
            keyId: oReasonCodeNode.ref
        };
        
        // Recursively transform children
        const aChildren = oReasonCodeNode.resourceReasonCodeNodeCollection;
        if (Array.isArray(aChildren) && aChildren.length > 0) {
            oTreeNode.reasonCodes = this._toReasonCodeTreeNode(aChildren, iLevel + 1);
        }
        
        return oTreeNode;
    });
}
```

## Tree Filtering Pattern ⭐⭐⭐

```javascript
import Filter from "sap/ui/model/Filter";
import FilterOperator from "sap/ui/model/FilterOperator";

_onResourceHierarchySearch(oEvent) {
    const sQuery = oEvent.getParameter("newValue");
    const oTree = this._getTree();
    const oBinding = oTree.getBinding("items");
    
    if (sQuery && sQuery.length > 0) {
        const aFilters = [
            new Filter("resourceName", FilterOperator.Contains, sQuery)
        ];
        const oCombinedFilter = new Filter({ filters: aFilters, and: false });
        oBinding.filter(oCombinedFilter);
    } else {
        oBinding.filter([]);
    }
}
```

## Pre-selection Pattern ⭐⭐⭐

```javascript
async open() {
    await this._loadResourceHierarchy();
    
    // Pre-select tokens
    const oTree = this._getTree();
    if (oTree && this.#aSelectedResources.length > 0) {
        const oSelectedSet = new Set(this.#aSelectedResources);
        for (const oItem of oTree.getItems()) {
            const oContext = oItem.getBindingContext("resourceHierarchy");
            if (oContext) {
                const sResourceName = oContext.getProperty("resourceName");
                if (oSelectedSet.has(sResourceName)) {
                    oTree.setSelectedItem(oItem, true);
                }
            }
        }
    }
    
    this._getDialog().open();
}
```

## Best Practices

### ✅ DO
- Use TreeTable for tabular hierarchies
- Use Tree for rich custom content
- Specify arrayNames in binding parameters
- Handle leaf-only selection in logic
- Transform data recursively
- Pre-select items after data loads
- Filter hierarchies with UI5 filters

### ❌ DON'T
- Don't forget arrayNames parameter (crashes binding)
- Don't allow parent node selection without checking
- Don't create circular references in data
- Don't flatten hierarchies unnecessarily
- Don't pre-select before data loads

---

**See**: [form-patterns.md](form-patterns.md#token-input), [dialog-patterns.md](dialog-patterns.md), [cache-patterns.md](cache-patterns.md)
