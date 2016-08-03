# Application structure

The CLI initially puts all the main application files in the root of your `src` folder. 

## Resources

You will also notice a sub-folder `resources` which contains *reusable* resources for your app. These resources are globalized by the `main.ts` configuration using the `.feature('resources')` which simply loads and executes the `resources/index.ts` file.

By convention, the `index` file of a feature should be responsible for globalizing the resources it contains. Global resources are simply resources that can be used anywhere in your app. Very convenient!

```ts
export function configure(config: FrameworkConfiguration) { //config.globalResources([]);}
```

More on resources later ;)

## Composing an app

In order to structure our app we need to compose it from several smaller elements, ie. pages or components. To start simple we can use the `<compose>` element to show how you can compose elements.

Create a new view model and view pair (*vm/v*) called `world`.

```ts
export class World {}
```

```ts
<template>
  <h2>world</h2>
</template>
```

Now compose the main `app.html` page from this new `world` element by using `<compose>` element to link to world via the `view-model` attribute set to `world`. 

```html
<template>
  <h1>hello</h1>
  <compose class="col-md-2 non-printable" view-model="world" />
</template>
```

The Aurelia convention, is to load a view model and then determine the view to use to display the model, by default a view at the same location with the same name.We will later see how we can override and change this convention if needed, such as dynamically deciding the view depending on some data or logic.

Now expand on this concept by 

```html
<template>
  <h1>hello</h1>
  <compose class="col-md-2 non-printable" view-model="world" />
</template>
```














