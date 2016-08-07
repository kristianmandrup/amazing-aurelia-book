# Smart resource management

To avoid having to require the value converter each time, we can add it as a global resource in `resources/index.ts`

```ts
export function configure(config: FrameworkConfiguration) {
  config.globalResources([
    // ...
    './value-converters/date-day-name'
  ]);
}
```

Alternatively, add an `index.ts` file in `value-converters` folder to re-export an array of value converters which you then add to the array.

`index.ts`

```ts
export default [
  'date-day-name'
]
```

```ts
import converters from './value-converters';
import elements from './elements';
// ...

const resources = {
  converters: converters.map(name => './value-converter/' + name),
  elements: elements.map(name => './elements/' + name)
  // ...
}

Using [Array.reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) we can aggregate all the resources

const allResources = Object.values(resources)).reduce((all, res) => all.concat(res), [])

// Then make all resources global :)
export function configure(config: FrameworkConfiguration) {
  config.globalResources(allResources);
}
```

