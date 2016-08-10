# Binding models to RethinkDB

[aurelia-rethink-bindtable](https://github.com/kristianmandrup/aurelia-rethink-bindtable) is a library to bind to RethinkDB tables via a socket connection.

### Installation

`npm i aurelia-rethink-bindtable --save`

### Usage

Using rethinkDB bindtable is super simple. You simple include the `@bindable` decorator and use it on a VM to add a `bindable` which binds to a rethinkDB table via a socket connection to a server.

You pass `@bindable` the name of the table, such as `questions` and optionally the server host (default: `localhost`).

```js
import {bindable} from 'aurelia-rethink-bindtable';

@bindable('questions', 'www.mydomain.com')
export class Questions {
  constructor(bindable, filters) {
    super({logging: true});
    this.bindable = bindable;
  }

  selectRow(row) {
    this.selectedRow = row;
  }

  deleteSelected() {
    this.table.delete(this.selectedRow);
  }
}
```

The `bindable` decorator also creates two getter methods `rows` and `table` which delegate to the bindable properties of the same name. 
You can use `rows` with `repeat.for` to dynamically display the row data of the table.

```html
<template>
    <ul repeat.for="row of rows">
      <li click.bind="selectRow(row)">${row.id}</li>
      <li click.bind="selectRow(row)">${row.name}</li>
    </ul>
    <button click.bind="deleteSelected()">Delete</button>
</template>
```

### Adding UI virtualization

[aurelia-ui-virtualization](https://github.com/aurelia/ui-virtualization) is a plugin which provides infinite list display using advanced UI virtualization techniques to reuse the same DOM elements for display of huge lists of data to ensure fast frame rates and minimal DOM operations.

Look at this [demo](http://aurelia.io/ui-virtualization/) to get an impression of its power!

### Plugin installation & config

We install via npm

`npm i aurelia-ui-virtualization --save`

Then we configure the plugin

```js
export function configure(aurelia) {
  aurelia.use
    // ...
    .plugin('aurelia-ui-virtualization');
```

Then we simply replace the `repeat.for` with `virtual-repeat.for` :)

`<ul virtual-repeat.for="row of rows">`

Awesome!!



