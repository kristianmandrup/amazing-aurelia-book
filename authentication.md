# Authentication

Using [aurelia-auth](https://github.com/paulvanbladel/aurelia-auth)

*aurelia-auth* is a port of the great [Satellizer](https://github.com/sahat/satellizer/) library to ES6 and packaged as an Aurelia plugin.

aurelia-auth does not use any cookies but relies on a JWT (json web token) stored in the local storage of the browser.
Both local storage as well as session storage can be used (via the aurelia-auth security configuration file).

The aurelia token will be sent automatically to your API when the user is authenticated.

aurelia-auth is a simple service with following interface:

- `login(email, password)`
- `logout(redirectUri)`
- `authenticate(provider, redirect, userData)`
- `signup(displayName, email, password)`
- `getMe()`
- `isAuthenticated()`
- `getTokenPayload()`
- `unlink(provider)`
- `setToken(token)`

`login` is used for the local authentication strategy (email + password). `authenticate` is for social media authentication and is also used for linking a social media account to an existing account.

```js
var configForDevelopment = {
    providers: {
        google: {
            clientId: '239531826023-ibk10mb9p7ull54j55a61og5lvnjrff6.apps.googleusercontent.com'
        }
        ,
        linkedin:{
            clientId:'778mif8zyqbei7'
        },
        facebook:{
            clientId:'1452782111708498'
        }
    }
};

var configForProduction = {
    providers: {
        google: {
            clientId: '239531826023-3ludu3934rmcra3oqscc1gid3l9o497i.apps.googleusercontent.com'
        }
        ,
        linkedin:{
            clientId:'7561959vdub4x1'
        },
        facebook:{
            clientId:'1653908914832509'
        }

    }
};
var config ;
if (window.location.hostname==='localhost') {
    config = configForDevelopment;
}
else{
    config = configForProduction;
}


export default config;
```

In your plugin configuration, use the 

```js
import config from './authConfig';

export function configure(aurelia) {
  aurelia.use
    // ...
    .plugin('aurelia-auth', (baseConfig)=>{
     baseConfig.configure(config);
});
```

### Configure the Fetch Client

In your aurelia app file, inject the {FetchConfig} class from aurelia-auth. We need to explicitely opt-in for the configuration of your fetch client by calling the configure function of the FetchConfig class:

```js
import 'bootstrap';

import {inject} from 'aurelia-framework';
import {Router} from 'aurelia-router';
import {FetchConfig} from 'aurelia-auth';
@inject(Router,FetchConfig, AppRouterConfig )
export class App {

  constructor(router, fetchConfig, appRouterConfig){
    this.router = router;
    this.fetchConfig = fetchConfig;
  }

  activate(){
    this.fetchConfig.configure();
  }
}
```

### How it works

A default `Accept` header is added and a request interceptor is injected. `tokenInterceptor` can be found in `authentication.js` and is responsible for adding the bearer token to each request message making use of the default Http Client.

What if we don't want this to happen for all Http request we make or imagine we have multiple end points and some of them do not require authentication.

The solution is very simple and based inheritance to derive a custom Http Client.

Here is how we can create a custom Http Client:

```js
import {HttpClient} from 'aurelia-fetch-client';
import {inject} from 'aurelia-framework';
import 'isomorphic-fetch'; // if you need a fetch polyfill
import {AuthService} from 'aurelia-auth';

@inject(AuthService)
export class CustomHttpClient extends HttpClient {
    constructor(auth) {
        super();
        this.configure(config => {
            config
                .withBaseUrl('http://localhost:4000/')
                .withDefaults({
                    credentials: 'same-origin',
                    headers: {
                        'Accept': 'application/json',
                        'X-Requested-With': 'Fetch'
                    }
                })
                //we call ourselves the interceptor which comes with aurelia-auth
                //obviously when this custom Http Client is used for services 
                //which don't need a bearer token, you should not inject the token interceptor
                .withInterceptor(auth.tokenInterceptor)
                //still we can augment the custom HttpClient with own interceptors
                .withInterceptor({
                    request(request) {
                        console.log(`Requesting ${request.method} ${request.url}`);
                        return request; // you can return a modified Request, or you can short-circuit the request by returning a Response
                    },
                    response(response) {
                        console.log(`Received ${response.status} ${response.url}`);
                        return response; // you can return a modified Response
                    }
                });
                });
    }
}
```

no specialised aurelia plugins needed for supporting multiple endpoints !

We can consume this custom Http Client as follows:

```js
import {inject, useView} from 'aurelia-framework';
import {CustomHttpClient} from './customHttpClient';
import 'isomorphic-fetch'; // if you need a fetch polyfill
@inject(CustomHttpClient)
@useView('./customer.html')
export class Customer2{
  heading = 'Customer management with custom http service';
  customers = [];

  url = 'api/customer';

  constructor(http){
    this.http = http;
  }

  activate(){
   return this.http.fetch(this.url)
    .then(response =>  response.json())
    .then(c => this.customers = c);
  }
}
```

Provide a UI for a login, signup and profile.

See aurelia-auth-samples for more details.

Button actions are passed to the corresponding view model via a simple click.delegate:

<button class="btn btn-block btn-google-plus" click.delegate="authenticate('google')">
          <span class="ion-social-googleplus"></span>Sign in with Google
</button>
The login view model will speak directly with the aurelia-auth service, which is made available via constructor injection.

import {AuthService} from 'aurelia-auth';
import {inject} from 'aurelia-framework';
@inject(AuthService)

export class Login{
    constructor(auth){
        this.auth = auth;
    };

    heading = 'Login';

    email='';
    password='';
    login(){
        return this.auth.login(this.email, this.password)
        .then(response=>{
            console.log("success logged " + response);
        })
        .catch(err=>{
            console.log("login failure");
        });
    };

    authenticate(name){
        return this.auth.authenticate(name, false, null)
        .then((response)=>{
            console.log("auth response " + response);
        });
    }
}
On the profile page, social media accounts can be linked and unlinked. For a nice UI experience, use if.bind for either showing the link or unlink button:

<button class="btn btn-sm btn-danger" if.bind="profile.facebook" click.delegate="unlink('facebook')">
    <i class="ion-social-facebook"></i> Unlink Facebook Account
</button>
<button class="btn btn-sm btn-primary" if.bind="!profile.facebook" click.delegate="link('facebook')">
    <i class="ion-social-facebook"></i> Link Facebook Account
</button>
Making the Aurelia Router authentication aware

The logout and profile links are only shown when the user is authenticated, whereas the login link is only visible when the user is not authenticated.

<div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
    <ul class="nav navbar-nav">
        <li repeat.for="row of router.navigation | authFilter: isAuthenticated" class="${row.isActive ? 'active' : ''}">
            <a data-toggle="collapse" data-target="#bs-example-navbar-collapse-1.in" href.bind="row.href">${row.title}</a>
        </li>
    </ul>

    <ul if.bind="!isAuthenticated" class="nav navbar-nav navbar-right">
        <li><a href="/#/login">Login</a></li>
        <li><a href="/#/signup">Sign up</a></li>
    </ul>
    <ul if.bind="isAuthenticated" class="nav navbar-nav navbar-right">
        <li><a href="/#/profile">Profile</a></li>
        <li><a href="/#/logout">Logout</a></li>
    </ul>

    <ul class="nav navbar-nav navbar-right">
        <li class="loader" if.bind="router.isNavigating">
            <i class="fa fa-spinner fa-spin fa-2x"></i>
        </li>
    </ul>
</div>
Menu items visibility can also be linked with the authFilter to the isAuthenticated value.

In the router config function, you can specifify an auth property in the routing map indicating wether or not the user needs to be authenticated in order to access the route:

```js
configure(){
  var appRouterConfig = function(config){
    config.title = 'Aurelia';
    config.addPipelineStep('authorize', AuthorizeStep); // Add a route filter to the authorize extensibility point.

    config.map([
      { route: ['','welcome'],  moduleId: './welcome',      nav: true, title:'Welcome' },
      { route: 'flickr',        moduleId: './flickr',       nav: true, title:'Flickr' },
      { route: 'customer',        moduleId: './customer',       nav: true, title:'CRM', auth:true },

      ...

    ]);
  };

  this.router.configure(appRouterConfig);
}
```

In the above example the customer route is only available for authenticated users.

### Events

At each step of authentication (login, logout, authenticate, signup, unlink), an event is published to Aurelia's event aggregator. The events published are as follows:

login: `auth:login`
logout: `auth:logout`
authenticate: `auth:authenticate`
signup: `auth:signup`
unlink: `auth:unlink`

From [creating your first aurelia app from authentication to calling anapi](https://auth0.com/blog/creating-your-first-aurelia-app-from-authentication-to-calling-an-api/)

### Installation

`npm install aurelia-auth --save`

### Server setup

[nodejs-jwt-authentication](https://github.com/auth0-blog/nodejs-jwt-authentication-sample)

### Auth Config

`auth-config.ts`

```
var config = {

  // Our Node API is being served from localhost:3001
  baseUrl: 'http://localhost:3001',
  // The API specifies that new users register at the POST /users enpoint.
  signupUrl: 'users',
  // Logins happen at the POST /sessions/create endpoint.
  loginUrl: 'sessions/create',
  // The API serves its tokens with a key of id_token which differs from
  // aureliauth's standard.
  tokenName: 'id_token',
  // Once logged in, we want to redirect the user to the welcome view.
  loginRedirect: '#/welcome',

}

export default config;
```

The API is accessible at `localhost:3001`, so we set this as our `baseUrl`. Next, we set up the proper endpoints that we'll need for signing users up and logging them in. We also need to override the `tokenName` with what our API serves, which in this case is `id_token`. Finally, we say that we want to redirect the user to the `welcome` view once they login.

### Application Routing Config

We'll now need to set up the application's routing configuration. Let's first set up the HTML that will require and load our nav bar and other views:

```html
  <!-- client/src/app.html -->
  <template>
    <require from='./nav-bar'></require>

    <nav-bar router.bind="router"></nav-bar>  

    <div class="container">      
      <router-view></router-view>
    </div>

  </template>
```

Here, we are requiring the nav-bar and binding it to the router. We will serve our views from the <router-view> within our containing <div>.

```js
// client/src/app.js

import 'bootstrap';
import 'bootstrap/css/bootstrap.css!';

import {inject} from 'aurelia-framework';
import {Router} from 'aurelia-router';
import HttpClientConfig from 'paulvanbladel/aureliauth/app.httpClient.config';
import AppRouterConfig from 'router-config';

// Using Aurelia's dependency injection, we inject Aurelia's router,
// the aurelia-auth http client config, and our own router config
// with the @inject decorator.
@inject(Router, HttpClientConfig, AppRouterConfig)

export class App {

  constructor(router, httpClientConfig, appRouterConfig) {

    this.router = router;

    // Client configuration provided by the aureliauth plugin
    this.httpClientConfig = httpClientConfig;

    // The application's configuration, including the
    // route definitions that we've declared in router-config.js
    this.appRouterConfig = appRouterConfig;
  };

  activate() {

    // Here, we run the configuration when the app loads.
    this.httpClientConfig.configure();
    this.appRouterConfig.configure();

  };
}
```

The HTTP configuration that aurelia-auth provides is what handles adding the JWT as a header if the user is authenticated. The httpClientConfig file has logic that checks for the existence of a token in localstorage and then adds an Authorization header with a value of Bearer <token> if one exists. The token will be sent for all HTTP requests to the API but will obviously only be needed for protected resources.

We can keep our routing logic within the main app.js file, as is done in many Aurelia projects, but in our case, we'll put this configuration in a separate file called router-config.js that we are injecting. Let's set up this routing configuration:

```js
// client/src/router-config.js

import {AuthorizeStep} from 'paulvanbladel/aureliauth';
import {inject} from 'aurelia-framework';
import {Router} from 'aurelia-router';

// Using Aurelia's dependency injection, we inject Router
// with the @inject decorator
@inject(Router)

export default class {

  constructor(router) {
    this.router = router;
  };

  configure() {

    var appRouterConfig = function(config) {

      config.title = 'Random Quotes App';

      // Here, we hook into the authorize extensibility point
      // to add a route filter so that we can require authentication
      // on certain routes
      config.addPipelineStep('authorize', AuthorizeStep);

      // Here, we describe the routes we want along with information about them
      // such as which they are accessible at, which module they use, and whether
      // they should be placed in the navigation bar
      config.map([
          { route: ['','welcome'], name: 'welcome', moduleId: './welcome', nav: true, title:'Welcome' },
          { route: 'random-quote', name: 'random-quote', moduleId: './random-quote', nav: true, title:'Random Quote' },          
          // The secret-quote route is the only one that the user needs to be logged in to see,  so we set auth: true
          { route: 'secret-quote', name: 'secret-quote', moduleId: './secret-quote', nav: true, title:'Super Secret Quote', auth: true },
          { route: 'signup', name: 'signup', moduleId: './signup', nav: false, title:'Signup', authRoute: true },
          { route: 'login', name: 'login', moduleId: './login', nav: false, title:'Login', authRoute: true },
          { route: 'logout', name: 'logout', moduleId: './logout', nav: false, title:'Logout', authRoute: true }
        ]);
      };

    // The router is configured with what we specify in the appRouterConfig
    this.router.configure(appRouterConfig);

  };
}
```

Aurelia gives us the ability to customize the navigation pipeline with some extensibility points, including an authorize route filter. Using this filter means we can specify which routes we would like authentication to be required for. Since our super-secret-quotes route needs to remain top secret until the user is logged in, we put auth: true in it. We hook into this filter by calling addPipelineStep, passing in the AuthorizeStep that is provided by the aurelia-auth plugin.

With the configuration out of the way, let's get to coding the actual routes and their views! We'll need to have files that take care of each route in place before the app will work so you can comment out the routes in router-config.js that aren't ready yet.

### Setting up Routes and Views

Two files are required for each route in Aurelia--a JavaScript file for the view model logic and an HTML file for the view itself. Views are enclosed within `<template>` tags but are otherwise created with normal HTML that can make use of Aurelia's databinding.

### The Nav Bar and Welcome Route

Let's start at the top and setup the navigation bar.

```html
  <!-- nav-bar.html -->

  ...

  <ul class="nav navbar-nav">
    <li repeat.for="row of router.navigation | authFilter: isAuthenticated" class="${row.isActive ? 'active' : ''}">
      <a data-toggle="collapse" data-target="#bs-example-navbar-collapse-1.in" href.bind="row.href">${row.title}</a>
    </li>
  </ul>

  <ul if.bind="!isAuthenticated" class="nav navbar-nav navbar-right">
    <li><a href="/#/login">Login</a></li>
    <li><a href="/#/signup">Signup</a></li>
  </ul>

  <ul if.bind="isAuthenticated" class="nav navbar-nav navbar-right">
    <li><a href="/#/logout">Logout</a></li>
  </ul>

  ...
```

Notice here that we're running a filter on the repeated navigation items with authFilter: `isAuthenticated`. This allows us to hide any nav menu items that are to be protected if the user isn't authenticated, and this is how we will hide the super-secret-quote menu item when the user isn't logged in. We're also conditionally showing the `Signup`, `Login`, and `Logout` links. See the GitHub repo for the rest of the markup.

```js
// client/src/nav-bar.js

import {bindable} from 'aurelia-framework';
import {inject} from 'aurelia-framework';
import {AuthService} from 'paulvanbladel/aureliauth';

@inject(AuthService)

export class NavBar {
  // User isn't authenticated by default
  _isAuthenticated = false;
  @bindable router = null;

  constructor(auth) {
    this.auth = auth;
  };

  // We can check if the user is authenticated
  // to conditionally hide or show nav bar items
  get isAuthenticated() {
    return this.auth.isAuthenticated();
  };
}
```

Here in the `nav-bar.js` file, we have a method that checks whether the user is logged in, which is what we hook into in the view.

The Aurelia seed comes with a welcome route, but in our case, we can trim it down to be simpler.

```html
  <!-- client/src/welcome.html -->

  <template>
    <section>
      <h2>${heading}</h2>

      <div class="well">
        <h4>${info}</h4>
      </div>

    </section>
  </template>
```

The JavaScript becomes simpler as well.

```js
// client/src/welcome.js

export class Welcome {

  heading = 'Welcome to the Random Quotes App!';
  info = 'You can get a random quote without logging in, but if you do log in you can get a super secret quote!';

}
```

## Signup, Login, and Logout

Next, let's set up the `signup`, `login`, and `logout` routes.

### Signup

```html
  <!-- client/src/signup.html -->

  ...

  <form role="form" submit.delegate="signup()">
    <div class="form-group">
      <label for="email">Email</label>
      <input type="text" value.bind="email" class="form-control" id="email" placeholder="Email">
    </div>
    <div class="form-group">
      <label for="password">Password</label>
      <input type="password" value.bind="password" class="form-control" id="password" placeholder="Password">
    </div>
    <button type="submit" class="btn btn-default">Signup</button>
  </form>
  <hr>
  <div class="alert alert-danger" if.bind="signupError">${signupError}</div>

  ...
```

In this view, we're providing two <input>s that take the user's email and password. We've also got an alert box at the bottom to show the user any errors that are returned. We'll need to set up the view models for these next.

```js
// client/src/signup.js

import {inject} from 'aurelia-framework';
import {AuthService} from 'paulvanbladel/aureliauth';

// Using Aurelia's dependency injection, we inject the AuthService
// with the @inject decorator
@inject(AuthService)

export class Signup {

  heading = 'Sign Up';

  // These view models will be given values
  // from the signup form user input
  email = '';
  password = '';

  // Any signup errors will be reported by
  // giving this view model a value in the
  // catch block within the signup method
  signupError = '';

  constructor(auth) {
    this.auth = auth;
  };

  signup() {

    // Object to hold the view model values passed into the signup method
    var userInfo = { email: this.email, password: this.password }

    return this.auth.signup(userInfo)
    .then((response) => {
      console.log("Signed Up!");
    })
    .catch(error => {
      this.signupError = error.response;
    });

  };
}
```

The `signup()` method uses aurelia-auth to send a POST request to the API, which either creates a new user or returns an error if there was a problem.

### Login

The login route is pretty similar. You'll just need to swap out `submit.delegate="signup()"` for `submit.delegate="login()"` and adjust the other pieces of markup appropriately.

The JavaScript for login looks similar as well, but this time, we are sending the POST request to the `sessions/create` endpoint:

```js
// client/src/login.js

...

  login() {
    return this.auth.login(this.email, this.password)
    .then(response => {
      console.log("Login response: " + response);
    })
    .catch(error => {
      this.loginError = error.response;
    });
  };

...
```

### Logout

The logout route essentially follows the same pattern using `authService.logout()` to remove the user's JWT from localstorage. See the repo for further detail.

With all this in place, we should now be able to signup, login, and logout users. Test it out to make sure that everything is running as expected. If everything is working properly, when the user logs in there will be a JWT set in localstorage.

### The Random Quote and Super-Secret Quote Routes

With `signup`, `login`, and `logout` in place, we now need to create the files for our quote routes. Let's first take care of the `random-quote` route.

```html
  <!-- client/src/random-quote.js -->

  <template>
    <section class="col-sm-12">
        <h2>${heading}</h2>
        <div class="row">
          <div class="well">
            <h4>${randomQuote}</h4>
          </div>
        </div>
    </section>
  </template>
```

This view simply displays the heading and the text of the quote that we retrieve.

```js
// client/src/random-quote.js

import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-http-client';

// Using Aurelia's dependency injection, we inject HttpClient
// with the @inject decorator to make HTTP requests
@inject(HttpClient)

export class RandomQuote {

  heading = 'Random Quote';

  // View model that will be populated with the
  // the random quote retrieved from the API and
  // displayed in the view
  randomQuote = '';

  constructor(http) {
    this.http = http;
  };

  activate() {
    return this.http.get('http://localhost:3001/api/random-quote')
    .then(response => {
      this.randomQuote = response.content;
    }).catch(error => {
      console.log('Error getting quote');
    });
  };
}
```

We want to fetch the quote when the route is hit, so within the `activate()` method, we are making a `GET` request to our `random-quote` endpoint, which is located at `localhost:3001/api/random-quote`. If we get a good response, we set the quote text onto `randomQuote` so that it can be accessed in the view.

The `super-secret-quote` route is pretty much the same, except that we make our requests to a different endpoint. For the view in `secret-quote.html`, make sure to change `${randomQuote}` to `${secretQuote}`

```js
// client/src/secret-quote.js

...

activate() {
  return this.http.get('http://localhost:3001/api/protected/random-quote')
  .then(response => {
    this.secretQuote = response.content;
  }).catch(error => {
    console.log('Error getting quote');
  });
}
...
```

As you can see, the only real difference here is that the GET request we're making is going to the protected/random-quote endpoint. If there is no valid JWT in localStorage, we won't be able to get to this route. If somehow we got to it, the request would fail because no JWT would be sent to the server.
