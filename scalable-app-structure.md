# Scalable app structure

[Application structure](http://patrickwalters.net/application-structure/)

The basis of the application structure are the five types of directories -

- Models
- Resources
- Routes
- Services
- Components(or templates)

## Components (or templates)
Components (or templates) are basically where shared components exist. If I have a person model I want to make sure I have an edit-person component as well as a person-details component. This is so anywhere I am showing a person I can use the details view and anytime I need to make an edit, the template is ready to rock.

## Models
Models are self-explanatory. These are the classes that define the various models in your application, at each level. At the root level we have application-wide models such as the User.

## Resources
Resources are kind of the catch all. Anything that acts as a supplement to your application goes here. I put various classes in charge of model collections here as well as utility functions.

## Routes
The routes directory is a great way of grouping content. Typically if a route implements a child router all of it's content and files relevant only to that child route go nested inside to keep things clean.

## Services
In the Services directory goes all of your JavaScript which performs a service for other files such as your view-models. These are usually services for things like fetching data and returning it in a standard format.

## Naming
You'll notice that the directories are specifically named. Ex. services. This is so that as things grow the name duplication isn't necessary.

Example of why naming is important:

```bash
- app
|-- person
  |-- person.js // view-model
  |-- person.html // view
  |-- person-model.js // model
  |-- person-service.js // model
There is a lot of duplication here. Instead by using the directory as the description of the purpose or intent our nested content becomes easier to read 
```

```bash
- app
|-- routes
  |-- person
    |-- index.js
    |-- index.html
|-- models
  |-- person.js
|-- services
  |-- person.js
```


## Scaling the application
As the application continues to grow each of the routes start to have child routers for displaying the relevant content within. Each of the parent routes should start out by adding each of the five pillars of the application within themselves.

Example:

```bash
- app
|-- components
  |-- edit-user.js
  |-- edit-user.html
  |-- user-details.js
  |-- user-details.html
|-- models
  |-- user.js
|-- resources
|-- routes
  |-- persons
    |-- models
      |-- person.js
    |-- resources
    |-- services
      |-- person.js
    |-- index.js
    |-- index.html
  |-- admin
    |-- index.js
    |-- index.html
|-- services
  |-- user.js
```

Each of the routes should have an `index.js` and `index.html`. This is to act as the controller and define the child routes.

- `src` Root directory and contains; `app.html`, `app.js` and `main.js`
- `assets` Where all assets of the app live (with separate sub-folders)
- `components` Where common application components like headers, navigation, footer, etc live
- `lib` Where generic non-app specific code lives. Utility classes and functions for doing utility-like tasks
- `pages` Where our route matching page Views and ViewModels live. This matches the predefined routes
- `resources` Where our custom elements and value converters live
- `services` A folder for singleton service classes like keeping track of state or working with API’s

```
src/
    app.html
    app.js
    main.js

    assets/
        fonts/
        images/
        styles/

    components/

    lib/

    pages/
        page1/
            index.html
            index.js

            page1Sub/
                index.html
                index.js

    resources/
        custom-elements/
        value-converters/

    services/
```

## Multiple apps

For large applications it can make sense to break the app into multiple sub applications per user role or business domain.

Typically you would subdivide on:
- guest
- user
- admin

The `guest` app would be the application the user can navigate as as guest, ie. before successful login. It is often a minimal app with a splash screem, login screen, a login service and so on.

```
/guest
  /pages
    router.js
    /login
      index.js
      index.html

  /services
    index.js // re-exports all

  /components
    /modal
      ...
  /resources
```

The `user` app is for the logged in normal user. This is the core app.

```
/user
  /pages
    router.js
    /contacts
      index.js
      index.html
      /one
        index.js
        index.html

        /session
          index.js
          index.html

      /services
        logger.js

      /components
        index.html

        /contacts
          index.js
          index.html

          /details
            index.js
            index.html

          /assets
            contact.png

          service.js
          store.js
          package.json
          install-config.json

        /user
          ...

      /resources
        index.html
        /attributes
        /binding-behaviors
        /elements
          /progress-bar
            index.js
            index.html
            package.json
            bundles.json

        /value-converters
```

The `admin` app is for users logged in as administrators. It might consist of a dashboard and perhaps some way to navigate and update all the models of the app, monitor logs and users, display various usage statistics as charts etc.

```
    /admin
      /pages
      /services
      /components
      /resources
```

The final application layout would then look something like the following:

```bash
src/
  apps/
    /guest
    /user
    /admin

  /common
    /assets
      /images
      /styles
      /fonts

    /services
    /components
    /resources

  /plugins
    index.js
    auth.js
    ...
```

### Assets

We recommend using Webpack as the package manager and bundler tool of choice!
Load your assets directly from the `.js` files which need them!

### Plugins

Use `feature('plugins')` when configuring aurelia to centralize plugins configuration. Have separate plugin config files
for plugins that require more refined customization.

### Components

Use `feature('components')` and `feature('resources')` so as to faciliate choosing which ones to make `globalResources`.

Pages are components that usually has a View (the page to display), but can optionally use another rendering method. An element is a reusable page fragment.

Any component can have its own repo. A component can also be a hierarchy of sub-components with their own repo.
To manage dependencies to external libs, each component can have a `vendor-bundles.json` file.

Note also that certain components may have their own assets in an `/assets` folder. Include the `assets` (using appropriately installed Webpack loader) directly from your `.js` file.

### Pages

Pages are special components to display a full "page" in your app. They can be seen as top-level molecules.

### Services

Services should always be Singletons and so should never use `@transient`.
Note that some services are general purpose for the entire app. 
For services specific to a page or component, put it there where it rightfully belongs.

You should always be able to freely move a domain object: page or component as an independent, encapsulated entity.

## Switching apps on type of user

The key is switching the root of the app, using `aurelia.setRoot(<app location>));`

```js
aurelia.start().then(() => aurelia.setRoot('apps/guest'));

// when logged in as user
aurelia.setRoot('apps/user'));

// when logged in as admin
aurelia.setRoot('apps/admin'));
```

