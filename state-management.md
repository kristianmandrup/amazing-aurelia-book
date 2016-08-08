# State management

Using Breeze, ORMs, RethinkDB and other state management solutions.

- [aurelia-breeze](https://github.com/jdanyow/aurelia-breeze)
- [aurelia-orm](https://github.com/SpoonX/aurelia-orm)

## Aurelia ORM

Working with endpoints and client-side entities is unavoidable when doing API driven development. You end up writing a small wrapper for your XHRs / websocket events and putting your request methods in one place. Another option is just using [Breeze](breezejs.com), which is large, complex and not meant for most applications. Even though your endpoints are important, you still end up neglecting them.

Enter [aurelia-orm](https://github.com/SpoonX/aurelia-orm). This module provides you with some useful and cool features, such as:

- Entity definitions
- Repositories
- Associations
- Validation
- Type casting
- Self-populating select element
- And more

This makes it easier to focus on your application and organize your code.