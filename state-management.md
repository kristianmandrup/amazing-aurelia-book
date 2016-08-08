# State management

Using Breeze, ORMs, RethinkDB and other state management solutions.

- [aurelia-breeze](https://github.com/jdanyow/aurelia-breeze)
- [aurelia-orm](https://github.com/SpoonX/aurelia-orm)

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


