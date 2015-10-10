# Ember CLI Deploy
[![Build Status](https://travis-ci.org/ember-cli/ember-cli-deploy.svg?branch=master)](https://travis-ci.org/ember-cli/ember-cli-deploy) [![Code Climate](https://codeclimate.com/github/ember-cli/ember-cli-deploy/badges/gpa.svg)](https://codeclimate.com/github/ember-cli/ember-cli-deploy)

Simple, flexible deployment for your Ember CLI app

## Installation

```
npm install ember-cli-deploy --save-dev
```
## Quick start

After installation, choose [deployment adapters](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/plugins/) matching your deployment environment, [configure](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/configuration/) your deployment script appropriately and you're ready to [start deploying](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/how-to-use/).

## In-depth documentation

[Visit the Docs site](http://ember-cli.github.io/ember-cli-deploy/)

## Development workflow

`ember-cli-deploy` is an addon for deploying apps. It works fine for cases where your ember-cli app and API app are mostly developed in isolation. However, there are some cases where using the `--proxy` command line option is inadequate. For example:
 * authentication happens within your API application
 * you are progressively updating an app to Ember or developing a hybrid app (i.e. some pages are served by the API application) and you need both applications to work together seamlessly
 * the API app injects some initial state (e.g. session info or model preloads) into your ember-cli app's index page so that it is available without having to make xhr requests on boot

The development workflow simply writes your ember-cli app's index to your key-value store on each build so that your API app can read (and possibly modify) it just as it would in production.

During app development `ember server` is used to recompile and serve the assets as usual while the app index is served by your API app, eliminating the need to setup CORS or proxies.

This strategy can be easily accomplished with some `ember-cli-build.js` configuration. Keep in mind that you will need to modify your `fingerprintOptions` in development to prepend your ember-cli server host and port. This way requests for Ember assets still go to ember-cli and not your API application.

The following is an example that pushes the `index.html` to redis on each build:

in your `ember-cli-build.js`

```javascript
  module.exports = function(defaults) {
    var env = EmberApp.env()|| 'development';
    ...

    var fingerprintOptions = {
      enabled: true,
      ...
    };

    switch (env) {
      case 'development':
        fingerprintOptions.prepend = 'http://localhost:4200/'; // use fingerprinting to prepend your ember server domain path
      break;
      ...
    }

    var app = new EmberApp(defaults, {
      fingerprint: fingerprintOptions,
      emberCLIDeploy: {
        runOnPostBuild: (env === 'development') ? 'development-postbuild' : false, // returns the deployTarget
        configFile: 'config/deploy.js', // optionally specifiy a different config file
        shouldActivate: true, // optionally call the activate hook on deploy
      },
      ...
    });

    return app.toTree();
  };
```

in `config/deploy.js` you can define a custom pipeline for your deploy env.

```javascript
  if (deployTarget === 'development-postbuild') {
    ENV.plugins = ['redis']; // only use the redis pluging
    ENV.build = {
      environment: 'development'
    };

    ENV.redis = {
      keyPrefix: 'edd-cli',
      revisionKey: '__development__',
      allowOverwrite: true,
      type: 'redis', // this can be omitted because it is the default
      host: 'localhost',
      port: 6379,
      distDir: function(context) {
        return context.commandOptions.buildDir;
      }
    };
  }
```

## Contributing

Clone the repo and run `npm install`. To run tests,

    npm test
