---
title: Dependency injection in Express.js
date: 2023-03-07 10:11:00 +0000
tags: [JavaScript, Node]
render_with_liquid: false
---
In this post I want to look at dependency injection in [Express](https://expressjs.com/). 

> Most of what's written here is based on the [Practical Microservices](https://pragprog.com/titles/egmicro/practical-microservices/) book by Ethan Garofolo. You should check that out for a more detailed explanation.

What is dependency injection?

> Dependency injection is basically providing the objects that an object needs (its dependencies) instead of having it construct them itself. It's a very useful technique for testing, since it allows dependencies to be mocked or stubbed out.
https://martinfowler.com/articles/injection.html

What this means for Express is each module should export a function that accepts dependencies as arguments. 

Starting at the top level, we'll create a function to scaffold an Express application. This function will accept the `config` and `env` as arguments. 

```js
// src/app/express/index.js
const express = require('express');

function createExpressApp({ config, env }) {
  const app = express();

  return app;
}

module.exports = createExpressApp;
```

We'll typically want to mount middleware and routes at this level – to do this we'll define two more modules. These modules will also export a function which accepts its dependencies. 

```js
// src/app/express/index.js
const express = require('express');

// new
const mountMiddleware = require('./mount-middleware');

// new
const mountRoutes = require('./mount-routes');

function createExpressApp({ config, env }) {
  const app = express();

  // new
  mountMiddleware(app, env);

  // new
  mountRoutes(app, config);

  return app;
}

module.exports = createExpressApp;
```

In `mountMiddleware` we import some custom middleware and mount these on the app. To illustrate how this works I've borrowed some example middleware from the [Express documentation](https://expressjs.com/en/guide/writing-middleware.html).

```js
// src/app/express/mount-middleware.js
const express = require('express');
const { join } = require('path');

const logger = require('./logger');
const requestTime = require('./request-time');

function mountMiddleware(app, env) {
  app.use(logger);
  app.use(requestTime);
  app.use(
    express.static(join(__dirname, '..', 'public'), { maxAge: 86400000 }),
  );
}

module.exports = mountMiddleware;
```

The `logger` middleware logs a message to console. 

```js
// src/app/express/logger.js
function logger(req, res, next) {
  console.log('Logging!');

  next();
}

module.exports = logger;
```

The `requestTime` middleware stores the time of the current request on Express's `req` object.

```js
// src/app/express/requestTime.js
function requestTime(req, res, next) {
  req.requestTime = Date.now();

  next();
}

module.exports = requestTime;
```

The `mountRoutes` module will handle mounting all routes onto the application. We'll get back to this later. For now, we can use a stub function to get things going.

```js
function mountRoutes(app, config) {
  // TODO create application feature and mount here
}

module.exports = mountRoutes;
```

With the `createExpressApp` module in place, we can go ahead and use this to create an Express application.

```js
// src/index.js
const path = require('path');
const dotenv = require('dotenv').config({ path: path.resolve(__dirname, '../.env') });

const createExpressApp = require('./app/express');
const createConfig = require('./config');
const createEnv = require('./env');

const env = createEnv({ env: dotenv });
const config = createConfig({ env });
const app = createExpressApp({ config, env });

function onStart() {
  console.log('Express application started');
  console.table([['Port', env.port], ['Environment', env.env]]);
}

function start() {
  app.listen(env.port, onStart);
}

module.exports = {
  app,
  config,
  start,
};
```

We're still missing two more modules – `createConfig` and `createEnv`. 

In `createConfig` we will wire up the different modules/dependencies in our application – this includes instantiating the database client and any features we will be building. For now, we'll return the `env` until we have something more meaningful to do here. 

```js
// src/config.js
function createConfig({ env }) {
  return {
    env,
  }
}

module.exports = createConfig;
```

The `createEnv` module is responsible for providing the `env` variables to the application.

```js
// src/env.js
function getEnvVar(env, key) {
  if (!process.env[key]) {
    // eslint-disable-next-line no-console
    console.error(`[APP ERROR] Missing env variable: ${key}`);

    return process.exit(1)
  }

  return process.env[key]
}

function createEnv({ env }) {
  if (env.error) {
    // eslint-disable-next-line no-console
    console.error(
      `Failed to load env: ${env.error}`
    )

    process.exit(1)
  }

  return {
    appName: getEnvVar(env, 'APP_NAME'),
    env: getEnvVar(env, 'NODE_ENV'),
    port: parseInt(getEnvVar(env, 'PORT'), 10),
    databaseUrl: getEnvVar(env, 'DATABASE_URL'),
    messageStoreConnectionString: getEnvVar(env, 'MESSAGE_STORE_URL'),
    sqlFilePath: getEnvVar(env, 'SQL_FILE_PATH'),
  };
}

module.exports = createEnv;
```

With those modules in place, we're ready to start the server. We'll create a script which can be called from a `package.json` file which will serve as the entrypoint for the application. 

```js
// bin/start.js
const { start } = require('../');

start();
```

If you are following along and want to run the application, you will need to add the following `package.json` file to your project.

```json
{
  "name": "example",
  "version": "1.0.0",
  "description": "",
  "main": "./src/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start-dev-server": "nodemon -- src/bin/start-server.js --color"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "better-sqlite3": "^8.1.0",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "nodemon": "^2.0.21"
  },
  "devDependencies": {}
}

```

Then install dependencies and run the server

```bash
npm i
npm run start-dev-server

...

Express application started
┌─────────┬───────────────┬───────────────┐
│ (index) │       0       │       1       │
├─────────┼───────────────┼───────────────┤
│    0    │    'Port'     │     4000      │
│    1    │ 'Environment' │ 'development' │
└─────────┴───────────────┴───────────────┘
```

The server is running, but we still don't have any routes. 

We'll define a new module which will display some users in the application. Again, this module will export a function which accepts dependencies as arguments.

```js
const express = require('express');

function createHandlers({ queries }) {
  return {

  }
}

function createQueries({ db }) {
  return {

  }
}

function createUsers({ db }) {
  const queries = createQueries({ db });
  const handlers = createHandlers({ queries });

  const router = express.Router();

  router.route('/').get(handlers.index);
  router.route('/:userId').get(handlers.show);

  return {
    handlers,
    queries,
    router,
  }
}

module.exports = createUsers;
```

The `createQueries` function accepts the database client and returns `getUsers` and `getUser` queries.

```js
function createQueries({ db }) {
  function getUsers() {
    return db.prepare('SELECT * FROM users').all();
  }

  function getUser(userId) {
    return db.prepare('SELECT * FROM users WHERE id = ?').get(userId);
  }

  return {
    getUsers,
    getUser,
  }
}
```

The `createHandlers` function accepts the database `queries` and returns route handlers for `index` and `show`. 

```js
function createHandlers({ queries }) {
  function index(req, res, next) {
    const users = queries.getUsers();

    return res.render('users/templates/index', { users })
      .catch(next)
  }

  function show(req, res, next) {
    const { params: { userId } } = req;
    const user = queries.getUser(userId);

    return res.render('users/templates/show', { user })
      .catch(next);
  }

  return {
    index,
    show,
  }
}
```

With the module defined we can revisit the `createConfig` module to see how `createUsers` is instantiated. We also need to implement one more module, `createDbClient`.

```js
// src/config.js
const createDbClient = require('./db-client');
const createUsersApp = require('./app/users');
function createConfig({ env }) {
  const db = createDbClient({
    sqlFilePath: env.sqlFilePath,
  });

  const usersApp = createUsersApp({ db });

  return {
    env,
    db,
    usersApp,
  }
}

module.exports = createConfig;
```

The `createDbClient` modules accepts a file path argument and returns an `sqlite3` database client.

```js
// src/db-client.js
const db = require('better-sqlite3');

function createDbClient({ sqlFilePath }) {
  const client = db(sqlFilePath);

  return client;
}

module.exports = createDbClient;
```

Now we're ready to mount the `users` module onto the application routes. 

```js
function mountRoutes(app, config) {
  app.use('/users', config.usersApp.router);
}

module.exports = mountRoutes;
```

And that's it. You should now have an API which you can use to query users.

> To run this example you will need the sql file which can be found [here](https://github.com/aaronmarr/express-di-example/blob/main/src/sqlite3.db) and should be placed in `/src/sqlite3.db`

```bash
curl http://localhost:4000/users  

[{"id":1,"username":"hamish","email":"hamish@example.com"},{"id":2,"username":"sarah","email":"sarah@example.com"},{"id":3,"username":"jason","email":"jason@example.com"}]    

curl http://localhost:4000/users/1

{"id":1,"username":"hamish","email":"hamish@example.com"}  
```

I think this is a great approach. Each module accepts dependencies as arguments which makes things straightforward to test – any module can be tested directly, and we can easily pass in dependencies, which makes mocking/stubbing modules a lot easier. 

I hope you've found this useful. The source code used in this post is available on [Github](https://github.com/aaronmarr/express-di-example). Thanks for reading.
