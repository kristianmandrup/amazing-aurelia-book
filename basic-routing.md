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
      { route: '',          moduleId: 'home',     name: 'home',    nav: true,   title: 'Home' },
      { route: 'contacts',  moduleId: 'contacts', name:'contacts', nav: true,   title: 'Contacts' }
    ]);

    this.router = router;
  }
}
```

The App vm will automatically be injected with a `Router` and `RouterConfiguration` singletons when the application starts.

We add a `configureRouter(config: RouterConfiguration, router: Router){` method to our root view model, which is called as the vm is initialized.
We then set the `title` of the route config and set the navigation map via `map`. Each route is an object with various required and optional settings.
For a route to work it must have a:

- `route` the routing pattern
- `moduleId` the location of the view model module
- `name` the name (logical identifier) of the route

The `moduleId` must link to another view model in your app, relative to the location of this router. The other view model can add its own router configuration, to create a nested routing hierarchy.

Now let's do some routing. First create a new contacts VM/V pair.

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

Let's first try to add a link to the `app.html` page

```html
<template>
  <h1>Hello World</h1>
  <a href="/contacts">go to contacts</a>
<template>
```

Obviously this won't work. 

We should rarely use static routes like this however. Much better to let the Aurelia router take care of creating the routes for us.

## route-href attribute

The `<a>` anchor tag supports a special attribute `route-href` which works with the current router. It can be used to automatically generate and update the HTML `href` attribute of the link and also handles finding and calling the route in question with the href. 

```html
<template>
 <h1>Hello World</h1>
 <a route-href="contacts">go to contacts</a>
<template>
```

If you want to pass data to the route you have to use a more advanced variant which we will look at later

You will notice that by default the full screen is swapped with the view of the route being routed to. In most real apps however, you want to have a basic single page layout and then swap out one or more regions of the page when you route to a new page, while keeping the surrounding layout in place.

For this we need, Aurelia provides us with a `<router-view>` element which we will now look into.

## router-view element

The special `<router-view>` element acts as a placeholder for view content of routes being routed to. By convention if you only have a single nameless `<router-view>` it will be filled in by the first view of the route being routed to.

If your route routes to multiple named views, you can have multiple named `<router-view>` elements on your layout, designating a destination for each router view.

### Multiple router views

Let's create a layout with two named router views, one for the `main` content and one for the `sidebar`.

```html
<template>
  <section id="main" class="container main">
    <router-view name="main" />
  </section>
  <section id="sidebar" class="container sidebar">
    <router-view name="sidebar" />
  </section>
<template>
```







