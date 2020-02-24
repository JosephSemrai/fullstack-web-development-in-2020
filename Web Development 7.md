## Module Bundling and Handling JS Files

It's time to learn about what's going on behind the scenes and uncover a certain degree of abstraction. We had bootstrapped our application by utilizing `create-react-app` which handled all of the configuration for us.

However, it was not always like this, in the past, creating a React app was considered daunting with many tools requiring configuration.

`create-react-app` uses "webpack" to bundle images, assets, scripts, etc. The image provided on the homepage of `webpack.js.org` provides a good overview of what "webpack" does:

![Webpack Image](https://i.imgur.com/Y0ilaHv.png)

> Image retrieved from https://webpack.js.org/ on 02/19/20

### Bundling Modules

Even though ES6 modules have been standard for quite a while now, most browsers cannot handle code that's been split up into modules. This means that if we just tried to serve an application with one initial JavaScript file as an entry point and tried importing other files, our application would simply not work.

To solve this issue, we have *bundlers* that will *bundle* our modules into single files that contain all of the necessary application code. For example, when we run `npm run build`, our bundler (webpack) will end up building the production version of the single JavaScript files. Our bundler will also manage other types of files for us (CSS, images, etc.). Bundling and building our application has previously resulted in the following files:

![Webpack](https://i.imgur.com/NAUdw9f.png)



Essentially, webpack takes all of our import statements, grabs the content from those modules, and dumps it all into "one" file. It continues to do so "recursively" for all imports.

Here, you'll see how our application is reduced into a few files by our bundler. We access these bundled JavaScript files in our **index.html** file to which our browser can now understand how to run our application:

```html
  <link href="/static/css/main.ebd3d852.chunk.css" rel="stylesheet">

  <!-- ... -->

  <script src="/static/js/2.c1f08b50.chunk.js"></script>
  <script src="/static/js/main.46edc656.chunk.js"></script>
```

You'll find the above tags in our **index.html** file to which you can see that our application's CSS and JavaScript files are bundled into singular files where to which we can then reference in our HTML.

### Using Webpack

Let's create our own application using webpack without bootstrapping from `create-react-app`. We'll first create a project with the following project structure:

![Project Structure](https://i.imgur.com/rp5l8N8.png)

We'll setup our **package.json** with the following contents:

**/package.json**

```json
{
  "name": "3_project8_webpack",
  "version": "0.0.1",
  "description": "webpack from scratch",
  "scripts": {},
  "license": "MIT"
}
```

We can then go ahead and install webpack by running:

```bash
npm install --save-dev webpack webpack-cli
```

We can then make use of webpack with some initial configuration in **webpack.config.js**:

**/webpack.config.js**

```js
const path = require('path')

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  }
}
module.exports = config
```

Here, we define the configuration file that webpack will use. It's written in plain JavaScript and uses Node's module syntax (since we're not using a bundler to enable ES6 imports just yet).

The `entry` property of our `config` object simply specifies the entry point of our application. This is the root of our application (like the **index.js** file of our applications) and is where all of our imports will stem from.

The `output` property just describes the `path` of which our bundled assets will be stored as well as the `filename` that we'd like. `path` must be defined as an absolute path to a directory on the machine. We use `path.resolve()` to help us with creating this absolute path as well as the `__dirname` global variable to which Node stores the path of the current directory in.

Now, we just need a way to start the bundling process. We can do this through a npm script:

**/webpack.config.js**

```json
//...

"scripts": {
  "build": "webpack --mode=development"
},

//...
```

Finally, we can start writing our test application in our **/src** directory:

**/src/index.js**

```js
const test = () => {
  console.log('Hello World!')
}
```

After this setup, we can finally bundle and build our 'vanilla' JavaScript application by running `npm run build`:

![webpack output](https://i.imgur.com/PGrotIZ.png)

In the output, we can see how our **index.js** file is picked up by webpack and is bundled into the **main.js** asset. Let's create our first imported module so that we can see how webpack handles modules:

**/src/App.js**

```js
const App = () => {
  return "Hello World!"
}

export default App
```

We'll also make the following changes to our **index.js** file so that we're making use of importing modules:

**/src/index.js**

```js
import App from './App'

const test = () => {
  App()
}

test()
```

Now, upon building our project using `npm run build`, we're given the following output:

![Webpack with Multiple Modules](https://i.imgur.com/HaoLL3K.png)

Here, you can see how webpack picks up on both of our files, **App.js** and **index.js**. The resulting application code from these two source files can be found, again, in **main.js** (as specified in **webpack.config.js**):

**/build/main.js**

```js
//WEBPACK BOOTSTRAP CODE

//...

/***/ "./src/App.js":
/*!********************!*\
  !*** ./src/App.js ***!
  \********************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\nconst App = () => {\r\n  return \"Test\"\r\n}\r\n\r\n/* harmony default export */ __webpack_exports__[\"default\"] = (App);\n\n//# sourceURL=webpack:///./src/App.js?");

/***/ }),

/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _App__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./App */ \"./src/App.js\");\n\r\n\r\nconst test = () => {\r\n  Object(_App__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()\r\n}\r\n\r\ntest()\n\n//# sourceURL=webpack:///./src/index.js?");

/***/ })

/******/ });
```

Here, you can see the result of the individual source files in our bundled code.

### Manually Bundling and Configuring React

Let's go through the process of adding React to our manually configured webpack application.

First, we have to install `react` and `react-dom`:

```bash
npm i react react-dom
```

Let's now try to edit our source files to create a React application:

**/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

ReactDOM.render(<App />, document.getElementById('root'))
```

This is extremely similar to how our previous React applications were created with regard to their entry points.

**/src/App.js**

```jsx
import React from 'react'

const App = () => (
  <div>Hello World!</div>
)

export default App
```

Let's also add a **index.html** file in our **/build** directory (since it doesn't need to be bundled) so that we can actually serve our bundled JavaScript file:

**/build/index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Hello World</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="text/javascript" src="./main.js"></script>
  </body>
</html>
```

If we try to bundle our application, we'll run into the following error:

![webpack error](https://i.imgur.com/hwRAFqm.png)

This is simply because we make use of JSX in the definition of our `App` module. Recall that, although React offers a pure JS way of rendering components, we elect to use JSX for rendering our views with its XML-like format:

```jsx
const App = () => (
  <div>Hello World!</div>
)
```

Using JSX leads to an error as webpack, by default, only can bundle "vanilla" JavaScript. We can fix this by following the suggestion in the error message.

#### Webpack Loaders

webpack enables the use of loaders to allow us to bundle resources beyond "vanilla" JavaScript. We can even write our own loaders using Node.js. These loaders simply process our files before they are bundled, allowing webpack to proceed with bundling.

We can add a loader that transforms our JSX into "vanilla" JavaScript by adding the following to our webpack configuration:

**/webpack.config.js**

```js
const path = require('path')

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-react'],
        },
      },
    ],
  }
}
module.exports = config
```

Here, we define a loader under the `module` property in its `rules` array.

In order to define a loader, we need to provider a `test` property, a `loader` property, and a `options` property:

* The `test` property tells the file endings that our loader should target. It takes in a regular expression and our files must match this condition.
* The `loader` property simply allows us to specify a loader to be used for the processing of files that match our `test`.
* The `options` property just allows us to configure our loader. Here we just set the our `babel-loader` loader's preset to the default preset for React.

We'll also need to install the loader and its presets as referenced by our configuration:

```bash
npm i --save-dev @babel/core babel-loader @babel/preset-react
```

We will now be able to build our application successfully:

![Successful Application Build](https://i.imgur.com/lfhlil3.png)

Our JSX is now transformed into regular JavaScript to which it is then bundled with the rest of our application source.

### Polyfills

As the default React babel configuration targets ES2015, we'll need a polyfill (**code that implements a feature that hasn't yet been supported for some targets**) for async/await syntax (or promises depending on the target).

This could become a problem if we use `async/await` in our applications, as it may completely break our applications in some browsers.

Let's install `core-js` in order to remedy this:

```bash
npm i core-js
```

We can then adjust the `entry ` property of our webpack configuration in order to polyfill using `core-js`:

```js
const path = require('path')

const config = {
  entry: ['core-js', './src/index.js'],
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-react'],
        },
      },
    ],
  }
}
module.exports = config
```

Here, we changed the value of `entry` to an array with `'core-js'` as its first value.

This is not a devDependency (`--save-dev`) as we need it to run before our source code for the actual application, even when its not in development.

If we serve our **index.html** file with a HTTP server and open it in our browser, we'll see the following result as expected:

![Expected Application](https://i.imgur.com/rW5iTus.png)

When we "convert" this JSX code into "vanilla" JavaScript code, we are *transpiling* the code. **Transpilers are tools that read source code written in one programming language (or superset/subset) and produce the equivalent in another language.**

In our configuration, we utilize `babel` which takes care of transpiling our code. Now, most browsers do not support the ES6 and ES7 features that we make use of (we did not use polyfills for every single feature). We can solve this by using a preset that will transpile our code written in newer JavaScript standards to code that operates in older standards. This allows us to use newer features without losing any compatibility.

In order to do this, we'll simply add a preset that will allow us to use the latest JavaScript without micromanaging polyfills and syntax transforms; this also comes with the benefit of making our JavaScript bundles smaller in some cases.

Install the preset by running the following command:

```bash
npm i --save-dev @babel/preset-env
```

As this package is only used by `babel` when transpiling our application, we only need to install it as a devDependency.

Now, we can just add it to our `presets` property in our loader options for `babel-loader`:

```js
const path = require('path')

const config = {
  entry: ['core-js', './src/index.js'],
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react'], // New Preset
        },
      },
    ],
  }
}
module.exports = config
```

Now, our application will successfully transform our code into older JavaScript standards. For example, since arrow functions are not in the ES5 standard, they'll be converted to regular functions.

### Other Polyfills

If we want to target Internet Explorer, we'll also need to add a polyfill for Promise support. Internet Explorer understands the syntax and keywords associated with Promises, but simply fails to implement their functionality. For this, we'll add another polyfill from the massive library of polyfills in order to fill in support for Internet Explorer (`promise-polyfill`).

Let's install the package with the following command:

```bash
npm i promise-polyfill
```

This polyfill is not added by webpack, but rather we just inject it into our application by importing the package:

**/src/App.js**

```jsx
//...

import 'promise-polyfill/src/polyfill'

//...
```

Here, if our window object (global) does not have a `Promise` property,  this `import` will add a global Promise object (including support for Node which does not have a `window` object).

#### CSS Loader

If we want to be able to use CSS and load it from our source/use it in our components, we'll need to add a loader for CSS (depending on how we want to implement things). We'll be looking to define our CSS in our **main.js** file.

We can do this by installing `css-loader` which will load our CSS files by interpreting our `import`s and `style-loader` which will inject our CSS into the DOM:

```bash
npm i --save-dev css-loader style-loader
```

We can set the target for these loaders to files ending in **.css** as follows:

```css
const path = require('path')

const config = {
  entry: ['core-js', './src/index.js'],
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react'],
        },
      },
      {
        test: /\.css$/,
        loaders: ['style-loader', 'css-loader'],
      }
    ],
  }
}
module.exports = config
```

Now, we can use CSS in our application by creating a CSS file, importing it in a JavaScript module, and referencing the CSS that has been injected:

**/src/index.css**

```css
.test {
  color: red;
}
```

**/src/App.js**

```jsx
import React from 'react'
import './index.css'

const App = () => (
  <div className="test">Hello World!</div>
)

export default App
```

This yields the following result when served by a HTTP server:

![Styles with Webpack](https://i.imgur.com/ZXbD47w.png)

Hopefully this clears up any confusion regarding how our app actually receives its styles (we import the styles to which they are made available in the DOM).

### Development Tools for webpack

Previously, when working with `create-react-app`, our app would automatically recompile whenever its code changed. This is not the case in our current configuration.

There are three options with webpack that will automatically compile our code:

* webpack Watch Mode
* `webpack-dev-server`
* `webpack-dev-middleware`

For our purposes, as recommended by the official docs, we'll be exploring `webpack-dev-server`.

Let's install it by running the following command:

```bash
npm i --save-dev webpack-dev-server
```

Now, we can create a script that resembles `create-react-app` in that it will "start" our application with `webpack-dev-server` so that our changes trigger automatic recompiles:

**/package.json**

```json
{
  "name": "3_project8_webpack",
  "version": "0.0.1",
  "description": "webpack from scratch",
  "scripts": {
    "build": "webpack --mode=development",
    "start": "webpack-dev-server --mode=development"
  },
  //...
```

We'll also configure the server to be served on the port, `3000`, and also enable gzip compression for everything being served (optimizes file sizes):

**/webpack.config.js**

```js
const path = require('path')

const config = {
  entry: ['core-js', './src/index.js'],
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  },
  devServer: {
    contentBase: path.resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
  },
  //...
```

Upon running `npm start`, our application will now be re-compiled upon any changes to its source. It is also served to us:

![Preview](https://i.imgur.com/4ZUX5LV.png)

It's important to note that, even though our development process seems similar to using `create-react-app`, error messages will not be displayed in our terminal, so we must pay attention to our console.

#### Errors with Bundlers and Source Maps

The nature of how our application's source code is transformed into bundled and transpiled code that ultimately ends up being executed on the browser poses an issue for locating errors. This is because the bundled code is drastically different from our source code, and when an error occurs, we'll be pointed to its location in our bundled code, but we would have trouble finding that instance in our source code.

It also poses a challenge in that all of our code is now found in one file. Our stack trace upon an error will only point to our bundled, singular file, which isn't exactly helpful when we want to know the source file that the error came from.

For example, let's intentionally cause an error in our application:

**/src/App.js**

```jsx
import React, { useState } from 'react'
import './index.css'

const App = () => {
  const [testArray, setTestArray] = useState() // Defined with no value

  const handleArrayAdd = () => {
    setTestArray(testArray.concat('Added!')) // Tries to concat 'Added!' to an object that isn't an array
  }

  return (
    <>
      {testArray}
      <button onClick={handleArrayAdd} className="test">Add value to array</button>
    </>
  )
}

export default App
```

Here, we try to `concat()` a value to `testArray` which has not been initialized with any value. This will cause an error as `concat()` is not a property of `null` (as it is an array method).

Now, in our application, upon pressing the button and triggering the `handleUpdate`, we're served with the following errors in our developer console:

![errors](https://i.imgur.com/pLcdPQs.png)

If we tried clicking "App.js:25" which should direct us to the statement that caused the error, we'll find code that does not match our source:

![source not matched](https://i.imgur.com/Hc6LSo2.png)

We can see the relation to our source code here as our application is quite small, but this poses a large issue in more complex projects.

Luckily, we can resolve this through using source maps. Source maps consist of a bunch of information that can be used to map compressed/transpiled/bundled JavaScript files to its original source. We can instruct webapck to generate these source maps for us, so that we can more effectively debug our application.

This culminates in the ability to map errors that occur in execution of the bundled code to the corresponding line(s) of the source.

It's as simple as adding the following `devtool` property to our webpack config:

**/webpack.config.js**

```js
const path = require('path')

const config = {
  entry: ['core-js', './src/index.js'],
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  },
  devServer: {
    contentBase: path.resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
  },
  devtool: 'source-map',
  //...
```

We'll need to restart webpack for these changes to take effect (as it requires some further configuration for webpack to watch for changes made to itself and restart automatically).

With this, if we cause the same error, upon viewing the location of the error, we'll be given the source file with the proper location:

![Source mapped](https://i.imgur.com/sXiGAMb.png)

We're also told that this file was source mapped from **main.js** (the bundle) in the bottom right of our developer tools.

This file is served from webpack at the address `webpack:///./src/App.js`. This also allows us to use the Chrome Debugger with our source code without issue.

### Large Bundle Sizes + Minification

With larger projects, it may become important to monitor our package size. *BundlePhobia* (bundlephobia.com) is a great tool for monitoring the cost of adding a package to our application.

We can also upload our **package.json** file to BundlePhobia to get some information on the cost of each package.

Upon viewing a package, we can view the *minified* size of the package (+ gzipped) as well the estimated times it would take to download these packages:

![Package](https://i.imgur.com/WTLXuWu.png)

You may not have heard of the term, *minification*, just yet. It simply refers to the compression of our files, optimizing the formatting by removing whitespace, shortening variable names, replacing verbose functions with more concise functions, etc.

You can conceptually think of the minification process as transpiling our JavaScript code into a more "space-efficient" version of JavaScript.

Enabling minification with webpack does not require much configuration, we just have to bundle the code in production mode:
**package.json**

```json
{
  "name": "3_project8_webpack",
  "version": "0.0.1",
  "description": "webpack from scratch",
  "scripts": {
    "build": "webpack --mode=production", // Builds with production mode
    "start": "webpack-dev-server --mode=development"
  },
    
  //...
```

This will minifiy our application and lead to a drastically reduced bundle size when running `npm run build`:

![minified](https://i.imgur.com/ryTRlde.png)

The resulting bundle looks like this internally:

```js
!function(t){var e={};function n(r){if(e[r])return e[r].exports;var o=e[r]={i:r,l:!1,exports:{}};
//...
```

This is not developer friendly, but is much more efficient with regard to space.

#### Manual Webpack Configuration with `create-react-app`

As we mentioned earlier, `create-react-app` uses webpack for its bundling. Its configuration is abstracted behind-the-scenes and is configured for us. If we wish to change the default configuration, we must *eject* from the tool. This essentially makes `create-react-app` a boilerplate generator to which we are given all of the configuration files needed for the project. We are essentially on our own after ejecting.

We can eject our project by running `npm run eject`. This will copy all of the configuration files and its dependencies right into our project directory, giving us full control over them. React Scripts commands will still work, but they will now point to scripts found within our directory. There is a lot to the default configuration, and it can be quite hard to grasp. Thus, it may be a good idea, if you're planning on ejecting in the future, to just write your own webpack configuration from the start.

As we discuss and begin to fill in the conceptual gaps that we have with regard to React, the topic of class-based components comes up as a gap in our knowledge.

## Class-Based Components

React offers two ways to define components: the functional component or the class-based component. We've worked with functional components throughout this course. However, there was a time (before React 16.8) where our only option was to use class components since hooks were not introduced just yet.

As the name suggests, class-based components take advantage of JavaScript's class syntax (which in itself is just an implementation on top of prototypal inheritance but are not exactly the same thing).

The React documentation suggests that we use hooks for all of our components (all new components that we write), but there are a few cases where we cannot use hooks/functional components since there are no hook equivalents for the uncommon `getSnapshotBeforeUpdate` and `componentDidCatch` lifecycles (more on what lifecycles are in a moment).

It is quite important to know how to navigate, create, and use class components, however, as there may be packages, examples, etc. that make use of class components that we'll have to use.

We'll create a new project using class components to demonstrate some of the differences. We'll use the webpack configuration from earlier to bundle our project.

A barebones class component looks like this:

**/src/App.js**

```jsx
import React from 'react'

class App extends React.Component {
  constructor(props) {
    super(props)
  }

  render() {
    return (
      <div>
        <h1>Note App</h1>
      </div>
    )
  }
}

export default App
```

Here, you'll see that `App` is no longer a constant that references a function which returns JSX. Instead, we have a class named `App` containing a constructor and a `render()` method to which this method returns the JSX that we wish to be rendered.

The constructor for a class component is called before the component is ever mounted and should call `super(props)` before any other statement in `React.Component` subclasses (meaning the class `extends React.Component`). If we do not call `super(props)`, the `props` object that our class may depend on will be undefined.

React class-based component constructors are used to initialize local state through an object assigned to `this.state` (`state` being bound to the class via `this`). 

They are also used to bind event handler methods to an instance of the class.

For example,

```jsx
constructor(props) {
  super(props);
  this.handleClick = this.handleClick.bind(this);
}
```

This will bind `this` in the `handleClick()` method to the actual instance of the class so that we can update its state:

```jsx
handleClick() {
  this.setState(state => ({
    notes: ['first note']
  }))
}
```

Without binding `this` in the constructor, we would not be able to call `setState()` on the class as `this` would not refer to the class but rather the method that called it, leading to bugs.

You can start to identify how state is managed in class components using `setState()` but we will explain it in more detail shortly.

Let's initialize our state with a `notes` array and display the `notes`:

**/src/App.js**

```jsx
import React from 'react'
import 'promise-polyfill/src/polyfill'

class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      notes: []
    }
  }

  render() {
    if (this.state.notes.length === 0 ) {
      return <p>There are no notes</p>
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>
          {this.state.notes.map(n => n.content)}
        </div>
        <button>Remove Notes</button>
      </div>
    )
  }
}

export default App
```

Notice how we keep our `notes` piece of state as a property of the state object (`this.state`). `this.state.notes` is like the `notes` constant given to us by `const [notes, setNotes] = useState([])`. Within our component, we must also refer to our pieces of state as properties of `this.state`.

Here, we simply return a paragraph containing "There are no notes" if the length of our `notes` piece of state is 0. If this condition is not satisfied (meaning that there are notes), we simply return the application, rendering a heading, the notes, and a button. This is all done in the `render()` method of the class.

This yields the following result:

![no notes](https://i.imgur.com/0YfB20h.png)

This is rendered as we do not have any notes in our state. Let's go about fetching some notes to demonstrate this component.

Previously, we had used the `useEffect()` hook in order to perform side effects in our function components. Among these side effects were network requests to fetch data. We can achieve the same result with class components through the use of *lifecycle methods*.

The lifecycle of a React class component refers to three phases: mounting, updating, and unmounting. Mounting a component simply refers to inserting elements into the DOM.

When mounting a component, React calls the following methods in order:

1. `constructor(props)`
2. `getDerivedStateFromProps()` (allows us to derive the state depending on the props passed to it since **we should not refer to `props` from within the constructor**)
3. `render()` (as discussed previously, outputs JSX to be put into the DOM)
4. `componentDidMount()` (**method that is called after a component has been rendered**)

Based on the definition of `componentDidMount()` this method sounds the most like the `useEffect()` hook.

There are plenty of other lifecycle methods that are called when updating and unmounting, but we shouldn't have to deal with those in most cases, to which we can just explore the documentation when we do.

Since we had not discussed it yet, `getDerivedStateFromProps()` allows us to set our state based on the props as follows:

```jsx
class Test extends React.Component {
  constructor(props) {
    super(props);
    this.state = {someValue: "test"};
  }
  static getDerivedStateFromProps(props, state) {
    return {someValue: props.valuePassedInFromProps };
  }
  render() {
    return (
      <h1>This value was passed in from `props` {this.state.someValue}</h1>
    );
  }
}
```

Here, we initialize the `someValue` piece of state in our `constructor(props)` and set it to the value passed into our `props` within `getDerivedStateFromProps(props, state)`. 

Circling back to fetching our data, the `componentDidMount()` method runs after our component output has been rendered to the DOM for the first time.

We'll install `axios` to make our request:

```bash
npm i axios
```

Here, we can make our `axios` request for the notes and update our state appropriately:

**/src/App.js**

```jsx
import React from 'react'
import 'promise-polyfill/src/polyfill'
import axios from 'axios'

class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      notes: []
    }
  }

  componentDidMount() {
    axios.get('http://localhost:3001/notes').then(res => {
      this.setState({ notes: res.data })
    })
  }

  render() {
    if (this.state.notes.length === 0 ) {
      return <p>There are no notes</p>
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>
          {this.state.notes.map(n => n.content)}
        </div>
        <button>Remove Notes</button>
      </div>
    )
  }
}

export default App
```

We'll also use `json-server` to serve the following file to simulate our backend:

**/db.json**

```json
{
  "notes": [
    {
      "content": "This was the first note",
      "id": "1223123"
    },
    {
      "content": "This is a note!",
      "id": "342343242"
    }
  ]
}
```

With a global installation of `json-server`, we'll run the following command to serve our notes:

```bash
json-server --watch db.json --port 3001
```

This yields the following result:

![Result](https://i.imgur.com/pzh8Bqj.png)

There is a obvious lack of styling or UX design, but our component is demonstrating basic state management and the use of lifecycle methods.

Finally, we'll implement the "Remove Notes" button which should give us experience with event handlers in class components:

**/src/App.js**

```jsx
//...

class App extends React.Component {
  //...

  handleClick() {
    console.log(this)
    this.setState({
      notes: []
    })
  }

  render() {
    if (this.state.notes.length === 0 ) {
      return <p>There are no notes</p>
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>
          {this.state.notes.map(n => n.content)}
        </div>
        <button onClick={this.handleClick}>Remove Notes</button>
      </div>
    )
  }
}

export default App
```

Here, we have intentionally introduced a bug. Recall that we talked about binding `this` to the actual class since `this` will refer to the method that had called it (in this case, the `onClick` event or `undefined`). As `this` does not refer to the class, we do not have access to its `setState()` method, leading to a thrown error upon clicking the button:

![error](https://i.imgur.com/o7erbKY.png)

Here, we are shown that the value of `this` is `undefined` and that we cannot access the `setState()` method of the `undefined` value of `this`.

We can fix this by binding the value of `this` to the class in its constructor:

**/src/App.js**

```jsx
import React from 'react'
import 'promise-polyfill/src/polyfill'
import axios from 'axios'

class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      notes: []
    }

    this.handleClick = this.handleClick.bind(this) // Binds this
  }

//...
```

Upon doing this, our application behaves as expected:

![expected](https://i.imgur.com/dYuKzhi.png)

The console displays the `App` class as the value of `this`:

![console](https://i.imgur.com/D3vNb9t.png)

There are other ways of binding event handlers, but this method should serve us quite well for now and is quite simple/effective for what it is.

Our final application's `App` component looks like this as a class component:

**/src/App.js**

```jsx
import React from 'react'
import 'promise-polyfill/src/polyfill'
import axios from 'axios'

class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      notes: []
    }

    this.handleClick = this.handleClick.bind(this)
  }

  componentDidMount() {
    axios.get('http://localhost:3001/notes').then(res => {
      this.setState({ notes: res.data })
    })
  }

  handleClick() {
    this.setState({
      notes: []
    })
  }

  render() {
    if (this.state.notes.length === 0 ) {
      return <p>There are no notes</p>
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>
          {this.state.notes.map(n => n.content)}
        </div>
        <button onClick={this.handleClick}>Remove Notes</button>
      </div>
    )
  }
}

export default App
```

The "same" component written as a functional component looks like the following:

```jsx
import React, { useState, useEffect} from 'react'
import 'promise-polyfill/src/polyfill'
import axios from 'axios'

const App = () => {
  const [notes, setNotes] = useState([])

  useEffect(() =>{
    axios.get('http://localhost:3001/notes').then(res => {
      setNotes(res.data)
    })
  }, [])

  const handleClick = () => {
    setNotes([])
  }

  if (notes.length === 0) {
    return <p>There are no notes</p>
  }
  
  return (
    <div>
      <h1>Notes App</h1>
      <div>
        {notes.map(n => n.content)}
      </div>
      <button onClick={handleClick}>Remove Notes</button>
    </div>
  )
}

export default App
```

Here, you can somewhat see the differences between functional and class components. The advantages of functional components become much more realized with larger components and projects.

`useEffect()` often becomes much more effective for dealing with side effects rather than the many lifecycle methods found in class components. We also do not have to deal with referencing `this` or being restricted to one object as our component's state.

One potential use for class components that still has not been resolved is the use of class components as *error boundaries*, these components wrap other components and display a fallback UI upon catching an error in its child components.

Error boundaries depend on the `componentDidCatch(error, info)` lifecycle method, to which these error boundaries can only be class components (although they may still wrap functional components).

Functional components can try to display fallback UIs and catch errors using the `try`/`catch` syntax, but this only works for imperative JavaScript code:

```jsx
try {
  renderComponent()
} catch (error) {
  displayFallbackUI()
}
```

rather than for declarative code as React components should be:

```jsx
<ErrorBoundary>
	<SomeComponent />
</ErrorBoundary>
```

Error boundaries also behave as we would expect, even if an error occurs far back in a tree where a `try`/`catch` block might not catch it as all of the children aren't necessarily updating together.

Nonetheless, while it is important to be able to work with class components, you should continue to write functional components unless there is a need to work with class components.

## React UI Libraries and Further Styling

Previously, we talked about using traditional CSS files and inline-styles to style our application. There are plenty of other options to styling your applications both in and outside of the React world, though. We'll be looking at two popular options, one involving an easier/better way to define our own styles in React and another involving pre-made styles defined for us.

### UI Libraries

An extremely popular option, with the component and compositional nature of React, for making styled applications in React is to use a "UI framework/library". These frameworks often provide tons of styling options, defaults for styling, and functionality for things like menus, showing modals, etc.

Some UI frameworks have been "translated" for use with React in that their "components" (not actual React components, but rather a set of CSS, HTML, and JavaScript in order to make something that resembles a styled component with some functionality) are converted into actual React components. An example of this would be Bootstrap being made for use in React with `react-bootstrap` or `reactstrap`.

As of writing, the two biggest UI frameworks for React, by a large margin, are Ant Design and Material UI with Ant Design leading the pack by a slight margin.

We'll now start adding a few Material UI components to our application to demonstrate how these components come with styling and functionality built right into them. The reason that we are using Material UI is that it provides more flexibility with regard to styling options. Most concepts of component usage and layout should be the same in Ant Design, allowing developers to use the more appropriate framework for the project.

### Material UI

Let's start by installing our needed packages to our application that we had just worked on implementing React Router with:

```bash
npm install @material-ui/core
```

As these components make use of the Roboto font, we'll need to add a `link` element to our **index.html** file in our public server that is served to the web browser so the web browser knows to download the font:

**/Frontend/public/index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <meta
      name="viewport"
      content="minimum-scale=1, initial-scale=1, width=device-width"
    />
    <link rel="apple-touch-icon" href="logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />  <!-- Font Link -->

<!-- ... -->
```

We also added a `meta` tag to our `head` element which just specified certain behaviors ensuring proper rendering and touch zooming as recommended in the Material UI documentation.

After installing this package, we're given access to a ton of components spanning from components that assist us with responsive layouts to components that give child components animated transitions.

#### Setting up Theming with Material UI

Material UI Components come with a default theme, but if we wish to customize any of the components using the built-in theme system, we'll have to wrap our application with the `ThemeProvider` component in order to inject a theme into it. `ThemeProvider` relies on the React context feature, so it needs to be a parent of all of the components that we are attempting to customize.

With this, we're able to specify the color palette, typography, spacing, responsive breakpoints, etc. of our components through changing theme configuration variables.

To initially explore the concept of theming, we'll add a simple Material UI checkbox component to our application:

```jsx
//...

import { Checkbox } from '@material-ui/core'

//...

const App = (props) => {
  //...
  return (
    <div>
      <Checkbox />
    {/*...*/}
  )
}
```

This yields a checkbox that looks like this when checked:

![Checkbox](https://i.imgur.com/k13euzX.png)

This looks much better than the default checkbox rendered by our browser. Let's now use Material UI's theming system to provide some of our own styling options to the component:

**/Frontend/src/theme.js**

```jsx
import { createMuiTheme } from '@material-ui/core'
import { orange } from '@material-ui/core/colors'

const baseTheme = {
  status: {
    danger: orange[500],
  }
};

const theme = createMuiTheme(baseTheme)

export default theme
```

Here, we leverages `createMuiTheme` in order to create a theme object that we'll pass to a theme provider which will apply our styles specified in `baseTheme` to our application.

We'll then use `ThemeProvider` to inject our `theme` object into our application:

**/Frontend/src/index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'
import { ThemeProvider } from '@material-ui/core'
import theme from './theme'
import './index.css'

ReactDOM.render(
  <ThemeProvider theme={theme}>
    <App />
  </ThemeProvider>
  , document.getElementById('root'))

```

With regard to providing styles and theme information within our components, there are three APIs that we can use to generate and apply styles, but they all share the same underlying object, so we'll just elect to use the Hook API for now:

```jsx
//...
import { Checkbox, makeStyles } from '@material-ui/core'

const useStyles = makeStyles(theme => ({
  checked: {
    color: theme.status.danger,
    '&$checked': {
      color: theme.status.danger,
    },
  }
}));

//...

const App = (props) => {
  const classes = useStyles()
  //...
  return (
    <div>
      <Checkbox
        className={classes.checked}
      />
    {/*...*/}
  )
}
```

Here, we use `makeStyles()` in order to access our theme object and generate a hook to which we can call within our component to get `classes`. From this, we can access the CSS properties stored in the properties of the object returned by the function passed to `makeStyles()`.  We access the class through providing the reference to `className`.

This yields an orange checkbox that looks like this:

![Orange Checkbox](https://i.imgur.com/WXPqH7z.png)

There are plenty of other features with regard to Material UI theming that far outscope this book. If you proceed with using Material UI in your applications, it is recommended that you look at the documentation and learn some more about the useful features it provides.

It is suggested to use Material UI's styling solution when using Material UI as it is generally very fast, comes with most of the advantages of styled-components, and comes with no bundle size increase. You can use Material UI's styling solution even if you aren't using Material UI components.

Now that we have surveyed some of the theming possibilities brought by Material UI, let's explore some of the components that this framework gives us.

With regard to layout, the most basic layout element that we're provided is the `Container` component which will center our content horizontally and apply a width to our content to "contain" it:

**/Frontend/src/App.js**

```jsx
//...

import { Checkbox, makeStyles, Container } from '@material-ui/core'

//...

const App = (props) => {
  //...
    
  return (
   	<Container maxWidth="md">
     {/*...*/}
    </Container>
```

Here, we wrapped our entire app in a `Container` component which gives us the following result:

![Example](https://i.imgur.com/qCH6vax.png)

Our app now is "contained" and fits to a medium `maxWidth`.

For more complex grid layouts, Material UI provides the grid component, but there is no need for it in our application, so we will continue on without it. If you choose to continue using Material UI, it is highly recommended to browse through the component examples and their API documentation to take advantage of the thousands of features provided by the framework.

To keep things simple, instead of taking advantage of a list or such, since we already have the logic for creating lists, we'll just use Material UI's `Card` component to redesign our `Note` component:

**/Frontend/src/components/Note.js**

```jsx
import React from 'react'
import { Link } from 'react-router-dom'
import { Card, CardContent, Checkbox, Typography, makeStyles } from '@material-ui/core'

const useStyles = makeStyles({
  root: {
    marginBottom: 10,
  }
})

const Note = ({ note, toggleFlagged }) => {
  const classes = useStyles()

  if (!note) {
    note = {
      flagged: false,
      content: 'Note content not found'
    }
  }
  return (
    <li>
      <Card className={classes.root}>
        <CardContent>
          <Checkbox checked={note.flagged} onChange={toggleFlagged} />
          <Typography component={Link} to={`/notes/${note.id}`} color="primary">
            {note.content}
          </Typography>
        </CardContent>
      </Card>
    </li>
  )
}

export default Note
```

Here, there are a few interesting things to note. We've replaced the references to our original CSS stylesheet and instead are using Material UI components for the design of our `Note` component. You'll notice how we used the `makeStyles()` hook generator in order to give us a class that will add a bottom margin to the card that makes up our note component.

You'll also notice the introduction of a `Typography` component that acts as a wrapper for text when using Material UI. It'll give us some pre-defined styles as well as help us keep to consistent type sizes that work well together, colors, etc. Within the typography component, we still retain our functionality with our `Link` component that allowed us to view an individual note by specifying the link component as the root node to which we can still pass the `to` prop to.

This is what our app looks like in this state:

![App Preview](https://i.imgur.com/tm3VVlp.png)



The application is not pretty by any means, but it does show how Material UI can quickly give us a presentable UI to which it can be worked on to provide beautiful experiences.

Let's go on to replace the tabs at the top of our application, making use of the `Tabs` component given to us by Material UI:

**/Frontend/src/App.js**

```jsx
//...

import { Checkbox, makeStyles, Container, Tabs, Tab } from '@material-ui/core'

//...

const App = (props) => {
  //...
    
  return (
    <Container maxWidth="md">
      <Checkbox
        className={classes.checked}
      />
      <h1>Notes</h1>
      <Router>
        <div>
          <div>
          <Tabs aria-label="simple tabs example">
            <Tab label="Home" component={Link} to="/home" />
            <Tab label="Notes" component={Link} to="/notes" />
            <Tab label="Login" component={Link} to="/login" />
            {user === null ?
              null :
              <Tab label="Create" component={Link} to="/create" />}
          </Tabs>
     {/*...*/}
    </Container>
```

Here, you'll see how we used the same method of setting our `Link` components as the root component in order to retain the functionality provided to us by `react-router-dom`. Our `Tabs` component here is not fully implemented as it also relies on a `value` prop that tells it what page we're on. We could implement this using `react-router-dom` and inform it of our application's whereabouts so it can underline the active tab, but due to the nature of our application with regard to the `Route` components being placed at the same level as our tabs, we cannot easily access our location.

These changes result in the following appearance for our application:

![Preview](https://i.imgur.com/WQ8Y5Z1.png)

We'll move our attention over to the login form that we haven't touched yet. Currently, signing up with an email looks like this:

![Email Signup](https://i.imgur.com/PfatYQ6.png)

Here, we'll take advantage of some of Material UI's input elements:

```jsx
import React from 'react'
import PropTypes from 'proptypes'
import { Input, Button, Typography } from '@material-ui/core'

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
        <Typography>
          Username
        </Typography>  
        <Input
          type="text"
          value={username}
          name="Username"
          onChange={handleUsernameChange}
        />
      </div>
      <div>
        <Typography>
          Password
        </Typography> 
        <Input
          type="password"
          value={password}
          name="Password"
          onChange={handlePasswordChange}
        />
      </div>
      <Button 
        variant="contained"
        type="submit"
        color="primary"
      >
        Login
      </Button>
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

Here, we just replaced our `input` elements with `Input` elements from Material UI. We also wrapped our text in `Typography` components and replaced our standard `button` with a `Button` component from Material UI with the extra props `variant` and `color`.

Our result is much better looking than before (but not nearly good enough for a production application):

![Better example](https://i.imgur.com/MQ2eeUw.png)

In a production application, we would have to take much more consideration with regard to the color palette of our application, the spacing and distribution of components/elements, etc. With this, we have explored the basics of using a UI framework. There are plenty of other components that you can take advantage of in your future applications, but we have covered just about every component that we would be able to use in our application here. 

With this, we'll explore one other method of styling React components. This method is often considered the best way to style components (to which the many advantages of styled components are inspired and found in Material UI's theming system).

#### Styled Components

Styled components allow us to create reusable components that we can use throughout our application with styling information baked in. For example, if we wanted to create a styled regular `button` element, we would do the following:

```jsx
import styled from 'styled-components'

const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;
`
```

Here, we create a styled version of `button` using the style information provided between the backticks. This syntax may look awkward with it being a relatively niche feature introduced in ES6. Our styles here were defined using tagged template literals (enclosed by the backticks ``).

Tagged template literals take the template literal (the text in the backticks) and call the `tag` expression (the function, expression, etc. preceding the template literal) with the value of the template literal.

Let's just render this button to see its result:

```jsx
const TestButton = () => (
	<Button>This is a test of Styled Components!</Button>
)
```

This yields the following result:

![Styled Components Test](https://i.imgur.com/4MHQqlj.png)

Material UI allows for it to be styled using `styled-components` to which we use the `styled()` method provided by `styled-components` in order to create new styled components.

For example, if we wanted to create a styled `Button` component (not a plain `button`) with the `Button` component coming from Material UI, we would do the following:

```jsx
import React from 'react'
import styled from 'styled-components'
import Button from '@material-ui/core/Button'

const StyledButton = styled(Button)`
  background-color: #6772e5;
  color: #fff;
  box-shadow: 0 4px 6px rgba(50, 50, 93, 0.11), 0 1px 3px rgba(0, 0, 0, 0.08);
  padding: 7px 14px;
  &:hover {
    background-color: #5469d4;
  }
`
```

Here, we call the `styled()` method with the `Button` component that we want to style, to which we put the template literal containing the styling information to add after.