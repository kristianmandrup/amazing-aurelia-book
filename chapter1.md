# Getting started

In this book I have decided to stick with TypeScript and use Webpack as the package manager and application builder.

Most if not all apps dicsussed will be using these technologies. Furthermore we will leverage [aurelia CLI](https://github.com/aurelia/cli) wherever possible.

## Install TypeScript

Just go to the [typescript](https://www.typescriptlang.org/) site and follow install instructions.

`npm install -g typescript`

Also be sure to install typings, the tool to manage TypeScript typings \(ie. type definitions\)

`npm install typings --global`

## Install and use the Aurelia CLI

If you haven't already got the CLI, install it via npm as follows

`npm install aurelia-cli -g`

Test that it is installed by running `au`

You should see some ASCII art with the Aurelia and the commands available.

Now run `au new` to create a new project. Use the app name \`au-app-0\`  and select all the defaults. 
Be sure to choose _typescript_ and _install all dependencies_.

For more fine grained control over your starting configuration, instead create a new empty folder. Enter this folder and run `au new --here`

This runs the CLI in _advanced_ mode, specifically targeting ASP.NET Core. However it also works just fine without using .NET and gives you more options, such as selecting your CSS processor, preferred editor, unit testing on\/off and more...

## 

