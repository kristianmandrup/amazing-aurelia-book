# Modal wizard

[example](https://github.com/cmichaelgraham/aurelia-typescript/tree/master/pwkad-aurelia-samples)

- [aurelia-modal](https://github.com/PWKad/aurelia-modal)
- [aurelia-dialog](https://github.com/aurelia/dialog)

[aurelia dialog tutorial](http://www.tutorialspoint.com/aurelia/aurelia_dialog.htm)

## Content selectors

### The Problem
I want to have modals in my application but I don't want to have to include a ton of html in my base view and I also want to toggle their visibility separately.

I also want to be able to skin them differently depending on content shown.

### The Solution
With Aurelia we can take advantage of Content Selectors to solve both of these problems.

Aurelia uses concepts from the Shadow DOM to separate the content from the presentation.

We will build a single, shared modal that dynamically changes its content as needed. This will give us a single visibility property to manage. We will use compose and custom elements to swap out the content.

We will also use content selectors to allow re-skinning the modal more easily and further abstracting style specific logic out of our view so it is easier to adjust styling.

### Starting with a fresh skeleton
As always let's get an Aurelia app running.

Begin by download the navigation skeleton. Then we need to do some basic setup at the root of that project directory.

```
$ npm install
$ jspm install -y

$ gulp watch
```

Now we have our app being served @ localhost:9000

Adding our modal to app.html
In our src/app.html file, let's add the basics parts of a modal -

```html
<modal>  
  <modal-header></modal-header>
  <modal-body></modal-body>
  <modal-footer></modal-footer>
</modal>  
```

Here we are using a modal custom element and and setting the header, body, and footer of it. So far we haven't added any bindable properties so this just renders the html templates for now. We will add those bindings in a bit.

Adding a modal template
First we need our base modal template to use. We could just put all of the html in our app.html but by abstracting it out into a modal template it is more re-usable and can be easily swapped out if we found a more complex need.

`View - modal.html`

```html
<template>  
  <div class="modal fade">
    <div class="modal-dialog">
      <div class="modal-content">
        <content select="modal-header"></content>
        <content select="modal-body"></content>
        <content select="modal-footer"></content>
      </div>
    </div>
  </div>
</template>  
```

Here we are using Bootstraps default modal implementation. Notice the content selectors in the modal-content div? That is how we tell our app where to render the content we defined in the app.html.

`View-model - modal.js`

```js
import {inject} from 'aurelia-framework';  
import $ from 'jquery';

@inject(Element)
export class Modal {  
  constructor(element) {
    this.element = element;
  }
  attached(){
    $(this.element).find('.modal').modal();
  }
}
```


You can see in our `attached` callback we are finding the modal class as a descendant of our element and calling the `modal()` function bootstrap provides us.

### Adding header and footer templates
Now let's add the html for the header and also give it a settable title -

`View - modal-header.html`

```html
<template>  
  <div class="modal-header">
    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
      <span aria-hidden="true">&times;</span>
    </button>
    <h4 class="modal-title">${title}</h4>
  </div>
</template>  
```

Again, this is just a base bootstrap modal header. The only adjustment we've made is to add string interpolation to render the title in our h4.

`View-model - modal-header.js`

```js
import {bindable} from 'aurelia-framework';

export class Header {  
  @bindable title = '';
}
```

We create the bindable property title with a default value of '' or string empty. Also note we are using ES7 property initializers to create the property. If we weren't wanting to use ES7 features we could always write that same code like this -

```js
@bindable('title')
export class ModalHeader{}  
```

`View - modal-footer.html`

```html
<template>  
  <div class="modal-footer">
    <button type="button" class="btn btn-default" repeat.for="button of buttons">${button}</button>
  </div>
</template>  
```

Base bootstrap footer, but we've changed the button so we can display more when they get passed in.

`View-model - modal-footer.js`

```js
import {bindable} from 'aurelia-framework';

export class ModalFooter {  
  @bindable buttons = [];
}
```

Again we create a buttons property that is bindable and is an array which will contain the labels for our buttons.

### Adding a body
We need our modal-body template that will actually be in charge of which template gets shown in the main content area.

`View - modal-body.html`

```
<template>  
  <div class="modal-body">
    <compose view-model.bind="content"></compose>
  </div>
</template>  
```

Base bootstrap body but we've added a compose binding in to render whatever gets set to the content.

`View-model - modal-body.js`

```
import {bindable} from 'aurelia-framework';

export class ModalBody {  
  @bindable content;
}
```

Our content will also be a string. This is the property that tells the view what to compose as the body.

Adding a template to render in the body
We need to have some content to dynamically compose in the body of our modal. Let's just steal from welcome.html to make it simple -

`View - person-information.html`

```
<template>  
  <form role="form">
    <div class="form-group">
      <label for="fn">First Name</label>
      <input type="text" value.bind="person.firstName" class="form-control" id="fn" placeholder="first name">
    </div>
    <div class="form-group">
      <label for="ln">Last Name</label>
      <input type="text" value.bind="person.lastName" class="form-control" id="ln" placeholder="last name">
    </div>
  </form>
</template>  
```

`View-model - person-information.js`

```
export class PersonInformation {
  constructor(person) {
    this.person = new Person();
  }
}

class Person{  
  firstName = 'Patrick';
  lastName = 'Patrick';
}
```

Here we just create a person class and instantiate it to be our 'person'. This will provide what we need in our person-information.html

### Tying it all together
The base of our modal is there. We have all the pieces we need and if our attached callback called the modal and instantiated it we would see it, albeit empty at this point.

### Adding the bindable properties
Let's add the following bindings to our modal html in app.html to bind the properties -

`app.html`

```
<modal>  
  <modal-header title="View Person"></modal-header>
  <modal-body content="/person-information"></modal-body>
  <modal-footer buttons.bind="['Cancel']"></modal-footer>
</modal>  
```

For the title we just used a string. The modal-body has a content property that tells it which template to render as the body. You can see this in our modal-body.html where we used the compose binding to render some content. Finally, the buttons is an array of button labels to show in our modal.

Now we have a functioning modal!

We've separated our template logic that is specific to bootstrap out of our main app. Now if we wanted to we could switch from Bootstrap to Semantic-ui or some other CSS framework and swap it in and out as needed!
