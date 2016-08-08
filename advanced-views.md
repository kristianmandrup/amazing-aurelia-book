# Advanced views

## View strategy



### Jade views

[Jade](http://jade-lang.com/) is a popular templating language. We will use it as an example for how you can use a templating language of your choice with Aurelia, instead of relying on barebones `.html` files. Templating languages such as Jade can greatly simplify your views and come with a number of features that you might find useful.

Jade syntax example:

```jade
body
    h1 Jade - node template engine
    #container.col
      if youAreUsingJade
        p You are amazing
      else
        p Get on it!
```

To configure your project to use Jade (or similarly for any other custom templating engine), a simple solution is to create a `jade` gulp compile task which compiles all `.jade` files in `src` to `.html` (or precompiled `.js`) files of the same name.

```js
var gulp = require('gulp'),
    jade = require('gulp-jade');

gulp.task('jade-html', function() {
    return gulp.src('src/**/*.jade')
        .pipe(jade()) // pipe to jade plugin, .html files
        .pipe(gulp.dest('src')); // tell gulp our output folder
});

gulp.task('jade-js', function() {
    return gulp.src('src/**/*.jade')
        .pipe((jade({
          client: true // .js files
        }))) // pipe to jade plugin
        .pipe(gulp.dest('src')); // tell gulp our output folder
});


gulp.watch('src/**/*.jade', ['jade-js']);
```

Then you can use the `.html` files directly using basic Aurelia view location/load conventions. For `.js` files you can use the special Jade ViewStrategy as described next.

## Jade ViewStrategy

[aurelia-jade-viewstrategy](https://github.com/Craga89/aurelia-jade-viewstrategy) can be used either with the special SystemJS jade loader or directly with precompiled `.js` files by using the `isCompiled` flag.

```ts
import {JadeConventionView} from 'aurelia-jade-viewstrategy';
const isCompiled = true; // to use pre-compiled .jade.js file

class ViewModel {
    getViewStrategy() {
        return new JadeConventionView(isCompiled);
    }
}
```

### Customizing the ViewLocator

You can also set up the `ViewLocator` to load a `.jade` view by default.
This [recipe](https://github.com/aurelia/skeleton-navigation/issues/396#issuecomment-207823852) currently requires that you use webpack with a `jade` loader.

```ts
import {bootstrap} from 'aurelia-bootstrapper-webpack'
import {ViewLocator} from 'aurelia-framework'

bootstrap(function(aurelia) {
  aurelia.use
  // ...

  ViewLocator.prototype.convertOriginToViewUrl = function (origin) {
    let moduleId = origin.moduleId
    let id = (moduleId.endsWith('.js') || moduleId.endsWith('.ts')) ? moduleId.substring(0, moduleId.length - 3) : moduleId
    return id + '.jade'
  }
  // ...
```

