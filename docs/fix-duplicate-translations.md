# Fix for Duplicate Translation Entries in gulp-angular-translate-extractor

## Overview
This patch fixes an issue in the `gulp-angular-translate-extractor` module where HTML elements with both a `translate` attribute and text content were creating duplicate translation entries in JSON output files.

## Problem Description
When processing HTML elements like:
```html
<uib-tab-heading translate=\"products.tab.BRANCH_PRICING\">Branch Pricing</uib-tab-heading>
```

The module was creating **two** entries in the translation JSON files:
1. `\"products.tab.BRANCH_PRICING\": \"\"`
2. `\"Branch Pricing\": \"\"`

However, it should only create **one** entry:
1. `\"products.tab.BRANCH_PRICING\": \"\"`

## Root Cause
The issue occurred because two different regex patterns in the extractor both matched the same element:

1. **HtmlDirectiveStandalone pattern**: `translate=\"((?:\\\\.|[^\"\\\\])*)\"`
   - Correctly matches the `translate` attribute value: `products.tab.BRANCH_PRICING`

2. **HtmlDirective pattern**: `<[^>]*translate[^{>]*>([^<]*)</[^>]*>`
   - Incorrectly matches the element's text content: `Branch Pricing`

Both patterns processed their matches independently and added separate entries to the translation results.

## Solution
The fix modifies the processing logic in `index.js` to:

1. **First**, process all `HtmlDirectiveStandalone` patterns to collect translate attribute values
2. **Then**, process all other patterns except `HtmlDirectiveStandalone` and `HtmlDirective`
3. **Finally**, process `HtmlDirective` patterns with special logic to skip elements that have translate attributes

This prevents duplicate entries by ensuring only the translate attribute value is used as the translation key.

## Detailed Changes

### File: master/node_modules/gulp-angular-translate-extractor/index.js

#### 1. Fixed regex escaping in `_extractTranslation` function
- Line 53: Changed `translationKey.replace(/\\'/g, \"'\")` to `translationKey.replace(/\\\\'/g, \"'\")`
- Line 57: Changed `translationKey.replace(/\\\"/g, '\"')` to `translationKey.replace(/\\\\\"/g, '\"')`

#### 2. Fixed regex patterns in the `regexs` object
- Line 136: Fixed quote escaping in `commentDoubleQuote` pattern
- Line 140: Fixed quote escaping in `HtmlFilterDoubleQuote` pattern
- Line 142-145: Fixed quote escaping in `HtmlDirective`, `HtmlDirectiveStandalone`, `HtmlDirectivePluralLast`, `HtmlDirectivePluralFirst`, and `HtmlNgBindHtml` patterns
- Line 147: Fixed quote escaping in `JavascriptServiceDoubleQuote` pattern
- Line 149: Fixed quote escaping in `JavascriptServiceInstantDoubleQuote` pattern
- Line 151: Fixed quote escaping in `JavascriptFilterDoubleQuote` pattern

#### 3. Modified processing logic in the `extract` function
- Lines 169-181: Added new logic to first process all `HtmlDirectiveStandalone` patterns and collect translate attribute values
- Lines 185-188: Modified the loop to skip both `HtmlDirectiveStandalone` and `HtmlDirective` patterns
- Lines 212-227: Added special handling for `HtmlDirective` patterns to only process elements that DON'T have translate attributes

## Verification
The fix was tested with a minimal HTML file containing:
```html
<div translate=\"test.BRANCH_PRICING\">Branch Pricing</div>
```

Before the fix, this created two entries:
1. `\"test.BRANCH_PRICING\": \"test.BRANCH_PRICING\"`
2. `\"Branch Pricing\": \"Branch Pricing\"`

After the fix, only one entry is created:
1. `\"test.BRANCH_PRICING\": \"test.BRANCH_PRICING\"`

Elements without translate attributes still correctly create entries for their text content.

## Files Modified
- `master/node_modules/gulp-angular-translate-extractor/index.js`

## How to Apply
1. Apply the patch file: `patch -p1 < fix-duplicate-translations.patch`
2. Or manually replace the content of `master/node_modules/gulp-angular-translate-extractor/index.js` with the fixed version

## Testing
To test the fix:
1. Create an HTML file with elements that have both translate attributes and text content
2. Run `gulp generate-translate`
3. Check the generated JSON translation files to ensure only one entry is created per element

## Notes
- The fix preserves existing functionality for elements without translate attributes
- The fix works with all supported translation patterns in the module
- No changes are needed to the gulp configuration or other parts of the application