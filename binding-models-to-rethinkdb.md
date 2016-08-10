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





