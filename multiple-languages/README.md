# Step-by-step

This is what I do to add multi-language support in my Angular projects.

1. Create a new angular project

```bash
> npm i -g @angular/cli
> ng new <project name>
```

2. Add localize package

```bash
> ng add @angular/localize
```

3. Configure new output path for `.xlf` files in your project

I like to do this as it helps me keep the project organized. This will output and on every new extract generate a file `messages.xlf` in `src/locale` directory

```json
"extract-i18n": {
    "builder": "@angular-devkit/build-angular:extract-i18n",
    "options": {
    "browserTarget": "multiple-languages:build",
    "outFile": "src/locale/messages.xlf"
    }
},
```

4. Next, add `i18n` attribute for translation in any `.html` file

I do this to have some example data to verify later on

In app.component.html

```html
<span i18n>Welcome</span>
```

5. Extract translations

```bash
> ng extract-i18n
```

6. Add a package called [xliffmerge](https://github.com/martinroob/ngx-i18nsupport/tree/master/projects/xliffmerge)

This package helps with creating new translation files and keeping them up-to-date each time you extract new translations

```bash
> npm i -D ngx-i18nsupport
```

7. Create a custom profile

This will tell the xliffmerge tool which languages you want extracted.

```bash
> touch src/locale/xliffmerge.json
```

Then add the configuration. This is the default I used, I created one additional language config for Swedish (`sv`)

```json
{
  "xliffmergeOptions": {
    "srcDir": "./src/locale",
    "genDir": "./src/locale",
    "i18nFile": "messages.xlf",
    "encoding": "UTF-8",
    "defaultLanguage": "en",
    "languages": ["sv"],
    "removeUnusedIds": false,
    "supportNgxTranslate": true,
    "useSourceAsTarget": false,
    "autotranslate": false,
    "verbose": false,
    "quiet": false
  }
}
```

8. Create a new npm command

Let's create a new npm command in `packages.json` to both extract translation and generate language specific files. Now you can run everything with just `npm run i18n`

```json
"scripts": {
    "ng": "ng",
    "i18n": "npm run ng -- extract-i18n && xliffmerge --profile src/locale/xliffmerge.json"
},
```

9. Configure `angular.json` to compile with correct language file

In `angular.json`, create a new `i18n` property above `architect` and add the `localize` property under `architect:build:options`

```json
"i18n": {
    "sourceLocale": "en-US",
    "locales": {
        "sv-SE": {
            "translation": "src/locale/messages.sv.xlf"
        }
    }
},
"architect": {
    "build": {
        "builder": "@angular-builders/custom-webpack:browser",
        "options": {
            "localize": true,
        }
    }
}
```

10. Translate and build

Send your language specific files to a linguist or translate them yourself. Once completed, run the `npm run i18n` command again to ensure line number and id's are correct, then proceed to build angular in production mode like normal.

```bash
> npm run i18n
> npm run build
```

This will result in two output directories under `/dist`, one fore each language. You can then configure in your hosting provider which language version a users should be redirected to.

- en-US
- sv-SE
