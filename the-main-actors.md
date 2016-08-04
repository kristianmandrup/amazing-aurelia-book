# The main actors

An Aurelia app operates with these main actors:

- `app configuration` (`aurelia` object)
- `view model`
- `view`
- `router`
- `route`
- `resources`

## App configuration

Configuration of your app by default takes place in your `main.ts` file. You can have multiple sub-applications, each with their own configuration. 

Example: TODO

```ts
```

## View model

A _view model_ contains the data (model) for a view and controls the behavior for that view. It can be seen as a combined model and controller.

## View

A _view_ (V) reflects an underlying _view model_ (VM), usually with one-day data binding. Sometimes the view is also reflected back into the model for a two-way binding.

We will delve more into the Aurelia binding system later!

## Router

The router is used to manage state transitions in your app and hence what gets displayed for each state.

Example:

```ts
```

A router is configured with one or more routes and sub-routers. A sub-router acts as a branch in the routing tree configuration.

## Route

A route defines how a specific URL pattern is treated, in order to load and render a specific UI component (VM/V) in a view port.

## Resources

- `custom element`
- `custom attribute`
- `value converter`
- `binding behavior` (_esoteric_)

### Custom element

You can define your own elements, essentially _web components_ which you can use to compose your application.

### Custom attribute

You can define your own custom attributes which can be used to link view data to a view model in a specific way. 

## Value converter

A value converter can be used as a gateway or bridge, for transferring values between the _view_ and the _view model_.

## User Interface component

User Interface components have two parts, a _view_ (V) and a view model (VM). The _view_ is written in html (.html files) and is rendered into the DOM of the page. The _view model_ is written in javascript (or transpiled to js) and provides data and behavior to the view. Aurelia's _data binding_ engine links the two pieces together, so that changes in your data are reflected in the view and vice versa!



