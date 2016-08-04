# Basic routing

Composition has its uses but also its limitations. For most real apps we want to use routing as a way to navigate our application through multiple pages and views.

Change the `app.ts` to the following:

```ts
import {Router, RouterConfiguration} from 'aurelia-router';

export class App {
  router: Router;

  constructor() {}

  configureRouter(config: RouterConfiguration, router: Router){
    config.title = 'Contacts';
    config.map([
      { route: ['', 'home'],          moduleId: 'home',     name: 'home',    nav: true,   title: 'Home' },
      { route: 'contacts',  moduleId: 'contacts', name:'contacts', nav: true,   title: 'Contacts' }
    ]);

    this.router = router;
  }
}
```

The `App` VM will automatically be injected with a `Router` and `RouterConfiguration` singletons when the application starts.

We add a `configureRouter(config: RouterConfiguration, router: Router){` method to our root view model, which is called as the VM is initialized.

We then set the `title` of the route config and set the navigation map via `map`. Each route is an object with various required and optional settings.

For a route to work it must have a:

- `route` the routing pattern(s)
- `moduleId` the location of the view model module
- `name` the name (logical identifier) of the route

The route is a pattern which is matched by the routing engine in order to determine which route to activate. A route can have multiple patterns such as the `home` route in the example above, which matches both the root pattern `''` (essentially `/` empty) and `welcome` (ie. `/welcome`.

The `moduleId` must link to a view model in your app. The moduleId is calculated relative to the app root, typically `/src`. 
The view model activated can add its own router config to the (parent) router config to create a "nested" routing hierarchy.

However please not that by default, nested routes are still essentially "flat". To make a truly nested router where routes extend a root base, we must currently do some heavy lifting which we will explore later on.

Now let's do some routing. First create a new `contacts` VM/V pair.

`contacts.ts`

```ts
export class Contacts {
}
```

`contacts.html`

```html
<template>
  <h1>My Contacts</h1>
</template>
```

Let's now try to add a link to the `app.html` page

```html
<template>
  <h1>Hello World</h1>
  <a href="/contacts">go to contacts</a>
<template>
```

Obviously this won't work! The browser only knows to use the href to retrieve a `/contacts.html` file and render it as a new page.

We need Aurelia routing mechanics to handle routes for us. The magical `route-href` attribute to the rescue!

## route-href attribute

Aurelia comes with a special attribute `route-href` which works with the current router. It can be used to automatically generate and update the HTML `href` attribute of the link. It also handles finding and calling the route in question with the `href`. 

```html
<template>
 <h1>Hello World</h1>
 <a route-href="contacts">go to contacts</a>
<template>
```

If you want to pass data to the route you have to use a more advanced variant with `params` which we will look at later when we go more in depth. For now let's keep it simple!

You will notice that by default the full screen is swapped with the view of the route being routed to. In most real apps however, you want to have a basic single page layout and then swap out one or more regions of the page when you route to a new page, while keeping the surrounding layout in place.

For this we need, Aurelia provides us with a `<router-view>` element which we will now look into.

## router-view element

The special `<router-view>` element acts as a placeholder for view content of routes being routed to. By convention if you only have a single nameless `<router-view>` it will be filled in by the first view of the route being routed to.

If your route routes to multiple named views, you can have multiple named `<router-view>` elements on your layout, designating a destination for each router view.

### Multiple router views

Let's create a layout with two named router views, one for the `main-content` content and one for the `sidebar`.

```html
<template>
  <section id="main" class="container main page-host">
    <router-view name="main-content" />
  </section>
  <section id="sidebar" class="container sidebar page-host">
    <router-view name="sidebar" />
  </section>
<template>
```

Now create `articles` and `stocks` dummy VM/V pairs.

`articles.ts`

```ts
export class Articles {
}
```

`articles.html`

```html
<template>
  <h1>Articles</h1>
</template>
```

Stock quotes

`stocks.ts`


```ts
export class Stocks {
}
```

`stocks.html`

```html
<template>
 <h1>Stocks</h1>
</template>
```

Now let's update our router using the `viewPorts` key to define how to route to our two named viewports.

```ts
 config.map([
 ...
 { 
  route: 'stocks', 
  viewPorts: {
    main: {moduleId: './articles'}, 
    sidebar: {moduleId: './stocks'} 
  }, 
  name:'stocks', 
  nav: true, 
  title: 'Stocks' 
  }
 ]);
```

## Layouts

You can add a layout to a router view vie the special `layout` attribute as follows:

```
<template>
  <div class="page-host">
    <router-view layout="views/layout-default.html"></router-view>
  </div>
</template>
```














