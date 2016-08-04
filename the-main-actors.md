# The main actors

An Aurelia app operates with these main actors:

- `app configuration` (`aurelia` object)
- `router`
- `route`
- `view model`
- `view`

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



