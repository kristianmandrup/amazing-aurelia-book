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

The *Route config decoration* section above sketched out some options for encoding some conventions on the router and having the configuration configured automatically to adhere to these conventions. However it seems a bit too cumbersome to do it via decoration on the route config level.
Would be better to simply instruct the routing engine or the router to use a different convention on all routes. We will explore how to achieve this next, how you can cutomize and encode your own routing strategies and conventions on the router level.

The routing engine can be found in the [router](https://github.com/aurelia/router) and [template-routing](https://github.com/aurelia/templating-router) modules.

To patch the routing engine for your own needs, you need to deep dive into the internals of the engine to understand what goes on. Please clone the above mentioned repos locally, then `npm link` each one.

Now create a new project `app-custom-router` via the CLI.

```
$ npm link aurelia-router
$ npm link aurelia-templating-router
```

This will make symbolic links from the modules in `node_modules` to your locally linked ones, which again link to your cloned repos. In short, your project now uses your local cloned repos for `aurelia-router` and `aurelia-templating-router` which you can debug and play around with.

## Changing default routing strategy

The `TemplatingRouteLoader` loads a route via `loadRoute`.

`templating-router/route-loader.js`

```js
import {inject} from 'aurelia-dependency-injection';
import {CompositionEngine} from 'aurelia-templating';
import {RouteLoader, Router} from 'aurelia-router';

@inject(CompositionEngine)
export class TemplatingRouteLoader extends RouteLoader {
  constructor(compositionEngine) {
    super();
    this.compositionEngine = compositionEngine;
  }
  ...
  loadRoute(router, config) {
    ...
  }
}
```

As you can see `TemplatingRouteLoader` extends `RouteLoader` which has a simple interface:

```js
export class RouteLoader {
  loadRoute(router: any, config: any, navigationInstruction: any) {
    throw Error('Route loaders must implement "loadRoute(router, config, navigationInstruction)".');
  }
}
```

You could write your own `MyTemplatingRouteLoader` and have Aurelia use that instead.

The `aurelia-templating-router` is a standard plugin as you can see in `aurelia-templating-router.js`

```ts
function configure(config) {
  config
    .singleton(RouteLoader, TemplatingRouteLoader)
    .singleton(Router, AppRouter)
    .globalResources('./router-view', './route-href');

  config.container.registerAlias(Router, AppRouter);
}

export {
  TemplatingRouteLoader,
  RouterView,
  RouteHref,
  configure
};
```

## Writing a TemplatingRouteLoader plugin

You could write your own plugin the same way.

Create a new folder `my-templating-router` and run `npm init` from inside.
Then install `aurelia-templating-router` as a dependency as you want to build on top of and customize it.

`npm install aurelia-templating-router --save`

Create a file `my-route-loader.ts` with a class `MyRouteLoader`. 

```ts
import { TemplatingRouteLoader } from 'aurelia-templating-router';

class MyRouteLoader extends TemplatingRouteLoader {
  // custom override functionality
}
```

In your `index.ts` file add the following code. Notice how `.singleton(RouteLoader, MyTemplatingRouteLoader).` registers `MyTemplatingRouteLoader` as the `RouteLoader` for the framework, essentially a singleton registry. Excellent!

```ts
import {Router, AppRouter, RouteLoader} from 'aurelia-router';
import {MyRouteLoader} from './my-route-loader';

function configure(config) {
  config
    .singleton(RouteLoader, MyRouteLoader)
}

export {
  MyRouteLoader
};
```

Then you just need to make Aurelia use your custom templating router plugin ;)
Instead of using the `standardConfiguration`

```js
export function configure(aurelia) {
   aurelia.use
   .standardConfiguration()
```

We tell Aurelia boostrapper exactly what to use...

```
export function configure(aurelia) {
   aurelia.use
   .defaultBindingLanguage()
   .defaultResources()
   .developmentLogging()
   .router()
   .history()
   .eventAggregator();

   // ...
}
```

The key here is the `router()` which if we look into [framework-configuration](https://github.com/aurelia/framework/blob/master/src/framework-configuration.js) adds the [aurelia-templating-router](https://github.com/aurelia/templating-router) plugin.

```js
  router(): FrameworkConfiguration {
    return this._addNormalizedPlugin('aurelia-templating-router');
  }
```

We want to still keep the registrations for `Router` and the global resources for `router-view` and `route-href`.

```ts
    .singleton(Router, AppRouter)
    .globalResources('./router-view', './route-href');

  config.container.registerAlias(Router, AppRouter);
```

However we want to override the registration entry for `RouteLoader`.

To achieve this override we can add `aurelia.use.plugin('my-templating-router')` at the end of the configuration. This will  effectively override the registry for `RouteLoader` with `MyTemplatingRouteLoader`. Perfect :)

```js
export function configure(aurelia) {
   aurelia.use
   .defaultBindingLanguage()
   // ...
   .router()
   .eventAggregator();

   // Use my own templating route loader!!
   aurelia.use.plugin('my-templating-router');

   aurelia.start().then(() => aurelia.setRoot());
}
```

Now we can customize our `MyRouteLoader` to suit our needs :)

### RouteLoader

If we look at `TemplatingRouteLoader`, we see that the default stragegy for `loadRoute` to find a VM module is defined for `instruction.viewModel`.

```js
import {relativeToFile} from 'aurelia-path';

export class TemplatingRouteLoader extends RouteLoader {
  // ...

  loadRoute(router, config) {
    let childContainer = router.container.createChild();
    let instruction = {

      viewModel: relativeToFile(config.moduleId, Origin.get(router.container.viewModel.constructor).moduleId);
    // ...
```

The relative file path of the VM module is calculated from the route `moduleId` and the router parent (container) `moduleId`.

`Origin.get(router.container.viewModel.constructor).moduleId` finds the moduleId of the parent router viewModel.

The full path is thus calculated like this:

`/<parent module id>/<route module id>`

You could f.ex change this to `relativeToFile(config.moduleId)` to have it be the route `moduleId` only, or alternatively `relativeToFile('index', config.moduleId);` to have `./contacts` be resolved to `./contacts/index`.

However instead of changing strategy in the templating router, why not let each individual `router` have the option to implement its own strategy.

## Putting the router in charge

Let's change our `TemplatingRouteLoader` to achieve this. We change the `instruction.viewModel` to `viewModel: this.viewModelLocation(router, config)`

```js
  loadRoute(router, config) {
    let childContainer = router.container.createChild();
    let instruction = {
      viewModel: this.viewModelLocation(router, config),  // <--- CHANGED
      childContainer: childContainer,
      view: config.view || config.viewStrategy,
      router: router
    };
```

Now lets introduce the function `viewModelLocation`, where we first try to use the `router` delegate function with a fallback to the default strategy from before.

```js
  viewModelLocation(router, config) {
    if (router.viewModelLocation) {
      return router.viewModelLocation(config);
    } else {
      return relativeToFile(config.moduleId, Origin.get(router.container.viewModel.constructor).moduleId);
    }
  }
```

## Customizing the router

On the `Router`, we can then introduce the `viewModelLocation` delegate function which we set to use the standard resolution strategy by default as well.

*Important*: We must now import the function `relativeToFile` from `aurelia-path`.

`router/router.js`

```js
import {Router} from 'aurelia-router';
import {relativeToFile} from 'aurelia-path';

export class MyRouter extend Router {
  ...

  viewModelLocation(config) {
      return relativeToFile(config.moduleId, Origin.get(this.container.viewModel.constructor).moduleId);
  }
```

Now for any viewModel with router we can instead inject the `MyRouter` singleton which provides a `viewModelLocation` we can use to define different VM location strategy, while still have access to the original Router with the default strategy only. Pure Awesomeness!

### Router API

- `isRoot(): boolean` is it the root router?
- `reset()` - resets the router's internal state
- `hasRoute(name: string): boolean` - find route by name (incl parent routers)
- `hasOwnRoute(name: string): boolean` - find route by name (only this router)
- `updateTitle()` - update document title using current navigation instruction
- `navigateBack(): void` - Navigates back to the most recent location in history.

`refreshNavigation(): void` - Updates the navigation routes with hrefs relative to the current location

`configure(callbackOrConfig: RouterConfiguration|((config: RouterConfiguration) => RouterConfiguration)): Promise<void>` - configure the router

`navigate(fragment: string, options?: any): boolean` - navigate to location

`navigateToRoute(route: string, params?: any, options?: any): boolean` - navigate to location corresponding to the route and params

`createChild(container?: Container): Router` - Creates a child router of the current router.

`addRoute(config: RouteConfig, navModel?: NavModel): void` - Registers a new route with the router.

`handleUnknownRoutes(config?: string|Function|RouteConfig): void` - use a `moduleId`, a function that selects the `moduleId` or an alternative `RouteConfig` to use to find the module of the VM for an unknown route.

`generate(name: string, params?: any, options?: any = {}): string` - Generates a URL fragment matching the specified route pattern. Uses the `aurelia-route-recognizer` plugin ;)

`createNavModel(config: RouteConfig): NavModel` - Creates a NavModel for the specified route config.

### AppRouter API

The AppRouter is the top level router for the app.

- `loadUrl(url): Promise<NavigationInstruction>` - loads the url
- `activate(options: Object)` - activate router
- `deactivate()` - deactivate router

### Router Configuration API

`addPipelineStep(name: string, step: Function|PipelineStep): RouterConfiguration` - Adds a step to be run during the Router's navigation pipeline.

`addAuthorizeStep(step: Function|PipelineStep): RouterConfiguration`

`addPreActivateStep(step: Function|PipelineStep): RouterConfiguration`

`addPreRenderStep(step: Function|PipelineStep): RouterConfiguration`

`addPostRenderStep(step: Function|PipelineStep): RouterConfiguration`

`map(route: RouteConfig|RouteConfig[]): RouterConfiguration` - Maps one or more routes to be registered with the router

`mapRoute(config: RouteConfig): RouterConfiguration` - Maps a single route to be registered with the router.

`mapUnknownRoutes(config)` - Registers an unknown route handler to be run when the URL fragment doesn't match any registered routes.

`config` - A string containing a moduleId to load, a RouteConfig, or a function that takes the NavigationInstruction and selects a moduleId to load.

`exportToRouter(router: Router): void` - Applies the current configuration to the specified Router.

## More advanced routing recipes

We will now explore a few different common routing recipes.

- Dynamic routes

### Dynamic routes

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
    router:Router;
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

