# I18n and localization


```js
  .plugin('aurelia-i18n', (instance) => {
    // register backend plugin
    instance.i18next.use(Backend);

    // adapt options to your needs (see http://i18next.com/docs/options/)
    // make sure to return the promise of the setup method, in order to guarantee proper loading
    return instance.setup({
      backend: {                                  // <-- configure backend settings
        loadPath: './locales/{{lng}}/{{ns}}.json', // <-- XHR settings for where to get the files from
      },
      lng : 'de',
      attributes : ['t','i18n'],
      fallbackLng : 'en',
      debug : false
    });
  });
```

You may also group your translations by namespaces, spread across multiple files. Say you have the standard translation.json and an additional nav.json for the navigation items, you can configure aurelia-i18n by passing the ns setting in the config object containing the different namespaces as well as the default namespace.

```js
instance.setup({
  ...
  ns: {
    namespaces: ['translation','nav'],
    defaultNs: 'translation'
  }
});
```

## Typescript

`typings install dt~i18next --global`

use the typings file from this repositories doc folder `doc/i18next-xhr-backend.d.ts` 

Copy `*.d.ts` files from `doc/*.d.ts` to `/custom_typings`

The next step is to let the compiler know about your `*.d.ts` files. 
Add the following section to your `tsconfig.json` file.

```json
  "filesGlob": [
    "./typings/browser.d.ts",
    "./custom_typings/**/*.d.ts"
  ],
```

## Safari support

It comes from the fact that Safari doesn't support the internationalization (`window.Intl`) API.

In order to solve it, manually include the `Intl.js` polyfill. Since it is already pulled as a dependency, adapt the bootstrap function of Aurelia with the setup code for Webpack of `Intl.js`:

```js
bootstrap(aurelia => {
     if (!global.Intl) {
         console.log('Intl not present')
         require.ensure([
             'intl',
             'intl/locale-data/jsonp/en.js'
         ], function (require) {
             require('intl');
             require('intl/locale-data/jsonp/en.js');
             boot(aurelia);
         });
     } else {
         boot(aurelia);
     }
 });


 function boot(aurelia) {
     aurelia.use
         .standardConfiguration()
         .developmentLogging()
         .plugin('aurelia-i18n', (instance) => {
             let language = navigator.language.split('-')[0];

             // register backend plugin
             instance.i18next.use(XHR);

             function loadLocales(url, options, callback, data) {
                 switch (url) {
                     case 'en':
                         callback(enTranslations, { status: '200' });
                         break;
                     case 'fr':
                         callback(frTranslations, { status: '200' });
                         break;
                     default:
                         callback(null, { status: '404' });
                         break;
                 }
             }

             instance.setup({
                 backend: {
                     loadPath: '{{lng}}',
                     parse: (data) => data,
                     ajax: loadLocales,
                 },
                 lng: language,
                 attributes: ['t', 'i18n'],
                 fallbackLng: 'en',
                 debug: false,
             });
         });

     aurelia.start().then(() => aurelia.setRoot('app', document.body));
 }
 ```