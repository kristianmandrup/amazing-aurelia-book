# Best practices

From [solving-the-m-in-mvvm](http://patrickwalters.net/my-best-practices-for-aurelia-solving-the-m-in-mvvm/)

In the past with JavaScript we've been limited by how well we can solve for the model aspect in the MVVM pattern. Traditionally we've been limited to creating a function that we pass off as a class or using a more robust client-side ORM such as Breeze.js. The function was a little dumbed down and sometimes an ORM is overkill when starting a new project.

What about the middle ground where we want to have a model we can work with?

ES6 Classes to the rescue
With the newest standard being approved we now have ES6 classes. With classes we can have a single area to start with to extend our models.

export class Person {  
  constructor(data){
    Object.assign(this, data);
  }
}
Object.assign()
Now we've created a model for a Person. We know that there is some JSON we are getting from the server and we want to 'cast' that data into an instance of a Person. We can use Object.assign() here in our constructor to tell our model that whatever properties the server is returning we want to keep. This means that if our person's data is returned with a property like firstName it will also exist in our model. Then we can extend it.

That works great, but doesn't really offer us any tangible benefits yet.

Adding properties
Our person is lonely and broke. Whenever we create a new person let's give them no money whatsoever and make them understand the hard lessons of life and respect what it means to earn a buck -

export class Person {  
  constructor(data){
    Object.assign(this, data);
    this.money = 0;
  }
}
Now anywhere that we reference our person in our application we can rely on him to have a property called money.

But what if we want to give our person some money?

Fatten up our model
In Ruby on Rails there is a principle of Fat Models, Skinny Controllers that many people like, and many others don't. I personally have found it to be a breath of fresh air for me for two main reasons -

It promotes re-use of code
It is much easier for me to test
I'm not going to argue those points much but I will say this - when all you need to do is instantiate your model and then call methods it's much easier than instantiating a view-model or controller and wiring everything up and then testing it.

Ok add a method already!

export class Person {  
  constructor(data){
    Object.assign(this, data);
    this.money = 0;
  }
  giveMoney(amount){
      this.money += amount;
  }
}
We've added a simple method to give our person some money. This seems a bit trivial, so why add a method just to change a property? Why not simply change the property in our view-model or service? Because business logic. We are starting out a simple app and we want to put some business logic in for when we give money to our person. You could argue that this needs to be extracted and follow a better architecture pattern to which I would say I will when I need to, and I don't need to yet.

Imagine that when we give our person some money we also want to improve his credit score. A person with more money is more likely to be lended to. Instead of everywhere we give our person money having to put in the same logic of improving the credit score, we put it in our method -

export class Person {  
  constructor(data){
    Object.assign(this, data);
    this.money = 0;
    this.creditScore = 500;
  }
  giveMoney(amount){
      this.money += amount;
      this.creditScore += amount / 2;
  }
}
Now anywhere in our app we can give our person some money and expect that the appropriate change to credit score is applied. When we write our unit test we can verify that the credit score changed relevant to the amount of money the person was given.

Casting our JSON as a Person
Thus far we've only written code which is agnostic of the framework. These principles can be applied to any JavaScript you write. We could continue on this path, but for this write-up I want to use the HttpClient that Aurelia provides to showcase how to 'cast' your data to our Person class.

Service
Let's create a simple service which retrieves some people and casts them as a Person.

import {HttpClient} from 'aurelia-http-client';  
import {Person} from './models';  
export class PersonService {  
  constructor(){
      this.http = new HttpClient().configure(x=> {
        x.withReviver((k,v) => {        
          return typeof v === 'object' ? new Person(v) : v;
        });  
    });
  }
  getPeople(){
      return this.http.get('/people');
  }
}
Whoa whoa whoa too much at once!

Ok fine let's break it down -

We imported our Person from our models.js file where we exported it.
We instantiated an instance of the HttpClient provided by Aurelia. (We didn't inject it with DI because it would be a singleton and we don't want to affect our other services!)
We configured our HttpClient to always use a reviver.
We added a method on our service to use the HttpClient we've created to get people from our resource on the server.
Reviver
A reviver is awesome. It will save you if you die in your favorite First Person Shooter. In JavaScript, it will also be called once for each item in the array when our JSON is parsed from the server. By using withReviver() and passing in a function that casts the value to a Person, Aurelia helps us do something like this -

JSON.parse(response, (key, value) => {  
  if (typeof value === 'object') {
      return new Person(value);
  } 
  return value;
});
Sample JSON payload
If you want to try this out, use this example JSON payload -

[{"name": "Jane"},{"name": "Bob"}]
Conclude this post already!
We've created a model, we've used the HttpClient to cast it, and we've touched on some topics that are a bit risque.

Let's end on a high note by showing our usage from our view-model -

import {PersonService} from './person-service';  
import {inject} from 'aurelia-framework';

@inject(PersonService)
export class MyViewModel {  
  constructor(personService){
      this.jane = {};
      this.personService = personService;
      this.personService.getPeople().then(response => {
        this.jane = response.content[0];
        this.jane.giveMoney(100);
    });
  }
}
Great! We've improved Jane's credit score and made her rich!