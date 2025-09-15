# Fix for Duplicate Translation Entries - Summary

## Problem
The `gulp-angular-translate-extractor` module was creating duplicate entries for HTML elements with both a `translate` attribute and text content:
```html
<uib-tab-heading translate="products.tab.BRANCH_PRICING">Branch Pricing</uib-tab-heading>
```

This created two entries instead of one:
- `"products.tab.BRANCH_PRICING": ""`
- `"Branch Pricing": ""`

## Solution
Modified the processing logic in `master/node_modules/gulp-angular-translate-extractor/index.js` to:

1. First process `HtmlDirectiveStandalone` patterns to collect translate attribute values
2. Skip `HtmlDirectiveStandalone` and `HtmlDirective` in the general processing loop
3. Special handling for `HtmlDirective` to only process elements WITHOUT translate attributes

## Key Changes
- Fixed regex escaping issues throughout the file
- Added new logic to prevent duplicate processing
- Modified the processing loop to exclude problematic patterns
- Added special handling for HtmlDirective patterns

## Files
- Patch: `docs/fix-duplicate-translations.patch`
- Documentation: `docs/fix-duplicate-translations.md`

## Verification
Tested with minimal HTML file - now correctly creates only one entry per element.