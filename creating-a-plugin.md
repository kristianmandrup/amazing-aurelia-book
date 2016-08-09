# Creating a plugin

From [making our first plugin](http://patrickwalters.net/making-out-first-plugin/)

Creating a plugin for Aurelia can be a daunting task up front. Where do we start? What conventions do we follow?

We all probably have a great idea for what would be a good plugin every time we are writing code. The difficult part is just getting started. I know for me everytime I think I have a new idea for a plugin or an app the hardest part is getting to write the first piece of relevant code.

### Introduction
The [skeleton-plugin repository](https://github.com/aurelia/skeleton-plugin) is a starter kit for developing a plugin for Aurelia. Using this repository we can go from 0 to about 40 mph instantly. No extra set-up required yet just to see our idea start to take shape. Let's take a lap around the repo to see how the pieces fit together.

### Configuration
The main entry point for our plugin from the outside world is going to be our package.json. The package.json file is where our most common package managers, including JSPM and NPM, will store key information about our package for consumers to use when installing. Under the covers JSPM uses this same file as well when creating the config.js file. You won't see a config.js file to start with in the skeleton-plugin because it gets created the first time we run $ jspm install in the root directory.

At the time of writing this JSPM maintains a list of available packages to install in the JSPM registry which as you can imagine is simply a pointer to the github or NPM package. We'll look at this a bit more in a bit.

### JSPM-specific configuration
For JSPM to be able to consume our plugin we need to set a few parameters in the jspm area of our `package.json`. Here's a sneak peek at that section -

```
"jspm": {
  "main": "index",
  "format": "amd",
  "directories": {
    "lib": "dist/amd"
  },
  "dependencies": {}
}
```

You can see the main is just like it sounds - the main entry point that JSPM will load your plugin at. Typically this should be an index.js file or a file with a name that matches your plugin. The reason you might want to make it named the same as your plugin is because you want the consumer to know where problems might be coming up later without having to deduce which index.js file it is. Take for example a developer with 10 plugins - if all 10's main files were named index.js it would be hard to figure out which is failing up front. If each were named appropriately it helps the developer figure out wherein the problem lies.

The format property indicates which type of module system.js can expect to load. We specify amd at the moment due to some performance issues we ran in to with the system spec in early early alpha. You could choose any formats that system.js supports.

The directories property is a pointer to where your library lives. In our case we actually are building our plugin during gulp build in to all three module formats (system, AMD, and commonjs) so we need to point to the correct one to use that corresponds to our format defintion from earlier.

Last, if we have any dependencies they would be listed in the dependencies section. You can see from other plugins such as the aurelia-http-client that this could be aurelia dependencies or even other things like core-js.

### Creating our main entry point
For our purpose let's call our entry point index.js. Our index.js file goes in our src directory (where all of the files we want to edit go) and should always export a configure method. It doesn't have to export a class, just the configure function. The reason we need to export a configure function is so that Aurelia can call the function by convention and allow it to configure itself without having to make the plugin consumer copy a bunch of boilerplate code in to their app.

In our `configure` function we can use the same `globalResources` method available from the `aurelia.use` object to expose a custom element to all of the consumers' views. This can be really helpful in preventing the consumer from having to require it all over their app.

In our `index.js` we can also export classes, objects, or functions. This allows the consumer to import and use those classes if they need to.

### Creating our plugin
For actually creating the plugin we need to put all of our files in the `/src` directory. We can put custom elements, custom attributes, or anything else we are trying to expose in there. Your plugin idea may have many different needs so it will probably be difficult to get down in to the nitty gritty of what to put in there. In essence, if you think it's re-usable put it in your plugin! To get some ideas for plugins check out the official Auredlia plugin repository for a list of official and community plugins.

### Packaging it up
Say we've finished up our plugin and we want to share it now. What's the best way to do it?

- Get some tests in place - no one wants an untested plugin!
- Run gulp build - this creates the distributable files for others to consume.
- Add your plugin to the JSPM and NPM registries 
- Share it!

## Plugin registration
This allows others to easily install it. 

### JSPM
Fork the [registry repository](https://github.com/jspm/registry), add your information in to the [registry.json](https://github.com/jspm/registry/blob/master/registry.json) file, and it immediately becomes available for others to `jspm install my-plugin`.

### NPM
Add a `package.json` file via `npm init`. Then `npm publish`.


