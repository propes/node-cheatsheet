### Globals


```js
console.log();
setTimeout();
clearTimeout();
setInterval();
clearInterval();

// Equivalents
global.console.log();
global.setTimeout();
...

```

### Modules


Every file is a module and is wrapped by the runtime with an IIFE:
```js
(function (exports, require, module, __filename, __dirname) {
    // file details
})
```

Log module details
```js
console.log(module);
```

Exporting from module
```js
var foo = function() {
}

module.exports.foo = foo; //or
exports.foo = foo;

// Can do
module.exports = foo;
// But not
exports = foo; // because exports is a reference to module.exports
```


### Paths


```js
const path = require('path');

var pathObj = path.parse(__filename);
```


### Events

```js
const EventEmitter = require('events);
const emitter = new EventEmitter();

// Register a listener
emitter.addListener(...) // or
emitter.on('messageLogged', function() {
   console.log('listener called');
});

// Raise event
emitter.emit('messageLogged');
```


### NPM

```sh
npm list // list all packages
npm list --depth=0 // list only direct dependencies
npm view lodash // view package info
npm view lodash dependencies // view package dependencies
npm outdated // view outdated packages
npm update // update outdated packages

npm i underscore // install
npm un underscore

// All of the above apply for -g as well

// Up-versioning
npm version major
npm version minor
npm version patch
```


### Environment variables

```js
const port = process.env.PORT || 3000;
```


### Config
Package options:
- rc
- config

Setup:
```
npm i config
```

Folder structure in project:
```
config
├── default.json
├── development.json
├── production.json
└── custom-environment-variables.json
```

development.json:
```json
{
  "name": "My App - Development",
  "mail": {
    "host": "dev-mail-server"
  }
}
```

Setting the node env:
```sh
export NODE_ENV=development
```

Map to environment variables using 'custom-environment-variables.json':
```json
{
  "jwtPrivateKey": "appName_jwtPrivateKey" // Key is as appears in config files, value is the name of the environment variable
}
```

Getting config values:
```js
const config = require('config');

config.get('jwtPrivateKey');
```

If using an environment variable, have a catch in index.js:
```js
if (config.get('jwtPrivateKey')) {
  console.error("FATAL ERROR: jwtPrivateKey is not defined.")
  process.exit(1);
}
```


### Arrays - Sorting

```js
// The worst way - using built in sort function - sorts array in place

const people = [{
    { name: 'bilbo' },
    { name: 'frodo' } 
}];

people.sort((a, b) => a.name > b.name ? 1 : -1);

console.log(people);


// Better alternative - using lodash

const _ = require('lodash');

const people = [...]

// Ascending only
const sortedPeople = _.sortBy(people, p => p.name); // or
const sortedPeople = _.sortBy(people, ['name', 'age']);

// Ascending or descending
const sortedPeople = _.orderBy(people, ['name', 'age'], ['asc', 'desc']);

```

### Express

```js
const express = require('express');
const app = express();

app.use(express.json()); // Required middleware for parsing json strings to objects in post/put requests.

app.get('/', (req, res) => {
   res.send('Hello world');
});

app.get('/api/courses', (req, res) => {
   res.send([1, 2, 3]);
});

app.get('/api/courses/:id', (req, res) => {
   res.send(req.params.id);
});

app.listen(3000, () => console.log('Listening on port 3000...'));
```


### MongoDB

term:
```sh
npm i mongoose
```

code:
```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/mydb') // Database will automatically be created the first time we write to it
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.error('Could not connect to MongoDB...', err));

const courseSchema = new mongoose.Schema({
  name: String,
  author: String,
  tags: [ String ],
  date: { type: Date, default: Date.now },
  isPublished: Boolean
});

// Adding a method to the schema
courseSchema.methods.returnSomething = function() {
  ...
}


async function createCourse() {
  const Course = mongoose.model('course', courseSchema); // Course is a class. First arg is the name of the collection in MongoDB (should be singular even if collection name is plural in db).
  const course = new Course({
    name: 'Node.js course',
    author: 'dilbert,
    tags: [ 'node', 'backend' ],
    isPublished: true,
    price: Number
  });

  const result = await course.save();
  console.log(result);
}
createCourse();


async function getCourses() {
  const courses = await Course.find(); // Get all courses
  
  // Comparison query operators: eq, ne, gt, gte, lt, lte, in, nin
  const courses = await Course
    .find({ author: 'dilbert', isPublished: true, price: { $gt: 10, $lte: 20 }, qty: { $in: [10, 15, 20] } })
    .limit(10
    .sort({ name: 1, author: -1 }) // 1 for asc, -1 for desc
    .select({ name: 1, tags: 1 });

  const courses = await Course
    .find({ tags: 'backend' }); // Matching tag from array

  const courses = await Course
    .find({ tags: [ 'backend', 'frontend' ] }); // Matches courses that have both tags

  const courses = await Course
    .find({ tags: { $in: [ 'backend', 'frontend' ] }}) // Matches courses that have either tag

  const courses = await Course
    .sort('-name'); // Alternative sorting syntax, '-' for desc

  const courses = await Course
    .sort('name author'); // Alternative select syntax

  const courses = await Course
    .find({ author: 'dilbert' })
    .or([ { tags: 'backend' }, { isPublished: true } ]);

  const courses = await Course
    .find()
    .and([ { author: 'dilbert' }, { isPublished: true } ]); // Logically equivalent to .find({ author: 'dilbert', isPublished: true }) but has advanced applications

  const courses = await Course
    .find({ author: /^dil/ }); // String that starts with 'dil'.

  const courses = await Course
    .find({ author: /Bert$/i }); // String that ends with 'bert'. Add 'i' at end for case insensitive.

  const courses = await Course
    .find({ author: /.*Dilbert.*/i }); // Contains 'Dilbert', case insensitive

  console.log('Courses', courses);
}
getCourses();


async function getCourseCount() {
  const count = await Course
    .find({ author: 'dilbert' })
    .count();

  console.log('Count', count);
}
getCourseCount();


async function getCoursesWithPagination() {
  const pageNumber = 2;
  const pageSize = 10;

  const courses = await Course
    .find({ author: 'dilbert' })
    .skip((pageNumber - 1) * pageSize)
    .limit(pageSize);

  console.log(courses);
}
getCoursesWithPagination();


async function updateCourse(id) {
  // Query first method - best if business rules are required (see example below)

  const course = await Course.findById(id);
  if (!course) return;
  if (course.isPublished) return; // Example of business rule

  course.isPublished = true;
  course.author = 'gilbert';
  // Or
  course.set({
    isPublished: true,
    author: 'gilbert'
  });
  const result = await course.save();
  console.log(result);


  // Update first method - can update multiple documents in one go
  const result = await Course.update({ _id: id }, {
    $set: {
      author: 'gilbert',
      isPublished: false
    }
  });
  const result = await Course.update({ isPublished: false }, {...}); // update all unpublished courses
  console.log(result);

  // Update first and return object    
  const course = await Course.findByIdAndUpdate(id, {
    $set: {
      author: 'gilbert',
      isPublished: false
    }
  }, { new: true }); // Third argument returns new object. Drop arg to return old object.
  console.log(course);
}
updateCourse('id from mongodb');


async function removeCourse(id) {
  const result = await Course.deleteOne({ _id: id });
  const result = await Course.deleteMany({ isPublished: true });
  console.log(result);

  // Remove and return object
  const course = await Course.findByIdAndRemove(id); // Returns null if the course doesn't exist
  console.log(course);
}
removeCourse('id from mongodb');
```

### MongoDB - Validation

```js
const courseSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    minlength: 5,
    maxlength: 255,
    match: /pattern/
  },
  category: {
    type: String,
    enum: ['web', 'mobile', 'network'],
    lowercase: true // converts to lowercase
    trim: true
  }
  isPublished: true,
  price: {
    type: Number,
    required: function() => { return this.isPublished; } // cannot use arrow function here because 'this' must refer to surrounding object rather that the enclosing execution context
    min: 10,
    max: 200,
    get: v => Math.round(v); // custom getter - on read
    set: v => Math.round(v); // custom setter - on update
  },
  tags: {
    type: Array,
    validate: {
      validator: function(v) {
        return v && v.length > 0; // Array must have at least one element
      },
      message: 'A course should have at least one tag.'
    }
  }
});

// Implicit validation
async function createCourse() {
  const Course = mongoose.model('course', courseSchema);
  const course = new Course({});

  try {
    await course.save(); // Will throw exception
  }
  catch (ex) {
    for(field in ex.errors) {
      console.error(ex.errors[field]); // Or
      console.error(ex.errors[field].message);
    }
  }
}
createCourse();

// Manual validation
async function createAndValidate() {
  const Course = mongoose.model('course', courseSchema);
  const course = new Course {};

  try {
    await course.validate(); // returns void, throws exception if validation fails
  }
  catch (ex) {
    for(field in ex.errors) {
      console.error(ex.errors[field]);
    }
  }

  // To use value of validate have to use callback, e.g. (might be worth checking latest docs on this)
  course.validate((err) => {
    if (err) // Do something
  });
}
createAndValidate();


// Async validator
const courseSchema = new mongoose.Schema({
  ...
  tags: {
    type: Array,
    validate: {
      isAsync: true,
      validator: function(v, callback) {
        setTimeout(() => {
          const result = v && v.length > 0;
          callback(result);
        }, 1000);
      },
      message: 'A course should have at least one tag.'
    }
  }
  ...
});
```


### MongoDB - Document Relationships

```js
const Author = mongoose.model('author', new mongoose.Schema({
  name: String,
  bio: String,
  website: String
}));

const Course = mongoose.model('course, new mongoose.Schema({
  name: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'author'
  }
}));

const course = new Course({
  name: 'Node course',
  author: 'mongodb id'
});

async function listCourses() {
  const courses = await Course
    .find()
    .populate('author', 'name bio -_id') // include name and bio, exclude id
    .select('name author');
  console.log(courses);
}
listCourses();
```


### MongoDB - Embedding Documents

```js
const authorSchema = new mongoose.Schema({
  name: String,
  bio: String,
  website: String
});

const Author = mongoose.model('author', authorSchema);

const Course = mongoose.model('course, new mongoose.Schema({
  name: String,
  author: {
    type: authorSchema,
    required: true
  },
  authors: [ authorSchema ]
}));

const course = new Course({
  name: 'Node course',
  author: new Author({ name: 'dilbert' }),
  authors: [ new Author({ name: 'dilbert' }), new Author({ name: 'dolbert' })]
});

async function addAuthor(courseId, author) {
  const course = await Course.findById(courseId);
  course.authors.push(course);
  course.save();
}
addAuthor('courseid', new Author({ name: 'gilbert' }));

async function removeAuthor(courseId, authorId) {
  const course = await Course.findById(courseId);
  const author = course.authors.id(authorId);
  author.remove();
  course.save();
}
removeAuthor('courseid', 'authorid');
```


### MongoDB - Transactions

- 2 phase commits are the closest thing to transactions in RDMSes
- Can use package Fawn to take advantage of this

```js
const Fawn = require('fawn');

Fawn.init(mongoose);

new Fawn.Task()
  .save('rentals', rental) // First arg is name of the collection in mongodb
  .update('movies', { _id: movie._id }, {
    $inc: { numberInStock: -1 }
  })
  .remove(...)
  .run();
```


### MongoDB - ObjectID

12 bytes
- First 4 bytes: timestamp (date created)
- Next 3 bytes: machine identifier
- Next 2 bytes: process identifier
- Next 3 bytes: counter

Can sort by _id to sort by date created

Code:
```js
// Manually generate an ObjectID
const id = new mongoose.Types.ObjectId();

// Get the timestamp from an ObjectID
_id.getTimestamp();

// Validate an ObjectID
mongoose.Types.ObjectId.isValid('1234');
```


### Hashing

```js
const bcrypt = require('bcrypt');

async function run() {
  const salt = await bcrypt.genSalt(10);
  const hashed = await bcrypt.hash('1234', salt);
  console.log(salt);
  console.log(hashed);
}
run();
```


### JSON Web Tokens (jwt)
Setup:
```sh
npm i jsonwebtoken
```

Code:
```js
const jwt = require('jsonwebtoken');

const token = jwt.sign({ _id: this._id }, config.get('jwtPrivateKey'));

// Send with response
res.header('x-auth-token', token).send(...);

// Get from request
const token = req.header('x-auth-token');
if (!token) res.status(401).send('Access denied.No token provided.');

try {
  const decoded = jwt.verify(token, confg.get('jwtPrivateKey')); // throws exception if token is not valid
  req.user = decoded;
}
catch (ex) {
  res.status(400).send('Invalid token.');
}
```


### Logging
Setup:
```sh
npm i winston
```

Code:
```js
const winston = require('winston');

winston.add(winston.transports.File, { filename: 'logfile.log' });
// Built transports for Console, File, Http. Plugins available for MongoDB, CouchDB, redis, loggly.

winston.log('error', err.message, err); //or
winston.error(err.message, err); // Second arg is metadata

// Logging levels:
// - error
// - warn
// - info
// - verbose
// - debug
// - silly

// Manually throwing an error for testing:
throw new Error('Something went wrong'); // Error object includes stack trace
```

Logging to MongoDB:
Setup:
```sh
npm i winston-mongodb
```

Code:
```js
winston.add(winston.transports.MongoDB, { db: config.get('dbConnectionString') });

// Log only specified levels to db
winston.add(winston.transports.MongoDB, {
  db: config.get('dbConnectionString'),
  level: 'info' // Means everything above and including 'info' is logged (i.e. error, warn, info)
});
```


### Handling Uncaught Exceptions
```js
// For synchronous exceptions
process.on('uncaughtException', (ex) => {
  winston.error(ex.message, ex);
  process.exit(1);
});

// Better alternative
winston.handleExceptions(new winston.transports.File({ filename: 'uncaughtExceptions.log' }));

// For unhandled promises
process.on('unhandledRejection', (ex) => {
  winston.error(ex.message, ex);
  process.exit(1);

  // Or just
  throw ex // If using winston to handle uncaught exceptions
});
```
