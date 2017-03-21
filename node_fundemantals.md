# Getting started with Node

### What is Node and the advantages?

1. Node.js is the javascript runtime that uses the V8 engine, which is the open-source javascript engine written in C++ that takes in *javascript code* and compile into the *machine code*.
2. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. **Callback function** makes Node.js non-blocking I/O model compared to other back-end languages.
3. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
4. **window** is the global object in browser, whereas **global** is the global object in Node. And the **document** becomes **process**.
5. Check node version `$ node -v`

### DRY in Node

Using **require** to load module that comes with Node.js

```javascript
const fs = require('fs');
const os = require('os');

const user = os.userInfo();
fs.appendFile('greetings.txt', `Hello ${user.username}!`, (err) => {
  if (err) throw err;
  console.log('The "data to append" was appended to file!');
})
```

Using **require** to load module that we defined

```javascript
//In module file, called notes.js
const add = (a, b) => a + b;
const minus = (a, b) => a - b;
module.exports = {
  add,
  minus,
}

//In app.js
const notes = require('./notes.js');
console.log(notes.add(1, 3)); // => 4
```

Using **require** to load 3rd party modules

+ [**lodash**](https://lodash.com/)

  First install locally, `$ npm install --save lodash`

  ```javascript
  const _ = require('lodash');

  const data = [1, 3, 'kevin', 4, 'john', 'kevin', 1, 2];
  console.log(_.uniq(data)); // => [1, 3, 'kevin', 4, 'john', 2]
  ```

+ Restart app automatically with [**nodemon**](https://github.com/remy/nodemon)

  1. Install globally by running `$ sudo npm install -g nodemon`
  2. Run `$ nodemon app.js`

### How to get input from user inside the command line?

```shell
$ node yourfile.js read # with argument
```

You can access inside the command line arguments by `process.argv`

```javascript
const command = process.argv;
console.log(command); // => [0] is the addres of Node, [1] is the file you're running on
// the rest are the command arguments
```

Simplified inputs with [**yargs**](https://github.com/yargs/yargs)

How to read more detail arguments like the following?

```shell
# with arguments
$ node yourfile.js --read
$ node yourfile.js -r
# more specific arguments
$ node yourfile.js --read=record_1
$ node yourfile.js --add="Node is awesome!" # Notice! Using double quotes instead of single quote
```

We're going to using yargs by `$ npm install yargs --save`. If we type `$ node app.js hello --post=hello` and have the .js file bellow:

```javascript
const argv = require('yargs').argv;
console.log(argv); // => { _: [ 'hello' ], post: 'hello', '$0': 'app.js' }
```

Options are going to be just a hash! And every non-hyphenated options will be an array inside `argv._`
    
### Working with JSON
    
:bulb: JSON.stringify(): Converts an **object** or an **array** to a JSON string

```javascript
const person = {
  name: 'kevin',
  age: 30,
}
const jsonString = JSON.stringify(obj);
console.log(typeof jsonString); // => string
console.log(jsonString); // => '{"name": "kevin", "age": 30}'
```

:bulb: JSON.parse(): Converts a JSON string into an **object** or an **array**

```javascript
const carString = '{"name": "BMW", "color": "white"}';
const car = JSON.parse(carString);
console.log(typeof car); // => object
console.log(car); // => {name: 'BMW', color: "white"}
```

Now, we can CRUD note with .json files:

```javascript
const addNote = (title, body) => {
  let notes = [];
  const newNote = {
      title,
      body,
  };

  try {
    const prevNotes = JSON.parse(fs.readFileSync('notes-data.json'));
    notes = prevNotes;
  } catch (error) {
    // ...
  }

  const nonDuplicatedNotes = notes.filter(note => note.title !== title);
  nonDuplicatedNotes.push(newNote);
  fs.writeFileSync('notes-data.json', JSON.stringify(nonDuplicatedNotes));
};
```

### Debugger node.js applications

1. Add `debugger;` inside your code
2. Run `$ node debug YOURFILE.js --argument=="hello"`
3. Press `n` to step next or `c` to directly go the **debugger**
4. Run `$ repl` and then you can inspect all the stuff in your program
5. Press `ctrl + c` to exit

:x: GUI debugging(Still not available..)
