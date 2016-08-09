# Modal wizard

[example](https://github.com/cmichaelgraham/aurelia-typescript/tree/master/pwkad-aurelia-samples)

- [aurelia-modal](https://github.com/PWKad/aurelia-modal)
- [aurelia-dialog](https://github.com/aurelia/dialog)

[aurelia dialog tutorial](http://www.tutorialspoint.com/aurelia/aurelia_dialog.htm)

`npm install aurelia-dialog --save`

Typings

`typings install github:aurelia/dialog --save`

```js
export function configure(aurelia) {
  aurelia.use
    // ...
    .plugin('aurelia-dialog');
```

You can use the dialog service to open a prompt -

```js
import {DialogService} from 'aurelia-dialog';
import {Prompt} from './prompt';
export class Welcome {
  static inject = [DialogService];
  constructor(dialogService) {
    this.dialogService = dialogService;
  }
  submit(){
    this.dialogService.open({ viewModel: Prompt, model: 'Good or Bad?'}).then(response => {
      if (!response.wasCancelled) {
        console.log('good');
      } else {
        console.log('bad');
      }
      console.log(response.output);
    });
  }
}
```

This will open a prompt and return a promise that resolves when closed. If the user clicks out, clicks cancel, or clicks the 'x' in the top right it will still resolve the promise but will have a property on the response wasCancelled to allow the developer to handle cancelled dialogs.

There is also an output property that gets returned with the outcome of the user action if one was taken.

You can create your own view / view-model and use the dialog service to call it from your app's view-model -

```js
import {EditPerson} from './edit-person';
import {DialogService} from 'aurelia-dialog';
export class Welcome {
  static inject = [DialogService];
  constructor(dialogService) {
    this.dialogService = dialogService;
  }
  person = { firstName: 'Wade', middleName: 'Owen', lastName: 'Watts' };
  submit(){
    this.dialogService.open({ viewModel: EditPerson, model: this.person}).then(response => {
      if (!response.wasCancelled) {
        console.log('good - ', response.output);
      } else {
        console.log('bad');
      }
      console.log(response.output);
    });
  }
}
```

This will open a dialog and control it the same way as the prompt. The important thing to keep in mind is you need to follow the same method of utilizing a DialogController in your EditPerson view-model as well as accepting the model in your activate method -

```js
import {DialogController} from 'aurelia-dialog';

export class EditPerson {
  static inject = [DialogController];
  person = { firstName: '' };
  constructor(controller){
    this.controller = controller;
  }
  activate(person){
    this.person = person;
  }
}
```

and the corresponding view -

```html
<template>
  <ai-dialog>
    <ai-dialog-body>
      <h2>Edit first name</h2>
      <input value.bind="person.firstName" />
    </ai-dialog-body>

    <ai-dialog-footer>
      <button click.trigger="controller.cancel()">Cancel</button>
      <button click.trigger="controller.ok(person)">Ok</button>
    </ai-dialog-footer>
  </ai-dialog>
</template>
```

### attach-focus custom attribute

The modal exposes an `attach-focus` custom attribute that allows focusing in on an element in the modal when it is loaded. You can use this to focus a button, input, etc... Example usage -

```html
<template>
  <ai-dialog>
    <ai-dialog-body>
      <h2>Edit first name</h2>
      <input attach-focus="true" value.bind="person.firstName" />
    </ai-dialog-body>
  </ai-dialog>
</template>
```

You can also bind the value of the attach-focus attribute if you want to alter which element will be focused based on a view model property.

```
<input attach-focus.bind="isNewPerson" value.bind="person.email" />
<input attach-focus.bind="!isNewPerson" value.bind="person.firstName" />
```

### Global Settings

You can specify global settings as well for all dialogs to use when installing the plugin via the configure method. If providing a custom configuration, you must call the useDefaults() method to apply the base configuration.

```js
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging()
    .plugin('aurelia-dialog', config => {
      config.useDefaults();
      config.settings.lock = true;
      config.settings.centerHorizontalOnly = false;
      config.settings.startingZIndex = 5;
    });

  aurelia.start().then(a => a.setRoot());
}
```

## Creating a reusable aurelia dialog

From [creating a reusable aurelia dialog](http://lukeautry.com/Blog/creating-a-reusable-aurelia-dialog)

`forgotPassword.js`

```ts
import {autoinject} from "aurelia-framework";
import {DialogController} from "aurelia-dialog";

@autoinject
export class ForgotPassword {
  protected Email = "";
  constructor(private DialogController: DialogController) { }

  protected Submit() {
    // talk to the server
  }
}
```

`forgotPassword.html`

```html
<template>
  <ai-dialog>
    <ai-dialog-header>
      <h2>Forgot Password</h2>
      <button click.trigger="DialogController.close()">x</button>
    </ai-dialog-header>
    <ai-dialog-body>
      <input value.bind="Email" required />
    </ai-dialog-body>
    <ai-dialog-footer>
      <button click.trigger="Submit()">Ok</button>
    </ai-dialog-footer>
  </ai-dialog>
</template>
```

`login.ts`

```ts
import {autoinject} from "aurelia-framework";
import {DialogService} from "aurelia-dialog";
import {ForgotPassword} from "./forgotPassword"
@autoinject
export class Login {
  constructor(private DialogService: DialogService) { }

  protected OpenForgotPasswordDialog() {
    this.DialogService.open({ viewModel: ForgotPassword });
  }
}
```

*Problem* every instance of a dialog we need in the app is going to require a fair amount of boilerplate code. In practice, 95% of DietBuilder's dialog boxes are going to follow the same basic script: a "title bar" with some text and a close button, a body (this is going to be different for basically every dialog), and a footer consisting of a single button that triggers an async request to the server.

What I really want here is a dialog "shell" that can be used for most or all of my dialogs. I don't want to have to create the entire dialog markup structure above every time. I want to be able to plug in a few values and be on my way.

The solution I settled on was to create an abstract "Dialog" class that contains the building blocks that all dialogs share.

`dialog.ts`

```ts
import {DialogController} from "aurelia-dialog";
import {Origin} from "aurelia-metadata";
import {useView} from "aurelia-framework";

@useView("app/common/components/dialog/dialog.html")
export abstract class Dialog {
  protected ViewPath: string;

  constructor(protected DialogController: DialogController) {
    this.SetViewPath();
  }

  protected abstract Title(): string;
  protected abstract SubmitText(): string;
  protected abstract Submit(): Promise<any>;

  protected AttemptSubmission() {
    this.IsProcessing = true;
    this.Submit();
  }

  protected Close() {
    this.DialogController.cancel();
  }

  private SetViewPath() {
    const moduleId: string = (Origin.get(this.constructor) as any).moduleId;
    const relevantPathIndex = moduleId.indexOf("app");
    this.ViewPath = moduleId.slice(relevantPathIndex, moduleId.length).replace(".js", ".html");
  }
}
```

`dialog.html`

```html
<template>
  <ai-dialog>
    <ai-dialog-header>
      <h1>${Title()}</h1>
      <i class="fa fa-close" click.trigger="Close()"></i>
    </ai-dialog-header>
    <form class="form-group" submit.delegate="AttemptSubmission()">
    <ai-dialog-body>
      <compose view="${ViewPath}"></compose>
    </ai-dialog-body>
    <ai-dialog-footer>
    <button class="attention" type="submit" innerHTML.one-way="SubmitText()"></button>
      </ai-dialog-footer>
    </form>
  </ai-dialog>
</template>
```

So what's going on here? The abstract base class Dialog assumes that every dialog needs a few things: a "Title" (the text that appears at the top of the dialog), submit button text (e.g. "Submit"), and it needs to fire off an async submit operation. The implementations/values for these items are going to be different for every dialog.

There are several things that will always be the same for every dialog, though. Every dialog needs a "close" button, and this will always function in basically the same way. Nothing mind blowing here.

The tricky part is this: every dialog is going to use the same outer markup, e.g. what you see in dialog.html above. Every dialog is going to have its own body, though. That's where "ViewPath" comes in.

In dialog.ts, you'll notice a couple of things. #1: the class is decorated with a "useView" declaration, pointing to the path of dialog.html. This means that any dialog that inherits from Dialog will load up the view of dialog.html instead of going by convention. #2: we have to calculate the path of the "inner" view which will consitute the "body" of the dialog. Here's how that looks in practice:

`forgotPassword.ts`

```ts
import {autoinject} from "aurelia-dependency-injection";
import {Dialog} from "app/common/components/dialog/dialog";
import {DialogController} from "aurelia-dialog";

@autoinject
export class ForgotPassword extends Dialog {
  protected Email = "";

  constructor(dialogController: DialogController) {
    super(dialogController);
  }

  protected Title() {
    return "Send Reset Password Email";
  }

  protected SubmitText() {
    return "Send Email";
  }

  protected Submit() {
    // talk to the server
  }
}
```

`forgotPassword.html`

```html
<template>
  <input type="email" value.bind="Email" required>
</template>
```

The end result is that I really don't have to write much code to have create a new type of dialog, and the code I do have to write is directly applicable to the functionality of the new dialog type.

This comes with the obvious benefits when you're not duplicating code.

