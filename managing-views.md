# Managing views

Using the [aurelia-view-manager](https://github.com/SpoonX/aurelia-view-manager) plugin to manage and change the view location strategy to better suit different devices or for changing ui framework.

### Install

`npm i aurelia-view-manager`

### Configuration

```ts
export function configure(app) {
  let viewManagerConfig = app.container.get(ViewManagerConfig);
  viewManagerConfig.configureNamespace('spoonx/modal', {
    base: './views/{{framework}}',
    map: {
      modal: '{{base}}/{{view}}.html'
    }
  });
}
```

`main.ts`

```ts
export function configure(aurelia) {
  aurelia.use
    .plugin('aurelia-view-manager', view => {
      view.configureDefaults({
        framework: 'foundation'
      });
    })
    .plugin('spoonx-modal', modal => {
      /* ... */
    })
    /* ... */
}
```

```ts
import {Config as ViewManagerConfig} from 'aurelia-view-manager';
viewManagerConfig.configureNamespace('spoonx/modal', {
  map: {
    modal: ./views/{{framework}}.html
  }
});
```

Aurelia view manager has the reserved namespace defaults. The properties defined in this namespace are used by other namespaces when the other namespaces do not have that property defined on them. Understanding this is essential if you want to keep your configurations concise.
We are going to change the default location view manager is going to resolve to and we are going to define some default properties..

```ts
viewManagerConfig.configureNamespace('defaults', {
  device: getDeviceType(),
  permissions: 'user'
});
```

For all namespace configurations that do not have a device specified on their configurations [aurelia-view-manager](https://github.com/SpoonX/aurelia-view-manager) will user the ones defined in the defaults.

Assuming that you use [aurelia-form](http://aurelia-form.spoonx.org/). You have created a gorgeous datepicker and you want to use that one instead of the one provided with aurelia-form. Aurelia-form being a plugin that leverages view-manager, you are able to overwrite the template it will use to render dates. You only need to know what the namespace is. For ease of use I choose to keep it the same as the plugin's name.

```ts
viewManagerConfig.configureNamespace('spoonx/form', {
  map: {
    /***
     + When no .html is provided it knows you are trying to load a element
     + with a view model. That feature is specific to aurelia-form.
     */
    date: '/my/views/date'
  }
});
```

That is how you would overwrite the view of a plugin which you have installed.


## Usage

Decorate your custom elements with `@resolvedView`

```ts
@resolvedView('spoonx/modal', 'modal')
export class ModalCustomElement {
  /* code */
}
```


