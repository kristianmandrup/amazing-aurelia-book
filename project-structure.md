# Project structure

Your project structure should look as follows:

```bash
/aurelia_project
/custom_typings
/node_modules
/scripts
/src
/typings
```

We will go through each top level folder to get an idea of the main project concepts involved ;\)

## aurelia\_project

The folder `aurelia_project` contains the environment configuration, generators available and tasks.

```bash
/aurelia_project
  /environments
    dev.ts
    prod.ts
    stage.ts
  /generators
    attribute.json
    attribute.ts
    ...
  /tasks
    build.json
    build.ts
    ...
  aurelia.json
```

### Environments

The CLI by default configures your project with environment settings for `dev`, `stage` and `prod`.

Simply use the `-env` flag on any command to specify under which environment to run under. For example: `au run prod --watch`

### Generators

Generators can be run with `au generate <resource>` such as `au generate element`

You can add your own resource generator as we will see later.

### Tasks

Tasks are commands that can be run directly by the CLI: 
The project comes with a few built-in core tasks.

`build` builds your entire project, useful to build a distribution for `stage` or `prod`.

`run` builds your project and runs it.

`test` runs your unit tests

`process-css` processes your styles and converts them to css.
`process-markup` processes your markup \(templates\)

`transpile` transpiles your ES6 or typescript to ES5 javascript.

### aurelia.json

This is the core project configuration file.

## custom\_typings

This folder only contains the typings file for aurelia protractor, a port of Angular protractor for use with Aurelia, used for End to End \(E2E\) testing.

## node\_modules

All your dependency modules, such as core aurelia modules and 3rd party + vendor libaries.

## scripts

A few small scripts like `app-bundle` and `vendor-bundle` which processes your dependency definitions in the `aurelia.json` config file.

## src

The source files for your app.

### Application environment settings

`environment.ts` contains basic environment settings such as wheter to run in debug mode and if you want to run tests as well.

```js
export default { debug: true, testing: true};
```

These settings are by default used in `main.ts` which configures your application. The environment settings are used to determine if certain plugins should be used or not.

```js
 if (environment.debug) { aurelia.use.developmentLogging(); }

 if (environment.testing) { aurelia.use.plugin('aurelia-testing'); }
```

You can add your own environment settings, such as if you want authentication or authorization to be turned on etc.

### Application files

The entry point for your app is by default `main.ts` which is linked to from the `index.html` file in the root of the project.

```html
<body aurelia-app="main">
  <script src="scripts/vendor-bundle.js" data-main="aurelia-bootstrapper">
  </script>
</body>
```

The `main.ts` configures your Aurelia app and bootstraps it by calling start. When the app is ready (a promise) `then` it sets the root of the app to an element on the document, by default the body. Also by default it finds the `app.ts` file in `src` and uses it as the view model for the root element.

```ts
export function configure(aurelia: Aurelia) {
  aurelia.use
    .standardConfiguration()
    .feature('resources');

  aurelia.start().then(() => aurelia.setRoot());
}
```

`app.ts` is thus a regular view model, and can be as simple as:

```ts
export class App { }
```

```html
<template>
  Hello World
</template>
```

There you have it :)

## Typings

The folder `typings` is used by the TypeScript `typings` installer to install type definitions for 3rd party libaries written in Javascript.

The preferred way to install typings is in `typings/globals`

```bash
/typings
  /globals
    angular-protractor/
      index.d.ts
      typings.json
  index.d.ts
```

For example, to install `sortable.js` typings:

`typings install dt~sortablejs --global`

## .editorconfig

The editor config can be used to configure conventions for your editor of choice. Most modern editors support this!

Typically you might want to adjust indentation preferences

```
# 2 space indentation[**.*]indent_style = spaceindent_size = 2
```

## index.html

index.html is the starting point of your app. 

```
 <body aurelia-app="main"> <script src="scripts/vendor-bundle.js" data-main="aurelia-bootstrapper"></script> </body>
```

The `aurelia-app="main"` attribute, tells Aurelia where to mount an aurelia app and which file to kick off the bootstrapping of that app.

You can mount multiple apps on your `index.html` page if you like.

```
 <body>
  <section aurelia-app="admin" />
  <section aurelia-app="main" /> <script src="scripts/vendor-bundle.js" data-main="aurelia-bootstrapper"></script> </body>
```

Here we mount two apps `main` and `admin` each in their own section of the page. We will go more in depth on this approach later.

## package.json

Your node package manager (`npm`) config file. Here you define project information for npm such as `name`, `version` and `description` of your project along with development and runtime dependencies.

## tsconfig.json

Contains TypeScrupt compiler configurations.
Key for Aurelia is `"experimentalDecorators":  true` which enables decorators in TypeScript. Aurelia relies heavily on decorators to minimize boiler plate cruft code in your code base. 

## tsline.json

Contains linting configurations for your TypeScript compiler.
Linting is the art of checking that your code follows certain (strict) conventions so it stays clean.

## typings.json

Registry file used by `typings` binary to manage the type definitions installed.


 





