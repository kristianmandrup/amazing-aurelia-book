# Forms and Validation

Using Aurelia forms and bindings, [aurelia-form](https://github.com/SpoonX/aurelia-form) and [aurelia-validation](https://github.com/aurelia/validation) plugins.

At the end also an example of Using [Breeze with Validations](https://www.danyow.net/form-validation-with-breeze-and-aurelia/)

## Text input

Let's make a simple signup form with email and password entries.

`signup.ts`

```ts
import User from './models/user';

export class Signup {

   email = '';
   password = '';

   signup() {
      var myUser = new User({ email: this.email, password: this.password });
      myUser.save();
   };
}
```

The `signup` method in this example uses a hypothetical `User` model class to save the user data.

`signup.html`

```html
<template>
   <form role="form" submit.delegate="signup()">
      <label for="email">Email</label>
      <input type="text" value.bind="email" placeholder="Email">
      <label for="password">Password</label>
      <input type="password" value.bind="password" placeholder="Password">
      <button type="submit">Signup</button>
   </form>
</template>
```

For `<input>` elements, `value.bind` creates a two-way binding by default.
We use `submit.delegate` to delegate form submission to the `signup` function of the VM.

### Text area

```ts
export class SomeViewModel {
    textAreaValue = '';
}
```


```html
<template>
    <form role="form">
        <textarea value.bind="textAreaValue"></textarea>
    </form>
</template>
```

### Content editable

Any non-input field can be made contenteditable via `contenteditable="true"`

```html
<template>
    <form role="form">
        <div textcontent.bind="textContentValue" contenteditable="true"></div>
    </form>
</template>
```

### Create registration form

Now let's create a simple registration form:

- `firstName`
- `lastName`
- `email`

`registration-form.js`

```ts
import { computedFrom } from 'aurelia-framework';

export class RegistrationForm {  
  firstName = '';
  lastName = '';
  email = '';

  @computedFrom('firstName', 'lastName')
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  submit() {
    // todo: call server...
  }
}
```

We use the `@computedFrom('firstName', 'lastName')` decorator on the `fullName` getter method, to recompute the `fullName` whenever `firstName` or `lastName` changes.

`registration-form.html`

```html
<template>
  <form submit.delegate="submit()">
    <div class="form-group">
      <label class="control-label" for="first">First Name</label>
      <input type="text" class="form-control" id="first" placeholder="First Name" value.bind="firstName">
    </div>
    <div class="form-group">
      <label class="control-label" for="last">Last Name</label>
      <input type="text" class="form-control" id="last" placeholder="Last Name"
             value.bind="lastName">
    </div>
    <div class="form-group">
      <label class="control-label" for="email">Email</label>
      <input type="text" class="form-control" id="email" placeholder="Email"
             value.bind="email">
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
  </form>
</template>
```

## Single selection

```ts
export class MyForm  {
   constructor() {
      this.isChecked = false;
   }

   submit(){
      console.log("isChecked: " + this.isChecked);
   }
}
```

Form with a checkbox, where `checked` is bound to `isChecked` property on the VM via `checked.bind="isChecked"`

```html
<template>
  <form role="form" submit.delegate="submit()">
    <label for="checkbox">Checkbox</label>
    <input type="checkbox" id="checkbox" checked.bind="isChecked"><br/>
    <button type="submit">SUBMIT</button>
  </form>
</template>
```

For single select, we need to change the `selected` property to a string or number such as `selectedVal = 3;`

```html
<template>
    <form role="form">
        <select value.bind="selectedVal">
            <option repeat.for="option of someOptions" model.bind="option">${option.name}</option>
        </select>
    </form>
</template>
```

### Radio buttons

```html
<template>
    <form role="form">
        <label repeat.for="option of someOptions">
            <input type="radio" model.bind="option" name="testOptions" checked.bind="$parent.selectedOptions">${option.name}
        </label>
    </form>
</template>
```

## Multi selection

Multi selection is where the user can select multiple values, typically either via a group of checkboxes or a drop down multi selector.

### Checkboxes

For a view model `selectedOptions` and a list of Objects with `value` and `name` we have several display options.

```ts
export class SomeViewModel {
    selectedOptions = [];

    someOptions = [
        {value: 1, name: 'Test 1'},
        {value: 2, name: 'Test 2'},
        {value: 3, name: 'Test 3'}
    ];
}
```

Group of check boxes via `repeat.for` on `someOptions` Array.

```html
<template>
    <form role="form">
        <label repeat.for="option of someOptions">
            <input type="checkbox" model.bind="option" checked.bind="$parent.selectedOptions">${option.name}
        </label>
    </form>
</template>
```

Multiple select drop down via `repeat.for` on `someOptions` Array for `<option>` elements.

```html
<template>
    <form role="form">
        <select value.bind="selectedValues" multiple>
            <option repeat.for="option of someOptions" model.bind="option">${option.name}</option>
        </select>
    </form>
</template>
```

## Using aurelia-form plugin

What Aurelia form does, is leverage Aurelia's strengths, and create a standardized way of describing forms simply using objects and arrays.
Some of the features this plugin provides include:

- multi css-framework support
- [aurelia-orm](http://aurelia-orm.spoonx.org/) support (forms based on entity definition, no need to make a schema)
- validation
- concise way of describing forms using schemas
- ...

### Installation

`npm install aurelia-form`

Then add the plugin in your plugins configuration, such as in `main.ts`

`.plugin('aurelia-form')`

### Usage

`aurelia-form` requires a schema definition of the form as follows:

```ts
export class Signup {
  constructor() {
    let schema = [{
      key: 'email',
      type: 'string'
    },{
      key: 'password',
      type: 'string'
    }];

    let credentials = {
      email: '',
      password: ''
    };

    let submit = (credentials) => {
      console.log(credentials);
    };

    this.loginForm = {schema, credentials, submit};
  }
}
```

`signup.html`

```html
<template>
  <schema-form
    submit.delegate="loginForm.submit(loginForm.credentials)"
    schema.bind="this.loginForm.schema"
    model.bind="this.loginForm.credentials"
  ></schema-form>
<template>
```

Aurelia-form leverages [aurelia-view-manager](aurelia-view-manager.spoonx.org)

## Models

At the highest level is the Form model

```
let user = {
  name: 'John'
  email: 'john@jmail.tron'
};
```

The element defines presentation details for the form field of the specific model attribute targeted by `key`.

```ts
let nameElement = {
  // Hide the label when is false.
  label: false,
  type: 'string',

  // key what the property to alter in the User model
  key: 'name'
}
```

You can add an `attributes` object to specify specific attribute values of the field.

```ts
let nameElement = {
  // ...
  attributes: {
    readonly: true
  }
}
```

A schema is a collection of elements:

```
let username = {
  type:'text',
  key: 'username'
};

let password = {
  type:'password',
  key: 'password'
};

this.credentialsSchema = [
  username,
  password
];
```

Having a schema is not enough. We also need a model to populate it. That is where the key property in our schema definitions comes into play.

### Types

- text
- button
- color
- date
- datetime
- 'datetime-local'
- email
- month
- number
- password
- range
- search
- tel
- time
- urlweek

### Collections

Collections are used when having a one to many relationship between "things". An example would be a `shoppingCart` with `products`.


```
/* model */

let shoppingCart = {
  name: 'Mario',
  products: [
    {name: 'cheesy pizza' 10, amount: 2},
    {name: 'meat pizza' 10, amount: 2},
    {name: 'hot-dog crust pizza' 10, amount: 2}
  ]
}
```

```ts
let productSchema = [
  nameElement,
  amountElement
];

let productElement = {
  type: 'collection',
  key: 'products'
  schema: productSchema
};

let shoppingCartSchema = [
  nameElement,
  productElement
];
```

### fieldset nested

TODO

### checkboxes

TODO

### radios

TODO

### select

TODO

### actions buttons

TODO