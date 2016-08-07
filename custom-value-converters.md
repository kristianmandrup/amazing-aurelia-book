# Value converters

Writing custom value converters, with at look at some typical [value converters](https://github.com/Vheissu/aurelia-code-snippets/tree/master/Value%20Converters)

First check out these [value converter samples](http://jdanyow.github.io/aurelia-converters-sample/)

## Creating your own

The Aurelia CLI comes with various generators, including one for value converters.

`au generate` will list the generators available

```
value-converter - Creates a value converter class and places it in the project resources.
```

`$ au generate value-converter <name>`

### Day name converter

Let's create `date-day-name` converter

`$ au generate value-converter date-day-name`

We will use [momentjs](http://momentjs.com/) as the `Date` utility library.

Create a `date-day-name.ts` file in `/src`

```ts
import * as moment from 'moment';

export class DayNameValueConverter {
    toView(value) {
        return  moment(value).format('dddd');
    }
```

The `toView` method to converts the value passed in for display to the view.

### Testing the converter

We can test it like this:

`showdate.ts` VM

```ts
export class ShowDate {
  date: string = 'December 17, 1995 03:24:00';
}
```

`showdate.html` template

```html
<template>
 <require from="./resources/value-converters/date-day-name"></require>
 <h1 textContent.bind="date | dayName">Sunday</h1>
</template>
```

Alternatively you can make the value converter global and drop the `<require>` as described in the chapter *Smart resources*.

### Bi-directional converter

You can make the converter bi-directional, by adding a `fromView` method.
Here we set the week of the day, using the `day(weekDay)` setter, then convert to a Javascript Date via `toDate()`.

```ts
export class DayNameValueConverter {
  fromView(value) {
    return  moment(value).day(value).toDate();
  }
```

### Locales

To use a different locale, simply configure it from `main.ts` or wherever you control your locale and similar configurations (such as from a custom locale selector element?)

```ts
import 'moment/locale/pt-br';
moment.locale('pt-BR');
```

