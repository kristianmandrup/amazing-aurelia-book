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

hello

