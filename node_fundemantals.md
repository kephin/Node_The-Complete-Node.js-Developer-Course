#Getting started with Node

###What is Node and the advantages?
1. Node.js is the javascript runtime that uses the V8 engine, which is the open-source javascript engine written in C++ that takes in *javascript code* and compile into the *machine code*.
2. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. **Callback function** makes Node.js non-blocking I/O model compared to other back-end languages.
3. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
4. **window** is the global object in browser, whereas **global** is the global object in Node. And the **document** becomes **process**.
5. `$ node -v`: Running node with the v flag

###Node fundemantals

1. Using **require** to load module that comes with Node.js

  ```javascript
  const fs = require('fs');
  const os = require('os');

  const user = os.userInfo();
  fs.appendFile('greetings.txt', `Hello ${user.username}!`, (err) => {
    if (err) throw err;
    console.log('The "data to append" was appended to file!');
  })
  ```
2. Using **require** to load module that we defined

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
3. Using **require** to load 3rd party modules

  First install locally, `$ npm install --save lodash`

  ```javascript
  const _ = require('lodash');
  
  const data = [1, 3, 'kevin', 4, 'john', 'kevin', 1, 2];
  console.log(_.uniq(data)); // => [1, 3, 'kevin', 4, 'john', 2]
  ```

4. Restart app automatically with [**nodemon**](https://github.com/remy/nodemon)

  1. Install globally by running `$ sudo npm install -g nodemon`
  2. Run `$ nodemon app.js`
5. Getting input from user inside the command line

  ```shell
  $ node yourfile.js read # with argument
  ```
  You can access inside the command line arguments by `process.argv`

  ```javascript
  const command = process.argv;
  console.log(command); // => [0] is the addres of Node, [1] is the file you're running on
  // the rest are the command arguments
  ```
6. Simplified inputs with [**yargs**](https://github.com/yargs/yargs)

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
7. Working with JSON
  
  + JSON.stringify(): Converts an **object** or an **array** to a JSON string
  ```javascript
  const person = {
    name: 'kevin',
    age: 30,
  }
  const jsonString = JSON.stringify(obj);
  console.log(typeof jsonString); // => string
  console.log(jsonString); // => '{"name": "kevin", "age": 30}'
  ```
  + JSON.parse(): Converts a JSON string into an **object** or an **array**
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

    }

    const nonDuplicatedNotes = notes.filter(note => note.title !== title);
    nonDuplicatedNotes.push(newNote);
    fs.writeFileSync('notes-data.json', JSON.stringify(nonDuplicatedNotes));
  };
  ```
8. Debugger node.js applications

  1. add `debugger;` inside your code
  2. run `$ node debug YOURFILE.js --argument=="hello"`
  3. press `n` to step next or `c` to directly go the **debugger**
  4. run `$ repl` and then you can inspect all the stuff in your program
  5. press `ctrl + c` to exit

9. GUI debugging(Still not available..)