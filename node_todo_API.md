#Building a todo web API

###Using Mongoose ORM(object relational mapping)
1. Install [**Mongoose**](http://mongoosejs.com/) by

  ```bash
  $ npm i mongoose -S
  ```
2. Connect to the local server and create a model with certain attributes, so mongoose knows how to store data

  ```javascript
  const mongoose = require('mongoose');

  // use the built-in Promise library
  mongoose.Promise = global.Promise;

  // mongoose maintains collections over time, so we don't have to use callback function as MongoClient do
  mongoose.connect('mongodb://localhost:27017/TodoApp');

  //create model with certain attributes
  const Todo = mongoose.model('Todo', {
    text: {
      type: String,
    },
    completed: {
      type: Boolean,
    },
    // createdAt is stored in ObjectID
    completedAt: {
      type: Number,
    },
  });
  ```
3. Create a new todo and insert it into the database

  ```javascript
  const newTodo = new Todo({
    text: 'Feed the cat',
    completed: true,
    //This field will be filtered out since it's not in the Todo model
    test: 123,
  });

  newTodo
    .save()
    .then(todo => console.log(todo))
    .catch(err => console.log('Unable to save todo', err));
  ```
4. [Validators](http://mongoosejs.com/docs/validation.html), types, defaults and [schemas](http://mongoosejs.com/docs/guide.html)

  ```javascript
  // we can create the schema object first
  const TodoSchema = new mongoose.Schema({
    text: {
      type: String,
      required: true,
      minlength: 1,
      trim: true,
    },
    completed: {
      type: Boolean,
      default: false,
    },
    // createdAt is stored in ObjectID
    completedAt: {
      type: Number,
      default: null,
    },
  });
  // and stack into the model
  const Todo = mongoose.model('Todo', TodoSchema);
  ```

5. Use **Postman**, which is a central tool for building a REST APIs, helps you create HTTP request and fire them off.

6. Refactor and Rearrange your file structure. Put todo.js and user.js inside the models folder, and mongoose.js into db folder. So server.js is only responsible for managing routes. View on [**my repo**](https://github.com/kephin/Node_Todo-API).

###CRUD with Mongoose and Test
1. **[Create]** Resourse creation - Post /todos. We need to install **express** and [**body-parser**](https://github.com/expressjs/body-parser) middleware(Parse incoming request bodies in a middleware before your handlers, in another word, it let us send JSON to the server)

  :mag: You can find extra information below

  |[Resourse naming](http://www.restapitutorial.com/lessons/restfulresourcenaming.html)|[HTTP Status Codes](http://www.restapitutorial.com/httpstatuscodes.html)|
  |---|---|

  ```javascript
  // ./server/db/mongoose.js
  const mongoose = require('mongoose');

  mongoose.Promise = global.Promise;
  mongoose.connect('mongodb://localhost:27017/TodoApp');

  module.exports = {
    mongoose,
  };

  // ./server/models/todo.js
  const mongoose = require('mongoose');

  const TodoSchema = new mongoose.Schema({
    text: {
      type: String,
      required: true,
      minlength: 1,
      trim: true,
    },
    completed: {
      type: Boolean,
      default: false,
    },
    completedAt: {
      type: Number,
      default: null,
    },
  });

  const Todo = mongoose.model('Todo', TodoSchema);

  module.exports = {
    Todo,
  };

  // ./server/models/user.js
  const mongoose = require('mongoose');
  
  const UserSchema = new mongoose.Schema({
    email: {
      type: String,
      required: true,
      trim: true,
      minlength: 1,
    },
  });

  const User = mongoose.model('User', UserSchema);

  module.exports = {
    User,
  };

  // ./server/server.js
  const express = require('express');
  const bodyParser = require('body-parser');

  const { mongoose } = require('./db/mongoose');
  const { Todo } = require('./models/todo');
  const { User } = require('./models/user');

  const app = express();
  app.use(bodyParser.json());

  app.post('/todos', (req, res) => {
    // console.log(req.body);
    const todo = new Todo(req.body);

    todo
      .save()
      //status(200) is default
      .then(newTodo => res.send(newTodo))
      .catch(err => res.status(400).send(err));
  });

  app.listen(3000, () => console.log('Start on port 3000...'));
  ```
2. Test POST /todos

  |Test case|Expect to receive|
  |---|---|
  |if send correct data|status 200 with completed document including id|
  |if send bad data|status 400 and error object|

  Install dependent packages(**expect** for assertions; **mocha** for entire test suites; **supertest** to test HTTP requests) by

  ```bash
  $ npm i mocha expect supertest nodemon -D
  ```

  ```javascript
  // ./server/tests/server.test.js
  const expect = require('expect');
  const request = require('supertest');

  const { app } = require('./../server');
  const { todo } = require('./../models/todo');

  //add testing life cycle method: beforeEach, which let us run some code before every single test case
  //here we want to make sure DB is empty before any test
  beforeEach(done => {
    Todo
      .remove({})
      .then(() => done());
  });

  describe('POST /todos', () => {
    it('should create a new todo with valid data', (done) => {
      const text = 'new todo for testing';

      request(app)
        .post('/todos')
        //send data
        .send({ text })
        //assert if the request status is 200
        .expect(200)
        //assert if the respond body has a text property equals to the one we sent
        .expect(res => {
          expect(res.body.text).toBe(text);
        })
        .end((err, res) => {
          if (err) return done(err)
          //assert if the data got stored in mongoDB collection
          Todo
            .find()
            .then(todos => {
              expect(todos.length).toBe(1);
              expect(todos[0].text).toBe(text);
              done();
            })
            .catch(err => done(err));
        });
    });

    it('should NOT create todo with invalid body data', (done) => {
      request(app)
        .post('/todos')
        .send({})
        .expect(400)
        .end((err, res) => {
          if (err) return done(err);

          Todo
            .find()
            .then(todos => {
              expect(todos.length).toBe(0);
              done();
            })
            .catch(err => done(err));
        });
    });

  });
  ```
3. **[Review]** GET /todos

  ```javascript
  // ./server/server.js
  app.get('/todos', (req, res) => {
    Todo
    .find()
    .then(todos => res.send({ todos }))
    .catch(err => res.status(400).send(err));
  });
  ```
4. Test GET /todos

  |Test case|Expect to receive|
  |---|---|
  |if request for all docs|status 200 with 2 docs|

  ```javascript
  // ./server/tests/server.test.js
  const testTodos = [{
    text: 'Testing data #1',
  }, {
    text: 'Testing data #2',
  }];

  //now we want to make DB has some data before the test
  beforeEach(done => {
    Todo
      .remove({})
      .then(() => Todo.insertMany(testTodos))
      .then(() => done());
  });

  describe('GET /todos', () => {
    it('should list all todos', (done) => {
      request(app)
        .get('/todos')
        .expect(200)
        .expect(res => {
          expect(res.body.todos.length).toBe(2);
        })
        .end(done);
    });
  });
  ```
5. Mongoose queries and ID validation

  ```javascript
  // playground/mongoose-queies.js
  const { ObjectID } = require('mongodb');
  const { mongoose } = require('./../server/db/mongoose');
  const { Todo } = require('./../server/models/todo');

  const id = '5890488d4f2946281029c7c3';

  if (!ObjectID.isValid(id)) console.log('ID not valid');

  Todo
  // grab all and returns an array of objects
    .find({
      // mongoose will convert string id to the ObjectID for you
      // _id: id,
      completed: false,
    })
    .then(todos => console.log(todos));

  Todo
  //grab the first query, which only returns one object
    .findOne({
      // _id: id,
      completed: false,
    })
    .then(todos => console.log(todos));

  Todo
  // grab data with specific ID and returns it
    .findById(id)
    .then(todo => {
      // for valid ID, but not found -> return Null
      if (!todo) return console.log('ID not found.');
      console.log(todo);
    })
    // for invalid ID -> return error
    .catch(err => console.log(err));
  ```
6. **[Review]** Getting an individual resourse - GET /todos/:id

  > Key point: How to fetch a variable that gets passed via URL?

  ```javascript
  // server/server.js

  // set a variable, named 'id', followed by semicolon
  app.get('/todos/:id', (req, res) => {
    // then we can get an object with id property by req.params
    res.send(req.params);
  });

  app.get('/todos/:id', (req, res) => {
    const id = req.params.id;
    if (!ObjectID.isValid(id)) return res.status(404).send();

    Todo
      .findById(id)
      .then(todo => {
        if (!todo) return res.status(404).send();
        res.send({ todo });
      })
      .catch(err => res.status(404).send(err));
  });
  ```
7. Test GET /todos/:id

  First, we need to add **_id property** in the testTodos, so that we can fetch by _id
  ```javascript
  // ./server/tests/server.test.js
  const { ObjectID } = require('mongodb');

  const testTodos = [{
    _id: new ObjectID(),
    text: 'Testing data #1',
  }, {
    _id: new ObjectID(),
    text: 'Testing data #2',
  }];
  ```

  |Test case|Expect to receive|
  |---|---|
  |if request for individual doc by correct ID|status 200 with one we sent|
  |if request for individual doc by correct ID but doesn't exist |status 404|
  |if request for individual doc by invalid ID|status 404|

  ```javascript
  // ./server/tests/server.test.js
  describe('GET /todos/:id', () => {
    it('should return todo', (done) => {
      request(app)
      // trasform an ObjectID to string
        .get(`/todos/${testTodos[0]._id.toHexString()}`)
        .expect(200)
        .expect(res => {
          expect(res.body.todo.text).toBe(testTodos[0].text);
        })
        .end(done);
    });

    it('should return 404 if todo not found', (done) => {
      const testID = new ObjectID();
      request(app)
        .get(`/todos/${testID.toHexString()}`)
        .expect(404)
        .end(done);
    });

    it('should return 404 if invalid id', (done) => {
      request(app)
        .get('/todos/123')
        .expect(404)
        .end(done);
    });
  });
  ```
8. **[Destroy]** Delete a resource - DELETE /todos/:id

  ```javascript
  // ./server/server.js
  app.delete('/todos/:id', (req, res) => {
    const id = req.params.id;
    if (!ObjectID.isValid(id)) return res.status(404).send();

    Todo
      .findByIdAndRemove(id)
      .then(todo => {
        if (!todo) return res.status(404).send();
        res.send({ todo });
      })
      .catch(err => res.status(400).send(err));
  });
  ```
9. Test DELETE /todos/:id

  |Test case|Expect to receive|
  |---|---|
  |if request to remove a doc|status 200 with that deleted doc and also doesn't exist in db|
  |if request to remove a doc by correct ID but doesn't exist |status 404|
  |if request to remove a doc by invalid ID|status 404|

  ```javascript
  // ./server/tests/server.test.js
  describe('DELETE /todos/:id', () => {
    it('should remove a todo', (done) => {
      request(app)
        .delete(`/todos/${testTodos[1]._id.toHexString()}`)
        .expect(200).expect(res => {
          expect(res.body.todo.text).toBe(testTodos[1].text);
        })
        .end((err, res) => {
          if (err) return done(err);
          // check if the deleted one is gone
          Todo
            .findById(testTodos[1]._id.toHexString())
            .then(todo => {
              expect(todo).toNotExist();
              done();
            })
            .catch(err => done(err));

          // or check the count of total documents
          // Todo
          //   .find()
          //   .count()
          //   .then(count => {
          //     expect(count).toBe(1);
          //     done();
          //   })
          //   .catch(err => done(err));
        });
    });

    it('should return 404 if todo not found', (done) => {
      const testID = new ObjectID();
      request(app)
        .delete(`/todos/${testID.toHexString()}`)
        .expect(404)
        .end(done);
    });

    it('should return 404 if invalid id', (done) => {
      request(app)
        .delete('/todos/123')
        .expect(404)
        .end(done);
    });
  });
  ```
10. **[Update]** Update a resource - PATCH /todos/:id

  First, we need [**lodash**](https://github.com/lodash/lodash) because of the object filter function, call *pick()*

  ```bash
  $ npm i lodash -S
  ```

  ```javascript
  // ./server/server.js
  app.patch('/todos/:id', (req, res) => {
    const id = req.params.id;
    // screen out properties that shouldn't be touched by user
    const body = _.pick(req.body, ['text', 'completed']);

    if (!ObjectID.isValid(id)) return res.status(404).send();

    // logic between completed and completedAt
    if (typeof body.completed === 'boolean' && body.completed) {
      body.completedAt = new Date().getTime();
    } else {
      body.completed = false;
      body.completedAt = null;
    }

    Todo
      .findByIdAndUpdate(id, { $set: body }, { new: true })
      .then(todo => {
        if (!todo) return res.status(404).send();
        res.send({ todo });
      })
      .catch(err => res.status(400).send(err));
  });
  ```
11. Test PATCH /todos/:id

  |Test case|Expect to receive|
  |---|---|
  |if request to update a doc|status 200 and update correctly and screen-out other properties|
  |if request to update a doc with completed = false|status 200 and completedAt wil set to null|

  ```javascript
  // ./server/tests/server.test.js
  describe('PATCH /todos/:id', () => {
    it('should update the todo', (done) => {
      const patchData = {
        text: 'learn node',
        completed: true,
        shouldNotExist: 'wrong data',
      };

      request(app)
        .patch(`/todos/${testTodos[0]._id.toHexString()}`)
        .send(patchData)
        .expect(200)
        .expect(res => {
          expect(res.body.todo.text).toBe(patchData.text);
          expect(res.body.todo.completed).toBe(patchData.completed);
          expect(res.body.todo.completedAt).toBeA('number');
          expect(res.body.todo.shouldNotExist).toNotExist();
        })
        .end((err, res) => {
          if (err) return done(err);

          Todo
            .findById(testTodos[0]._id.toHexString())
            .then(todo => {
              expect(todo.text).toBe(patchData.text);
              expect(todo.completed).toBe(patchData.completed);
              expect(todo.completedAt).toBeA('number');
              expect(todo.shouldNotExist).toNotExist();
              done();
            })
            .catch(err => done(err));
        });
    });

    it('should clear completedAt when todo is not completed', (done) => {
      request(app)
        .patch(`/todos/${testTodos[0]._id.toHexString()}`)
        .send({
          completed: false,
        })
        .expect(200)
        .expect(res => {
          expect(res.body.todo.completed).toBe(false);
          expect(res.body.todo.completedAt).toNotExist();
        })
        .end((err, res) => {
          if (err) return done(err);

          Todo
            .findById(testTodos[0]._id.toHexString())
            .then(todo => {
              expect(todo.completed).toBe(false);
              expect(todo.completedAt).toNotExist();
              done();
            })
            .catch(err => done(err));
        });
    });
  });
  ```

###Deploy Todo API to Heroku
1. Change port 3000 to environment variable:

    ```javascript
    // ./server/server.js
    const port = process.env.PORT || 3000;

    app.listen(port, () => {
      console.log(`Listening on port ${port}`);
    });
    ```
2. Add `npm start` for Heroku and specify node version

    ```javascript
    // package.json
    {
      "scripts": {
        "start": "node server/server.js",
        "test": "mocha server/**/*.test.js",
        "test-watch": "nodemon --exec \"npm test\""
      },
      "engines": {
        "node": "6.8.1"
      },
    }
    ```
3. Set up database with Heroku add-on

    ```bash
    $ heroku create

    # configure mongolab add-ons with our app
    $ heroku addons:create mongolab:sandbox

    # we can see MONGODB_URI
    $ heroku config
    ```
    So we need to set our mongoose connection to MONGODB_URI

    ```javascript
    // ./server/db/mongoose.js
    mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/TodoApp');
    ```
4. Finally

    ```bash
    # push to Heroku
    $ git push heroku master

    # view logs
    $ heroku logs

    # open in the browser
    $ heroku open
    ```
5. Setting up a test database for testing, which we should have done at development

  We seperate test and development environments by **process.env.NODE_ENV**. In production, like Heroku, **process.env.NODE_ENV** is set to be 'production'. But in our local computer, **process.env.NODE_ENV** is not defined.

  So we can set *port number* and *database URI* by telling this parameter is 'development' or 'test'. In production, we don't need to set them, because they are already defined by Heroku.
  ```javascript
  // package.json
  {
    "scripts": {
      "test": "export NODE_ENV=test || SET \"NODE_ENV=test\" && mocha server/**/*.test.js"
    }
  }

  // ./server/config/config.js
  const env = process.env.NODE_ENV || 'development';

  if (env === 'development') {
    process.env.port = 3000;
    process.env.MONGODB_URI = 'mongodb://localhost:27017/TodoApp';
  } else if (env === 'test') {
    process.env.port = 3000;
    process.env.MONGODB_URI = 'mongodb://localhost:27017/TodoAppTest';
  }
  ```

