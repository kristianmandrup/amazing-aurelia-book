# State management

Using Models and Services and ORMs for state management.

- Simple Models and JSON services
- [Aurelia ORM](https://github.com/SpoonX/aurelia-orm)

### Simple Models and JSON services

From [solving-the-m-in-mvvm](http://patrickwalters.net/my-best-practices-for-aurelia-solving-the-m-in-mvvm/)

In the past with JavaScript we've been limited by how well we can solve for the model aspect in the MVVM pattern. Traditionally we've been limited to creating a function that we pass off as a class or using a more robust client-side ORM such as Breeze.js. The function was a little dumbed down and sometimes an ORM is overkill when starting a new project.

What about the middle ground where we want to have a model we can work with?

### ES6 Classes to the rescue
With the newest standard being approved we now have ES6 classes. With classes we can have a single area to start with to extend our models.

```
export class Person {  
  constructor(data){
    Object.assign(this, data);
  }
}
```

### Object.assign()
Now we've created a model for a `Person`. We know that there is some JSON we are getting from the server and we want to 'cast' that data into an instance of a `Person`. We can use `Object.assign()`` here in our constructor to tell our model that whatever properties the server is returning we want to keep. This means that if our person's data is returned with a property like firstName it will also exist in our model. Then we can extend it.

That works great, but doesn't really offer us any tangible benefits yet.

### Adding properties
Our person is lonely and broke. Whenever we create a new person let's give them no money whatsoever and make them understand the hard lessons of life and respect what it means to earn a buck -

```
export class Person {  
  constructor(data){
    Object.assign(this, data);
    this.money = 0;
  }
}
```

Now anywhere that we reference our person in our application we can rely on him to have a property called `money`.

But what if we want to give our person some money?

### Fat models
Let's fatten up our model with accessors and behaviour

```
export class Person {  
  constructor(data){
    Object.assign(this, data);
    this.money = 0;
  }
  giveMoney(amount){
      this.money += amount;
  }
}
```

We want to put some business logic in for when we give money to our person.

Imagine that when we give our person some money we also want to improve his credit score.

```
export class Person {  
  constructor(data){
    Object.assign(this, data);
    this.money = 0;
    this.creditScore = 500;
  }
  giveMoney(amount){
      this.money += amount;
      this.creditScore += amount / 2;
  }
}
```

Now whenever we give a person some money the appropriate change to credit score is applied.

### Casting our JSON as a Person
We will now use the `HttpClient` that Aurelia provides to retrieve the JSON data from the server, then 'cast' the data to our `Person` class for each entry retrieved.

### Service
Let's create a simple service which retrieves some people and casts them as a `Person`.

```
import {HttpClient} from 'aurelia-http-client';
import {Person} from './models';

export class PersonService {
  constructor(){
      this.http = new HttpClient().configure(x=> {
        x.withReviver((k,v) => {
          return typeof v === 'object' ? new Person(v) : v;
        });
    });
  }
  getPeople(){
      return this.http.get('/people');
  }
}
```

We import our `Person` from our `models.js` file where we exported it.
We instantiated an instance of the `HttpClient` provided by Aurelia.

We configure our `HttpClient` to always use a `reviver`.
We added a method on our service to use the HttpClient we've created to get people from our resource on the server.

### Reviver
A `reviver` will be called once for each item in the array when our JSON is parsed from the server. By using `withReviver()`` and passing in a function that casts the value to a Person, it helps us do something like this:

```
JSON.parse(response, (key, value) => {
  if (typeof value === 'object') {
      return new Person(value);
  }
  return value;
});
```

### Sample JSON payload
If you want to try this out, use this example JSON payload -

`[{"name": "Jane"},{"name": "Bob"}]`

We've created a model, we've used the `HttpClient` to cast it, and we've touched on some topics that are a bit risque.

Let's end on a high note by showing our usage from our view-model

```
import {PersonService} from './person-service';
import {inject} from 'aurelia-framework';

@inject(PersonService)
export class MyViewModel {
  constructor(personService){
      this.jane = {};
      this.personService = personService;
      this.personService.getPeople().then(response => {
        this.jane = response.content[0];
        this.jane.giveMoney(100);
    });
  }
}
```


## Aurelia ORM

Working with endpoints and client-side entities is unavoidable when doing API driven development. You end up writing a small wrapper for your XHRs / websocket events and putting your request methods in one place. Another option is just using [Breeze](breezejs.com), which is large, complex and not meant for most applications. Even though your endpoints are important, you still end up neglecting them.

Enter [aurelia-orm](https://github.com/SpoonX/aurelia-orm). This module provides you with some useful and cool features, such as:

- Entity definitions
- Repositories
- Associations
- Validation
- Type casting
- Self-populating select element
- And more

This makes it easier to focus on your application and organize your code.

### Installation

`npm i aurelia-orm --save`

Aurelia-orm uses [aurelia-api](https://www.gitbook.com/read/book/rwoverdijk/aurelia-api) to talk to the server.

Aurelia-api is a module wrapped around aurelia-fetch-client that allows you to:

- Perform the usual CRUD
- Supply criteria for your api
- Manage more than one endpoint
- Add defaults
- Add interceptors
- And more

Please read more [about aurelia-api](http://aurelia-api.spoonx.org/Quick%20start.html)

Add the following to the `build.bundles.dependencies` section of `aurelia-project/aurelia.json` to install and configure dependencies.

```json
"dependencies": [
  // ...
  "get-prop",
  "typer",
  {
  "name": "aurelia-orm",
  "path": "../node_modules/aurelia-orm/dist/amd",
  "main": "aurelia-orm",
  "resources": [
    "component/association-select.html",
    "component/paged.html"
  ]},
  {
  "name": "aurelia-validation",
  "path": "../node_modules/aurelia-validation/dist/amd",
  "main": "index"
  },
  // ...
],
```

### Configuration

Configure [aurelia-api]() and register a new endpoint.

```ts
// Load the plugin, and set the base url.
.plugin('aurelia-api', config => {
  config
    .registerEndpoint('github', 'https://api.github.com/')
    .setDefaultEndpoint('github');
});
```

Configure with model Entities (similar to Breeze Entities)

```ts
// Import your entities
import * as entities from 'config/entities';

export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()

    // Register the plugin, and register your entities
    .plugin('aurelia-orm', builder => {
      builder.registerEntities(entities);
    });

  aurelia.start().then(a => a.setRoot());
}
```

Here's what your `config/entities.js` file might look like:

```ts
export {User} from 'entity/user';
export {Article} from 'entity/article';
export {Category} from 'entity/category';
```

### Resource
The name of a resource located at the API.

Some examples of resources are: user, category and product. Each resource has an associated Entity and Repository.

### Entity

An instance or class of type Entity that defines, and holds the data of a single resource record.

An entity defines what a record looks like. Which properties it has and which validations apply.

### Collection

An array of entities.
A collection is a simple javascript array containing one or more entity instances.

### Repository

Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
The repository handles the logic, and returns entity instances. Some ORMs put this logic on the entity and call them "models". An example of such a method is:

```ts
  repository.findBySomething(specialCriteria).then(result => doSomethingWithTheResponse)
```

In summary: all fetch methods and response mutations belong in a repository.


### Association

The relationship between two or more entities.
Some entities have a relationship with each other. For instance, entity User has many groups of type Group. The term for this is collection association.

### Entity manager

Manages Repository and Entity instances for resources.
Each resource has an entity, and a repository. They are all registered with the entity manager.

### Populate

To build up an instance with the provided data.
Entities can be populated with data. This means they get data assigned to them.

## Entities

Entities represent the schema of your resources. They can hold validation rules, associations, type hinting and more. In this chapter, we'll be looking at what an entity looks like, how you can create one and what they're responsible for.

#### Creating an Entity

```ts
import {Entity} from 'aurelia-orm';

export class Product extends Entity {
}
```

After creating your gem of an `Entity`, you'll want to register it with the `EntityManager`. Then the `Repository` responsible for your resource will populate using your new entity!

Here's how:

```ts
import {EntityManager} from 'aurelia-orm';
import {inject} from 'aurelia-dependency-injection';
import {Product} from './entity/product';

@inject(EntityManager)
class SomeClass {
  constructor(entityManager) {
    entityManager.registerEntity(Product);
  }
}
```

There's an easier way to do this, as described in the chapter Configuration.

## Repositories

Creating a custom Repository is simple.

```
import {Repository} from 'aurelia-orm';

export class Custom extends Repository {
}
```

A repository referenced by entities. Here's an example of how to do this:

```ts
import {Entity, repository} from 'aurelia-orm';
import {Custom} from 'repository/custom';

@repository(Custom)
class Product extends Entity {
}
```

And done! You can now start adding magical methods to your repository, and separate logic. Here's an example that's a bit more extended:

```ts
import {Repository} from 'aurelia-orm';

export class Custom extends Repository {
  findUserPosts(user) {
    return this.find({user: user.id});
  }

  findNewPosts() {
    return this.find({read: false});
  }
}
```

## Entity Manager

The `EntityManager` has a special job: it links entities, repositories and resources together. When you want to start working with a resource, you ask the `EntityManager` to give you what you need.

### Create new record

```ts
import {inject}        from 'aurelia-framework';
import {EntityManager} from 'aurelia-orm';

@inject(EntityManager)
export class SomeViewModel {
  constructor (entityManager) {
    let entity = entityManager.getEntity('user');

    // Note: You could also do `user.username='';`. `.setData()` is optional.
    entity.setData({username: 'Bob', password: 'Burger'}).save()
      .then(result => {
        console.log('Created a new user!');
      });
  }
}
```

### Fetch records

```ts
import {inject}        from 'aurelia-framework';
import {EntityManager} from 'aurelia-orm';

@inject(EntityManager)
export class SomeViewModel {
  constructor (entityManager) {
    let repository = entityManager.getRepository('user');

    repository.find({username: 'bob'})
      .then(result => {
        console.log('Found User!', result);
      });
  }
}
```

## Decorators

Here's an example using many of the decorators available. See [here](http://aurelia-orm.spoonx.org/decorators.html) for more details.

```ts
import {Entity, resource, repository, validation, association, type} from 'aurelia-orm';
import {ensure} from 'aurelia-validation';
import {CustomRepository} from 'repository/custom-repository';

@resource('my-endpoint')
@repository(CustomRepository)
@validation()
export class MyEntity extends Entity {
  @ensure(it => it.isNotEmpty().hasLengthBetween(3, 20))
  @type('string')
  name = null;

  // Will use string 'onetoone' as resource name.
  @association()
  oneToOne = null

  // Will use 'multiple' as resource name.
  @association('multiple')
  oneManyToMany = null
}
```

## Components

The `<association-select />` component composes a `<select />` element, of which the options have been populated from an endpoint. The selects value is the `id` property and the displayed textContents are the `name` properties of the resource. You can change the textContent by setting the `property` attribute of the `association-select`

### Basic example

Get all entries of the view-models 'userRepository' repository property and populate the select using the 'id' and the 'fullName' properties of your resource for the value and the textContent respectively. The selects current value is two-way bound to the 'data.author' property of your view-model.

```html
<association-select
  property="fullName"
  value.bind="data.author"
  repository.bind="userRepository"
></association-select>
```

### Extended example

```html
<!-- First, populate a list of categories -->
<association-select
  value.bind="data.category.id"
  repository.bind="categoryRepository"
></association-select>

<!-- Then populate a list of pages -->
<association-select
  value.bind="data.page.id"
  repository.bind="pageRepository"
></association-select>

<!-- Then populate a list of groups -->
<association-select
  value.bind="data.group.id"
  repository.bind="groupRepository"
></association-select>

<!-- And finally, populate a list of authors based on the previous selects -->
<!-- With custom selectable placeholder (value===0)                        -->
<association-select
  value.bind="data.author.id"
  error.bind="error"
  repository.bind="userRepository"
  property="username"
  association.bind="[data.page, data.group]"
  manyAssociation.bind="data.category"
  criteria.bind='{where:{age:{">":18}}}'
  selectablePlaceholder="true"
  placeholderText="- Any -"
  if.bind="!error"
></association-select>

<div class="alert alert-warning" if.bind="!!error">
  Server error:${error.statusText}
</div>
```


