# SAP Digital Manufacturing Supported Languages

Complete list of all languages supported by SAP Digital Manufacturing for i18n localization.

**Source**: SAP Note 2722461 - "Languages Supported in SAP Digital Manufacturing"

---

## Supported Languages (28 Total)

SAP Digital Manufacturing supports 28 languages for user interface localization and i18n resource bundles.

| # | Language | ISO 639-1 Code | i18n File Suffix | SAP Language Code |
|---|----------|----------------|------------------|-------------------|
| 1 | Bulgarian | bg | `i18n_bg.properties` | BG |
| 2 | Chinese (Simplified) | zh | `i18n_zh.properties` | ZH |
| 3 | Chinese (Traditional) | zh-TW | `i18n_zh_TW.properties` | ZF |
| 4 | Croatian | hr | `i18n_hr.properties` | HR |
| 5 | Czech | cs | `i18n_cs.properties` | CS |
| 6 | Danish | da | `i18n_da.properties` | DA |
| 7 | Dutch | nl | `i18n_nl.properties` | NL |
| 8 | English | en | `i18n_en.properties` | EN |
| 9 | French | fr | `i18n_fr.properties` | FR |
| 10 | German | de | `i18n_de.properties` | DE |
| 11 | Hungarian | hu | `i18n_hu.properties` | HU |
| 12 | Italian | it | `i18n_it.properties` | IT |
| 13 | Japanese | ja | `i18n_ja.properties` | JA |
| 14 | Korean | ko | `i18n_ko.properties` | KO |
| 15 | Lithuanian | lt | `i18n_lt.properties` | LT |
| 16 | Polish | pl | `i18n_pl.properties` | PL |
| 17 | Portuguese (Brazilian) | pt-BR | `i18n_pt.properties` | PT |
| 18 | Romanian | ro | `i18n_ro.properties` | RO |
| 19 | Russian | ru | `i18n_ru.properties` | RU |
| 20 | Serbian (Latin) | sr | `i18n_sr.properties` | SH |
| 21 | Slovak | sk | `i18n_sk.properties` | SK |
| 22 | Slovenian | sl | `i18n_sl.properties` | SL |
| 23 | Spanish | es | `i18n_es.properties` | ES |
| 24 | Swedish | sv | `i18n_sv.properties` | SV |
| 25 | Thai | th | `i18n_th.properties` | TH |
| 26 | Turkish | tr | `i18n_tr.properties` | TR |
| 27 | Ukrainian | uk | `i18n_uk.properties` | UK |
| 28 | Vietnamese | vi | `i18n_vi.properties` | VI |

---

## Important Notes

### Serbian Language Code Distinction
**Two codes exist for Serbian** — use the right one depending on context:
- **i18n file naming**: use `sr` → `i18n_sr.properties`
- **`LanguageUtils` BCP-47 portal code**: use `sh` → `LanguageUtils.getLanguage("sh")`

The BTP Portal personalization service requires `sh` (the code used in `LanguageUtils`), while the i18n bundle file is named with `sr`. SAPUI5 resolves both correctly internally.

### Spanish Language Support
**Only one variant of Spanish is supported** - SAP uses a generic Spanish (ES) that covers multiple regions. Other Spanish variants (e.g., Mexican Spanish, Argentine Spanish) are **not supported** as separate languages.

### Portuguese Language Support
**Brazilian Portuguese only** - SAP DM supports Brazilian Portuguese (pt-BR) but uses the `pt` suffix in file naming. European Portuguese is not separately supported.

### Chinese Language Support
**Two variants supported**:
- **Simplified Chinese** (`zh`) - Used in mainland China
- **Traditional Chinese** (`zh_TW`) - Used in Taiwan, Hong Kong, Macau

### Default Language
If no language-specific file exists, SAP DM falls back to:
1. `i18n.properties` (default bundle)
2. English (`i18n_en.properties`)

---

## File Naming Convention

### Standard Pattern
```
i18n_<ISO-639-1-code>.properties
```

### Examples
```
i18n_en.properties      # English
i18n_de.properties      # German
i18n_ja.properties      # Japanese
i18n_zh.properties      # Simplified Chinese
i18n_zh_TW.properties   # Traditional Chinese (note underscore!)
i18n_pt.properties      # Brazilian Portuguese
```

### Special Cases
- **Traditional Chinese**: Uses `zh_TW` (underscore, not hyphen)
- **Portuguese**: Uses `pt` (not `pt_BR` or `pt-BR`)
- **Serbian**: Uses `sr` suffix (SAP code is `SH` but file uses `sr`)

---

## Plugin i18n File Structure

When creating a POD 2.0 plugin with full language support, create all 28 files:

```
yourplugin/
└── i18n/
    ├── i18n.properties          # Default (English fallback)
    ├── i18n_bg.properties       # Bulgarian
    ├── i18n_zh.properties       # Simplified Chinese
    ├── i18n_zh_TW.properties    # Traditional Chinese
    ├── i18n_hr.properties       # Croatian
    ├── i18n_cs.properties       # Czech
    ├── i18n_da.properties       # Danish
    ├── i18n_nl.properties       # Dutch
    ├── i18n_en.properties       # English
    ├── i18n_fr.properties       # French
    ├── i18n_de.properties       # German
    ├── i18n_hu.properties       # Hungarian
    ├── i18n_it.properties       # Italian
    ├── i18n_ja.properties       # Japanese
    ├── i18n_ko.properties       # Korean
    ├── i18n_lt.properties       # Lithuanian
    ├── i18n_pl.properties       # Polish
    ├── i18n_pt.properties       # Portuguese (Brazilian)
    ├── i18n_ro.properties       # Romanian
    ├── i18n_ru.properties       # Russian
    ├── i18n_sr.properties       # Serbian (Latin)
    ├── i18n_sk.properties       # Slovak
    ├── i18n_sl.properties       # Slovenian
    ├── i18n_es.properties       # Spanish
    ├── i18n_sv.properties       # Swedish
    ├── i18n_th.properties       # Thai
    ├── i18n_tr.properties       # Turkish
    ├── i18n_uk.properties       # Ukrainian
    └── i18n_vi.properties       # Vietnamese
```

---

## i18n Loading Behavior

SAP DM uses SAPUI5's ResourceModel which follows this loading priority:

1. **Exact match**: `i18n_<user-locale>.properties` (e.g., `i18n_de_DE.properties`)
2. **Language match**: `i18n_<language>.properties` (e.g., `i18n_de.properties`)
3. **Default fallback**: `i18n.properties`

### Example - German User
User's browser locale: `de_DE` (German - Germany)

Loading sequence:
1. Try `i18n_de_DE.properties` (not typically created)
2. Try `i18n_de.properties` ✅ **Found - Use this**
3. Fallback to `i18n.properties` (if #2 not found)

### Example - Traditional Chinese User
User's browser locale: `zh_TW` (Chinese - Taiwan)

Loading sequence:
1. Try `i18n_zh_TW.properties` ✅ **Found - Use this**
2. Try `i18n_zh.properties` (if #1 not found)
3. Fallback to `i18n.properties` (if #1 and #2 not found)

---

## Best Practices

### 1. Always Create Default Bundle
**Always create `i18n.properties`** as the ultimate fallback:
```properties
# i18n.properties (English content as default)
button.text=Start
message.success=Operation completed successfully
```

### 2. Start with Core Languages
If not translating all 28 immediately, prioritize:
1. **English** (`en`) - Global standard
2. **German** (`de`) - SAP's home market
3. **Simplified Chinese** (`zh`) - Largest manufacturing base
4. **Japanese** (`ja`) - Major SAP DM market
5. **Spanish** (`es`) - Latin America manufacturing

### 3. Use Translation Keys, Not Hardcoded Text
```javascript
// ❌ WRONG - Hardcoded text
new Button({ text: "Start SFC" });

// ✅ CORRECT - i18n key
new Button({ text: this.getI18nText("button.startSfc") });
```

### 4. Keep Keys Consistent Across Languages
All language files must have **identical keys**:

**i18n_en.properties:**
```properties
button.start=Start
message.error=Error occurred
```

**i18n_de.properties:**
```properties
button.start=Starten
message.error=Fehler aufgetreten
```

**i18n_ja.properties:**
```properties
button.start=開始
message.error=エラーが発生しました
```

### 5. Handle Pluralization
Different languages have different plural rules. Use parameters:

```properties
# English
message.itemsSelected={0} item(s) selected

# German (same pattern works)
message.itemsSelected={0} Element(e) ausgewählt

# Japanese (no plural form)
message.itemsSelected={0}個のアイテムが選択されました
```

### 6. Right-to-Left (RTL) Languages
**SAP DM does not currently support RTL languages** (Arabic, Hebrew). The 28 supported languages are all LTR (left-to-right).

---

## Validation Checklist

Before deploying a plugin with i18n:

- [ ] All 28 language files created (or subset with clear documentation)
- [ ] `i18n.properties` (default) exists with English content
- [ ] All language files have identical keys
- [ ] No hardcoded UI text in JavaScript
- [ ] Text uses `this.getI18nText()` method calls
- [ ] Tested language switching in SAP DM user preferences
- [ ] Parameter placeholders (`{0}`, `{1}`) work in all languages

---

## Testing Language Switching

### In SAP DM:
1. Log in to SAP Digital Manufacturing
2. Navigate to User Menu → Settings
3. Change language preference
4. Refresh POD
5. Verify plugin text updates to selected language

### In POD Designer (Design Mode):
POD Designer typically uses the system language or English by default. Full language testing requires deploying to SAP DM runtime environment.

---

## Common Mistakes

### ❌ Mistake 1: Using Country Codes Instead of Language Codes
```
i18n_US.properties  # ❌ WRONG - US is country code
i18n_en.properties  # ✅ CORRECT - en is language code
```

### ❌ Mistake 2: Creating Region-Specific Variants
```
i18n_en_US.properties  # ❌ WRONG - Not needed, use en
i18n_en_GB.properties  # ❌ WRONG - Not needed, use en
i18n_es_MX.properties  # ❌ WRONG - Only one Spanish supported
```

Exception: Traditional Chinese (`zh_TW`) is the only region-specific variant supported.

### ❌ Mistake 3: Missing Default Bundle
```
i18n/
├── i18n_en.properties
├── i18n_de.properties
└── # ❌ MISSING: i18n.properties
```

Always create `i18n.properties` as fallback.

### ❌ Mistake 4: Inconsistent Keys Across Languages
```properties
# i18n_en.properties
button.submit=Submit

# i18n_de.properties
button.send=Senden  # ❌ WRONG - Key doesn't match!
```

Should be:
```properties
# i18n_de.properties
button.submit=Absenden  # ✅ CORRECT - Same key as English
```

---

## Resources

- **SAP Note 2722461**: Languages Supported in SAP Digital Manufacturing
- **SAP Help Portal**: [Internationalization (i18n)](https://help.sap.com/docs/sap-digital-manufacturing)
- **SAPUI5 Documentation**: [ResourceModel API](https://sapui5.hana.ondemand.com/sdk/#/api/sap.ui.model.resource.ResourceModel)

---

## Quick Reference - ISO Code Lookup

When you need to quickly find a language code:

| Need | ISO Code | File Suffix |
|------|----------|-------------|
| Bulgarian | bg | `_bg` |
| Chinese (Simplified) | zh | `_zh` |
| Chinese (Traditional) | zh-TW | `_zh_TW` |
| Croatian | hr | `_hr` |
| Czech | cs | `_cs` |
| Danish | da | `_da` |
| Dutch | nl | `_nl` |
| English | en | `_en` |
| French | fr | `_fr` |
| German | de | `_de` |
| Hungarian | hu | `_hu` |
| Italian | it | `_it` |
| Japanese | ja | `_ja` |
| Korean | ko | `_ko` |
| Lithuanian | lt | `_lt` |
| Polish | pl | `_pl` |
| Portuguese (Brazilian) | pt-BR | `_pt` |
| Romanian | ro | `_ro` |
| Russian | ru | `_ru` |
| Serbian (Latin) | sr | `_sr` |
| Slovak | sk | `_sk` |
| Slovenian | sl | `_sl` |
| Spanish | es | `_es` |
| Swedish | sv | `_sv` |
| Thai | th | `_th` |
| Turkish | tr | `_tr` |
| Ukrainian | uk | `_uk` |
| Vietnamese | vi | `_vi` |

---

**Last Updated**: 2026-05-17  
**Version**: 1.0.0
