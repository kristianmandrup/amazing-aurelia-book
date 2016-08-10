# Authorization

Authorization strategies.

Routing strategies on viewModels such as `canActivate` etc.
Using a 3rd party library such as [permits]().

### Authorized Routing

First we add a pipeline step to the router:

`config.addPipelineStep('authorize', AuthorizeStep);`

```js
router.configure(config => {
    config.addPipelineStep('authorize', AuthorizeStep);
    config.map([
        { route: '', moduleId: './logged-in' },
        { route: 'login', moduleId: './login' }
    ]);
}
```

```js
export class AuthorizeStep {
    run (routingContext, next) {
        if (routingContext.nextInstruction[0].config.route == 'login')
            return next();
        return next.cancel(new Redirect('login'));
    }
}
```

### Take 2

Aurelia has a pipeline structure for the navigation pipeline. When a request comes in, the request is first passed through a series of extensibility steps where you can add your own code. There are two steps defined by default – `authorize` and `modelbind`. You can define your own as well and fit them into the pipeline. The step `authorize` comes before `modelbind`. We can use this to authenticate users.

First of all, let's created a new class called `AuthorizeStep`. We placed this class in the `app.ts` file.

```ts
class AuthorizeStep {
    run(routingContext, next) {
        if (routingContext.nextInstructions.some(i => i.config.auth)) {
            var isLoggedIn = AuthorizeStep.isLoggedIn();
            if (!isLoggedIn) {
                alert("Not Logged In!\nClick the Sign In icon to log in");
                return next.cancel();
            }
        }
        return next();
    }

    static isLoggedIn(): boolean {
        var auth_token = localStorage.getItem("auth_token");
        return (typeof auth_token !== "undefined" && auth_token !== null);
    }
}
```

The `isLoggedIn()` method checks for my `auth_token` in local storage and returns if the user is logged in or not (true/false).

Now let's add the `AuthorizeStep` to the router configuration:

```ts
export class App {
    public router: any;

    configureRouter(config, router) {
        config.title = 'Aurelia';
        config.addPipelineStep('authorize', AuthorizeStep);
        config.map([
            { route: 'welcome', name: 'welcome', moduleId: './pages/welcome', nav: true, title: 'Welcome' },
            { route: 'flickr', name: 'flickr', moduleId: './pages/flickr', nav: true, auth: true, title: 'Flickr' },
            { route: '', redirect: 'welcome' }
        ]);
        this.router = router;
    }
}
```

We would ideally like want to hide the authenticated pages unless the user is signed in. Aurelia has a navigation data model that it constructs from the route map. It is triggered by the nav property on each route. So far, all our routes have had nav: true on them.

However, if we adjust the `nav` to be the value of `AuthorizeStep.isLoggedIn()`, then the links will disappear from the navigation model when the user is not logged in. We can adjust the route configuration as follows:

```js
config.map([
    { route: 'welcome', name: 'welcome', moduleId: './pages/welcome', nav: true, title: 'Welcome' },
    { route: 'flickr', name: 'flickr', moduleId: './pages/flickr', nav: AuthorizeStep.isLoggedIn(), auth: true, title: 'Flickr' },
    { route: '', redirect: 'welcome' }
]);
```

However there is a bug here. Run this project, sign in and note that the Flickr link appears. Then sign out. Note that the Flickr link does not disappear. Refresh the screen and note that the link is gone. In short, the navigation does not get redrawn when the user signs out.

We need to adjust `app-authenticator` and `nav-bar` so that they know about one another. When the user clicks on sign out, the `app-authenticator` sends a signal to the body of the document that says `signed out`. The nav bar can then listen to that and delete or hide the li elements that are authenticated. One can get these by using the ${row.config.auth} to get the configuration of the route.

