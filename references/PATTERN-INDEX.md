# POD 2.0 Pattern Index

Quick reference index organized by widget type, use case, technical pattern, and complexity.

---

## Quick Links

| Need | Go To |
|------|-------|
| **New to POD 2.0?** | [Getting Started](#i-need-to) |
| **Building an Action (button/execution)?** | [Action Patterns](action-patterns.md) ⭐ |
| **Building a table?** | [TableWidget Patterns](#tablewidget-patterns) |
| **Something not working?** | [Common Mistakes](common-mistakes.md) |
| **Memory leak?** | [Mistake #3: Missing onExit()](common-mistakes.md#mistake-3) |
| **POD Designer shows wrong properties?** | [Mistake #16](common-mistakes.md#mistake-16) |
| **Migrate from POD 1.0?** | [Migration Guide](#migration-from-pod-10) |

---

## FAQ

**Q: How do I create a simple widget?**  
→ [ControlWidget Pattern](widget-patterns.md) ⭐

**Q: How do I create a button that executes an SFC operation?**  
→ [Action Patterns](action-patterns.md) ⭐ — extend `SfcExecutionAction`, implement `execute()`

**Q: What's the difference between an Action and a Widget?**  
→ Actions = stateless execution triggered by buttons. Widgets = stateful display with lifecycle.

**Q: My widget is leaking memory!**  
→ [Mistake #3: Missing onExit()](common-mistakes.md#mistake-3) - ensure `onExit()` calls `PodContext.unsubscribe()`

**Q: POD Designer doesn't show my properties!**  
→ Check [EXCLUDE_PROPERTIES](production-patterns-unified.md#10-exclude_properties-pattern) isn't hiding them

**Q: How do I react to backend events in real time?**  
→ [notification-patterns.md](notification-patterns.md) - PodNotificationWebSocket and ManagedSubscription

**Q: How do I navigate to another page or open a dialog programmatically?**  
→ [PodRuntime](pod2-api-reference.md#podruntime) - `navigateToPage()`, `showDialog()`

**Q: How do I wait for data that might not be loaded yet?**  
→ `PodContext.getWhenAvailable(ModelPath.WorkListItems)` - see [pod2-api-reference.md](pod2-api-reference.md#core-data-access)

**Q: How do I call SAP DM APIs?**  
→ Use [ApiClient](pod2-api-reference.md#public-api-clients) - see [sapdm-api-reference.md](sapdm-api-reference.md) for 70+ APIs

**Q: How do I react to worklist selection changes?**  
→ [PodContext.subscribe()](pod2-api-reference.md#subscription-management) with `ModelPath.SelectedWorkListItems`

---

## Migration from POD 1.0

**Recommendation**: RE-ARCHITECT, don't directly convert.

POD 1.0 (XML views, controllers) and POD 2.0 (ES6 classes, programmatic views) are fundamentally different:

| POD 1.0 | POD 2.0 |
|---------|---------|
| XML Views | `_createView()` returns controls |
| Controller methods | Class methods |
| `this.getView().getModel()` | `this._getModel()` |
| `Component.js` | `extension.json` |
| `manifest.json` | Widget metadata static methods |

**Migration approach:**
1. Extract business logic from POD 1.0 controllers
2. Create new POD 2.0 widget using [widget-patterns.md](widget-patterns.md)
3. Adapt business logic to POD 2.0 patterns
4. Use [delegates](delegate-architecture.md) instead of custom data fetching

---

## By Widget Type

### Action Patterns (NEW)
| Pattern | File | Complexity | Description |
|---------|------|------------|-------------|
| Basic Action | action-patterns.md | ⭐⭐ | Extends SfcExecutionAction, getProperties() + execute() |
| Action with Properties | action-patterns.md | ⭐⭐⭐ | Configurable properties (Boolean, String, Integer, Dropdown) |
| Action with Dialog | action-patterns.md | ⭐⭐⭐ | Confirmation dialog before execution |
| Action with Partial Success | action-patterns.md | ⭐⭐⭐⭐ | Process response items, separate successes from failures |
| Action + Widget Combined | action-patterns.md | ⭐⭐⭐⭐ | Action updates context, Widget reacts |

### ControlWidget Patterns
| Pattern | File | Complexity | Description |
|---------|------|------------|-------------|
| Basic ControlWidget | widget-patterns.md | ⭐ | Single control wrapper (Button, Input, Text) |
| ControlWidget with Properties | widget-patterns.md | ⭐⭐ | Configurable properties via POD Designer |
| ControlWidget with Events | widget-patterns.md | ⭐⭐ | Custom event handling |

### LayoutWidget Patterns
| Pattern | File | Complexity | Description |
|---------|------|------------|-------------|
| Basic LayoutWidget | widget-patterns.md | ⭐⭐ | Container for child controls |
| LayoutWidget with Refresh | widget-patterns.md | ⭐⭐⭐ | API integration with refresh button |
| Master-Detail Nav | advanced-patterns.md | ⭐⭐⭐⭐ | NavContainer with navigation |

### TableWidget Patterns
| Pattern | File | Complexity | Description |
|---------|------|------------|-------------|
| Basic TableWidget | widget-patterns.md | ⭐⭐⭐ | Simple data table |
| TableWidget Complete | tablewidget-complete.md | ⭐⭐⭐⭐ | Full-featured with pagination |
| Growing Table | widget-patterns.md | ⭐⭐⭐⭐ | GrowingJSONModel pagination |
| Custom Toolbar | advanced-patterns.md | ⭐⭐⭐⭐ | Override _createToolbar() |
| Complex Cells | tablecell-patterns.md | ⭐⭐⭐⭐ | 13 cell types (text, date, actions, charts) |
| Selection Sync | common-mistakes.md | ⭐⭐⭐⭐⭐ | Bidirectional PodContext sync |

### ContentHandler Patterns
| Pattern | File | Complexity | Description |
|---------|------|------------|-------------|
| Dialog + Form | advanced-patterns.md | ⭐⭐⭐⭐⭐ | Complex form with validation |
| Custom Dialog | advanced-patterns.md | ⭐⭐⭐⭐ | Extend sap.m.Dialog directly |
| Form Validation | form-dialog-patterns.md | ⭐⭐⭐⭐⭐ | Real-time validation patterns |

---

## By Use Case

### Data Display
- [Basic Table](widget-patterns.md#tablewidget) - Simple data table
- [Growing Table](widget-patterns.md#tablewidget-with-growingjsonmodel-pagination-pattern) - Paginated table
- [Table Cells](tablecell-patterns.md) - 13 cell types
- [No-Data Messages](advanced-patterns.md#17-contextual-no-data-messages) - Context-aware messages

### Data Input & Forms
- [Form Patterns](form-dialog-patterns.md) - Complete form guide
- [ContentHandler](advanced-patterns.md#7-contenthandler-with-dialog-and-form) - Dialog forms
- [Validation](form-dialog-patterns.md#complex-form-validation) - Real-time validation
- [Custom Fields](advanced-patterns.md#12-custom-field-extensibility-pattern) - Extensible fields

### API Integration
- [API Patterns](widget-patterns.md#api-integration-widget) - REST API calls
- [Error Handling](advanced-patterns.md#15-error-handling--retry-pattern) - Complete error management
- [Retry Pattern](advanced-patterns.md#15-error-handling--retry-pattern) - Tolerance warnings

### State Management
- [PodContext Subscription](widget-patterns.md#context-aware-widget) - Subscribe to changes
- [Selection Sync](common-mistakes.md#mistake-27) - Table↔PodContext sync
- [Data Delegates](advanced-patterns.md#11-data-delegate-pattern) - Shared state
- [Optimistic UI](advanced-patterns.md#23-optimistic-ui-update-pattern) - Instant feedback
- [Real-Time Notifications](notification-patterns.md) - WebSocket events (SFC status, resource changes)
- [PodRuntime Navigation](pod2-api-reference.md#podruntime) - Programmatic page/dialog navigation

### User Interaction
- [Button Actions](widget-patterns.md#controlwidget) - Button press handlers
- [Popover](advanced-patterns.md#16-async-popover-pattern) - Async popovers
- [Warning Dialogs](advanced-patterns.md#13-warning-dialog-pattern) - Custom actions
- [Dynamic Enabling](advanced-patterns.md#6-dynamic-button-enabling-with-authorization) - Authorization checks

---

## By Technical Pattern

### Modern JavaScript
- [Private Fields (#)](advanced-patterns.md#1-modern-javascript-private-fields) - ES2022 encapsulation
- [Static Enums](advanced-patterns.md#2-static-propertyid-enum-pattern) - Object.freeze() type safety
- [JSDoc @extensible](advanced-patterns.md#16-jsdoc-extensible-markers-pattern) - Extension points

### Binding & Formatting
- [Multi-Part Binding](binding-patterns.md#multi-part-bindings) - Composite values
- [Expression Binding](advanced-patterns.md#14-expression-binding) - Computed properties
- [Formatter Classes](advanced-patterns.md#9-formatter-utility-class-pattern) - Reusable formatters
- [Defensive Formatters](common-mistakes.md#mistake-28) - Null-safe formatters

### Lifecycle & Memory
- [onExit Cleanup](common-mistakes.md#mistake-1) - Memory leak prevention
- [Dialog Destruction](common-mistakes.md#mistake-19) - afterClose destroy
- [Timer Management](production-patterns-unified.md#pattern-22) - Interval cleanup
- [Subscription Arrays](production-patterns-unified.md#pattern-13) - Multi-subscribe

### Enterprise Patterns
- [Dynamic Columns](advanced-patterns.md#14-dynamic-column-creation-pattern) - Conditional columns
- [UOM Handling](advanced-patterns.md#10-uom-unit-of-measure-handling-pattern) - Unit conversion
- [Authorization](advanced-patterns.md#6-dynamic-button-enabling-with-authorization) - Permission checks
- [Audit Trail](production-patterns-unified.md#pattern-18) - MessageHistory patterns

---

## By Complexity Level

### ⭐ Beginner (1 star)
- [Basic ControlWidget](widget-patterns.md#controlwidget)
- [Basic Properties](widget-patterns.md#widget-with-properties)

### ⭐⭐ Intermediate (2 stars)
- [LayoutWidget](widget-patterns.md#layoutwidget)
- [API Widget](widget-patterns.md#api-integration-widget)
- [Context Subscription](widget-patterns.md#context-aware-widget)

### ⭐⭐⭐ Advanced (3 stars)
- [Basic TableWidget](widget-patterns.md#tablewidget)
- [Growing Table](widget-patterns.md#tablewidget-with-growingjsonmodel-pagination-pattern)
- [Custom Toolbar](advanced-patterns.md#4-advanced-tablewidget---custom-toolbar)

### ⭐⭐⭐⭐ Expert (4 stars)
- [TableWidget Complete](tablewidget-complete.md)
- [Complex Cells](tablecell-patterns.md)
- [Custom Dialog](advanced-patterns.md#8-custom-dialog-extension-pattern)
- [Dynamic Columns](advanced-patterns.md#14-dynamic-column-creation-pattern)

### ⭐⭐⭐⭐⭐ Master (5 stars)
- [ContentHandler + Forms](advanced-patterns.md#7-contenthandler-with-dialog-and-form)
- [Error Handling](advanced-patterns.md#15-error-handling--retry-pattern)
- [Selection Sync](common-mistakes.md#mistake-27)
- [Complete Form Patterns](form-dialog-patterns.md)

---

## Validation Checklists

### Pre-Generation Checklist
Before generating ANY widget code:

- [ ] PlacementType from `"sap/m/PlacementType"` (not sap/ui/core/library)
- [ ] PodContext from `"sap/dm/dme/pod2/context/PodContext"` (not model/)
- [ ] ModelPath constants are exact (SelectedWorkListItems not Item)
- [ ] i18n uses static getI18nModel() pattern
- [ ] Defensive type checking with Array.isArray()

**See**: [Pre-Generation Checklist](../SKILL.md#pre-generation-validation-checklist)

### Post-Generation Checklist
After creating widget:

- [ ] Every `PodContext.subscribe()` has matching `unsubscribe()` in `onExit()`
- [ ] `onExit()` method exists and calls `super.onExit()`
- [ ] JSONModels initialized in `_createView()` BEFORE creating controls
- [ ] No binding syntax in WidgetProperty displayName/description
- [ ] Parent properties spread only for TableWidget/LayoutWidget
- [ ] Dialogs destroyed in afterClose handler

**See**: [Common Mistakes](common-mistakes.md) for all validation checks

### TableWidget Checklist
Specific to TableWidget:

- [ ] IGNORE_TABLE_PROPERTIES for widget config properties
- [ ] EXCLUDE_PROPERTIES for designer-hidden properties
- [ ] Selection sync: _syncSelectionsWithPodContext() implemented
- [ ] Initial sync called in onInit()
- [ ] _onSelectionChange preserves existing selections
- [ ] Multi-part formatters have null checks for ALL parameters

**See**: [Mistake #26](common-mistakes.md#mistake-26) and [Mistake #27](common-mistakes.md#mistake-27)

---

## Quick Pattern Selection

### I need to...

**Create an execution action (start/complete/signoff)** → [Action Patterns](action-patterns.md) ⭐⭐
**Display a single control** → [ControlWidget](widget-patterns.md#controlwidget) ⭐
**Show a container** → [LayoutWidget](widget-patterns.md#layoutwidget) ⭐⭐
**Display tabular data** → [TableWidget](widget-patterns.md#tablewidget) ⭐⭐⭐
**Create a form** → [ContentHandler](advanced-patterns.md#7-contenthandler-with-dialog-and-form) ⭐⭐⭐⭐⭐

**Call an API** → [API Widget](widget-patterns.md#api-integration-widget) ⭐⭐
**React to selections** → [Context Subscription](widget-patterns.md#context-aware-widget) ⭐⭐
**Paginate data** → [Growing Table](widget-patterns.md#tablewidget-with-growingjsonmodel-pagination-pattern) ⭐⭐⭐⭐
**Custom table columns** → [Dynamic Columns](advanced-patterns.md#14-dynamic-column-creation-pattern) ⭐⭐⭐⭐

**Validate input** → [Form Validation](form-dialog-patterns.md#complex-form-validation) ⭐⭐⭐⭐⭐
**Handle errors** → [Error Handling](advanced-patterns.md#15-error-handling--retry-pattern) ⭐⭐⭐⭐⭐
**Share data** → [Data Delegates](advanced-patterns.md#11-data-delegate-pattern) ⭐⭐⭐⭐
**Show warnings** → [Warning Dialog](advanced-patterns.md#13-warning-dialog-pattern) ⭐⭐⭐

---

## Common Pattern Combinations

### TableWidget + API + Pagination
1. Start with [TableWidget Complete](tablewidget-complete.md)
2. Add [GrowingJSONModel](widget-patterns.md#tablewidget-with-growingjsonmodel-pagination-pattern)
3. Implement [Selection Sync](common-mistakes.md#mistake-27)
4. Add [Complex Cells](tablecell-patterns.md) as needed

### Form + Validation + API
1. Start with [ContentHandler](advanced-patterns.md#7-contenthandler-with-dialog-and-form)
2. Add [Form Validation](form-dialog-patterns.md#complex-form-validation)
3. Implement [Error Handling](advanced-patterns.md#15-error-handling--retry-pattern)
4. Add [Custom Fields](advanced-patterns.md#12-custom-field-extensibility-pattern) if needed

### Context-Aware Table
1. Start with [Basic TableWidget](widget-patterns.md#tablewidget)
2. Add [PodContext Subscription](widget-patterns.md#context-aware-widget)
3. Implement [Selection Sync](common-mistakes.md#mistake-27)
4. Add [Contextual No-Data](advanced-patterns.md#17-contextual-no-data-messages)

---

## File Quick Reference

| File | Focus | Patterns Count |
|------|-------|----------------|
| action-patterns.md | Action templates, code quality rules | 3 patterns + production checklist |
| widget-patterns.md | Widget templates | 8 core patterns |
| tablewidget-complete.md | Complete TableWidget | 1 comprehensive guide |
| tablecell-patterns.md | Table cells | 13 cell types |
| binding-patterns.md | Bindings & formatters | 6 binding patterns |
| advanced-patterns.md | Enterprise patterns | 23 production patterns |
| production-patterns-unified.md | SAP patterns | 35 battle-tested patterns |
| form-dialog-patterns.md | Forms, dialogs & validation | 8 patterns |
| common-mistakes.md | Error prevention | 28 mistakes + fixes |
| delegate-architecture.md | Delegates | 7 official delegates |

---

## Version History

- **v1.0.0** (2026-04-18): Initial pattern index created
- Consolidates patterns from 5 major reference files
- 400+ lines of organized pattern references
- Covers all complexity levels (⭐ to ⭐⭐⭐⭐⭐)

**See**: [CHANGELOG.md](../CHANGELOG.md) for complete skill version history
