# Advanced routing

In Aurelia you can choose to dynamically route rather than pre-configuring all your routes up front. 

Here's how you configure a router to do that:

```ts
router.configure(config => { 
  config.mapUnknownRoutes(instruction => { 
    //check instruction.fragment 
    //set instruction.config.moduleId 
  }); 
});

All you have to do is set the `config.moduleId` property and you are good to go. You can also return a promise from `mapUnknownRoutes` in order to asynchronously determine the destination.

The router also has an `addRoute` method that can be used to add a  route later.

## Multi level menu

[Multi level menu example](https://github.com/cmichaelgraham/aurelia-typescript/tree/master/multi-level-menu)

## Dynamic routes

See 



