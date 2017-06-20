# Testing in Node

### [Mocha](https://mochajs.org/) as testing framework

:mag: *Mocha is the simple, flexible, fun javascript test framework for node.js & the browser.*

:arrow_down: Install **mocha** by `$ npm i mocha -D`

Create a folder named **utils or test**. We will put our testings inside here by naming it **YOURFILE.test.js**

```javascript
// package.json
{
  "scripts": {
    "test": "mocha **/*.test.js",
    "test-watch": "nodemon --exec \"npm test\""
  },
}
```

Now, we can just run `$ npm test` to start the test or `$ npm run test-watch` to auto-refresh the test.

We can run some code before every test using **beforeEach()**

```javascript
describe('utils', () => {
  const user;
  // run before every test
  beforeEach(() => {
    user = new User({
      firstName: 'Kevin',
      lastName: 'Hsaio',
      age: 30,
    })
  });

  it('can extract it name', () => {
    // assertions...
  });
});
```

### **[Expect](https://github.com/mjackson/expect)** / **[Chai](http://chaijs.com/)** for assertions

:mag: *Expect / Chai lets you write better assertions.*

:arrow_down: Install **expect** by `$ npm i expect -D`

Example assertions:

```javascript
//util.js
module.exports.square = a => {
  if(typeof a === 'string') throw new Error('Only accept numbers.');
  return a * a;
};
module.exports.setName = (user, fullName) => {
  const names = fullName.split(' ');
  user.firstName = names[0];
  user.lastName = names[1];
  return user;
};

//util.test.js
describe('Utils', () => {
  it('should square a number', () => {
    //arrange
    const input = 14;
    const expected = 196;
    //act
    const actual = utils.square(14);
    //assert
    expect(actual).toBeA('number').toBe(expected);
  });

  it('should throw an error', () => {
    //arrange
    const input = 'This is a string';
    //act
    const actual = utils.square(14);
    //assert - test for errors
    expect(function () {
      utils.square(input);
    }).toThrow(Error);
  });

  it('should set first and last name', () => {
    let user = {
      age: 29,
    };
    user = utils.setName(user, 'Kevin Hsaio');
    expect(user).toBeA('object').toInclude({
      firstName: 'Kevin',
      lastName: 'Hsaio',
    });
  });
});
```

:bulb: To test asynchronous functions, we pass an **done** argument to the test function, and call done() once the tests are finished. Mocha will wait 2 seconds and cut the test.

```javascript
//util.js
module.exports.asyncSquare = (a, callback) => {
  setTimeout(() => {
    callback(a * a);
  }, 1000);
};

//util.test.js
it('should async square number', done => {
  utils.asyncSquare(15, square => {
    expect(square).toBeA('number').toBe(225);
    done();
  });
});
```

### **[SuperTest](https://github.com/visionmedia/supertest)** for integration test

:mag: *SuperTest is the super-agent driven library for testing node.js HTTP servers.*

:arrow_down: Install **supertest** by `$ npm i supertest -D`

To verify that if we make a http **get** request in the following url, we get 'Hello world' back. First, export app from server.js

```javascript
//server.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello world!');
});
//We can customize the status for your response
app.get('/bad', (req, res) => {
  res.status(404).send({
    error: 'Page not found!',
  });
});
app.listen(3000);

module.exports.app = app;

//server.test.js
const request = require('supertest');

const app = require('./server').app;

it('should return hello world response', (done) => {
  request(app)
    .get('/')
    .expect(200)
    .expect('Hello world!')
    .end(done);
});

it('should return page not found response', (done) => {
  request(app)
    .get('/bad')
    .expect(404)
    .expect({
      error: 'Page not found!',
    })
    .end(done);
});
```

Using **expect** library inside the **supertest** library. With **expect**, we can access headers, body...anything from the http response.

```javascript
//server.test.js
const request = require('supertest');
const expect = require('expect');

const app = require('./server').app;

it('should return hello world response', (done) => {
  request(app)
    .get('/')
    .expect(200)
    .expect((res) => {
      //Here! We are utilizing the expect library
      expect(res.body).toInclude({
        error: 'Page not found!',
        });
      })
    .end(done);
});
```

### Using **[spy](https://github.com/mjackson/expect#spy-tohavebeencalled)**

:mag: *Spy let you swap out a real function for a testing utility. When that test function gets called, you can create various assertion about it. Making sure it was called with certain arguments.*

For example, we have a function called `handleSignup()` in app.js, `handleSignup()` calls `saveUser()` from db.js. Now if we want to verify `handleSignup()`, we **just** want to test if `handleSignup()` calls `saveUser()`(and also with correct arguments), but **NOT** to test whether `saveUser()` works correctly.

```javascript
//db.js
module.exports.saveUser = (user) => {
  //...saving the user to the db
}

//app.js
// we have to use 'let' to declare because later we will need to modify the db object by 'rewire'
let db = require('./db');

module.exports.handleSingup = (email, password) => {
  //Check if email already exist
  // Save the user to the database
  db.saveUser({
    email,
    password,
  });
  //Send the welcome email
};
```

So we use **spy** to simulate/replace the functions inside `handleSignup()`, which in this case is `saveUser()`.

:arrow_down: install **[rewire](https://github.com/jhnns/rewire)** by `$ npm i rewire -D`

```javascript
//app.test.js
const expect = require('expect');
const rewire = require('rewire');

const app = rewire('./app');

describe('App', () => {
  const db = {
    saveUser: expect.createSpy(),
  };
  app.__set__('db', db);

  it('should call the spy correctly', () => {
    const spy = expect.createSpy();
    spy('kevin', 30);
    expect(spy).toHaveBeenCalled();
    expect(spy).toHaveBeenCalledWith('kevin', 30);
  });

  it('should call saveUser with user object', () => {
    const email = 'kevin@example.com';
    const password = '12345';

    app.handleSignup(email, password);
    expect(db.saveUser).toHaveBeenCalledWith({ email, password });
  });
});
```
