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

However please not that by default, nested routes are evaluate relative to the parent router. To make a truly nested router where routes extend a root base, we must currently do some "heavy lifting", which we will explore later on...

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
<template bindable="router">
 <h1>Hello World</h1>
 <a route-href="route: contacts">go to contacts</a>
<template>
```

The `route: contacts` tells the router to go to the route named `contacts`.

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
  quotes = [{
    id: AAPL,
    price: 105,
  },
  {
    id: MSFT,
    price: 56,
  },
  {
    id: GOOGL,
    price: 784,
  }];
}
```

`stocks.html`

Here we use the special `repeat.for` attribute provided by Aurelia, to iterate over the quotes and display the quote info using string interpolation via the well known `${}` syntax.

```html
<template>
  <h1>Stocks</h1>
  <ul>
    <li repeat.for="quote of quotes">
      <span class="stock">${quote.id} ${quote.price}</span>
    </li>
  </ul>
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

```html
<template>
  <div class="page-host">
    <router-view layout="./layouts/default.html"></router-view>
  </div>
</template>
```

Now let's create our `layouts/default` layout view.

```html
<template>
  <div class="left-content">
    <slot name="aside-content"></slot>
  </div>
  <div class="right-content">
    <slot name="main-content"></slot>
  </div>
</template>
```

Here we use the special `slot` element, which is part of the [Shadow DOM v1](https://developers.google.com/web/fundamentals/primers/shadowdom) as described in this [Aurelia blog post](http://blog.durandal.io/2016/05/23/aurelia-shadow-dom-v1-slots-prerelease/)

TODO: more on using slots and layout...

## Router navigation

To display the navigation links to each route, we can iterate `router.navigation`. Then for each navigation item (route) we can check if it is active or not via `isActive` and display it accordingly.

```html
  <li repeat.for="row of router.navigation" class="${row.isActive ? 'active' : ''}">
    <a href.bind="row.href">${row.title}</a>
  </li>
```

We can use this to generate a full navigation menu, here using bootstrap.

```html
<template>
    <nav class="navbar navbar-default navbar-static-top">
      <div class="container">
        <div id="navbar" class="navbar-collapse collapse">
          <ul class="nav navbar-nav">
            <li repeat.for="row of router.navigation" class="${row.isActive ? 'active' : ''}">
              <a href.bind="row.href">${row.title}</a>
            </li>
          </ul>
          <ul class="nav navbar-nav navbar-right">
            <li>
              <a href="#">log out</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>
  <div class="container">
    <div class="row">
      <router-view class="col-md-8"></router-view>
    </div>
  </div>
</template>
```

Notice the `href.bind="row.href"` on the anchor inside the `repeat.for`. Here we directly bind the normal `<a>` anchor `href` DOM attribute with the `row.href` from the navigation map of the router.

## Parameterized routing

A parameterized route is one which has one or more parameter placeholder in its routing pattern in the form `:<parameter name>`, such as `route: 'contacts/:id'` in this routing config example.

```ts
    config.map([
      { route: ['', 'home'],          moduleId: 'home',     name: 'home',    nav: true,   title: 'Home' },
      { route: 'contacts',  moduleId: 'contacts', name:'contacts', nav: true,   title: 'Contacts' },
      { route: 'contacts/:id',  moduleId: 'contact', name:'contact', nav: true,   title: 'Contact' }
    ]);
```

If you run this, an error message will appear, saying:

```bash
Error: Invalid route config for "/contact/:id" : dynamic routes must specify an "href:" to be included in the navigation model.
```

You need to supply a `href` for the navigation model used for displaying hrefs. So to fix it, we can add it as shown here:

```js
 { 
   route:    'contacts/:id', 
   moduleId: 'contact', 
   name:     'contact', 
   href:     'contact',
   nav: true,
   title:    'Contact' 
}
```

Now it should work! 

It feels a bit redundant to specify the name `contact` for both `name`, `href` and `moduleId`. We will later explore how we can encode such patterns in the router or route configuation setup using different stategies.

## Using the parameterized route

Now use the parameterized route as follows:

`<a route-href="route: contact; params.bind: {id: 1}">Contact #1</a>`

Here we bind the object `{id: 1}` to params of the route named `contact`.

Personally I cringe a bit on this syntax. I would much have preferred to split params into its own special attribute like this:

`<a route-href="route: contact" route-params.bind="{id: 1}">Contact #1</a>`


## Entity routing best practices

We encourage you in most cases to only have two main routes for any entity, one to handle the list case and one for the single item case.

Then in the view use binding, composition etc. to handle the display of the entity or a form to create or update it.

The `contact-list` view, from [Contact Manager](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/contact-manager-tutorial/6) example.

```html
<template>
  <div class="contact-list">
    <ul class="list-group">
      <li repeat.for="contact of contacts" class="list-group-item ${contact.id === $parent.selectedId ? 'active' : ''}">
        <a route-href="route: contacts; params.bind: {id:contact.id}" click.delegate="$parent.select(contact)">
          <h4 class="list-group-item-heading">${contact.firstName} ${contact.lastName}</h4>
          <p class="list-group-item-text">${contact.email}</p>
        </a>
      </li>
    </ul>
  </div>
</template>
```

Notice that we use the form `route: contact; params.bind: {id:contact.id}` for the `route-href` attribute. the `params.bind` tells the router which parameters to bind to the route.

## Child routers

In your parent router config, such as in `app.ts` you simply link to a view model that is itself a router.

`{ route: 'contacts',  moduleId: './contacts/index', nav: true, title:'Contacts' }`

Then for the child route you inject the router and ...

```ts
import {Router} from 'aurelia-router';

@inject(Router)
export class Contacts {
  constructor(router){
    this.heading = 'Child Router';
    this.router = router;

    router.configure(config => {
      config.map([
        { route: 'contacts/details',  moduleId: './contacts/details',      nav: true, title:'Contact Details' },
        { route: 'contacts/charts', moduleId: './contacts/charts', nav: true, title: 'Contact Charts' },
      ]);
    });
  }
}
```

As you can see the child router contains a lot of redundancy which we would like to get rid off, so that the route patterns and moduleIds are calculated relative to the parent "root". We will explore how to achieve this and much more in the chapter *Advanced routing*.

### Using the Child router

To use the child router, simply create a view for it. 


