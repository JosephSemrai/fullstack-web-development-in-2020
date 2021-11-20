Web Development 3

# TODO

# TALK ABOUT DOCKER

As our product scales in complexity, you'll find that our project quickly becomes hard to maintain with regard to the organization of our files. This leads to the following project structure that most Express projects follow:

```jsx
├── index.js
├── app.js
├── build
│   ├── ...
├── controllers
│   ├── notes.js
│   └── ...
├── models
│   ├── note.js
│   └── ...
├── requests
│   └── ...
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   ├── middleware.js
│   └── ...
```

You'll notice the addition of a few new folders and files in our root directory here. Let's start discussing their  functions and the various changes that we'll have to make to our notes application.

First, one change that we'll make is to put everything that isn't involved with starting our application into **app.js**. Then, **index.js** will import our `app` and start the server using that component as follows

**/Backend/index.js**

```js
const app = require('./app') // Our `app` module that is started by this file
const http = require('http')
const config = require('./utils/config')

const server = http.createServer(app) // Creates the server using the `app` module

server.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT}`)
})
```

Here, you'll notice references to a `config` constant that seems to be imported from the file **./utils/config**.

**/Backend/utils/config.js**

```js
require('dotenv').config()

let PORT = process.env.PORT
let MONGODB_URI = process.env.MONGODB_URI

module.exports = {
  MONGODB_URI,
  PORT
}
```

Here is where we handle our environment variables and export them for use in other modules via importing this configuration module.

We also moved the routes dealing with our `notes` end point to its own file under the **/controllers** directory. The handlers for our route methods are known as *controllers*, thus they will now fall under the **/controllers** directory for the sake of organization. We also take advantage of a few more Express features here as shown below

**/Backend/controllers/notes.js**

```js
const notesRouter = require('express').Router()
const Note = require('../models/note')

notesRouter.post('/', (req, res) => {
    const body = req.body

    if (!body.content) {
        return res.status(400).json({
            error: 'No content specified in note body'
        })
    }

    const newNote = new Note({
        content: body.content,
        flagged: body.flagged || false,
        date: new Date(),
    })

    newNote.save().then(savedNote => {
        res.json(savedNote.toJSON())
    })
        .catch(error => next(error))
})

notesRouter.get('/', (req, res) => {
    res.send('<h1>Welcome to our Notes Application Backend!</h1>')
})

notesRouter.get('/', (req, res) => {
    Note.find({}).then(notes => {
        res.json(notes.map(note => note.toJSON()))
    })
})

notesRouter.delete('/:id', (req, res, next) => {
    Note.findByIdAndRemove(req.params.id)
        .then((result) => {
            res.status(204).end()
        })
        .catch(error => next(error))
})

notesRouter.get('/:id', (req, res, next) => {
    Note.findById(req.params.id)
        .then(note => {
    	if (note) {
                res.json(note.toJSON())
            } else {
                res.status(404).end() // Condition fail
            }
  	})
  	.catch(error => next(error))
})

notesRouter.put('/:id', (req, res, next) => {
    const body = req.body

    const note = {
        content: body.content,
        flagged: body.flagged,
    }

    Note.findByIdAndUpdate(req.params.id, note, { new: true })
        .then(updatedNote => {
            res.json(updatedNote.toJSON())
        })
        .catch(error => next(error))
})

modules.exports = notesRouter
```

Here, you'll notice how we took in the `Note` model so that we could reference our database from within these controllers. We also took advantage of `Router` in Express. A `Router` is like its own mini Express application by providing us with context-specific routing APIs (like `get()` and `put()`). Routers are actually a piece of middleware which can be used for defining and associating related routes.

```js
const notesRouter = require('express').Router()

//...
```

This allows us to use routing methods of our `notesRouter` rather than `app`. Additionally, as we can define a base URL for our `notesRouter`, we can shorten our endpoints to reflect them being extensions of a specific base URL. For example, `/api/notes/:id` would turn into `/:id`.

We can then make use of this router in our `app` module where we also register the base URL for this router upon which all these endpoints are appended onto.

**/Backend/app.js**

```js
const config = require('./utils/config')
const express = require('express')
const bodyParser = require('body-parser')
const app = express()
const cors = require('cors')
const notesRouter = require('./controllers/notes')
const middleware = require('./utils/middleware')
const mongoose = require('mongoose')

console.log('Connecting to:', config.MONGODB_URI)

mongoose.connect(config.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false
})
  .then(() => {
    console.log('Connected to MongoDB')
  })
  .catch((error) => {
    console.log('Error connecting to MongoDB:', error.message)
  })

// Middleware
app.use(cors())
app.use(express.static('build'))
app.use(bodyParser.json())
app.use(middleware.requestLogger)

// Routers
app.use('/api/notes', notesRouter)

// After router middleware
app.use(middleware.unknownEndpoint)
app.use(middleware.errorHandler)

module.exports = app
```

In our app component, you'll notice quite a few changes from our app's last checkpoint. First, you'll notice that we now import all of our middleware from **/utils/middleware.js** to which it contains:

**/Backend/utils/middleware.js**

```js
const unknownEndpoint = (req, res) => {
    res.status(404).send({ error: 'Endpoint does not exist (Unknown Endpoint)' })
}

const errorHandler = (err, req, res, next) => {
    console.error(err.message)

    if (err.name === 'CastError' && err.kind === 'ObjectId') {
        return res.status(400).send({ error: 'Incorrect format for id parameter' })
    } else if (error.name === 'ValidationError') {
        return response.status(400).json({ error: error.message })
    }

    next(error)
}

module.exports = {
    unknownEndpoint,
    errorHandler
}
```

Then, you'll notice how we now connect to our database from within our `app` module as we only need to maintain one connection to the database for our entire app for now. Our other approach which involved connecting to the database for each model is not ideal. This results in an updated **/models/note.js** file since we no longer need to connect to our database within each module:

**/Backend/models/note.js**

```js
const mongoose = require('mongoose')

const noteSchema = new mongoose.Schema({
    content: {
        type: String,
        minlength: 1,
        required: true
    },
    date: {
        type: Date,
        required: true
    },
    flagged: Boolean,
})

noteSchema.set('toJSON', {
    transform: (document, returnedObject) => {
        returnedObject.id = returnedObject._id.toString()
        delete returnedObject._id
        delete returnedObject.__v
    }
})

module.exports = mongoose.model('Note', noteSchema)
```

This should mark all the changes with moving to a more structured project structure. Making good use of modules and organizing them will prove to make development for larger applications much easier. Keep in mind that the structure that we demonstrated here is only one of the many recommended project structures.

### Automated Testing (CHANGE THE CODE EXAMPLE HERE)

Another area of development that we can make easier is the act of finding issues with our code and fixing them. Automated testing allows us to identify issues with our code during development without having to manually test our applications (hence the term, automated testing). There are four main testing "levels":

1. Unit Testing - testing small units of code for expected values, etc.
2. Integration Test - testing interaction between different parts or units that unit testing would not be able to cover.
3. System Testing - testing the system as a whole to check if all of the components are working together correctly with a simulation of the production environment.
4. Acceptance Testing - testing the entire system's compliance with requirements and determines if the system is production ready (typically through test scenarios or end-to-end testing).

Let's first start with unit tests. Now, our notes application does not consist of any functions that are complex enough to break into separate parts to test their outcomes, so let's make a new file that we can apply unit tests to.

**Backend/utils/test_example.js**

```js
const palindrome = string => {
  return string
    .split('')
    .reverse()
    .join('')
}

const average = array => {
  const reducer = (sum, addition) => {
    return sum + addition
  }

  return array.reduce(reducer, 0) / array.length
}

module.exports = {
  palindrome,
  average,
}
```

You might not recognize the `reduce()` array method used above, with this, let's talk about and review some concepts in functional programming (with JavaScript).

## What is functional programming?

Functional programming typically allows for the easier reasoning of logic and higher reuse of code.

### Higher-Order Functions

In JavaScript, all functions are 'values' to which you can create an anonymous function and store it in a variable like so:

```js
var double = (x) => {
  return x * 2
}

var test = double

test(2) // returns 4
```

Here, you can see how we manipulate this function as if it were a value, because it is in fact a value just like strings, numbers, etc. This also allows us to **pass functions into other functions**.

Functions that take functions as parameters are typically referred to as *higher-order functions*. This allows for **composition** to where we can make a bunch of functions that achieve small tasks and *compose* them into bigger functions that achieve a bigger task.

We've dealt with higher order functions in the past, `filter()`, `map()`, etc. 

For example, let's just say that we wanted to filter an array by a property's value, in this case, we'll be looking to filter for only people whose favorite color is red.

Attempting that filter with a regular for-loop would look something like this:

```js
var filtered = []
for (var i = 0; i < people.length; i++) {
  if (people[i].favoriteColor === 'red') {
    filtered.push(people[i])
  }
}
```

Now, attempting to do the above in a functional way using a higher-order function would be written as follows:

```js
var filtered = people.filter(person => person.favoriteColor === 'red')
```

> Recall that the `filter()` higher-order function appends the current iterable object if the function (passed in as a property) returns true. It will pass in every single object through the function provided as a property.

These functions allow us to write much cleaner code than we would usually be able to with using a regular for-loop. It is also much easier to follow what's going on with regard to logic.

Taking advantage of composition (although its benefits are not too apparent in this example as these functions are quite simple), we can separate out the callback and compose everything together later in the code as follows:

```js
var isFavoriteColorRed = person => person.favoriteColor === 'red'
var filtered = people.filter(isFavoriteColorRed)
```

Aside from being easier to read, this also allows for the easier reuse of logic and functionality. For example, let's say that we wanted to now filter **out** people with the favorite color red (instead of only including them). We could pass in the `isFavoriteColorRed` function to the `reject()` higher-order function to achieve that as follows

```js
var filteredOut = people.reject(isFavoriteColorRed)
```

Using the traditional for-loop approach would require rewriting multiple lines of code and would result in logic that is harder to follow. Looking at the `isFavoriteColorRed` function, you'll also notice that we never write `return` but rather take advantage of implicit returns to which if our logic for our function fits in one line, we can implicitly return the evaluation of the expression to the right of our 'arrow'.

### Reduce

Now, let's talk about a higher-order function that you have not been previously introduced to (except for the code block above where we make use of this function). `reduce()` is used for any array transformation that you cannot express with `map()`, `filter()`, etc. This is the function to fall back on if you can't find a prebuilt array transformation.

For example, let's just say we wanted to transform an array of number values (in an array called `numbers`) into one figure that represented the sum of the array. Let's look into doing that with a traditional for-loop first:

```js
var total = 0

for (var i = 0; i < numbers.length; i++) {
  total += numbers[i]
}
```

Using `reduce()`, we can convert the above functionality into:

```js
var total = numbers.reduce((sum, number) => sum + number, 0)
```

Just like `map()` and `filter()`, `reduce()` still takes in a callback function, but it also takes in an object to be used as a 'starting point'. Basically, what's happening here is that our first parameter of our callback function, `sum`, is being initialized with the value provided as the second parameter of the `reduce()` method. Then, for each iteration upon `numbers` we are passing the current object pertaining to our iteration into the second parameter of our callback function, to which this number is added to `sum`. This continues throughout the array, to which `sum` is added to every time. Every time an iteration is completed, the new `sum` value will be passed to the callback function for the next iteration with the next object in the array passed as the second parameter.

This initially can be hard to understand coming from more traditional approaches to programming, but it is important to attempt to understand these concepts as there are some clear advantages to functional programming in web development.

With this, we can now circle back to our test application to which we make use of the `reduce()` higher-order function.

#### Testing, continued

Looking at our previous test code, you'll now be able to understand how our `average` function works with the `reduce()` higher-order function.

```js
//...

const average = array => {
  const reducer = (sum, addition) => {
    return sum + addition
  }

  return array.reduce(reducer, 0) / array.length
}

//...
```

Our `average()` function simply takes in an array and applies the array reduce method upon it with our `reducer` function and `0` passed into it as our initialization value. Within our reducer method, we have two parameters, `sum`, which is initialized with the value of `0` and `addition` which will be the current object of our iteration. Over every iteration, we take the `sum` value that was passed to the function and add the value of `addition` to it. We then return this new value and pass it in to be used for the next iteration. After we have iterated through our entire array, we get the first parameter, `sum`, returned to us. This is the sum of all of the values in our array to which we divide by the amount of items in the array to get the average of the array.

Now, with regards to testing this program, there are a variety of test runners that we can make use of. One popular choice is `jest`, a tool developed by Facebook. Naturally, coming from the developers of React, it works well with testing React frontends. It also serves us quite well with testing backends, making it a good choice for testing.

We can install `jest` to our project as a development dependency (since we do not need to test our code in production) by executing the following command:

```bash
npm install --save-dev jest
```

Now that we have `jest` installed, we can make an npm script so that we won't have to specify the path to `jest` within **./node_modules** as it is not installed globally. Additionally, this may make dealing with automated workflows easier in the future.

**/Backend/package.json**

```js
//...

"scripts": {
    "start": "node index.js",
    "build:ui": "rm -rf build && cd ../Frontend && npm run build --prod && cp -r build ../Backend/",
    "deploy": "git push heroku master",
    "deploy:full": "npm run build:ui && git add . && git commit -m UIBuild && npm run deploy",
    "logs:prod": "heroku logs --tail",
    "test": "jest --verbose"
  },
    
//...
```

We also need to specify that our code will actually be executing in Node rather than the browser by adding the following to the end of **package.json**:

**/Backend/package.json**

```js
//...

"jest": {
    "testEnvironment": "node"
  }
}
```

With all of this, we can now get started with writing tests for our applications.

First, let's create a new directory to organize and hold all of our tests called **tests** in the root of our backend directory. We can then create a test file to test our above `palindrome` function that makes use of various string methods. Jest, by default, expects that the names of test files contain '.test' somewhere in its name. For organization and ease of management, we'll be ending our test files with '.test.js'.

**/Backend/tests/palindrome.test.js**

```js
const palindrome = require('../utils/test_example').palindrome // Grabs the `palindrome` function from our 'for_testing' module

test('Palindrome of z', () => {
    const result = palindrome('z')

    expect(result).toBe('z')
})

test('Palindrome of test', () => {
    const result = palindrome('test')

    expect(result).toBe('tset')
})

test('Palindrome of palindrome', () => {
    const result = palindrome('palindrome')

    expect(result).toBe('emordnilap')
})
```

Upon viewing this code in the editor, we're faced with the following:

![image-20191126131154686](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191126131154686.png)

It seems that ESLint is complaining about `test` and `expect` not being defined previously (es-noundef). We can fix this by adding the following property to our `module.exports` object in **.eslintrc.js**

**/Backend/.eslintrc.js**

```js
module.exports = {
  'env': {
    'commonjs': true,
    'es6': true,
    'node': true,
    'jest': true
  },
```

Here, we added the `jest` property with a value of true to the `env` property's object. The errors should now disappear.

Let's take a look at our test file here. We define tests by calling the `test()` function. The `test()` function takes in a test description string as its first parameter. As its second parameter, it takes in a function that actually performs the test. For example, the function provided as the second parameter for our first test is

```js
() => {
    const result = palindrome('z')

    expect(result).toBe('z')
}
```

This function performs the function that we want to test, `palindrome`, upon a predetermined value. We then store this result so that we can compare and verify the result with an expected value. We do this by calling the `expect()` function which takes in the result of our test. We then can compare it by using **matcher functions** (made available to us via methods upon the `expect()` function). In this case, since we're just comparing two strings together, we can just use the `toBe()` matcher function to which we pass in our expected value.

With this, we can execute all our tests and see if our application behaves as expected.

![image-20191126134650097](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191126134650097.png)

From this, we can see that all of our unit tests have passed.

Now, you might be wondering what Jest gives us were a test to fail. Let's provide a false expectation to see what it gives us by changing the expectation in our first function to 'a':

```js
() => {
    const result = palindrome('z')

    expect(result).toBe('a') // Incorrect
}
```

Running our tests now yields the following output

![image-20191126141051765](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191126141051765.png)

Here, you can see that Jest provides us with rich error messages relating to the test that failed, the received value, the expectation involved, and more. These error messages are particularly useful when dealing with errors that may not be easy to identity. Let's look into an example of this in our tests relating to the `average` function.

Let's create a test file **average.test.js** and add the following to the file

**/Backend/tests/average.test.js**

```js
const average = require('../utils/test_example').average

describe('Average', () => {
  test('of one value is the same value', () => {
    expect(average([1])).toBe(1)
  })

  test('of many', () => {
    expect(average([1, 2, 3, 4, 5, 6])).toBe(3.5)
  })

  test('of an empty array is 0', () => {
    expect(average([])).toBe(0)
  })
})
```

Here, we introduce a new function, `describe`, which wraps tests into collections to which we can give the group a name

```js
describe('Name', () => {
  
  // Test 1
  // Test 2
 	// Test ...
  
})
```

You'll also notice the descriptions of our unit tests now start with 'of' as Jest in its output will prepend the name of the collection before the test name. This just makes everything much more readable.

Let's try running these tests. The output that Jest gives us uncovers one error

![image-20191126152516561](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191126152516561.png)

Here, Jest's detailed output comes in handy as we can see that our function, in this case, is returning NaN rather than the expected value of 0 for when averaging an empty array. This is because our array is being divided by the length of our array (0), and, in JavaScript, dividing an empty array by 0 yields NaN (through our reducer, it actually ends up dividing an empty object by 0 which still yields NaN).

We can fix this by updating our `average` function as follows

```js
const average = array => {
  const reducer = (sum, addition) => {
    return sum + addition
  }

  return array.length === 0 ? 0 : array.reduce(reducer, 0) / array.length
}
```

which is updated to return 0 if `array.length === 0`.

We now pass all of our unit tests.

![image-20191126153312664](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191126153312664.png)

## Testing our Backend

Now, let's move on to testing our backend. Unit tests don't really make sense for our backend just yet as we do not utilize any complex logic that we could break into unit tests. So, we can move on to testing our application through actually testing its API which also interacts with our database. This is the next 'level' of testing, integration testing, that we discussed earlier.

#### Environment Modes

Node observes the value of an environment variable `NODE_ENV` for certain behaviors. For example, when `NODE_ENV` is set to 'production', every single development dependency (`devDependencies` in **package.json**) will be ignored with running `npm install`.

We can change the scripts in **package.json** so that we pass in the right value for `NODE_ENV` depending on the context that it is started from.

**/Backend/package.json**

```json
//...

	"scripts": {
    "start": "NODE_ENV=production node index.js",
    "watch": "NODE_ENV=development nodemon index.js",
    "build:ui": "rm -rf build && cd ../Frontend && npm run build --prod && cp -r build ../Backend/",
    "deploy": "git push heroku master",
    "deploy:full": "npm run build:ui && git add . && git commit -m UIBuild && npm run deploy",
    "logs:prod": "heroku logs --tail",
    "test": "NODE_ENV=test jest --verbose --runInBand"
  },

//...
```

Here, we updated our `start`, `watch`, and `test` scripts to reflect the environment mode that we're in. Additionally, we added the `runInBand` flag to our `test` script since we don't want our tests running in parallel (the reason for this will be explained later). 

One issue that we have is that this method of setting environment variables does not work on Windows.  Most Windows command prompts won't accept environment variables being set like `NODE_ENV=production` as they require them to be set using `set` or `$env:node_env = 'production'`. We can fix this by installing `cross-env` which allows for the cross-platform setting of environment variables. 

```bash
npm install cross-env --save-dev
```

With this, we can just use the library in our scripts to allow our environment variables to work cross-platform.

**/Backend/package.json**

```json
//...

	"scripts": {
    "start": "cross-env NODE_ENV=production node index.js",
    "watch": "cross-env NODE_ENV=development nodemon index.js",
    "build:ui": "rm -rf build && cd ../Frontend && npm run build --prod && cp -r build ../Backend/",
    "deploy": "git push heroku master",
    "deploy:full": "npm run build:ui && git add . && git commit -m UIBuild && npm run deploy",
    "logs:prod": "heroku logs --tail",
    "test": "cross-env NODE_ENV=test jest --verbose --runInBand"
  },

//...
```

Now, in addition to the changes that Node brings with the differing environment modes, we can provide our own differing behaviors dependent on the mode environment variable.

One example of this would be the usage of a test database for running our tests.

One implementation of the above example would involve the creation a local database (we don't want to just create another cloud database as this would not work if we ever got to the point where multiple people were working on the same project, but on different things). Each developer would then have a database created locally with it being used for testing. In order to achieve this, we would use a Docker container or by running Mongo in-memory (we'll talk about this later).

For demonstrative purposes, we'll just continue to use our MongoDB Atlas database for our tests since we just want to understand how to go about integration testing our backend for now.

With this, we can add some logic to our application pertaining to using a different MongoDB URI if we're in 'test mode'.

**/Backend/utils/config.js**

```js
require('dotenv').config()

let PORT = process.env.PORT
let MONGODB_URI = process.env.MONGODB_URI

if (process.env.NODE_ENV === 'test') {
    MONGODB_URI = process.env.TEST_MONGODB_URI
}

module.exports = {
    MONGODB_URI,
    PORT
}
```

Here, we simply assign our `MONGODB_URI` variable that gets exported out of this module with `TEST_MONGODB_URI` if we are running in 'test mode' (indicated by `NODE_ENV === 'test'`). We can now specify the value of `TEST_MONGODB_URI` in our **.env** file (recall that in production mode, these environment variables will not come from our **.env** file but rather the environment variables defined using Heroku). 

**/Backend/.env**

```text
MONGODB_URI=mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority
PORT=3001
TEST_MONGODB_URI=mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/test-note-app?retryWrites=true&w=majority
```

Our `TEST_MONGODB_URI` value comes from our original URI, just with a changed database name (found after the last forward slash) where we changed `note-app` to `test-note-app`.

### Using Supertest

Prior to working in this section, let's delete all of our test test files (not a typo, we're referring to the test files for testing that we worked with earlier) including: **average.test.js**, **palindrome.test.js**, and **text_example.js**.

We can now get into using the `supertest` library which provides a layer of abstraction for testing HTTP (making it easier), while also allowing us to drop down to the lower-level APIs were there a need to (giving us the same amount of flexibility).

Let's install the npm module by running the following command:

```bash
npm install supertest --save-dev
```

Now, we can write our first backend integration test by writing the following to **notes_api.test.js**.

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')

const api = supertest(app)

test('notes returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

afterAll(() => {
  mongoose.connection.close()
})
```

Here, you'll notice quite a few new concepts. We'll go through these new concepts one at a time. First, we import our `app` module (as we need this to 'start' our application so that we can interact with it to perform our integration tests). We pass our `app` module into the `supertest()` function which transforms our `app` module into a **superagent** (basically transforms the module into a lightweight, client-side HTTP server that supports HTTP requests). We then assign this superagent to the `api` variable. Now, our tests can use `api` to make HTTP requests to the backend.

Throughout this code, you'll see various references to the `async`/`await` keywords. This refers to another way to write asynchronous code (essentially syntactic sugar around promises). We will discuss this syntax later on.

You can see how we make these requests and compare the results in the following block of code:

```js
test('notes returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})
```

Here, wrapped in a test function, we have a function that performs a HTTP GET request to our backend (/api/notes) through calling the `get()` method upon our `api` variable with the parameter being the endpoint that we want to make our request to.

Then, to 'test' our response, we make the expectation that the response code should be `200` through the use of the `expect()` method. We also do the same to make sure that our `Content-Type` header is set to `application/json` (notice how we have to escape the second forward slash as it must be included).

Finally, after all of our tests have run, we want to close our database connection so that our application can terminate and so that we don't leave any unneeded connections open to our database.

We do this by utilizing the `afterAll()` method to which a callback is provided as a parameter to which it is executed after all of the tests have ran (hence the name).

```js
afterAll(() => {
  mongoose.connection.close()
})
```

Here, were we not to initialize the `testEnvironment` property with the value of `node` in the `jest` object of **package.json** as we did earlier, we would have encountered the following error:

![image-20191127012415162](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191127012415162.png)

This line essentially just tells Jest to use a node-like environment for testing rather than the default `jsdom` environment that simulates a browser-like environment. This could lead to some unexpected behaviors with the execution of our code were it left to its defaults.

Also, take note with how we passed in our `app` module rather than make use of **index.js** somehow. This is because we don't need to manually start our Express application or define ports as `supertest` will bind our server to an ephemeral port (ports allocated automatically by software).

Let's increase the complexity of our tests by adding two more tests to **notes_api.test.js**:

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')

const api = supertest(app)

test('notes returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

test('there are three notes', async () => {
    const response = await api.get('/api/notes')
  
    expect(response.body.length).toBe(3)
})

test('the first note is about it being the first note', async () => {
    const response = await api.get('/api/notes')

    expect(response.body[0].content).toBe('This is the first note!')
})

afterAll(() => {
  mongoose.connection.close()
})
```

You'll notice that our tests not depend on the state of our database. This is not good, because the 'state' of our database persists across separate test runs. This means that the results of one test run could differ from the other without changing any of the tests.

#### Initializing Test Databases

In order for our tests to be consistent and not dependent on the state of our database, let's reset and initialize our database prior to each test run.

First, let's initialize our database before the tests run by making use of the `beforeEach()` function. Similarly to how the `afterAll()` allowed us to execute a callback function after all of the tests had ran, the `beforeEach()` function allows us to execute a function prior to running any test.

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)
const Note = require('../models/note')

const initializationNotes = [
  {
    content: 'This is the first note!',
    date: new Date(),
    flagged: true,
  },
  {
    content: 'This is a note that happens to be the second note.',
    date: new Date(),
    flagged: false,
  },
  {
    content: 'This is the third note.',
    date: new Date(),
    flagged: true,
  }
]

beforeEach(async () => {
  await Note.deleteMany({})

  let noteObject = new Note(initializationNotes[0])
  await noteObject.save()

  noteObject = new Note(initializationNotes[1])
  await noteObject.save()
  
  noteObject = new Note(initializationNotes[2])
  await noteObject.save()
})

// ...
```

Here, we created an `initialNotes` array which contains the three notes. Then, in the `beforeEach()` callback function, we wipe the database using the `deleteMany()` method of the `Note` model. The `deleteMany()` method deletes all of the documents that match the conditions passed into it, to which we had passed an empty object, deleting all of the documents in the collection.

We then create two new note objects by using the `Note` model constructor function and passing in the respective note to that function. We then preform `save()` upon the `noteObject` variable which holds the respective note to be saved that was just created by our `Note` model. This note is then saved by calling hte `save()` method upon the document.

After this, our database should be set up to perform our tests. With this, let's make a few updates to our last two tests so that they make use of the information now granted to us about our database due to its state being set up predictably.

**/Backend/tests/notes_api.test.js**

```js
//...

test('there are three notes', async () => {
    const response = await api.get('/api/notes')
  
    expect(response.body.length).toBe(initializationNotes.length)
})

test('the first note is about it being the first note', async () => {
    const response = await api.get('/api/notes')
    
    const allContents = response.body.map(n => n.content)

    expect(allContents).toContain('This is the first note!')
})

//...
```

In the first test, we simply just make use of the fact that our database is now reset and initialized to the `intitializationNotes` array so that we can just compare the length of our response (which should contain the entire resource collection of the notes) to the size of that array.

For the second test, we first take our response (containing all of the notes) and map over it to get an array containing only the contents of all of the notes. Then, we make use of another matcher function, `toContain()` that allows us to check if a specific value is contained in an array. Thus, if any of these notes has `'This is the first note!'` as its `content` value, then the test will pass.

### Logger Module

As our project scales, it's helpful to create a module that is purely responsible for console logging so that we can do thing like prevent console logging in 'test mode'.

We do not want console logging in test mode as it can produce undesirable output such as the following:

![image-20191127022421594](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191127022421594.png)

Here, you can see the 'Connecting to: ...' and 'Connected to MongoDB' console logs polluting the display of our results.

With this, let's create the **logger.js** utility module.

**/Backend/utils/logger.js**

```js
const info = (...params) => {
  if (process.env.NODE_ENV !== 'test') {
    console.log(...params)
  }
}

const error = (...params) => {
  console.error(...params)
}

module.exports = {
  info, error
}
```

Here, in our `logger` module, we export two functions, the `info` function which contains a check to prevent these log statements from printing if our application is in test mode, and the `error` function which will `console.error` whatever parameter(s) is passed to it and will execute regardless of the environment mode.

Let's also build a `logRequest` middleware that takes advantage of these new functions.

```js
const logger = require('./logger')

const logRequest = (req, res, next) => {
    logger.info('Method:  ', req.method)
    logger.info('Path:    ', req.path)
    logger.info('Body:    ', req.body)
    logger.info('--------')
    next()
}

const unknownEndpoint = (req, res) => {
    res.status(404).send({ error: 'Endpoint does not exist (Unknown Endpoint)' })
}

const errorHandler = (err, req, res, next) => {
    logger.error(err.message)

    if (err.name === 'CastError' && err.kind === 'ObjectId') {
        return res.status(400).send({ error: 'Incorrect format for id parameter' })
    } else if (err.name === 'ValidationError') {
        return res.status(400).json({ error: err.message })
    }

    next(err)
}

module.exports = {
    logRequest,
    unknownEndpoint,
    errorHandler
}
```

Here, you'll notice how we build the `logRequest` function that logs various bits of information about the request using `logger.info()`. We also updated our `errorHandler` middleware to use `logger.error()` function.

We also need to make the appropriate changes to **app.js** involving our new **logger.js** module, as well as adding `logRequest` among our middleware.

**/Backend/app.js**

```js
//...

const logger = require('./utils/logger')

logger.info('Connecting to:', config.MONGODB_URI) // Now uses logger

mongoose.connect(config.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false
})
    .then(() => {
        logger.info('Connected to MongoDB') // Now uses logger
    })
    .catch((error) => {
        logger.error('Error connecting to MongoDB:', error.message) // Now uses logger
    })

// Middleware
app.use(cors())
app.use(express.static('build'))
app.use(bodyParser.json())
app.use(middleware.logRequest) // The addition of our `logRequest` middleware

//...
```

### Running Individual Tests

We can run tests individually by calling Jest directly without using our `npm test` script. We can do this by installing `jest` globally and calling tests by file name or name.

First, let's install `jest` globally by using the **-g** flag.

```bash
npm install -g jest
```

With this, we can now call Jest from the command line with the `jest` command.

We can run notes by file name as follows:

```bash
jest tests/notes_api.test.js
```

We can run notes by name by using the **-t** argument as follows. This matches tests based on the name of the test or the first parameter of the `describe` block.

```bash
jest -t 'notes returned as json'
```

This argument can also be used to run multiple tests that contain the given parameter as follows:

```bash
jest -t 'notes' --runInBand
```

This will execute all tests that contain 'notes' in their name. Also, recall that we must provide the **--runInBand** flag if we want the tests to run serially (not all at the same time).

### Async/Await

Circling back to when we first introduced `supertest`, we touched on the introduction of two new keywords, `async` and `await`. This syntax was first introduced in ES7 and allows us to write asynchronous code in a synchronous manner.

Let's first look at how we would achieve the task of removing the first note (in a verbose way) using promises.

```js
Note.find({}).then(notes => {
  return notes[0].remove()
})
.then(res => {
  console.log('successfully removed the first note')
})
```

Here, we use the `find()` method upon the `Note` model in order to get all of the notes, then we access the result of this promise through registering a callback to the `then()` method which we then remove the first note in. This is an example of a promise chain (which we have discussed in-depth earlier). Another approach to making multiple asynchronous function calls in sequence would be to register callbacks within callbacks, but this is undesirable as it quickly becomes hard to read and leads to what is called *callback hell*.

As syntactic sugar around these promises, **generator functions** were introduced in ES6 as well that provided a way, similar to promises, to write asynchronous code that resembled synchronous code. However, generator functions were never widely used with its awkward syntax. Although, it is interesting to know that generators and promises formed the foundation for the async/await expressions. 

Now, let's dive in to doing the above using the asynx/await keywords. We can use the `await` operator to *wait* for a promise (like the one `Note.find({})` returns).

```js
const notes = await Note.find({})
```

Now, the execution of our code pauses at this line until the promise (the expression following the `await` keyword) is fulfilled. Once this promise is fulfilled, the execution of our code will continue. Here, you can see how our code looks like synchronous code.

Now, let's add the removal of the first note.

```js
const notes = await Note.find({})
const res = await notes[0].remove()
```

Here, we first await for the promise returned by `Note.find({})` to be fulfilled. Then, we assign the value returned by that promise to `notes`. Then, we access the first element of the array and call `remove()` upon it, to which we wait for the promise returned by that call to be fulfilled and then store the returned result to `res`.

There are two main requirements to be aware of when working with `async`/`await`.

1. You can only use the `await` operator with asynchronous operations that return a promise. This is not too much of a constraint, however, as most asynchronous functions using callbacks can be wrapped in promises quite easily.
2. You can only use the `await` operator inside an **async function** denoted by the `async` keyword before the function definition as follows:

```js
const exampleAsyncFunction = async () {
  // I can now use `await` here!
}

exampleAsyncFunction()
```

### Using Async/Await in the Backend

We can rewrite our controllers to use `async`/`await` without changing all too much.

Here is what our controller originally looked like:

```js
notesRouter.get('/', (req, res) => {
    Note.find({}).then(notes => {
        res.json(notes.map(note => note.toJSON()))
    })
})
```

This is what the controller looks like using `async`/`await`:

**/Backend/controllers/notes.js**

```js
notesRouter.get('/', (req, res) => {
    const notes = await Note.find({})
    res.json(notes.map(note => note.toJSON()))
})
```

Now, with refactoring our code, we must test to see if everything works as encountering **regression** (the breaking of existing functionality) is always a risk when refactoring.

In this case, we can test our route just by visiting the respective path or by using testing tools, but this can become tedious when dealing with many controllers and routes as we're about to. As an alternative, we can use our testing suite, Jest, in order to automate this process for us.

Let's write a test for each route of our API.

First, let's write a test that tests our HTTP POST controller that creates a new note.

**/Backend/tests/notes_api.test.js**

```js
//...

test('a valid note can be added ', async () => {
  const newNote = {
    content: 'This is a new note!',
    flagged: true,
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(201)
    .expect('Content-Type', /application\/json/)

  const response = await api.get('/api/notes')

  const contents = response.body.map(r => r.content)

  expect(response.body.length).toBe(initializationNotes.length + 1)
  expect(contents).toContain(
    'This is a new note!'
  )
})

//...
```

Here, you'll notice how we send the note and then await for the response. Upon receiving the response, we check to see if the length of the notes array is one higher than the initial array length that we set our database state to earlier. We also check if the collection of notes contains the text of our note that we had added. Additionally, we just check if the status code of the response and content-type header are correct.

Upon running this test, we encounter a few issues. Our test does not pass, so we'll have to make some changes to our controller as we had not set the correct status all along. We never set our status code before sending our response using the `json()` method, so we'll just call the `status()` method to do so beforehand. Our updated controller looks like this:

```js
notesRouter.post('/', (req, res, next) => {
  const body = req.body

  if (!body.content) {
    return res.status(400).json({
      error: 'No content specified in note body'
    })
  }

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  })

  newNote.save().then(savedNote => {
    res.status(201).json(savedNote.toJSON())
  })
    .catch(error => next(error))
})
```

Now, our test passes! Let's also test our edge case by writing a test that checks if a note with no content will not be saved into the database.

**/Backend/tests/notes_api.test.js**

```js
//...

test('note without content is not saved', async () => {
  const newNote = {
    flagged: true
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(400)

  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(initializationNotes.length)
})

//...
```

Here, we just expect that the status code returned will be `400` and that the length of `response` (which contains the results of a HTTP GET request to our notes collection) is the same as the length of the array that we had initialized our database with.

Note, when we run our tests serially, the previous tests do not affect the outcome of subsequent tests as we reset the state of our database before each and every test. Otherwise, the previous test where we added a note to our database would affect our test results here as the length of `response` would be one higher than it is supposed to be.

Now, you'll notice that we're starting to repeat some code between our tests. This is usually a good indicator that there is something better to do with our code. Let's abstract around our code and create a few helper functions in their own module that we can use in our tests.

**/Backend/tests/test_helper.js**

```js
const Note = require('../models/note')

const initializationNotes = [
  {
    content: 'This is the first note!',
    date: new Date(),
    flagged: true,
  },
  {
    content: 'This is a note that happens to be the second note.',
    date: new Date(),
    flagged: false,
  },
  {
    content: 'This is the third note.',
    date: new Date(),
    flagged: true,
  }
]

const deletedNoteId = async () => {
  const note = new Note({ 
    content: 'This note was created as part of a test and will be deleted', 
    date: new Date() 
  })
  
  await note.save()
  await note.remove()

  return note._id.toString()
}

const dbNotes = async () => {
  const notes = await Note.find({})
  return notes.map(note => note.toJSON())
}

module.exports = {
  initializationNotes, deletedNoteId, dbNotes
}
```

Here, we simply created two helper functions and moved the definition of our `initializationNotes` to this file. In `deletedNoteId`, we create a new note, save the note to our database, and then delete the not. After this, we fetch the `_id` property of the note object that we created using the `Note` model constructor function and return that, representing the ID of a database object that has since been deleted. In `dbNotes` we simply fetch all of the notes and create an array of notes mapped through `toJSON()`.

Now, we can use these helper functions in our tests as follows:

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)

const Note = require('../models/note')

const helper = require('./test_helper')

beforeEach(async () => {
  await Note.deleteMany({})

  let noteObject = new Note(helper.initializationNotes[0])
  await noteObject.save()

  noteObject = new Note(helper.initializationNotes[1])
  await noteObject.save()

  noteObject = new Note(helper.initializationNotes[2])
  await noteObject.save()
})

test('notes returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

test('there are three notes', async () => {
  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(helper.initializationNotes.length)
})

test('the first note is about it being the first note', async () => {
  const response = await api.get('/api/notes')

  const allContents = response.body.map(n => n.content)

  expect(allContents).toContain('This is the first note!')
})

test('a valid note can be added ', async () => {
  const newNote = {
    content: 'This is a new note!',
    flagged: true,
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(201)
    .expect('Content-Type', /application\/json/)

  const allNotes = await helper.dbNotes()

  const contents = allNotes.map(r => r.content)

  expect(allNotes.length).toBe(helper.initializationNotes.length + 1)
  expect(contents).toContain(
    'This is a new note!'
  )
})

test('note without content is not saved', async () => {
  const newNote = {
    flagged: true
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(400)

  const allNotes = await helper.dbNotes()

  expect(allNotes.length).toBe(helper.initializationNotes.length)
})

afterAll(() => {
  mongoose.connection.close()
})
```

We can delete our `initializationNotes` constant since we can reference it from the helper file. In the above block, we changed all references to `initializationNotes` to `helper.initializationNotes`. Additionally, in our 'a valid note can be added' test, we use our `dbNotes()` helper function to get all the notes rather than making a HTTP GET request for all the notes and using the response. We also do the same for the ''note without content is not saved" test.

Upon running our test script, we'll find that all of our tests pass. With this, we can now begin to refactor our code to use `async`/`await`.

We can change our HTTP POST handler (that handles the adding of notes) to use `async`/`await` as follows:

**/Backend/controllers/notes.js**

```js
//...

notesRouter.post('/', async (req, res, next) => {
  const body = req.body

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  })

  const savedNote = await newNote.save()
  res.status(201).json(savedNote.toJSON())
})

//...
```

There is one more thing we have to consider. As we are no longer using promises, we no longer make use of the `catch` method upon the promise. So, you might be wondering, how would we handle errors with an unhandled promise rejection? In our current solution, if the promise were to be rejected, the request would never receive a response as we simply do not have any handler for a rejection.

We still receive a warning for any errors in the console, but our error handling middleware does not receive the exception. Let's observe this by triggering a validation error (recall that we specified that the content field is required in our `Note` schema).

![image-20191129130235189](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191129130235189.png)

Here, we encounter the default behavior for unhandled promise rejections which is to log the exception. It is an expectation that every promise will have a promise rejection handler (have a `catch`); thus, in the future, unhandled promise rejections will simply terminate the process. The request is still logged as we had bound the `logRequest` middleware prior to all of our routers in the `App` module.

We can add a promise rejection handler through the use of the `try...catch` statement as follows:

**/Backend/controllers/notes.js**

```js
//...

notesRouter.post('/', async (req, res, next) => {
  const body = req.body

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  })

  try {
    const savedNote = await newNote.save()
    res.status(201).json(savedNote.toJSON())
  } catch(exception) {
    next(exception)
  }
})

//...
```

This is read as 'trying' the code in the first block, and if there is an exception in the first block, the second block will 'catch' the exception. Here, in the catch block, we simply call the `next` function to pass the exception down to our error handling middleware.

Now, moving on to the functionality regarding fetching a specific note, we can write the test as follows:

**/Backend/tests/notes_api.test.js**

```js
//...

test('a specific note can be fetched', async () => {
  const allNotes = await helper.dbNotes()

  const noteToFetch = allNotes[0]

  // Convert date value to string as the response will contain a string
  noteToFetch.date = noteToFetch.date.toISOString()

  const fetchedNote = await api
    .get(`/api/notes/${noteToFetch.id}`)
    .expect(200)
    .expect('Content-Type', /application\/json/)

  expect(fetchedNote.body).toEqual(noteToFetch)
})

//...
```

Additionally, we can write the test surrounding deleting a specific note as follows:

**/Backend/tests/notes_api.test.js**

```js
//...

test('a specific note can be deleted', async () => {
  const unalteredNotes = await helper.dbNotes()
  const noteToDelete = unalteredNotes[0]

  await api
    .delete(`/api/notes/${noteToDelete.id}`)
    .expect(204)

  const alteredNotes = await helper.dbNotes()

  expect(alteredNotes.length).toBe(
    helper.initializationNotes.length - 1
  )

  const contents = alteredNotes.map(r => r.content)

  expect(contents).not.toContain(noteToDelete.content)
})

//...
```

Here, these tests simply perform the operation and a comparison to the original state of the database (whether that be the database state being stored in a variable via a call to `dbNotes()` or via a direct comparison to the `initializationNotes` array). 

Now, we can go ahead and refactor the routes involving deleting and fetching a specific note as follows:

```js
//...

notesRouter.delete('/:id', async (req, res, next) => {
  try {
    await Note.findByIdAndDelete(req.params.id)
    res.status(204).end()
  } catch (exception) {
    next(exception)
  }
})

notesRouter.get('/:id', async (req, res, next) => {
  try {
    const note = await Note.findById(req.params.id)

    if (note) {
      res.json(note.toJSON())
    } else {
      res.status(404).end()
    }
  } catch (exception) {
    next(exception)
  }
})

//...

```

Here, the benefits of `async`/`await` is not fully realized. When dealing with multiple asynchronous function calls, these operators come in very handy. However, in situations like this, `async`/`await` does not offer a clear benefit.

### Optimizations

As our project gets more complex and we begin to take advantage of our linter more often, it may be a good idea to remove the build folder automatically when we run our `lint:fix` script.

We can update our script as follows, adding `cross-env rm -rf build &&` to the beginning:

**/Backend/package.json**

```json
//...  
"scripts": {
    "start": "cross-env NODE_ENV=production node index.js",
    "watch": "cross-env NODE_ENV=development nodemon index.js",
    "build:ui": "rm -rf build && cd ../Frontend && npm run build --prod && cp -r build ../Backend/",
    "deploy": "git push heroku master",
    "deploy:full": "npm run build:ui && git add . && git commit -m UIBuild && npm run deploy",
    "logs:prod": "heroku logs --tail",
    "test": "cross-env NODE_ENV=test jest --verbose --runInBand",
    "lint:fix": "cross-env rm -rf build && eslint . --fix"
  },

//...
```

With this, we can run and fix our programming style much faster as ESLint will no longer look through the massive **build** directory for errors. Additionally, the stylistic deviations in the built files will no longer pollute our linter's output.

This example illustrates the difference in amount of errors as a result of this change. Before making the change, we received the following output:

![image-20191129234113979](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191129234113979.png)

After the changes, we receive the following output:

![image-20191129234148865](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191129234148865.png)

Now, we can easily see all of the problems involved with our project and fix them easily.

#### Refactoring `beforeEach`

Next, let's refactor our `beforeEach` function in our test file as we're repeating code and generally not following good programming practices.

We can take advantage of the `forEach` higher-order function in order to store our notes in our database for each note in our `initializationNotes` as follows:

**/Backend/tests/notes_api.test.js**

```js
//...

beforeEach(async () => {
  await Note.deleteMany({})
  console.log('Deleted all notes!')

  helper.initializationNotes.forEach(async (note) => {
    let newNote = new Note(note)
    await newNote.save()
    console.log('Saved the new note!')
  })
  console.log('Done with all of the operations!')
})

//...
```

Now, this code looks like it'll work just fine, but upon trying to run our tests, we run into the following issue:

![image-20191130011745234](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191130011745234.png)

All of our tests had failed! Our `console.log()` statements above can shed some reason as to why our tests are failing. Using `console.log()` statements are mostly a good idea when dealing with asynchronous code, as they provide a "synchronous" way of looking at things upon execution.

![image-20191130012343743](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191130012343743.png)

Here, we can see that we reach the end of the `beforeEach()` function (signaled by the "Done with all of the operations" log statement) before any of our notes are saved. This is because, inside our `forEach()` loop, for each iteration, we create a new function that contains the `await` operator, so that the execution of this new function will wait for this statement marked by the `await` operator (saving the note) to complete. Thus, for each iteration, we get entirely new functions inside our `beforeEach()` function that wait for the note to be saved, but since the `await` operators were not defined in the `beforeEach()` function, the `beforeEach()` function does not wait for these commands to finish. With this, the database state is not set properly before each test is executed, leading to the failing of all of our tests.

#### Promise.all()

We can fix this by using `Promise.all()` method. This method returns a promise that fulfills when **all** of the promises passed to it have been fulfilled or when it contains 0 promises. With regard to rejection behavior, the promise will reject as soon as it encounters the first rejection among the promises. The solution would be as follows:

**/Backend/tests/notes_api.test.js**

```js
//...

beforeEach(async () => {
  await Note.deleteMany({})

  const allNotes = helper.initializationNotes
    .map(note => new Note(note))
  const promiseArray = allNotes.map(note => note.save())
  await Promise.all(promiseArray)
})

//...
```

Here, we map over our `initializationNotes` to create an array of Mongoose objects (created using the `Note` constructor function). Then, we take this array of Mongoose objects and map over it, within the map function, we call `save()` upon the Mongoose object which returns a promise to be fulfilled. This gives us an array of promises that are to be fulfilled. Note that these promises are in the process of being fulfilled/rejected as we have already executed the `save()` operation, but we can ensure the fulfillment/rejection of all of these promises before ending the function by passing this array to `Promise.all()` with the `await` operator in front of it. The `Promise.all()` method essentially creates a "master" promise that we can `await` since it will only fulfill when all of the promises have fulfilled and is located in `beforeEach()` so the function will actually wait for its execution to be completed.

> To clarify, promises cannot be "executed", promises themselves are wrappers around a created task and represent only the results of that task. Thus, when we call `note.save()` above, the task of saving the note has started and we are returned with a promise that represents the result of that task which can be passed in to `Promise.all()`. Execution simply refers to the underlying task being performed, which must already have been performed for the promise to be created. In short, `Promise.all()` is not executing anything, but rather just ensuring that all of these promises are fulfilled (or one is rejected) before continuing.

Now, as these promises are executed within the function provided to `allNotes.map()`, all of these promises are executed in parallel being that we can't execute them sequentially with no guarantees as to the order of the promises being finished. As we can iterate through our note objects and create our promises on-demand, we can ensure a specific execution order by creating a promise, waiting until its fulfillment and then proceeding as follows:

```js
beforeEach(async () => {
  await Note.deleteMany({})

  for (let note of helper.initializationNotes) {
    let newNote = new Note(note)
    await newNote.save()
  }
})
```

Here, for each note in our `initializationNotes` we just create a new note, save it, and await that promise before continuing to the next iteration.

Finally, with this handled, we can go over tests for invalid IDs and non-existent as follows:

```js
test('fails with status code 404 if ID does not exist', async () => {
  const validNonexistentId = await helper.deletedNoteId()

  await api
    .get(`/api/notes/${validNonexistentId}`)
    .expect(404)
})

test('fails with statuscode 400 if id is invalid', async () => {
  const invalidId = 'thisisaninvalidid'

  await api
    .get(`/api/notes/${invalidId}`)
    .expect(400)
})
```

With this, our test coverage covers most major functionality of our application and some edge cases. The following is the entire test file with the culmination of all of our tests that we've written:

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)

const Note = require('../models/note')

const helper = require('./test_helper')

beforeEach(async () => {
  await Note.deleteMany({})

  const allNotes = helper.initializationNotes
    .map(note => new Note(note))
  const promiseArray = allNotes.map(note => note.save())
  await Promise.all(promiseArray)
})

test('notes returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

test('there are three notes', async () => {
  const response = await api.get('/api/notes')

  expect(response.body.length).toBe(helper.initializationNotes.length)
})

test('the first note is about it being the first note', async () => {
  const response = await api.get('/api/notes')

  const allContents = response.body.map(n => n.content)

  expect(allContents).toContain('This is the first note!')
})

test('a valid note can be added ', async () => {
  const newNote = {
    content: 'This is a new note!',
    flagged: true,
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(201)
    .expect('Content-Type', /application\/json/)

  const allNotes = await helper.dbNotes()

  const contents = allNotes.map(r => r.content)

  expect(allNotes.length).toBe(helper.initializationNotes.length + 1)
  expect(contents).toContain(
    'This is a new note!'
  )
})

test('note without content is not saved', async () => {
  const newNote = {
    flagged: true
  }

  await api
    .post('/api/notes')
    .send(newNote)
    .expect(400)

  const allNotes = await helper.dbNotes()

  expect(allNotes.length).toBe(helper.initializationNotes.length)
})

test('a specific note can be fetched', async () => {
  const allNotes = await helper.dbNotes()

  const noteToFetch = allNotes[0]

  // Convert date value to string as the response will contain a string
  noteToFetch.date = noteToFetch.date.toISOString()

  const fetchedNote = await api
    .get(`/api/notes/${noteToFetch.id}`)
    .expect(200)
    .expect('Content-Type', /application\/json/)

  expect(fetchedNote.body).toEqual(noteToFetch)
})

test('a specific note can be deleted', async () => {
  const unalteredNotes = await helper.dbNotes()
  const noteToDelete = unalteredNotes[0]

  await api
    .delete(`/api/notes/${noteToDelete.id}`)
    .expect(204)

  const alteredNotes = await helper.dbNotes()

  expect(alteredNotes.length).toBe(
    helper.initializationNotes.length - 1
  )

  const contents = alteredNotes.map(r => r.content)

  expect(contents).not.toContain(noteToDelete.content)
})

test('fails with status code 404 if ID does not exist', async () => {
  const validNonexistentId = await helper.deletedNoteId()

  await api
    .get(`/api/notes/${validNonexistentId}`)
    .expect(404)
})

test('fails with statuscode 400 if id is invalid', async () => {
  const invalidId = 'thisisaninvalidid'

  await api
    .get(`/api/notes/${invalidId}`)
    .expect(400)
})


afterAll(() => {
  mongoose.connection.close()
})
```

We could improve these tests, provide additional code coverage, and improve the readability of these tests (adding describe blocks as an example), but this is a good starting point and should demonstrate how to write integration/unit tests.



