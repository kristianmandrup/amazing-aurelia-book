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




