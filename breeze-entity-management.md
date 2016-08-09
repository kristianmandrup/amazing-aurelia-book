# Breeze entity management

All about aurelia with Breeze.

## Installation

`npm install aurelia-breeze --save`

```js
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .plugin('aurelia-breeze');

    // ...
```

```
import breeze from 'breeze';

let query = new breeze.EntityQuery();
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

`entity-manager-factory.ts`

```js
import settings from './settings';

var entityManager;

/**
* Creates Breeze EntityManager instances.
*/
export function createEntityManager() {
  if (entityManager) {
    return Promise.resolve(copyEntityManager()); 
  }

  entityManager = new breeze.EntityManager(settings.serviceName);
  return entityManager.fetchMetadata()
    .then(() => copyEntityManager());
}

function copyEntityManager() {
  var copy = entityManager.createEmptyCopy();
  copy.entityChanged.subscribe(logChanges);
  return copy;
}

// log entity changes to the console for debugging purposes.
function logChanges(data) {
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