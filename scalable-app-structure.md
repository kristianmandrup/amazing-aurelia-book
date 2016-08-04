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




