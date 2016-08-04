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

The router also has an `addRoute` method that can be used to add a  route later.

## Routing recipes

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





