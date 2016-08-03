# Application structure

The CLI initially puts all the main application files in the root of your `src` folder.

## Resources

You will also notice a sub-folder `resources` which contains _reusable_ resources for your app. These resources are globalized by the `main.ts` configuration using the `.feature('resources')` which simply loads and executes the `resources/index.ts` file.

By convention, the `index` file of a feature should be responsible for globalizing the resources it contains. Global resources are simply resources that can be used anywhere in your app. Very convenient!

```ts
export function configure(config: FrameworkConfiguration) { //config.globalResources([]);}
```

More on resources later ;\)

## 

