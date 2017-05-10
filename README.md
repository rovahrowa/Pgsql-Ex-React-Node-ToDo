# Pgsql-Ex-React-Node-ToDo
Before we begin...

Let's take a moment to review the tools we're going to be using:

NodeJS - We're going to use this to run JavaScript code on the server. I've decided to use the latest version of Node, v6.3.0 at the time of writing, so that we'll have access to most of the new features introduced in ES6.
Express - As per their website, Express is a "Fast, unopinionated, minimalist web framework for Node.js", that we're going to be building our Todo list application on.
PostgreSQL - This is a powerful open-source database that we're going to use to store our Todos. Unfortunately, details on how to install and configure PostgreSQL on your particular system fall beyond the scope of this tutorial. However, if you face issues while installing PostgreSQL, or you don't want to dive into installing it, you can opt for a version of PostgreSQL hosted online. I recommend ElephantSQL. I found it's pretty easy to get started with. However, the free version will only give you a 20MB allowance. Taking into consideration the pretty small size of the application we're building, this should be more than enough.
Sequelize - In addition, we're going to use Sequelize, which is a database ORM that will interface with the Postgres database for us.
Postman - A Chrome app that we'll use to practically test our API.
This tutorial assumes at least some prior knowledge of the JavaScript language. In addition, we're going to be leveraging some of the latest JavaScript features and it's recommended that you go through these tutorials from scotch.io ( Part I, Part II, Part III) to familiarize yourself with the new ES6 syntax. We're also assuming that you're at least comfortable working on the command line (creating folders, files, changing directories, e.t.c).

Code blocks will look like this:
```
function helloWorld(name) {
    if (name) {
        console.log(`Hello ${name}, and welcome to the world of computing!`);
    } else {
        console.log('Hello world!');
    }
}
```
while shell commands will look like this:

$ echo "This is awesome!"
When you encounter shell commands, what you'll be expected to type in will be everything except the leading $. This leading $ is called a shell prompt, and may be different depending on the terminal you're working in.

When you encounter any piece of code wrapped with ellipses (...), it means that's a code snippet which should be taken into context with the surrounding code in that file. This is used to save on space and reduce redundancy.

Alright, let's get to it! :)

# Project Setup

Let's begin by setting up our workspace. While we could use express-generator to scaffold the project structure for us, we're going to be creating one from scratch. Run the following commands.

$ mkdir -p postgres-express-react-node-tutorial/{bin,server}
$ cd postgres-express-react-node-tutorial
We're leveraging shell expansion to create three directories, a top level postgres-express-react-node-tutorial directory containing bin and server. All of the code necessary to create our server-side application will go into the server folder.

All subsequent commands will assume that you're in the top-level postgres-express-react-node-tutorial folder.

That done, we're going to initialize a NodeJS application, with the help of npm, which should have come bundled with your NodeJS install. Run:
```
$ npm init -y
```
This will create a package.json file with some sensible default config. Omit the -y if you want more control over this config. Your project structure should now look like:
```
postgres-express-react-node-tutorial
├── bin
├── package.json
└── server
```
# Express Setup

Install Express and a few of it's dependencies.
```
$ npm install --save express body-parser morgan
```
The --save flag will save these packages to the dependencies section of your package.json file. You could save some typing by running the above command as
```
$ npm i -S express body-parser morgan
```
i is an alias for install while -S is an alias for --save.

Create a file in the root folder and call it app.js.

$ touch app.js
In this file, let's create our Express application.
```
const express = require('express');
const logger = require('morgan');
const bodyParser = require('body-parser');

// Set up the express app
const app = express();

// Log requests to the console.
app.use(logger('dev'));

// Parse incoming requests data (https://github.com/expressjs/body-parser)
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

// Setup a default catch-all route that sends back a welcome message in JSON format.
app.get('*', (req, res) => res.status(200).send({
  message: 'Welcome to the beginning of nothingness.',
}));

module.exports = app;
```
Inside the bin folder, create a file and call it www.
```
$ touch bin/www
```
Put the following code inside bin/www
```
// This will be our application entry. We'll setup our server here.
const http = require('http');
const app = require('../app'); // The express app we just created

const port = parseInt(process.env.PORT, 10) || 8000;
app.set('port', port);

const server = http.createServer(app);
server.listen(port);
```
With that in place, we'll need a way to restart the server every time we change something in our code. For that, we'll use the excellent nodemon npm package.
```
$ npm i -D nodemon
```
Then, open up your package.json file and create a command to run the server. That command will be created under the scripts section. Edit your package.json in the scripts section as follows:
```
...
"scripts": {
  "start:dev": "nodemon ./bin/www",
  "test": "echo \"Error: no test specified\" && exit 1"
},
...
```
Now try running the application by executing
```
$ npm run start:dev
```
and visiting http://localhost:8000. You should see {"message":"Welcome to the beginning of nothingness."}

At this point in time, your project structure should look like:
```
postgres-express-react-node-tutorial
├── app.js
├── bin
│   └── www
├── package.json
└── server
```
For comparison, you can find the code for this section here.

# Sequelize Setup

For this part, we are going to require a working PostgreSQL installation. There are lots of resources on the web on how to install and configure Postgres, so I will not concentrate on that.

Next, we are going to require Sequelize. This is an ORM that will interface with the Postgres database for us. Feel free to go through it's documentation to get a feel of it. We are going to be making use of the Sequelize CLI package to bootstrap the project for us. It will also help us generate database migrations.

Let's begin by installing Sequelize CLI package.
```
$ npm install -g sequelize-cli
```
You can install the sequelize-cli package in your project locally by using -D (equivalent to using --save-dev) instead of the -g (--global) flag. The downside of doing this will be that you'll need to prefix every call to the sequelize command with ./node_modules/.bin.

Next, we need to configure Sequelize for our project. For that, we will create a .sequelizerc file in our project's root folder. In this file, we are going to be specifying the paths to files required by Sequelize. Put the following content into this file:
```
const path = require('path');

module.exports = {
  "config": path.resolve('./server/config', 'config.json'),
  "models-path": path.resolve('./server/models'),
  "seeders-path": path.resolve('./server/seeders'),
  "migrations-path": path.resolve('./server/migrations')
};
```
The config.json file contain our application configuration settings, such as database authentication configuration. migrations folder will hold our application's migrations, while the models folder will hold the application models. Seed data is initial data provided with a system for testing, training or templating purposes. The seeders folder typically holds seed data, but we're not going to be using that in this tutorial.

At this point, we are going to need to install the actual Sequelize package, alongside its dependencies.
```
$ npm install --save sequelize pg pg-hstore
```
pg will be responsible for creating the database connection while pg-hstore is a module for serializing and deserializing JSON data into the Postgres hstore format.

Now, with the paths defined, we will need to run the init command in order to create the specified folders and generate boilerplate code.
```
$ sequelize init
```
If you inspect your directory right now, you will realize that the above command just created the directories and generated the boilerplate code. Your directory structure should now look like this.
```
postgres-express-react-node-tutorial
├── app.js
├── bin
│   └── www
├── package.json
└── server
    ├── config
    │   └── config.json
    ├── migrations
    ├── models
    │   └── index.js
    └── seeders
    ```
Let's consider, for example, the server/models/index.js file that was autogenerated.
```
'use strict';

var fs        = require('fs');
var path      = require('path');
var Sequelize = require('sequelize');
var basename  = path.basename(module.filename);
var env       = process.env.NODE_ENV || 'development';
var config    = require(__dirname + '/../config/config.json')[env];
var db        = {};

if (config.use_env_variable) {
  var sequelize = new Sequelize(process.env[config.use_env_variable]);
} else {
  var sequelize = new Sequelize(config.database, config.username, config.password, config);
}

fs
  .readdirSync(__dirname)
  .filter(function(file) {
    return (file.indexOf('.') !== 0) && (file !== basename) && (file.slice(-3) === '.js');
  })
  .forEach(function(file) {
    var model = sequelize['import'](path.join(__dirname, file));
    db[model.name] = model;
  });

Object.keys(db).forEach(function(modelName) {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```
In this file, we are requiring the modules we're going to be using. Then, we're reading the configuration specific to our current Node environment. If we don't have a Node environment defined, we're defaulting to development. Then, we are establishing a connection with our database, after which we read our models folder, discovering and importing any and all the models in it, adding them to the db object and applying relationships between the models, if such relationships exist.

Refactoring server/models/index.js

Since the generated server/models/index.js file is in ES5 syntax, we are going to refactor it to ES6 syntax. If you are not familiar with ES6 syntax, you can learn more about it from this awesome Scotch.io tutorial series: Part I, Part II, Part III.
```
const fs = require('fs');
const path = require('path');
const Sequelize = require('sequelize');
const basename = path.basename(module.filename);
const env = process.env.NODE_ENV || 'development';
const config = require(`${__dirname}/../config/config.json`)[env];
const db = {};

let sequelize;
if (config.use_env_variable) {
  sequelize = new Sequelize(process.env[config.use_env_variable]);
} else {
  sequelize = new Sequelize(
    config.database, config.username, config.password, config
  );
}

fs
  .readdirSync(__dirname)
  .filter(file =>
    (file.indexOf('.') !== 0) &&
    (file !== basename) &&
    (file.slice(-3) === '.js'))
  .forEach(file => {
    const model = sequelize.import(path.join(__dirname, file));
    db[model.name] = model;
  });

Object.keys(db).forEach(modelName => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```
With the application bootstrapped, the only thing that we still have to do is creating our database then updating the config.json file to reflect our various environments settings.

First, we need to create a development database.
```
$ createdb todos-dev
```
createdb command will be available to us after installing PostgreSQL.

A Quick Note

If we opted to use a Postgres database hosted online, we will need the database url provided to us by the database host we chose. Assuming we're using ElephantSQL, we'll need to go to the ElephantSQL dashboard and click on details to view the details of our free database instance. We then copy the URL provided in the details. We are going to need this URL in the configuration below.

Then, we need to update our config to use the db we just created.

If we're using a local database, we replace username with our username and password with our database's password. In my case, notice I didn't create any password for my db, so I'm just going to leave that field null. Remember to change the dialect to postgres.
```
{
  "development": {
    "username": "waiyaki",
    "password": null,
    "database": "todos-dev",
    "host": "127.0.0.1",
    "port": 5432,
    "dialect": "postgres"
  },
  "test": {
    "username": "waiyaki",
    "password": null,
    "database": "todos-test",
    "host": "127.0.0.1",
    "port": 5432,
    "dialect": "postgres"
  }
}
```
If we're using an online database, we will need to replace the development environment database configuration settings with our database URL. Since we're only concerned about the development environment because it's all we're going to use, that's what we'll replace. We are going to be exporting the URL we copied earlier into our development environment as DATABASE_URL. Our config will now look like:
```
{
  "development": {
    "use_env_variable": "DATABASE_URL"
  },
  "test": {
    "username": "waiyaki",
    "password": null,
    "database": "todos-test",
    "host": "127.0.0.1",
    "port": 5432,
    "dialect": "postgres"
  }
}
```
This signals Sequelize to look inside our environment and extract the key whose name is DATABASE_URL and use that to connect to our DB. The specific logic that does that is in server/models/index.js, as shown in this snippet:
```
...
let sequelize;
if (config.use_env_variable) {
  // From the environment, extract the key with the name provided in the config as use_env_variable
  // and use that to establish a connection to our database.
  sequelize = new Sequelize(process.env[config.use_env_variable]);
} else {
  sequelize = new Sequelize(
    config.database, config.username, config.password, config
  );
}
...
```
Finally, we'll need to actually export our database URL into our environment. In our terminal, let's issue the following command:
```
$ export DATABASE_URL=our-database-url
```
where our-database-url is the URL we copied from ElephantSQL. Every time we need to run this application, we will need to export the DATABASE_URL first. Fortunately, there exists dotenv, an npm package that makes automatically exporting values into our app environment a breeze. It reads key-value pairs stored in a config file, typically named .env and exports them into our environment. We won't use it in this application but I recommend you check it out.

We are not actually going to use the test environment in this tutorial. However, I'm including it here for didactic purposes to show that we can have different databases for different environments, which is what we'd actually want in a real-world application.

For purposes of comparison, you can find the code to this section here.

# Generating Models

With our configuration in place, we are now ready to generate models. We are going to have two models, Todo and TodoItem. The relationship between a Todo and it's TodoItems is going to be one-to-many, such that a Todo can have many TodoItems while a TodoItem can only belong to one Todo.

Run the following command.

$ sequelize model:create --name Todo --attributes title:string
This will generate a todo.js file in the server/models folder as well as a <date>-create-todo.js migration file in the server/migrations folder. <date> will be the date the model was generated.

The generated Todo model code is:
```
'use strict';
module.exports = function(sequelize, DataTypes) {
  var Todo = sequelize.define('Todo', {
    title: DataTypes.STRING
  }, {
    classMethods: {
      associate: function(models) {
        // associations can be defined here
      }
    }
  });
  return Todo;
};
```
In this file, we are defining our Todo model. It's going to have a single attribute, title, that is a String.

If you inspect the two generated files, you'll realize that they are in ES5. We're going to refactor them into ES6, for consistency with the rest of our project. Before we do that, let's generate our TodoItem model.

$ sequelize model:create --name TodoItem --attributes content:string,complete:boolean
In addition to refactoring, we're going to be editing the generated model fields a little bit to better suit our needs. We are also going to be defining the relationships between our models in the classMethods section of the generated model code.

After refactoring, editing the model fields and defining the relationships between our models, we arrive at:
```
server/models/todo.js

module.exports = (sequelize, DataTypes) => {
  const Todo = sequelize.define('Todo', {
    title: {
      type: DataTypes.STRING,
      allowNull: false,
    },
  }, {
    classMethods: {
      associate: (models) => {
        Todo.hasMany(models.TodoItem, {
          foreignKey: 'todoId',
          as: 'todoItems',
        });
      },
    },
  });
  return Todo;
};
```
Notice that we edited the title field and added a not-null constraint. This means that the database will not allow us to write to it if we don't provide a value for the title field. We also defined the relationship between a Todo and it's TodoItems in the classMethods.associate method. The as: 'todoItems means that every time we query for a todo and include it's todo items, they'll be included under the key todoItems instead of TodoItems (Sequelize defaults to using the model name). We're going to see how to make that inclusion a little later. Personally, I think it looks better this way.

server/models/todoitem.js
```
module.exports = (sequelize, DataTypes) => {
  const TodoItem = sequelize.define('TodoItem', {
    content: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    complete: {
      type: DataTypes.BOOLEAN,
      defaultValue: false,
    },
  }, {
    classMethods: {
      associate: (models) => {
        TodoItem.belongsTo(models.Todo, {
          foreignKey: 'todoId',
          onDelete: 'CASCADE',
        });
      },
    },
  });
  return TodoItem;
};
```
Notice that we've edited both the content and complete fields. We've added a not-null constraint in the content field and a default value for the complete field. In general, having a default value means that if we don't provide a value for that field when creating it, the database is going to use the provided default value for that field. In addition to that, we've also defined the relationship between the TodoItems and the Todo objects. The onDelete: CASCADE tells Postgres that if we delete a todo, it's associated todo items should be deleted as well (cascade the delete action).

For consistency, we're also refactoring our migration files to ES6 and ending up with:

server/migrations/<date>-create-todo.js

module.exports = {
  up: (queryInterface, Sequelize) =>
    queryInterface.createTable('Todos', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      title: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
    }),
  down: (queryInterface /* , Sequelize */) => queryInterface.dropTable('Todos'),
};
server/migrations/<date>-create-todo-item.js
module.exports = {
  up: (queryInterface, Sequelize) =>
    queryInterface.createTable('TodoItems', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      content: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      complete: {
        type: Sequelize.BOOLEAN,
        defaultValue: false,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      todoId: {
        type: Sequelize.INTEGER,
        onDelete: 'CASCADE',
        references: {
          model: 'Todos',
          key: 'id',
          as: 'todoId',
        },
      },
    }),
  down: (queryInterface /* , Sequelize */) =>
    queryInterface.dropTable('TodoItems'),
};
When we run these migrations, the up function will be executed. It will take care of creating the table and it's associated columns for us. If, for whatever reason, we needed to rollback (undo) the migration, the down function would be executed and it would undo whatever the up function did, thus returning the our database to the same state it was in before we performed the migration.

These migrations are a representation of how we want our models to look like in the database. Notice we define the relationship between our models in the create-todo-item.js migration file as well. The todoId field was not automatically generated and we've had to manually define it. Sequelize automatically generates the id, createdAt and updatedAt fields for you. In addition to that, any time a model is saved, the updatedAt field is automatically updated to reflect the new update time.

With the models and migrations in place, we're now ready to persist the models to the database by running the migrations. To do this, we run the following command:

$ sequelize db:migrate
This will discover the migrations in our migrations folder and execute them. If you try running the same command again, it would not execute any migrations since it's clever enough to know that all of the current migrations have been executed.

For purposes of comparison, you can find the code to this section here.

# Creating Controllers and Routing

With our models in place, let's move on to creating the controllers. We're going to have two controllers, todosController and todoItemsController. The todosController will be responsible for creating, listing, updating and deleting todos, while the todoItemsController will be responsible for creating, updating and deleting todo items.

# Creating Todos

Create a todo.js file inside server/controllers/. Inside this file, let's add the functionality to create todos.

server/controllers/todos.js
```
const Todo = require('../models').Todo;

module.exports = {
  create(req, res) {
    return Todo
      .create({
        title: req.body.title,
      })
      .then(todo => res.status(201).send(todo))
      .catch(error => res.status(400).send(error));
  },
};
```
The above code snippet creates a new todo and if successful, it returns it. If it encounters an error, it returns that error instead. (Granted, this isn't the best way to handle these errors, but we'll go with it for now, for the sake of simplicity. ;))

This create function is designed to be a route handler for whichever Express route we'll choose to attach it to. The req parameter is the incoming request from the client. The res parameter is the response we're preparing to eventually send back to the client in response to their request :). All Express route handlers follow this method signature. We can have a third parameter, conventionally named next, which is a function that passes the request on to the next route handler (meaning that a route can be handled by multiple route handlers, in which case it's piped or passed along all of those route handlers). We are, however, not going to see a use case for that in this application :(.

Next, we create an index.js file inside server/controllers, where we're going to be exporting our controllers from. I find this helpful since it helps me consolidate my imports (require statements) from once central place.

server/controllers/index.js
```
const todos = require('./todos');

module.exports = {
  todos,
};
```
Next, we need to add an API route that maps to this functionality. Create a routes folder inside the server folder. Inside the new routes folder, create an index.js file. We are going to place all our routes in this index.js file. However, in a real-world application, you might want to split up your routes and place then in different folders.

Inside server/routes/index.js, add the following code:
```
const todosController = require('../controllers').todos;

module.exports = (app) => {
  app.get('/api', (req, res) => res.status(200).send({
    message: 'Welcome to the Todos API!',
  }));

  app.post('/api/todos', todosController.create);
};
```
This will add two new routes, a welcome route at /api and a route to create todos at /api/todos. When we hit /api, we are instructing our application to send back a JSON object welcoming the user to our life-changing Todos API. If we post some data to /api/todos, we are telling our application to run the todosController.create function, which will take the request object, extract the posted data and create a todo from it. In this case, we say that the todosController.create function is the POST route handler for the /api/todos endpoint.

Next, we need to make the application aware that we just added the routes. Open up your app.js. We're going to be adding a require statement right before the route we'd earlier created, such that our app.js file now looks like:

app.js
```
const express = require('express');
const logger = require('morgan');
const bodyParser = require('body-parser');

const app = express();
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

// Require our routes into the application.
require('./server/routes')(app);
app.get('*', (req, res) => res.status(200).send({
  message: 'Welcome to the beginning of nothingness.',
}));

module.exports = app;
```
Note that we have to require our routes before the app.get('*', ...) catch-all route we'd added earlier. This is because the catch-all route will match any route and serve the welcome message, hence if we require our other routes after it, those other routes will never be hit.

Next, we open up Postman and issue a POST request to create a new todo item as in the image below.

Listing todos

Next, we're going to add functionality to list all todos. Add the following code snippet to your todosController after the create method.

server/controllers/todos.js
```
...
list(req, res) {
  return Todo
    .all()
    .then(todos => res.status(200).send(todos))
    .catch(error => res.status(400).send(error));
},
...
```
In this code snippet, we're fetching all todos from our database and sending them back to the user as an array in the response. If we encounter an error while fetching the todos from the database, we send that error object instead.

Next, open up server/routes/index.js and create a new url that maps a todos GET request to the list method right below the POST route we'd added earlier.

server/routes/index.js
```
...
app.post('/api/todos', todosController.create);
app.get('/api/todos', todosController.list);
...
```
Open up Postman and try out this new route.

List Todos

If you inspect the output, you'll realise that our listed todos do not have any todoitems. Let's add functionality to create todoitems next, after which we'll modify our list method to return todos together with their todoitems.

# Creating todo Items

Create a todoitems.js file inside your controllers directory. In this file, let's add functionality to create a todoitem.

server/controllers/todoitems.js
```
const TodoItem = require('../models').TodoItem;

module.exports = {
  create(req, res) {
    return TodoItem
      .create({
        content: req.body.content,
        todoId: req.params.todoId,
      })
      .then(todoItem => res.status(201).send(todoItem))
      .catch(error => res.status(400).send(error));
  },
};
```
server/controllers/index.js
```
const todos = require('./todos');
const todoItems = require('./todoitems');

module.exports = {
  todos,
  todoItems,
};
```
In the above code snippet, we're creating a todoitem and associating it with a particular todo. We are grabbing the id of that particular todo from the request params. We are also adding the todoItems controller to our default exports. Notice that we're using the ES6 object shorthand notation to add the methods to module.exports.

Let's set up the route for creating a new todoitem and see how the todoId is specified. Open up your routes file and add the following route:

server/routes/index.js
```
...
app.post('/api/todos/:todoId/items', todoItemsController.create);
...
```
The :todoId in the route is made available to us by Express in the request.params object as todoId and is the same one we're accessing in our controller. Do not forget to require todoItems controller as todoItemsController at the top of your routes file.

With this new route in place, we can go ahead and try it out in Postman.

Create a todo item

# Listing todo-items inside todos

Now that we can create todo items, let's modify our todosController.list code so that it returns a todo together with it's associated items.
```
server/controllers/todos.js

...
list(req, res) {
  return Todo
    .findAll({
      include: [{
        model: TodoItem,
        as: 'todoItems',
      }],
    })
    .then(todos => res.status(200).send(todos))
    .catch(error => res.status(400).send(error));
},
...
```
In the above code snippet, we find all todos and include all associated todoitems from the TodoItem model. We include them as todoItems, as we did when defining the relationship in the Todo model. Remember to require the TodoItem model at the top of your server/controllers/todos.js file.

We can view the results by making a GET request to /api/todos in Postman:

List todos with associated todo items

# Retrieving a single todo

Next, we're going to add functionality to retrieve one todo based on it's id. Let's add the following code inside server/controllers/todos.js

server/controllers/todos.js
```
...
retrieve(req, res) {
  return Todo
    .findById(req.params.todoId, {
      include: [{
        model: TodoItem,
        as: 'todoItems',
      }],
    })
    .then(todo => {
      if (!todo) {
        return res.status(404).send({
          message: 'Todo Not Found',
        });
      }
      return res.status(200).send(todo);
    })
    .catch(error => res.status(400).send(error));
},
...
```
In the code snippet above, we're finding the todo whose id matches the todoId we get from the request parameters and we're also including it's associated todo items. If such a todo exists, we're sending it back in the response. If it doesn't, we're sending back an error message letting the user know we didn't find the specified todo. If we encounter an error when processing this request, we're sending back the error object.

Then add a new route that maps to the retrieve view:

server/routes/index.js
```
...
app.get('/api/todos/:todoId', todosController.retrieve);
...
If you make a GET request to /api/todos/1using Postman, you should see the todo with id 1, with it's todo-items included in an array as well.

Retrieve a single todo

# Updating a single todo

Let's now add functionality to update a single todo:

server/controllers/todos.js

...
```
update(req, res) {
  return Todo
    .findById(req.params.todoId, {
      include: [{
        model: TodoItem,
        as: 'todoItems',
      }],
    })
    .then(todo => {
      if (!todo) {
        return res.status(404).send({
          message: 'Todo Not Found',
        });
      }
      return todo
        .update({
          title: req.body.title || todo.title,
        })
        .then(() => res.status(200).send(todo))  // Send back the updated todo.
        .catch((error) => res.status(400).send(error));
    })
    .catch((error) => res.status(400).send(error));
},
...
```
In the code snippet above, we're finding the todo whose id matches the todoId supplied in the request params. We are then updating it's title. If no title was provided, we're defaulting to the title the todo already had.

We also need to add a new route that maps to the update method:

server/routes/index.js
```
...
app.put('/api/todos/:todoId', todosController.update);
...
```
You can issue a PUT request using Postman to practically test this route:

Updating a todo

# Deleting todos

Finally, let's add functionality to delete todos:

server/controllers/todos.js
```
...
destroy(req, res) {
  return Todo
    .findById(req.params.todoId)
    .then(todo => {
      if (!todo) {
        return res.status(400).send({
          message: 'Todo Not Found',
        });
      }
      return todo
        .destroy()
        .then(() => res.status(204).send())
        .catch(error => res.status(400).send(error));
    })
    .catch(error => res.status(400).send(error));
},
...
```
This code is almost the same as the one we had for updating a todo, except we're not including the todo items. Remember that when you delete a todo, it's corresponding todo items are deleted as well. This is because we specified the onDelete action as CASCADE when we were setting up our models.

Then add the corresponding route:

server/routes/index.js
```
...
app.delete('/api/todos/:todoId', todosController.destroy);
...
```
If you try this out in Postman, you might be surprised that you don't get any data back. You can modify the delete code to return a 200 status code and a delete successful message as shown in the code snippet below:

...
  return todo
    .destroy()
    .then(() => res.status(200).send({ message: 'Todo deleted successfully.' }))
    .catch(error => res.status(400).send(error));
...
Personally, I prefer returning 204 No Content.

# Updating and Deleting Todo Items

Having gone through updating and deleting todos, it'll be a breeze going through updating and deleting todo items since the code is very similar. As such, we're going to do it all in one go.

Add the following code to your todoItemsController.

server/controllers/todoitems.js
```
...
update(req, res) {
  return TodoItem
    .find({
        where: {
          id: req.params.todoItemId,
          todoId: req.params.todoId,
        },
      })
    .then(todoItem => {
      if (!todoItem) {
        return res.status(404).send({
          message: 'TodoItem Not Found',
        });
      }

      return todoItem
        .update({
          content: req.body.content || todoItem.content,
          complete: req.body.complete || todoItem.complete,
        })
        .then(updatedTodoItem => res.status(200).send(updatedTodoItem))
        .catch(error => res.status(400).send(error));
    })
    .catch(error => res.status(400).send(error));
},

destroy(req, res) {
  return TodoItem
    .find({
        where: {
          id: req.params.todoItemId,
          todoId: req.params.todoId,
        },
      })
    .then(todoItem => {
      if (!todoItem) {
        return res.status(404).send({
          message: 'TodoItem Not Found',
        });
      }

      return todoItem
        .destroy()
        .then(() => res.status(204).send())
        .catch(error => res.status(400).send(error));
    })
    .catch(error => res.status(400).send(error));
},
...
```
Edit: As pointed out by good people in the comments, you'll notice that we're finding the todo item to either update or delete by two criteria: it's own id which we're grabbing from the params as todoItemId and the id of it's parent todo, which we're obtaining from the params object as todoId.

Let us, for a moment, focus on the update method. In the update method, we are grabbing the provided todoItemId from the request. We are then finding the todo item with that id and in readiness to update it. If we don't find it, we return early and send and error message to the user.

Earlier on, when we were updating the todo title, we had this statement:
```
...
.update({
  title: req.body.title || todo.title,
})
...
```
To recap, we said that we either use the new title or default to the old one if a title was not provided. You will notice the same pattern when we're updating the todo item in this statement:
```
...
.update({
  content: req.body.content || todoItem.content,
  complete: req.body.complete || todoItem.complete,
})
...
```
While this approach works for our application, since we have a small number of fields, it wouldn't scale very well if you had to update a model with many fields. As such, you might want to use another approach where you give the Sequelize model update function the data and then specify the fields it should update. Using this approach, we change change our .update statement to:
```
...
.update(req.body, { fields: Object.keys(req.body) })
...
```
To recap, using this approach, we pass the whole update object we get from the request (req.body) to the update function. Using ES6's Object.keys function, we extract the keys from the update object and tell the TodoItem Sequelize model to only update the fields that are present in the update data object. If we have a field in our model that's missing from the update object, the update operation will leave that field untouched. This saves us the trouble of having to define defaults using the || operator. You can read more about updating models in these Sequelize docs.

The destroy method finds a todo item with the stipulated id and deletes it.

Finally, we will add the two new routes in our routes file, right below the route to create todo items. Our complete routes file now looks like:

server/routes/index.js
```
const todosController = require('../controllers').todos;
const todoItemsController = require('../controllers').todoItems;

module.exports = (app) => {
  app.get('/api', (req, res) => res.status(200).send({
    message: 'Welcome to the Todos API!',
  }));

  app.post('/api/todos', todosController.create);
  app.get('/api/todos', todosController.list);
  app.get('/api/todos/:todoId', todosController.retrieve);
  app.put('/api/todos/:todoId', todosController.update);
  app.delete('/api/todos/:todoId', todosController.destroy);

  app.post('/api/todos/:todoId/items', todoItemsController.create);
  app.put('/api/todos/:todoId/items/:todoItemId', todoItemsController.update);
  app.delete(
    '/api/todos/:todoId/items/:todoItemId', todoItemsController.destroy
  );

  // For any other request method on todo items, we're going to return "Method Not Allowed"
  app.all('/api/todos/:todoId/items', (req, res) =>
    res.status(405).send({
      message: 'Method Not Allowed',
  }));
};
```
This was a long post but yay! We're done! Well, almost. :)

There are some improvements you could decide to make, for example:

Better error handling. Currently, we're assuming that all errors are due to the data the user has provided. We're also sending back the whole error object. That could be a security issue since you might leak information about your architecture to the end user.
Form fields validation. We currently have no front-facing input fields validation. Whenever you're building a web application, it's imperative that you validate user input before it hits the database. Our current validation (not null constraint) occurs at the database level. One way of performing this validation would be by intercepting the request in a middleware and validating that it contains the required fields.
Thanks.

