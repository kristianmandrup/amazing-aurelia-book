# Advanced routing

In Aurelia you can choose to dynamically route rather than pre-configuring all your routes up front.

Here's how you configure a router to do that:

```ts
router.configure(config => {
  config.mapUnknownRoutes(instruction => {
    //check instruction.fragment
    //set instruction.config.moduleId
  });
});
```

All you have to do is set the `config.moduleId` property and you are good to go. You can also return a promise from `mapUnknownRoutes` in order to asynchronously determine the destination.

The router also has an `addRoute` method that can be used to dynmically add a route later.

## Loading routes from a config file

The [aurelia-router-loader](https://github.com/Vheissu/aurelia-router-loader) allows you to load routes from a JSON configuration file.

For a non-root view model, you can use it like this:

```ts
import { Router } from 'aurelia-router';
import { RouterLoader} from 'router-loader/router-loader';
import {Loader} from 'aurelia-loader';

@inject(Router)
class ChildViewModel {
    configureRouter(config, router) {
        this.router = router;

        const loader = new Loader();
        const routeLoader = new RouterLoader(loader, router);

        routeLoader.defineRoutes([
            './routes/main.json',
            './routes/admin.json'
        ])

        // router config
    }

    goSomeWhere() {
        this.router.navigate('somewhere');
    }
}
```

Obviously if you want to go down this path, you should define an abstract baseclass (or mixin) where you can put this logic for reuse across various routable view models ;)

```ts
import { RouterLoader} from 'router-loader/router-loader';
import {Loader} from 'aurelia-loader';

class RouteLoading {
  loadRoutes(routeFiles) {
      const loader = new Loader();
      const routeLoader = new RouterLoader(loader, this.router);

      routeLoader.defineRoutes(routeFiles)      
  }
}
```

Then instead use the `RouteLoading` base class in a routable view model.

```ts
import { Router } from 'aurelia-router';

@inject(Router)
class ChildViewModel extends RouteLoading {  
    configureRouter(config, router) {
      this.router = router;
      this.loadRoutes([
        './routes/main.json',
        './routes/admin.json' 
      ])
    }
}
```


## Route config decoration

The project for this recipe can be found [here](https://github.com/kristianmandrup/aurelia-routing-demo)

Configuring a router to work gracefully with the [Aurelia best practices app layout](http://patrickwalters.net/my-best-practices-for-aurelia-application-structure/)

The application is structured into a nested hierarchy as follows:

```bash
/pages
  - index.ts
  - index.html

  /account
    - index.ts
    - index.html
```

Normally you have to specify the exact path to each ViewModel moduleas follows:

```js
[
  { route: '', moduleId: 'pages/index', title: 'Home', name: 'home', nav: true },
  { route: 'account',  moduleId: 'pages/account/index', title: 'Account', name: 'account', nav: true }
]
```

Looks kinda ugly (and redundant!) having to specify the `index` at the end each time.
We would instead like to achieve the following, and have the router figure out how to resolve the moduleId.

```js
[
  { route: '', moduleId: 'pages', title: 'Home', name: 'home', nav: true },
  { route: 'account',  moduleId: 'pages/account', title: 'Account', name: 'account', nav: true }
]
```

For now the Aurelia `Router` can only use its default module resolution strategy.

We thus want each route decorated with a custom `navigationStrategy` which adds the `index` at the end of each `moduleId` before passing it on to the router module resolution engine.

We have chosen to decorate all the routes in the router with a special `settings` option: `{indexed: true}`.
To achieve this we have built a special `decorateSettings` function which we can use with `map`, mapping over each route.

`const routes = short.map(decorateSettings({indexed: true}));`

Alternatively we could modify the `moduleId` directly on the routes (however this kinda defeats the purpose?).

We ideally want the route `moduleId` to signal the navigation intent (navigate to `pages/account` or even just `/account`) leaving the details of module resolution to the Aurelia routing engine. It is an `Id` after all, not a path. 

Adding a setting is metadata for the engine, so we keep things separate.

Simply decorate `moduleId` of each route.

`function decorateModuleId(route) { ... }`

You can also use `createDecorator(transform, options = {})` in `indexed.ts` to transform moduleIds on all routes given a transform function with options. The options available are `root` and `page`. Root is prepended on the moduleId
and page is a static override of the page name (by default the `route.name`) or a even function to create the page name from the route name.

`routes.map(createDecorator(nestedModuleId, {root: 'pages', page: 'index'}));`

This will transform `account` to `pages/account/index`. Without the `page` option: `pages/account/account`.

## Improving the Routing engine

The *Route config decoration* section above sketched out some options for encoding some conventions on the router and having the configuration configured automatically to adhere to these conventions.



## Advanced routing recipes

We will now explore a few different common routing recipes.

- Dynamic routes
- Multi level menu

### Dynamic routes

See [example](https://github.com/cmichaelgraham/aurelia-typescript/tree/master/code-sandbox#adding-a-route-dynamically)

`dynamic.html`

```html
<template>
  <h2>Dynamic Route</h2>
  <button click.delegate="addDynamicRoute()">Add Dynamic Route</button>
</template>
```

`dynamic.ts`

```ts

import {inject} from "aurelia-framework"
import {Router, RouterConfiguration, RouteConfig} from "aurelia-router";

@inject(RouterConfiguration, Router)
export class Dynamic {
    public addedDynoViewRoute: boolean = false;
    theRouter:Router;
    config: RouterConfiguration;

    constructor(config: RouterConfiguration, router: Router) {
        this.config = config;
        this.router = router;
    }

    addDynamicRoute() {
        let newRoute: RouteConfig = {
            route: 'dyno-view',
            name: 'dyno-view',
            moduleId: 'views/dyno-view',
            nav: true,
            title: 'dyno-view'
        };

        this.theRouter.addRoute( newRoute );
        this.theRouter.refreshNavigation();
        this.theRouter.navigateToRoute('dyno-view');
    }
}
```

### Multi level menu

[@cmichaelgraham](https://github.com/cmichaelgraham) has a nice [Multi level menu example](https://github.com/cmichaelgraham/aurelia-typescript/tree/master/multi-level-menu)

The key is the use of `config.addPipelineStep` to add a step to the routing pipeline.

```
import {Router} from 'aurelia-router';
import { MultiLevelMenuPipelineStep } from './MultiLevelMenuPipelineStep';

@inject(Router)
export class App {
  constructor(public router: Router) {
    this.router.configure((config) => {
      config.title = "Aurelia Multi level menu";
      config.addPipelineStep('modelbind', MultiLevelMenuPipelineStep);

      config.map([
          { route: ['', 'home'], moduleId: 'views/home', nav: true, title: 'home', settings: { level: 0, show: true } },

          { route: ['item-1'], moduleId: 'views/item-1', nav: true, title: 'item 1', settings: { level: 0, show: true } },
          { route: ['item-1-1'], moduleId: "views/item-1-1", nav: true, title: 'item 1.1', settings: { level: 1, show: false } },
          { route: ['item-1-2'], moduleId: 'views/item-1-2', nav: true, title: 'item 1.2', settings: { level: 1, show: false } },
          // ...
      ]);
    });
  }
}
```


The `MultiLevelMenuPipelineStep` finds and sets the `targetRouteIndex`

```ts
import { NavigationContext } from 'aurelia-framework';
import { Router } from 'aurelia-router';
import { MultiLevelMenuUtil } from './MultiLevelMenuUtil';

export class MultiLevelMenuPipelineStep {
    run(routingContext: NavigationContext, next: { (): void; cancel(): void; }) {
        var targetRouteIndex = MultiLevelMenuUtil.getTargetRouteIndex(routingContext.router, routingContext.plan.default.config.moduleId);
        MultiLevelMenuUtil.setForTarget(routingContext.router, targetRouteIndex);
        return next();
    }
}
```

`navigate-up` custom element

`navigate-up.html`

```html
<template>
    <button class="btn btn-info" click.delegate="navigateUp()">-^-</button>
</template>
```

A view helper

```ts
import aurelia from 'aurelia-framework';
import { Router } from 'aurelia-router';
import { MultiLevelMenuUtil } from './MultiLevelMenuUtil';

export class MultiLevelMenuHelper {

    public router: router.Router;

    static metadata = aurelia.Behavior.withProperty('router');

    navigateUp() {
        MultiLevelMenuUtil.goUp(this.router);
    }
}
```

To really explore this example, please see the [MultiLevelMenuUtil](https://github.com/cmichaelgraham/aurelia-typescript/blob/master/multi-level-menu/multi-level-menu/views/MultiLevelMenuUtil.ts)





