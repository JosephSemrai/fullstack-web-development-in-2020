## Server Communication with Redux

Up until this point, our new simplified notes application that makes use of Redux does not do any communication with a backend server.

Now, bringing in an API with a Redux application comes with its own design paradigms which we'll explore in a moment. Instead of creating an entire backend once again (or adapting our previous backend), we'll make use of the powerful development tool `json-server` that we used much earlier.

Let's install `json-server` to our new project by running:

```bash
npm install json-server --save-dev
```

Recall that `json-server` will create a "mock API" that we can use in our frontend based off of a file, (in our case) **db.json**. Let's create the file in the root of our project directory:

**/db.json**

```json
{
    "notes": [
        {
        "textContent": "This was served by json-server",
        "flagged": false,
        "id": 1
        },
        {
        "textContent": "We're keeping track of all of these notes in the Redux Store",
        "flagged": true,
        "id": 2
        }
    ]
}
```

We can then go ahead and create a npm script that launches our `json-server` with the port `3001` based off of the file **db.json**:

```json
//...
    "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "test": "react-scripts test",
        "eject": "react-scripts eject",
        "lint:fix": "eslint . --fix",
        "server:start": "json-server -p3001 db.json"
    },
//...
```

We can now run the server by executing:

```bash
npm run server:start
```

With our server up and running, we can now go ahead and add a service in our React application so that we can communicate with our backend. We'll create our notes service in **/src/services/noteService.js**:

**/src/services/noteService.js**

```jsx
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

const getAll = async () => {
  const res = await axios.get(baseUrl)
  return res.data
}

export default { getAll }
```

> `getAll` makes use of the async/await syntax.

We had never installed axios, so we can install it by running:

```bash
npm install axios
```

### Initializing State from the Backend

We have the ability to communicate with our backend now, but how do we actually make our application reflect that with its state dependent on the Redux store?

One "hacky" solution involves fetching the notes and just dispatching a `NEW_NOTE` action for every note that we receive:

**/src/index.js**

```jsx
//...

import noteService from './services/noteService' // Import our Notes Service

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const store = createStore(reducer)

noteService.getAll().then(notes => // Gets all notes using the Notes Service and dispatches a NEW_NOTE action for each note
  notes.forEach(note => {
    store.dispatch({
      type: 'NEW_NOTE',
      data: note
    });
  })
);


ReactDOM.render(
  <Provider store={store}> 
    <App store={store} />
  </Provider>,
  document.getElementById('root')
)
```

This gets the job done, but we can do better. Let's add support for a new `INIT_NOTES` action in our notes reducer so that we can initialize our notes by dispatching only one action:

**/src/reducers/noteReducer.js**

```jsx
//...

const noteReducer = (state = [], action) => {
  switch (action.type) {
  case 'INIT_NOTES':
    return action.data // Simply sets the `notes` part of the store to the action's data
  case 'NEW_NOTE':
    return [...state, action.data]
  case 'TOGGLE_FLAGGED': {
    const noteId = action.data.id
    const originalNote = state.find(n => n.id === noteId)
    const updatedNote = {
      ...originalNote,
      flagged: !originalNote.flagged
    }
    return state.map(n => n.id === noteId ? updatedNote : n)
  }
  default:
    return state
  }
}

//...
```

We can also create an action creator for using this new action type:

**/src/reducers/noteReducer.js**

```jsx
//...

export const initNotes = (notes) => {
  return {
    type: 'INIT_NOTES',
    data: notes,
  }
}

//...
```

Finally, we can make use of this new action in **index.js** where our initialization simplifies to:

**/src/index.js**

```jsx
//...

import noteReducer, { initNotes } from './reducers/noteReducer'

//...

noteService.getAll().then(notes => // We get all notes and use the `initNotes` action creator with `notes` passed to it to dispatch an action that will initialize the `notes` part of our store
  store.dispatch(initNotes(notes))
);

//...
```

We don't make use of `mapDispatchToProps` here since we are not in a React component. Let's move this initialization logic into our `App` component:

**/src/App.js**

```jsx
import React, { useEffect } from 'react' // Import useEffect which will call initNotes
import { connect } from 'react-redux' // Import connect so we can create our connected component

import Notes from './components/Notes'
import NewNote from './components/NewNote'
import NoteFilter from './components/NoteFilter'

import { initNotes } from './reducers/noteReducer' // Imports our `initNotes` action creator

import noteService from './services/noteService' // Imports our notes service so that we can fetch our notes

const App = ({ initializeNotes }) => { // Destructures `initNotes` provided to our connected component through its props

  useEffect(() => {
    noteService.getAll().then(notes => initializeNotes(notes)) // Retrieves all notes and then calls the `initNotes` prop upon the object it retrieves
  }, [initializeNotes])


  return (
    <div>
      <NewNote />
      <NoteFilter />
      <Notes />
    </div>
  )
}

const ConnectedApp = connect(null, { initializeNotes: initNotes })(App) // Creates a connected component with a mapDispatchToProps object
export default ConnectedApp
```

Read the comments in the code above to see every single change that we had made and why we made them. In short, we make use of `react-redux` to create a connected component that has the `initNotes` action creator passed to it as a dispatcher function through its props. This `initNotes` dispatcher (created automatically by `connect()()` from our action creator) is called in the function specified to `useEffect()` upon the fulfillment of the `noteService.getAll()` promise. We make use of the effect hook as we do not want to fetch the data from the server every single time our component renders (we register `initializeNotes` as a dependency of the effect hook as to make sure that we call the function if `initializeNotes` is ever updated and to silence React warnings).

Let's extend this logic to creating new notes so that our notes can be reflected in our "database" as well:

**/src/services/noteService.js**

```jsx
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

const getAll = async () => {
  const res = await axios.get(baseUrl)
  return res.data
}

const create = async textContent => { // New service that posts a note object to our "database"
  const note = { 
    textContent,
    flagged: false 
  }
  const res = await axios.post(baseUrl, note)
  return res.data
}

export default { 
  getAll,
  create
}
```

We can then update the `addNote` method in the `NewNote` component to make use of this service:

**/src/components/NewNote.js**

```jsx
import React from 'react'
import { connect } from 'react-redux'

import { createNote } from '../reducers/noteReducer' // Relevant imports

import noteService from '../services/noteService' // Relevant imports

const NewNote = ({ createNewNote }) => {
  
  const addNote = async (event) => {
    event.preventDefault()
    const textContent = event.target.noteField.value
    event.target.noteField.value = '' // Moved before the awaited method call to preserve atomic updates
    const receivedNote = await noteService.create(textContent) // Uses the create method of our service
    createNewNote(receivedNote) // Dispatches the `createNote` action using the received note from the post request
  }
  
  return (
    <form onSubmit={addNote}>
      <input name="noteField"/> 
      <button type="submit">Create Note</button>
    </form>
  )
}

//...
```

The comments above describe the changes that we had made to the code. Essentially, we just send a post request using the `create()` method of our notes service and use the returned note with the `createNewNote()` dispatcher to update our state (instead of the text content).

As we are now passing in the entire note object rather than just the text content to our `createNewNote` dispatcher, we can update the `createNote` action creator to just take in the note without generating an ID (since that's handled for us in the backend):

**/src/reducers/noteReducer.js**

```jsx
//...

export const createNote = (data) => {
  return {
    type: 'NEW_NOTE',
    data
  }
}

//...
```

Finally, we'll add the ability to update a note in our "backend":

**/src/services/noteService.js**

```jsx
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

const getAll = async () => {
  const res = await axios.get(baseUrl)
  return res.data
}

const create = async textContent => {
  const note = { 
    textContent,
    flagged: false 
  }
  const res = await axios.post(baseUrl, note)
  return res.data
}

const update = async (id, newObject) => { // Added update method that takes in the id and the object to replace it with
  const res = await axios.put(`${baseUrl}/${id}`, newObject)
  return res.data
}

export default { 
  getAll,
  create,
  update
}
```

Now, let's make use of this new service method that we had created in our `Notes` component where we'll create a `toggleNoteFlagged()` wrapper function around our `toggleFlagged()` dispatcher that will both make the appropriate API calls and update our local state:

**/src/components/Notes.js**

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import { connect } from 'react-redux'
import Note from './Note'
import noteService from '../services/noteService'

const Notes = ({ 
  notes,
  toggleFlagged 
}) => {

  const toggleNoteFlagged = async note => {
    const toggledNote = { // Create the updated note with the toggled flag
      ...note,
      flagged: !note.flagged
    }
    await noteService.update(note.id, toggledNote) // Update the note with the updated note
    toggleFlagged(note.id) // Toggle the flagged property in our local state (in a proper solution, we should update the entire object upon receiving a response)
  }

  return (
    <ul>
      {notes.map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleNoteFlagged(note)}
        />
      )}
    </ul>
  )
}


const noteList = ({ notes, filter }) => {
  if (filter === 'FLAGGED') {
    return notes.filter(n => n.flagged)
  }
  return notes
}

const mapStateToProps = (state) => {
  return {
    notes: noteList(state)
  }
}

const mapDispatchToProps = {
  toggleFlagged: toggleFlaggedOf
}

const ConnectedNotes = connect(
  mapStateToProps,
  mapDispatchToProps
)(Notes)
export default ConnectedNotes
```

### Abstraction with Redux-Server Communication

From the code that we have written so far, you might have been wondering if we could refactor the code into something cleaner and easier to understand (especially in the case of functions like `toggleNoteFlagged()`).

Currently, we're communicating with the server using functions that reside inside of our functions. Let's apply the "separation of concerns" design pattern and attempt to abstract these functions away from the insides of our components.

`redux-thunk` allows us to do this by enabling us to write action creators that return a function instead of an action creator. These functions, called *thunks*, receive `dispatch()` as an argument and can call it asynchronously. This allows us to communicate with the server from within the action creators that are created instead of writing an entire wrapper function that calls the dispatcher and the relevant statements to communicate with the server (like we did with `toggleNoteFlagged()`).

To illustrate this, our initialization of notes would be simplified from this:

```jsx
useEffect(() => {
    noteService.getAll().then(notes => initializeNotes(notes))
  }, [initializeNotes])
```

to this:

```jsx
useEffect(() => {
    initializeNotes(notes)
  },[initializeNotes])
```

This is because we would be able to perform the server communication involving fetching the notes in the function that is returned (as we are no longer just returning the action).

Let's start by installing `redux-thunk` by executing:

```bash
npm install redux-thunk
```

`redux-thunk` is a *redux middleware*. Recall that middleware in Express is simply some piece of code that you can put between some code during the lifecycle of a request to the server. Redux middleware implements this concept by providing a third-party extension point between dispatching an action and reaching the reducer. Thus, you have an extension point where you can execute code that will be executed after you dispatch an action, but before the reducer receives it (like how you can use Express middleware to modify the `req` object before it actually reaches the controller).

Before we work with `redux-thunk`, let's organize our project further by placing the initialization of our store and combining of our reducers in its own module:

**/src/store.js**

```jsx
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'

import filterReducer from './reducers/filterReducer'
import noteReducer from './reducers/noteReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const store = createStore(reducer, applyMiddleware(thunk))

export default store
```

Here, we create our store with our combined reducers and add the `thunk` middleware using `applyMiddleware()`. `applyMiddleware()` is a *store enhancer* which is a higher-order function that composes a store creator to return a new, enhance store creator. It is unlikely that you will have to work store enhancers, but it is helpful to understand what `applyMiddleware()` actually is.

We can then just import `store` from the module in **index.js**, dramatically simplifying the file:

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import App from './App'

import store from './store'

ReactDOM.render(
  <Provider store={store}> 
    <App store={store} />
  </Provider>,
  document.getElementById('root')
)
```

Let's write our first *thunk* involving the initialization of notes in our application:

**/src/reducers/noteReducer.js**

```jsx
//...

export const initNotes = () => {
  return async dispatch => {
    const notes = await noteService.getAll()
    dispatch({
      type: 'INIT_NOTES',
      data: notes
    })
  }
}

//...
```

Here, `initNotes` (the *thunk* or wrapper function) returns an asynchronous function that awaits the retrieval of notes from the backend and dispatches a `INIT_NOTES` action with the notes in order to initialize our store.

In our `App` component, we can go ahead and use the simplified code that we talked about earlier:

**/src/App.js**

```jsx
import React, { useEffect } from 'react'
import { connect } from 'react-redux'

import Notes from './components/Notes'
import NewNote from './components/NewNote'
import NoteFilter from './components/NoteFilter'

import { initNotes } from './reducers/noteReducer'

const App = ({ initializeNotes }) => {

  useEffect(() => {
    initializeNotes()
  }, [initializeNotes])

  return (
    <div>
      <NewNote />
      <NoteFilter />
      <Notes />
    </div>
  )
}

const ConnectedApp = connect(null, { initializeNotes: initNotes })(App) 
export default ConnectedApp
```

Here, we can just call `initNotes` and watch as it updates our store using data that has been fetched in the thunk. One thing to note is that: when we call `connect()()`, Redux will no longer add the `dispatch` logic for us as we are now defining it in the function form rather than the object shorthand form.

The logic for initializing our notes now resides outside of any of our components. We can apply the same logic to `createNote`:

```jsx
//...

export const createNote = textContent => {
  return async dispatch => {
    const newNote = await noteService.create(textContent)
    dispatch({
      type: 'NEW_NOTE',
      data: newNote
    })
  }
}

//...
```

With thunks, our wrapper function (the actual thunk) returns a curried function with any of the parameters that we had passed in, to which Redux will pass in `dispatch()` and execute the function when we call the wrapper function.

> The function returned by our asynchronous action creators (thunks) also receive the store's `getState()` method which allows us to read values from the current state of the store by calling `getState()`.

Our `NewNote` component now simply just calls the `createNewNote` thunk rather than making the API requests itself in `addNote()`:

```jsx
import React from 'react'
import { connect } from 'react-redux'

import { createNote } from '../reducers/noteReducer'

const NewNote = ({ createNewNote }) => {
  const addNote = (event) => {
    event.preventDefault()
    const textContent = event.target.noteField.value
    event.target.noteField.value = ''
    createNewNote(textContent)
  }
  
  return (
    <form onSubmit={addNote}>
      <input name="noteField"/> 
      <button type="submit">Create Note</button>
    </form>
  )
}

const mapDispatchToProps = {
  createNewNote: createNote
}

const ConnectedNewNote = connect(
  null,
  mapDispatchToProps
)(NewNote)
export default ConnectedNewNote
```

At this point, we're managing much of our application's state using Redux. This is often considered the "right" way to use React with it primarily focused on generating views with the application state being found in Redux with its actions, reducers, etc. However, there still are cases when we might be better served by keeping some logic within React. For example, if we had a simple form whose state does not need to be reflected across the application, the `useState()` hook would serve us just fine over using Redux.

### Redux DevTools

It may be hard to keep track of the state of the Redux store and how actions are being dispatched. Luckily, there is a chrome extension, Redux DevTools, that can assist us with this process.

We can start by installing [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd) by remotedevio in the Chrome Web Store. Then, in order for this extension to be aware of the state of the store, our enhancers, and middleware, we need to wrap all of this in `composeEnhancers()` provided by `redux-devtools-extension`. Let's install `redux-devtools-extension` by running:

```bash
npm install redux-devtools-extension
```

We can then modify our `store` module by wrapping all of our enhancers/middleware in `composeWithDevTools()` and then passing the return value of that function into the second argument of `createStore()`:

**/src/store.js**

```jsx
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import { composeWithDevTools } from 'redux-devtools-extension'

import filterReducer from './reducers/filterReducer'
import noteReducer from './reducers/noteReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const enhancer = composeWithDevTools(
  applyMiddleware(thunk)
)

const store = createStore(reducer, enhancer)

export default store
```

Upon restarting our browser, a new "Redux" tab will appear in our Chrome Developer Tools. Upon viewing the tab, we'll see information relating to the store of our application:

![Redux Preview](https://i.imgur.com/j2xM5pM.png)

There are various features that assist us with development. One notable thing that we can do with this is dispatch actions manually from the developer tools:

![Redux Demonstration](https://i.imgur.com/HeH2fJE.png)



# Chapter 35135: React Navigation, Unpacking Create-React-App, and Component Libraries

## Navigation: React Router

Within React applications, we often need the ability to provide the user with the ability to navigate between pages, visit specific pages at certain URLs, etc. In traditional web applications, your browser would make a request to a completely different URL to which it would receive the HTML for the page to render in its response. For example, you could click a button with a `href` element that would direct you to the home page of an application, to which your browser would load the resources for that page and display it.

What we have been making with React so far is a single-page application (you can make multi-page applications with server-side rendering with React, but we won't be covering that). Single-page applications are, in reality, just one page (hence the name). We can provide the illusion of different pages by displaying different "page" components, etc. The only HTTP requests that we make following the page load are for fetching JSON data, not view data.

We could provide the illusion of multiple pages (assuming that we have components for the pages `Home`, `Notes`, and `Users`) as follows:

```jsx
//...

const App = () => {
 const [page, setPage] = useState('home') // State tracking the page that we should render

  const toPage = (page) => (event) => { // Updates the state
    event.preventDefault()
    setPage(page)
  }

  const content = () => { // Returns the component that we should render dependent on the state
    if (page === 'home') {
      return <Home />
    } else if (page === 'notes') {
      return <Notes />
    } else if (page === 'users') {
      return <Users />
    }
  }

  return (
    <div>
      <div> {/* Start navigation bar */}
        <a href="" onClick={toPage('home')}>
          Home
        </a>
        <a href="" onClick={toPage('notes')}>
          notes
        </a>
        <a href="" onClick={toPage('users')}>
          users
        </a>
      </div> {/* End navigation bar */}
      {content()} {/* Content of "page" */}
    </div>
  )
}
```

Here, we create a helper function, `toPage()` that takes in the name of the page component that we want to load. This function prevents the default behavior of an anchor element (which is to redirect the page to the value specified to `href`) and sets the `page` piece of state that we're on. This affects the page that is rendered, because in our `content()` method (which returns the JSX of the page that we render), we check the `page` piece of state and return the component that we want to render.

All of this combines to provide the illusion that we're navigating between three pages: "Home", "Notes", and "Users". In reality, we're just calling functions that cause different components to be rendered by taking advantage of conditional renders dependent on the `page` piece of state.

We still do not provide the complete experience that a traditional web application would in that the address does not change at all when we navigate between these "pages". This may not seem like too big of a problem right now, but it is considered good practice to have most views accessible by a unique address (so that bookmarks, linking to others, etc. still works). The browser's navigation (back and forward) does not work as expected either since it is unaware that we had visited another "page".

We could manually provide an illusion here using the DOM `Window`'s History API which allows us to navigate through the users history stack as well as modify the address bar. This is not feasible with larger projects, however, as it will become very challenging to manage routing (referring to the navigation of our application) when we add more components, ways to access them, etc.

A solution for this problem, as with most things in the JavaScript world, comes in the form of a npm package that handles most of our routing needs for us. This package, `react-router`, provides a way for us to declaratively route our applications.

We'll first install the package `react-router-dom` by running:

```bash
npm install react-router-dom
```

This allows us to use DOM elements with bindings for React Router.

We can use these bindings to emulate the above functionality as follows:

```jsx
//...

import {
  BrowserRouter as Router,
  Route, Link, Redirect, withRouter
} from 'react-router-dom'

const App = () => {

  return (
    <div>
      <Router>
        <div>
          <div>
            <Link to="/">home</Link>
            <Link to="/notes">notes</Link>
            <Link to="/users">users</Link>
          </div>
          <Route exact path="/" render={() => <Home />} />
          <Route path="/notes" render={() => <Notes />} />
          <Route path="/users" render={() => <Users />} />
        </div>
      </Router>
    </div>
  )
}
```

Here, we specify the routing of our application. Here, routing is just conditional rendering of components based on the URL that we are currently visiting to which the components are children of the `Router` component.

Notice that we are importing `BrowserRouter` as `Router`. This is because `Router` is a common low-level interface for other higher-level router components. Thus, we wouldn't use `Router`, but rather `BrowserRouter` (or some other higher-level router component).

`BrowserRouter` is a `Router` that uses the HTML5 history API (uses `pushState`, `replaceState`, and the `popstate` event) to keep our UI in sync (with regard to conditional rendering and such) with the URL.

`BrowserRouter` will also give us logic pertaining to the back and forward buttons on our browser, allowing our web app to behave somewhat like a traditional web app/page.

To make this all happen, we define a `Link` component for each route that we want to "visit":

```jsx
<Link to="/route-here">Clicking me will take you to /route-here</Link>
```

Here, we created a link to the `/route-here` address of our application. This component will render the text "Clicking me will take you to /route-here" upon which a click will change the URL to `/route-here`.

We don't want to just change the URL, we also want to conditionally render components depending on our "route". We can do this by using the `Route` component:

```jsx
<Route path="/route-here" render={() => <ExamplePage />}
```

Now, with this `Route` component, if our URL is set to `baseUrl/route-here`, then the `ExamplePage` component will be rendered.

This logic of providing a relative path/route and a component to render extends to our `/notes` and `/users` routes above. You'll notice something different about the `/` route, however:

```jsx
<Route exact path="/" render={() => <Home />} />
```

Here, we have to specify `exact` as a prop since this will tell React Router that the route must match `/` exactly for this component to be rendered; otherwise, anything following `/` will still evaluate to the route for `/`.

Something interesting to note is that React Router takes advantage of *dynamic routing*. Static routing (found in Express, Ember, Angular, and older versions of React Router) has routes declared before the app listens for any requests (declaring routes up front before the app renders).

React Router with its dynamic routing has routing taking place as the app renders. Routes do not lie within a configuration or convention running outside of the app. This gives us the advantage of dynamically changing routes depending on screen size, state, etc. For example, we might want to render a dashboard view instead of individual sections of the dashboard on larger screens. With this declarative method, on a larger screen, we could automatically redirect the user to the dashboard route and "remove" the routes relating to the smaller screen sizes. This would not be possible with static routing.

#### Transitioning

> From this point on, we'll be going back to our project with an Express backend since we will need a backend server for certain features related to authentication in the future. This is also contingent on having navigation in our application which is why we are making the changes now.

Our old project is in a horrendous state. Tons of logic can be found all in our `App` component where we have a ton of functional components cobbled together to which our app consists of. Let's try to refactor our application a tiny bit so that it is somewhat manageable (please note that this none of the following practices should be looked to as an example). Let's first create a `LoginContainer` component that contains all of the logic that we need surrounding login functionality:

**/Frontend/src/components/LoginContainer.js**

```jsx
import React, { useState } from 'react'
import loginService from '../services/login'
import noteService from '../services/notes'
import Togglable from './Togglable'
import LoginForm from './LoginForm'

const LoginContainer = ({ user, setUser, setNotificationMessage }) => {

  const [username, setUsername] = useState('')
  const [password, setPassword] = useState('')

  const handleLogin = async (event) => {
    event.preventDefault()
    try {
      const user = await loginService.login({
        username, password,
      })

      setUser(user)
      setUsername('')
      setPassword('')
      window.localStorage.setItem(
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

  const loginForm = () => {
    return (
      <Togglable buttonLabel='Login'>
        <LoginForm
          handleLogin={handleLogin}
          handleUsernameChange={({ target }) => setUsername(target.value)}
          handlePasswordChange={({ target }) => setPassword(target.value)}
          username={username}
          password={password}
        />
      </Togglable>
    )
  }

  return user === null ?
    loginForm() :
    <div>
      <p>Welcome, {user.name}</p> <button onClick={handleLogout}>Sign Out</button>
    </div>
}

export default LoginContainer
```

Note how `LoginContainer` takes in `user`, `setUser`, and `setNotificationMessage` in from its props. We did not move these functions and pieces of state into this component as multiple components rely on this state (lifting state up).

When we eventually render this component in `App`, we will pass the props as follows:

```jsx
<LoginContainer user={user} setUser={setUser} setNotificationMessage={setNotificationMessage} />
```

We'll delete the excess code from `App` and continue with moving note creation into its own component:

**/Frontend/src/components/NoteFormContainer.js**

```jsx
import React, { useState } from 'react'
import noteService from '../services/notes'
import Togglable from './Togglable'
import NoteForm from './NoteForm'

const NoteFormContainer = ({ notes, setNotes }) => {

  const [noteField, setNoteField] = useState(
    'Enter a new note here...'
  )

  const noteFormRef = React.createRef()

  const handleInputChange = (event) => {
    console.log('Input form changed', event.target.value)
    setNoteField(event.target.value)
  }

  const addNote = (event) => {
    event.preventDefault()
    noteFormRef.current.toggleVisibility()
    const newNoteObject = {
      content: noteField,
      date: new Date(),
      flagged: Math.random() > 0.5,
    }

    noteService.create(newNoteObject)
      .then(data => {
        setNotes(notes.concat(data))
        setNoteField('Enter a new note here...')
      })
  }

  return (
    <Togglable buttonLabel='Create Note' ref={noteFormRef}>
      <NoteForm
        addNote={addNote}
        noteField={noteField}
        handleInputChange={handleInputChange}
      />
    </Togglable>
  )
}

export default NoteFormContainer
```

As with `LoginContainer `, `NoteFormContainer` takes in state through its props since this state needs to be shared between components (`notes` piece of state). When we use this component, we would pass in the props as follows:

```jsx
<NoteFormContainer notes={notes} setNotes={setNotes} />
```

Finally, let's create a component for the displaying of notes and the toggling of `flagged` for particular notes:

**/Frontend/src/components/NoteContainer.js**

```jsx
import React, { useState } from 'react'
import noteService from '../services/notes'
import Note from './Note'

const NoteContainer = ({ notes, setNotes }) => {

  const [showAll, setShowAll] = useState(true)

  const toggleFlagged = id => {
    const referencedNote = notes.find(n => n.id === id)
    const updatedNote = { ...referencedNote, flagged: !referencedNote.flagged }

    noteService
      .update(id, updatedNote)
      .then(responseNote => {
        setNotes(notes.map(note => note.id === id ? updatedNote : note))
      })
  }

  const filteredNotes = showAll ? notes : notes.filter(note => note.flagged)

  const noteList = () => filteredNotes.map(note =>
    <Note
      key={note.id}
      note={note}
      toggleFlagged={() => toggleFlagged(note.id)}
    />
  )

  return (
          <>
            <div>
              <button onClick={() => setShowAll(!showAll)}>
                Show {showAll ? 'Flagged' : 'All' }
              </button>
            </div>

            <ul>
              {noteList()}
            </ul>
          </>
  )
}

export default NoteContainer
```

As with `NoteFormContainer `, `NoteContainer` takes in the `notes` piece of state since it needs to be shared between components. When we use this component, we would pass in the props as follows:

```jsx
<NoteContainer notes={notes} setNotes={setNotes} />
```

With all of this, our `App` component is dramatically simplified and only contains the shared state and effects for grabbing the user's information and loading the notes:

**/Frontend/src/App.js**

```jsx
import React, { useState, useEffect } from 'react'
import Notification from './components/Notification'
import Separator from './components/Separator'
import noteService from './services/notes'
import LoginContainer from './components/LoginContainer'
import NoteFormContainer from './components/NoteFormContainer'
import NoteContainer from './components/NoteContainer'
import {
  BrowserRouter as Router, Route, Link
} from 'react-router-dom'

const App = (props) => {
  const [notes, setNotes] = useState([])
  const [notificationMessage, setNotificationMessage] = useState()
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

  const linkStyle = { padding: 10 }

  return (
      {/*...*/}
  )
}

export default App
```

We can now go ahead and use these components to be rendered conditionally dependent on a route. We'll also go ahead and add links to change the routes:

**/Frontend/src/App.js**

```jsx
//...

  return (
    <div>
      <h1>Notes</h1>
      <Router>
        <div>
          <div>
            <Link style={linkStyle} to="/">Home</Link>
            <Link style={linkStyle} to="/notes">Notes</Link>
            <Link style={linkStyle} to="/login">Login</Link>
            {user === null ?
              null :
              <Link style={linkStyle} to="/create">Create</Link>}
          </div>
          <Notification message={notificationMessage} />
          <div>
            <Route exact path="/" render={() => <h2>Home</h2>} />
            <Route path="/login" render={() => <LoginContainer user={user} setUser={setUser} setNotificationMessage={setNotificationMessage} />} />
            <Route path="/notes" render={() => <NoteContainer notes={notes} setNotes={setNotes} />}/>
            <Route path="/create" render={() => <NoteFormContainer notes={notes} setNotes={setNotes} />}/>
          </div>
        </div>
      </Router>
      <Separator />
    </div>
  )

//...
```

Here, we establish three links that will always be displayed near the top of the page (links to `/`, `/login`, and `/notes`). We conditionally render the `/create` route since we don't want to display the page that allows a user to create a note when the user cannot do so. This outlines an advantage to the dynamic nature of React Router: if we wanted to apply the conditional logic to the `Route` component, the `/create` route would not "exist" when the user is not signed in as it is dynamically created only when the user is created:

```jsx
{user ? <Route path="/create" render={() => user ? <NoteFormContainer notes={notes} setNotes={setNotes} /> : <Redirect to="/login" />}/> : null}
```

We always display the `/login` link (even when the user is signed in) as the `LoginContainer` component also contains the button for logging out of the application.

All of this yields the following navigation:

![Initial](https://i.imgur.com/sp1rxta.png)



Logging in gives us access to the "Create" tab:

![Create Tab](https://i.imgur.com/9t7TgU4.png)



### URL Parameters

Now, how might we access specific resources based on the route? This is applicable to profiles (where we need to fetch a particular profile based on the route involving the name) and many other areas.

For our purposes, let's just say that we wanted to make our individual notes link to "pages" displaying the single note. Let's first expand our routes to handle cases that match `/notes/${id}` where the number following `/notes/` is the note that we want to be displayed:

**/Frontend/src/App.js**

```jsx
//...

  return (
	  <Router>
        <div>
          <div>
            <Link style={linkStyle} to="/">Home</Link>
            <Link style={linkStyle} to="/notes">Notes</Link>
            <Link style={linkStyle} to="/login">Login</Link>
            {user === null ?
              null :
              <Link style={linkStyle} to="/create">Create</Link>}
          </div>
          <Notification message={notificationMessage} />
          <div>
            <Route exact path="/" render={() => <h2>Home</h2>} />
            <Route path="/login" render={() => <LoginContainer user={user} setUser={setUser} setNotificationMessage={setNotificationMessage} />} />
            <Route exact path="/notes" render={() => <NoteContainer notes={notes} setNotes={setNotes} />}/>
            <Route path="/notes/:id" render={({ match }) => <Note notes={getNoteById(match.params.id)} setNotes={setNotes} />}/>
            <Route path="/create" render={() => <NoteFormContainer notes={notes} setNotes={setNotes} />}/>
          </div>
        </div>
      </Router>
  )
```

Notice how we had to specify `exact` for the `/notes` route since the start of the route would also apply to the `/notes/:id` route that we want to target.

Here, similar to how parameters are defined in Express, we specify parameters in the form of `:PARAMETER`. In our `/notes/:id` route, we make use of parameters as well as a new `getNoteById()` helper:

```jsx
<Route path="/notes/:id" render={({ match }) => <Note notes={getNoteById(match.params.id)} />}/>
```

You'll notice that we're able to grab our parameters from the `params` property of `match`. The `match` object simply contains information on how the `Route` *matched* the URL.

Our helper is simply defined as:

```jsx
const getNoteById = (id) => notes.find(note => note.id === id)
```

This helper gives us the information for the note with the id specified in our URL, allowing us to pass it in as a prop to the `Note` component.

Now, we can dive into our `Note` component and modify it so that the text content of the note links to the about route with the appropriate ID:

**/Frontend/src/components/Note.js**

```jsx
import React from 'react'
import { Link } from 'react-router-dom'

const Note = ({ note, toggleFlagged }) => {
  if (!note) {
    note = {
      flagged: false,
      content: "Note content not found"
    }
  }
  return (
    <li className="mainNote card">
      <input type="checkbox" alt="Toggle Flagged" checked={note.flagged} onChange={toggleFlagged} />
        <Link to={`/notes/${note.id}`}>
          {note.content}
        </Link>
    </li>
  )
}

export default Note
```

We also added a default note (since we don't have actual proper state management with Redux in this application) for an use case in our application. If we decided to visit a component's link directly, for example: `http://localhost:3000/notes/5dcb86f0e84c9f54b0e9f7b4`, we would encounter a `TypeError` relating to us accessing properties of `note` when `note` is `undefined`. `note` is `undefined` when we visit the URL directly, because we immediately attempt to render the `Note` component with a note fetched by `getNoteById`. The issue lies in that `getNoteById` depends on the `notes` piece of state which is empty when the application first attempts to render since our effect (`loadNotesEffect`) only loads our notes after the first render. Adding a default allows our component to successfully render to which our effect can run and replace the state, causing our component to re-render with the proper state.

This allows us to click on the text of a note:

![Text Link](https://i.imgur.com/3aLcVCc.png)

Upon clicking this link, we'll be taken to a page featuring just the one note:

![Note Page](https://i.imgur.com/zzIp2U2.png)

We won't be able to toggle the importance of a note since our `onChange` handler is defined in `NoteContainer` and is passed to `Note`s there.



#### withRouter() HOC

We can programmatically send users to different routes by changing the URL being "accessed". We can do this in our components by wrapping them with the `withRouter()` higher-order component which creates a component that has access to the `history` prop. The `history` prop allows us to change the URL of the browser which will cause the respective conditional render to change.

For example, if we wanted to direct the user to the home page after logging in, we could just make use of the `push()` method of the `history` object:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

import { withRouter } from 'react-router-dom' // Import the `withRouter` HOC

const LoginContainer = ({ history, user, setUser, setNotificationMessage }) => { // Destructure the history object
    
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
      window.localStorage.setItem(
        'noteAppUser', JSON.stringify(user)
      )
      noteService.setToken(user.token)
      history.push('/') // Navigates the user to the `/` route (home)
    } catch (exception) {
      console.log(exception)
      setNotificationMessage('Incorrect credentials: ' + exception)
      setTimeout(() => {
        setNotificationMessage(null)
      }, 3000)
    }
  }

//...

export default withRouter(LoginContainer) // Wrap our component with `withRouter`
```

Upon a successful call to `handleLogin()`, `history.push('/')` will change the URL to `localhost:3000/` which causes our "home screen" to be rendered.

#### <Redirect />

We can use the `Redirect` component to redirect the user declaratively (as we did imperatively above). For example, if we wanted to redirect the user to the login page if they weren't signed in and visited the `create` route, we could conditionally render the `Redirect` component if `user` didn't exist in the `Route`'s `render` property:

```jsx
<Route path="/create" render={() => user ? <NoteFormContainer notes={notes} setNotes={setNotes} /> : <Redirect to="/login" />}/>
```

If we are not signed in and attempt to visit `/create`, we will be taken to `/login`.

Looking at the `App` component, you can see the various components that we had used to accomplish our navigation:

**/Frontend/src/App.js**

```jsx
import React, { useState, useEffect } from 'react'
import Notification from './components/Notification'
import Separator from './components/Separator'
import noteService from './services/notes'
import LoginContainer from './components/LoginContainer'
import NoteFormContainer from './components/NoteFormContainer'
import Note from './components/Note'
import NoteContainer from './components/NoteContainer'
import {
  BrowserRouter as Router, Route, Link, Redirect
} from 'react-router-dom'

const App = (props) => {
  const [notes, setNotes] = useState([])
  const [notificationMessage, setNotificationMessage] = useState()
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

  const getNoteById = (id) => notes.find(note => note.id === id)

  const linkStyle = { padding: 10 }

  return (
    <div>
      <h1>Notes</h1>
      <Router>
        <div>
          <div>
            <Link style={linkStyle} to="/">Home</Link>
            <Link style={linkStyle} to="/notes">Notes</Link>
            <Link style={linkStyle} to="/login">Login</Link>
            {user === null ?
              null :
              <Link style={linkStyle} to="/create">Create</Link>}
          </div>
          <Notification message={notificationMessage} />
          <div>
            <Route exact path="/" render={() => <h2>Home</h2>} />
            <Route path="/login" render={() => <LoginContainer user={user} setUser={setUser} setNotificationMessage={setNotificationMessage} />} />
            <Route exact path="/notes" render={() => <NoteContainer notes={notes} setNotes={setNotes} />}/>
            <Route path="/notes/:id" render={({ match }) => <Note note={getNoteById(match.params.id)} />}/>
            <Route path="/create" render={() => user ? <NoteFormContainer notes={notes} setNotes={setNotes} /> : <Redirect to="/login" />}/>
          </div>
        </div>
      </Router>
      <Separator />
    </div>
  )
}

export default App
```

You'll also notice that the "Notes" header is rendered regardless of our route. This is because it is found outside of the `Router` and any `Route`s and will not be conditionally rendered.

There are plenty of features in React Router, and when you get into more complex applications, these features may come into much use. It is recommended to read up on the latest React Router documentation on *reacttraining.com*.

## Passport.js and OAuth

Previously, we talked about using JSON Web Tokens (JWT) for authentication and implemented it manually in our application using `bcrypt` and `jsonwebtoken`. We'll now transition into using Passport.js which gives us the ability to integrate with login strategies for Facebook, Google, LinkedIn, Twitch, Twitter, and many more. 

If you just need username/password-like authentication, it is recommended to just implement the feature manually, but if you need to take advantage of OAuth, OpenID, etc. you'll usually need to take advantage of Passport.js.

Recall that we had implemented token-based authentication. As a refresher and to draw another difference, the three main options for authentication are:

* **Session-Based Authentication:** Associates a "session" with the client through storing the user state in the server's memory. When the user logs in, the session ID is stored in a cookie in the user's browser. This ID is used to identify the user and is used to be compared to the stored session data in the server's memory in order to authenticate.
* **Cookie-Based Authentication:** Identifier for the user is stored in a cookie (usually a token) and is used to automatically identify API requests. This is easier to deal with, but it comes with some security flaws as cookies are automatically included in all suitable requests to the host.
* **Token-Based Authentication:** Like cookie-based authentication (the definition that we explored earlier in this book), but you must manually include it with requests (in the `Authorization` header).

CodeRealm (https://github.com/alex996) provides a very good presentation highlighting the differences in authentication methods (skip to #Passport Modules if you wish to skip to right with working with token-based authentication):

* **Session Flow**: user submits email/password (credentials) -> server verifies credentials from the data it has stored in the database -> server creates a temporary user **session** -> server takes the session's ID and sends it back -> user sends the cookie with the ID stored with each request -> server validates the cookie against the session store and grants access if correct -> user logs out and the server destroys the session.
  * Stateful (sessions stored on the server whether it's a database or in memory)
*  **Token Flow**: user submits email/password (credentials) -> server verifies credentials from the data it has stored in the database -> server creates a temporary **token** and embeds the user's data in it -> server sends the token back -> client stores the token -> client sends the token along with requests -> server validates the token and grants access if correct -> user logs out and the client deletes the token
  * **Stateless JWT** (JWT token contains the session data encoded right in the token)
  * **Stateful JWT** (JWT token just contains a reference to the session and the session data is still stored server-side)

### Passport Modules

Passport.js (referred to as Passport from this point on) has a ton of extendable modules that we can use for plenty of login strategies and authentication requirements. These strategies range from creating our own local strategies from scratch to depending on third parties to do everything for us.

In this project, we're just going to be working with the `passport-google-oauth`, `passport-facebook`, `passport-local` and `passport-jwt` modules which will allow us to add these social logins as well as JWT token-based authentication.

As a general overview of what happens, both `passport-google-oauth` and `passport-facebook` provide endpoints that authenticate against Google or Facebook (dependent on the module) to which a JWT will be generated that we can use for other functions in our application. `passport-local` allows us to authenticate users via username and password.

Let's go ahead and install all of these packages by running:

```bash
npm install passport passport-google-oauth passport-facebook passport-jwt passport-local
```

### Facebook and Google API Credentials

#### Creating Google Credentials

In order to use OAuth with Facebook and Google, we'll need to get some API credentials so that we can use their authentication services.

Please note that these directions can change at any time due to changes with their platforms.

For Google, we need to visit https://console.developers.google.com/apis/dashboard and then login.

Then, we'll press "Select a Project", then "New Project":

![Project Creation](https://i.imgur.com/FJwecjF.png)



We'll be prompted to enter a project name and organization. We can just proceed by pressing "Create" for now. We can then select our new project from the "Select a project" dropdown.

Next, we'll need to visit the "OAuth consent screen" tab on the sidebar and select "External":

![External](https://i.imgur.com/2CX7Mti.png)

On the next screen, we can enter in whatever information we would like for the application name and logo, but, for development:

![Test Name](https://i.imgur.com/vJ12WOF.png)



After saving these changes, we can head to the "Credentials" tab, press "Create Credentials", and then press "OAuth client ID":

![](https://i.imgur.com/IjMfgtn.png)

We'll then specify the "Web application" application type, give our project a name, and fill in the redirect URLs with the endpoints that our Express server will handle as well as the authorized origins:

![](https://i.imgur.com/TdXXWOx.png)

In this case, we want to be redirected to the endpoint (`/api/authentication/google/redirect) of our Express server running on `localhost` (while we're testing) so that we can handle the rest of the authentication process. We also authorize requests coming from our Express server.

Upon pressing create, we are given our "Client ID" and "Client Secret" credentials that we need for the Google Passport strategy:

![](https://i.imgur.com/uSBTgjR.png)

For now, we'll just hold onto these credentials while we get our Facebook credentials.

#### Creating Facebook Credentials

Let's visit the "Facebook for Developers" portal (https://developers.facebook.com/).

Here, we'll login with a Facebook account and press "My Apps" followed by "Create App": C

![](https://i.imgur.com/wEEogkW.png)

Here, we can give our app any name that we wish as well as provide a contact email:

![](https://i.imgur.com/BOhbOpW.png)

We'll then be taken to the "Add a Product" screen where we'll press "Set Up" on the card related to "Facebook Login":

![](https://i.imgur.com/tvOqEKh.png)

Next, we'll select "Web" as our target platform:

![](https://i.imgur.com/HFRjnMX.png)

Here, we'll enter in a "Site URL" of `http://localhost:3000`:

![](https://i.imgur.com/7Dazd3t.png)

We'll then press "Save" and navigate to the "Basic" option under "Settings" on the sidebar:

![](https://i.imgur.com/EzVDcn1.png)

Here, we need to provide a valid URL for the "Privacy Policy URL" field:

![](https://i.imgur.com/5rerP2U.png)

Here, we used `http://github.com/help` as Facebook will only accept URLs with a valid SSL certificate. Let's press "Save Changes" to update this field. We'll also copy down the `App ID` and `App Secret` values on this page:

![](https://i.imgur.com/hPzlrCY.png)

Now, we can navigate to the "Settings" option under "Facebook Login" on the products sidebar:

![](https://i.imgur.com/Lc2mODM.png)

We'll then enter `http://localhost:3000/api/authentication/facebook/redirect` in the "Valid OAuth Redirect URIs" box:

![](https://i.imgur.com/QBORn2V.png)

We'll then press "Save Changes". We are now done with setting up our Facebook credentials!

The temporary privacy policy that we had added earlier should serve only for development purposes, but its presence will allow to make our Facebook application "go live". We won't do this for now, however, as our application is still under development.

### Implementing Passport

We can now dive into adding Passport into our application. Let's first start by adding the ability to fetch the credentials that we just created. Remember, for anything that might be sensitive or private, we would want to store these items in our environment variables or `.env` file, to which we'll access from a configuration JavaScript module.

Let's add the following lines to our `.env` file:

```env
GOOGLE_CLIENTID=884456547996-2svdip83utcr1ndmgua0flcqqu3u9f90.apps.googleusercontent.com
GOOGLE_CLIENTSECRET=-XuxIaDF-GVIC_zFgFrrGpSv
FACEBOOK_CLIENTID=520139281950065
FACEBOOK_CLIENTSECRET=31abe32d718023622a3c8f82dfa06a49
```

We'll then make the following changes to our config module:

**/Backend/utils/config.js**

```jsx
require('dotenv').config()
const logger = require('./logger')

let isRequired = (name, ...values) => {
  values.forEach(value => {
    if (value == null || value === '') { // Takes advantage of coercing to check if the value is undefined or null or blank
      logger.error(`${name} is missing a value!`)
    }
  })
}

// Express Configuration
let PORT = process.env.PORT
isRequired('Express Configuration', PORT)

// Database Configuration
let MONGODB_URI = process.env.MONGODB_URI
isRequired('MongoDB Configuration', MONGODB_URI)

// Google API Configuration
let GOOGLE_CLIENTID = process.env.GOOGLE_CLIENTID
let GOOGLE_CLIENTSECRET = process.env.GOOGLE_CLIENTSECRET
isRequired('Google API Configuration', GOOGLE_CLIENTID, GOOGLE_CLIENTSECRET)

// Facebook API Configuration
let FACEBOOK_CLIENTID = process.env.FACEBOOK_CLIENTID
let FACEBOOK_CLIENTSECRET = process.env.FACEBOOK_CLIENTSECRET
isRequired('Facebook API Configuration', FACEBOOK_CLIENTID, FACEBOOK_CLIENTSECRET)


if (process.env.NODE_ENV === 'test') {
  MONGODB_URI = process.env.TEST_MONGODB_URI
}

module.exports = {
  MONGODB_URI,
  PORT,
  GOOGLE_CLIENTID,
  GOOGLE_CLIENTSECRET,
  FACEBOOK_CLIENTID,
  FACEBOOK_CLIENTSECRET
}
```

Here, we simply added the ability to fetch our API credentials from the `config` module. We have also added a `isRequired()` helper function that will tell us if any of our sections are missing anything configuration wise (by checking if the value is `undefined`, `null` or blank `""`).

In larger projects, it might be wise to create multiple configuration files with each configuration file related to a certain function. Instead of manually doing validation, it might be helpful to use something like `node-convict`, a configuration management library for Node, that will allow us to perform validation, provide defaults, easily create nested structures, etc.

Let's install `node-convict` by running the following command:

```bash
npm install convict
```

We'll now update our `config` module as follows:

**/Backend/utils/config.js**

```jsx
require('dotenv').config()
const convict = require('convict')

const config = convict({
  port: {
    doc: 'Port that Express listens on',
    format: 'port',
    default: 3000,
    env: 'PORT'
  },
  database: {
    doc: 'MongoDB URI for database connection',
    format: String,
    default: '',
    env: 'MONGODB_URI'
  },
  authentication: {
    google: {
      clientId: {
        doc: 'Google Auth Client ID',
        default: '',
        env: 'GOOGLE_CLIENTID'
      },
      clientSecret: {
        doc: 'Google Auth Client Secret',
        default: '',
        env: 'GOOGLE_CLIENTSECRET'
      }
    },
    facebook: {
      clientId: {
        doc: 'Facebook Auth client ID',
        default: '',
        env: 'FACEBOOK_CLIENTID'
      },
      clientSecret: {
        doc: 'Facebook Auth client secret',
        default: '',
        env: 'FACEBOOK_CLIENTSECRET'
      }
    },
    token: {
      secret: {
        doc: 'Secret key for JWT (signer)',
        default: 'jDGWbgiUDHGIWOGE&G8DUGVHUI',
        env: 'SECRET'
      },
      issuer: {
        doc: 'JWT Issuer',
        default: 'Test App'
      },
      audience: {
        doc: 'JWT Audience',
        default: 'Test App'
      }
    }
  }
})

// Perform validation
config.validate({ allowed: 'strict' })

module.exports = config.getProperties()
```

The above represents a new way of defining our configuration variables. You'll see how we are able to provide descriptions and validation for our environment variables with the above syntax. We're also able to nest configuration within "objects" (for example, our actual credentials are nested within `facebook` and `google` properties of the `authentication` object).

We'll also update all relevant references to our config file (for example `config.MONGODB_URI` to `config.database`). Additionally, we'll change `PORT` in our **.env** file to `3000` since that is what our authentication providers are expecting. React will automatically adjust its port when it detects that `3000` is occupied. Although React will adjust its own port automatically, we will have to manually adjust its proxy to send relative requests to `http://localhost:3000` instead of `http://localhost:3001`:

**/Frontend/package.json**

```json
//...

{
    //...
    "proxy": "http://localhost:3000"
}

//...
```

#### Passport Initialization

We can now ago ahead and provide the initial setup for Passport. This setup merely involves setting up the middleware so that we can actually use it.

To do this, we'll just require the `passport` module in our `app` module to which we'll bind the middleware with `app.use()` upon `passport.initialize()`:

**/Backend/app.js**

```js
//...

const passport = require('passport')

//...

app.use(passport.initialize())

//...
```

The Passport documentation instructs us to add the `passport.session()` middleware, but since we have no need for persistent login sessions (as we're using stateless JWT), we can skip its binding.

### JWT Token Authentication Utility

We can now go ahead and set up JWT token authentication with Passport. For this strategy, we'll be using the `passport-jwt` module which allows us to look for an "Authorization" header with the JWT inside it and will decode the JWT so that we can manipulate the payload of the token however we wish. This module will also reject the request for us automatically if the token is invalid.

For the sake of organization, we'll be putting the configuration for the strategy in a `jwt` module within a new directory, **authentication**, found in the **utils** folder:

**/Backend/utils/authentication/jwt.js**

```jsx
const passport = require('passport')
const passportJwt = require('passport-jwt')
const config = require('../config')
const User = require('../../models/user')

const jwtOptions = {
  // Reads JWT from the HTTP Authorization header with the 'bearer' scheme
  jwtFromRequest: passportJwt.ExtractJwt.fromAuthHeaderWithScheme('Bearer'),
  // Secret used to sign the JWT
  secretOrKey: config.authentication.token.secret,
  // Issuer stored in JWT
  issuer: config.authentication.token.issuer,
  // Audience stored in JWT
  audience: config.authentication.token.audience
}


passport.use(new passportJwt.Strategy(jwtOptions, async (payload, done) => {
  // In production, you should check to see if the token has not expired here
  console.log(payload.sub)
  const user = await User.findById(payload.sub)
  console.log(user)
  if (user) {
    return done(null, user, payload)
  }
  return done(null, false, { message: 'User not found.' })
}))
```

Here, we simply configure the JWT decoder with our knowledge of the secret, issuer, and audience. We also configure the JWT decoder to get the JWT from the Authorization header as a bearer token. If the issuer or audience does not match with what we have in our configuration, the authentication will fail.

When our token is decoded by `passport-jwt`, the token data is passed as a `payload` to the callback that we specify as the second argument to `passportJwt.strategy()`.

We then use the information from the token via the `payload` object to find the associated user from which the request was sent. If the user is found, we provide it to Passport who will then make it available to any following middleware/controllers under the `user` property of `req` (`req.user`).

In a production app, we should add the ability to revoke tokens and check if a token is revoked before granting access. This would require a JWT ID (explained below) to be issued.

### JWT Token Generation Utility

Now that we had implemented the ability to decode our JWTs, let's create a dedicated function for generating an authentication token for a given user (which will decode to reveal the user ID in its `subject` to which we can use the user ID to fetch the user for more information).

**/Backend/utils/authentication/token.js**

```jsx
const jwt = require('jsonwebtoken')
const config = require('../config')

// Generate an Access Token for the given User ID
function generateAccessJWT(userId) {
  // Time until token expires
  const expiresIn = '1 hour'
  // Issuer of token
  const issuer = config.authentication.token.issuer
  // Intended audience/service for token
  const audience = config.authentication.token.audience
  // Signing key for token
  const secret = config.authentication.token.secret

  const token = jwt.sign({}, secret, {
    expiresIn: expiresIn,
    audience: audience,
    issuer: issuer,
    subject: userId.toString()
  })

  return token
}

module.exports = {
  generateAccessJWT
}
```

Here, `generateAccessJWT()` takes in a user's ID and creates a JWT that will allow them to authenticate as that user. Notice how this token does not contain any other information in its payload where we previously were adding the user's username and id (instead we take the ID from the subject instead of other payload properties):

```jsx
const userForToken = {
    username: user.username,
    id: user._id,
}

const token = jwt.sign(userForToken, process.env.SECRET)
```

We set the `subject` option that `jwt.sign()` takes in its third parameter to the userID so that we can simply check the token's payload for the `sub` property (subject is stored within `sub` in the payload of a token) for our user's ID.

Within a token's payload, there are numerous *registered claims* which provide a set of useful, interoperable claims such as `iss` (issuer), `exp` (expiration time), `sub` (subject), and `aud` (audience).

See "Payload" of https://jwt.io/introduction/ for more information.

With our above generation of a token, we do not implement the generation or specification of a JWT ID. This means that our tokens do not have a unique identifier to which we could use to check if the token had been revoked. Expiration of the token will still work as it depends on time, but if we programmatically or manually revoke a token, we must have specified a JWT ID so that we could create a store of revoked IDs. We would then check the tokens that come with requests to see if they are among the tokens in the store. JWT IDs usually come in the form of UUIDs to which we can randomly assign the IDs without worrying about collisions (since the probability is so low, think one in one trillion trillion trillion trillion).

### Social Login Provider Strategies

With the ability to generate and decode tokens completed, we now just need to create controllers that will actually handle the initiation of our authentication process.

When using social login providers, the user is redirected to a social login provider to which, upon success, they are redirected back to us where we can complete the process.

#### Google Authentication

Let's create a `google` module that will configure Passport with `passport-google-oauth` and add a new strategy to which we create a new user (if one doesn't already exist):

**/Backend/utils/authentication/google.js**

```jsx
const passport = require('passport')
const passportGoogle = require('passport-google-oauth')
const config = require('../config')
const User = require('../../models/user')

const passportConfig = {
  clientID: config.authentication.google.clientId,
  clientSecret: config.authentication.google.clientSecret,
  callbackURL: 'http://localhost:3000/api/authentication/google/redirect'
}

if (passportConfig.clientID) {
  passport.use(new passportGoogle.OAuth2Strategy(passportConfig, function (request, accessToken, refreshToken, profile, done) {
    // Creates user if this user does not already have an account with this googleId
    User.findOrCreate({ name: profile.name.givenName, providerId: profile.id }, function (err, user) {
      return done(err, user)
    })
  }))
}
```

Here, we specify the behavior of the redirect request where we create a user with the `googleId` and `name` set to information provided to us by Google's systems. We make use of what appears to be a mongoose method `findOrCreate()`, but it does not actually exist. We can add it by installing the npm package `mongoose-findorcreate` and then registering the plugin on our `userSchema` in our user model. Let's install the package by running:

```bash
npm install mongoose-findorcreate
```

Then, we'll make the following changes to our model:

**/Backend/models/user.js**

```jsx
//...

const findOrCreate = require('mongoose-findorcreate')

const userSchema = new mongoose.Schema
	username: String // Removed the `unique` property as usernames are no longer required
    
	//...
    
})

//...

userSchema.plugin(findOrCreate)

//...
```

This mechanism registers the user upon first login, but does not create duplicate users with our check to see if there already is a user that was created with this Google account.

We also removed the requirement that `username` be unique since we no longer depend on the username for identifying/fetching a user (instead we use `id`). This will temporarily break username/password authentication and registration, but we'll get around to fixing it.

**We have to drop the users collection in mongoDB as it was initially expecting a unique value for the `username` value** (we could use `dropIndexes` if we wished to preserve the collection).

Now, we just have to set up route handlers to manage our Google authentication requests. We'll place these in an `auth` module found in our controllers directory:

**/Backend/controllers/auth.js**

```jsx
const authRouter = require('express').Router()
const passport = require('passport')
const token = require('../utils/authentication/token')
require('../utils/authentication/jwt')
require('../utils/authentication/google')
require('../utils/authentication/facebook')

function generateUserJWTAndRedirect(req, res) {
  const accessJWT = token.generateAccessJWT(req.user.id)
  res
    .status(200)
    .redirect(`http://localhost:3001/login?token=${accessJWT}`)
}

// Google Routes
authRouter.get('/google/start',
  passport.authenticate('google', { session: false, scope: ['openid', 'profile', 'email'] }))
authRouter.get('/google/redirect',
  passport.authenticate('google', { session: false }),
  generateUserJWTAndRedirect)

module.exports = authRouter
```

Here, we chain our controllers/middleware.

We'll need to bind our router to our Express application in **App.js** as follows:

**/Backend/App.js**

```js
//...

const notesRouter = require('./controllers/notes')
const usersRouter = require('./controllers/users')
const loginRouter = require('./controllers/login')
const authRouter = require('./controllers/auth')

//...

// Routers
app.use('/api/authentication', authRouter)
app.use('/api/notes', notesRouter)
app.use('/api/users', usersRouter)
app.use('/api/login', loginRouter)

//...
```

#### Frontend Changes

Now, let's delve into the frontend and add a simple "Sign In with Google" button that should trigger this entire authentication flow using Google OAuth:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

  const authenticateWithProvider = (provider) => {
    window.location = ('http://localhost:3000/api/authentication/' + provider + '/start')
  }

  const loginForm = () => {
    return (
      <>
        <Togglable buttonLabel='Sign Up With Email'>
          <LoginForm
            handleLogin={handleLogin}
            handleUsernameChange={({ target }) => setUsername(target.value)}
            handlePasswordChange={({ target }) => setPassword(target.value)}
            username={username}
            password={password}
          />
        </Togglable>
        <button onClick={() => authenticateWithProvider('google')}>Sign In with Google</button>
      </>
    )
  }
  
//...
```

Here, we simply create an event handler that opens a new window that loads the start URL of whatever provider we are using. While user authentication and creation is successful at this moment. We need to do a few more things to make our frontend work with our new authentication system. Right now, our browser is redirected to the OAuth provider, to which the OAuth provider redirects us to a callback URL on our backend server. In the controller for this URL, we take the information given to us by Google and generate a token that the user can use to authenticate. We then take this token and redirect the user to the address of the frontend with the token embedded in it as a URL parameter.

This means that instead of getting the token through an `axios` request like the following:

```jsx
const user = await loginService.login({
  username, password
}) // Gives us username, name, and token in the user object
```

We have to fetch it from the URL:

```jsx
const params = (new URL(document.location)).searchParams
const token = params.get('token')
```

Our backend route will redirect us to the login route of our frontend with the token as a URL parameter to which we need to access and set the appropriate tokens in our modules/add it to local storage. This presents a perfect use case for `useEffect()` as we need something to occur after the component has rendered as a side effect:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

import authenticatedState from '../helpers/authenticatedState'

const LoginContainer = () => {
    
  //...
    
  const tokenEffect = () => {
  const params = (new URL(document.location)).searchParams
  const token = params.get('token')
  if (token) {
    setUser(authenticatedState(token))
  }

    // For production cases, you should remove the parameters from the URL so the user cannot accidentally copy and send their token when trying to share their link
  }

  useEffect(tokenEffect, [])
}
```

Here, our effect simply takes the token (if there is one) from the URL upon initial render of the page. We also set our user to the return value of `authenticatedState(token)`:

Since we are no longer receiving our `token`, `username`, and `name` all in one response when we login with passport (unlike logging in through our `/login` route), we must take this token that we receive and use it to fetch our user information. With this, we created our first *helper* in our frontend that will go ahead and do everything for us regarding setting the tokens for our services, getting our `user` object, setting `localStorage`, setting state state, etc.

**/Frontend/src/helpers/authenticatedState.js**

```jsx
import noteService from '../services/notes'
import loginService from '../services/login'

const authenticatedState = async (token) => {
  noteService.setToken(token)
  loginService.setToken(token)
  let user = await loginService.profile()
  user = {
    token,
    ...user
  }
  window.localStorage.setItem(
    'noteAppUser', JSON.stringify(user)
  )
  return user
}

export default authenticatedState
```

Here, you'll see that we take advantage of a method of `loginService`, `profile()` to which we'll add as the following:

**/Frontend/src/services/login.js**

```jsx
import axios from 'axios'
const baseUrl = '/api/login'

let token = null

const setToken = userToken => {
  token = `bearer ${userToken}`
}

const login = async credentials => {
  const res = await axios.post(baseUrl, credentials)
  return res.data
}

const profile = async () => {
  const config = {
    headers: { Authorization: token },
  }

  const res = await axios.get(`${baseUrl}/profile`, config)
  return res.data
}

export default { setToken, login, profile }
```

This route does not yet exist in our backend, so we'll add the following to our `login` controller:

**/Backend/controllers/login.js**

```js
//...

const passport = require('passport')
require('../utils/authentication/jwt')

//...

loginRouter.get('/profile',
  passport.authenticate('jwt', { session: false }),
  async (req, res) => {
    const user = req.user
    const userForProfile = {
      username: user.username,
      id: user._id
    }
    res.json(userForProfile)
  }
)

module.exports = loginRouter
```

Here, since the goal is to simply provide an endpoint that returns the information that our frontend needs regarding the user, we can just return a JSON object containing the user information. Passport, upon authentication, makes the user object representing the user that is authenticated available under `req.user`. **We can "authenticate"/protect our controller by chaining the `passport.authenticate()` middleware before our actual controller.** This will ensure that the request is authenticated before we run any logic and will allow us to access `req.user`.

We'll also replace the manual logic for verifying a JWT with a simple call to `passport.authenticate()` before the controller for creating a note:

**/Backend/controllers/notes.js**

```jsx
//...

const passport = require('passport')
require('../utils/authentication/jwt')

//...

notesRouter.post('/',
  passport.authenticate('jwt', { session: false }),
  async (req, res, next) => {
    const body = req.body

    try {
      const user = await User.findById(req.user.id)

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
  }
)

//...
```

We can also make a few updates to our `App` component so that we set the token for our login service as well:

**/Frontend/src/App.js**

```jsx
//...

  const signedInEffect = () => {
    const loggedInUserJSON = window.localStorage.getItem('noteAppUser')
    if (loggedInUserJSON) {
      const storageUser = JSON.parse(loggedInUserJSON)
      console.log('user')
      setUser(storageUser)
      console.log(storageUser)
      noteService.setToken(storageUser.token)
      loginService.setToken(storageUser.token)
    }
  }

//...
```

With this, our app should work as it did before (except for username and password authentication which we'll add in a moment). We could make use of our `authenticatedState` helper here, but we don't need to re-request for our profile information as `authenticatedState` does since we have no way of changing this information currently. Additionally, with this project lacking Redux, the organization of our project will not be optimal.

#### Facebook Authentication

We can now easily add Facebook authentication to our app by creating a `facebook` auth module (that is almost exactly like our `google` auth module), the appropriate controllers (again, very similar to the Google setup), and a button in our frontend that makes the initial request.

Let's create the `facebook` strategy as follows:

**/Backend/utils/authentication/facebook.js**

```js
const passport = require('passport')
const passportFacebook = require('passport-facebook')
const config = require('../config')
const User = require('../../models/user')

const passportConfig = {
  clientID: config.authentication.facebook.clientId,
  clientSecret: config.authentication.facebook.clientSecret,
  callbackURL: 'http://localhost:3000/api/authentication/facebook/redirect'
}

if (passportConfig.clientID) {
  passport.use(new passportFacebook.Strategy(passportConfig, function (request, accessToken, refreshToken, profile, done) {
    // Creates user if this user does not already have an account with this facebookId
    User.findOrCreate({ name: profile.displayName, providerId: profile.id }, function (err, user) {
      return done(err, user)
    })
  }))
}
```

We can then add the following routes to our `auth` controller:

**/Backend/controllers/auth.js**

```js
const authRouter = require('express').Router()
const passport = require('passport')
const token = require('../utils/authentication/token')
require('../utils/authentication/jwt')
require('../utils/authentication/google')
require('../utils/authentication/facebook')

function generateUserJWTAndRedirect(req, res) {
  const accessJWT = token.generateAccessJWT(req.user.id)
  res
    .status(200)
    .redirect(`http://localhost:3001/login?token=${accessJWT}`)
}

// Google Routes
authRouter.get('/google/start',
  passport.authenticate('google', { session: false, scope: ['openid', 'profile', 'email'] }))
authRouter.get('/google/redirect',
  passport.authenticate('google', { session: false }),
  generateUserJWTAndRedirect)

// Facebook Routes
authRouter.get('/facebook/start',
  passport.authenticate('facebook', { session: false }))
authRouter.get('/facebook/redirect',
  passport.authenticate('facebook', { session: false }),
  generateUserJWTAndRedirect)

module.exports = authRouter
```

Finally, let's just add a button in the frontend that makes the first request for Facebook authentication:

```jsx
//...

  const loginForm = () => {
    return (
      <>
        <Togglable buttonLabel='Sign Up With Email'>
          <LoginForm
            handleLogin={handleLogin}
            handleUsernameChange={({ target }) => setUsername(target.value)}
            handlePasswordChange={({ target }) => setPassword(target.value)}
            username={username}
            password={password}
          />
        </Togglable>
        <button onClick={() => authenticateWithProvider('google')}>Sign In with Google</button>
        <button onClick={() => authenticateWithProvider('facebook')}>Sign In with Facebook</button>
      </>
    )
  }
  
//...
```

### Fixing Username/Password Authentication

We can fix username/password authentication with the help of `passport-local`. We'll create a new authentication configuration module as follows:

**/Backend/utils/authentication/local.js**

```jsx
const passport = require('passport')
const LocalStrategy = require('passport-local').Strategy
const User = require('../../models/user')
const bcrypt = require('bcrypt')

passport.use(new LocalStrategy(
  function(username, password, done) {

    // Find user with matching username
    User.findOne({ username: username }, async function(err, user) {
      if (err) { return done(err) }
      if (!user) {
        return done(null, false, { message: 'Username not found.' })
      }

      // Checks to see if hashed password matches hash of provided password
      const passwordCorrect = user === null
        ? false
        : await bcrypt.compare(password, user.passwordHash)

      if (!passwordCorrect) {
        return done(null, false, { message: 'Incorrect password.' })
      }
      return done(null, user)
    })
  }
))
```

Here, you can see how we're pretty much left to ourselves in terms of describing what should authenticate and what shouldn't. We even describe how to check passwords. Passport does not implement signup methods as it is just an authorization library, so we are given the freedom with regard to how we create users as well. For now, our previous logic found in the `users` controller will work.

Now, we just have to make use of our new `LocalStrategy` in our `auth` controller that will authenticate us and then generate a token that we can use for further authentication around our apps:

```jsx
const authRouter = require('express').Router()
const passport = require('passport')
const token = require('../utils/authentication/token')
require('../utils/authentication/jwt')
require('../utils/authentication/google')
require('../utils/authentication/facebook')
require('../utils/authentication/local') // Local strategy import

function generateUserJWTAndRedirect(req, res) {
  const accessJWT = token.generateAccessJWT(req.user.id)
  res
    .status(200)
    .redirect(`http://localhost:3001/login?token=${accessJWT}`)
}

// Google Routes
authRouter.get('/google/start',
  passport.authenticate('google', { session: false, scope: ['openid', 'profile', 'email'] }))
authRouter.get('/google/redirect',
  passport.authenticate('google', { session: false }),
  generateUserJWTAndRedirect)

// Facebook Routes
authRouter.get('/facebook/start',
  passport.authenticate('facebook', { session: false }))
authRouter.get('/facebook/redirect',
  passport.authenticate('facebook', { session: false }),
  generateUserJWTAndRedirect)

// Local Authentication Routes
authRouter.get('/local/start',
  passport.authenticate('local', { session: false }),
  generateUserJWTAndRedirect)

module.exports = authRouter
```

Here, we don't need a redirect route that we then authenticate with as we can simply authenticate upon our initial request (as we have all the information that we need), generate a token, and redirect the user.

Finally, we just need to change our login form to actually submit a request to the `/api/authentication/local/start` route:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

  const loginForm = () => {
    return (
      <>
        <Togglable buttonLabel='Sign Up With Email'>
          <LoginForm
            loginAddress={'http://localhost:3000/api/authentication/local/start'}
            handleUsernameChange={({ target }) => setUsername(target.value)}
            handlePasswordChange={({ target }) => setPassword(target.value)}
            username={username}
            password={password}
          />
        </Togglable>
        <button onClick={() => authenticateWithProvider('google')}>Sign In with Google</button>
        <button onClick={() => authenticateWithProvider('facebook')}>Sign In with Facebook</button>
      </>
    )
  }

//...
```

Here, we pass in a new prop `loginAddress` which replaced our `handleLogin` handler. Additionally, we removed the extraneous imports of our application.

Now, we can just update the JSX for our login form to make a POST request to the `loginAddress` which triggers the local authentication process:

**/Frontend/src/components/LoginForm.js**

```jsx
import React from 'react'
import PropTypes from 'proptypes'

const LoginForm = ({
  loginAddress,
  handleUsernameChange,
  handlePasswordChange,
  username,
  password
}) => {

  return (
    <form action={loginAddress} method={'post'}>
      <div>
            Username
        <input
          type="text"
          value={username}
          name="username"
          onChange={handleUsernameChange}
        />
      </div>
      <div>
            Password
        <input
          type="password"
          value={password}
          name="password"
          onChange={handlePasswordChange}
        />
      </div>
      <button type="submit">Login</button>
    </form>
  )
}

LoginForm.propTypes = {
  loginAddress: PropTypes.string.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired
}

export default LoginForm
```

We also changed the names of our input fields to lowercase as that is what Passport is expecting.

Our app will now redirect instead of simply remaining on the application page and making HTTP requests. We could implement a solution that allows us to read information from a request or open another window for the sign in process to which we send the information back to our main window, but we'd like to keep the logic surrounding all three authentication methods as close as possible for now.