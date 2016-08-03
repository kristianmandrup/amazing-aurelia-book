# Composing an app

In order to structure our app we need to compose it from several smaller elements, ie. pages or components. To start simple we can use the `<compose>`element to show how you can compose elements.

Create a new view model and view pair (vm/v) called world.

```ts
export class World {}
```

```html
<template> <h2>world</h2> </template>
```

Now compose the main app.html page from this new world element by using `<compose>` element to link to world via the view-model attribute set to world.

```html
<template>
  <h1>hello</h1>
  <compose class="col-md-2 non-printable" view-model="world" /> 
</template>
```

The Aurelia convention, is to load a view model and then determine the view to use to display the model, by default a view at the same location with the same name.We will later see how we can override and change this convention if needed, such as dynamically deciding the view depending on some data or logic.

Now expand on this concept by composing twice with the same element.

```html
<template>
  <h1>hello</h1>
  <compose class="col-md-2 non-printable" view-model="world" />
  <compose class="col-md-2 non-printable" view-model="world" />
</template>
```

Now experiment by creating a new vm/v called bye and compose it as well. You can even try having the world element being composed of bye in a nested composition.

