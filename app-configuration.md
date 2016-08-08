# App Configuration

Basics of Aurelia configuration and using [aurelia-configuration] plugin...

(https://github.com/Vheissu/Aurelia-Configuration)

## Installation

`npm install aurelia-configuration --save`

Add plugin to your app's main.js:

```ts
export function configure(aurelia) {
    aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin('aurelia-configuration');

    aurelia.start().then(a => a.setRoot());
}
```

Use the plugin to set configuration in your app's main.js:

```ts
import {Configure} from "aurelia-configuration";

export function configure(aurelia) {
    aurelia.use
        .standardConfiguration()
        .plugin('aurelia-i18n', (instance) => {
                let configInstance = aurelia.container.get(Configure);
                let apiEndpoint = configInstance.get('api.endpoint');
    // ...
}
```

Create a config file. By default the plugin will assume a configuration file called: `config.json` inside of a root directory called `config` - the contents of the JSON file can be anything you like as long as it is a JSON object. You can configure the plugin to use a different config file if you like.

```json
{
    "name": "Test Application",
    "version": "1.2",
    "api": {
        "key": "somekey",
        "endpoint": "http://www.google.com/"
    }
}
```

### Using with your ViewModel

```ts
import {inject} from 'aurelia-framework';
import {Configure} from 'aurelia-configuration';

@inject(Configure)
export class ViewModel {
    constructor(config) {
        this.config = config;

        // Get configuration data using this.config
        // Single non-namespace item:
        this.config.get('name') // Using above sample config would return 'Test Application'
        // Single namespaced item:
        this.config.get('api.key') // Using above sample config would return 'somekey'

        // Get config item and if it doesn't exist return a default value provided as the second argument
        this.config.get('name', 'default value');

        // Setting a temporary config value non-namespaced:
        this.config.set('newkey', 'surprise!') // Would store a value of 'surprise!' on object {newkey: 'surprise!'}
        // Setting a temporary config value namespaced:
        this.config.set('websites.name', 'Google'); // Would store a value of 'Google' on object {websites: {name: 'Google'}}
    }
}
```

### Changing environment

The concept of environment configuration exists within this plugin. By default the plugin assumes no environment is specified, however you can have different levels of environment config values which you can configure the plugin to look in first.

```ts
export function configure(aurelia) {
    aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin('aurelia-configuration', config => {
            config.setEnvironment('development'); // Environment changes to development
        });

    aurelia.start().then(a => a.setRoot());
}
```

If a value is not found inside of a configuration group, it will attempt to look for it further up the configuration file. Situations where this might be useful is different API details for different environments.

```ts
{
    "name": "Test Application",
    "api": {
        "key": "somekey",
        "endpoint": "http://www.google.com/"
    },
    "development": {
        "api": {
            "key": "developmentonlykey938109283091",
            "endpoint": "http://localhost/api/v1"
        }
    }
}
```

If you have specified a particular environment and a config value does not exist, it will look for it further up the config file. For example if you have your environment set to `"development"`` and you request an API key but it isn't specified and you have a value specified futher up, it will use that.

The idea is environment specific config values extend parent values, similar to what Ruby on Rails does with its configuration. By default this behaviour is turned on, but if you don't want values to be searched outside of your specified environment, disable cascading (below).

### Dynamic Environment Switching

Manually specifying an environment might not be as efficient in all cases. In this instance you can configure the plugin to dynamically change your environment based on the current URL.

Doing so requires specifying one or more domains for particular environments.

```ts
export function configure(aurelia) {
    aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin('aurelia-configuration', config => {
            config.setEnvironments({
                development: ['localhost', 'dev.local'],
                staging: ['staging.website.com', 'test.staging.website.com'],
                production: ['website.com']
            });
        });

    aurelia.start().then(a => a.setRoot());
}
```

Changing directory and/or filename

If you want to change the location of where your configuration files are loaded from or the name, you configure it during the bootstrapping phase within your `main.js`

```ts
export function configure(aurelia) {
    aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin('aurelia-configuration', config => {
            config.setDirectory('config-files'); // Will make plugin look for config files in a directory called "config-files"
            config.setConfig('mycoolconfig.json'); // Will look for mycoolconfig.json as the configuration file
        });

    aurelia.start().then(a => a.setRoot());
}
```

### Changing cascade mode

By default, if you're using an environment value using `setEnvironment`, then by default if a value is not found in a particular environment it will search for it further up the configuration file sort of like inheritance. You might not want this behaviour by default, so you can turn it off by using the `setCascadeMode` method.

```ts
export function configure(aurelia) {
    aurelia.use
        .standardConfiguration()
        .developmentLogging()
        .plugin('aurelia-configuration', config => {
            config.setCascadeMode(false); // Disable value cascading
        });

    aurelia.start().then(a => a.setRoot());
}
```