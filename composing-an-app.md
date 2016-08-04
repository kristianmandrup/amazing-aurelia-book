# Basic composition

In order to structure our app can start simple by composing it from several smaller elements, ie. pages or components. To start simple we can use the `<compose>` element to show how you can compose views from smaller components and elements. 

By `element` we mean a a custom element that can be reused across many views. By component we mean a vm/v pair which is not necessarily registered as a custom element.

Create a new view model and view pair \(vm\/v\) called `world`.

View model: `world.ts`

```ts
export class World {}
```

View: `world.html`

```html
<template> 
  <h2>world</h2>
</template>
`

Now compose the main `app.html` page from this new world element by using `<compose>` element to link to world via the view-model attribute set to world.

```html
<template>
  <h1>hello</h1>
  <compose view-model="world" />
</template>
```

The Aurelia convention, is to load a view model and then determine the view to use to display the model, by default a view at the same location with the same name. We will later see how we can override and change this convention if needed, such as dynamically deciding the view depending on some data or logic.

Now expand on this concept by composing twice with the same element. This will create two instances of the vm: `world` each with its own view instance as well.

```html
<template>
  <h1>hello</h1>
  <compose view-model="world" />
  <compose view-model="world" />
</template>
```

Now experiment by creating a new vm\/v called `bye` and compose it as well. You can even try having the world element being composed of bye in a nested composition.

Now cleap up the `world` and `bye` VMs if you haven't already, before we move on.

