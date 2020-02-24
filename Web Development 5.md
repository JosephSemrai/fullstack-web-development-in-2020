## Composition and React `props.children`

Although we have been writing in a mostly functional side, React resembles an object oriented nature with many parallels such as allowing the defining of objects that contain and directly manipulate data without defining how they are used. Components are logically separated and can be used as building blocks for bigger purposes.

This brings us to two ideas: composition and inheritance. In basic terms, composition is when an object/class utilizes another object to provide some or all of its functionality. Problems are broken down into modular solutions to which components build on top of each other to provide the ultimate solution. Inheritance involves creating a basic structure to which other subclasses inherit the implementation and augment/reuse this basic structure.

With this, React has a powerful composition model and there are almost no cases where one would benefit from using inheritance over composition to reuse code between components. With this, in order to utilize composition, we have to build upon smaller components. 

Our current way of dealing with components does not work in some cases as some components don't know their children ahead of time. This is where `props.children` comes in. 

Let's update our application so that the login form is only displayed when the user wants to sign in. We'll achieve this by creating a button that causes the form to appear and also by creating a button that hides the form. For the sake of organization and maintainability, as our **App.js** file is growing too large, we'll extract the login form into its own dedicated component:

**/Frontend/src/components/LoginForm.js**

```jsx
import React from 'react'

const LoginForm = ({
    handleLogin,
    handleUsernameChange,
    handlePasswordChange,
    username,
    password
    }) => {

    return (
        <form onSubmit={handleLogin}>
          <div>
            Username
              <input
              type="text"
              value={username}
              name="Username"
              onChange={handleUsernameChange}
            />
          </div>
          <div>
            Password
              <input
              type="password"
              value={password}
              name="Password"
              onChange={handlePasswordChange}
            />
          </div>
          <button type="submit">Login</button>
        </form>      
    )
}

export default LoginForm;
```

As our state is outside of this component and we have not yet moved it over, we created two new functions `handleUsernameChange` and `handlePasswordChange` that will update our state for us from within our component. We'll talk about the definition of these functions in a moment.

Additionally, we can keep the JSX that our component returns the same for the most part for now. We're able to keep these references, even though that we're now referring to props, as we destructure these objects from the `props` object in the parameter list of our function.

For example, instead of writing `this.props.handleLogin` in a class component or `props.handleLogin` when `props` is passed in as the sole parameter of our function as follows:

```jsx
const example = (props) => (
	<form onSubmit={props.handleLogin}>
  	{/* ... */}
  </form>
)
```

We can instead destructure the `handleLogin` function from the `props` object in the parameter list, so that we can instead write:

```jsx
const example = ({handleLogin}) => (
	<form onSubmit={handleLogin}>
  	{/* ... */}
  </form>
)
```

Doing this comes with a number of advantages. Assigning objects to a local variable (doesn't have to just be through destructuring) can allow your code to be minified better when the value is used repeatedly (since the reassigned name can be mangled).

Destructuring also prevents having to do repeated property lookups. Property lookups can be "slow" for objects like `this.props` or `this.state`, although, in practice, this might not have too much of an effect.

Let's proceed with adding a way to track the visibility of our `LoginForm` in our state as well as control the visibility of certain elements based on this state in a new component:

**/Frontend/src/App.js**

```jsx
import LoginForm from './components/LoginForm'

//...

const App = () => {
  
  //...
  
  const [loginVisible, setLoginVisible] = useState(false)
  
  //...
  
  const loginForm = () => {
    const hideWhenVisible = { display: loginVisible ? 'none' : '' }
    const showWhenVisible = { display: loginVisible ? '' : 'none' }

    return (
      <div>
        <div style={hideWhenVisible}>
          <button onClick={() => setLoginVisible(true)}>Log In</button>
        </div>
        <div style={showWhenVisible}>
          <LoginForm
            handleLogin={handleLogin}
            handleUsernameChange={({ target }) => setUsername(target.value)}
            handlePasswordChange={({ target }) => setPassword(target.value)}
            username={username}
            password={password}
          />
          <button onClick={() => setLoginVisible(false)}>Cancel</button>
        </div>
      </div>
    )
  }
  
  //...
  
}

//...
```

Here, we create a new "component" that makes use of our `LoginForm` component in it.

Stepping through the changes, we added a piece of state called `loginVisible` that contains a boolean value pertaining to whether the login form should be displayed.

Then, from within our `loginForm` "component" that returns JSX that takes advantage of our `LoginForm` component, we define two style objects that contain properties that depend on the `loginVisible` state.

```jsx
 const hideWhenVisible = { display: loginVisible ? 'none' : '' }
 const showWhenVisible = { display: loginVisible ? '' : 'none' }
```

`hideWhenVisible` sets `display` to `none` when `loginVisible` is `true` and `showWhenVisible` does the opposite.

We then add our two buttons that toggle our `loginVisible` piece of state:

```jsx
return (
      <div>
        <div style={hideWhenVisible}>
          <button onClick={() => setLoginVisible(true)}>Log In</button>
        </div>
    
    	<div style={showWhenVisible}>
    		{/* ... */}
   			<button onClick={() => setLoginVisible(false)}>Cancel</button>
      </div>
    )
```

Here, you can see that our buttons are wrapped in `div` elements that have their inline style properties set to one of our style objects. This causes their display to be controlled based on the values of `hideWhenVisible` or `showWhenVisible`. Our login form is also within one of these `div` elements, so it will be controlled as well.

### Introduction to `props.children`

Now, if we take a step back and think about what we're doing here for a second, we might come across the idea of reusing this logic of controlling whether a component is displayed or not. With this, we can extract this exact functionality into its own separate component.

Let's create a new component, `Togglable`, that controls the display of a component. For example, instead of the above code that is unique to the `loginForm` "component", we could just write:

```jsx
<Togglable buttonLabel='Login'>
  <LoginForm
    handleLogin={handleLogin}
    handleUsernameChange={({ target }) => setUsername(target.value)}
    handlePasswordChange={({ target }) => setPassword(target.value)}
    username={username}
    password={password}
  />
</Togglable>
```

Previously, most of our components had been *singleton tags* or *self-closing tags* where we do not have an opening and closing tag, but rather just one, single tag in the format of `<Element />`.

Here, `Togglable` has both an opening and closing tag. With this, any elements that fall between the tags `<Togglable>` and `</Togglable>` are considered child components of `Togglable`.

The functionality of `Togglable` involves creating a button with its label set to the `buttonlabel` prop. Upon clicking this button, all child elements will be visible and a new "Cancel" button will appear, allowing the user to hide all of the elements again.

Let's actually write our `Togglable` component:

**/Frontend/src/components/Togglable.js**

```jsx
import React, { useState } from 'react'

const Togglable = ({
  	buttonLabel,
  	children
	}) => {
  
  const [visible, setVisible] = useState(false)

  const hideWhenVisible = { display: visible ? 'none' : '' }
  const showWhenVisible = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{buttonLabel}</button>
      </div>
      <div style={showWhenVisible}>
        {children}
        <button onClick={toggleVisibility}>Cancel</button>
      </div>
    </div>
  )
}

export default Togglable
```

Here, our logic involving toggling the visibility of elements has mostly stayed the same with the exception of a dedicated function that now toggles the `visibility` piece of state, and the movement of the piece of state that tracks the visibility of our component into the `Togglable` component itself.

Everything here seems familiar up until the reference to `children` which has been destructured from `props`.

This entire time, React, in the background, has secretly been adding a `children` property to our `props` object. `props.children` is used in order to reference the child components of our component, so that we can actually include it in our JSX that our component returns. Recall that we discussed some potential limitations with components that might depend on their children or need to know what their children return ahead of time prior to rendering. This is what `props.children` solves.

The `children` property contains an array of all the elements that were defined between the opening and closing brackets of the component. If the component is a singleton, `props.children` will just contain  an empty array.

We can now update our `App` component to simplify our code and make use of this new `Togglable` component:

**/Frontend/src/App.js**

```jsx
import Togglable from './components/Togglable'

//...

const App = () => {

  //...
  
  const loginForm = () => {
    return (
      <Togglable buttonLabel='Login' >
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
  
  //...
  
}

//...
```

With this, we have drastically simplified the `loginForm` function in our `App` component, increasing maintainability and readability. Let's also apply this to our notes form. First let's move the form into its own component:

**/Frontend/src/components/NoteForm.js**

```js
const noteForm = ({
    addNote,
    noteField,
    handleInputChange
    }) => (
    <form onSubmit={addNote}>
        <input
        value={noteField}
        onChange={handleInputChange}
        />
        <button type="submit">Add Note</button>
    </form>  
)

export default noteForm
```

Now, in our `App` component, we can wrap our new component in a `Togglable` component within a functional component:

**/Frontend/src/App.js**

```js
//...

const noteForm = () => (
    <Togglable buttonLabel="Create Note">
      <NoteForm
        addNote={addNote}
        noteField={noteField}
        handleInputChange={handleInputChange}
      />
    </Togglable>
  )
  
//...
```

Now, with `Togglable`, we control the visibility of the component through state that is found within the component rather than within our `App` component. This is better for organization, but it limits us in some ways. For example, how might we go about hiding the note form after submitting a form (handled by a different function) if the visibility of the form is controlled by state found within the component?

We can solve this problem by using refs which gives us a reference to the rendered elements (DOM nodes) to which we can interact with manually. This is often not the best solution as there is often a solution involving a better place to "own" state higher up in the component hierarchy, allowing us to do things purely through state. For now, though, we'll just be using `ref`s:

**/Frontend/src/App.js**

```jsx
//...

	const noteFormRef = React.createRef()
  
  const noteForm = () => (
    <Togglable buttonLabel="Create Note" ref={noteFormRef}>
      <NoteForm
        addNote={addNote}
        noteField={noteField}
        handleInputChange={handleInputChange}
      />
    </Togglable>
  )
  
//...
```

Here, we use the `createRef` method to create a ref that we then assign to the `Togglable` wrapping component that wraps the note form. With this, `noteFormRef` is a reference to our `Togglable` component.

To do this, you'll notice that we had passed in the ref as an attribute. Let's update our `Togglable` component to support refs:

**/Frontend/src/components/Togglable.js**

```jsx
import React, { useState, useImperativeHandle } from 'react'

const Togglable = React.forwardRef(({
  	buttonLabel,
  	children
    }, 
    ref
    ) => {
  
  const [visible, setVisible] = useState(false)

  const hideWhenVisible = { display: visible ? 'none' : '' }
  const showWhenVisible = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

  useImperativeHandle(ref, () => {
    return {
      toggleVisibility
    }
  })

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{buttonLabel}</button>
      </div>
      <div style={showWhenVisible}>
        {children}
        <button onClick={toggleVisibility}>Cancel</button>
      </div>
    </div>
  )
})

export default Togglable
```

Here, we made a few changes. We first wrapped our functional component inside of a call to `forwardRef`  which will allow our component to access the reference that was passed as an attribute to it. 

We also call the `useImperativeHandle` hook which takes in the reference and a function that returns an object containing all of the functions that we might want to make accessible via the ref. In this example, we give it an object containing the `toggleVisibility` function, allowing us to call it through the reference available outside of the component.

Being able to call `toggleVisibility` within that component from outside of the component using the `noteFormRef` that we had created allows us to hide the form after creating a new note. We can do this by updating our `addNote` handler to use this ref:

**/Frontend/src/App.js**

```js
//...

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

//...
```

With this, our app will now hide the form after creating a note. This method of dealing with inaccessible state in an imperative manner is not recommended, and there is almost always a better solution. Please don't rush to use refs as taking a step back can often yield a declarative solution. For example, most of the time, it becomes clear that we can just lift our state up, whether it be creating a component that contains both components with the state for both components being found in that shared component or something else.

You'll find that later, when we introduce Redux, we've been working toward being able to use React properly. In fact, you'll find that React should only be responsible for generating views with state being handled completely separately. For now, though, we'll continue to break some conventions and good practices for the sake of being able to illustrate and learn new concepts. It would be impossible to simply pick up the proper way of using React without first building up to it.

### PropTypes

As our application grows, we'll find that a lot of our bugs could be solved by simply performing basic validation on our props (type checking).

With this, we could make use of TypeScript or Flow to add type checking to our entire application, but for now, we can use React's built-in type checking abilities by assigning the `propTypes` property upon a component.

We can also use `propTypes` to enforce certain behaviors in our application through type checking. For example, we could declare a `Togglable` component without providing the `buttonLabel` prop, and the prop would render without issue. The problem arises in that the button that triggers the display of the prop's children will have no label text. To solve this, we can enforce the provision of a `buttonLabel` prop.

First, let's install the prop-types package:

```bash
npm install prop-types
```

Now, we can update our `Togglable` component to make use of this new property:

**/Frontend/src/components/Togglable.js**

```jsx
//...

import PropTypes from 'prop-types'

const Togglable = React.forwardRef(({
      buttonLabel,
      children
      }, 
      ref
    ) => {
  
Togglable.propTypes = {
  buttonLabel: PropTypes.string.isRequired
}
  
export default Togglable
```

Here, we define the type of `buttonLabel` as a `string` and mention that it is required. Now, if we forget to provide a `buttonlabel` prop, we'll encounter an error:

![image-20191224211155118](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191224211155118.png)

We are not forced to resolve these errors as our application will still continue to run, but we should always resolve to fix these issues.

Let's also define `propTypes` for our `LoginForm` component:

```jsx
import React from 'react'
import PropTypes from 'proptypes'

const LoginForm = ({
      handleLogin,
      handleUsernameChange,
      handlePasswordChange,
      username,
      password
    }) => {

    return (
        {/*...*/}
    )
}

LoginForm.propTypes = {
  handleLogin: PropTypes.func.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired
}

export default LoginForm;
```

### ESLint in React

Recall that we had created our React application using `create-react-app`. This means that the installation of ESLint has already been handled for us and that we **should not install ESLint manually** as we may install a version that is incompatible with the rest of our dependencies.

With this, we still need to configure ESLint for our purposes and our project.

As we begin writing frontend tests, our linter will freak out as it is not expecting the odd syntax of our tests. With this, we can install the `eslint-jest-plugin` package in order to solve this issue:

```bash
npm install eslint-plugin-jest
```

We can now create a **.eslintrc.js** file and put the following rules inside of it (read below for a summary of our changes):

**/Frontend/src/.eslintrc.js**

```json
module.exports = {
  "env": {
      "browser": true,
      "es6": true,
      "jest/globals": true
  },
  "extends": [ 
      "eslint:recommended",
      "plugin:react/recommended"
  ],
  "parserOptions": {
      "ecmaFeatures": {
          "jsx": true
      },
      "ecmaVersion": 2018,
      "sourceType": "module"
  },
  "plugins": [
      "react", "jest"
  ],
  "rules": {
      "indent": [
          "error",
          2
      ],
      "quotes": [
          "error",
          "single"
      ],
      "semi": [
          "error",
          "never"
      ],
      "eqeqeq": "error",
      "no-trailing-spaces": "error",
      "object-curly-spacing": [
          "error", "always"
      ],
      "arrow-spacing": [
          "error", { "before": true, "after": true }
      ],
      "no-console": 0,
      "react/prop-types": 0
  }
};
```

Here, some notable changes include adding our `jest` linter plugin, extending our rules from the recommended ESLint and React rules, and the setting of our indentation amount to 2.

We also added some custom rules to the bottom of the file.

Let's also add a **.eslintignore** file to the root of our frontend directory since we don't need to check anything that's not our source code:

**/Frontend/src/.eslintignore**

```json
node_modules
build
```

Finally, we can create a npm script that will lint our entire directory:

**/Frontend/package.json**

```json
{
  //...
  {
 		//...
  	"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "server": "json-server -p3001 db.json",
    "lint": "eslint .",
  	"lint:fix": "eslint --fix ."
  	},
		//...
	}
	//...
}
```

## Frontend Testing

Let's now take a look at testing our React application.

We can do this through `jest` just as we did with our backend. Now, since our browser does our rendering, and we don't have access to our browser when running our tests, we'll install `react-testing-library` in order to render our components for testing:

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

We can now start writing our tests to **App.test.js** (named by convention) in our **src** directory (tests are stored in the same directory as the component being tested by convention, but it is also acceptable to make a dedicated directory for tests):

```jsx
import React from 'react'
import '@testing-library/jest-dom/extend-expect'
import { render } from '@testing-library/react'
import Note from './components/Note'

test('note renders content', () => {
  const note = {
    content: 'This is a test note',
    flagged: true
  }

  const noteComponent = render(
    <Note note={note} toggleFlagged={jest.fn()} />
  )

  expect(noteComponent.container).toHaveTextContent(
    'This is a test note'
  )
})
```

Here, we create an object representing the content of our note and pass that into our note component which is wrapped by a call to `render` from `react-testing-library`. We also pass in a placeholder function provided by `jest.fn()` since we will get a warning upon rendering that we had not satisfied the prop that is referenced by the `onChange` property within our component.

This component is not rendered to the DOM (or VDOM) like React components typically are. Instead, this component is rendered into an object to which the object has a property `container` which contains the HTML that would be added to our DOM.

With this, we describe an assertion using `expect()` to which we expect the note component's HTML to have the content of our note object somewhere in it.

#### Default Test Behavior

Upon running `npm test`, you'll find that the test process does not end but rather continues watching for any changes in the directory. 

Here, you may run into a warning that `watchman` is not installed. `watchman` is used to detect when we make changes. With this, follow the instructions provided to install it on your respective OS. If you are on macOS and have `brew` installed, you can simply run the command:

```bash
brew install watchman
```

We can emulate the behavior of the tests in our backend of which the process will terminate after running the tests by passing the `CI` argument with a value of true:

```bash
CI=true npm test ./src/App.test.js
```

#### Additional Component Checks

As with most testing libraries, there are plenty of ways to achieve the same checking of a component. For example, we could use the following methods to perform the above check:

- `toHaveTextContent()` to just see if the container has our desired text
- `getByText()` to which we pass in the desired text and check if the object is defined using `toBeDefined()`. If the object is defined, then we have a component that contains our desired text.
- `querySelector()` to which we select our note, not by reference through a variable, but instead through its `className` and then perform `toHaveTextContent()` upon it.

### Debug Tools

The objects returned by `render()` contain a `debug()` method that will log the HTML of the component of the console so that we can investigate our component further. Let's add this to our test so that we can see the HTML of our component while our test is running:

**/Frontend/src/App.test.js**

```jsx
import React from 'react'
import '@testing-library/jest-dom/extend-expect'
import { render } from '@testing-library/react'
import Note from './components/Note'

test('note renders content', () => {
  const note = {
    content: 'This is a test note',
    flagged: true
  }

  const noteComponent = render(
    <Note note={note} toggleFlagged={() => {}} />
  )
  
  noteComponent.debug() // Logs the HTML of the component to the console

  expect(noteComponent.container).toHaveTextContent(
    'This is a test note'
  )
})
```

Upon running our tests, we receive the following output with the HTML of our component:

![image-20191226194358291](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191226194358291.png)

`debug()` is simply a shortcut for `console.log(prettyDOM())` to which `prettyDOM` comes from the `DOM Testing Library` and just lays out the HTML of our component in a readable manner.

Let's import and use `prettyDOM ` manually to inspect the contents of a smaller element from within our component:

```js
import React from 'react'
import '@testing-library/jest-dom/extend-expect'
import { render } from '@testing-library/react'
import { prettyDOM } from '@testing-library/dom' // prettyDOM import
import Note from './components/Note'

test('note renders content', () => {
  const note = {
    content: 'This is a test note',
    flagged: true
  }

  const noteComponent = render(
    <Note note={note} toggleFlagged={() => {}} />
  )
  
  noteComponent.debug() // Logs the HTML of the component to the console
  const listElement = noteComponent.container.querySelector('li') // use querySelector to get our list element

  console.log(prettyDOM(listElement)) // logs the prettyDOM object

  expect(noteComponent.container).toHaveTextContent(
    'This is a test note'
  )
})
```

Now, as we begin to write extensive tests, you'll find that things can become quite repetitive with all of the patterns involving checking for an element's attributes, text content, finding by CSS class, etc.

`jest-dom` attempts to provide a partial solution by providing custom jest matchers that test the state of the "DOM".

We could import the matchers by adding the following line to our test files:

```jsx
import '@testing-library/jest-dom/extend-expect'
```

You'll notice that we are not assigning it to a variable or anything. With this, to prevent the repetitive nature of having to import all of these modules in every test file that we write, let's create a configuration file that imports these modules for us (in newer versions of `create-react-app` the file may already be created):

**/Frontend/src/setupTests.js**

```jsx
import '@testing-library/jest-dom/extend-expect'
```

> This is only recommended if you are unable to get `jest` to use **setupTests.js**. This method is not supported by some versions of `create-react-app`. By default, `jest` should make use of this configuration file, but we can force it to by adding the following to our `jest` object in our **package.json** file:
>
> **/Frontend/package.json**
>
> ```json
>  "jest": {
>     //...
>     "setupFiles": [
>       "<rootDir>/src/setupTests.js"
>     ],
>     //...
>   }
> ```

### Testing Events

In our `Note` component, we feature a checkbox that should fire the `toggleFlagged` event handler function. We can test this functionality by making use of the `fireEvent` function provided to us by `react-testing-library`:

**/Frontend/src/App.test.js**

```js
//...

test('checking the checkbox fires event handler once', () => {
    const note = {
        content: 'This is a test note',
        flagged: true
      }
  
    const mockHandler = jest.fn()
  
    const { getByAltText } = render(
      <Note note={note} toggleFlagged={mockHandler} />
    )
  
    const checkbox = getByAltText('Toggle Flagged')
    fireEvent.click(checkbox)
  
    expect(mockHandler.mock.calls.length).toBe(1)
  })
  
  
  //...
```

Here, we create a mock handler as we did earlier using `jest.fn()`. This time, we store the mock function in a constant so we can look to see how many times it has been called later (all handled by `jest`).

Then, we find the our checkbox by the alt text, "Toggle Flagged" (that we just added through the `alt` property of the `input` element), and fire a `click` event upon it using `fireEvent()`. We then access our `mockHandler` constant so that we can assert that it should have been called once.

A few things to note include us dereferencing only the functions that we need (`getByAltText`) from the object that `render()` returns, as well as the existence of `jest.fn()` which allows us to declare functions that behave predictably, create stub functions, and gain access to properties that provide information that we would not otherwise be able to retrieve.

### Writing More Tests

Let's write a few more tests relating to our component, `Togglable`.

First, to make the `div` element surrounding the children of the component easier to target, let's add a class name to it:

**/Frontend/src/components/Togglable.js**

```jsx
import React, { useState, useImperativeHandle } from 'react'
import PropTypes from 'proptypes'

const Togglable = React.forwardRef(({
      buttonLabel,
      children
      }, 
      ref
    ) => {
  
  //...

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{buttonLabel}</button>
      </div>
      <div style={showWhenVisible} className="togglableContent">
        {children}
        <button onClick={toggleVisibility}>Cancel</button>
      </div>
    </div>
  )
})

//...

export default Togglable
```

Now, we can go ahead and write tests for our `Togglable` component:

**/Frontend/src/Togglable.test.js**

```jsx
import React from 'react'
import { render, fireEvent } from '@testing-library/react'
import Togglable from './Togglable'

describe('<Togglable /> Component Tests', () => {
  let component

  beforeEach(() => {
    component = render(
      <Togglable buttonLabel="Show Children">
        <div className="childDiv" />
      </Togglable>
    )
  })

  test('children are rendered', () => {
    component.container.querySelector('.childDiv')
  })

  test('children are not displayed when not shown', () => {
    const div = component.container.querySelector('.togglableContent')

    expect(div).toHaveStyle('display: none')
  })

  test('children are displayed after clicking show button', () => {
    const button = component.getByText('Show Children')
    fireEvent.click(button)

    const div = component.container.querySelector('.togglableContent')
    expect(div).not.toHaveStyle('display: none')
  })
})
```

Here, we simply wrap all of our tests in a describe block and employ other concepts that we have already been exposed to. Before each test, we render the `Togglable` component to the `component` variable of which the `Togglable` component has a `div` element as its child. We then check if it is rendered, if the child div is not shown (has `display: 'none'`), and if the child div is shown after clicking on the button (does not have `display: 'none'`).

Now, something to note is that `querySelector` returns the first match that it gets based on the query. In our case, it returned the first button in our component and not the second "cancel" button that is currently hidden.

This leads to the question of retrieving elements that are not the first query match. This is done through the query using a selector like we would in CSS. Let's define a test that tests our components ability to hide the content upon clicking the cancel button:

**/Frontend/src/Togglable.test.js**

```jsx
//...

 test('children are hidden after clicking cancel button', () => {
    const showButton = component.getByText('Show Children')
    fireEvent.click(showButton)
  
    const cancelButton = component.container.querySelector(
    'button:nth-child(2)'
 	 	)
   	fireEvent.click(cancelButton)
  
    const div = component.container.querySelector('.togglableContent')
    expect(div).toHaveStyle('display: none')
  })

//...
```

Here, we simply use the query `button:nth-child(2)` to find the second element that matches the query. Finding components in this manner is not recommended and should be identified based on text, alt-text, or ID:

**/Frontend/src/Togglable.test.js**

```jsx
//...

test('children are hidden after clicking cancel button', () => {
    const showButton = component.getByText('Show Children')
    fireEvent.click(showButton)
  
    const cancelButton = component.getByText('Cancel')
    fireEvent.click(cancelButton)
  
    const div = component.container.querySelector('.togglableContent')
    expect(div).toHaveStyle('display: none')
  })

//...
```

#### Form and Input Testing

Let's expand our tests to test our `NoteForm` component.

In order to properly test this component, we would have to enter some value into our `input` element and check the outcome. We can do this by taking advantage of `fireEvent` once again to fill out our input element:

**/Frontend/src/components/NoteForm.test.js**

```jsx
import React from 'react'
import { render, fireEvent } from '@testing-library/react'
import NoteForm from './NoteForm'

const Wrapper = (props) => {

  const handleInputChange = (event) => {
    props.state.noteField = event.target.value
  }

  return (
    <NoteForm
      addNote={props.addNote}
      noteField={props.state.noteField}
      handleInputChange={handleInputChange}
    />
  )
}

test('<NoteForm /> updates through handleInputChange and calls addNote', () => {
  const addNote = jest.fn()
  const state = {
      noteField: ''
  }

  const component = render(
    <Wrapper addNote={addNote} state={state} />
  )

  const input = component.container.querySelector('input')
  const form = component.container.querySelector('form')

  fireEvent.change(input, { target: { value: 'Test form input' } })
  fireEvent.submit(form)

  expect(addNote.mock.calls.length).toBe(1)
  expect(state.noteField).toBe('Test form input')
})
```

Recall that our `NoteForm` component is dependent on the `App` component where we pass it a `handleInputChange` function, `addNote` function, and the `noteField` piece of state. The `NoteForm` component simply does the job of triggering a function on submission and updating the state of its parent component. With this, we made the helper component, `Wrapper`, that emulates our `App` component in the sense that it contains the state and everything that our `NoteForm` component needs.

Here, you can see how we pass the actual `addNote` mock function that we want to track through the props of the component. We also pass in a mock state through the `state` property to which we can access this `state` object from within our component as a property of our `props` object.

Then, we use the `handleInputChange()` function to actually update this state object from changes to our `input` element. We take advantage of the `change()` method of `fireEvent` in order to simulate a user writing a note in the `NoteForm` component.

Finally, we make some assertions at the end regarding the value of our state and the amount of calls that our mock function had received.

### Integration Testing

So far, we have only written unit tests for our application. While these tests verify that our components work individually, we need to actually verify that our components will actually work together to an extent. This is where integration tests come in.

Integration testing is often much more complicated than unit testing. It poses challenges such as needing to fetch data from a backend or from local state, giving us two more areas that we have to mock.

Let's start by mocking the LocalStorage API by creating an object that we can manage. We'll do this in our **setupTests.js** file so that we don't have to write the same block of code in every test and so that we have a single source for this mock:

**/Frontend/src/setupTests.js**

```jsx
import '@testing-library/jest-dom/extend-expect'

let savedItems = {}

const localStorageMock = {
  setItem: (key, item) => {
    savedItems[key] = item
  },
  getItem: (key) => savedItems[key],
  clear: () => {
    savedItems = {}
  }
}

Object.defineProperty(window, 'localStorage', { value: localStorageMock })
```

Here, we declared a `savedItems` varaibles which will act as the storage for our mock. We then have an object `localStorageMock` that contains two functions which will allow us to modify this object. Finally, we use `Object.defineProperty` to add this `localStorageMock` object to the `window` object under the property `localStorage`.

Now, to address the problem of fetching notes from our MongoDB database, we can simply replace it with a hardcoded object that represents the collection of our notes. The use of hardcoded data also allows increased speed and stability as we do not have to access a database whose state may change over time. 

With this, let's mock our `noteService` module with a module that makes use of and returns hardcoded data. In Jest, manual mocks of modules are defined through writing a module in a **\_\_mocks\_\_** subdirectory in the same directory of the module. As our `noteService` module (**/Frontend/src/services/notes.js**) lies within the **services** directory, we will make a **\_\_mocks\_\_** directory within it and place our mock service within it:

**/Frontend/src/services/\_\_mocks\_\_/notes.js**

```jsx
const notes = [
  {
  "content": "This is a mock note!",
  "flagged": true,
  "date": "2019-12-23T04:32:47.154Z",
  "user": {
    "username": "jassuf",
    "name": "joe",
    "id": "5df151019f8679503a3de183"
  },
  "id": "5e00436f4001db92b2ab0a00"
  },
  {
    "content": "This is another mock note.",
    "flagged": false,
    "date": "2019-12-23T04:33:21.129Z",
    "user": {
      "username": "jassuf",
      "name": "joe",
      "id": "5df151019f8679503a3de183"
    },
    "id": "5e0043914001db92b2ab0a01"
  },
  {
    "content": "This is the third mock note.",
    "flagged": false,
    "date": "2019-12-23T07:11:38.841Z",
    "user": {
      "username": "jassuf",
      "name": "joe",
      "id": "5df151019f8679503a3de183"
    },
    "id": "5e0068aa4001db92b2ab0a02"
  }
]

const getAll = () => {
  return Promise.resolve(notes)
}

export default { getAll }
```

Here, we hardcoded the data that our database might have. We also created a mock `getAll` function that we would find within our real `noteService`. This function actually returns a promise that will resolve to the list of notes. We do this because, in our actual application, we return a promise as a result of the `then()` method of the promise that `axios.get()` gives us.

We can now go ahead and write our integration test relating to our `App` component:

**/Frontend/src/App.test.js**

```jsx
import React from 'react'
import { render,  waitForElement } from '@testing-library/react'
jest.mock('./services/notes')
import App from './App'

describe('<App /> Integration Tests', () => {
  test('renders all notes from service', async () => {
    const component = render(
      <App />
    )
    component.rerender(<App />)
    await waitForElement(
      () => component.container.querySelector('.mainNote')
    )

    const notes = component.container.querySelectorAll('.mainNote')
    expect(notes.length).toBe(3) 

    expect(component.container).toHaveTextContent(
      'This is a mock note!'
    )
    expect(component.container).toHaveTextContent(
      'This is another mock note.'
    )
    expect(component.container).toHaveTextContent(
      'This is the third mock note.'
    )
  })
})
```

Through a call to `jest.mock()`, `jest` will find the import of the referenced module and check if there is a mock folder with the appropriate mock module in the same directory of the original module. If there is, it will use the manual mock, but if not, it will go ahead and create an *automatic mock* with functions that typically return `undefined`.

`jest` allows for automatic mocks and extending actual modules from within manual mocks, but given the size of our current project, we are not going to discuss this at this time. As your projects scale, it is recommended to read more of the `jest` documentation in order to take advantage of newer features.

Looking at our test, we first call a `rerender` and await the rendering of all of our elements that have the `.mainNote` class. We rerender our component just to ensure that our effects are triggered, even though our current events should execute after the first render.

### Determining Test Coverage

As our projects begin to grow larger and the amount of tests that we write increases, it would be helpful to be able to see what parts of our project do not have tests written for them. We can determine the *test coverage* of our project with `jest`'s integrated coverage reported which typically requires no configuration.

In order to generate a coverage report, add `-- --coverage` to your test command as such:

```bash
npm test -- --coverage
```

> The extra `--` in the middle of the command is not a typo.

This will run our tests and output a table form of our coverage report as well as the results from our tests:

![image-20191229020806939](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191229020806939.png)

We are also given a new **coverage** directory in the root of our project that gives us a web interface to browse our code coverage:

![image-20191229021024601](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191229021024601.png)

It even will highlight individual lines of code that have not been tested:

![image-20191229021104428](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191229021104428.png)

### Jest Snapshot Testing

Jest also offers another form of testing with regard to *snapshot* testing. Snapshot testing is useful for when you want to ensure that the user interface hasn't change. WIth this, the core principle of snapshot testing is very simple: upon creating the test, a snapshot will be generated of a given component, etc. to which this snapshot is compared to the live rendering of a component.

For example, with snapshot testing, we would render our `NoteForm` component to HTML and store it as a JSON representation (or something else) with this being our snapshot. Then, upon running our tests, we repeat the same rendering process and compare our output to this snapshot. If there are differences, then we are notified as to be made aware that there were some changes to the component that are reflected in the HTML.

### E2E Testing

E2E testing (or end-to-end testing) are the next logical progression of tests beyond integration testing. They simulate the tasks that a user will ultimately make on our application, leading to practical tests that test our application in a real-world way. These tests can truly make use of the entire system, and as a result, test the entire system.

E2E testing is achieved through simulating a browser (usually in a headless mode to which the browser never actually has to render anything graphically). `Cypress` and `Selenium` are examples of tools that allow us to run end-to-end tests that interact with the same interface that users will actually interact with.

While there is benefit in testing through the final interface that users will interact with, in that you are testing everything together when doing so, there are also a few big downsides. One large downside is that they are much harder to set up and write compared to unit or even integration tests. Another large downside which greatly reduces their practicality is their run time. Development usually requires a fast feedback loop to which developers greatly benefit from being able to identify errors and correct them quickly. Unit tests and integration tests shine in this aspect, but E2E tests often take minutes to hours to run, which makes quickly identifying problems impossible.

We will look further into E2E testing later after we increase the complexity of our project with a state management library.

## Custom Hooks

For the most part, most of our component logic has been found within a specific component and could not be reused between other components.

We've used reusable component logic in the sense of using the `useState` and `useEffect` hooks, but everything that we've written has been component-specific.

In addition to the 10 built-in hooks and their APIs that come with React, we are also given the option to create custom hooks. With this, building our own hooks will allow us to extract component logic into reusable functions.

For the most part, React hooks are regular JavaScript functions that have to adhere to two main rules:

* Hooks can only be called at the top level of the component
  * This means that they should not be called inside loops, conditional blocks, nested functions, etc.
    * This ensures that the hooks are called in the exact same order every time the component renders, allowing React to hold on and preserve the state between multiple calls.
    * As for background, when taking the `useState()` hook as an example, you might wonder how React knows which piece of state corresponds to each `useState()` call. It does this through tracking the order of which the hooks are called. This is why it is important that hooks are called within the top level of the component, because if they are executed conditionally or multiple times, it alters the order and leads to the behavior of our hooks to shift.
* Hooks can only be called from React functions
  * We can only call hooks from React function components and within other custom hooks.
    * This allows us to see all stateful logic related to a component from standardized areas in its source code.

# INSERT MORE INFORMATION ON WHY REACT HOOKS RELY ON ORDER HERE

As per convention, we should also make sure that the names of our hooks start with `use`.

ESLint has a plugin that can help us out with these rules so that we don't accidentally violate one of these rules. We can install it by running the following command:

```bash
npm install --save-dev eslint-plugin-react-hooks
```

After installing the plugin, we can configure it by adding it to our **.eslintrc.js** file:

```js
module.exports = {
  //...
  "plugins": [
        "react",
        "jest",
        "react-hooks"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    //...
  }
}; 
```

We will now gain access to syntax highlighting and underlining for hooks within VSCode and will get thrown an error if we violate these rules upon linting our project.

As an introduction to hooks, let's just create a simple counter using custom hooks. Previously, we might have created a counter as follows:

```jsx
import React, { useState } from 'react'
const App = (props) => {
  const [counter, setCounter] = useState(0)

  return (
    <div>
      <div>{counter}</div>
      <button onClick={() => setCounter(counter + 1)}>
        plus
      </button>
      <button onClick={() => setCounter(counter - 1)}>
        minus
      </button>      
      <button onClick={() => setCounter(0)}>
        zero
      </button>
    </div>
  )
}
```

With React Hooks, we can first abstract the logic involving the counter and its state to its own hook:

```jsx
const useCounter = () => {
  const [value, setValue] = useState(0)

  const increase = () => {
    setValue(value + 1)
  }

  const decrease = () => {
    setValue(value - 1)
  }

  const reset = () => {
    setValue(0)
  }

  return {
    value, 
    increase,
    decrease,
    reset
  }
}
```

Notice how we call the `useState()` hook from within our own custom hook. Here, our hook behaves similarly to how an abstraction over our mock local storage module behaved. This hook, upon being called, will return an object containing functions that allow us to set the internal state of this component, as well as the internal state itself.

Now, we can make use of this hook within a component:

```jsx
const App = (props) => {
  const counter = useCounter()

  return (
    <div>
      <div>{counter.value}</div>
      <button onClick={counter.increase}>
        Increase
      </button>
      <button onClick={counter.decrease}>
        Decrease
      </button>      
      <button onClick={counter.zero}>
        Reset
      </button>
    </div>
  )
}
```

Here, in our component, we just lay out the elements of our counter and make use of the `useCounter()` hook for all of our logic and state relating to the counter.

In this case, unlike what we did earlier with `useState()`, we did not destructure the object that our hook returned.

This code is arguably more manageable than the solution mentioned previously that did not make use of a custom hook, but the true potential of custom hooks is realized when we reuse this logic.

If we wanted a component that required two counters within it, rather than making a specific counter component and then a parent wrapper component, we can easily take advantage of our custom hook in one component. For example, let's create an component that has counters for the amount of times a button has been clicked/side has been voted on:

```jsx
const App = () => {
  const leftVote = useCounter()
  const rightVote = useCounter()

  return (
    <div>
      {leftVote.value}
      <button onClick={leftVote.increase}>
        Vote for Left
      </button>
      <button onClick={rightVote.increase}>
        Vote for Right
      </button>
      {rightVote.value}
    </div>
  )
}
```

Here, our component contains two separate counters with their logic in separate instances of the `useCounter()` hook. We then call the respective functions of the hooks and display their states to provide the interaction and display of the two counters.

Now, let's look into a more applicable hook for our application. From our previous experiences, dealing with forms in React can be quiet unorganized sometimes, especially when we need to assume control of multiple input elements (making them into controlled components so that we can track their state).

Thinking about this, the logic behind creating a controlled form element can be reused across multiple instances in a form. With this, let's create a hook that will hold and manage the state for a given controlled element:

```jsx
const useField = (type) => {
  const [value, setValue] = useState('')

  const onChange = (event) => {
    setValue(event.target.value)
  }

  return {
    type,
    value,
    onChange
  }
}
```

Here, we create a piece of state so that we can keep track of what is typed in our input element. We then return a function that takes in an event and sets our internal state to the target value of that event (this is what we can pass to the `onChange` property of our actual form element). We also keep track of the element type and return that as well as the state.

Later, if you end up using Formik in an application (form library for React), `useField()` is a custom React hook that will allow you to hook up inputs to Formik. We emulate this behavior here.

As a quick example, we can use the hook as such:

```jsx
const App = () => {
  const userEmail = useField('text')
  // ...

  return (
    <div>
      <form>
        <input
          type={userEmail.type}
          value={userEmail.value}
          onChange={userEmail.onChange} 
        /> 
      </form>
    </div>
  )
}
```

Here, our `input` element is controlled by the logic in the `userEmail` hook. In this case, since our key values are the same as the key values in the object that we're trying to pass values from, we can use the spread operator to break down our object into the correct fields:

```jsx
const App = () => {
  const userEmail = useField('text')
  // ...

  return (
    <div>
      <form>
        <input {...userEmail} /> 
      </form>
    </div>
  )
}
```

This produces the exact same result as manually referencing the properties of the object and matching them up with the appropriate property of the element.

As per the React documentation, the following two components are equivalent:

```jsx
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

You can see how the `props` object contains the exact same properties that the `Greeting` component takes in, to which when we use the spread operator upon the object, it is broken down and the properties of it are assigned to their respective fields.

With this, many libraries are starting to come with their own custom hooks. Additionally, you can find plenty of hooks and examples of real-world usage of hooks on the Internet. One especially good source for this content would be the `awesome-react-hooks` repository on GitHub.

# State Management with Redux

So far, we've just dealt with simple component state where we just followed the convention of lifting state up to the greatest common ancestor component and passed it down through props. This isn't too manageable with large, complex applications. To address this, Facebook created an alternative approach to state management.

### Flux Architecture

Flux is the application architecture that Facebook uses (they are transitioning to Redux, however) to build client-side web applications. It utilizes a unidirectional data flow, similar to how passing down state through components is unidirectional. Rather than a full framework, Flux is essentially a pattern of state management, to which there are several implementations (see Redux below).

One key difference is that the state is separated completely from React components into their own *stores*. This state is then modified (not directly, but updated) through the use of *actions*. Typically, when an action changes the state of a store,  they emit an event that causes the views (components) to be rerendered.

![img](https://miro.medium.com/max/2600/1*Ek68XwgLgxwlAZl6hhxx1w.png)

> Annotated Flux Overview by Facebook: [facebook.github.io/flux/docs/in-depth-overview.html](https://facebook.github.io/flux/docs/in-depth-overview.html)

Here, you can see how state is updated and triggers events in components while being separate from the components. For example, if you clicked a button that incremented a piece of state in a store, the change would have to be made with an action. This action is then provided to the dispatcher who sends the action to all stores. These stores will then update themselves and their state and emit a *change event* once they are done. Then the *controller-views* consisting of components listen for these changes and update their state, triggering a rerender (usually).

### Redux

While Facebook has an implementation for their Flux pattern, we will be using the Redux library. Redux also follows a strict unidirectional data flow. To provide a gentle introduction to the topic of Redux, we'll just be making a simple counter application without any backend for now.

Let's create a brand new project, and initialize it with `create-react-app`. We'll also go ahead and install Redux by executing:

```bash
npm install redux
```

With Redux, the entire application state can be found in just one JavaScript object in the store. Now, with this application, since we're only keeping track of the value of the counter, we'll save our state right to the store object, but in more complicated applications, you would save the state to separate fields of the object.

#### Actions

Recall that we update our state of the store through the use of actions. By definition, actions are payloads of information that send data from our application to the store. Just like everything else in JavaScript, actions are objects which contain, at the very least, a property describing the type of the action.

For example, with our incrementor, we need an action with the type of `INCREMENT`:

```js
{
  type: 'INCREMENT'
}
```

The type of our action can be anything, it just needs to be specified when we actually send out our action. Other than `type`, the structure of an action could be anything. If our application was more complicated and required specific data to be sent in the payload, we could specify more fields, but since we just need to know whether to increment, decrement, or reset with our application, we can just look to the `type` property.

#### Reducers

In order to actually update our application's state in response to these actions, we need to specify this behavior through the use of **reducers**. To which, a reducer is essentially a function that takes the current state and an action and returns a new state dependent on that action (similar to how `map()` and other functions do not mutate the original state, but take in an action and return a new state).

With this, let's specify a reducer for our counter application:

```js
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT:
      return state + 1
    case 'DECREMENT'
      return state - 1
    case 'RESET'
      return 0
    default:
      return state
  }
}
```

Here, dependent on the action type, we update our state correspondingly. Reducers will take in the original state and the action as inputs and will return the updated state as the output. Now, we must create a store that will bring our actions and reducers together. We also use a *default function parameter* to set `state = 0` if no value or `undefined` is passed to `state`.

#### Store

The Store is the object that brings the actions (which essentially describe "what happened") and the reducers (which update the state according to the actions) together.

It is important to realize that a reducer should never (except in some very odd cases) be called directly from the application code. This is what the store is for.

The store is responsible for:

* Holding the application state
* Allowing access to the state through a call to `getState()`
* Allowing updates to the state through a call to `dispatch(action)`
* Registering listeners through a call to `subscribe(listener)`
* Handling unregistering of listeners through the function returned by `subscribe(listener)`

The last 2 responsibilities may sound very unfamiliar. Do not worry as we will soon cover these topics. Additionally, it is important to note that we'll only have a singular store in our entire application. Eventually, when we want to use multiple reducers to split our data handling logic, we'll use *reducer composition* to combine our separate reducers into one single reducer that we can give to our store.

Moving back to our counter application, we can create the store by passing the reducer to `createStore()`:

```jsx
import { createStore } from 'redux'

const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    case 'RESET':
      return 0
    default:
      return state
  }
}

const store = createStore(counterReducer)
```

The store now has our `counterReducer` to handle any actions that we *dispatch* (send) to it using the `dispatch()` method:

```js
store.dispatch({type: 'INCREMENT'})
```

We can call `dispatch(action)` from anywhere in our application. If we wanted to get the state of our store, we can just call `getState()` upon the store. 

#### Registering Listeners via Subscribe

For the purposes of testing our application for now, let's register a listener upon our store that listens to every single change upon it:

```jsx
//...

const store = createStore(counterReducer)

store.subscribe(() => {
  const currentStore = store.getState()
  console.log(currentStore)
})
```

Here, we listen for any change upon the store. Upon a change, the function that we had provided will be executed. In this case, the function simply just logs the store object to the console.

Let's add the following to our application:

```jsx
store.dispatch({ type: 'INCREMENT' })
store.dispatch({ type: 'INCREMENT' })
store.dispatch({ type: 'DECREMENT' })
store.dispatch({ type: 'RESET' })
store.dispatch({ type: 'INCREMENT' })
```

Upon executing our code, these actions are dispatched and our reducers update our store accordingly. Then, our listeners observe the change and print the following to the console:

```bash
1
2
1
0
1
```

We can now render a UI for our counter that calls these dispatchers upon button clicks as follows:

**/counter-application/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'

const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    case 'RESET':
      return 0
    default:
      return state
  }
}

const store = createStore(counterReducer)

store.subscribe(() => {
  const currentStore = store.getState()
  console.log(currentStore)
})

const App = () => {
  return (
    <div>
      <div>
        {store.getState()}
      </div>
      <button 
        onClick={e => store.dispatch({ type: 'INCREMENT' })}
      >
        Increment
      </button>
      <button
        onClick={e => store.dispatch({ type: 'DECREMENT' })}
      >
        Decrement
      </button>
      <button 
        onClick={e => store.dispatch({ type: 'RESET' })}
      >
        Reset
      </button>
    </div>
  )
}

const renderDOM = () => {
    console.log('Rendering application...')
    ReactDOM.render(<App />, document.getElementById('root'))
}

renderDOM()
store.subscribe(renderDOM)
```

Here, we create an `App` component that simply calls the appropriate dispatcher for the button that emits the `onClick` event. We display the current counter value by grabbing the state of the store using `getState()`. Finally, at the end, we create a function that will call `ReactDOM.render()` to render our `App` component. We also subscribe to the store and pass the `renderDOM` function to it so that our component will be rerendered upon every change to our state.

> Note that we have to manually call `renderDOM()` as the function would not execute otherwise, leading to the component not being displayed.

### Redux + Notes Application

With the general idea of Redux and Flux Architecture down, let's first start making a version of our notes application that uses Redux but does not connect to the backend (for the sake of simplicity):

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'

import noteReducer from './reducers/noteReducer'

const store = createStore(noteReducer)

store.dispatch({
  type: 'NEW_NOTE',
  data: {
    content: 'This note is stored in the Redux store',
    flagged: true,
    id: 1
  }
})

store.dispatch({
  type: 'NEW_NOTE',
  data: {
    content: 'This is a new note created with Redux',
    flagged: false,
    id: 2
  }
})

const App = () => {
  return (
    <div>
      <ul>
        {store.getState().map(note=>
          <li key={note.id}>
            <input type="checkbox" alt="Toggle Flagged" checked={note.flagged} />
            {note.content} 
          </li>
        )}
        </ul>
    </div>
  )
}

const renderDOM = () => {
    console.log('Rendering application...')
    ReactDOM.render(<App />, document.getElementById('root'))
}

renderDOM()
store.subscribe(renderDOM)
```

Our initial app does not have any project organization with separate modules, but it does lay out the groundwork for the structure of our notes and being able to send out note actions to add notes to our store using our `noteReducer`.

The two `dispatch(action)` calls create two `NEW_NOTE` actions to which the payload has a `data` property which contains our actual note "object". This payload is received by the `noteReducer` reducer to which it concatenates the data if the action was of the type `NEW_NOTE`. We use `concat()` rather than `push()` (just as we did with React components) on the data as reducers are meant to be *pure functions* (functions that take in an input and return a value without modifying anything) to which we do not modify the store/state directly, but rather return a new state with the updated information. Pure functions, due to their nature of not being dependent on state or affecting state will return the same response every single time they are called with the same parameters.

Another thing to note is the manner of which we map over our store's state in order to render an unordered list of our store's notes.

### Testing Reducers

Let's employ some aspects of test-driven development by creating a test for our `noteReducer`'s current features as well as its ability to toggle the flagged property of a note, keeping in mind that we had not implemented this yet.

Before creating our tests, let's move our reducer over to its own dedicated directory, **reducers**, for the sake of organization:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
    switch (action.type) {
        case 'NEW_NOTE':
            return state.concat(action.data)
        default:
            return state
    }
}

export default noteReducer
```

For our test, we'll be utilizing a library, `deep-freeze`, that will allow us to freeze objects recursively. This is important, because we can freeze our state and ensure that nothing (our reducers, etc.) is directly modifying it. We can install `deep-freeze` by running:

```bash
npm install --save-dev deep-freeze
```

Similar to React components, we define the tests for a given reducer in the same directory as the reducer, just with a **.test.js** file ending:

**/src/reducers/noteReducer.test.js**

```jsx
import noteReducer from './noteReducer'
import deepFreeze from 'deep-freeze'

describe('noteReducer Unit Tests', () => {
  test('action NEW_NOTE returns new state', () => {
    const state = []
    const action = {
      type: 'NEW_NOTE',
      data: {
        content: 'This should update the store state',
        flagged: false,
        id: 1
      }
    }

    deepFreeze(state)
    const newState = noteReducer(state, action)

    expect(newState.length).toBe(1)
    expect(newState).toContainEqual(action.data)
  })
})
```

Here, we simply freeze the state so that we can't modify it directly, allowing us to see via a thrown error if we are not defining reducers correctly. With this, we just attempt to get the updated state using the `NEW_NOTE` action to which we store it in `newState` and expect it to contain 1 object as well as the notes data.

Let's change our `noteReducer` to utilize `array.push()` instead of `array.concat()` so that it attempts to directly modify the state:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
    switch (action.type) {
        case 'NEW_NOTE':
        		state.push(action.data)
            return state
        default:
            return state
    }
}

export default noteReducer
```

Upon running our tests, we receive the following error as a result of directly mutating the state that has been frozen:

![image-20191230182443818](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191230182443818.png)

Let's now revert our changes and use `array.concat()` once again. With this, the test surrounding the `NEW_NOTE` action of `noteReducer` passes. 

As stated earlier, we'll go ahead and create a test for toggling the flagged property of our notes before actually implementing the feature (test-driven development):

**/src/reducers/noteReducer.test.js**

```jsx
//...

test('action TOGGLE_FLAGGED returns new state with note flagged changed', () => {
    const state = [
      {
        content: 'This is stored in the store',
        flagged: false,
        id: 1
      },
      {
        content: 'This state will be updated via actions',
        flagged: true,
        id: 2
      }]
  
    const action = {
      type: 'TOGGLE_FLAGGED',
      data: {
        id: 2
      }
    }
  
    deepFreeze(state)
    const newState = noteReducer(state, action)
  
    expect(newState.length).toBe(2)
  
    expect(newState).toContainEqual(state[0])
  
    expect(newState).toContainEqual({
      content: 'This state will be updated via actions',
      flagged: false,
      id: 2
    })
})
```

Here, we write a test surrounding the `TOGGLE_FLAGGED` action of `noteReducer`.  The action that we perform in the test is:

```jsx
{
  type: 'TOGGLE_FLAGGED',
  data: {
    id: 2
  }
}
```

This action should toggle the flagged property of the note with `id` 2.

Now, let's update our reducer with the functionality outlined in this test:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
    switch (action.type) {
        case 'NEW_NOTE':
            return state.concat(action.data)
        case 'TOGGLE_FLAGGED': {
            const noteId = action.data.id
            const originalNote = state.find(n => n.id === noteId)
            const updatedNote = {
                ...originalNote,
                flagged: !originalNote.flagged
            }
            return state.map(n => n.id === noteId ? updatedNote : n)
        default:
            return state
        }
    }
}

export default noteReducer
```

We now have a case that handles the `TOGGLE_FLAGGED` action type. The logic behind updating our note is similar to how we have updated it in our previous application without Redux.

To update the flagged property, we first find the note with the matching `id` using the array `find()` method:

```js
const originalNote = state.find(n => n.id === noteId)
```

Recall that this method returns the object being passed in the current iteration when the return value of the specified function is truthy.

We then create the updated note object with the `flagged` property:

```js
const updatedNote = {
  ...originalNote,
  flagged: !originalNote.flagged
}
```

The object spread operator (`...`) is used here to copy the properties over (shallowly) so that we don't have to manually copy over each of the properties. We also made use of this in our other application.

Finally, we return the updated state by mapping over the old state and replacing the referenced nte with the updated note:

```jsx
return state.map(n => n.id === noteId ? updatedNote : n)
```

We do this by mapping over the array with a function that checks if the current note that is being iterated over matches the `id` of the action. If the `id` matches, then we simply replace the note with the `updatedNote`. This line makes use of the conditional (ternary) operator.

#### Array Spread Operator

After taking a look at the object spread operator in this reducer, let's talk about the *array spread operator* which is quite similar conceptually.

The array spread operator allows for an iterable to be expanded into areas that take in more than one argument. Basically, this operator breaks an array into its individual objects, to which these objects are passed in, one by one, into the area where it is called. Let's take a look at an example:

```js
const alphabet = ['a', 'b', 'c', 'd']

const moreLetters = [...alphabet, 'e', 'f']

const lettersAndNumbers = [1, 2, ...alphabet]

const [first, second, ...rest] = moreLetters

console.log(alphabet)
console.log(moreLetters)
console.log(lettersAndNumbers)
console.log(first)
console.log(second)
console.log(rest) 
```

This code outputs the following:

![image-20191230204841434](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191230204841434.png)

The first use of the spread operator in the assignment of `moreLetters` breaks up the `alphabet` array into its individual letters and puts it inside the new array that we are creating. We then add a few more letters to the end of the array.

The use of the array spread operator for `lettersAndNumbers` follows the same concept, only we add two numbers before the use of the spread operator.

Arguably, the most interesting bit of the code is the destructuring of the array and the use of the spread operator in the line:

```js
const [first, second, ...rest] = moreLetters
```

Here, we destructure based on position. The variable specified in index 0 of the variable declaration side of the expression will be assigned the value of index 0 in the array and so on. Then, the variable following the array spread operator will receive the rest of the array. In other words, when using the array spread operator in assignment, anything that has not been destructured will go into the variable prefixed by the array spread operator.

> Note that you should not use the spread operator as the first variable in an attempt to destructure variables at the end of the array. This essentially just copies the array, and you will not be able to destructure the variables at the end.

In our simplified notes application, the array spread operator actually has an application:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
  switch (action.type) {
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

export default noteReducer
```

As our state object is just an array, we can use the array spread operator to create a copy of the original state array in our new state array and just add our new note to the end of the array.

#### Forms with Redux

Now, moving on to the ability to actually add notes via the frontend of our application, let's first create various UI elements that dispatch actions for us:

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'

import noteReducer from './reducers/noteReducer'

const store = createStore(noteReducer)

const generateId = () => Math.round(Number((Math.random() * 10000)))

const App = () => {

  const addNote = (event) => {
    event.preventDefault()
    const textContent = event.target.noteField.value
    store.dispatch({
      type: 'NEW_NOTE',
      data: {
        textContent,
        important: false,
        id: generateId()
      }
    })
    event.target.noteField.value = ''
  }

  const toggleFlagged = (id) => () => {
    store.dispatch({
      type: 'TOGGLE_FLAGGED',
      data: { id }
    })
  }

  return (
    <div>
      <form onSubmit={addNote}>
        <input name="noteField"/> 
        <button type="submit">Create Note</button>
      </form>

      <ul>
        {store.getState().map(note =>
          <li key={note.id}>
            <input 
              name="note" 
              type="checkbox" 
              alt="Toggle Flagged" 
              checked={note.flagged} 
              onClick={toggleFlagged(note.id)}
            />
            {note.textContent} 
          </li>
        )}
        </ul>
    </div>
  )
}

const renderDOM = () => {
    console.log('Rendering application...')
    ReactDOM.render(<App />, document.getElementById('root'))
}

renderDOM()
store.subscribe(renderDOM)
```

In the above block of code, you'll find comments describing most of the major additions to our previous iteration of our app. Here, we create a wrapper function around our `TOGGLE_FLAGGED` action dispatch call that takes in an `id` of a note to be updated. Then, this function is set as the `onClick` handler in the JSX for our checkboxes.

In `toggleFlagged`, we utilize function currying once again to return a function, unique to the `id` passed to it, that will go ahead and toggle the flagged property upon being called via a click. We do this as React (if given a function call) calls the function passed to it once when being rendered, and we want the actual function to be assigned to the `onClick` property rather than whatever the function actually returns. We do not have to utilize function currying for the `addNote` function as we do not have to pass a parameter to it, so we can just pass the function without calling it (via the variable name without parentheses).

We also created a function, `addNote`, that is simply the `onSubmit` handler of our form that we created. It just takes in the data from the note input field and creates a note using that information and a randomly generated `id` created by the helper function `generateId`.

One thing to note, unlike our previous note application without Redux, our form is not controlled, meaning that the internal state of the form fields (our `input` element in this case) are not bound to a piece of state, leaving only the DOM to handle the form data (although we can retrieve the data in our event handlers). The React documentation refers to these as uncontrolled components.

It is generally recommended to use controlled components to implement forms as we can later add dynamic validation of inputs, etc. but this implementation will serve us just fine for now.

### Synchronous Action Creators

In the block of code above, we wrapped the `TOGGLE_FLAGGED` action dispatch call into its own function. These functions are called *action creators*. As React components do not actually need to observe the implementation of actions, including their type, we can just abstract these calls into their own functions and simply call the functions from within our components.

With this, putting our action dispatch calls into action creator functions would look something like this:

**/src/index.js**

```jsx
//...

const createNote = (textContent) => {
  return {
      type: 'NEW_NOTE',
      data: {
        textContent,
        important: false,
        id: generateId()
      }
  }
}

const toggleFlaggedOf = (id) => {
  return {
    type: 'TOGGLE_FLAGGED',
    data: { id }
  }
}

//...
```

We can then call these action creators from within our handler functions in our `App` component:

**/src/index.js**

```jsx
//...

const App = () => {

  const addNote = (event) => {
    event.preventDefault()
    const textContent = event.target.noteField.value
    store.dispatch(createNote(textContent))
    event.target.noteField.value = ''
  }

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id))
  }
  
  //...
```

Here, instead of creating the actions manually, we rely on the action creators to do so, leaving us only having to provide relevant information in our handlers. We then take the action that was returned and dispatch it to our store.

### Fowarding the Store

When organizing our project, we separate out our code into several files and components. With our current implementation, we can only access the store from within **index.js**. In order to support larger projects, we must explore a way to access the store from other modules.

One method of accessing the store in separate modules/components is to pass the store using props. This is the simplest way, but also is not recommended in many instances. For now, we'll use this method to abstract `App` out into its own module/component:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
  switch (action.type) {
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

const generateId = () => Math.round(Number((Math.random() * 10000)))

export const createNote = (textContent) => {
  return {
    type: 'NEW_NOTE',
    data: {
      textContent,
      flagged: false,
      id: generateId()
    }
  }
}

export const toggleFlaggedOf = (id) => {
  return {
    type: 'TOGGLE_FLAGGED',
    data: { id }
  }
}
export default noteReducer
```

First, we moved the action creators to the `noteReducer` module as we may use it in multiple components, and it makes logical sense for these action creators to be with the reducer. We also moved over the `generateId` helper function, as `createNote` relies on it. We then named exported these two functions. Next, let's reflect these changes in the `App` component:

**/src/App.js**

```jsx
import React from 'react'
import { createNote, toggleFlaggedOf } from './reducers/noteReducer'

const App = ({ store }) => {

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id))
  }

  return (
    <div>
      <ul>
        {store.getState().map(note =>
          <li key={note.id}>
            <input 
              name="note" 
              type="checkbox" 
              alt="Toggle Flagged" 
              checked={note.flagged} 
              onClick={toggleFlagged(note.id)}
            />
            {note.textContent} 
          </li>
        )}
        </ul>
    </div>
  )
}

export default App
```

Then, we move the `App` component into its own module from which we take in the store from the props by destructuring `store` from the object. We also have to import `createNote` and `toggleFlaggedOf` from the `noteReducer` module. 

Finally, we're left with a simplified **index.js** file:

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import App from './App'

import noteReducer from './reducers/noteReducer'

const store = createStore(noteReducer)

const renderApp = () => {
  ReactDOM.render(
    <App store={store}/>,
    document.getElementById('root')
  )
}

renderApp()
store.subscribe(renderApp)
```

This is a good start, but we can go even further by moving everything involved with creating a note into its own component:

**/src/components/NewNote.js**

```jsx
import React from 'react'

const NewNote = ({ store }) => {
  
  const addNote = (event) => {
    event.preventDefault()
    const textContent = event.target.noteField.value
    store.dispatch(createNote(textContent))
    event.target.noteField.value = ''
  }
  
  return (
  	<form onSubmit={addNote}>
        <input name="noteField"/> 
        <button type="submit">Create Note</button>
    </form>
  )
}

export default NewNote
```

Here, we moved the function that essentially bridged our React part of our application to the Redux part of our application into a component that renders the new note form. This makes sense as this form is the only element that makes use of the `addNote` function.

For further simplification, let's create a singular note component and a component that encompasses a bunch of notes in a list:

**/src/components/Note.js**

```jsx
import React from 'react'

const Note = ({ note, toggleFlagged }) => {

  return (
    <li>
      <input 
        name="note"
        type="checkbox" 
        alt="Toggle Flagged" 
        checked={note.flagged} 
        onChange={toggleFlagged}
      />
      {note.textContent}
    </li>
  )
}

export default Note
```

Here, we have a component for a singular note. We just take in the `toggleFlagged` `onChange` handler and the actual note content so that we can compose the individual note row. This component will be the child component of the `Notes` component which will compose a list of these `Note` components:

**/src/components/Notes.js**

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import Note from './Note'

const Notes = ({ store }) => {

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id))
  }

  return (
    <ul>
      {store.getState().map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

export default Notes
```

The `Note` component actually takes in the store as a prop and gets the current state of the store to which we map over it with a function. This function renders a `Note` component for each note in an unordered list with us passing in the note, a key, and the actual `toggleFlagged` function.

As `Note` simply returns JSX and does not contain any logic within it (all logic is passed in through props), we refer to it as a *presentational component*.

`Notes` is referred to as a *container component*, not because it contains multiple instances of another component, but rather because it contains some form of application logic and configures presentational components. In this case, it defines some logic surrounding the event handlers of the `Note` component (upon which the `Note` component will use for a `onChange` event).

All of these changes leave us with a very simple `App` component:

```jsx
import React from 'react'
import Notes from './components/Notes'
import NewNote from './components/NewNote'

const App = ({ store }) => {

  return (
    <div>
      <NewNote store={store} />
      <Notes store={store} />
    </div>
  )
}

export default App
```

You'll notice that, even though we receive the `store` prop, we do not actually use it within our `App` component exclusively. Herein lies a downside to forwarding the store to all components, going down the hierarchy. We will explore another approach to accessing the store in just a bit.

### Filtering the Store

Now, like in our previous notes application, let's add the ability to only show notes that have been flagged and vice versa.

First, let's implement the user interface that will call a placeholder handler function:

**/src/App.js**

```jsx
import React from 'react'
import Notes from './components/Notes'
import NewNote from './components/NewNote'

const App = ({ store }) => {

  let showAll = true // placeholder

  const filterNotes = (value) => () => {
    alert(value)
    showAll = !showAll
  }

  return (
    <div>
      <NewNote store={store} />

      <div>
        <button onClick={filterNotes(showAll ? 'FLAGGED' : 'ALL')}>
          Show {showAll ? 'Flagged' : 'All' }
        </button>
      </div>

      <Notes store={store} />
    </div>
  )
}

export default App
```

Here, we create a placeholder `showAll` piece of state (just a variable) so that we can better visualize the logic here. We have a `filterNotes` placeholder function that will be called by the button that we had added.

The button passes in a different string literal to `filterNotes` dependent on the current value of `showAll` using a ternary operator.

With this, we'd like our `filterNotes` handler to trigger an action creator that creates an action that creates a `filter` property upon our store:

```js
{
  notes : [
    //...
  ]
  filter: 'ALL'
}
```

Our state object not has two properties: `notes` which contains an array of objects representing the data of our notes, and `filter` which contains a string describing what kind of nots should be displayed.

### Multiple Reducers

For the sake of organization, we'll define a separate module representing the reducer for the `filter` property. This is logical separation in which each property has its own reducer.

Recall that the `createStore(reducer)` function takes in only one reducer, yet we are now creating multiple reducers. We'll explore the solution to this problem in just a moment by combining our reducers.

First, let's implement the reducer that simply returns a new state that contains only the notes that we want displayed:

**/src/filterReducer.js**

```jsx
const filterReducer = (state = 'ALL', action) => {
  switch (action.type) {
  case 'SET_FILTER':
    return action.filterType
  default:
    return state
  }
}

export const filterNotesBy = filterType => {
  return {
    type: 'SET_FILTER',
    filterType
  }
}

export default filterReducer
```

Note how this reducer seems to only be concerned about its "part" of the state object (the `filter` property, rather than the rest of the state). We'll discuss how this is possible in a moment.

We also have defined an action creator that will create `SET_FILTER` actions.

With this new reducer and action creator defined, let's actually try to use it. Since `createStore(reducer)` only takes in one reducer, we can use the `combineReducer()` function to combine the reducers before calling `createStore(reducer)`:

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, combineReducers } from 'redux'
import App from './App'

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const store = createStore(reducer)

const renderDOM = () => {
  console.log('Rendering application...')
  ReactDOM.render(<App store={store} />, document.getElementById('root'))
}

renderDOM()
store.subscribe(renderDOM)
```

Here, you'll see how we use `combineReducers()` to combine the reducers into one reducer that we can then pass to `createStore(reducer)`. Herein this block of code also explains how these reducers only have to worry about their part/property of the state object.

Within `combineReducers()`, we specify the property relating to the actual property in our state object that we want to manage with its respective reducer as its value. In this case, `noteReducer` will manage the `notes` property of our state object while `filterReducer` manages the `filter` property of our state.

The state that is returned by our reducers registered to the specific properties will replace the value of the property rather than the entire state object. So, if our `filterReducer` returned `'ALL'`, the state object's `filter` property would have the value `'ALL'` rather than the state being replaced with `'ALL'`.

For the following explanation, we'll add a manual dispatch call so that we can observe an interesting behavior:

**/src/index.js**

```jsx
//...

store.dispatch({type: 'SET_FILTER', filterType: 'ALL'})

//...
```

If we replace our `App` component with a fragment so that our app will run and add log statements to both reducers, outputting the action they receive, we'll notice something quite interesting:

![image-20191231202220109](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191231202220109.png)

Our action was dispatched to the store who attempted to match it against both reducers. This is because, when combining our reducers, every single action will get handled by the reducer as a whole (including every single separate reducer that was combined). This usually does not lead to any side effects as we usually only create new versions of the state via certain action types unique to our reducers. The reason for this behavior lies in wanting to run multiple changes to our state in different reducers in response to one action. For example, let's just say, in a chat application, a user was banned from a channel. Upon that same action, we might want to handle it in our chat reducer by filtering out all of the user's messages while also filtering out the user from the member's list in our member's list reducer.

Circling back to our application, it currently does not start as a result of our changes. Let's take a look at the error message and traceback:

![image-20191231203044612](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191231203044612.png)

Our application is unable to find a `map()` method of the object that is returned by `store.getState()`. This line of code no longer works as our state is no longer solely an array. We now want to be able to have an object as our state object to which we can have multiple properties. What would have represented the previous state is now found in the `notes` property of our state object which does contain an array:

**/src/index.js**

```jsx
//...

const Notes = ({ store }) => {

  //...

  return (
    <ul>
      {store.getState().notes.map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

export default Notes
```

Upon accessing the `notes` property of the state object instead of the state object itself and mapping over that object, our application will now start.

Again, looking at our `App` component, we can further componentize our application by extracting the note filter into its own component:

**/src/components/NoteFilter.js**

```jsx
import React from 'react'
import { filterNotesBy } from '../reducers/filterReducer'

const NoteFilter = ({ store }) => {

  const filterNotes = (value) => () => {
    store.dispatch(filterNotesBy(value))
  }

  const showAll = store.getState().filter === 'ALL'

  return (
    <div>
      <button onClick={filterNotes(showAll ? 'FLAGGED' : 'ALL')}>
          Show {showAll ? 'Flagged' : 'All' }
      </button>
    </div>
  )
}

export default NoteFilter
```

Here, we moved the JSX and helper function related to the filtering aspect of our application into its own module. Additionally, we replaced the placeholder helper function with a function that will actually create the appropriate action and dispatch the action. We also set `showAll` to appropriate value, dependent on the value of `store.filter`.

Now, for our application to work, all we have to do is update the `Notes` component to take into account the value of the `filter` property of our state object:

**/src/components/Notes.js**

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import Note from './Note'

const Notes = ({ store }) => {

  const {notes, filter} = store.getState()

  const noteList = () => {
    if (filter === 'FLAGGED') {
      return notes.filter(n => n.flagged)
    } else {
      return notes
    }
  }

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id))
  }

  return (
    <ul>
      {noteList().map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

export default Notes
```

In our `Notes` component, we created a new `noteList()` helper function that will give us an array of notes containing only the notes that we should display, dependent on the value of `filter` which was destructured from the object returned by `store.getState()`. We then call this function in our JSX, to which we map over the array that it returns, creating an instance of our `Note` component for each array element.

### Redux Connect

Recall that we earlier stated that there are multiple ways of accessing the Redux store in our React components. So far, we've only explored one way where we passed the store down via props. This can limit us in some ways and requires us to write unnecessary code as we might have to pass the store down to components that don't even use it.

Fortunately, there is a better way to approach accessing the store. The `connect()` function provided by React Redux connects a React component to a Redux store. Its goal is to pass the connected component with the data it needs from the store and the functions that it needs to dispatch actions to the store.

As with most things in functional programming, it doesn't modify our component, but rather wraps our component, returning a new connected component.

Right now, using the `connect()` function is usually the best way to grab state from our Redux store in our components.

We're now branching into the bridge between React and Redux rather than pure redux. With this, we can install the official React bindings for Redux that give us access to wrapper components that will manage store interaction logic for us automatically. This comes with the added benefit of automatically having complex optimizations baked in for us (our components will only re-render when the data actually changes, etc.):

```bash
npm install react-redux
```

This gives us access to the `connect()` function. In order to use `connect()` within our components, we have to make use of `<Provider />` which makes the Redux store available to any nested components wrapped in `connect()`. Usually, we'll render a `<Provider />` at the top level of our application so that our entire app's component tree has the store available to it:

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, combineReducers } from 'redux'
import { Provider } from 'react-redux'
import App from './App'

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const store = createStore(reducer)


ReactDOM.render(
<Provider store={store}> 
  <App store={store} />
</Provider>,
document.getElementById('root'))

```

Here, we got rid of the `renderDOM()` function and the subscription that took it in as a parameter as we no longer need a listener to manually trigger rerenders when our store changes. Instead, we just call the `render()` method of `ReactDOM` directly on a `Provider` which wraps our `App` component.

Now that our `App` component is wrapped in a `Provider`, every single nested component, including `App`, has access to the `connect()` function. Let's first make our `Notes` component into a connected component:

**/src/components/Notes.js**

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import { connect } from 'react-redux'
import Note from './Note'

const Notes = ({ store }) => {

  const {notes, filter} = store.getState()

  const noteList = () => {
    if (filter === 'FLAGGED') {
      return notes.filter(n => n.flagged)
    } else {
      return notes
    }
  }

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id))
  }

  return (
    <ul>
      {noteList().map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

const ConnectedNotes = connect()(Notes)
export default ConnectedNotes
```

Here, we import `connect` and use it to create a new connected component to which we store in `ConnectedNotes`. We then export `ConnectedNotes` rather than `Note`. We are not quite done yet.

The `connect()` function takes in four different parameters with each one being optional:

- `mapStateToProps`
- `mapDispatchToProps`
- `mergeProps`
- `options`

#### `mapStateToProps`

For now, let's talk about `mapStateToProps`. The `connect()` function takes in `mapStateToProps` as its first parameter. If a `mapStateToProps` function is specified, the wrapper/connected component that is created will subscribe to any Redux store update upon which it'll attempt to merge the results of whatever function we specify into the original component's props. We can pass `null` or `undefined` if we don't want this function to subscribe to store updates. You can find more details about this function in the React Redux documentation.

In our `Notes` component, we have historically accessed the notes through `props.notes` or `notes` (when we had destructured `props`). Without mapping the state to our props, to do the same in our new application, we'd have to fetch the store from the props and get the information directly through `props.store.getState().notes`.

Let's update our component to use props over directly accessing the store:

**/src/components/Notes.js**

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import { connect } from 'react-redux'
import Note from './Note'

const Notes = ({ notes, filter, store }) => {

  const noteList = () => {
    if (filter === 'FLAGGED') {
      return notes.filter(n => n.flagged)
    }
    return notes
  }

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id))
  }

  return (
    <ul>
      {noteList().map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter,
  }
}

const ConnectedNotes = connect(mapStateToProps)(Notes)
export default ConnectedNotes
```

A few things have changed here:

- First, we added a `mapStateToProps` function to which we specify a `state` object where we define the props that we want mapped as the keys and the actual reference to the state as the value.
- Then, we make use of this function in our call to `connect()` where we pass it in as our first parameter. Our Notes component is passed in through the second set of parentheses (passed into the function that is returned by the call with the first set of parentheses).
- Finally, we remove the destructuring and assignment of `notes` and `filter` from a direct call to `getState()` and retrieve these values through destructuring `props` that gets passed into our component.

We still need access to the `store` passed down into our prompts for dispatch calls, but this will change in a moment.

The syntax of having two sets of parentheses after a function call is also worth taking note of. What we're doing here, is calling a function with the parameter(s) in the first set of parentheses and then calling the function that the original function returns with the parameter(s) in the second set of parentheses.

For example:

```jsx
const add = (x) => (y) => {
  return (x + y)
}

const addOne = add(1)

const testOne = addOne(5) // 1 + 5 === 6

const testTwo = add(1)(5) // 1 + 5 === 6
```

Here, with `addOne`, we assign it with a function call to `add` while only providing one set of parentheses. This returns a function that we can later provide the second set of parentheses to which the first value was placed within the function (`x` in the function declaration was replaced with the value `1`). This is simply the practice of currying functions that we've been exposed to in the past, only with variables that are not immediately curried (return/curried functions that take in parameters).

With `testTwo`, our first parentheses creates a function with `x` replaced with `1` just like we did with `addOne`, only this time, we immediately execute the function that was returned with the parameter(s) in the second parameter and store the return value of that function instead.

![IMG_2D180F778260-1](/Users/josephsemrai/Downloads/IMG_2D180F778260-1.jpeg)

The above diagram illustrates how `connect()()` returns a "connected" component that has access to certain parts of the store via its props dependent on the assignments specified in `mapStateToProps()`.

### `mapDispatchToProps`

Looking back to our Notes component, you'll notice that, even though we eliminated the need to directly call the store with regard to retrieving state, we still need to call `dispatch()` upon the `store` object in order to dispatch our actions.

React Redux provides a similar solution for dispatching actions: `mapDispatchToProps`. We can pass an object to the second parameter of the first call to `connect()()` that specifies the action creators that we wish to use in our component to which we then will dispatch the actions. Let's make these changes:

**/src/components/Notes.js**

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import { connect } from 'react-redux'
import Note from './Note'

const Notes = ({ 
  notes,
  filter,
  toggleFlagged 
}) => {

  const noteList = () => {
    if (filter === 'FLAGGED') {
      return notes.filter(n => n.flagged)
    }
    return notes
  }

  return (
    <ul>
      {noteList().map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter
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

Here, we made a few changes to support dispatching from props using `mapDispatchToProps` rather than calling `dispatch()` upon our store directly. Here, we specify the `mapDispatchToProps` object which contains the action creators that we want to be passed to the connected component as props. We don't need to call `dispatch()` manually here, as `connect()()` will create a function that dispatches a given action creator for each one. Within the `mapDispatchToProps` object, we specify the name of the prop that we want for our dispatcher as the property name. The value for that property will be the action creator that is used to create the function that eventually dispatches this action.

With everything handled on the React Redux side, we can just destructure the new `toggleFlagged` prop that we have access to in our component. This prop calls a function that will dispatch the action created by the `toggleFlaggedOf` action creator as specified in our `mapDispatchToProps` object.

Now, instead of having a `toggleFlagged` function that calls `dispatch` directly on the store using the action creator (`store.dispatch(toggleFlagged(note.id))`), we can just call `toggleFlagged(note.id)` to dispatch the action. With this, we no longer need to pass the store into our components.

You might have noticed that the value of `mapDispatchToProps` is not a function unlike `mapStateToProps`. This is the more declarative method of just passing an object as the value of `mapDispatchToProps`. We can provide a function as well and will explore this method in a moment.

#### Side-Effects of Calling Store Directly

We can also go ahead and use `connect()()` to fix a bug that had been introduced. Previously, we would not be able to toggle the filter with 0 elements as our `NoteFilter` component did not rerender when our store's `filter` property changed. As a result, the button would contain text that did not reflect the current state and handler logic that would not affect our state. Using `connect()()` will allow our component to subscribe to the store and rerender when necessary. Let's make the same changes that we did for `Notes` to `NoteFilter`:

**/src/components/NoteFilter.js**

```jsx
import React from 'react' // eslint-disable-line no-unused-vars
import { filterNotesBy } from '../reducers/filterReducer'
import { connect } from 'react-redux'
const NoteFilter = ({ filter, filterNotes }) => {

  const showAll = filter === 'ALL'

  return (
    <div>
      <button onClick={() => filterNotes(showAll ? 'FLAGGED' : 'ALL')}>
          Show {showAll ? 'Flagged' : 'All' }
      </button>
    </div>
  )
}

const mapStateToProps = (state) => {
  return {
    filter: state.filter
  }
}

const mapDispatchToProps = {
  filterNotes: filterNotesBy
}

const ConnectedNoteFilter = connect(
  mapStateToProps,
  mapDispatchToProps
)(NoteFilter)
export default ConnectedNoteFilter
```

#### The Two Forms of `mapDispatchToProps`

The `mapDispatchToProps` parameter of `connect()()` can have either a function or an object as its value:

* Function form: allows us to access `dispatch()` and make the call to it ourselves, giving us more flexibility in customizing its behavior.
* Object form: easier to use as it will automatically convert the action creators specified in the object into functions that call `dispatch()` upon these actions.

We have just explored the object form where:

```jsx
//...

const mapDispatchToProps = {
  filterNotes: filterNotesBy
}

const ConnectedNoteFilter = connect(
  mapStateToProps,
  mapDispatchToProps
)(NoteFilter)
export default ConnectedNoteFilter
```

Allows us to access `filterNotes()` from within our component, to which the action creator, `filterNotesBy`, is automatically dispatched via a call to our prop. 

Alternatively, we can utilize the function form in order specify the implementation of the dispatch call:

```jsx
//...

const mapDispatchToProps = dispatch => {
    return {
        filterNotes: filter => {
            dispatch(filterNotesBy(filter))
        }
    }
}

const ConnectedNoteFilter = connect(
  mapStateToProps,
  mapDispatchToProps
)(NoteFilter)
export default ConnectedNoteFilter
```

Here, we create a function that will eventually take in the `dispatch()` function (provided by `connect()()`) and will return an object containing functions where the keys will be the names of the props passed to the component. This function form achieves the same result as the object form, but gives us some more flexibility with its implementation.

The practicality of this more complex implementation is realized when you need to access the component's props to create a particular action. We can do this through taking in the optional `ownProps` argument.

For example, let's say that we needed to call an action creator `removePostById()` and pass in the component's ID stored in its props. We would do this as follows:

```jsx
const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    removePost: () => {
      dispatch(removePostById(ownProps.postid));
    }
  }
}
```

Here, we specify a new parameter, `ownProps`, to which we can use the argument to access the `postid` prop of our component. We then provide this value to our `removePostById` action creator so that we can update the correct post. Finally, we dispatch the action that is provided to us.

All of this might beg the question. Why not just call the action dispatchers from within the component where we have access to the props as such?

```jsx
const someFunctionalComponent = () => {
    return (
        <button onClick{() => removePost(postid)}</button>}
    )
}

const mapDispatchToProps = (dispatch) => {
    return {
        removePost: (id) => {
            dispatch(removePostById(id))
        }
    }
}
```

Here, instead of using `ownProps`, we simply bind the `postid` prop to the button's handler function within the render function where we have access to the components props.

While this approach does work without having to use `ownProps`, the component's props are bound every single time the component is rerendered. Instead, with the previous solution of using `ownProps`, we only bind the props to the functions specified within our function when the component's props actually change.

With this, our `mapDispatchToProps` function will be called every time our connected component's props are updated since our implementations need to reflect the component's props. This can offer performance improvements over binding these props every single time the component rerenders.

### Component Organization Patterns

Previously, we had discussed how presentational components should only be concerned with the appearance of a component whereas a container component should configure presentational components and be concerned with the logic of operations.

Let's apply this "separation of concerns" to our `Notes` component. Currently, we have a `noteList` function in our component function that returns an array of the notes that we need to display:

**/

```jsx
//...

const Notes = ({ 
  notes,
  filter,
  toggleFlagged 
}) => {

  const noteList = () => {
    if (filter === 'FLAGGED') {
      return notes.filter(n => n.flagged)
    }
    return notes
  }

  return (
    <ul>
      {noteList().map(note =>
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}

//...
```

Instead, we can employ the "separation of concerns" design principle (to a certain extent) by moving the logic that determines what notes will be rendered outside of the function:

```jsx
import React from 'react'
import { toggleFlaggedOf } from '../reducers/noteReducer'
import { connect } from 'react-redux'
import Note from './Note'

const Notes = ({ 
  notes, // Subset of notes returned by the `noteList` selector
  toggleFlagged 
}) => {

  return (
    <ul>
      {notes.map(note => // We map over the `notes` prop now as it will contain only the notes that we want to display
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleFlagged(note.id)}
        />
      )}
    </ul>
  )
}


const noteList = ({ notes, filter }) => { // Destructures `notes` and `filter` from the state object passed to it
  if (filter === 'FLAGGED') {
    return notes.filter(n => n.flagged)
  }
  return notes
}

const mapStateToProps = (state) => {
  return {
    notes: noteList(state) // Call to `noteList` returns only the notes that we want to display dependent on the value of `filter` in our state
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

Read the comments in the code above to get a general idea of the changes that we had made. Here, instead of taking in the entire `notes` piece of state and then calling `noteList` upon the prop to filter out the notes that we don't want to display, we moved the logic for filtering the notes outside of the component. `noteList`, which lies outside of our functional component, now takes in the entire `state` object that will be given to it in our connected component. From this `state` object, it filters or *selects* a subset of the store and returns it. These simple functions that return a subset of data passed to it are called *selectors*.

Within `mapStateToProps()`, we call the `noteList` selector upon the `state` object and pass the resulting object to the `notes` prop. This allows our component to just receive the `notes` prop and map over it without any consideration of logic. All of this culminates in us creating a component that is closer to a *presentational component*.

Recall that a presentational component is not concerned about any of the logic surrounding it (that's left to *container components*), rather, it just takes in values and returns JSX to be *presented*.

### Expanding Upon Presentational and Container Components

Dan Abramov, co-author of Redux and Create React App, states the following regarding presentational components:

"Presentational components:

- Are concerned with *how things look*.
- May contain both presentational and container components** inside, and usually have some DOM markup and styles of their own.
- Often allow containment via *this.props.children*.
- Have no dependencies on the rest of the app, such as Flux actions or stores.
- Don’t specify how the data is loaded or mutated.
- Receive data and callbacks exclusively via props.
- Rarely have their own state (when they do, it’s UI state rather than data).
- Are written as [functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) unless they need state, lifecycle hooks, or performance optimizations."

Our above `Notes` functional component follows most of these guidelines. When we pass this functional component into `connect()()`, it becomes a container component. Dan Abramov also states that container components:

- "Are concerned with *how things work*.
- **May contain both presentational and container components** inside but usually don’t have any DOM markup of their own except for some wrapping divs, and never have any styles.
- Provide the data and behavior to presentational or other container components.
- Call Flux actions and provide these as callbacks to the presentational components.
- Are often stateful, as they tend to serve as data sources.
- Are usually generated using [higher order components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) such as *connect()* from React Redux, *createContainer()* from Relay, or *Container.create()* from Flux Utils, rather than written by hand."

Usually, with separating our components in this manner, we gain a handful of benefits. Dan Abramov also lists the benefits of his separation criteria (listed above) as:

- "Better separation of concerns. You understand your app and your UI better by writing components this way.
- Better reusability. You can use the same presentational component with completely different state sources, and turn those into separate container components that can be further reused.
- Presentational components are essentially your app’s “palette”. You can put them on a single page and let the designer tweak all their variations without touching the app’s logic. You can run screenshot regression tests on that page.
- This forces you to extract “layout components” such as *Sidebar, Page, ContextMenu* and use *this.props.children* instead of duplicating the same markup and layout in several container components."

Splitting up components like this may be a good idea sometimes, but it is not good to force this design upon your project if it does not fit well, especially in the case of using the React Hooks API. React Hooks allow us to "separate" stateful logic from the component without splitting the component up.

Dan Abramov mentions a term, "higher-order component", that we had not discussed previously. A higher-order function is simply a function that takes in a component and returns a new component (similar to how a higher-order function takes in a function). Higher-order components are not part of the React API but instead are an advanced pattern that emerges due to the compositional nature of React.

These higher-order components provide an alternative to using mixins, objects that are used to mix additional functionality into React components, with regard to reusing generic functionality. Mixins were determined to be more trouble than they were worth by Facebook and are being removed from React and its documentation.

This concept of defining generic functionality that can be applied to components using a function (for example, `connect()()` applied the connected functionality that our component needs to talk to our store) also loosely relates to the concept of inheritance in object-oriented programming.

