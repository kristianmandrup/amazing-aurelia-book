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

We will go through each top level folder to get an idea of the main project concepts involved ;)

## aurelia_project

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
`process-markup` processes your markup (templates)

`transpile` transpiles your ES6 or typescript to ES5 javascript.








