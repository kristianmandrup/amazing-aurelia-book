# Breeze entity management

Now let's add Breeze entity management to our application.

"Breeze is a library for rich client applications written in HTML and JavaScript. It concentrates on the challenge of building and maintaining highly responsive, data-intensive applications in which users search, add, update, and view complex data from different angles as they pursue solutions to real problems in real time."

Breeze supports:
- Rich queries
- Client-side caching
- Change tracking
- Validation
- Pluggable back-end
- Data management
- Batched saves
- Library and Tooling support

We will use the excellent [aurelia-breeze](https://github.com/jdanyow/aurelia-breeze) plugin for this.

## Install and configure

We can install the aurelia-breeze via npm:

`npm install aurelia-breeze --save`

Then we configure the plugin as usual in our `configure(aurelia)` function (usually in `main.ts`)

```js
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin('aurelia-breeze');

    // ...
```

From here on we can import `breeze` and use it.

```
import breeze from 'breeze';

let query = new breeze.EntityQuery();
```

## Breeze REST server API

We can use [breeze-rest-adapter](https://github.com/kristianmandrup/breeze-rest-adapter) which is "A dataservice adapter for BreezeJS to connect to a generic REST API".

Note: I have recently updated the adapter to use ES6 classes and modules.

### Installation

`import { ServiceAdapter, ResultsAdapter, default as register } from 'breeze-rest-adapter/es6';`

Then register the `ServiceAdapter` with breeze: `register(breeze);`

This will call `breeze.config.registerAdapter('dataService', ServiceAdapter);`

The `ServiceAdapter` has `name = 'REST'` which identifies it.

Now configure breeze to use the `REST` data service as follows:

`breeze.config.initializeAdapterInstances({dataService: 'REST'});`

The REST adapter expects the server to have a JSON REST API which delivers entities as follows:

```json
  // ...
  "orderLineItems": [
      {
          productId: 3,
          amount: 1,
          "entityAspect": {
              "entityType": "OrderLineItem",
              "entityState": "Added"
          }
      },
      // ...
```

The data service adapter will look at all the changed entities in the local cache and build an object graph (based on the defined relationships in the metadata).

This adapter makes the assumption that the backend service provides an `entityAspect` object that is a property of every entity. This is a helper property that allows the adapter to know which type of entity is being provided.

You may also look at the generic [Edmunds sample](http://breeze.github.io/doc-samples/edmunds.html) for inspiration.
Here is a [SPA example](http://breeze.github.io/doc-samples/intro-to-spa-ruby.html) using a [Ruby on Rails](http://rubyonrails.org/) Breeze adapter.

## Breeze overview

Breeze revolves around the concept of *Entities*, which are domain models that keep in sync with an underlying data store via en *Entity Manager*.

This is very similar how regular ORMs often operate such as [aurelia-orm entities](aurelia-orm.spoonx.org/entities.html).

## Entity Manager

The [EntityManager](http://breeze.github.io/doc-js/entitymanager-and-caching.html) is the gateway to the persistence service and holds a cache of entities that the application is working with, including entities that have been queried, added, updated, and marked for deletion. The `EntityManager` is the core class in Breeze, and this page discusses its primary capabilities.

The `EntityManager` serves three main functions:

- It communicates with the persistence service.
- It queries and saves entities.
- It holds entities in a local container called the entity cache.

When the client application requires data, it typically calls a method on an instance of an `EntityManager`. The `EntityManager` establishes communication channels, sets up the client’s security context, serializes and deserializes data, and regulates the application-level flow of traffic with the persistence service.

### Create an EntityManager

A query can’t execute itself. We’ll need a Breeze `EntityManager`. The EntityManager is a gateway to the persistence service which will execute the query on the backend and return query results in its response. The next few lines give us an `EntityManager`:

```js
let serviceName = 'breeze/todos'; // route to the Web Api controller
let manager = new breeze.EntityManager(serviceName);
```

The `serviceName` identifies the service end-point, the route to the Web API controller `breeze/todos`

### Todos query

We’re ready to write the first query. The `Todo` app query method is called `getAllTodos` and it looks like this:

```
function getAllTodos(includeArchived) {
    var query = breeze.EntityQuery // [1]
            .from("Todos")         // [2]
            .orderBy("CreatedAt"); // [3]

    // ... snip ...

    return manager.executeQuery(query);
};
```

- creates a new Breeze `EntityQuery` object
- aims the `query` at a method on the Web API controller named `Todos` that returns `Todo` items.
- adds an `orderBy` clause that tells the remote service to sort results by the `CreatedAt` property before sending them to the client.

The manager makes a promise. We execute the query with the EntityManager

`return manager.executeQuery(query);`

The `executeQuery` method does not return `Todos`. It can’t return `Todos`. A JavaScript client cannot freeze the browser and wait for the server to reply. The `executeQuery` method does its thing asynchronously.

It must return something and it must do so immediately. 

### Accepting a promise

A caller of the `dataservice.getAllTodos` method typically attaches both a success and failure callback to the returned promise. Here’s how the `Todo` app's ViewModel calls `getAllTodos`:

```ts
getTodos() {
  dataservice.getAllTodos(includeArchived)
    .then(querySucceeded)
    .fail(queryFailed);
}
```

### Process the query results

If the query returns from the server without error, the promise calls the ViewModel’s `querySucceeded` method, passing in a data packet with query results from the EntityManager.

Get them from the `data.results` property as the ViewModel does. In this example, each `Todo` item is pushed into an observable array bound to a list (via `repeat.for`) in the view.

```js
class Todos {
  items = []; // bound to view in repeat.for

  querySucceeded(data) {
    ...
    data.results.forEach((item) => {
        ...
        this.items.push(item);
    });
    ...
  }
}
```

And just like that, the view fills with Todos.

## Entity Manager Factory

`entity-manager-factory.ts`

```js
import settings from './settings';
import { logChanges } from './logger';

@singleton
export class EntityManagerFactory {
  /**
  + Creates Breeze EntityManager instances.
  */
  constructor() {
    return Promise.resolve(this.copy());
  }

  copy() {
    const copy = this.entityManager.createEmptyCopy();
    copy.entityChanged.subscribe(logChanges);
    return copy;
  }

  get entityManager() {
    this._entityManager = new breeze.EntityManager(settings.serviceName);
    return this._entityManager.fetchMetadata()
      .then(() => this.copy());
  };
}
```

### Logger

We introduce a `logChanges` function to keep track of entity data changes for debugging purposes.

```js
// log entity changes to the console
export function logChanges(data) {
  var message = 'Entity Changed.  Entity: ' + (data.entity ? data.entity.entityType.name + '/' + data.entity.entityAspect.getKey().toString() : '?') + ';  EntityAction: ' + data.entityAction.getName() + '; ';
  if (data.entityAction === breeze.EntityAction.PropertyChange) {
    var pcArgs = data.args;
    message += 'PropertyName: ' + (pcArgs.propertyName || 'null') + '; Old Value: ' + (pcArgs.oldValue ? pcArgs.oldValue.toString() : 'null') + '; New Value: ' + (pcArgs.newValue ? pcArgs.newValue.toString() : 'null') + ';';
  }
  if (data.entityAction === breeze.EntityAction.EntityStateChange) {
    message += 'New State: ' + data.entity.entityAspect.entityState.getName() + ';';
  }
  console.log(message);
};
```

## Entities and domain models

“Entity-ness”
A domain model object represents something significant in the application domain. A “Customer”, for example, has data properties (“Name”), relationships to other entities (“Orders”) and perhaps some business logic (“isGoldCustomer”). We bind these object members to UI controls and reason about them in application code. They are what matters most to users and other application stakeholders. They define “Customer-ness”.

The “Customer” is also an entity, a long-lived object with a permanent key. We can fetch it from a database, hold it in cache, check for changes, validate, and save it. When the developer’s attention turns to whether an object has changed or not, what its values used to be, how it is persisted, whether it has validation errors … the developer is thinking about the object’s entity nature. Breeze is responsible for the object’s entity nature, its “entity-ness”. You access an entity’s entity nature through its `entityType` and `entityAspect` properties.

### Entity type

Every Breeze entity instance has an `entityType` property that returns an EntityType object which is the metadata that describe its properties and other facts about the type.

### Entity aspect

A Breeze entity is “self-tracking”. It maintains its own entity state, and the means to change that state, in the EntityAspect object returned by its entityAspect property.

An object becomes a Breeze entity when it acquires its EntityAspect which it does when it

first enters the cache as a result of a query or import OR
is created with the EntityType.createEntity factory method OR
is explictly added or attached to an EntityManager

### Creating a new entity

Breeze creates new entity instances on two primary occasions: (1) when it “materializes” entities from query results and (2) when you ask it to create a brand new entity.

Entity materialization is largely hidden from the developer. You issue a query; you get entities back. Behind the scenes Breeze converts the stream of model object data into entities in cache. The developer only becomes aware of entity creation details when making new model objects.

### EntityManager.createEntity

The standard approach is to call the Breeze `createEntity` factory function on an `EntityManager`:

```
let newCustomer = manager.createEntity('Customer', {name: 'Acme'});
```

The first parameter, `'Customer'`, is the name of the `EntityType`
In this example, we also passed in an optional property `initializer`, a hash that sets the new customer’s name: `{name: 'Acme'}`. You may not need an initializer but it is often the easiest way to simultaneously create and initialize a new entity.

If the entity key is client-generated, then you must specify the key in the initializer or you’ll likely get an exception.

```js
let newOrderDetail = manager.createEntity('OrderDetail', {orderId: oid, productId: pid});
```

### Lookups

```js
import {createEntityManager} from './entity-manager-factory';

var customersQuery = new breeze.EntityQuery
  .from('Customers')
  .select('CustomerID, CompanyName')
  .orderBy('CompanyName');

var productsQuery = new breeze.EntityQuery
  .from('Products')
  .select('ProductID, ProductName, UnitPrice')
  .orderBy('ProductName');

/**
* Manages the application's shared lookups.
* Eagerly loading the lookups because there are only two.
*/
export class Lookups {
  customers;
  products;

  load() {
    return createEntityManager()
      .then(em => Promise.all([em.executeQuery(customersQuery), em.executeQuery(productsQuery)]))
      .then(queryResults => {
        this.customers = queryResults[0].results;
        this.products = queryResults[1].results;
      })
  }
}
```

### Entity View Model

```js
export class EntityViewModel {
  service;
  entityManager;
  entity;

  constructor(service) {
    this.service = service;
  }

  activate(info) {
    var promise;

    // load or create the entity.
    if (info.id === 'new') {
      promise = this.service.createNew();
    } else {
      promise = this.service.loadExisting(info.id);
    }

    return promise.then(result => {
      this.entityManager = result.entityManager;
      this.entity = result.entity;
    });
  }

  canDeactivate() {
    // permit navigating away from new entities.
    if (this.entity.entityAspect.entityState.isAdded()) {
      Materialize.toast('Add-new cancelled.', 2000);
      return true;
    }

    // disallow navigating away from modified entities.
    if (this.hasChanges) {
      // throttle the amount of toast we pop.
      if (!this._lastPop || +new Date() - this._lastPop > 2000) {
        this._lastPop = +new Date();
        Materialize.toast('Navigation cancelled.  Save your changes!', 2000);
      }
      return false;
    }

    // permit navigating away from unmodified entities.
    return true;
  }

  get hasChanges() {
    return this.entityManager.hasChanges();
  }

  save() {
    // fake save...
    this.entityManager.acceptChanges();
    Materialize.toast('Changes saved.', 2000)
  }

  revert() {
    this.entityManager.rejectChanges();
    Materialize.toast('Changes reverted.', 2000)

    // workaround Materialize datepicker binding timezone issue.
    if (this.hasChanges) {
      this.entityManager.rejectChanges();
    }
  }
}
```

`settings.js`

```js
export default {
  serviceName: "http://sampleservice.breezejs.com/api/northwind",
  pageSize: 100,
};
```

`list-view-model`

```js
import settings from './settings';

export class ListViewModel {
  router;
  route;
  service;
  entities = [];
  pageSize = settings.pageSize;
  pageCount = 0;
  pageIndex = 0;
  isLoading = false;

  constructor(route, router, service) {
    this.route = route;
    this.router = router;
    this.service = service;
  }

  activate() {
    this.load();
  }

  load() {
    this.isLoading = true;
    this.service.getPage(this.pageIndex)
      .then(result => {
        this.entities = result.entities;
        this.pageCount = result.pageCount;
        this.isLoading = false;
      });
  }

  setPage(index) {
    this.pageIndex = index;
    this.load();
  }

  open(id) {
    this.router.navigate(this.route + '/' + id);
  }
}
```

### Orders

`order-service.ts`

```js
import breeze from 'breeze';
import settings from '../settings';
import {createEntityManager} from '../entity-manager-factory';

export class OrderService {
  getPage(pageIndex) {
    var query = new breeze.EntityQuery
      .from('Orders')
      .select('OrderID, Customer.CompanyName, Employee.FirstName, Employee.LastName, OrderDate, Freight')
      .orderByDesc('OrderDate')
      .skip(pageIndex * settings.pageSize)
      .take(settings.pageSize)
      .inlineCount();

    return createEntityManager()
      .then(em => em.executeQuery(query))
      .then(queryResult => {
        return {
          entities: queryResult.results,
          pageCount: 8, //Math.ceil(queryResult.inlineCount / this.pageSize);
        };
      });
  }

  loadExisting(id) {
    var orderQuery = new breeze.EntityQuery().from('Orders').where('OrderID', '==', id),
        detailQuery = new breeze.EntityQuery().from('OrderDetails').where('OrderID', '==', id);

    return createEntityManager()
      .then(em => Promise.all([em.executeQuery(orderQuery), em.executeQuery(detailQuery)]))
      .then(values => {
        var queryResult = values[0];
        return {
          entity: queryResult.results[0],
          entityManager: queryResult.entityManager
        };
      });
  }

  createNew() {
    return createEntityManager()
      .then(em => {
        return {
          entity: em.createEntity('Order'),
          entityManager: em
        };
      });
  }
}
```

#### Order list

`order-list.js`

```
import {ListViewModel} from '../list-view-model';
import {inject, singleton} from 'aurelia-dependency-injection';
import {AppRouter} from 'aurelia-router';
import {OrderService} from './order-service';

@inject(AppRouter, OrderService)
@singleton()
export class OrderList extends ListViewModel {
  constructor(router, service) {
    super('orders', router, service)
  }
}
```

`order-list.html`

```html
<template>
  <div class="container">

    <h3 class="left">Orders</h3>
    <a href="#orders/new" class="waves-effect waves-light btn right header-button">New</a>

    <table class="bordered hoverable table-responsive">
      <thead>
        <tr>
          <th>Order #</th>
          <th>Date</th>
          <th>Freight</th>
          <th>Customer</th>
          <th>Employee</th>
        </tr>
      </thead>
      <tbody class="cursor-pointer">
        <tr repeat.for="entity of entities" click.delegate="$parent.open(entity.OrderID)">
          <td>${entity.OrderID}</td>
          <td>${entity.OrderDate | dateFormat:'M/D/YYYY'}</td>
          <td>${entity.Freight | numberFormat:'$0,0.00'}</td>
          <td>${entity.Customer_CompanyName}</td>
          <td>${entity.Employee_FirstName + ' ' + entity.Employee_LastName}</td>
        </tr>
      </tbody>
    </table>

    <div class="progress" show.bind="isLoading">
      <div class="indeterminate"></div>
    </div>

    <pager page-count.bind="pageCount"
           page-index.bind="pageIndex"
           set-page.call="setPage($event)">
    </pager>

  </div>
</template>
```

#### Order

`order.js`

```js
import {EntityViewModel} from '../entity-view-model';
import {inject} from 'aurelia-dependency-injection';
import {OrderService} from './order-service';
import {Lookups} from '../lookups';

@inject(OrderService, Lookups)
export class Order extends EntityViewModel {
  customers;

  constructor(service, lookups) {
    super(service);
    this.customers = lookups.customers;
  }

  get title() {
    if (this.entity.OrderID <= 0) {
      return 'New Order';
    }
    return `Order #${this.entity.OrderID}`;
  }
}
```

`order.html`

```html
<template>
  <div class="container">
    <form submit.delegate="save()" novalidate>
      <!-- header -->
      <compose view="../entity-header.html"></compose>

      <!-- order fields -->
      <div class="row">
        <div class="input-field col s12 m6">
          <select value.bind="entity.CustomerID" materialize="select">
            <option value="">Select a customer</option>
            <option repeat.for="customer of customers" value.bind="customer.CustomerID">${customer.CompanyName}</option>
          </select>
          <label for="Customer">Customer</label>
        </div>
        <div class="input-field col s12 m3">
          <input id="OrderDate" type="text" materialize="datepicker" value.bind="entity.OrderDate | dateFormat:'M/D/YYYY'">
          <label for="OrderDate" materialize>Order Date</label>
        </div>
        <div class="input-field col s12 m3">
          <input id="ShippedDate" type="text" materialize="datepicker" value.bind="entity.ShippedDate | dateFormat:'M/D/YYYY'">
          <label for="ShippedDate" materialize>Shipped Date</label>
        </div>
      </div>

      <!-- order shipping fields -->
      <div class="row">
        <h4>Shipping</h4>
        <div class="input-field col s12 m5">
          <input id="ShipName" type="text" value.bind="entity.ShipName">
          <label for="ShipName" materialize>Name</label>
        </div>
        <div class="input-field col s12 m5">
          <input id="ShipCountry" type="text" value.bind="entity.ShipCountry">
          <label for="ShipCountry" materialize>Country</label>
        </div>
        <div class="input-field col s12 m2">
          <input id="Freight" type="number" value.bind="entity.Freight">
          <label for="Freight" materialize>Freight $</label>
        </div>
      </div>
      <div class="row">
        <div class="input-field col s12 m4">
          <input id="ShipAddress" type="text" value.bind="entity.ShipAddress">
          <label for="ShipAddress" materialize>Address</label>
        </div>
        <div class="input-field col s12 m4">
          <input id="ShipCity" type="text" value.bind="entity.ShipCity">
          <label for="ShipCity" materialize>City</label>
        </div>
        <div class="input-field col s12 m2">
          <input id="ShipRegion" type="text" value.bind="entity.ShipRegion">
          <label for="ShipRegion" materialize>Region</label>
        </div>
        <div class="input-field col s12 m2">
          <input id="ShipPostalCode" type="tel" value.bind="entity.ShipPostalCode">
          <label for="ShipPostalCode" materialize>Postal Code</label>
        </div>
      </div>

      <!-- order details grid -->
      <compose model.bind="entity" view-model="orders/order-details"></compose>
    </form>
  </div>
</template>
```

#### Order details

`order-details.js`

```js
import {inject} from 'aurelia-dependency-injection';
import {Lookups} from '../lookups';

@inject(Lookups)
export class OrderDetails {
  order;
  allProducts;
  products;
  productsIndex = {};
  detail = null;

  constructor(lookups) {
    this.products = this.allProducts = lookups.products;
    this.products.forEach(p => this.productsIndex[p.ProductID] = p);
  }

  activate(order) {
    this.order = order;
  }

  addDetail() {
    this.detail = this.order.entityAspect.entityManager
      .createEntity('OrderDetail', { OrderID: this.order.OrderID, Quantity: 1 });
    this.openDetail();
  }

  editDetail(detail) {
    this.detail = detail;
    this.openDetail();
  }

  removeDetail(detail) {
    detail.entityAspect.setDeleted();
  }

  detailPropertyChanged(args) {
    // autofill UnitPrice based on selected Product
    if (args.propertyName !== 'ProductID') {
      return;
    }
    var product = this.productsIndex[args.newValue];
    this.detail.UnitPrice = product ? product.UnitPrice : null;
  }

  openDetail() {
    // subscribe to Product change to autofill UnitPrice
    this._subscription = this.detail.entityAspect.propertyChanged.subscribe(args => this.detailPropertyChanged(args));

    // prevent selecting the same product twice.
    this.products = this.allProducts
      .filter(p => this.order.OrderDetails.filter(d => d.ProductID === p.ProductID && d !== this.detail).length === 0);

    // open the modal.
    $('#detail').openModal();
  }

  closeDetail() {
    this.detail.entityAspect.propertyChanged.unsubscribe(this._subscription);
    $('#detail').closeModal();
  }

  calculateDetailCost(detail) {
    return detail.Quantity * detail.UnitPrice * (1 - detail.Discount);
  }

  get detailTotal() {
    return this.order.OrderDetails.map(this.calculateDetailCost).reduce((a, b) => a + b, 0);
  }
}

/**
* Value converter for the "discount" field to allow users to enter discounts as whole numbers
* even though they are stored as decimals.
*/
export class DiscountValueConverter {
  toView(value) {
    return value === null ? null : Math.floor(value * 100);
  }

  fromView(value) {
    value = +value;

    if (isNaN(value) || value >= 100) {
      return 0;
    }

    return (value / 100).toFixed(2);
  }
}
```

`order-details.html`

```html
<template>
  <!-- header -->
  <div class="row">
    <h4 class="left">Items</h4>
    <button type="button" click.delegate="addDetail()" class="right waves-effect waves-light btn header-button">
      Add
    </button>
  </div>

  <!-- grid -->
  <table class="bordered hoverable table-responsive">
    <thead>
      <tr>
        <th></th>
        <th>Product</th>
        <th>Unit Price</th>
        <th>Quantity</th>
        <th>Discount</th>
        <th>Total</th>
      </tr>
    </thead>
    <tbody class="cursor-pointer">
      <tr repeat.for="detail of order.OrderDetails" click.delegate="$parent.editDetail(detail)">
        <td click.delegate="$parent.removeDetail(detail)">
          <i class="mdi-navigation-close red-text darken-4" />
        </td>
        <td>${$parent.productsIndex[detail.ProductID].ProductName}</td>
        <td>${detail.UnitPrice | numberFormat:'$0,0.00'}</td>
        <td>${detail.Quantity | numberFormat:'0,0'}</td>
        <td>${detail.Discount | numberFormat:'0%'}</td>
        <td>${detail.Quantity * detail.UnitPrice * (1 - detail.Discount) | numberFormat:'$0,0.00'}</td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td colspan="4"></td>
        <td><strong class="right-align">Subtotal:</strong></td>
        <td>${detailTotal | numberFormat:'$0,0.00'}</td>
      </tr>
      <tr>
        <td colspan="4"></td>
        <td><strong class="right-align">Freight:</strong></td>
        <td>${order.Freight | numberFormat:'$0,0.00'}</td>
      </tr>
      <tr>
        <td colspan="4"></td>
        <td><strong class="right-align">Total:</strong></td>
        <td><strong>${detailTotal + order.Freight | numberFormat:'$0,0.00'}</strong></td>
      </tr>
    <tfoot>
  </table>

  <!-- modal -->
  <div id="detail" class="modal">
    <div class="modal-content">
      <h4>Order Detail</h4>
      <div class="row">
        <div class="input-field col s8">
          <select value.bind="detail.ProductID" id="Product" materialize="select">
            <option model.bind="0">Select a product</option>
            <option repeat.for="product of products" model.bind="product.ProductID">${product.ProductName}</option>
          </select>
          <label for="Product">Product</label>
        </div>
        <div class="col s4">
          <br/>
          Unit Price: ${detail.UnitPrice | numberFormat:'$0,0.00'}
        </div>
      </div>
      <div class="row">
        <div class="input-field col s4">
          <input id="Quantity" type="number" value.bind="detail.Quantity">
          <label for="Quantity" class="active">Quantity</label>
        </div>
        <div class="input-field col s4">
          <input id="Discount" type="number" value.bind="detail.Discount | discount">
          <label for="Discount" class="active">Discount %</label>
        </div>
        <div class="col s4">
          <br/>
          Total: ${detail.Quantity * detail.UnitPrice * (1 - detail.Discount) | numberFormat:'$0,0.00'}
        </div>
      </div>
    </div>
    <div class="modal-footer">
      <a href="#" click.delegate="closeDetail" class="modal-action modal-close waves-effect waves-green btn-flat">OK</a>
    </div>
  </div>
</template>
```