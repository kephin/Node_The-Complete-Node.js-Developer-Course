# MongoDB Fundemantals

### [Installation](https://www.mongodb.com/)

**Windows**

Create a folder named *mongo-data* to store local data

Inside the bin folder, we can start the MongoDB server by setting up the MongoDB environment with the folder path (To stop, press `Ctrl + c`)

```bash
$ E:\\MongoDB\\Server\\3.4\\bin\\mongod.exe --dbpath E:\\mongo-data
```

Next, open another tab of comman-line editor. Inside the bin, run the following command to connect to the local mongoDB database

```bash
$ E:\\MongoDB\\Server\\3.4\\bin\\mongo.exe
```

It will put us in a **command prompt-view** so that we are able to issue various mongo command to manipulate the data.

**Linux(Ubuntu)**

No need to create a folder for local data, MongoDB stores its data files in `/var/lib/mongodb` by default

After we start the server, we can check its log files inside `/var/log/mongodb`

```bash
$ sudo service mongod start
$ sudo service mongod stop
$ sudo service mongod restart
```

**After installation, we can manipulate with Mongo in terminal:**

We can execute commands, for example:

```bash
# Insert data
$ db.Todos.insert({text: 'Hello world'})
# Fetch data
$ db.Todos.find()
```

Install [**robomongo**](https://robomongo.org/), a GUI for mongoDB. It lets you view all the data and manipulate it.

:bulb: **Comparison between SQL and NoSQL**

| |SQL|NoSQL|
|---|---|---|
|Whole data|tabel|collection|
|A piece of data|row/record|document|
|Property(id, name..)|column|field|

### Manipulate data with mongoDB(CRUD)

Check documentation: [**MongoDB Node.JS Driver**](http://mongodb.github.io/node-mongodb-native/)

Install mongoDB inside the project

```bash
$ npm i mongodb -S
```

Using **MongoClient** to connect to mongo server and issue commands to manipulate the database

**[Create]** There is no need to create a collection(aka Table in SQL) before adding a document(aka row/record in SQL), should check [insertOne](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html#insertOne), [insertMany](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html#insertMany)

```javascript
const MongoClient = require('mongodb').MongoClient;

MongoClient.connect('mongodb://localhost:27017/TodoApp', (err, db) => {
  if (err) {
  return console.log('Unable to connect to MongoDB server!');
  }
  console.log('Connected to MongoDB server!');

  // here we can just insert data before even create a collection
  db.collection('Todos')
  .insertOne({
    text: 'Something to do',
    completed: false,
  })
  //ops argument is storing all of the documents we inserted
  .then(err => console.log(JSON.stringify(result.ops, undefined, 2));)
  .catch(res => console.log('Unable to insert todo', err))

  db.close();
});
```

What is objectID?
+ objectID stores timestamp, machine identifier(aka which computer you use), process ID and counter(random value)
+ we can log out timestamp from objectID:

  ```javascript
  console.log(JSON.stringify(result.ops[0]._id.getTimestamp()));
  ```

+ we can create on our own

  ```javascript
  const {ObjectID} = require('mongodb');

  const obj = new ObjectID;
  console.log(obj); //588b0d9fbfb33f2ef8578315
  ```

**[Review]** Fetch data, go check [find](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html#find), [toArray](http://mongodb.github.io/node-mongodb-native/2.2/api/Cursor.html#toArray), [count](http://mongodb.github.io/node-mongodb-native/2.2/api/Cursor.html#count)

```javascript
const { MongoClient, ObjectID } = require('mongodb');

MongoClient.connect('mongodb://localhost:27017/TodoApp', (err, db) => {
  if (err) {
    return console.log('Unable to connect to MongoDB server!');
  }
  console.log('Connected to MongoDB server!');
  
  db.collection('todos')
    // fetch all documents in Todo collection
    .find()
    .toArray()
    .then(docs => console.log(JSON.stringify(docs, undefined, 2)))
    .catch(err => console.log('Unable to fetch!', err));
  
  db.collection('Todos')
    // fetch documents with certain field -- just like filtering
    .find({ completed: false })
    .toArray()
    .then(docs => console.log(JSON.stringify(docs, undefined, 2)))
    .catch(err => console.log('Unable to fetch!', err));

  db.collection('Todos')
    // fetch documents with certain id
    .find({ _id: new ObjectID('588b0d9fbfb33f2ef8578315') })
    .toArray()
    .then(docs => console.log(JSON.stringify(docs, undefined, 2)))
    .catch(err => console.log('Unable to fetch!', err));

  db.collection('Todos')
    .find()
    // output the number of the documents
    .count()
    .then(count => console.log(`Todos: ${count}`))
    .catch(err => console.log('Unable to fetch!', err));

  //db.close();
});
```

**[Distroy]** Remove data, there are deleteMany, deleteOne, go and check [findOneAndDelete](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html#findOneAndDelete), which returns the deleted information

```javascript
db.collection('Todos')
  // target many documents and delete them
  .deleteMany({text: 'Eat lunch'})
  .then(result => console.log(result));


db.collection('Todos')
  // target one document and delete it
  .deleteOne({text: 'Eat lunch'})
  .then(result => console.log(result));

db.collection('Todos')
  // remove individual document and return it
  .findOneAndDelete({text: 'Eat lunch'})
  .then(results => console.log(JSON.stringify(results, undefined, 2)));
```

**[Update]** Just like [Distroy], it contains updateOne, updateMany, see more in [findOneAndUpdate](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html#findOneAndUpdate), also with [update operators](https://docs.mongodb.com/manual/reference/operator/update/)

```javascript
db.collection('Todos')
  // update the item and get the new document back
  .findOneAndUpdate({ _id: new ObjectID('588c14bfeac1f6b8e0b5f0fb') }, {
    // update operators
    $set: {
      text: 'walk the cat',
    },
  }, {
    returnOriginal: false,
  })
  .then(result => console.log(result));
```
