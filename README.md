# gulp-angular-translate-extractor
Gulp plugin extracts the translation keys for angular-translate.

## Install
via npm  
`npm install gulp-angular-translate-extractor`

## Usage  
Example:
```
var extractTranslate = require('gulp-angular-translate-extractor');

gulp.task('taskName', function () {
  var i18nsrc = ['./index.html', './app.js'];  // your source files  
  var i18ndest = './src/assets/translations'; //destination directory
  return gulp.src(i18nsrc)
      .pipe(extractTranslate({
        defaultLang: 'en-US',         // default language
          lang: ['en-US', 'ru-RU'],   // array of languages
          dest: i18ndest,             // destination, default '.'
          prefix: 'prefix_',          // output filename prefix, default ''
          suffix: '.suffix',          // output filename suffix, default '.json'
          safeMode: false,            // do not delete old translations, true - contrariwise, default false
          stringifyOptions: true,     // force json to be sorted, false - contrariwise, default false
      }))
      .pipe(gulp.dest(i18ndest));
});
```
This task will parse your src files, extract all the translation keys and creates two files 'en-US.json' and 'ru-RU.json' in dest directory. For the default lang ('en_US.json') file will be formatted like:
```
{
    "1st Translation": "1st Translation",
    "2nd Translation": "2nd Translation",
    "3rd Translation": "3rd Translation",
    ...
}
```
For the non-default lang ('ru_RU.json') file will be formatted like:
```
{
    "1st Translation": "",
    "2nd Translation": "",
    "3rd Translation": "",
    ...
}
```

## Fixed Issues

### Duplicate Translation Entries (v1.0.3+)
Fixed an issue where HTML elements with both a `translate` attribute and text content were creating duplicate entries in translation JSON files.

**Before fix:**
```html
<uib-tab-heading translate="products.tab.BRANCH_PRICING">Branch Pricing</uib-tab-heading>
```
Created two entries:
1. `"products.tab.BRANCH_PRICING": ""`
2. `"Branch Pricing": ""`

**After fix:**
Only one entry is created:
1. `"products.tab.BRANCH_PRICING": ""`

The fix modifies the processing logic to:
1. First process `HtmlDirectiveStandalone` patterns to collect translate attribute values
2. Skip `HtmlDirectiveStandalone` and `HtmlDirective` in the general processing loop
3. Special handling for `HtmlDirective` to only process elements that DON'T have translate attributes

This prevents duplicate entries while preserving all existing functionality.

## Supported Patterns

The plugin supports various patterns for extracting translation keys:

1. HTML filter: `{{ 'TRANSLATION_KEY' | translate }}`
2. HTML directive: `<translate>TRANSLATION_KEY</translate>`
3. HTML directive with translate attribute: `<any translate="TRANSLATION_KEY"></any>`
4. JavaScript service: `('TRANSLATION_KEY')`
5. JavaScript filter: `('translate')('TRANSLATION_KEY')`

## Options

- `defaultLang`: Default language for translation files
- `lang`: Array of languages to generate files for
- `dest`: Destination directory for generated files
- `prefix`: Output filename prefix
- `suffix`: Output filename suffix
- `safeMode`: Do not delete old translations
- `stringifyOptions`: Force JSON to be sorted
