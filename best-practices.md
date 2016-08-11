# Best practices

## Iterating objects

[iterating-objects-using-repeat-for-in-aurelia](http://ilikekillnerds.com/2015/08/iterating-objects-using-repeat-for-in-aurelia)

Use value converters such as `objectValues` or `objectKeys`:

```html
<li repeat.for="prop of myObj | objectKeys">${prop}</li>
<li repeat.for="val of myObj | objectValues">${val}></li>
```

