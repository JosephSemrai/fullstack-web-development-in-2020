Web Development 4



## Authentication

Almost every application on the Internet requires some form of user authentication (whether that be through a Google account, phone number, email, etc.) in order to restrict access to certain resources. With this, let's begin to add a way to authenticate users in our application.

To authenticate users, we first need a way to keep track of them and store user information in a persistent manner. Our database fits these requirements perfectly, so we will store our users in the database.

As an overview of what we'll be doing: our database will hold information pertaining to the user's credentials and the notes that they "own". Users will only be able to modify the notes that they own (similar to how users can only manage their own posts on social media platforms).

Thinking about the relationship between our user database objects and the notes, we would have a one-to-many relationship where one user has many notes. Now, you might have worked with relational databases in the past where implementing the above relationship is quite straightforward. Due to the nature of NoSQL databases (recall that the NO stands for "not only"), we are not restricted to a subset of implementations but rather are given many different ways to model this situation. One approach would be to have a collection of *notes* and a collection of *users*. There are quite a few features found in relational databases that document databases (like MongoDB) do not support to which ODMs such as Mongoose try to emulate. This statement is becoming less true over time, however, as MongoDB and other document databases add more features.

### Referencing Objects in Other Collections

In document databases, like we did earlier, we refer to documents via object IDs (similar to referring to a foreign key in a relational database that provides a link between two objects in two different tables).

> Please note that the IDs in the following examples are not representative of anything, but rather just illustrate the relationships between objects.

**Users** collection:

```js
[  
	{
    username: 'josuff',
    _id: 123456,
  },
  {
    username: 'markus',
    _id: 111111,
  },
]
```

**Notes** collection:

```js
[
  {
    content: 'My first note!,
    flagged: false,
    _id: 225533,
    user: 123456,
  },
  {
    content: 'This is an interesting note',
    flagged: true,
    _id: 223266,
    user: 111111,
  },
  {
    content: 'This is a note by 111111!',
    flagged: false,
    _id: 123456,
    user: 111111,
  },
]
```

Here, in a relational database, each note object would contain a **reference key** to the user who "owns" the note. We emulate this in our document database through the use of a `user` property in our note schema that contains the ID of the user that "owns" the note.

We could also go about having the users contain an array of the IDs of all of the notes that they "own" as follows:

```js
[  
	{
    username: 'josuff',
    _id: 123456,
    notes: [225533],
  },
  {
    username: 'markus',
    _id: 111111,
    notes: [223266, 123456],
  },
]
```

Here, we illustrate the one-to-many relationship with that a user can have many notes (with the IDs of these notes being stored in the `notes` property of our user).

We can take either of the above approaches or use both if we truly needed to. The flexibility of document databases also allows us to do things such as nest the actual notes objects within the `notes` property of our user:

```js
[  
	{
    username: 'josuff',
    _id: 123456,
    notes: [
      {
        content: 'My first note!,
        flagged: false,
      },
    ],
  },
  {
    username: 'markus',
    _id: 111111,
    notes: [
      {
        content: 'This is an interesting note',
        flagged: true,
      },
      {
        content: 'This is a note by 111111!',
        flagged: false,
      },
    ]
  },
]
```

Here, our note objects do not have unique IDs given to them by the database as they "belong" to the users in the sense that they are properties of the users rather than independent objects with a relationship to a property of a user. 

As you can see with these various approaches, choosing the structure of a document database is not as clear as it might be with a relational database due to its flexibility. Thus, there may be times where you may have to take some time to explore different implementations without knowing which one may fit your use case the best (as you cannot fully predict the needs and requirements of your project well into the future).

#### User Schema

Let's actually go about defining the schema for our `User` model. For the next step, we'll be making the choice to simply store the references to the note objects that a user owns within the user (reference via `ObjectId`).

**/Backend/models/user.js**

```js
const mongoose = require('mongoose')

const userSchema = new mongoose.Schema({
  username: String,
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
    // For security purposes, the passwordHash of the user should not be sent with the user object (even though it is not the plaintext password, it can still be decrypted)
    delete returnedObject.passwordHash
  }
})

const User = mongoose.model('User', userSchema)

module.exports = User
```

Here, you'll notice that we specify the schema of the user object as having a `username` of type String, a `name` of type String, a `passwordHash` (hashed version of their password, we'll talk about this more later) of type String, and a `notes` array of a complex type. This `notes` array is specified to contain objects of the type `ObjectId` which simply refers to it containing `ObjectId`s of other database objects; additionally, we provide the model that these `ObjectId`s will reference to as the value of the `ref` parameter.

Now, motivated by the desire to have the ability to lookup the user that owns a note from a given note, we'll add a reference to the `User` model within our `Note` model's schema:

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
  user: { // Reference to the user that "owns" the note
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
  }
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

In this approach, the user model can reference the notes that "it" owns, and the notes can reference the user who "owns" them. This is in contrast to conventions of most relational databases, but is possible and acceptable with document databases.

## User Controller

Now, let's actually take advantage of our `User` model by creating a controller that will actually create our users.

Recall the mention of `passwordHash` from earlier and how it refers to a hashed version of a user's password. The value of this property will simply be assigned the output of a *one-way hash function* (essentially means that it will be close to impossible to derive the original text from this hashed string). We store this value as it is not proper with regard to security practices to store plaintext passwords. The topic of encryption and hashing as a subset of cryptography are very interesting topics and deserve a read if you are interested in the how and why with regard to how computers take advantage of cryptography and why it is used.

As with most things in the JavaScript ecosystem, we do not have to write our own functions (especially so in this case as we do not have the acumen nor the resources to maintain all of these functions) to generate our password hashes. We can utilize the `bcrypt` package for doing exactly this:

```bash
npm install bcrypt
```

 As we are *creating* a new resource when we create a new user, we'll be making and handling POST requests when creating these users as per RESTful conventions. Let's create the file **users.js** within our **/controllers** directory.

**/Backend/controllers/users.js**

```js
const bcrypt = require('bcrypt')
const usersRouter = require('express').Router()
const User = require('../models/user')

usersRouter.post('/', async (req, res, next) => {
  try {
    const body = req.body

    const saltRounds = 10
    const passwordHash = await bcrypt.hash(body.password, saltRounds)

    const user = new User({
      username: body.username,
      name: body.name,
      passwordHash,
    })

    const savedUser = await user.save()

    res.json(savedUser)
  } catch (exception) {
    next(exception)
  }
})

module.exports = usersRouter
```

Now, let's just make use of our new controller by binding the exported router to our application through a call to `app.use()`.
**/Backend/app.js**

```js
//...

const usersRouter = require('./controllers/users')

//...

app.use('/api/users', usersRouter)

//...
```

In the above controller, we never store the plaintext password that was sent over the request. Instead, we hash it and store the hash.

You'll notice our call to `bcrypt.hash()` and its parameter `saltRounds`. We specified the value for `saltRounds` earlier in the file as 10. We won't go into too much detail regarding the technical bits of hashing as that would far out-scope the content of this book (while interesting and potentially useful to know, knowledge of the workings of hashing is not required).

Basically, you might have heard of hash functions like MD5, SHA1, SHA2, etc., these are all *general purpose hash functions* that place an emphasis on hashing large amounts of data in a fast manner. This fast manner makes them almost useless for passwords as a modern server (especially one that is GPU accelerated) can calculate the hash extremely fast, making it possible for someone to try every single password combination (given certain constraints). `bcrypt` (based on the Blowfish block cipher cryptomatic algorithm) solves this by introducing a work factor to which we can adjust. `bcrypt` does not simply just hash passwords (as the exact same passwords would have the exact same hash going through the algorithm with the same work factor), but it instead passes in a *salt* (a randomized string) along with the password to hash, so that every single hash is truly unique, giving us an extra layer of security and preventing the use of rainbow tables (described below).

This random salt is controlled by the adjustable amount of salt rounds that we pass into the function and can compensate for the drastic increase of computing power (that makes it easier for computers to calculate hashes) by simply increasing the work factor. This makes it extremely resistant to hacks, especially involving password cracking using something called a rainbow table (databases of pre-calculated hashes for common passwords) as the hash output differs with work factor and between instances due to the randomly generated salts.

In the above example, we simply provide the amount of `saltRounds` as 10 and pass that in to the second parameter of the `hash()` method. The first parameter is simply the password that we wish to hash. It is important to find the right balance between security and usability, however, as increasing the `saltRounds` increases computation time as well as security.

To illustrate, let's write a simple program that will explore the time differences between different costs (`saltRounds`) :

```js
const bcrypt = require("bcrypt");
const plainTextPassword1 = "thisismypassword21173@";

for (let saltRounds = 10; saltRounds < 21; ++saltRounds) {
  console.time(`bcrypt | Salt Rounds: ${saltRounds}, Time:`);
  bcrypt.hashSync(plainTextPassword1, saltRounds);
  console.timeEnd(`bcrypt | Salt Rounds: ${saltRounds}, Time:`);
}
```

Running this on my personal computer (MacBook Pro, 2.3 GHz Intel Core i5 8259U, Intel Iris Plus Graphics 655), we get the following output as to the calculation times.

![image-20191210115205256](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191210115205256.png)

Here, you can see that we go from 71 milliseconds for 10 salt rounds to over 67 *seconds* for 20 salt rounds. This increase in time resembles a exponential relationship which is a good match to counteract the exponential increase of computing power that we experience (for the most part, as the increase in computing power does not rigidly follow Moore's Law as of about a decade).

Finally, it is important to realize that, even with relatively high work values, it is still feasible that a single password could be cracked with enough computing powers. Thus, in the event of a breach where password hashes are released, you should take care to have users reset their passwords (thus producing another hash).

#### Testing the User Controller

Let's write a few initial tests for our new endpoints. We'll add these tests to our existing test file for now, but as you continue to add tests, it may be wise to components out these tests into multiple files:

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)

const Note = require('../models/note')
const User = require('../models/user')

const helper = require('./test_helper')

describe('when there are a few initial notes saved to the db', () => {
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
})

describe('when there is one initial user in the db', () => {
  beforeEach(async () => {
    await User.deleteMany({})
    const user = new User({ username: 'root', password: 'password' })
    await user.save()
  })

  test('creation of a new user succeeds', async () => {
    const initialUsersState = await helper.usersInDb()

    const newUser = {
      username: 'josuff',
      name: 'Joseph Semrai',
      password: 'thisisapassword123',
    }

    await api
      .post('/api/users')
      .send(newUser)
      .expect(200)
      .expect('Content-Type', /application\/json/)

    const finalUsersState = await helper.dbUsers()
    expect(finalUsersState.length).toBe(initialUsersState.length + 1)

    const usernames = finalUsersState.map(u => u.username)
    expect(usernames).toContain(newUser.username)
  })
})

afterAll(() => {
  mongoose.connection.close()
})
```

The above code represents our entire test file. It is quite a bit of code, so let's step through it and provide an overview of what's going on and what has changed.

Firstly, we wrapped our original tests pertaining to our notes routes in a `describe` block so that we could register a different `beforeEach()` function. This allows us to set up our initial database state for our notes test and users test where we are expecting different initial states. Then, we specified our 'creation of a new user succeeds' test by comparing our user collection's state after we have created a new user to the original state when the test started. We do this by checking to see if the collection is one greater in length than the starting state and if the collection contains a username matching the user that we have just created.

You'll also notice that we had called a  `dbUsers()` helper function. We elected to make this helper function as we would likely need to fetch all of the users in multiple cases and tests. The function is as follows:

**/Backend/tests/test_helper.js**

```js
const Note = require('../models/note')
const User = require('../models/user')

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

const dbUsers = async () => {
  const users = await User.find({})
  return users.map(u => u.toJSON())
}

module.exports = {
  initializationNotes, deletedNoteId, dbNotes, dbUsers
}
```

Now, what would happen if we wrote a test testing our ability to create a user with an existing username?

**/Backend/tests/notes_api.test.js**

```js
//...

describe('when there is one initial user in the db', () => {
  
  //...
  
  test('user creation fails with statuscode and message if username taken', async () => {
    const initialUsersState = await helper.dbUsers()

    const newUser = {
      username: 'root',
      name: 'some other person',
      password: 'anotherexamplepassword',
    }

    const result = await api
      .post('/api/users')
      .send(newUser)
      .expect(400)
      .expect('Content-Type', /application\/json/)

    expect(result.body.error).toContain('`username` taken')

    const finalUsersState = await helper.dbUsers()
    expect(finalUsersState.length).toBe(initialUsersState.length)
  })
})
```

Here, our test expects that we'll receive an error message and a `400` status code response if we create a new user with a taken username. This test will not pass as we have not written the actual logic behind checking if a username is taken in our controller.

This practice of writing tests which defines the requirements of our functionality before writing the code for the functionality is described as *test-driven development*. In test-driven development, we turn our requirements into specific test cases and then improve our code so that our tests pass.

### Unique Validation

Let's improve our code so that the previous test will pass. To ensure that our username is unique, we'll take advantage of Mongoose validators. Recall that Mongoose validators comes with a very specific set of validators with uniqueness not being one; however, Mongoose validators are extensible and there is a package that provides exactly what we're looking for.

We can install this package by running:

```bash
npm install mongoose-unique-validator
```

We can now add this validator to our `user` model.

**/Backend/models/user.js**

```js
const mongoose = require('mongoose')
const uniqueValidator = require('mongoose-unique-validator')

const userSchema = new mongoose.Schema({
  username: { 
    type: String,
    unique: true
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
    delete returnedObject.passwordHash
  }
})

userSchema.plugin(uniqueValidator)

const User = mongoose.model('User', userSchema)

module.exports = User
```

Here, you'll see how we set the `username` property with an object that sets the type of the property to `String` and makes use of the `unique` validator that is given to us by binding the `uniqueValidator` module as a plugin to our schema.

Finally, we can write a GET route for our users controller so that we can see what we've done so far:

**/Backend/controllers/users.js**

```js
//...

usersRouter.get('/', async (req, res, next) => {
  try {
    const users = User.find({})
    res.json(users.map(u => u.toJSON()))
  } catch (exception) {
    next(exception)
  }
})

//...
```

Let's send a request to our server to create our first user:

**/Backend/requests/create_initial_users.rest**

```rest
post http://localhost:3001/api/users
Content-Type: application/json

{
	"username": "jassuf",
	"name": "joe",
    "password": "examplepassword123"
}
```

Now, visiting our users endpoint (**http://localhost:3001/api/users**) in a browser yields this result:

![image-20191211161916627](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211161916627.png)

### Assigning Objects to a User

We now have to figure out how to build the relationship between the user and its notes. We can change our notes controller to support this upon saving a new note:

```js
//...

const User = require('../models/user')

//...

notesRouter.post('/', async (req, res, next) => {
  const body = req.body

  const user = User.findById(body.userId)

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
    user: user._id
  })

  try {
    const savedNote = await newNote.save()
    user.notes = user.notes.concat(savedNote._id) // Adds the ID of the note we have just saved to the user's notes
    await user.save()
    res.status(201).json(savedNote.toJSON())
  } catch (exception) {
    next(exception)
  }
})

//...
```

Here, we expect that our requests will be sent with a parameter `userId` within its body to which we find the user with that `userId`. We then take this ID and add it to the `user` property of our notes object. Finally, after we save the note to our database, we take its ID and concatenate it to the given user's `notes` array property.

Let's update our **create_note.rest** request file to send the `userId` with its request as well:

**/Backend/requests/create_note.rest**

```rest
post http://localhost:3001/api/notes
Content-Type: application/json

{
	"content": "This was sent using REST Client and sends the user ID!",
	"flagged": true,
	"userId": "5df151019f8679503a3de183"
}
```

We manually inserted the userId given to us in the respond that we received from the user POST request:

![image-20191211164956957](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211164956957.png)

Upon sending the request to create the note, we receive the following response: 

![image-20191211165729249](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211165729249.png)

Now, let's explore this relationship by fetching all of the users at **http://localhost:3001/api/users**:

![image-20191211165830103](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211165830103.png)

Here, you can see how our user contains a `notes` property which has a value of an array that contains all of the IDs of the notes that the user owns. We can also fetch all of the notes to view how the notes contain a relationship to their "owner":

![image-20191211170045601](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211170045601.png)

Notice how the `user` property contains the ID of a `User` object that is the "owner" of this note.

### Data Population

Now, to achieve the final iteration of data structure that we discussed earlier, we would like our user objects to actually contain the contents/note objects rather than just their ID.

In a relational database, we would be able to use a SQL Join statement (used to combine data or rows from two or more tables based on a common field where the common field would be our note ID). While MongoDB in versions >= 3.2 has the `$lookup` aggregation operator which is quite similar to a left-outer join, Mongoose gives us access to `populate()` which lets us reference documents from other collections.

Now, one downside to this is that Mongoose accomplishes this join through multiple queries. In relational databases, join queries are *transactional*, ensuring that the state of the database cannot change while the query is executing. This means that we cannot guarantee that the state of the collections remains the same during our 'join' query, leading to some edge cases where the data that we receive may not match the true state of our database.

With this, let's actually perform the join using the `populate()` method:

**/Backend/controllers/users.js**

```js
//...

usersRouter.get('/', async (req, res) => {
  const users = await User
    .find({}).populate('notes')

  res.json(users.map(u => u.toJSON()))
})

//...
```

Here, we chain the `populate()` method after 'finding' all of our users, thus performing the populate operation on every single user document before the data is stored in the `users` constant. We pass in the property where we want references to be replaced with the actual documents. What this means is that any reference to an object in a collection found in the specified property will be replaced by the actual document, so that all the IDs found in the `notes` properties of all of the users will be replaced by the actual documents to which they refer to.

Mongoose knows that we are referring to the `notes` collection as we had provided the `ref` option earlier in our `User` schema. Without this, the database would not know that these IDs that we want to populate actually refer to documents in the `notes` collection:

```js
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    unique: true
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note' // We provide information as to what collection this ObjectId will refer to here
    }
  ],
})
```

Now, upon visiting this endpoint in our browser, we can see the actual note documents from within our user object:

![image-20191211194006275](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211194006275.png)

Now, let's improve on this implementation. We do not need the `user` field in the note object(s) as we have the relationship visible to us as the notes are now embedded in the user document. Let's filter out this field by specifying the fields that we want to include in an object that is used as the second parameter:

 **/Backend/controllers/users.js**

```js
//...

usersRouter.get('/', async (req, res) => {
  const users = await User
    .find({}).populate('notes', { content: 1, flagged: 1, date: 1 })

  res.json(users.map(u => u.toJSON()))
})

//...
```

Here, you can see how we are no longer given the `user` field.

![image-20191211194356591](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211194356591.png)

Let's apply these steps to our root notes endpoint:

```js
//...

notesRouter.get('/', async (req, res) => {
  const notes = await Note
    .find({}).populate('user', { username: 1, name: 1 })

  res.json(notes.map(note => note.toJSON()))
})

//...
```

Visiting this endpoint (**http://localhost:3001/api/notes**) in the browser, we receive the following response:

![image-20191211195040360](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211195040360.png)

Some of the content (the content that was not filtered out) is now included under the `user` property of our note's object rather than just its ID.

## Token-Based Authentication

Now, you might be wondering, how do we know which user is actually using our app and submitting the notes? Previously, we had just been manually inserting in the desired ID, but we need to add a way for users to log in to our application and for our application to know that this given user is currently signed in from its requests.

With this, we will add *token-based authentication* to our application. Tokens are one of the best ways to handle authentication for multiple users as it is easy to implement across various platforms, allows for stateless and scalable servers, extra security, and much more.

As the HTTP protocol is stateless, even after we authenticate a user, our backend would not know who the user is for every subsequent request unless we tell it so via the request. With this, we will expand upon this concept and create a token that will be passed to the client to which the client can use in every one of its requests.

The process of token-based authentication generally follows this sequence:

1. Server serves website to client.
2. User logs in with username and password on the client.
3. Client sends request to the server to authenticate.
4. The server takes the authentication request, and if the information is correct, it creates a signed token that is sent back to the client.
5. Client stores the token to be used with its requests.
6. **Subsequent requests to the server carries the token as a HTTP header** (the server will now respond to sensitive requests if this token is valid)

Here, nothing regarding a user's session state or records of the user's authentication is stored on our server which is a major benefit of token-based authentication. With what is called *server-based authentication*, the server would save user info in a session variable to which the client and server have to store information that is consistently compared for every request. This increases overhead with regard to having to create records of authentications, needing a designated area of session memory for users, having to account for CORS, and having to protect against CSRF (to a greater extent).

Let's apply the above logic to our application to see what we would have to do:

1. Create a form that the user can sign in with on the front end
2. Create an endpoint that takes in the information from the front end form
3. Create a function that generates a digitally signed token that identifies the user
4. Create the controller for the endpoint that generates the token and sends the response
5. Save the token in the state of our frontend application
6. Send the token in our requests from the frontend application
7. Check the token in the backend and identify the user

### Creating Tokens

With this, we can get started by first installing a package to our backend that helps us create JSON Web Tokens (industry standard token verification):

```bash
npm install jsonwebtoken --save
```

Next, let's create a file in our **/controllers** directory that handles our login routes:

**/Backend/controllers/login.js**

```js
const bcrypt = require('bcrypt')
const jwt = require('jsonwebtoken')
const loginRouter = require('express').Router()

const User = require('../models/user')

loginRouter.post('/', async (req, res) => {
  const body = req.body

  const user = await User.findOne({ username: body.username })
  const passwordCorrect = user === null // Checks if there is actually a user with the given username
    ? false
    : await bcrypt.compare(body.password, user.passwordHash)

  if (!user || !passwordCorrect) {
    return res.status(401).json({
      error: 'Invalid username or password'
    })
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  }

  const token = jwt.sign(userForToken, process.env.SECRET)

  res
    .status(200)
    .send({ token, username: user.username, name: user.name })
})

module.exports = loginRouter
```

Here, you can see how we attempt to find a user whose username matches the username that we were given in the body of the request. Then, we check if the `user` constant actually contains an object (indicating a successful find). If the `user` actually exists, then we check the password against the hashed password using the `bcrypt.compare()` method (how `bcrypt` actually goes about comparing the password to the hashed password is quite complex and would not be able to be explained in this book).

This call to `bcrypt.compare()` returns a promise that resolves into either `true` or `false` representing if the password is correct. Then, we have a logical block that will send a response with the status code `401` and the message, "Invalid username or password" if the user is missing or if the password is not correct.

After this, we encounter the block involving the `userForToken` object. We create this object as a stripped down version of our user as we only need the username and user ID to be digitally signed (in a token) for our purposes. This is because we just need to check the username/user ID in the token from a request to be able to know that the request is indeed coming from the authenticated user.

With this, we can create the token consisting of our object by calling `jsw.sign()` which takes in the object as its first parameter and a `SECRET` environment variable that we will specify. This environment variable is the key used to encrypt the value and is the only value that will be able to decrypt the data, thus is it important to keep it a secret. As this value is only stored on our server and is never sent, this ensures a way that we can verify and create digital signatures without worry of tampering.

After all of this is done, we simply just send the response back with a status of `200` (OK) and a body containing the token, the username of the user, and the name of the user.

Let's bind our login router to our Express application by adding the following to **app.js**:

**/Backend/app.js**

```js
//...

const loginRouter = require('./controllers/login')

//...

app.use('/api/login', loginRouter)

//...
```

Finally, we just need to specify the `SECRET` environment variable in our **.env** file:

```env
MONGODB_URI=mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority
PORT=3001
TEST_MONGODB_URI=mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/test-note-app?retryWrites=true&w=majority
SECRET=3298fhqw9uhvd87hgv9q78ehgv9qejgvqwdn
```

Just as a reminder, for your own projects, please generate your own value for `SECRET`. Now, we can test out our route by sending the following login request:

**/Backend/requests/login.rest**

```rest
post http://localhost:3001/api/login
Content-Type: application/json

{
	"username": "jassuf",
    "password": "examplepassword123"
}
```

Sending this request yields the following response with our token:

![image-20191214015124573](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191214015124573.png)

Here, you can see how a token is created and is returned in the body of the response so that we can now use it in our client.

Let's try sending a request with an incorrect username or password and see how our server responds:

**/Backend/requests/login_incorrect.rest**

```rest
post http://localhost:3001/api/login
Content-Type: application/json

{
	"username": "jassuf",
    "password": "wrongpassword123"
}
```

This request yields the following result:

![image-20191214015354863](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191214015354863.png)

We can observe that our controller had handled the error correctly with the correct status code and error message.

### Requiring Authentication

Now, let's actually make use of our tokens by limiting the ability to create notes to only requests that have a valid token.

There are multiple ways of sending a token with our request. We'll be taking advantage of the HTTP `Authorization` request header which takes in an authentication type/schema and the actual credentials (our token).

We will specify the authentication scheme as `Bearer` as it fits our token. This type is combined with our token in the `Authorization` header, separated by a space:

```token
Bearer exampleTokenContentsDFJ213ixbcajne12udSfhaSFsbDJfbkHBfjhahsbh
```

Now, let's update our controller involving creating a new note to require a valid token:

**/Backend/controllers/notes.js**C 

```js
//...

const jwt = require('jsonwebtoken')

//...

const getTokenFrom = req => {
  const authorization = req.get('authorization')
  if (authorization && authorization.toLowerCase().startsWith('bearer ')) {
    return authorization.substring(7)
  }
  return null
}

//...

notesRouter.post('/', async (req, res, next) => {
  const body = req.body

  const token = getTokenFrom(req)

  try {
    const decodedToken = jwt.verify(token, process.env.SECRET)
    if (!token || !decodedToken.id) {
      return res.status(401).json({ error: 'missing or invalid token' })
    }

    const user = await User.findById(decodedToken.id)

    const newNote = new Note({
      content: body.content,
      flagged: body.flagged || false,
      date: new Date(),
      user: user._id
    })


    const savedNote = await newNote.save()
    user.notes = user.notes.concat(savedNote._id) // Adds the ID of the note we have just saved to the user's notes
    await user.save()
    res.status(201).json(savedNote.toJSON())
  } catch (exception) {
    next(exception)
  }
})
```

Here, we define a helper function, `getTokenFrom`, which is purely responsible for retrieving the token from the `Authorization` header. Within this helper function, we check if it is indeed of a `bearer` type/scheme, if it is not, we simply return `null`.

Then, in our controller, we attempt to verify the token and decode the token into the original stripped down user object (`userForToken`) that we created in our login controller. We check if either the token does not exist or if we don't have an ID property of `decodedToken` (indicating an invalid token); if any of these conditions are true, we response with the status code `201` and an error message.

Within our token, we have an object that contains our user's `username` and `id`. We can use the `id` value to search our user's collection by ID and find the user that submitted the note. With this, we can now create the note and attach it to the user that we found as we did before.

Let's test our controller by sending a request:

**/Backend/requests/create_note_with_auth.rest**

```rest
post http://localhost:3001/api/notes
Content-Type: application/json
Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Imphc3N1ZiIsImlkIjoiNWRmMTUxMDE5Zjg2Nzk1MDNhM2RlMTgzIiwiaWF0IjoxNTc2NDcyNTQ3fQ.b17lRenxqf7xSKLGqmsCYbCfP7vflDTJEKC6VupE02s

{
    "content": "This note was sent by an authorized user",
    "flagged": false
}
```

Upon sending this request, we receive the following response:

![image-20191216000413198](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191216000413198.png)

Please bear in mind that some of our tests will now fail as we had not updated them to use tokens in order to send valid requests.

### Handling Token Verification Errors

When using `JsonWebToken`, we can encounter `JsonWebTokenError`s for various reasons. One of the most common reasons that we would encounter this exception is if we were to send an invalid token (through deleting a few characters from the token, trying to fake a token, etc.) to which we'd encounter the following error:

```error
JsonWebTokenError: invalid signature
	at...
```

We do not want to leave unhandled exceptions, so we will add support for these errors in our `errorHandler` middleware:

```js
const errorHandler = (err, req, res, next) => {
  logger.error(err.message)

  if (err.name === 'CastError' && err.kind === 'ObjectId') {
    return res.status(400).send({ error: 'Incorrect format for id parameter' })
  } else if (err.name === 'ValidationError') {
    return res.status(400).json({ error: err.message })
  } else if (err.name === 'JsonWebTokenError') {
    return res.status(400).json({ error: 'Invalid token' })
  }

  logger.error(err)

  next(err)
}
```

### Practicality

While going over the process required to verify and use the object within our token, you might have wondered if we could abstract this process to a middleware so that we could make use of it in multiple controller. This could be done through the use of a package like `express-jwt` which does the above task of validating a `JsonWebToken` to which it attaches the resulting object to the `user` property of the `req` object, allowing us to authenticate HTTP requests without writing the code that we did above for every controller.

## Frontend User Management

Let's now implement signing in and adding tokens to our requests so that we can add notes from our React app (as we updated our backend to require authentication).

If we try launching our frontend, you'll find that we can no longer add notes (since we implemented and required authentication in the controller for posting notes), yet we can still update the flagged property as we had not required or implemented authentication there.

Let's add a form so that we can sign in:

**/Frontend/src/app.js**

```js
import React, { useState, useEffect } from 'react'
import Note from './components/Note'
import Notification from './components/Notification'
import Separator from './components/Separator'

import axios from 'axios'

import noteService from './services/notes'

const App = (props) => {
  const [notes, setNotes] = useState([])
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
  const [notificationMessage, setNotificationMessage] = useState()
  const [username, setUsername] = useState('') // Add username piece to our state
  const [password, setPassword] = useState('') // Add password piece to our state

  //...

  const handleLogin = (event) => { // Form event handler, prevents page submission upon submitting the form
    event.preventDefault()
    console.log('logging in:', username, password)
  }

  //...

  return (
    <div>
      <h1>Notes</h1>
      
      <form onSubmit={handleLogin}>
        <div>
          Username
            <input
            type="text"
            value={username}
            name="Username"
            onChange={({ target }) => setUsername(target.value)}
          />
        </div>
        <div>
          Password
            <input
            type="password"
            value={password}
            name="Password"
            onChange={({ target }) => setPassword(target.value)}
          />
        </div>
        <button type="submit">Login</button>
      </form>
      
			//...

    </div>
  )
}

export default App;
```

Here, we first add two pieces of state to hold our username and password. Then, we add a form, similarly to how we added a form for submitting notes earlier. 

You'll notice that our `input` elements within our form have handlers assigned to the `onChange` events. This is so that we can update our pieces of state, `username` and `password`, to reflect the changes that the user is attempting to make in the input box. Since these `input` elements have their `value` property set to a piece of state, they are now considered *controlled components* where it takes its current value through props and depends on an external state. Thus, for the user to be able to type and make changes in the `input` element, we have to specify a handler to the `onChange` event so that we can modify the state as such:

```js
onChange={({ target }) => setPassword(target.value)}
```

This event handler simply gets the object that it is given and accesses the value property to call our state setter upon.

We also provided an event handler to the `onSubmit` event of our `form` element. Right now, the event handler just makes use of the `preventDefault()` method of the event in order to prevent the page from refreshing upon a submission (just like we did with the note submission form). The `handleLogin` handler also logs the `username` and `password` pieces of state upon being calld:

```js
const handleLogin = (event) => { // Form event handler, prevents page submission upon submitting the form
    event.preventDefault()
    console.log('logging in:', username, password)
}
```

Recall that we must make a HTTP POST request to **/api/login** in order to authenticate. We must also keep track of the token that is returned from a successful login, with this, as we may reuse this request multiple times, and write a considerable amount of code surrounding the request, we'll create a separate module for this request in **services**:

**/Frontend/services/login.js**

```js
import axios from 'axios'
const baseUrl = '/api/login'

const login = async credentials => {
  const res = await axios.post(baseUrl, credentials)
  return res.data
}

export default { login }
```

You may notice that we're able to send objects using `axios` by simply passing it as the second parameter.

Now, over in our `App` component, let's make use of this service in our `handleLogin()` function:

**/Frontend/src/App.js**

```jsx
//...

import loginService from './services/login' 

//...

const [user, setUser] = useState(null)

//...

const handleLogin = async (event) => {
    event.preventDefault()
    try {
      const user = await loginService.login({
        username, password,
      })

      setUser(user)
      setUsername('')
      setPassword('')
      
    } catch (exception) {
      console.log(exception)
      setNotificationMessage('Incorrect credentials: ' + exception)
      setTimeout(() => {
        setNotificationMessage(null)
      }, 3000)
    }
  }

//...
```

Stepping through this method, you'll see that we use our service to log in using our `username` and `password` pieces of state. Then, if we don't encounter an error, we store the `user` (containing user details and a token) that is returned into our `user` piece of state and replace the form fields with nothing. Other than the login form being cleared, our user is not given any indication of a successful login. Let's improve the design and usability of our application so that the login form disappears and the new notes form appears if the user is signed in and vice versa (giving the user an indication that they had successfully signed in):

**/Frontend/src/App.js**

```jsx
//...

const App = () => {
  
  //...
  
  const loginForm = () => ( // Login form JSX
    <form onSubmit={handleLogin}>
      <div>
        Username
          <input
          type="text"
          value={username}
          name="Username"
          onChange={({ target }) => setUsername(target.value)}
        />
      </div>
      <div>
        Password
          <input
          type="password"
          value={password}
          name="Password"
          onChange={({ target }) => setPassword(target.value)}
        />
      </div>
      <button type="submit">Login</button>
    </form>      
  )

  const noteForm = () => ( // Note form JSX
    <form onSubmit={addNote}>
      <input
        value={noteField}
        onChange={handleInputChange}
      />
      <button type="submit">Add Note</button>
    </form>  
  )
  
  return (
    <div>
      <h1>Notes</h1>
      
      {user === null && loginForm()} {/* Conditionally render */}

      <Notification message={notificationMessage} />
      
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          Show {showAll ? 'Flagged' : 'All' } 
        </button>
      </div>
      
      <ul>
        {noteList()}
      </ul>
      
      {user !== null && noteForm()} {/* Conditionally render */}
      
      <Separator />
      
    	</div>
  	)
	}
}

//...
```

Here, to make our code easier to read, we separated out our forms into functions that return JSX when called. Then, we conditionally render these forms by taking advantage of short-circuit evaluation which causes the second statement that generates and renders the form to not be executed if the first statement is falsy. This is because the computer knows that if one statement in the `&&` expression is false, the entire expression must be false, so it stops executing.

Let's take this a step further and apply the conditional operator to our code. This requires us to move our notes form up to where our login form is:

**/Frontend/src/App.js**

```js
//...

	return (
    <div>
      <h1>Notes</h1>
      
      {user === null ?
    		loginForm() :
  			<div>
          <p>Welcome, {user.name}</p>
          {noteForm()}
				</div>
    	} {/* Conditionally render */}

      <Notification message={notificationMessage} />
      
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          Show {showAll ? 'Flagged' : 'All' } 
        </button>
      </div>
      
      <ul>
        {noteList()}
      </ul>
      
      <Separator />
      
    	</div>
   )
}

//...
```

Here, when `user === null` evaluates to false, we actually render a `div` that contains a new paragraph element that welcomes the user by their name as well as the notes form.

#### Sending Auth Token

From the code above, we know that the `user` piece of state contains information about our user as well as our authentication token of the user. Now, to authenticate our requests, we simply have to add the token to the `Authorization` header of the requests.

Let's update our notes POST request:
**/Frontend/src/services/noteService.js**

```js
import axios from 'axios'
const baseUrl = '/api/notes'

let token = null

const setToken = userToken => {
  token = `bearer ${userToken}`
}

const getAll = () => {
  const req = axios.get(baseUrl)
  return req.then(response => response.data)
}

const create = async newObject => {
  const config = {
    headers: { Authorization: token },
  }
  
  const res = await axios.post(baseUrl, newObject, config)
  return res.data
}

const update = (id, newObject) => {
  const req = axios.put(`${baseUrl}/${id}`, newObject)
  return req.then(res => res.data)
}

export default {
  setToken: setToken,
  getAll: getAll, 
  create: create, 
  update: update
}
```

Here, we made a few updates, including updating our variable names to their shortened form to better reflect our Express application. With regard to updating our token, we created a `setToken()` function that updates the value of `token` (which is a private variable that belongs to the module).

Our `create()` function now uses `async`/`await` syntax and sets the appropriate `Authorization` header through an object given as the third parameter of the `axios` method call. Let's also update our `handleLogin()` function to set the token in our `notes` service module:

**/Frontend/src/App.js**

```js
//...

  const handleLogin = async (event) => {
    event.preventDefault()
    try {
      const user = await loginService.login({
        username, password,
      })

      setUser(user)
      setUsername('')
      setPassword('')
      noteService.setToken(user.token)
      
    } catch (exception) {
      console.log(exception)
       setNotificationMessage('Incorrect credentials: ' + exception)
      setTimeout(() => {
        setNotificationMessage(null)
      }, 3000)
    }
  }
  
//...
```

With this, we should now be able to post notes and pass authentication.

###Local Storage

Upon using the application, you might have discovered that you do not stay signed in upon a page refresh (or if we had to visit a different page) unlike other websites. This is because, when we authenticate, we simply save our token and user information in our application's state which is reset upon a refresh, etc. If we wanted this information to persist, just like we used a database on our backend, we can use a key-value database that comes with our browser to store our login information.

Using the `localStorage`/`Storage` Web API is easy. Let's change our application so that it saves the user's information and authentication token to the local storage:

**/Frontend/src/App.js**

```jsx
//...

	const handleLogin = async (event) => {
    event.preventDefault()
    try {
      const user = await loginService.login({
        username, password,
      })

      setUser(user)
      setUsername('')
      setPassword('')
      window.localStorage.setItem( // Saves user information to local storage
        'noteAppUser', JSON.stringify(user)
      ) 
      noteService.setToken(user.token) 
      
    } catch (exception) {
      console.log(exception)
      setNotificationMessage('Incorrect credentials: ' + exception)
      setTimeout(() => {
        setNotificationMessage(null)
      }, 3000)
    }
	}

//...
```

Here, we take the user object and convert it to a JSON string so that we can save the object to local storage as the local storage will only take `DOMstrings` (UTF-16 strings). We then call `window.localStorage.setItem()` and pass in the key (`noteAppUser`) and value (the stringified object).

Later on, when we read from the local storage, in order to get our object from the string, we must call `JSON.parse()` upon it.

Now, we actually have to figure out a way to check if there is a user object that we need to handle in the local storage. Recall that the `useEffect()` hook allows us to execute side effects after a component has rendered. We can utilize this hook to check if there is a user and to handle that user after our component has rendered:

```jsx
//...

const App = (props) => {
  const [notes, setNotes] = useState([]) // Updated to not use the state passed into the component as we'll be fetching it from the server
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
  const [notificationMessage, setNotificationMessage] = useState()
  const [username, setUsername] = useState('') 
  const [password, setPassword] = useState('') 
  const [user, setUser] = useState(null)

  const loadNotesEffect = () => {
    noteService
      .getAll()
        .then(initialNotes => {
        setNotes(initialNotes)
      })
      .catch(error => {
        setNotificationMessage(`${error}: Something went wrong while trying to retrieve data from the server.`)
      })
  }
  
  const signedInEffect = () => {
    const loggedInUserJSON = window.localStorage.getItem('noteAppUser')
    if (loggedInUserJSON) {
      const user = JSON.parse(loggedInUserJSON)
      setUser(user)
      noteService.setToken(user.token)
    }
  }

  useEffect(loadNotesEffect, [])
  useEffect(signedInEffect, [])
  
//...  
```

Here, we created a function, `signedInEffect`, which uses the local storage API to attempt to fetch the value of the `'noteAppUser'` key in the local storage. If there happens to be a string at that location, we parse the string into an object and set our `user` piece of state as well as change the private `token` variable in our notes service module based off of this information.

We then register this function as an effect by calling `useEffect` upon it. Recall that providing the empty array as the second parameter ensures that this effect will only execute after the first component render and no more. This is because React checks if the variables specified in the array passed as the second parameter have changed between renders, and will only execute the effect function if the variables had changed in values; as nothing will always equal nothing, or an empty array will always equal an empty array, this ensures that we just execute the function once, because all checks for changes will return false.

With this implementation, a user stays logged in until the browser's local storage is cleared with no way for the user to manually sign out. Let's add a logout button to our application:

**/Frontend/App.js**

```jsx
//...

const handleLogout = async (event) => {
    event.preventDefault()
    try {
      window.localStorage.removeItem('noteAppUser')
      noteService.setToken(null)
      setUser(null)
    } catch (exception) {
      console.log(exception)
    }
  }

//...
```

Here, we simply created a handler function for our logout button that removes the user information from local storage so that we are no longer automatically signed in upon loading the page. Additionally, we set our `user` piece of state and the token to `null` to 'log out' in our current session. Now, we just have to add the button to the JSX that our component returns:

**/Frontend/src/App.js**

```jsx
//...  
​````````
	return (
    <div>
      <h1>Notes</h1>
      
      {user === null ?
    		loginForm() :
  			<div>
          <p>Welcome, {user.name}</p> 
        	<button onClick={handleLogout}>Sign Out</button>
          {noteForm()}
				</div>
    	} {/* Conditionally render */}
      
      {/* ... */}
```

Here, we conditionally render the sign out button if the user is signed in. This button calls the `handleLogout` handler function upon the `onClick` event (when the button is clicked) which logs the user out by removing the user information from local storage.

