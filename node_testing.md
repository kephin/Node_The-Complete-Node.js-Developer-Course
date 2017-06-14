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
