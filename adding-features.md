# Adding features

Aurelia supports the notin of a _features_ which is a folder containing reusable global elements.


```js
configure(config)
aurelia.use
  // ...
  .feature('resources')
```

```bash
/resources
  /elements
    date-selector.ts
    date-selector.html
    ...
index.ts
```

`index.ts`

```ts
export function configure(config) {
  config.globalResources([
  './elements/date-selector', 
  // ...
  ]);
}
```

To avoid having to prefix each element with `./elements` you could add an `index.js` file to the elements folder which exports a list of all the elements.

TODO: Show this



