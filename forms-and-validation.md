# Forms and Validation

Using Aurelia forms and bindings, [aurelia-form](https://github.com/SpoonX/aurelia-form) and [aurelia-validation](https://github.com/aurelia/validation) plugins.

At the end also an example of Using [Breeze with Validations](https://www.danyow.net/form-validation-with-breeze-and-aurelia/)

## Simple forms

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
`submit.delegate`

## Validation

From this [alpha validation blog post](https://www.danyow.net/aurelia-validation-alpha/)

Validation consists of two separate libraries with robust set of standard behaviors and a simpler, more flexible API surface for easier customization:

`aurelia-validation` - a generic validation library that provides a `ValidationController`, a validate binding behavior, a `Validator` interface and more.

`aurelia-validatejs` - a `validatejs` powered implementation of the `Validator` interface along with fluent and decorator based APIs for defining rules for your components and data.

### Create registration form

`registration-form.js`

```ts
export class RegistrationForm {  
  firstName = '';
  lastName = '';
  email = '';

  submit() {
    // todo: call server...
  }
}
```

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

### Install validation plugin

`$ npm install aurelia-validation`

## Add validation to RegistrationForm

```ts
import {inject, NewInstance} from 'aurelia-dependency-injection';
import {ValidationController} from 'aurelia-validation';

@inject(NewInstance.of(ValidationController))
export class RegistrationForm {
  constructor(controller) {
    this.controller = controller;
  }
  // ...
```

The `ValidationController` manages a set of bindings and a set of renderers, controlling when to tell the renderers to render or unrender validation errors.

### Configure the ValidationController:

The default "trigger" that tells the validation controller to validate bindings is the DOM `blur` event. All in all there are three standard "validation triggers" to choose from:

`blur`: Validate the binding when the binding's target element fires a DOM "blur" event.

`change`: Validate the binding when it updates the model due to a change in the view.

`manual`: Manual validation. Use the controller's validate() and  reset() methods to validate all bindings.

To configure the controller's validation trigger, import the `validateTrigger` enum: `import {validateTrigger} from 'aurelia-validation';` and assign the controller's `validateTrigger` property:

`controller.validateTrigger = validateTrigger.manual;`


