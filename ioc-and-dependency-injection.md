# IoC and dependency injection

From [inversion of control with aurelia](https://www.danyow.net/inversion-of-control-with-aurelia-part-1/)

If you've never heard of DI or IoC, take a few minutes to read [Inversion of Control Containers and the Dependency Injection pattern](http://www.martinfowler.com/articles/injection.html) by Martin Fowler.

"Inversion of control (IoC) describes a design in which custom-written portions of a computer program receive the flow of control from a generic, reusable library. A software architecture with this design inverts control as compared to traditional procedural programming: in traditional programming, the custom code that expresses the purpose of the program calls into reusable libraries to take care of generic tasks, but with inversion of control, it is the reusable code that calls into the custom, or task-specific, code." - [Wikipedia](https://en.wikipedia.org/wiki/Inversion_of_control)

Consider a typical single page application with multiple screens. If you built this application without Aurelia you might find yourself doing a lot of this:

Load/instantiate a view-model.
Load/instantiate a view.
Bind the view to the view-model.
Append the view to the DOM.
User clicks a link.
Parse the url hash, determine which view-model to load/instantiate, check whether the current view can be deactivated, etc
Rinse and repeat.
Without Aurelia you're implementing the logic that controls the application lifecycle in addition to the features specific to you application.

Contrast this with an application built with Aurelia. You write none of the boiler-plate to control the application life code above. Instead you focus on writing the views, view-models, behaviors and routes that embody your application's custom logic and appearance. Aurelia inverts the control, handling the application lifecycle while allowing you to customize the default behavior at well defined lifecycle hooks.

Lifecycle hooks are optional methods you attach to view-models. Aurelia's router and templating engine will invoke these methods at the appropriate time, allowing you to control specific lifecycle steps.

There's the route configuration and screen activation hooks that allow you to control navigation within your app:

configureRouter(config, router) - Implement this hook if your view-model needs to translating url changes into application state.
canActivate(params, routeConfig, navigationInstruction) - Implement this hook if you want to control whether or not your view-model can be navigated to. Return a boolean value, a promise for a boolean value, or a navigation command.
activate(params, routeConfig, navigationInstruction) - Implement this hook if you want to perform custom logic just before your view-model is displayed. You can optionally return a promise to tell the router to wait to bind and attach the view until after you finish your work.
canDeactivate() - Implement this hook if you want to control whether or not the router can navigate away from your view-model when moving to a new route. Return a boolean value, a promise for a boolean value, or a navigation command.
deactivate() - Implement this hook if you want to perform custom logic when your view-model is being navigated away from. You can optionally return a promise to tell the router to wait until after your finish your work.
There's also the component/behavior lifecycle hooks that allow customizing a view, custom-element or custom-attribute's lifecycle:

created(view) - Invoked after both the view and view-model have been created. Allows your behavior to have direct access to the View instance.
bind(bindingContext) - Invoked when the databinding engine binds the view. The binding context is the instance that the view is databound to.
unbind() - Invoked when the databinding engine unbinds the view.
attached() - Invoked when the view that contains the extension is attached to the DOM.
detached() - Invoked when the view that contains the extension is detached from the DOM.
Last but not least, there are a number of decorators that allow you to further customize your component's behavior within Aurelia's composition lifecycle:

@processContent(false|Function) - Tells the compiler that the element's content requires special processing. If you provide false to the decorator, the the compiler will not process the content of your custom element. It is expected that you will do custom processing yourself. But, you can also supply a custom function that lets you process the content during the view's compilation. That function can then return true/false to indicate whether or not the compiler should also process the content. The function takes the following form function(compiler, resources, node, instruction):boolean
@useView(path) - Specifies a different view to use.
@noView() - Indicates that this custom element does not have a view and that the author intends for the element to handle its own rendering internally.
@inlineView(markup, dependencies?) - Allows the developer to provide a string that will be compiled into the view.
@containerless() - Causes the element's view to be rendered without the custom element container wrapping it. This cannot be used in conjunction with @sync or @useShadowDOM. It also cannot be uses with surrogate behaviors.
@useShadowDOM() - Causes the view to be rendered in the ShadowDOM. When an element is rendered to ShadowDOM, a special DOMBoundary instance can optionally be injected into the constructor. This represents the shadow root.
As you can see Aurelia makes heavy use of the IoC pattern to reduce the work required to build applications. It does this without sacrificing flexibility by using lifecycle hooks, overridable conventions and a pluggable architecture built upon a dependency injection container. In the next post I'll cover some of the inner workings of Aurelia's DI container, followed by a third post covering common uses for the container as well as it's use within the Aurelia framework.

We've covered how Aurelia uses the IoC pattern at the macro level- via lifecycle hooks and conventions, now let's take a look at how a sub-pattern of IoC, "dependency injection" works in Aurelia applications.

Dependency Injection
If you haven't heard of the dependency injection pattern (DI), here's what Wikipedia has to say about it:

Dependency injection is a software design pattern that implements inversion of control for resolving dependencies.

A dependency is an object that can be used (a service). An injection is the passing of a dependency to a dependent object (a client) that would use it. The service is made part of the client's state. Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.

Dependency injection allows a program design to follow the dependency inversion principle. The client delegates to external code (the injector) the responsibility of providing its dependencies. The client is not allowed to call the injector code. It is the injecting code that constructs the services and calls the client to inject them. This means the client code does not need to know about the injecting code. The client does not need to know how to construct the services. The client does not need to know which actual services it is using. The client only needs to know about the intrinsic interfaces of the services because these define how the client may use the services. This separates the responsibilities of use and construction. 
Wikipedia

Two modules are the key enablers for the DI pattern's application in Aurelia:

dependency-injection: Contains a lightweight, extensible dependency injection container for JavaScript.
metadata: Contains utilities for reading and writing the metadata of JavaScript functions. It provides a consistent way of accessing type, annotation and origin metadata across a number of languages and formats.
Example
To illustrate how the DI and metadata modules work together, let's define a typical view-model class that we can refer back to. The github users view-model in Aurelia's skeleton-navigation sample app is as good a candidate as any:

JavaScript:

import {inject} from 'aurelia-framework';  
import {HttpClient} from 'aurelia-fetch-client';

@inject(HttpClient)
export class Users {  
  http;
  constructor(http) {
    this.http = http;
    ...
  }
  ...
}
TypeScript:

import {autoInject} from 'aurelia-framework';  
import {HttpClient} from 'aurelia-fetch-client';

@autoInject()
export class Users {  
  constructor(private http: HttpClient) {
    ...
  }
  ...
}
How is a view-model created at runtime?
We know from the previous post that Aurelia takes care of the boilerplate work of locating and instantiating view-models, but how does this work exactly?

Aurelia uses the DI container to instantiate all view-models.
View-models following the dependency inversion principle such as the one above, do not instantiate or locate their own dependencies. They rely on Aurelia to supply the dependencies as constructor arguments.

To instantiate a class, Aurelia's DI container needs to know the class's dependencies.
In strongly-typed languages that support reflection, DI container implementations can determine a class's dependencies by reflecting on the constructor function's argument list. There are some caveats to this, but at the simplest level, this is how dependencies are discovered.

In JavaScript, type information can be stored as metadata.
In JavaScript, there is no reliable way to determine a constructor function's argument types. To overcome this, we must embed this information on the class itself, as "metadata". An ES7 proposal for a standard way of adding metadata to classes, called the Metadata Reflection API can be polyfilled and leveraged for this purpose.

By combining the Metadata Reflection API with the ES7 decorator proposal we can use decorators to add constructor signature information to our classes, to be consumed by Aurelia's DI container. That's what's happening on the @inject(HttpClient) line in the JavaScript version of the view-model.

TypeScript can automate the creation of type metadata.
If you're using TypeScript with the experimental emitDecoratorMetadata flag, it's even easier to add constructor signature information to your classes. Simply add the @autoInject() decorator to your class- no need to list the constructor's parameter types!

How does @autoInject() work? To understand this you need to understand what TypeScript's emitDecoratorMetadata flag does. The flag causes the TypeScript compiler to polyfill the Metadata Reflection API and add a special decorator definition to the transpiled TypeScript code. The decorator is then applied to your class's constructor, methods and properties in the transpiled code. This has the effect of adding type metadata for constructor parameters, method parameters and property-accessor return types.

Aurelia's @autoInject() decorator consumes the type metadata created by TypeScript's decorator and applies it to the class in the same way that the @inject(...) decorator does. In other words, @autoInject() translates TypeScript metadata to the metadata representation used by Aurelia.

Dependency resolution is a recursive process
In our example, the Users view-model has a dependency on the Aurelia HttpClient. When the DI container instantiates the Users class it first needs to retrieve the HttpClient instance or instantiate one if it doesn't already exist in the container. The HttpClient may have dependencies of it's own, which the DI container will recursively resolve until the full dependency chain has been identified.


