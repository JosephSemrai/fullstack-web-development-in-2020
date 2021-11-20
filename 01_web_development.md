[TOC]



# Book Todo

* Add Images/Diagrams written with the iPad Pro for
  * Document Object Model
  * HTTP Requests (Submitting New Data Using HTTP POST and Forms)
* Add section about style guides, linting, etc.
* Replace mentions of 'I' in first person
* Explain what the global object is in the JavaScript section



# Tips

As a beginner, whenever you encounter an error message, you should always take a second to understand the cause of the error message. With this, you should write your code advancing iin very small steps in order to avoid writing a lot of code that ends up not working. This way, you will only have a few lines that can be problematic at a given time and can take the time to understand why these lines are throwing errors, etc. 

# Introduction to Web Development

This book was written originally to take someone from knowing nothing to knowing a little bit of almost everything but not exactly everything that one would need to know to do something useful with web development. Looking at this now, you should probably have some programming experience or at least some background on how the magic that is the internet works, but if you don't, you'll probably be fine, just Google the things that aren't super clear to you. Have trust in the system.

## Hello World?

So, before we actually get into things, I just want to introduce you to some words that are super common in the world of web development and beyond. 

Also, please note that the explanations in Chapter 0 follow subpar programming practices coupled with mediocre systems design. These are just examples to be looked at as a high level overview of how things may work while giving you a taste at what these Application Programming Interfaces (APIs) can do. Additionally, there are some ommitted blocks of code that are not relevant to your understanding of these concepts. As a result, the code mentioned in Chapter 0 will not create a working app, but rather serve as a reference and a device for conceptual learning.

## How does a web page actually get to me?

Let's talk about web development in a very high level manner. Basically, when you visit a website in your browser, you are making a **HTTP GET** request to a web server somewhere that is connected to the internet.

### What is the HTTP Protocol?

HTTP is a protocol that allows for the fetching of resources as indicated by the receipient through requests. There is no link that is created, meaning that it is stateless. It also allows for extensibility with HTTP headers allowing for a simple agreement between a cleitn and a server to create new header semantics.

There are various HTTP methods used for various purposes.

| HTTP Methods | What does it do?                                             |
| ------------ | ------------------------------------------------------------ |
| GET          | Requests data from a specified resource                      |
| POST         | Sends data to a server to update or create something         |
| PUT          | Sends data to a server to update or create smething, but is *idempotent* (calling the same PUT request will always produce the same result, but multiple POST requests may produce duplicate results) |
| HEAD         | Pretty much identical to GET, but does not include the response body (does not return the contents of the response - often used for checking what a GET request will return before actuallying downloading it) |
| DELETE       | Deletes the specified resources                              |
| PATCH        | Update/Changes a resource. PATCH requests only contain the changes to the resource, not the entire resource. Thus, they should be in a patch language (JSON Patch, XML Patch, etc.) (not safe or idempotent on its own) |
| OPTIONS      | Describes/Outlines the comuncation options for the resource that is being targeted. |

These HTTP methods all have their own response codes which we'll talk about later. Also be aware that these methods all have different behaviors when cached, reloaded, called upon via the user pressing the back button, etc.

Now, visit a website and open up your developer tools panel (this book will reference the Chrome Development Tools) and click on the 'Network' tab. Upon refreshing or loading the page, you will see everything that the browser has requested. Click one of these requests to see more information about the request and response. Here you can see information such as the Content-Type of the page which indicates the format and type of the document returned. 

If you take a look at the reponse tab, you can see what was actually returned from the server from that GET request.

Further along the requests, you may see references to images that may have been loaded after receiving a HTML response with tags (such as the <img> tag) telling the browser to request other resources. Upon receiving an initial response from the server, a browser will make additional requests to load any additional resources that may be specified in the HTML.



## Delving into Dynamic Web Pages

Here we'll take a look at a code snippet for a dynamic web page that makes use of the Express framework for Node.js. We'll talk more about this later, so it's okay if you have a vague idea of what's going on here.

```javascript
const getHtml = (listLength) => {
  return(`
    <!DOCTYPE html>
    <html>
      <head>
      </head>
      <body>
        <div class='container'>
          <h1>This is an example</h1>
          <p>number of notes created ${listLength}</p>
        </div>
      </body>
    </html>
`)
} 

app.get('/', (req, res) => {
  const page = getHtml(list.length)
  res.send(page)
})
```

Here is an example of a dynamic web page that we generate on the server side. We take advantage of this fact by passing the page some data to display that we would not be able to do traditionally with a static file. Upon a request to the server, the server will take note of the length of a list stored in memory and return a HTML file with the value inserted in place of ```${listLength}```.

```javascript 
<p>Number of notes created ${listLength}</p>
```

> Important Note: For management purposes, it is typically not a good idea to write HTML in the middle of code. 

## Moving Application Logic to the Client

We don't have to run everything on the server and return a "static" HTML file. We can also incorporate client-side logic in the form of JavaScript files that make requests to the server and do their own processing.



We can mention a reference to a JavaScript file in our HTML in order to execute it and add client-side logic to our application.

```html
<script type="text/javascript" src="pathtofile.js"></script>
```

To dynamically load content in our application from the client side, we can take advantage of the XMLHttpRequest API based on HTTP which is primarily used to exchange data between a client and a service. The more modern Fetch API with a more powerful feature set will be discussed later after we gain our intial bearings.

**With code blocks such as these, please read along the code comments to follow what is going on**

```javascript
var xhttp = new XMLHttpRequest()

xhttp.onreadystatechange = function() {
  if (this.readyState == 4 && this.status == 200) { // Checks if the operation is complete and the HTTP status code is OK
    const data = JSON.parse(this.responseText) // Parses the body of the response
    console.log(data)

    var todoList = document.createElement('ul') // Creates an unordered list
    todoList.setAttribute('id', 'list') // Adds the class 'unordered' to the unordered list

    data.forEach(function(item) { // Creates a list item, adds the text content of each data item to the list item, and appends the list item to the unordered list for each item in 'data'.
      var li = document.createElement('li')

      todoList.appendChild(li)
      li.appendChild(document.createTextNode(item.content))
    })

    document.getElementById('list').appendChild(ul)
  }
)

xhttp.open('GET', '/data.json', true) // Defines a GET request to the serverAddress + /data.json path
xhttp.send() // Sends the request
```



Now, you might have noticed that we write the code for handling the response before we even send the request for the response. This is because

```javascript
http.onreadystatechange = function () {
  // Anything in here 
}
```

is an event handler defined on the ```xhttp``` object involved with the request that is called when the state of the object changes. Thus, if we assign a function to this event handler, it will be called when the state of this object changes.

When the request is completed, the state changes and calls this function. From within the function, we check if  ``readyState == 4 && this.status == 200`` to see if the operation is complete (readyState 4) and if the HTTP status code of the response is 200 (HTTP Status OK).

**This style of invoking event handlers is something that you will see a lot of in JavaScript.** The functions that are assigned to these event handlers are called **'callback functions'** where the code that you write does not invoke the function but rather the runtime environment (the browser in this case) will invoke the function when the event has occured.

### HTML Pages Expanded

#### Document Object Model

The Document Object Model, or DOM for short, is an API that allows for the editing of element trees programatically. What this means is that you can edit your page (like how we did in the above example) through the use of JavaScript by traversing the element tree that is a web page.

To have a glimpse at this tree-like structure, take a look at the 'Elements' tab in your console. Here you can see the various elements of a page nested within other elements, with the 'branches' extending out from their parents. This concept allows us to traverse from levels of branches from any point on the tree. 

You can also create elements and add them to the DOM and other objects in the DOM as we did earlier.

```JavaScript
var todoList = document.createElement('ul')
var li = document.createElement('li')
var li2 = document.createElement('li')
todoList.appendChild(li)
todoList.appendChild(li2)
```

We can also select elements programatically.

```javascript
li.setAttribute('class','list-item')
todoList.setAttribute('id', 'list')

document.getElementById('list').innerHTML = "example"
document.getElementsByClassName('list-item')[0].innerHTML = "example"
```

Here, ``getElementById()`` returns the one element with that ID of an element's child elements. The ``document`` object is the topmost node in the hierarchy of the DOM tree. As we are looking in the document's child elements, we are looking through all elements.

``getElementsByClassName()`` returns a collection of an element's child elements with the specified class name. In the above example, we adjust the innerHTML of the first and only element in that collection. InnerHTML relates to the HTML content of an element (the content within element tags).

## Styling with CSS

In order to provide our web page styling information, we can take advantage of Cascading Style Sheets (CSS for short) which allow us to define the apperance of web pages using a markup language.

Within the head element of the HTML code, we can create a ``<link>`` tag to fetch a CSS file from a particular address as such.

```HTML
<link rel="stylesheet" type="text/css" href="main.css" />
```

Now, this element refers to a file called "main.css" in the same directory as this HTML file. 

Within "main.css" we can write the following to define some styling information.

```css
.container {
  padding: 5em;
  border: 1px solid; 
}

.todoList {
  color: red;
}

#list {
  color: blue;
}

```

Here we define the styling information for two classes that we can specify as an attribute to an HTML element. A class selector definition in CSS always starts with a period and contains the actual name of the class.

You can write CSS rules to apply styling information to elements that contain these classes. Similarly, to use ID selectors, you would write ``#class-name`` as seen above with the 'list' ID.

###Combining CSS, HTML, and JavaScript

When you include CSS links and JavaScript files that make requests to the server, the browser will roughly follow this order (assuming the web application is layed out similarly to the one described above).

1. Browser fetches the HTML for the application through a HTTP GET request
2. Browser realizes that there is a link to a CSS file and a link to a JavaScript file
3. Browser fetches the CSS file
4. Browser fetches the JavaScript file
5. Browser executes the JavaScript code which has XMLHttpRequest object that makes a HTTP GET request in an attempt to retrieve JSON data.
6. Browser receives the JSON data to which the browser executes the event handler dealing with the XMLHttpRequest state change.
7. The callback function referenced by the event handler utilizes the DOM API in order to add and render the list.

## Submitting New Data Using HTTP POST and Forms

We can use HTML form elements in order to submit data to a server using the POST method. The syntax for doing so is as follows.

```html
<form action="/new_todo" method="POST">
  <input type="text" name="todo">
  <br>
  <input type="submit" value="Save">
</form>
```

Inserting this in our page gives us a text box and a "Save" button that allows the user to type in a todo and save it. Please note how the form tag takes in the attributes of ``action`` and ``method`` allowing us to specify the HTTP method and the address to point to.

When the button is clicked, the browser will send the values in the inputs (the textbox) to the server using a HTTP POST method directed at the "/new_todo" endpoint.

### ADD IMAGE HERE OF REQUESTS

Now, if you take a look at the Network tab when you submit this form, you might be surprised that it causes five HTTP requests. The reason for this being that the first event is the HTTP POST request to the server with the data to be submitted to which the server responds with the 302 HTTP status code that describes a URL redirect. This URL redirect tells the browser to do a new HTTP GET request to the address mentioned in the header of the response; as a result, the browser redirects to the location and loads the page bringing along more HTTP GET requests for each of the HTML, CSS, JavaScript, and raw data files (data.json that is fetched as part of the client side logic). 



Now, how might we handle this HTTP POST request on the server side of things?

On the server, we would write a few lines of code to handle this route.

```javascript
app.post('/new_todo', (req, res) => {
  todos.push({
    content: req.body.todo,
    date: new Date(),
  })

  return res.redirect('/todos')
})
```

Here, we handle a POST request to the '/new_todo' route. From this, we create a new todo object that contains the content of the object and the date. We then push this object to the todos array. After this is done, we put an instruction to redirect to '/todos' in the response.

Saving objects to memory does not permanently store it. A simple restart of the web server will cause everything to be lost as it is not saved to a database. We'll discuss how to save objects to a database later in this book. 

### Modern Conventions

Over time, these once traditional methods of how we just dealt with AJAX and our submission form became unacceptable. We have evolved from doing everything on the server side with HTML-code being generated entirely by the server to moving to what is called a Single Page App (SPA) with ever more logic being handled client side.

Traditional Web Apps (Fetching HTML from the web server) -> AJAX fetching data -> Modern APIs following a RESTful approach (We'll talk about this later in the book)

The example application that we just went through, by modern conventions, should not have fetched JSON data from a URL, but we'll talk about this more in depth later on. The application also utilized the traditional way of submitting a form to send data to our server. We can get closer to our goal of a SPA by changing a few things around.

### Adjustments

In order to prevent a redirect (against the goal of having a single-page app), we can write our form tag without any action or method attributes as follows.

```html
<form id="todos_form">
  <input type="text" name="todo">
  <br>
  <input type="text" value="Save">
</form>
```

Here, you'll notice that we got rid of the action and method attributes while also adding an ID to the form so that we can programatically select it using the DOM API.

Now, to send the form data the "SPA way", we must write some more JavaScript on the client side.

```javascript
var form = document.getElementById('todos_form')
form.onsubmit = function(e) {
  e.preventDefault() // Overrides the default behavior of some browers that may lead to a redirect, etc.

  var todo = {
    content: e.target.elements[0].value, // Gets the first input of the form and it's value (text content)
    date: new Date(),
  }

  todos.push(todo) // Adds the todo to the todos list so we can use it to rerender the page with it
  e.target.elements[0].value = '' // Clears the form text box
  redrawTodos() // Function that will cause all of the todos to be rerendered
  sendToServer(todo) // Function that will send the new todo to be sent to the server using the XMLHttpRequest API
}
```

Notice how we are creating functions that provide reusable functionality to make our code easier to maintain and read, especially as our projects continue to grow. Now, we'll go over the implementation of the ``sendToServer()`` method.

```javascript
var sendToServer = function(todo) {
  var xhttpForPost = new XMLHttpRequest()

  xhttpForPost.open('POST', '/new_todo', true)
  xhttpForPost.setRequestHeader(
    'Content-type', 'application/json'
  )
  xhttpForPost.send(JSON.stringify(todo))
}
```

In this method, we create a new `XMLHttpRequest` and set it's request header to match the type of data that we are sending. In this case, we are sending json objects, so we update the request header and use `JSON.stringify()` to convert our JavaScript `todo` object into a JSON string that can be sent to the server.



# JavaScript

This is just going to be a quick overview of some features of the JavaScript langauge and tips that will prove useful during development. This section will teach you everything that you will need to know to continue in this book, but by no means should be used to learn JavaScript. For the purpose of learning JavaScript, I heavily recommend the *You Don't Know JavaScript* book series.

## JavaScript Background

You should have experience with some language prior to going through this book; if that language was Java, great, you'll recognize some of the syntax. Do not mistake JavaScript as being similar to Java, however, as they are completely different in how they are used. Some may try to use JavaScript as if it were Java with regard to design patterns and features, but this is highly disapproved.

Another thing to know about JavaScript is that it is often transpiled down to older versions as browsers may not support all of the new features in newer versions of JavaScript. This is accomplished by using transpilers such as Babel which transpile JavaScript written in newer versions to older versions that are more compatible. When we break into creating our first React project with `create-react-app`, Babel will automatiically be configed to transpile our code.

JavaScript is widely known as a client-side langauge that enables interactivity on websites, but it can also run on anything from embedded machines to servers. Node.js, a JavaScript runtime environment based off of Chrome's V8 engine, allows for JavaScript to be executed outside of a browser.

## Variables

## Arrays

You can store an array in a const as follows

```js
const exampleArray = [5, 0, 3]
```

If you would like to add an item to the array, you can use

```js
exampleArray.push(6)
```

If you would like to execute a function for each item in an array, you can iterate through items in an array by using the `forEach()` function as follows

```js
exampleArray.forEach(valueFromArray => {
  console.log(valueFromArray)  // prints 4 lines containing 5, 0, 3, and 6
})
```

Here, you can see something called an 'arrow function' being defined and passed to the `forEach()` function. Upon receiving this function as a parameter, `forEach()` will execute that function for each of the items in the array, making sure to pass the individual element (for the current iteration) as a parameter for arrow function that we passed to it.

### Immutability and Concat

Now, when we eventually visit React, you'll soon find that a common feature is the use of immutable data structures. Thus, if we were to ever need to add another item to an array, we wouldn't add it to the same array reference but rather create a new array with our new value by using the `concat()` function.

```js
const newArray = exampleArray.concat(5)
console.log(exampleArray) // [5, 0, 3] is printed
console.log(newArray) // [5, 0, 3, 5] is printed
```

Here, you can see how using the `contcat()` method did not mutate our original array in any way but rather created and returned a new array with the function argument added on to it.

### The Map Expression

Expanding on this concept of immutability, when we want to modify or pass every value from an array into a function, the `map()` function (which will be used quite extensively in React) returns the results of an interable (array, list, etc.) after applying a given function. What this means for our arrays is that we can create completely new arrays where the function given as a parameter is used to create them based off of our original array. This is done without mutating our original array as it returns a completely new array.

```js
const exampleMap = [5, 3, 1]

const newExample = exampleMap.map(value => value * 2)
console.log(newExample)   // [10, 6, 2] is printed
```

### Destructuring Assignment Expression

You can assign individual items in an array to variables by using **destructuring assignment**.

```js
const toBeDestructured = [5, 6, 7, 8, 9]

const [firstItem, secondItem, ...everythingElse] = toBeDestructured

console.log(firstItem, secondItem)  // 5, 6 is printed
console.log(everythingElse)          // [7, 8, 9] is printed
```

Here, you can see that the variables `firstItem` and `secondItem` receive the first two integers of the array. Then the rest of the values get collected into a new array which is then assigned to the variable `everythingElse`. You can use any variable name and any amount of variables to get the values that you desire from the array.

## Objects and JavaScript Object Notation (JSON)

JavaScript contains objects that follow the JavaScript object notation. One way to create these objects is by using *object literals* in which you literally write the object in your code as follows

```js
const examplePersonObject = {
  name: {
    first: 'ExampleFirst',
    last: 'ExampleLast',
  },
  jobs: ['Teacher', 'Software Engineer'],
  workplace: 'Microsoft',
}
```

Within an object you have things called properties. These are essentially the key values that you would use in order to get the values of the properties. The syntax is, as you can see above, `property: value`. The values of these properties can be anything ranging from integers to full blown objects.

### Referencing Properties

In order to use values within objects, we can use dot notation or brackets to reference these properties and receive these values back.

```js
console.log(examplePersonObject.workplace) // Using dot notation to reference the property, 'Microsoft' is printed
console.log(examplePersonObject['workplace']) // Using brackets to reference the property, 'Microsoft' is printed
```

With brackets, we also gain the ability to reference properties based off of a variable instead of having to rely on string literals or hardcoded properties.

```js
const exampleField = 'workplace'
console.log(examplePersonObject[exampleField]) // Using bracket notation and a variable to reference the property specified in the variable, 'Microsoft' is printed
```

In addition, we can use the concepts from above to dive deeper into our nested object. Recall that our `name` property within our `examplePersonObject` object has an object of its own. We can access properties of that object as follows

```js
const requestedName = 'last'
console.log(examplePersonObject['name'][requestedName]) // Using bracket notation and a variable to reference the property within the object of the 'name' property, 'ExampleLast' is printed
console.log(examplePersonObject.name.first) // Using 'stacked' dot notation, we reference the name property and the 'first' property of the value of the name property leading to 'ExampleFirst' being printed
```

Here, you'll notice that we can 'stack' brackets or the use of dot notation multiple times to delve within objects. It may also be helpful to think of your objects as any file structure that you would find on your computer and the dot notation/brackets as the file path. 

> Please note, you *cannot* use variables with dot notation as of ES9. You also cannot reference properties with spaces or create new properties with spaces in their names when using dot notation as in the following example.

### Adding Properties

We can also add properties to any object after the fact by using brackets or dot notation as well.

```js
examplePersonObject.age = 204
examplePersonObject['favorite color'] = 'Blue'
```

Here, you'll notice, as mentioned earlier, we must use brackets to create the `'favorite color'` property as the name has a space in it.

Whenever we encounter these odd cases for property names, we can use string syntax when assigning our object literals as follows.

```js
const exampleObject2 = {
  'some random property': 25
}
```

Here, you'll notice that we wrapped the property name in quotes in order to accomplish this.

> Do not use awful, nondescript property names as the one illustrated above.

### Object Methods

You can also create methods for objects as well. You can simply add a function as the value to a property.

```js
exampleObject2.exampleFunction = function () {
  console.log("I'm called inside an object method!")
}

const exampleObject3 = {
  name: 'Some Random Value'
  bindDemo: function(e) {
    console.log(this.name)
  }
  talk: function(e) {
    console.log("Called from within an object with the argument: " + e)
  }
}
```

Now, to call these functions, we can use the usual dot notation or brackets, but w ecan also store a method reference in a variable and call that method through the variable as follows

```js
const referenceToTalk = exampleObject3.talk
referenceToTalk("Example") // Prints "Called from within an object with the argument: Example"
```

If you use the `this` keyword in any of your object methods, please note that `this` is defined based on how the method is called with these regular functions (explained in further detail below in the 'JavaScript Functions' section). Thus, if you call the function via reference, `this` becomes the global object and can lead to many errors.

For example, if we tried calling the `bindDemo` method from a reference, we will get an error

```js
const referenceToBindDemo = exampleObject3.bindDemo
referenceToBindDemo() // Error printed as there is no property in the global object
```

We can create new functions with `this` bound manually regardless of what is calling the method.

```js
const boundFunction = exampleObject3.bindDemo.bind(exampleObject3) // Bound to exampleObject3 so 'this' will work
```



With this, it is important to always keep track of `this` when writing JavaScript as it can lead to a plethora of issues. 

## INSERT INFORMATION ABOUT OBJECT METHODS IN JAVASCRIPT

### Constructor Functions

The use of a constructor may remind you of classes coming from a Java background or any of the many other languages that make use of this mechanism. I wouldn't get my hopes up too much, though, as JavaScript doesn't really have classes that behave in the same was as these other langauges.



We can write constructors as follows

```js
function Person(first, last, age, occupation) {
  this.firstName = first
  this.lastName = last
  this.age = age
  this.occupation = occupation
  this.changeAge = function (age) {
   	this.age = age
  }
}
```

Here, you can see the constructor function for the creation of a `Person` object for which we can use to create many objects of the `Person` "type". You'll also notice the use of a keyword `this` which is a substitute for the new object which is to be created. Upon the creation of this object, this binds itself to that object and the appropriate properties are created.

We can also add object methods to our constructor which will be applied to the objects that are created. You'll also notice how we use `this` to reference the object that we created using the constructor so that we can just call `changeAge()` in order to update a given `Person` object's age.

> It is recommended to name constructor functions with an upper-case first letter.

```js
var examplePerson = new Person("Jake", "Flannn", 25, "Software Engineer")
```

The above is how we would go about creating a object of the `Person` "type". If we wanted to go ahead and change the age of our `Person` object, we can call the object method stored in one of its properties.

```js
examplePerson.changeAge(25)
```

Here, the object method mentioned in the constructor has bound `this` to the `examplePerson` object, to which we are able to reference the object's age by `this.age` in the method and update it.

### Class Syntax

With version ES6 came the introduction of the *class syntax* which can help us structure objects and object-oriented classes. Now, this is where things might get familiar to those with a background in languages such as Java. Please keep in mind, although they behave similarly to Java objects, at their core, they are still JavaScript objects that are based on prototype inheritance.

#### Class Declarations

We can define a class by using a *class declaration* as follows

```js
class Person {
  constructor(firstName, lastName, age, occupation) {
    this.firstName = firstName
    this.lastName = lastName
    this.age = age
    this.occupation = occupation
  }
}
```

There can only be one `constructor` method for a class. When dealing with JavaScript inheritance and subclasses, you can use the `super` keyword to call the constructor of the super class. Additionally, you can not add a new property to an existing object constructor outside of the constructor function.

Also, please be sure to mention class declarations in your code before you actually attempt to create instances of these classes as class declarations are not hoisted unlike function declarations.

> If you care to read more about hoisting, I suggest you take a look at the 'JavaScript Nuances and Tips' section at the end of the chapter.

The body of a class is automatically put into "strict mode" for increased performance and safer guidelines (see 'JavaScript Nuances and Tips' for an in-depth explanation of what it does). 

#### Class Expressions

Class expressions are also another way to define a class in JavaScript versions past ES2015.

```js
var Square = class {
  constructor(dimension) {
    this.height = dimension
    this.width = dimension
  }
  area() {
    return this.height * this.width
  }
}

console.log(new Square(5).area()) // Prints 25
```

You'll notice that we omited the class name here, which you cannot do with class declarations. The constructor property is also optional for these classes with redeclaration being possible.

As mentioned earlier, the body of a class is automatically put into "strict mode".

#### Static Methods in Classes

When using the  `static` keyword to define a method for a class, we create a static method which can be called without instantiating the class (creating an instance of the class/the object created after using the constructor to create a new object). These methods **cannot** be called by class instances.

```js
class Example {
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  static add(a, b) {
    return a + b
  }
}
const exampleInstance = new Example(5, 5)
console.log(Example.add(5, 5)) // Prints 10
```

Notice how we called the `add()` static method from the actual class declaration rather than the `exampleInstance` instance that we have instantiated from the class. Static functions are used for utility functions that do not depend on properties of specific instances.

There are many more special behaviors with JavaScript classes, but this should suffice for most web development purposes. When the time comes to learn JavaScript behaviors, features, and syntax more deeply, it is recommended to read the 'MDN web docs' or the 'You Don't Know JavaScript' book series.



# JAVASCRIPT PROTOTYPES

Every object in JavaScript has an internal property known as 'prototype'





### JavaScript Functions

Now let's talk about one of the most important concepts that you'll find in JavaScript when dealing with web development, especially when dealing with React.

Nowadays, the most common way to define functions is by defining an arrow function. Arrow functions can be written as follows 

```js
const add = (a, b) => {
  return a + b
}

// To call our function
const sum = add(5, 5)
console.log(sum) // Prints 10
```

There are also shortcuts that we can take. If we just have one parameter that we need for our function, we can get rid of the parentheses around our parameters as follows

```js
const square = d => {
  return d * d
}
```

Going even further, if there's just one expression inside the body of our function, we can keep everything in one line and get rid of the brackets.

```js
const square = d => d * d
```

This shortened syntax often finds its use when dealing with the map method as it looks quite clean and can fit everything into one line.

```js
const toBeSquared = [5,2,3]
const squaredNumbers = toBeSquared.map(d => d * d)
```

Arrow functions behave differently than regular functions. For example, they bind the keyword `this` to the lexical scope instead of binding it based on context. Usually, regular functions bind this to the object that calls the function (the context), while arrow functions take the value of `this` in the scope that the function was defined. In other words, arrow functions bind the value of `this` to the value of `this` where it was defined. This makes them unsuitable for being used as an object method.

The differences between arrow functions and regular functions is something that you should definitely read up on later in your career, but for now, just go with the general rule that you must using function expressions for object methods while arrow functions are often better when dealing with many callbacks or methods such as `map()`, `reduce()` or `forEach()`.

### Function Expressions and Declarations (Regular Functions)

Before arrow functions, there were function expressions and function declarations. When we give a regular function a name, it is called a function declaration.

```js
function add(a, b) {
  return a + b
}
const sum = add(2, 9)
```

When we do not give a function a name, it is called a function expression:

```js
const add = function(a, b) {
  return (a + b)
}
const sum = add(2, 9)
```

Please do not worry about remembering all of this, we will revisit most relevant concepts later on when we encounter their actual real-world uses.

## JavaScript Nuances and Tips

### Using Strict Mode

With the release of ES5 came the introduction of a "strict mode" to the language. In general, the restrictions that this strict mode imposes keeps the code adhering to a set of safer guidelines and allows for further optimizations to be made by the JavaScript engine.

You can opt some or all of your code in to strict mode by writing

```javascript
"use strict";
```

"Strict mode" follows function scope, so you can declare it for a specific function or the entire global scope.

It is recommended to enable strict mode on the global scope as they solve common problems such as implicit auto-global variable declaration when you omit the 'var' declaration. Turning on strict mode might lead to encountering more errors or buggy behavior of code, but if this occurs, that is almost always a sign that there is something wrong with your program, thus you should keep strict mode on in most cases.

### Closures

To explain closures as simply as possible:

When definining a function inside another function, that inner function is able to access variables declared in the outer function. Thus, you can share variables between function calls without making the variables global. Then, when you call the inner function, it will have access to the variables defined in the outer function.

If you look online for some explanatoins, they will explain the technical reasons as to why this is in JavaScript, but if you just want a simple explanation for now, this will do for this book .

### Hoisting





## Spread Syntax



# Let's React

Now, there are a variety of JavaScript libraries for building user interfaces. React is just the most popular (as of writing) with a massive community and many supporters. To get started, we're going to make our first simple app using React as as the "view" layer of our web application.

The easiest way to get started is by using a tool called `create-react-app` which you can install using npm, the default package manager of Node. If you do not already have Node installed, feel free to install that now.

> Please note: I did not say that `create-react-app` was the best way to get started. The reason for this being that `create-react-app` is quite bloated with a huge bundle size and is quite messy when ejecting. This is because 'create-react-app' comes with many tools as a one-size-fits-all approach.

To create your first React application, run the following commands in a terminal.

> Make sure not to write the `$` in your commands as that simply denotes that the following line is to be typed into a terminal.

```shell
$ npx create-react-app project1
$ cd project1
```

This will create your app in a directory called project1 and configure it accordingly. Then, you must change your working directory to this new "project1" directory with the `cd` command. After this, you can finally start your application by running

```shell
$ npm start
```

Upon running this command, the application should run at the address `https://localhost:3000`

Chrome should have opened automatically, so you should now open your developer tools so we can get started.

For all intents and purposes with regard to learning React, we can simplify our `index.js` file and our project directory quite significantly. 

Please replace your `index.js` file with the contents below.

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

const App = () => (
  <div>
    <p>My first React app!</p>
  </div>
)

ReactDOM.render(<App />, document.getElementById('root'))
```

You can also go ahead and delete `App.js`, `App.css`, `App.test.js`, `logo.svg`, and `serviceWorker.js` as they are not needed for this project.



## Components

Within our `index.js` file, you'll find an item called a React-component with the name of `App`.

There is also a line that renders this component into view at the bottom of the file as follows

```jsx
 ReactDOM.render(<App />, document.getElementById('root'))
```

This line renders the component into the div-element with the id of 'root' that was specified in the `public/index.html` file.

Now, you can always write standard HTML into that file, and it will appear, but with React applications, we try to write everything defined as React components. Who would've known?



Let's take a dive into our `App` component. Looking at the code, you can see that the component places a `<p>` tag containing the text 'My first React app!' inside a `div` tag. Moving on to the JavaScript aspect, the component is an anonymous JavaScript function (a function that is not bound to an identifier or more simply, a function that does not have a name) with no parameters. This function is then stored inside the constant variable `App`.

This function syntax also follows something new that was introduced in ECMAScript 6 (ES6), arrow functions. We have also used a shorthand that returns the value of the expression within the parentheses,

`const App = () => ()`, where the evaluation of the parentheses will be returned.

Ommiting the shorthand, we can write the same block as

```jsx
const App = () => {
  return (
    <div>
      <p>My first React app!</p>
    </div>
  )
}
```

Where we now explicitly return the component that we want to render. Upon doing this, we can now render dynamic content as we can execute any JavaScript statement before the return statement.

For example, we can now write

```jsx
const App = () => {
  const now = new Date()
  const a = 50
  const b = 50

  return (
    <div>
      <p>My first React clock! It is {now.toString()}</p>
      <p>
        {a} plus {b} is {a + b}
      </p>
    </div>
  )
}
```

Here, you can see us execute JavaScript statements before the return statement which allows us to store values within variables to be referenced in our component. We reference these variables in our component by enclosing them within curly braces. In our return statement, we can put JavaScript code within curly braces which will be evaluated and inserted into our component.

Now, you might have noticed that the code that we are writing in our return statements looks like an awful lot like HTML markup. In reality, we are dealing with straight JavaScript using JSX. This JSX that will be returned will be returned as pure JavaScript. Now, this compiling from JSX to JS is done by something called Babel and is automatically set up when you use `create-react-app`. We will talk about this more later in the book once we dive into how everything works.

With JSX being compiled to JS anyway, you'll notice that you can write React using pure JavaScript without using JSX, but this is not recommended as JSX simplifies things dramatically with its declarative style and more.

This compiled JSX will look like

```js
var App = function App() {
  var now = new Date();
  var a = 50;
  var b = 50;
  return React.createElement("div", null, React.createElement("p", null, "My first React clock! It is ", now.toString()), React.createElement("p", null, a, " plus ", b, " is ", a + b));
};
```

in pure JavaScript. If you were to write this component in pure JavaScript, you would have to write this.

JSX isn't exactly like HTML though, it is "XML-like", meaning that every tag written needs a closing tag.

For example, when writing HTML, you can write a line break element as follows

```html
<br>
```

However, when writing JSX, you must have a closing tag as follows

```jsx
<br />
```

or when dealing with components that may require items between the starting and closing tags

```jsx
<h1>I need a closing tag!</h1>
```



**As you begin to write your own components, please note that a React-component must have a name that starts with a capital letter.** The reason for this being that the component use will be treated as a built-in element (such as a span or div) if the component does not start with a capital letter.

### Adding Complexity

We can include components inside components, allowing us to increase modularity and maintainability across our components.

```jsx
const ExampleComponent = () => {
  return (
    <div>
      <p>I will be used inside another component!</p>
    </div>
  )
}

const App = () => {
  return (
    <div>
      <h1>Here's my React App!</h1>
      <ExampleComponent />
      <ExampleComponent />
      <ExampleComponent />
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

Here, you can see that we mentioned `<ExampleComponent />` multiple times in our `App` component. This allows us to break our project into many components designed for a purpose that can be reused in other components, allowing us to keep our project maintainable as it grows larger.



## Components With Data (Props)

Now, let's combine the concept of using JavaScript in our JSX and the modular structure of React. Doing so requires the use of *props* which allow us to pass data to components.

For example, we can create the following components that pass data to each other and use it in a meaningful way.

```jsx
const Welcome = (props) => {
  return (
    <div>
      <p>Welcome, {props.title}{props.name}!</p>
    </div>
  )
}

const App = () => {
  const josephTitle = 'Mr. '
  const josephAge = 37
  
  return (
    <div>
      <h1>Members: </h1>
      <Welcome name="Grace" title="Ms. " age={5+20}/>
      <Welcome name="Joseph" title={josephTitle} age={josephAge}/>
    </div>
  )
}
```

Here, you can see that we make use of the `Welcome` component twice in our `App` component. Within our calls to the `Welcome` component, we pass in the prop `name` as a string parameter. We can also pass the result of a JavaScript expression by wrapping it in curly braces as you can see with our references to the `josephTitle` and `josephAge` variables as well as our evaluation of the expression `5+20`.
When you take a look at our `Welcome` component, you can see how we reference prop values via dot notation.

When writing your React components, keep iin mind that the content of a React component needs to contain one root element or return an array of components. You cannot write the `App` component as follows

```jsx
const App = () => {
  const josephTitle = 'Mr. '
  const josephAge = 37
  
  return (
    <h1>Members: </h1>
    <Welcome name="Grace" title="Ms. " age={5+20}/>
    <Welcome name="Joseph" title={josephTitle} age={josephAge}/>
  )
}
```

Here, without the `<div>` tag wrapping the three inner components/tags, we will get an error message relating to our code being unable to be compiled.

> ... Parsing error: Adjacent JSX elements must be wrapped in an enclosing tag. Did you want a JSX fragment <>...</>?

This is because we are no longer only returning the root `<div>` tag, but are instead trying to return 3 adjacent JSX elements.

As we have discussed earlier, we can also return an array of components as follows

```jsx
const App = () => {
  const josephTitle = 'Mr. '
  const josephAge = 37
  
  return [
    <h1>Members: </h1>
    <Welcome name="Grace" title="Ms. " age={5+20}/>
    <Welcome name="Joseph" title={josephTitle} age={josephAge}/>
  ]
}
```

This should be avoided, however, as it is quite unsightly and may violate some style guides.

With our first solution of wrapping multiple JSX elements in one enclosing tag, you might notice that it creates extraneous div-elements that pollute our DOM-tree. We can avoid this by using 'fragments' that were mentioned in the error message above.

```jsx
const App = () => {
  const josephTitle = 'Mr. '
  const josephAge = 37
  
  return (
    <>
      <h1>Members: </h1>
      <Welcome name="Grace" title="Ms. " age={5+20}/>
      <Welcome name="Joseph" title={josephTitle} age={josephAge}/>
    </>
  )
}
```

Here, the fragment's opening tag is `<>` with `</>` as the closing tag. Fragments are essentially empty elements that we can use to wrap the elements to be returned by the component. Doing this allows us to compile our code successfully without making it unsightly or adding extra div-elements to our DOM-tree.

## Dealing With Component State

Let's first start off by writing a component that greets a person with text inside a div.

```jsx
const Greet = (props) => {
  return (
    <div>
      <p>
        Welcome {props.name}, we have your age as: {props.age} 
      </p>
    </div>
  )
}

const App = () => {
  const exampleName = 'Jake'
  const exampleAge = 15

  return (
    <div>
      <Greet name="Jason" age={36 + 10} />
      <Greet name={exampleName} age={exampleAge} />
    </div>
  )
}
```

### Component Helper Functions

Now, if we wanted to write a component that also wrote about how many days the person has been alive based off of their age, we could make use of what is called a **component helper function**.

```jsx
const Greet = (props) => {
  const ageDays = () => {
    return props.age * 365
  }
  
  return (
    <div>
      <p>
        Welcome {props.name}, we have your age as: {props.age} 
      </p>
			<p>
        You are close to {ageDays()} days old
      </p>
    </div>
  )
}
```

As the function is defined within the component function, it has access to all the props passed to the components. Thus, we do not need to pass the age prop to the function. We then use the function that we defined within the `Greet` component in order to get, rougly, the amount of days that the user has been alive for. 

The result of this component should look like this:

![image-20190915002753027](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20190915002753027.png)

### Destructuring in JavaScript

We can streamline our references to our props by using something called destructuring to extract the values of an object's properties into separate variables without needing to reference them via brackets or dot notation.

```jsx
const Greet = (props) => {
  const { name, age } = props // Assigns the name property to the variable 'name' and the age property to the variable 'age'
  
  const ageDays = () => {
    return age * 365
  }
  
  return (
    <div>
      <p>
        Welcome {name}, we have your age as: {age} 
      </p>
			<p>
        You are close to {ageDays()} days old
      </p>
    </div>
  )
}
```

Here, you'll see that we no longer have to reference these properties through dot notation/brackets upon the object, we can just simply reference the properties by their names as they have been assigned to constants when they were destructured.

In order to further streamline this, we can destructure the props into the variables in the argument list when we define our arrow function.

```jsx
const Greet = ({ name, age }) => { // The props are destructured in the argument list
  const ageDays = () => {
    return age * 365
  }
  
  return (
    <div>
      <p>
        Welcome {name}, we have your age as: {age} 
      </p>
			<p>
        You are close to {ageDays()} days old
      </p>
    </div>
  )
}
```

Here, instead of passing the entire `props` object into the component function, we destructure the props into the two variables `name` and `age` which are then passed into our component without passing in the entire `props` object. Thus, any references to the `props` object will lead to a 'Failed to compile' error. With this, it is important to remember that you will not be able to reference the `props` object and will have to destructure any new props that you pass into your component.

### Component Re-Renders

Let's bring some interactivity into our components.

```jsx
const App = (props) => {
  const { count } = props
  return (
    <div>
      <p>
        The current count is: {count}
      </p>
    </div>
  )
}

let count = 1

const refresh = () => {
  ReactDOM.render(<App count={count} />, 
  document.getElementById('root'))
}

setInterval(() => {
  refresh()
  count += 1
}, 1000)
```

Now, you'll see here that we have a function called `refresh` that will call upon `ReactDOM.render` to render our component to the root element in our document. Within this `refresh` function, we pass the current value of the variable count to our `App` component which will then be rendered and added to the root element.

Moving down to the bottom of our snippet, you'll also notice that we have a `setInterval` method that calls our `refresh` function and then adds 1 to the count variable every 1000 milliseconds (1 second). This effectively renders our component every second: first with the count variable as 1, second with the count variable as 2, and so on.

This actually isn't how things are typically done with React though, we shouldn't need to call `ReactDOM.render` in order to re-render components. We'll discuss better ways to re-render our components next.

### Stateful Components

React recently had the addition of 'React Hooks' in React 16.8. We'll now be taking advantage of React's state hook. React Hooks allow us to add state to function components wihtout converting it to a class.

Let's rewrite our above app to make use of `useState`.

```jsx
import React, { useState } from 'react'
import ReactDOM from 'react-dom'

const App = (props) => {
  const [ count, setCounter ] = useState(0)

  setTimeout(
    () => setCounter(count + 1),
    1000
  )

  return (
    <div>
      <p>
        The current count is: {count}
      </p>
    </div>
  )
}

ReactDOM.render(
  <App />, 
  document.getElementById('root')
)
```

There are quite a few new concepts and changes to this application, so let's take a look into what has changed and what these new functions do.

You'll first notice that we imported the `useState` function from 'react' in order to take advantage of React's state hook.

Next, draw your attention to the following line

```jsx
const [ count, setCounter ] = useState(0)
```

Here, we make use of the destructuring syntax to assign the elements of the array that `useState` returns to the constants `count` and  `setCounter`. When you call `useState`, it creates this 'state variable' which will be preserved by React and not 'disappear' (due to the scope of our variables) when the function exists unlike normal variables. It also creates a function that we can use to update our state.

The first element in the array that is returned from the `useState` function call is the 'state variable'. The second element in the array that is returned from the call is our function that we can use to update our state. We then utilize array destucturing (the bracket syntax) in order to store the first element in the `count` constant and the update function `setCounter` constant.

One common point of confusion is simply just passing an expression to update the state in a `setTimeout` function or anywhere. You typically will have to create a function to change state and pass that to any function which will update your state. For example, in our application above, we wrote

```jsx
setTimeout(
    () => setCounter(count + 1),
    1000
  )
```

In this snippet, we call the `setTimeout` function and pass it an arrow function (with shortcut syntax) to increment the counter variable in our state and with a timeout duration of 1000 milliseconds. Every time this state modifying function is called, React will go ahead and re-render the component. When dealing with functional components, the entire function's body will be re-executed upon a re-render. Upon calling `setCounter`, the `count` variable will be updated and we can reference it as such.





Just to provide some contrast, if we were to not use React Hooks and convert it to a class, our component would look something like this.

```jsx
import React from 'react'
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    }
  }

  render() {
    return (
      <div>
        <p>
          The current count is {this.state.count}
        </p>
      </div>
    )
  }

  componentDidMount() {
    setInterval(() => { 
        this.setState({
            count: this.state.count + 1
        })
    }, 1000)
  }
}
```

Notice how things can be quite verbose and how we have to employ slightly different practices such as referring to our state variables with dot notation and `this.state`. 



## Event Handlers

Event Handlers are properties of methods that manage how a particular element reacts to events. Upon an event, the handler (a function that is invoked by an event listener) will be executed.

In React, button elements have a variety of mouse events. We can use the click event to make something happen as a result of pressing a button as follows

```jsx
class EventHandlerButton extends React.Component {
  handleClick = () => {
    console.log('A button has been clicked!');
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click Me!
      </button>
    );
  }
}
```

Here, we took the 'onClick' event handler and set the handler to our `handleClick` function in our class. Notice how follows the syntax of a property and is passed like a prop would be to our component.

Moving back to React Hooks and our functional component, here's an example of a more complex  component that increments our `count` value upon a button press. 

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => setCounter(count + 1)}>
        Increment
      </button>
    </div>
  )
}
```

Here, we have defined a function inside the onClick event handler which takes the current value of the `count` variable in our component's state, adds one to that value, and uses `setCounter` to set its value to the incremented value. 

In the interest of keeping things more manageable, we can declare our functions outside of the event handler as helper functions similarly to what we did with our React component class above.

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)

  const incrementOne = () => setCounter(count + 1)
  
  return (
    <div>
      <div>{count}</div>
      <button onClick={incrementOne}>
        Increment
      </button>
    </div>
  )
}
```

With this, it is fair to say that event handlers (handlers) are just functions. However, when setting our `onClick` property/event handler to a function, we can make a fatal mistake very easily by calling the function rather than a reference to the function.

### React Re-Rendering Error

```jsx
<button onClick={incrementOne()}>
        Increment
</button>
```

Writing this with the parentheses makes `incrementOne()` a **function call** which is not the reference to the function that it is expecting. This is an issue, because if we have a direct function call, the function will be executed upon the rendering of the component rather than just upon the event. This creates an infinite cycle as the `incrementOne()` function calls the update function `setCounter` which causes the component to be re-rendered executing the `incrementOne` function, starting the cycle once again.

Now, for this case, we could just have omitted the parentheses in order to make it a reference to the `incrementOne` constant once again, but what if we needed to pass parameters into the function?

Let's explore a function that sets the counter to a specific function.

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)

  const incrementOne = () => setCounter(count + 1)

  const setCounterValue = (value) => setCounter(value)
  
  return (
    <div>
      <div>{count}</div>
      <button onClick={incrementOne}>
        Increment
      </button>

      <button onClick={setCounterValue(0)}>
        Set Counter to 0
      </button>
    </div>
  )
}
```

Here, we have written a function that should take in a parameter and set the count variable to that given variable. We also have created a button that **incorrectly** makes a function call to `setCounterVariable()` rather than passing in a reference. This is because we only want to tell React the function to execute upon an event (in this case, the onClick event). When we pass a function call as the handler, we do not do this but rather execute the function leading to unwanted behavior.  This will cause the same issue as described earlier, so how can we fix this and still retain the ability to pass in parameters to our function?

One quick and easy way to do this is to make the event handler a function that calls our `setCounterValue` function.

```jsx
<button onClick={() => setCounterValue(0)}>
```

Now, `setCounterValue(0)` will not be executed when our component is rendered. Wrapping our function call inside an arrow function makes our function call behave similarly to a function reference and fixes our problem. This works because this syntax in which we wrap our actual function call inside an arrow function with shortened syntax that **returns** the function that we want to execute as a result of its call. So, when React goes ahead and executes this function as a result of us mentioning a function as our event handler, it is essentially given the function that we want to execute upon the event without executing it.

Another way we can solve this problem is by defining a function that ends up returning a function.

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)

  const incrementOne = () => setCounter(count + 1)

  const setCounterValue = (value) => {
		return () => {
      setCounter(value)
    }
  }
  return (
    <div>
      <div>{count}</div>
      <button onClick={incrementOne}>
        Increment
      </button>

      <button onClick={setCounterValue(0)}>
        Set Counter to 0
      </button>
    </div>
  )
}
```

Here, we edited the `setCounterValue` function to take in the value and return a function that calls the `setCounter` function in order to update our state variable. We can now write the "function call" without wrapping it in an arrow function or encountering an error as this "function call" behaves like a reference to the function with the call only returning the function to be exectuted rather than executing the function. Thus, when the component renders, the `setCounter` function will not be called, avoiding the issue of infinite re-renders. This is similar to our above solution in that, upon a call, it returns the function that we actually want to execute upon an event which behaves like a reference to the function.

With this implementation, you might have also noticed that we now no longer have a need for a dedicated `incrementOne` helper function. We can replace its reference in the "Increment" button as seen in the following implementation of our component

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)
  
  const setCounterValue = (value) => {
		return () => {
      setCounter(value)
    }
  }
  return (
    <div>
      <div>{count}</div>
      <button onClick={setCounterValue(count + 1)}>
        Increment
      </button>

      <button onClick={setCounterValue(0)}>
        Set Counter to 0
      </button>
    </div>
  )
}
```

This new component makes use of the `setCounterValue` function to both increment and set our counter to 0.

When dealing with React, you may also see things called "double arrow functions", these functions are a result of the above scenarios in which the first function call "configures" the second function through definining its parameters in which we are left with a returned function call with the parameter "inserted". For example, with this function

```jsx
const setCounterValue = (value) => {
		return () => {
      setCounter(value)
    }
}
```

Calling `setCounterValue(100)` returns

```jsx
() => {
      setCounter(100)
}
```

as `value` is passed in to the second arrow function "configuring" it with the value that we desire so that we do not have to pass in parameters to this function and can just take the function as is. This illustrates how these functions return other functions that contain our parameters inserted so they can be used almost as a reference and are not executed upon their initial call (they must be called two times in order to get our final result, React does this by first "unwrapping" the desired function through calling it upon a render and then by calling the desired function upon the specified event). 

Since we're dealing with arrow functions, we can take advantage of the shortened syntax that it offers in order to achieve the same result more elegantly.

The same function can be written as

```jsx
const setCounterValue = (value) => () => {
  setCounter(value)
}
```

since we can omit the return statement and the surrounding curly braces as there is only one expression for the surrounding/initial function.

We can take this one step further and omit the curly braces surrounding our returned function as we are only returning one expression once again.

This shortens our function that returns a function to

```jsx
const setCounterValue = (value) => () => setCounter(value)
```

which is a "double arrow function" with compact syntax. This way of dealing with functions is called "currying" and it involves nesting functions for each possible argument so that the inner components do not need to have all the arguments passed to them but rather get retain/get them through the natural closure created by the nested functions.



## Child Components

One of React's core principles is to write components that can be reused with maintainability in mind. Thus, it is recommended to keep components quite small to make things easier, especially as your project scales. Let's take a look at our application and split it up into three components that are logical with regard to what they are used for and achieve.

First, we can create a `Display` component that allows us to actually display the value of the counter. As you continue to familiarize yourself with React, you'll find that it is best to "lift the state up" as high as possible in the component hierarchy (i.e. keeping state accessible to all the components that reflect the same changing data through keeping the state above their closest common ancestor).

```jsx
const Display = ({ counter }) => <div>{counter}</div>
```

Here, our `Display` component gets the application state passed ot it and is able to display it in terms of returning the appropriate `div` which contains everything to be shown on the page.

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)
  
  const setCounterValue = (value) => () => setCounter(value) // Making use of double arrow functions described earlier
  return (
    <div>
      <Display counter={count} /> {/* Using our display component mentioned earlier */}
      <button onClick={setCounterValue(count + 1)}>
        Increment
      </button>

      <button onClick={setCounterValue(0)}>
        Set Counter to 0
      </button>
    </div>
  )
}
```

This new App component makes use of the Display component that we wrote earlier and passes the current `count` value from our state to the component.

We can also make a component for our buttons to simplify things a bit.

Here is the implementation of our `CounterButton` component.

```jsx
const CounterButton = ({ onClick, text }) => (
  <button onClick={onClick}>
    {text}
  </button>
)
```

Here's the updated `App` component that takes advantage of our `CounterButton` component.

```jsx
const App = (props) => {
  const [ count, setCounter ] = useState(0)
  
  const setCounterValue = (value) => () => setCounter(value) // Making use of double arrow functions described earlier
  return (
    <div>
      <Display counter={count} /> {/* Using our display component mentioned earlier */}
      <CounterButton 
        onClick={setCounterValue(count + 1)}
        text='Increment'
      />

      <CounterButton 
        onClick={setCounterValue(0)}
        text='Set Counter to 0'
      />
    </div>
  )
}
```

After this, we have three components with the `App` component making use of the `CounterButton` component in order to update the `count` state variable and the `Display` component in order to display this variable.

> Note that since our `CounterButton` component is custom, we do not have to name our `onClick` attibute 'onClick' as we are not using the default DOM `<button>` element. Thus, we are free to give any name to `CounterButton`'s `onClick` property. However, please remember that it is conventional to name props that represent events 'on[Event]' and to name methods that handle the events 'handle[Event]'.



## Effective State Practices

As your applications get more complex, your projects will progressively contain more components with more variables in their states. Most of the time, you'll find that just adding more uses of the `useState` function to create more state variables will do.

Let's start with an application designed to track the amount of times that two buttons labeled with 'no' and 'yes' were pressed by a user.

```jsx
const App = (props) => {
  const [noCount, setNo] = useState(0)
  const [yesCount, setYes] = useState(0)

  return (
    <div>
      <div>
        {noCount}
        <button onClick={() => setNo(noCount + 1)}>
          No
        </button>
        <button onClick={() => setYes(yesCount + 1)}>
          Yes
        </button>
        {yesCount}
      </div>
    </div>
  )
}
```

Here, we used `useState` two times to create two state 'variables' and functions that we can use to update our state variables. Now, there aren't really any rigid rules toward what a component's state can consist of, if we wanted to organize more complex state situations, we could throw these variables into an object as properties and update this object whenever we wanted to update the state rather than the indivdual variables. We would achieve this as follows

```jsx
const App = (props) => {
  const [clicksObject, setClicks] = useState({
    yes: 0,
    no: 0
  })
 

  const handleYesClick = () => {
    const updatedClicksObject = { 
      yes: clicksObject.yes + 1, 
      no: clicksObject.no 
    }
    setClicks(updatedClicksObject)
  }

  const handleNoClick = () => {
    const updatedClicksObject = { 
      yes: clicksObject.yes, 
      no: clicksObject.no + 1 
    }
    setClicks(updatedClicksObject)
  }
  
  return (
    <div>
      <div>
        {clicksObject.no}
        <button onClick={handleNoClick}>
          No
        </button>
        <button onClick={handleYesClick}>
          Yes
        </button>
        {clicksObject.yes}
      </div>
    </div>
  )
}
```

This achieves the same result as the application described earlier, just with a slightly different implementation. Let's break it down a bit.

In this application, we created the object `clicksObject` using the `useState` method. This means that our state consists of `clicksObject` and we can use the `setClicks` constant that we passed to `useState` as a function reference to update our state object. When we press the 'No' button, the `onClick` event is invoked and the `handleNoClick` handler function is called. 

```jsx
const handleNoClick = () => {
    const newClicks = { 
      yes: clicksObject.yes, 
      no: clicksObject.no + 1 
    }
    setClicks(updatedClicksObject)
}
```

This function creates a new object that takes the properties from the `clicksObject` object in our application state and puts it into our new object, `updatedClicksObject`, we then add one to the `no` property in order to reflect the updated count. Finally, we call` setClicks` and pass in our new object to set as our application state, updating everything and showing our new count. The `handleYesClick` handler functions the same way, only referencing the `yes` property of the object instead to be incremented by one.

Now, later on, as your objects grow in size and your application state becomes complex, you may write needlessly long, complex code in order to handle simple state updates with this solution. Fortuneately, there's a better way to handle things in a neater, more organized fashion. We can make use of the *object spread* syntax described earlier in order to copy the properties of our original object without having to specify each property as follows

```jsx
const handleYesClick = () => {
  const updatedClicksObject = { 
    ...clicksObject, 
    yes: clicksObject.yes + 1 
  }
  setClicks(updatedClicksObject)
}

const handleNoClick = () => {
  const updatedClicksObject = { 
    ...clicksObject, 
    no: clicksObject.no + 1 
  }
  setClicks(updatedClicksObject)
}
```

As a reminder, the object spread syntax essentially creates a copy of the object that follows the ``...``, thus we can use it to create a copy of an object and update only the property that we need to by explicitly referencing it.

This implementation also allows us to add new properties to our state object without needing to add it to all of our handler functions as they would create a new object that would not reflect these additional properties.

Now, in the nature of making our code clean, consise, and easy to read, we can just create the object inside the `setClicks` call and pass it in directly as follows

```jsx
const handleYesClick = () => setClicks({ ...clicksObject, yes: clicksObject.yes + 1 })

const handleNoClick = () => setClicks({ ...clicksObject, no: clicksObject.no + 1 })

```

Here's a point in which many may encounter an issue when dealing with React. In pursuit of the most simple solution, one may try to just update the state directly as such

```jsx
const handleYesClick = () => {
  clicksObject.yes++
}
```

or

```jsx
const handleYesClick = () => {
  clicksObject.yes++
  setClicks(clicksObject)
}
```

Neither of these solutions are acceptable as **React does not allow for state to be mutated directly**. Objects in state should never be mutated (as in updating the object that is referenced by a variable or constant) but rather replaced with a new object. If you tried the above incorrect solutions for the handler methods, you would find that the counters would never be updated as React would not trigger a rerender due to it thinking that nothing had changed. React thinks nothing has changed, because even though the object was updated, the object reference has not changed (a new object was not passed), thus React is none the wiser to any updates.

> Storing our two separate variables in one object for our application state is not a good application design approach. This was only meant to serve as an example for the instances where data structures involving objects may be more appropriate for your component.

## Arrays in State

Let's take the above application and adapt it to use a single array to keep track of our 'Yes' and 'No' clicks. We'll use this array to display a history of all clicks.

```jsx
const App = (props) => {
  const [clicksObject, setClicks] = useState({
    yes: 0,
    no: 0
  })
  
  const [clicksArray, setArray] = useState([])
 

  const handleYesClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = { 
      yes: clicksObject.yes + 1, 
      no: clicksObject.no 
    }
    setClicks(updatedClicksObject)
    
    // Handles updating the clicksArray
   	setArray(clicksArray.concat('Yes')) // We use concat here to create a new array and pass it to setArray as we cannot mutate the original array in React.
    
  }

  const handleNoClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = { 
      yes: clicksObject.yes, 
      no: clicksObject.no + 1 
    }
    setClicks(updatedClicksObject)
    
    // Handles updating the clicksArray
    setArray(clicksArray.concat('No'))
  }
  
  return (
    <div>
      <div>
        {clicksObject.no}
        <button onClick={handleNoClick}>
          No
        </button>
        <button onClick={handleYesClick}>
          Yes
        </button>
        {clicksObject.yes}
      </div>
      
      {/* Displays the contents of clicksArray */}
      <p>{clicksArray.join(' ')}</p>
      
    </div>
  )
}
```

Please take a look at the comments in our new application to see the changes that we have made in order to reflect this new array and our display of our click history. In order to create our history array, the `useState` hook was used to initialize our state with a blank array. We then use the updater method in order to update our array when one of the buttons is clicked as follows

```jsx
setArray(clicksArray.concat('Yes'))
```

Here, we use the `setArray` updater method to update our array in our component's state in order to reflect our new click. 

Now, you might think about using the `push()` method in order to add the elements to the array. You should **never** do this; recall that we discussed the nature of React and how state must not be mutated. Mutating state directly almost always leads to undesired behavior and issues later down the line. While this may be hard to grasp initially, it is best to reinforce these practices early on.

Looking at how we display the content of our clicksArray (the history of our clicks), we simply use the `join()` method in order to take the contents of our array and convert it into a string with a space between every entry which is then rendered to the page.

### Dynamic Content

In this section, *conditionals* will be introduced which allow us to conditionally (who would've thought?) render items in our component.

First, let's take the history section in our previous section and componentize it.

```jsx
const ClickHistory = (props) => {
  if (props.clicksArray.length === 0) {
    return (
      <div>
        Please press one of the buttons!
      </div>
    )
  }

  return (
    <div>
      History: {props.clicksArray.join(' ')}
    </div>
  )
}
```

Here, you'll notice that we introduced an `if` statement in our component. This allows us to **conditionally** render items in our component depending on what our expression evaluates to. In this case, if the array in our state has no items within it, the component will display a message to the user, 'Please press one of the buttons!', otherwise, we'll display the history of `clicksArray` to the user.

Let's now update the `App` component in order to take advantage of this new `ClickHistory` component.

```jsx
const App = (props) => {
  const [clicksObject, setClicks] = useState({
    yes: 0,
    no: 0
  })
  
  const [clicksArray, setArray] = useState([])
 

  const handleYesClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = { 
      yes: clicksObject.yes + 1, 
      no: clicksObject.no 
    }
    setClicks(updatedClicksObject)
    
    // Handles updating the clicksArray
   	setArray(clicksArray.concat('Yes')) // We use concat here to create a new array and pass it to setArray as we cannot mutate the original array in React.
    
  }

  const handleNoClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = { 
      yes: clicksObject.yes, 
      no: clicksObject.no + 1 
    }
    setClicks(updatedClicksObject)
    
    // Handles updating the clicksArray
    setArray(clicksArray.concat('No'))
  }
  
  return (
    <div>
      <div>
        {clicksObject.no}
        <button onClick={handleNoClick}>
          No
        </button>
        <button onClick={handleYesClick}>
          Yes
        </button>
        {clicksObject.yes}
      </div>
      
      {/* Displays the contents of clicksArray */}
      <ClickHistory clicksArray={clicksArray} />
      
    </div>
  )
}
```

Just like that, we have our new React app that is partially componentized with our `ClickHistory` component.

#### Proper Usage of React Hooks

As mentioned earlier, React Hooks are a relatively new addition to React. Thus, there are many pieces of code out there that still use React component classes rather than functional components with state (made possible by React Hooks).

With this, there are a few limitations and rules surrounding React Hooks that you should be made aware of.

You cannot call the `useState` function from within a loop, conditional expression, etc. (basically anywhere that is not in the root of the function defining the component). The reason for this being that hooks should always be called in the same order to prevent undesired behavior.

```jsx
const App = (props) => {
  // The following uses of `useState` are fine!
  const [variable, setVariable] = useState(0)
  const [variable2, setVariable2] = useState('Test')

  if ( variable > 10 ) {
    // You can't call `useState` from within an if statement! (not in the root of the functional component)
    const [test, setTest] = useState()
  }

  for (let i = 0; i < variable; i++) {
    // You can't call `useState` from within a loop!
    const [test2, setTest2] = useState()
  }
  
  
	// You can't call `useState` from within an arrow function inside the functional component!
  const dontDoThis = () => {
    const [test3, setTest3] = useState(50)
  }

  return (
    //...
  )
}
```

#### Other Common Errors

Circling back to our discussing about Event Handlers. When passing a handler as a property to an event such as the `onClick` event, always remember that event handlers **must** be a function or a reference to a function (think about the earlier section where we either wrapped our function inside an arrow function that returns our function or wrote our function to return the prepared function in order to make it behave like a reference).

This leads us to a common mistake of confusing expressions for functions. For example, when people are initially starting out with React, they might try to do something like

```jsx
<button onClick={objectCount = 5}> Press Me! </button>
```

If we tried to do something like this, we'll get thrown an error relating to being passed the wrong type of object as the handler for this property.

```js
index.js:1 Warning: Expected `onClick` listener to be a function, instead got...
```

Now, you might wonder, what would happen if we passed a statement as our event handler. Let's explore the following implementation

```jsx
<button onClick={console.log("I've Been Clicked!")}>
  Press Me!
</button>
```

With this, our React app compiles just fine and our message gets logged to the console right away; however, strangely subsequent presses of the button does not print anything to the console. Looking back to what was said earlier when we were talking about event handlers, the first console log is simply due to React executing the statement upon running the app in order to retrieve whatever the statement inside returns (before, our functions would return the prepared functions that we actually wanted to execute!). This means that the statement actually gets executed once and whatever gets returned from that statement is our actual event handler. This single execution is also why we ran into issues earlier, as this single execution would trigger a re-render which would set off an infinite loop. With this, our `console.log("I've Been Clicked!")` statement returns `undefined` upon its single execution. Thus, whenever we press our button, we actually are executing nothing.

To recap, with our above snippet of code, the `console.log` statement gets executed once when the component is rendered in order to assign the event handler as the object that gets returned from our statement execution. The statement returns `undefined` which is why nothing happens when we actually press the button since the event handler is actually just `undefined`.

Recall our discussion of currying and dealing with passing event handlers to components, the two ways of defining event handlers are ultimately up to personal preferences. You can either write the arrow function that returns the function call that you actually want in the event property, or you can write your helper function to return the prepared function.

## Passing Event Handlers to Child Components 

With this discussion of event handers, you might be wondering how components within components may call an event handler. For example, you might want a counter component that calls a function when the counter reaches 50 or when a button is pressed, that button within the counter component would have to have access to the event handler passed to the counter component.

We can pass props to child components simply by passing down the props of our existing component as follows

```jsx
const App = () => {
  
  const CustomButton = (props) => (
  	<button onClick={props.onClick}>{props.content}</button>
  )
  
  const printSomething = (value) => {
    console.log(value)
  }
  
  return (
    <div>
			<CustomButton onClick={() => printSomething(50)} content="Press Me!" />
    </div>
  )
}
```

Here, you'll see in our `CustomButton` component, we have received an event handler from the parent and passed it to a child `button` component through referencing the property via dot notation on the `props` object. Keep in mind, you must use the correct names when referencing the props of the parent component. For example, if we changed the property to `handleFunction` we would have to reference it as `props.handleFunction` as follows

```jsx
const CustomButton = (props) => (
  	<button onClick={props.handleFunction}>{props.content}</button>
  )

const App = () => {  
  const printSomething = (value) => {
    console.log(value)
  }
  
  return (
    <div>
			<CustomButton handleFunction={() => printSomething(50)} content="Press Me!" />
    </div>
  )
}
```

With this, we can continue to pass props down (or event handlers in this case) further and further down from child to child simply by referencing the `props` object of the parent component.



#### Common Mistake: Defining Components Within Components

When developing, you may think to define components within components. For example, it might make some sense to define the `CustomButton` component inside our `App` component as follows

```jsx
const App = () => {  
  
  const CustomButton = (props) => (
  	<button onClick={props.handleFunction}>{props.content}</button>
  )
  
  const printSomething = (value) => {
    console.log(value)
  }
  
  return (
    <div>
			<CustomButton handleFunction={() => printSomething(50)} content="Press Me!" />
    </div>
  )
}
```

People initially make this mistake with the idea of keeping things organized. If you're only using this custom component within this custom component, why not just define the component within the component that uses it? While, in this case, the application will appear to work, you should **never** do this as it can lead to unexpected issues with your components.

### Web Development Tips & Debugging

Typically, when developing, you should always leave your console open to monitor what's going on. Of course there are exceptions to this such as when you need all your screen real estate to test a layout, but this is just a general rule to follow. Additionally, developer workflows are highly aided when you have your code and your browser window open at the same time (especially when combined with things like live reload, which automatically refresh your browser window to allow you to see the changes you are making in real time). 

When dealing with debugging, there are a variety of tools that you will gradually learn over the years, but old school print debugging can always solve problems in a pinch. They allow you to see what's going on within your code with a few simple lines of code, so don't be afraid to write a few `console.log` statements in your code to see what's going wrong inside your code.

In order to make the most out of print debugging, be sure to not concatenate objects that you want to monitor using the `+` operator. Instead, log to the console by passing another argument as follows

```jsx
console.log('This is the History object: ', props.clicksArray)
```

Do **not** concatenate the object to the string using the `+` operator as follows

```jsx
console.log('This is the History object: ' + props.clicksArray)
```

Log statements that use a comma to pass another argument with the object we want to keep track of rather than just concatenating it to our string allow us to press an arrow in our console in order to see the contents of the object. When we just use the `+` operator to concatenate our object to the string, we'll get an unhelpful log message such as

```bash
This is the History object: [Object object]
```

This log message gives us no idea as to what the object actually contains, making it quite useless.

If you ever run into the need to step through your code and pause execution in order to see what's going on with your code at that point in time, you can write the `debugger` statement at the point which you want Chrome to pause execution. The debugger also gives us great features such as being able to step through our code line by line.

We can explore more of Chrome's features by stepping inside the `Sources` tab in our developer console. Here we can look at the variables that a given component has access to by looking under the `Scope` section on the side panel as well as add our own breakpoints without writing any more `debugger` statements.

Additionally, since Chrome, like Visual Studio Code, is heavily expandable, there are a plethora of extensions that add to a developer's experience. One great extension for debugging and working with React is the 'React Developer Tools' extension offered by Facebook (the developers of React) themselves. Upon installing this extension, you'll get access to a React tab in your developer tools, where you'll find all of your components laid out in an easy to read, tree format which you can use to then delve into your components and look at their state, etc.

As of August 15th, full support for React Hooks has been added to React DevTools, so you should not encounter any weird behaviors that were present in the past when trying to debug components that took advantage of React Hooks (such as not being able to see the contents of an array in the component's state from within the debugger).

### Programatically Rendering Collections

When dealing with long lists or dynamic content, it wouldn't make sense to "hard code" multiple instances of these components. This is where the `map` function comes into play. Let's take a note taking application as an example

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

const notes = [
  {
    id: 1,
    content: 'First Note',
    date: '2019-09-30T17:32:41.199Z',
    flagged: true
  },
  {
    id: 2,
    content: 'Second Note!',
    date: '2019-08-30T12:13:24.091Z',
    flagged: false
  },
  {
    id: 3,
    content: 'This is our third note.',
    date: '2019-08-30T12:20:14.998Z',
    flagged: true
  }
]

const App = ({ notes }) => {
  return (
    <div>
      <h1>Notes</h1>
      <ul>
        <li>{notes[0].content}</li>
        <li>{notes[1].content}</li>
        <li>{notes[2].content}</li>
      </ul>
    </div>
  )
}

ReactDOM.render(
  <App notes={notes} />,
  document.getElementById('root')
)
```

Here, you'll notice that we simply create three `li` elements and reference the content of our three different notes objects. However, we can take this a step forward and rewrite our `App` component to use the `map` function in order to create list items for every object within our `notes` array.

```jsx
const App = ({ notes }) => {
  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {notes.map(note => <li>Content: {note.content}</li>)}
      </ul>
    </div>
  )
}
```

This will generate the same result as the code snippet above.

We can go even further with regard to reusability by separating this `map` call into its own method.

```jsx
const App = ({ notes }) => {
  const noteList = () => notes.map(note => <li>Content: {note.content}</li>)

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {noteList()}
      </ul>
    </div>
  )
}
```

Everything seems good with this application, but we still have some work to do; if you take a look in your console, you'll find that three errors are thrown related to each `<li>` element that we have in our app.

![image-20191016195508067](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191016195508067.png)

This is because, when dealing with collections/lists/arrays, React needs to identify whcih items have changed, are added, are removed, etc. for various performance and internal state reasons. Thus, each list item should have a unique key that gives the element a stable identity. This is often done by assigning an ID to our data and then using that as the key. We assigned an ID to our data with the `id ` property of each 'note' item in our notes collection.

We can now update our `noteList` method to use our key values.

```jsx
const noteList = () => notes.map(note => <li key={note.id}>Content: {note.content}</li>)
```

The reason why our app works fine without assigning keys initially is that, by default, React uses indexes as keys as in the following example

```jsx
<li key={index}>Content: {note.content}</li>
```

Sometimes, in order to quickly resolve this error, some developers may manually set the key to the index via a solution similar to the one below

```jsx
const noteList = () => notes.map((note, i) => <li key={note.id}>Content: {note.content}</li>)
```

> Note how the map function passes the value of the index to `i` when called in this manner.

It is not recommended to use indexes for keys if the order of the list items changes as there are various performance issues and issues that arise with doing this as React isn't really able to track which element has a particular key as if an element is added or items are rearranged, the components will have different indexes. For example, if you had a list of input boxes with the keys assigned as their index and you typed something into the second input box (key value of 3) and then added an input box to the top of the list, the previously second input box would now become the third input box (key value of 4), but it wouldn't have the content that you typed in the text box as React believes the text content should belong to the input box with a key value of 3 (the one now above the desired input box).

### Componetizing

We've been exploring the idea of making things more manageable by splitting our project up into components since we've started talking about React, but you might have noticed that we have stuck everything in one file, posing potential managability issues. We can remedy this by using exports/imports as well as separate files for our components. Taking advantage of imports/exports and splitting our components into separate files will become increasingly necessary as your projects increase in complexity. 

#### Project Structures

Now, React projects range in terms of *project structure* (the layout of the files and folders for a given application), but typically the files relating to components will find themselves in a folder called 'components' in which that folder is found in a 'src' directory (source directory).

`create-react-app` will create a project structure similar to the one below

```tree
your-app-folder
├── build
├── node_modules
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
├── src
│   ├── App.css
│   ├── App.js
│   ├── App.test.js
│   ├── index.css
│   ├── index.js
│   ├── logo.svg
│   └── serviceWorker.js
├── .gitignore
├── package.json
└── README.md
```

Here, you can see the 'src' folder which contains our dynamic files/source code that will be processed by Webpack. You'll also notice the addition of a few new folders:

* 'public' is where static files typically reside. These files are not imported by the JavaScript application and will maintain their file name (files imported in 'src' will be given hashed names so that you always get the latest file rather than a cached vrsion of that file). Ultimately, this means that in production, the names for these files will be served up exactly as is; as a result, the client will usually cache these files and not fetch newer versions of it.
* 'node_modules' is where dependencies/packages installed by NPM or Yarn will go.
* 'build' is where the production-ready build is built to and typically will not exist until after you run `npm build` or `yard build`.

The 'src' folder in this generated project structure is not very pleasant to deal with, so it is common to see project structures with 'src' folders resembling the following structure

```tree
src
├── assets
│   └──images
│      └── logo.svg
├── components
│   └── app
│       ├── app.css
│       ├── app.js
│       └── app.test.js
├── index.css
├── index.js
└── service-worker.js
```

Here, you can see that we have introduced a components and assets folder. All files pertaining to our components, such as the one we will begin writing after this section, should fall in this components directory. The `app` component gets a sub-directory for all of its component files and so will the rest of your components as they grow to contain multiple files. If a component consists of just one file, it is commonly just put in the root of the /components directory.

If you're confused by any of this, don't worry, understanding of project stuctures will come with experience and will make more sense as you become more familiar with how React and other frameworks work.

### ES6 Modules (Imports and Exports)

Let's take the notes application from above and split it up. Commonly, components are declared as **ES6-modules** which are the modules that you import at the top of the page.

```jsx
import React from 'react'
```

is an example of a ES6-module import where the 'react' module is assignd to the variable `React`.  There are two types of exports:

#### Default Exports

A given file can only contain **one** default export. Default exports are written as follows

**./components/ExampleComponent.js**

```jsx
// Default exporting 'ExampleComponent'
const ExampleComponent = () => {}

export default ExampleComponent;

```

An example of importing a default export is as follows 

**./index.js**

```jsx
// Importing the above component in another file
import ICanUseAnyNameSinceItsADefaultExport from "./components/ExampleComponent"

```

> Note, we used a *relative path* to refer to our component and did not have to mention the '.js' file type in our path.

As a result of being able to have only one default export per file, you can use any name in the import of the file as illustrated by our random variable name in the above example.

In reality, the default export is actually just a standardized named input with the default name "default" allowing clients to understand what you're referring to without having to provide a name.

### Named Exports

Named exports are useful when we need to export multiple objects in a given file. The drawback being that we lose the ability to indepently name the variable that contains our imported object without special syntax. Named exports are written as follows

**./components/ExampleComponent.js**

```jsx
// Using named exports to export the two following components
export const ExampleComponent = () => {}
export const ExampleComponent2 = () => {}
```

An example of importing multiple named exports is as follows

**./index.js**

```jsx
// Importing multiple named exports at once
import { ExampleComponent, ExampleComponent2 } from "./components/ExampleComponent"
```

Notice how this is similar to the destructuring syntax in both form and function in that we are only taking certain objects from the source file via reference by name.

If you wanted to import a named export with your own custom name for the variable that it is stored in, you can write

```jsx
import { ExampleComponent as HereIsARandomName } from "./components/ExampleComponent"
```

Additionally, if you wanted to import every named export from a file, you can write

```jsx
import * from "./components/ExampleComponent"
```

If you wanted all of those named exports stored within one object, you could also write

```jsx
import * as ComponentsObject from "./components/ExampleComponent"
```

Where you would now reference the components as properties of this object, `ComponentsObject.ExampleComponent` and `ComponentsObject.ExampleComponent2`.

### Using Imports and Exports in React

We can write our first ES6-module default **export** for our note app as follows

**./components/Note.js**

```jsx
import React from 'react'

const Note = ({ note }) => {
  return (
    <li>{note.content}</li>
  )
}

export default Note
```

For every file that contains a React component, we must import React. Additionally, if we would like to actually use the component, we must export the component (it can be a named export or default export, just remember to import it correctly when the time comes). Now, we can use this `Note` component in **Note.js** by referencing it in our `App` component

**./App.js**

```jsx
import React from 'react'
import Note from './components/Note'

const App = ({ notes }) => {
  const noteList = () => notes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {noteList()}
      </ul>
    </div>
  )
}

export default App
```

Now, our `App` component makes use of the imported `Note` component that we exported earlier. We can now reference our overall `App` component in **index.js** as follows

**./index.js**

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

const notes = [
  // Here is where our note data comes from
]

ReactDOM.render(
  <App notes={notes} />,
  document.getElementById('root')
)
```

Notice how we imported the `App` component for use in our `index.js` file which renders this component to our root document element. This relationship is quite common in that components contain other components that all leads up to a component being rendered by the `ReactDOM.render()` function. This all serves as an example to how components would be split into separate files and used.

### Forms

Submitting data is a crucial aspect of almost any web application, with this we can use forms to submit input to our applications and components.

Let's take a look at our note taking application and figure out a way to add the ability for users to add new notes.

**./App.js**

```jsx
import React, { useState } from 'react'
import Note from './components/Note'

const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  
  const noteList = () => notes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {noteList()}
      </ul>
    </div>
  )
}

export default App
```

> Notice how the import for `useState` is a named import

In this new `App` component, we use the `useState` function in order to add state to our function. We initialize it with `props.notes` which contains the array of notes that we passed into the `App` component in our **index.js** file. Now that we have a way to keep our component state, we can create a way to add notes to our state

```jsx
import React, { useState } from 'react'
import Note from './components/Note'

const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  
  const noteList = () => notes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )
                                   
  const addNote = (event) => {
    event.preventDefault() // Prevents the page from being refreshed when we click the submit button
    console.log('Note form submitted', event.target)
  }

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {noteList()}
      </ul>
      <form onSubmit={addNote}>
      	<input />
        <button type="submit">Add Note</button>
      </form>
    </div>
  )
}

export default App
```

In this code snippet, you'll notice that we have written an `addNote` function which functions as an event handler upon the event that is the submission of the form that we have included in the JSX of our component. This function takes in a parameter called `event` which is the particular event that had triggered the call to this function. 

As a refresher, events are "things" that happen to HTML elements in which you can write event handlers that react upon these "things" that happen. There are plenty of DOM Events that are used to notify our event handlers that something has taken place. You can read more about these on a resource such as 'MDN web docs' if you're curious.

> If you've worked with HTML before, you might notice a difference with how we describe our events in React. React events are named using camelCase rather than just being undercase, so events such as `onclick="eventHandler()"` would be `onClick={eventHandler}` in React.

Circling back to the discussion of our above component, from the `event` parameter that this function takes in, it logs the target of the event `event.target` to the console so that we can see what triggered this event handler. In this case, when the submit button is pressed, `event.target` would be the form that contains the submit button defined in our component.

The next logical step would be to access the text that we put in our `input` element so that we can append it to our notes array. This brings about numerous solutions to this problem. The first solution involves **controlled components** which are components that maintain their own state and update it based upon user input.

We can accomplish this by adding a new piece of state that keeps track of the value of our input field as follows

```jsx
import React, { useState } from 'react'
import Note from './components/Note'

const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  
  const noteList = () => notes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )
                                   
  const addNote = (event) => {
    event.preventDefault() // Prevents the page from being refreshed when we click the submit button
    console.log('Note form submitted', event.target)
  }

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {noteList()}
      </ul>
      <form onSubmit={addNote}>
      	<input value={noteField} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  )
}

export default App
```

This is only a partial solution as this leaves us with a read-only field as our `input` element since upon every render, the value of our input field is set to `noteField`. Opening the console will also give you an error relating to providing a value to a form field without its own 'onChange' handler which leads to a field that is read-only.

To fix this, as our error message recommended to us, we can write an 'onChange' event handler that will sync any changes that the user makes to the input box with the component's state. This way, the change will be updated in the state so that the change is actually reflected upon the rendering of the component rather than the change being undone as a result of the `input`'s value being reset to the state's value every time.

```jsx
import React, { useState } from 'react'
import Note from './components/Note'

const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  
  const noteList = () => notes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )
                                   
  const addNote = (event) => {
    event.preventDefault() // Prevents the page from being refreshed when we click the submit button
    console.log('Note form submitted', event.target)
  }
  
  const handleInputChange = (event) => {
    console.log('Input form changed', event.target.value)
    setNoteField(event.target.value)
  }

  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {noteList()}
      </ul>
      <form onSubmit={addNote}>
      	<input 
          value={noteField} 
          onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  )
}

export default App
```

Let's take a look at what we added.

```jsx
const handleInputChange = (event) => {
  console.log('Input form changed', event.target.value)
  setNoteField(event.target.value)
}
```

First, we wrote an event handler that would update our `noteField` state to reflect the changes in our input box. We can get the value of the input box by retrieving the `value` property of the `target` of the `event` following the syntax that we had discussed earlier.

```jsx
<form onSubmit={addNote}>
      	<input 
          value={noteField} 
          onChange={handleInputChange} />
        <button type="submit">Add Note</button>
</form>
```

Then, we updated our form to include the 'onChange' event that would reference our `handleInputChange` handler. As the name would imply, on every change to our input box, the event handler passed to the 'onChange' property/event will be executed. As discussed earlier, our event handler function will receive the event object as a parameter to which we can reference our controlled `input` element via the `target` property and its corresponding value via the `value` property of the `target` property.

Another thing to note is that we don't need to prevent the default behavior of the `input` element by calling `event.preventDefault()` since (as of writing) input changes do not have a default action.

Since we included a `console.log` statement in our 'onChange' event handler, we can see how our handler is called as we make changes to our input box.

Now that we resolved this issue, `noteField` accurately reflects the value of the input box, so we can finish up our `addNote` function in order to add a note to our list of notes with the content set to the value of the input box.

```jsx
const addNote = (event) => {
  event.preventDefault() // Prevents the page from being refreshed when we click the submit button
  console.log('Note form submitted', event.target)
  
	const newNoteObject = {
    id: notes.length + 1, // May cause issues with duplicate ids if this array were to ever have elements deleted from it
    content: noteField,
    date: new Date().toISOString(),
    flagged: Math.random() > 0.75,
  }
  
  setNotes(notes.concat(newNoteObject))
  setNoteField('Enter a new note here...')
  
}
```

In the completion of this function, you'll notice a few things. We create the new note object by grabbing the current date through creating a new instance of `Date`, the content by looking at our component state in order to see the value of our input box, the important value by randomly generating a number and evaluating it which gives us a 25% chance of the note being marked as flagged, and an ID number that is generated by adding one to the length value of the array. If we were to have the feature of deleting notes, this ID solution would not work as we could run into issues with duplicate IDs given that a previous note was deleted, leading to the length calculation to providing us with an ID that is in use.

Then, we take this new note and concatenate it to our notes array. Recall that concat takes a new object and creates a brand new array with the new object appended. This allows us to follow the rule of never mutating the state directly but rather passing new objects with different memory locations. We then use `setNotes` to set our state to this new array.

After we update our `notes` piece of state, we set the `noteField` piece of state to our placeholder text. In practice, you would not set an input box's value with placeholder text but rather use a placeholder property.

### Filtering Elements

Let's make use of that `flagged` property in each one of our `notes` objects.

We can first track whether we're actually supposed to show all the notes or just the flagged notes by adding a piece of state using  `useState`.

```jsx
const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
```

We can now take advantage of the `filter` function in order to get only the notes that we want to display depending on our state.

```jsx
const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
  
  const filteredNotes = showAll ? notes : notes.filter(note => note.flagged)
  
  const noteList = () => filteredNotes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )
```

Here, you'll notice the addition of a `filteredNotes` constant which contains the entire notes array or only the flagged notes depending on whether `showAll` is true or not. We express this using a conditional (ternary) operator which allows us to express a logical branch in super simple terms with the following syntax

```jsx
const final = conditionToEvaluate ? caseIfTrue : caseIfFalse
```

Thus, if `conditionToEvaluate` is true, `final` will be set to the value of `caseIfTrue` and if it's false, it'll be set to `caseIfFalse`. This operator is usually used as a shortcut in place of `if` statements.

You can also see the usage of the `filter` method here where we filter for only notes where the `filtered` property is true. The syntax for the `filter` method typically resembles the following

```jsx
const newArray = originalArray.filter(function(arrayItem) {
  return condition; // If condition is true, then `arrayItem` will be included in the new array
})
```

What this does is take each individual item of the array, pass it into the function specified as an argument of the `filter` method in order to get a condition. If the condition is true, then the given element will be included in the creation of the new array.

Our implementation is quite similar, the difference being that we used an arrow function with shortened syntax as opposed to a function expression.

```jsx
const filteredNotes = notes.filter(note => note.flagged) // Assuming that `showAll` is always true
```

Here, you can see our arrow function where each element in `notes` is passed into the function as `note` to which the condition that is returned is the `flagged` property of the element.

Now, to finish everything up, let's add the ability to toggle our filter.

```jsx
import React, { useState } from 'react' 
import Note from './components/Note'

const App = (props) => {
  // ...
  return (
    <div>
      <h1>Notes</h1>
      <div>
        <button onClick={() => setShowAll(!showAll)}> {/* "Flips" the `showAll` state */}
          Show {showAll ? 'Flagged' : 'All' } {/* Changes button text depending on state */}
        </button>
      </div>
      <ul>
        {noteList()}
      </ul>
      <form onSubmit={addNote}>
      	<input 
          value={noteField} 
          onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  )
}
```

Here, we added a button with its `onClick` property set to toggle the `showAll` piece of state. Additionally, we put a conditional (ternary) operator in order to determine the text that was shown inside the button.

The code for our project thusfar is as follows (you can always refer to the project GitHub in order to see the source code for all of these projects)

**./index.js**

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

const notes = [
    {
      id: 1,
      content: 'First Note',
      date: '2019-09-30T17:32:41.199Z',
      flagged: true
    },
    {
      id: 2,
      content: 'Second Note!',
      date: '2019-08-30T12:13:24.091Z',
      flagged: false
    },
    {
      id: 3,
      content: 'This is our third note.',
      date: '2019-08-30T12:20:14.998Z',
      flagged: true
    }
  ]

ReactDOM.render(<App notes={notes} />, document.getElementById('root'));
```

**./App.js**

```jsx
import React, { useState } from 'react';
import Note from './components/Note'

const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
  
  const filteredNotes = showAll ? notes : notes.filter(note => note.flagged)

  const addNote = (event) => {
    event.preventDefault() // Prevents the page from being refreshed when we click the submit button
    console.log('Note form submitted', event.target)
    
    const newNoteObject = {
      id: notes.length + 1, // May cause issues with duplicate ids if this array were to ever have elements deleted from it
      content: noteField,
      date: new Date().toISOString(),
      flagged: Math.random() > 0.75,
    }
    
    setNotes(notes.concat(newNoteObject))
    setNoteField('Enter a new note here...')
  }

  const handleInputChange = (event) => {
    console.log('Input form changed', event.target.value)
    setNoteField(event.target.value)
  }
  
  const noteList = () => filteredNotes.map(note => 
  	<Note
      key={note.id}
			note={note}
    />
  )

  return (
    <div>
      <h1>Notes</h1>
      <div>
        <button onClick={() => setShowAll(!showAll)}> {/* "Flips" the `showAll` state */}
          Show {showAll ? 'Flagged' : 'All' } {/* Changes button text depending on state */}
        </button>
      </div>
      <ul>
        {noteList()}
      </ul>
      <form onSubmit={addNote}>
      	<input 
          value={noteField} 
          onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  )
}

export default App;
```

**./components/Note.js**

```jsx
import React from 'react'

const Note = ({ note }) => {
  return (
    <li>{note.content}</li>
  )
}

export default Note
```

Please also note that these files would be located in the `src` folder of most managed file structures (including the one set up by `create-react-app`)

## Introduction to the Backend

Let's start diving into the concept of retrieving information from a "backend" or server. At first, we're not going to set up a full blown server complete with a database, etc., we're just going to emulate retrieiving data from a server by using a tool designed to emulate this experience. To accomplish this, we're going to use a tool called 'JSON Server' which will act as our backend and serve up a simple JSON file.

To prepare, we're going to install `json-server` globally using npm by running the following command

```bash
npm install -g json-server
```

As of npm@5.2.0, you can use the `npx` command in order to invoke commands from packages that you don't have installed. We'll explore this command after some more setup.

Let's now create the JSON file that will function as our backend when we use `json-server` to run it. This file will reside in our root directory and will be named **db.json**

**db.json**

```jsx
{
  "notes": [
    {
      "id": 1,
      "content": "First Note",
      "date": "2019-09-30T17:32:41.199Z",
      "flagged": true
    },
    {
      "id": 2,
      "content": "Second Note!",
      "date": "2019-08-30T12:13:24.091Z",
      "flagged": false
    },
    {
      "id": 3,
      "content": "This is our third note.",
      "date": "2019-08-30T12:20:14.998Z",
      "flagged": true
    }
  ]
}
```

Now that we have our 'database', we can start up `json-server`. Here, we'll use the `npx` command in order to start it. When using the `npx` command, you don't actually have to install the package before using it; npm will figure out what you're trying to do based on the command and invoke the proper package from the2 npm registry. This comes with the advantage of not polluting your globals.

```bash
npx json-server --port 3001 --watch db.json
```

This command will use `json-server` to start a "server" that watches **db.json** and runs on port 3001 as create-react-app creates React projects that reserve port 3000.

It is also quite helpful to know how to delete global packages. If you had installed `json-server` before, it may be wise to run the following command

```bash
npm uninstall -g json-server
```

Now that everything is handled, we can visit the address of our server with a specific route in order to see our `notes` object.

Visiting **http://localhost:3001/notes** will display the `notes` object in **db.json**. With this, begin to conceptually understand that the frontend and the backend are separate entities that communicate with each other in order to achieve various tasks. For example, in this example, our frontend (our React app) will fetch the notes from our backend (the `json-server`) and submit notes to be stored in **db.json** by the backend.

### Fetching Data

You may recall `XMLHttpRequest` mentioned at the beginning of this book. You may also recall that this method of retrieving data is no longer recommended, however, it is still widely taught and is frequently used in legacy code.

```jsx
const xhttp = new XMLHttpRequest()

xhttp.onreadystatechange = function() {
  if (this.readyState == 4 && this.status == 200) {
    const data = JSON.parse(this.responseText) // Parses the response into an object
  }
}

xhttp.open('GET', '/data.json', true)
xhttp.send()
```

Here, we use a new `XMLHttpRequest` object to interact with our server, allowing us to retrieve data from our URL without refreshing the page. `XMHttpRequest()` is a constructor that initializes our new XMLHttpRequest via the `new` keyword. We then assign a function expression to the `onreadystatechange` property of the object which is an `EventHandler` that is called whenever the `readySatate` attribute changes. In other words, whenever the `xhttp` object changes in state, the `onreadystatechange` event handler is executed indicating that the response to the request has arrived.

Inside the function, we initially check the `readyState` and `status` properties of the response in order to see if we actually were returned the correct response. You'll learn more about what these status codes mean later in this book. We then get the actual content of our response (the JSON) by parsing the `responseText` property of our response.

You may have noticed that our code is *asynchronous* in this case, in that our code is not executed from "top to bottom" but rather is executed out of order with our event handler being executed only once our request has returned after being sent.

#### JavaScript's Asynchronous Model

JavaScript is a single-threaded language meaning that, in actuality, only one thing can happen at once. Thus, JavaScript runtimes or engines can only execute one statement at a time per thread (although, today, it is possible to run parallelized code on more than one thread wit h web workers; this does not make the event loop multithreaded).

Since JavaScript is single-threaded and can only make one 'thing' happen at once, we have to make sure that things that may take a while do not block interaction (such as making a web request like the one we did above but instead synchronously). This means that almost all I/O in JavaScript is non-blocking whether it be HTTP requests, reading/writing to the disk, fetching something from a database, etc. All of these actions would be done via the thread of execution providing a callback function that will be executed when this non-blocking operation is done so that the rest of the program can continue. 

For now, you can think of these concepts as having a list of chores: to wash your clothes, cook your dinner, and brush your teeth. The synchronous way of completing these tasks would be to put your clothes in the washer, wait until the washer is done, then start cooking your dinner, then wait until you finished cooking your dinner, then start brushing your teeth. 

The asynchronous way of doing this would be to start washing, then while the washer is running, you can go ahead and start cooking your dinner, after you start cooking your dinner, you can go ahead and brush your teeth. Then, you'll eventually get the callback function that will be executed for your dinner and washer (when those tasks are done, you'll have to tend to them), so a callback function for the washer might be to pick up the clothes, and a callback function for your dinner might be to put it on a plate and serve it to yourself. These callback functions are called once the task at hand is done. The event loop simply just keeps track of the order that everything is in and what needs to be executed.

# DESCRIBE THE EVENT LOOP IN DETAIL LATER

### The Axios Library

Now, let's take a look at using the *axios* library in order to fetch data from our server. It functions quite like fetch, a promise based function, but offers some additional features such as its wide browser support (for example, `fetch()` is not supported on IE11, but Axios functions just fine with its backwards compatability) and convenience features such as a timeout property, automatic JSON data transformation, and much more.

In order to use Axios, we must first install the package using npm. npm stands for 'node package manager' which is the main resource for external JavaScript libraries and is almost universally used among JavaScript projects.

npm makes use of a **package.json** file in the root of the project directory to which there are several properties relating to the package (your project) and the dependencies of your project. Running `npm init` in your project directory is a quick way to get started with creating a **package.json** file (**do not do this in your create-react-app directory**)

Here is an example of a **package.json** file.

```json
{
  "name": "project1",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.9.0",
    "react-dom": "^16.9.0",
    "react-scripts": "3.1.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
    
  },
  "eslintConfig": {
    "extends": "react-app"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}

```

Here, you can see the dependencies of the project as well as various other bits of information about our project. Don't worry about what everything means at the moment.

Although we could install Axios by adding it to our dependencies list in **package.json** and running `npm install`, it is considered better to just let npm manage that part by installing the package to our project directory by running

```bash
npm install axios --save
```

> Don't run npm commands anywhere except in the root of your project directory (unless otherwise specified when working with yarn workspaces and such later on). Running these commands may create other **package.json** files throughout your project and make your dependencies extremely hard to collect and use.

The `--save` flag instructs npm to include the package that we have just installed inside of our dependencies section in **package.json**. `--save` is now the default option when running `npm install`, so you will no longer have to specify this flag as it is implied. After running the above command, you'll find the `axios` package mentioned in the dependencies section.

```json
dependencies": {
    "axios": "^0.19.0",
// ...
```

We can now explore adding our own scripts to our **package.json** file in order to make development easier. In the scripts section, you'll find various scripts that `create-react-app` puts to make use of the `react-scripts` package.

```json
{
  // ... 
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "start": "json-server -p3001 db.json" // Executing this command would require that we have json-server installed whether it be globally or in our project
  },
  // ...
}
```

Since we need to have `json-server` installed for our 'start' script to run, we'll need to run

```bash
npm install json-server --save-dev
```

Which will install `json-server` as a development dependency (meaning that they are unneeded in production and will not be needed during production).

Now that we have everything set up, we can finally begin using Axios.

Let's first import and create two axios constants that are assigned with the promise that is returned from the `axios.get()` method. A promise is something that represents the eventual completion/failure of an asynchronous operation and the value that it returns.

```jsx
import axios from 'axios'

const examplePromise = axios.get('http://localhost:3001/notes')
console.log(examplePromise)

const examplePromise1 = axios.get('http://localhost:3001/randomtext')
console.log(examplePromise1)
```

A promise can have three different states

* **Pending**: Was not yet fulfilled nor rejected (initial state, the value has not been made available yet)
* **Fulfilled**: The async operation has completed successfully (the final value of the promise was successfully determined after the operation had completed)
* **Rejected**: The async operation has failed (the final value of the promise was unable to be determined)

Taking a look at your console to see the result of these `console.log` statements, you should find the following printed

![image-20191024205638403](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191024205638403.png)

> This image displays the Chrome Developer Tools Console in the native dark mode (supported as of Chrome 78). It may be beneficial to use Chrome in its dark mode depending on your environment.

Here, you can see how our first promise returned a promise status of being "resolved" as it was pointed to an actual valid URL, the notes route of our server. The second promise displays a status of "rejected" as we just gave it a random route to fetch something from.

If we wanted to get the result that our promise resolves to, we can provide a function to the `then` method of our promise.

```jsx
const examplePromise = axios.get('http://localhost:3001/notes')

examplePromise.then(response => {
  console.log(response)
})
```

What we're doing here is providing a callback function to the `then` method of the promise (which is included in the `Promise` prototype with the responsibility of appending fulfillment and rejection handlers to the given promise). This callback function with then be called when the operation has been completed/resolved and pass in the response object as the `response` parameter. This response object includes the typical response data that would come with an HTTP GET request response including the headers, status code, and the actual content/data.

The above code snippet would produce the following result in the console

![image-20191024214425345](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191024214425345.png)

The server returns the data as plain text, but we're able to parse the plaintext data into a JavaScript array since we know the data type as mentioned by the `content-type` property inside the `headers` property of our response as shown below.

![image-20191026173817715](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191026173817715.png)

In most cases, we wouldn't actually need to store the entire object in a constant or variable, so we can get rid of the assignment operator and variable declaration. We can also clean our code up by using a code formatting style in that we *chain* our methods

```jsx
axios
  .get('http://localhost:3001/notes')
  .then(response => {
    const notes = response.data
    console.log(notes)
  })
```

Now, let's take a look at an attempt to actually use the data returned from the server in our notes app from earlier.

**./index.js**

```jsx
import ReactDOM from 'react-dom'
import React from 'react'
import App from './App'

import axios from 'axios'

axios.get('http://localhost:3001/notes').then(response => {
  const notes = response.data
  ReactDOM.render(
    <App notes={notes} />,
    document.getElementById('root')
  )
})
```

Here, upon starting our app, we'll call the `get` method of `axios` and, with `then`, assign the callback function to get the data from the response and render our `App` component to the root element of our web page with the response passed in as the `notes` property.

This will go ahead and, upon resolving the promise involving our call to our server, render our `App` component with the `notes` property being assigned the data of our response, thus rendering our app with the notes. Notice how we're only fetching the data upon our app being loaded and rendering our entire React app inside the callback function; this could lead to some issues in the future, so let's explore another way to achieve our desired result

### Effect Hooks

Another type of hook that proves to be quite useful is the effect hook. Effect hooks allows for side effects (data fetching, setting up subscriptions, manually changing the DOM, etc.) to be performed in function components.

We'll be using the effect hook in our `App` component to fetch the notes data from our server, so we can get rid of our axios method call in **index.js** and the mention of a `notes` property in the usage of our `App` component.

**./index.js**

```jsx
import ReactDOM from 'react-dom'
import React from 'react'
import App from './App'

ReactDOM.render(<App />, document.getElementById('root'))
```

Here you can see how we simply render our `App` component to the root element of our document without doing anything else in our **index.js** file.

We can now go ahead and make our changes to **App.js** in order to enable the ability for our component to fetch its own data from the server.

**./App.js**

```jsx
import React, { useState, useEffect } from 'react' // Imported useEffect via Named Import (similar syntax to destructuring)
import Note from './components/Note'

import axios from 'axios' // Imported axios so we can request for the notes from our server

const App = (props) => {
  const [notes, setNotes] = useState([]) // Updated to not use the state passed into the component as we'll be fetching it from the server
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)

  // **Effect Hook**
  const effectHook = () => { 
    console.log('Reached Effect ')
    axios
      .get('http://localhost:3001/notes')
      .then(response => {
        console.log('The promise has been fulfilled :)')
        setNotes(response.data)
      })
  }

  useEffect(effectHook, [])
  console.log('Rendered', notes.length, 'notes')
  // **End Effect Hook**
  
  //...
```

Take a look at the comments in the code snippet above to get an idea of what we have added. Excluding the actual lines involving our implementing our effect hook, our changes involved simply importing `useEffect` from React, importing `axios` so we can make our server request, and updating our `useState` call to initialize our `notes` piece of state with an empty array.

Now, draw your attention to the lines involving our effect hook.

```jsx
  const effectHook = () => { 
    console.log('Reached Effect ')
    axios
      .get('http://localhost:3001/notes')
      .then(response => {
        console.log('The promise has been fulfilled :)')
        setNotes(response.data)
      })
  }

  useEffect(effectHook, [])
  console.log('Rendered', notes.length, 'notes')
```

Here, you'll notice how `useEffect` takes in a function to which we can use to fetch our data (this function will be run right after the component is rendered). `useEffect` also takes in a second parameter for which if you pass in an empty array ([]), like we did in this example, the effect will never be re-run as it tells React that it does not depend on any props or state (thus eliminating the need that it would need to be re-run since it's not dependent on anything that changes). If we were to omit this empty array as the second argument and use React's default, the effect would be run after every single completed render; that means, if something changes in your component to where it has to be re-rendered, this request would be made which may not be a good idea for performance reasons.

Taking a look at the console, assuming that everything works and the server is up, the following statements would be printed

![image-20191027003328132](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191027003328132.png)

* 'Rendered 0 notes' is printed first as the component is rendered for the first time without there being anything in our `notes` piece of state, simply because the function passed to `useEffect` had not been executed yet
* Then, after the components first render, the `effectHook` function is executed (which was passed into `useEffect`) which causes 'Reached Effect' to be printed.
* After our function had been executed, eventually (assuming that our promise is actually fulfilled), the fucntion that we passed into the `then()` method of our response will be executed with 'The promise has been fulfilled :)' being printed to the console.
* Finally, we set our `notes` piece of state to our response's data using the `setNotes` function that was supplied to us via the use of `useState`. This update to our state causes our component to re-render which leads ot the `console.log` statement involving our rendered components to be executed again; this time, however, our `notes` piece of state actually has three objects inside it, so 'Rendered 3   notes' is printed to the console.

> We still follow React's principle of not mutating data as our `response` object is new every time we make a request, so by setting our `notes` piece of state to `response.data`, we're setting it to a new object every single time.

Since `axios.get` returns a promise, and we're just accessing the `then` method of that property. We could also store that promise in a constant and use the `then` method to assign an event handler whose body handles our response as follows

```jsx
  const useEffect(() => { 
    console.log('Reached Effect ')
    
    const axiosEventHandler = response => { // Here's our handler for our response that used to be an anonymous arrow function defined in the `then` method's parameter list.
      console.log('The promise has been fulfilled :)')
      setNotes(response.data)
    }
    
    const axiosPromise = axios.get('http://localhost:3001/notes')
    axiosPromise.then(axiosEventHandler)
  }, [])
  
  console.log('Rendered', notes.length, 'notes')
```

Here, we made the changes to store the promise that axios returns in a constant so we can reference it later on when using our event handler defined in another constant to be set by the `then` method of the promise. This way of writing things, while it may allow for cleaner statements, is not usually the best way to go about these promises, the formatting of *chaining* usually works just fine. 

With regard to taking various approaches to solve problems, a good rule to follow involves writing your code to be easily read. Don't strive to have the "fanciest" solution with crazy one-liners, just write code in the most concise, human-readable form that you can. This makes maintaing your own projects and having other people work on them much easier; keep it stupid simple (KISS) is an acronym that you may want to have in mind when working on larger projects.

## Working With Data

### REST Terminology

If you've ever decided to look up what a REST (Representational State Transfer) API is supposed to do, you might have come across the four functions which use HTTP requests to: GET, PUT, POST, and DELETE data. REST APIs consist of data objects, called "resources" in REST terms, which all have unique addresses (the URL). For example, with the notes app that we've been working with, the URL **/notes/3** would return the note in the notes collection with an ID of 3 whereas a request to the URL **/notes** would return a resource collection of *all* the notes. Forward slashes (/) typically indicate hierarchical relationships. Additionally, is it good practice to not use trailing forward slashes (/ at the end of URLs) as it does not have any semantic value.

Resources are fetched via HTTP GET requests to the server. In our above app, we retrieved a resource collection containing all the notes through a HTTP GET request to our /notes URL. If we wanted to store a new note, instead of making a HTTP GET request, we would have to make an HTTP POST request. The following update to our `addNote` handler is an example of how we might make a HTTP POST request to a server

```jsx
const addNote = (event) => {
  event.preventDefault()
  const newNoteObject = {
    content: noteField,
    date: new Date(),
    flagged: Math.random() > 0.5,
  }

  axios
    .post('http://localhost:3001/notes', newNoteObject)
    .then(response => {
      console.log('Received Response', response)
      setNotes(notes.concat(response.data))
    })
  setNoteField('Enter a new note here...')
}
```

Here, you'll notice that we no longer create the `id` property in the object representing our new note since the server should keep track of IDs considering that the client does not have information pertaining to the IDs of the rest of the notes. We then added and event handler to handle the response from the server and assigned it to our promise using the `then` method. This event handler takes the response and logs it to the console; it also concatenates our new note that we received from the server in the `data` property of the `response` object to our array of notes, to which we use `setNotes` to update our `notes` piece of state and trigger a re-render. Upon submitting a note and receiving the response from the server, you should see the following in your console

![image-20191102175422148](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191102175422148.png)

Press the arrow next to 'Object' in order to see the `response` object that we logged to the console.

![image-20191102175321171](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191102175321171.png)

Within the `data` property of our `response` object, you'll find our actual note object that we sent to the server.

![image-20191102181918816](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191102181918816.png)

You might have noticed something interesting about our `response` object's `data` property. It seems that this object now has a `id` property when we did not specify it in our React app. This is because `json-server` adds an ID for us with a data type of `int`.

`json-server` requires that all data sent to it be sent with the `content-type` of `application/json` showing credence to its name of `json-server`. Taking a look at our network panel in our developer tools, we can take a look at our request and see if our `content-type` is correct

![image-20191102182414821](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191102182414821.png)

Looking under 'Request Headers', we can see that our `content-type` is indeed `application/json`. 

### Toggles (MAYBE REORGANIZE TOGGLES AND THE SHALLOW COPY SECTION)

Now, let's look at adding some more interactivity through adding the ability to toggle whether a note is flagged or not.

First, let's edit our `Note` component to reflect the ability to toggle whether a note if flagged

```jsx
const Note = ({ note, toggleFlagged }) => {
  return (
    <li>
    <input type="checkbox" checked={note.flagged} onChange={toggleFlagged} />
    {note.content}
    </li>
  )
}
```

Here, we simply added a checkbox whose `checked` property is set to the `flagged` property of the `note` passed in as a prop. Thus, if the `note` object's `flagged` property is true, the given list element for that `note` will have a checked checkbox and vice versa for a `note` object whose `flagged` property is false. We also set the `onChange` property of our checkbox to a `toggleFlagged` event handler that we'll implement next. The `onChange` event occurs whenever our checkbox is checked or unchecked.

Then, we'll update our `App` component to include our `toggleFlagged` event handler and its usage as follows

```jsx
const App = (props) => {
  const [notes, setNotes] = useState([]) // Updated to not use the state passed into the component as we'll be fetching it from the server
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)

  const toggleFlagged = id => {
    console.log(`To change ${id}'s flagged property`)
  }
  
  // ...
  
  const noteList = () => filteredNotes.map(note => 
  	<Note
      key={note.id}
      note={note}
      toggleFlagged={() => toggleFlagged(note.id)}
    />
  )
```

Here, you'll see our `toggleFlagged` event handler simply just outputs a statement involving the `id` parameter passed to it. Over in our `noteList` function, we set our `toggledFlagged` property to make use of our event handler where we also make use of function currying to 'create' a function that is specific to the `note`'s `id` passed to it. As a refresher of function currying, our function here, when executed, actually returns a unique function that pertains to a given note. For example, let's take note 4 whose `id` property is 4. When this function is executed, it creates a new function using the `id` property of this note, so that what is actually assigned to the `toggledFlagged` is a `toggleFlagged()` event handler that already has the parameters "baked in" as follows

```jsx
const toggleFlagged = () => {
    console.log(`To change 4's flagged property`)
  }
```

Another thing you might have noticed is the use of the *template strings* or *template literal* in our `toggleFlagged` event handler. With this, you can embed expressions into your strings through the use of simple syntax as follows ``` `Here is a template literal ${expression}` ```. Make note of how you must wrap your template literals in grave accents '`' rather than quotation marks. 

Now, when trying to toggle the checkboxes in our app, you'll notice that the specific event handler functions for each given `Note` component is executed, telling you that a given note's flagged property is to be chained. You'll also notice that our checkboxes aren't actually changing with regard to being checked or not. This is because they depend on the `Note`'s `flagged` property, and we actually aren't updating this property in our `toggleFlagged` event handler.

Let's update our event handler function to make a HTTP POST request to our server and update the given `note`'s `flagged` property.

```jsx
 const toggleFlagged = id => {
    const noteURL = `http://localhost:3001/notes/${id}`
    const referencedNote = notes.find(note => note.id === id)
    const updatedNote = { ...referencedNote, flagged: !referencedNote.flagged }

    axios.put(noteURL, updatedNote).then(response => {
      setNotes(notes.map(note => note.id === id ? response.data : note)) // Explained
      console.log(`Changed ${id}'s flagged property`)
    })
  }
```

Here, you'll see the combination of a few concepts that we have talked about previously. First, we reference the specific resource URL of the `note` that we want to update by creating a `noteURL` constant that makes use of the `id` parameter variable in order to access the `note` resource that we want. 

We then use the array's `find` method in order to find the note with the `id` that we're looking for  (we could've passed the entire `note` in as a parameter, but that makes for a messier solution when it comes to the comparisons that we'll do later). We then store the `note` with the desired `id` in the `referencedNote` constant.

From this, we create a new object that "copies" everything from the old `note` object using the spread syntax (...) except the `flagged` property from which we apply a logical negation `!` to toggle the property. We store this updated note object in the `updatedNote` constant.

> Remember: We should not mutate state in React; thus, we create a new constant that has our updated `note` rather than changing the `flagged` property of the `note` referenced by the `referencedNote` constant as such
>
> `referencedNote = !referencedNote.important`

### Shallow Copies vs Deep Copies

#### Shallow Copies/Clones

When we create a new `note` in this fashion with regard to using the spread syntax. We create a *shallow copy/clone*. Shallow copying only copies over field values over to our `updatedNote` object, so that the values of the properties of our new object still reference the same objects that `referencedNote` referenced. The only difference being that there is a new object that contains all of these fields and that we can make changes without mutating the original object by assigning new objects in place of these properties. In short, this means that if our original note had a reference to an object in one of its properties and we created a new note, `copiedNote` that utilized the spread syntax, the same property would reference the exact same object that the original note referenced. This is something to consider, because if you accidentally mutate the state of the referenced object, you mutate the object of the original note's property. Consider the following code snippet

```jsx
const obj1 = {
     enabled: true,
     size: {
          width:100,
          height:100
     }
}

const obj2 = {...obj1} // Shallow copy of obj1 using the spread operator

obj2.enabled = false // Updates the enabled property of the original object to false

obj1.size.width = 200 // Changes a property of an object that the original object refers to

console.log(obj2)
```

![image-20191104223920660](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191104223920660.png) Seeing the output of `console.log(obj2)` and diving into the `size` property of that object, you'll see how updating it via `obj1` still affected the object referenced by a property of `obj2` since both `obj1` and `obj2` refer to the exact same object with only the references being copied over to our new, distinct object (shallow copy). In the above snippet, you'll notice that the output of `obj2` uncovers that the `width` property of the `size` property of `obj2` is set to 200, even though we made the changes to `obj1`.

#### Deep Copies/Clones

Deep cloning involves going though all of an object's properties and cloning the objects and any objects that they might refer to in order to create a brand new object that does not share any references with the original object. This would be done in a recursive manner so that the cloning would continue from object reference to object reference until there are no more referred objects to be cloned. Unfortuneately, there is not a native way in JavaScript to deep clone an object which may prove useful in React where you may have to create objects that are entirely new. There are various custom solutions to this problem that you can find online. They all come with various benefits and disadvantages (for example, some do not support function object values or objects that contains a self reference to itself).

### Toggles Continuted

Going back to what our event handler does here, we make a HTTP PUT request to our backend server using `axios` telling it to replace the resource at `noteURL` with `updatedNote`.

Then, in our callback function from our HTTP PUT request, we use `setNotes()` to update our `notes` piece of state to a new array created by mapping over our original array and replacing the note that matches the `id` in our parameter variable with the `data` property of our response, essentially updating our state to reflect our backend and the change that we had just made.

```jsx
setNotes(notes.map(note => note.id === id ? response.data : note)) // Explained
```

Here, you can see how we map over the `notes` array in order to create a new array. Notice how we use a conditional (ternary) operator in order to see whether we should be replacing a given object with the updated object we had received in our response. If our `id` parameter variable (the ID of the note that we had just updated) matches the `id` of the note that we are currently mapping over in our `notes` array, then we will replace it with the updated object.

This method of dealing with updates also gives us the ability to verify that our change had indeed been reflected by our backend. If our state never updates, we know that we had not fulfilled our promise and that something has went wrong. We will soon go over how to handle these errors.

After this, we just log our changes to the console, explaining that we had successfully changed the `flagged` property of a given note.

### Organzing Backend Communication Services

An important computer programming principle to keep in mind while making these applications is the *single responsibility principle* which states that every single module, class, or function should have responsiblility over only a single part of the functionality of our overall application. Looking at our `App` component, you might have noticed that we're cramming quite a lot of code in this singular component and that it violates the single responsibility principle in that it also does the job of communicating with the backend. Let's look at moving our code involved with communicating with our backend server into its own module. 

In our project structure, we'll put all of our services in the directory **src/services**.

Within **src/services**, we'll create a **notes.js** file

**./services/notes.js**

```jsx
import axios from 'axios'
const baseUrl = 'http://localhost:3001/notes'

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

const create = newObject => {
  const request = axios.post(baseUrl, newObject)
  return request.then(response => response.data)
}

const update = (id, newObject) => {
  const request = axios.put(`${baseUrl}/${id}`, newObject)
  return request.then(response => response.data)
}

export default { 
  getAll: getAll, 
  create: create, 
  update: update 
}
```

Here, you can see how we used a default export to export an object that contains three functions defined earlier in this file (`getAll`, `create`, and `update`) with these functions representing the actions that we have used in our application already to communicate with our backend. 

Also, note how we're returning the `then` method of our original promise; this will allow us to write cleaner solutions later in our `App` component so that we won't have to reference the `data` property but rather just the returned object. We accomplish this by storing our promise in the `request` constant and then using the `then` method to assign a callback function that returns the `data` property of the `response` and then returning this new promise. When the promise returned by the `then()` method is resolved, it executes the respective handler function an returns a promise that resolves to the returned value. In short, this behaves as if we received only the data as a response and are returning that response instead of the entire response in a promise.

Now, in our `App` component, we can import this service and use it to talk to our backend rather than writing all of the functions within our `App` component as follows

```jsx
import noteService from './services/notes'

//...

const App = () => {
  
  //...
  
  useEffect(() => {
    noteService
      .getAll()
        .then(initialNotes => { // Promise Chaining, explained
        setNotes(initialNotes)
      })
  }, [])
  
  const toggleFlagged = id => {
    const referencedNote = notes.find(n => n.id === id)
    const updatedNote = { ...referencedNote, flagged: !referencedNote.flagged }

    noteService
    	.update(id, updatedNote)
    		.then(responseNote => { // Promise Chaining, explained
     	setNotes(notes.map(note => note.id === id ? updatedNote : note))
    })
  }
  
  const addNote = (event) => {
    event.preventDefault()
    const newNoteObject = {
      content: noteField,
      date: new Date(),
      flagged: Math.random() > 0.5,
    }
    
    noteService
    	.create(newNoteObject)
   			.then(responseNote => { // Promise Chaining, explained
      setNotes(notes.concat(responseNote))
      setNoteField('Enter a new note here...')
    })
  }

}
```

Here, we transitioned from making direct calls to `axios` methods to using our note service module to make requests for us and provide a layer of abstraction. You'll also notice instances of 'Promise Chaining' (demarcated by `// Promise Chaining, explained`). Here, we 'chain' another `then()` method to the previous promise returned by the functions in our notes function.

#### Promise Chaining Explained

The following diagram illustrates the act of a function creating an `axios` promise, calling its `then()` method in which upon resolving, the callback function that we have specified is executed, returning only the `response`'s data property rather than the entire response object in the form of a promise. When we call any of these functions in our `App` component, we get the `response`'s `data` property returned in the form of a promise that resolves to the `data`; we then 'chain' onto this and use its `then()` method that resolves and executes a callback that performs the appropriate operations with the returned data.

![IMG_2596AFD77492-1](/Users/josephsemrai/Downloads/IMG_2596AFD77492-1.jpeg)

To explain things as simply as possible and to provide an example, a call to a promise's `then()` method returns a promise, so that we can also call its  `then()` method and use the return value of its handler in our next promise. The following example illustrates how we access and use the return values of previous promises in subsequent promises

```js
new Promise(function(resolve, reject) {
  
  setTimeout(() => resolve(1), 1000) // (*)
  
}).then((result) => {

  console.log(result) // Alerts 1
  return result * 2;

}).then((result) => {

  console.log(result) // Alerts 2
  return result * 2

}).then((result) => {

  console.log(result) // Alerts 4
  return result * 2;
  
});
```

Here, the first promise resolves and in the handler that we specified in our first `then()` call, the number 1 is printed to the console. Then at the end of the handler, we return the result that our first promise resolved to multiplied by 2. The return is returned in the form of a promise that resolves to the value that we returned. As the returned value was a promise, we can make another `then()` call upon this new handler, only this new promise resolved to the value that was returned in handler of the promise mentioned one before in the chain. We then use this returned value in our new handler and return it again in the form of another promise so that we can use it in another `then() ` handler.

The topic of promises is quite confusing, so if you don't understand what's going on here, don't worry. There are plenty of resources online that go in depth with regard to the explanation of promises. Again. the 'You Do Not Know JavaScript' book series provides an excellent (but quite lengthy) explanation of promises.

#### Cleaner Code

Looking back to the code snippet, notice how we no longer have to specify the URL of our resources in our `App` component among other things, making the code look a lot cleaner.

We can also make our code a bit cleaner in respect to using shortened syntax. There are many instances in JavaScript where we can take advantage of the object-property value shorthand introduced in ES6. This object-property value shorthand allows us to take something like

```js
const year = 1995
const model = 'apple'

const car = {
  year: year,
  model: model
}
```

and turn it into

```js
const year = 1995
const model = 'apple'
const car = { year, model }
```

Thus, whenever both the property and variable reference name is the same, you can just write the name of the variable reference so that the variable reference will be stored in a property with the same name.

Applying this shorthand to our 'notes' service that we wrote earlier, we can now use the following in place of the object that we default exported as follows

**./services/notes.js**

```jsx
import axios from 'axios'
const baseUrl = 'http://localhost:3001/notes'

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

const create = newObject => {
  const request = axios.post(baseUrl, newObject)
  return request.then(response => response.data)
}

const update = (id, newObject) => {
  const request = axios.put(`${baseUrl}/${id}`, newObject)
  return request.then(response => response.data)
}

export default { getAll, create, update } // Taking advantage of the shorthand here
```

## Dealing With Promise Errors

If we ever encounter unexpected behavior in our web apps, we should be able to handle error messages gracefully and make the user aware that their request had not succeded. The console informing us of an error is simply not enough. Recall that there are three different 'states' of a promise, we have already discussed handling fulfilled promises, but now let's talk about when a given promise is  rejected.

One thing we can do here is terminate our `json-server` process so that when our React app tries to retrieve notes or interface with our backend in any way, it is unable to. Do this by closing the associated terminal/console window that you have ran the `npm run server` script in or by pressing `control` + `c` in the associated window.

Now that our backend server is offline, we can refresh the page of our React app and observe how it fails to retrieve our notes.

![image-20191103224302657](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191103224302657.png)

Here, you can see how we do not handle this error in any way. When our HTTP request fails, the promise that is returned is rejected. The only way a user would be able to observe this error is to notice unexpected behavior or check the console which is undesirable.

We can handle the rejection of a promise by providing our promise chain with a `catch()` method which returns a promise and deals with rejected cases.

```jsx
examplePromise.catch(onRejected) // where onRejected is a function that is called when the promise is rejected
```

In terms of an `axios` request, using the `catch()` method would probably look somewhat like the following

```jsx
axios
	.put('totallyfakeurl.com/will_fail')
	.then(response => {
  console.log("This actually didn't fail")
  })
	.catch(error => {
  console.log('This failed as expected!')
	})

```

Here, when the promise is rejected, the callback function provided to the `catch()` method in our promise chain will be execute with whatever return value that the promise had being passed in as `error`.

Putting this into practice, let's write a rejection handler for when we can't retrieve any notes from the server as follows

**./App.js**

```jsx
//...
const effectHook = () => {
    noteService
      .getAll()
        .then(initialNotes => { 
        setNotes(initialNotes)
      })
      .catch(error => {
        alert(`${error}: Something went wrong while trying to retrieve data from the server.`)
      })
  }
//...
```

Here, we simply added a `then()` method call to our promise chain involved with fetching all of our notes. The callback function we provided to the `then()` method call simply takes the error passed into it and alerts the user via the default alert box while also displaying a message.

![image-20191103225332897](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191103225332897.png)

When we lay everything out in our promise chain as such in the following code snippet, it becomes easier to see how the object returned by the promise previous in the chain is passed in as the argument to the next callback function in the chain.

```jsx
axios
  .put(`${baseUrl}/${id}`, newObject)
  .then(response => response.data)
  .then(responseNote => {
    // ...
  })
	.catch(error => {
    console.log(`${Error}: An error has occured.`)
  })
```

With this, you can now start writing error handlers for your promises that may get rejected.

### Styling in React

There are many ways to style your components in React. We'll dive into some of the more practical approaches later, but for now, we'll talk about styling in an old-fashioned sense in that we'll be adding CSS files to our application.

First, let's create a CSS file in the **/src** directory and name it **index.css**.

We can our CSS file to our application by simply importing it similarly to how we would import a module (we imported it in our **index.js** file so that these changes are reflected across all components)

**./index.js**

```jsx
import './index.css'
```

Now that we have our CSS file and it imported, we can start writing styles for our app! Let's first try styling our header with the text of 'Notes'

**./index.css**

```css
h1 {
    color: darkcyan; 
}
```

Now, as a quick overview of CSS, CSS consists of **selectors** (such as `h1`) and **declarations** (such as `color: darkcyan`). In the above snippet, the selector allows us to select all of the `h1` elements in our application and apply styling information to them. Then, within the **declaration block** (denoted by curly braces and contains declarations), we included a declaration tasked with setting the color of these elements to 'darkcyan'. CSS is very powerful; as such, there are many CSS selectors, declarations, and other features that this book will not cover. If you are interested in learning about how to style your web apps, I would suggest using the many resources available online for free to further your understanding. 

Using element selectors is usually not recommended as they would be applied to every element under an element selector; thus, class selectors and id selectors are used.

Class selectors allow us to give elements a class name to be targeted by via the `className` attribute. For example, we can target only the note components in our application by giving list element in our `Note` component a class name as follows

**./components/Note.js**

```jsx
import React from 'react'

const Note = ({ note, toggleFlagged }) => {
  return (
    <li className="mainNote">
    <input type="checkbox" checked={note.flagged} onChange={toggleFlagged} />
    {note.content}
    </li>
  )
}

export default Note
```

> Here, we use camelCase for our class names; it is common in the CSS world to use kebab-case (putting dashes between tokens). Most naming conventions are enabled by React, however, so it is up to your preferences and the style guide that you (and/or your team) are following.

Then, we can add the following snippet to our **index.css** file to target this new class as follows

```css
.mainNote {
  font-size: large;
  color: red
}
```

We can also add multiple classes to a given element as follows

**./components/Note.js**

```jsx
import React from 'react'

const Note = ({ note, toggleFlagged }) => {
  return (
    <li className="mainNote card">
    <input type="checkbox" checked={note.flagged} onChange={toggleFlagged} />
    {note.content}
    </li>
  )
}

export default Note
```

We can now add the CSS class to target this new class that we have added to the `li` element above

```css
.card {
    box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
    transition: 0.3s;
    border-radius: 5px;
    padding-top: 10px;
    padding-bottom: 10px;
    margin-bottom: 20px;
}
```

This card class gives elements that belong to this class the 'card' look. You can see the combination of these styles in the following image

![image-20191105001802321](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105001802321.png)

> Please note that this does not exemplify any design standards in any way. You can probably tell just from how bad this looks that this styling was just to demonstrate the concepts at hand.

With using class selectors instead of element selectors, we can create other `li` elements without them being affected by the styling information that we have specified, giving us increased flexibility.

As another example of styling, let's look at using styles in the creation of a new `Notification` component that could display messages such as the error messages that we were dealing with earlier.

First, let's create the actual notification component

**./components/Notification.js**

```jsx
import React from 'react'

const Notification = ({ message }) => {
    if (message === null) {
      return null
    }
  
    return (
      <div className="error">
        {message}
      </div>
    )
}

export default Notification
```

Then, we can add a piece of state to our application that keeps track of the current message that should be displayed by our notification component (as you can see in our `Notification` component, if the message parameter passed to it is `null` the component will not render in that it will return `null`). Also, take note of how we gave the `div` element that our component returns a class name of 'error'; this will be used to target this `div` via a class selector for styling.

**./App.js**

```jsx
import React, { useState, useEffect } from 'react'
import Note from './components/Note'
import Notification from './components/Notification' // Imported our new component

import axios from 'axios'

import noteService from './services/notes'

const App = (props) => {
  const [notes, setNotes] = useState([])
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
  const [notificationMessage, setNotificationMessage] = useState() // Our new piece of state that 'tracks' the message that should be displayed
```

Notice how we have imported our `Notification` component and utilized `useState()` in order to give us a piece of state, `notificationMessage`, which we can update with `setNotificationMessage`.

Let's now use our `Notification` component in the `div` that our `App` component returns

**./App.js**

```jsx
//...
	return (
    <div>
      <h1>Notes</h1>
      <Notification message={notificationMessage} /> {/* Notification Component */}
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          Show {showAll ? 'Flagged' : 'All' } 
        </button>
      </div>
      <ul>
        {noteList()}
      </ul>
      <form onSubmit={addNote}>
      	<input 
          value={noteField} 
          onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  )
}
```

We pass in our piece of state as the `message` property of our `Notification` component so that whenever we update our state involving `notificationMessage`, our `Notification` component will update appropriately.

Finally, we can replace our `alert()` call with a call to `setNotificationMessage` allowing us to display our error messages in our `Notification` component.

**./App.js**

```jsx
import React, { useState, useEffect } from 'react'
import Note from './components/Note'
import Notification from './components/Notification'

import axios from 'axios'

import noteService from './services/notes'

const App = (props) => {
  const [notes, setNotes] = useState([])
  const [noteField, setNoteField] = useState(
  	'Enter a new note here...'
  )
  const [showAll, setShowAll] = useState(true)
  const [notificationMessage, setNotificationMessage] = useState()
  
  //...

  const effectHook = () => {
    noteService
      .getAll()
        .then(initialNotes => {
        setNotes(initialNotes)
      })
      .catch(error => {
        setNotificationMessage(`${error}: Something went wrong while trying to retrieve data from the server.`) // Here we replace our alert() call with a call to setNotificationMessage()
      
      })
  }
  
  //...
```

We can now add styles that target the class of the `div` returned by our `Notification` component. Let's add the following to **./index.css**

```css
.error {
    margin: 10px 0px;
 		padding: 12px;
    color: #D8000C;
    background-color: #FFD2D2;
}
```

Now, if we terminate our `json-server` instance, we can observe our error message in the form of a `div` element that actually appears on our page with styling

![image-20191105174829318](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105174829318.png)

### Inline Styles

React also allows us to define styles from within our JavaScript code playing into the whole idea of having separate, logical components (having external CSS files would be incredibly hard to manage as your app scales).

When using inline styles, we provide React components/elements with a JavaScript object containing all of the CSS properties that we would want the component to be styled to. For example, taking our CSS properties for our error message:

```css
{
  margin: 10px;
  padding: 12px;
  color: #D8000C;
  background-color: #FFD2D2;
}
```

The following code snippet would illustrate converting those properties into a React inline style object

```jsx
{
  margin: '10px',
  padding: '12px',
  color: '#D8000C',
  background-color: '#FFD2D2'
}
```

Notice how the syntax becomes quite similar to a key/property-value pair object in JavaScript where the values are either strings or numbers.

We can now use inline styles in creating a `Separator` component as such

**./components/Separator.js**

```jsx
import React from 'react'

const Separator = () => {
  const separatorStyle = {
    color: 'white',
    padding: 15,
    fontStyle: 'bold',
    fontSize: 16,
    backgroundColor: '#05386B'
  }

  return (
    <div style={separatorStyle}>
      <p>This is the end of the above content</p>
    </div> 
  )
}

export default Separator
```

Notice how we have passed the inline style object, `separatorStyle`, into the `style` property of the `div` element.

Now, we can simply add our `Separator` component into the `div` element that our `App` component returns.

```jsx
import React, { useState, useEffect } from 'react'
import Note from './components/Note'
import Notification from './components/Notification'
import Separator from './components/Separator' // Separator Import

import axios from 'axios'

import noteService from './services/notes'

const App = (props) => {
  
 	//...
  
  return (
    <div>
      <h1>Notes</h1>
      
      <Notification message={notificationMessage} />
      
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          Show {showAll ? 'Flagged' : 'All' } 
        </button>
      </div>
      
      <ul>
        {noteList()}
      </ul>
      
      <form onSubmit={addNote}>
      	<input 
          value={noteField} 
          onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
      
      <Separator />
      
    </div>
  )
}
```

This updated code results in the following (assuming that we still have `json-server` terminated)

![image-20191105194723827](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105194723827.png)

## Express

Let's move beyond using `json-server` as our backend and work toward making our notes app interface with a 'Express backend'. Express is a web framework built on top of Node.js. It is primarily used to simplify development and the creation of secure, fast, and modular applications. As of writing, Express has over 46 thousand stars on GitHub, demonstrating how popular of a option it is for web developers.

### NPM and Node.js

> Since we are not using React in this project, we do not need to use `create-react-app` in order to create our project.

Let's first create a new project folder and run `npm init` from withtin the folder. Upon answering the intial prompts (as shown below), we will be presented with a template for our node application.

![image-20191105202150708](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105202150708.png)

Running this command yields us a **package.json** file in the root of our project directory. This JSON file contains the information that we have supplied the prompts with previously.

**/package.json**

```json
{
  "name": "1_project2_express",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

Since we want to use node to start our application with the entry point being **index.js** we can add the following to the object of the `scripts` property

**/package.json**

```jsx
{
  "name": "1_project2_express",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

Notice how we have assigned the `start` property with the value of `"node index.js"`; we will see this script in action in just a moment.

Now let's add a `console.log()` statement in our **index.js** file just so that we can observe something when we run our application.

**/index.js**

```js
console.log("Hello World, this is my Node application!")
```

Now, to run our node application, we can run the following command

```bash
node index.js
```

This runs the **index.js** file using the NodeJS JavaScript runtime. We can also use the script that we had defined earlier in **package.json** which runs the same command

```bash
npm start
```

Defining scripts like this becomes useful later on when we might want to use custom arguments or target different packages, allowing us to repeatedly do so with this simple command.

![image-20191105204050314](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105204050314.png)

### Web Servers in Node

Let's use Node's built-in web server module to convert this application into a web server.

**index.js**

```js
const http = require('http')

const webApp = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' })
  res.end('Hello World, this is my Node application!')
})

const port = 3001
webApp.listen(port)
console.log(`Web server running on port ${port}`)
```

Looking at this code, you'll see how we use Node's built-in web server via 'importing' it. 

```js
const http = require('http')
```

This syntax probably looks slightly odd to you. This is because most applications in Node.js still use CommonJS modules (since Node applications needed a solution for modules before the official specification with ES6 modules came out). Recently, with Node.js 12, support for ESM modules was released. Unfortuneately, it still is quite a pain to deal with over the support we have for it in browsers as (at the time of writing) you still have to enable the ``--experimental-modules` flag and use either the `.mjs` extension or add  `"type": "module"` to **package.json**. Because of this, we will still stick to using CommonJS modules.

Moving to the next block of code

```jsx
const webApp = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' })
  res.end('Hello World, this is my Node application!')
})
```

Here, we call the `createServer()` method of the `http` module that we have imported to create our server. This method takes in an event handler as a parameter which is called upon every single HTTP request to the server. 

This event handler takes in objects pertaining to the request, `req`, and the response, `res`. We then assign the response code and other header information (the content-type) through a call to the `writeHead()` method of `res`. 

Finally, we pass in a string argument to the `end()` method which adds the string to the response body. Our response will then be sent.

Moving to the next block of code

```jsx
const PORT = 3001
webApp.listen(PORT)
console.log(`Web server running on port ${PORT}`)
```

We assign the port that we want our server to listen on to the `PORT` constant. We make our HTTP server listen to requests on the port 3001 instead of 3000 as create-react-app typically runs the React development server on the port 3000. We then call the `listen()` method of the HTTP server that we stored in the `webApp` constant and pass in the port that we wish to use as a parameter. After this, we simply log that we have ran the web server listening on `PORT` successfully.

Visiting **localhost:3001** in our web browser yields the following result

![image-20191105213153375](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105213153375.png)

Here, you can start to see how this is similar to how `json-server` would serve up a response if we just returned a string in the `response-body`.

Let's make another stride toward our goal of writing our notes app in Node by hardcoding our notes array in **index.js** and returning it in our response.

**index.js**

```js
const http = require('http')

let notes = [
  {
      id: 1,
      content: 'First Note',
      date: '2019-09-30T17:32:41.199Z',
      flagged: true
  },
  {
      id: 2,
      content: 'Second Note!',
      date: '2019-08-30T12:13:24.091Z',
      flagged: false
  },
  {
      id: 3,
      content: 'This is our third note.',
      date: '2019-08-30T12:20:14.998Z',
      flagged: true
  }
]

const webApp = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' })
  res.end(JSON.stringify(notes))
})

const PORT = 3001
webApp.listen(PORT)
console.log(`Web server running on port ${PORT}`)
```

Notice how we changed the `Content-Type` of our response to `'application/json'`. Also, notice how we added our notes array to the response body by using the `JSON.stringify()` method which converts a JavaScript object/value into a JSON string that can be included in our response.

Now, let's terminate the server and start it up again (we will look into tools such as nodemon that will automatically do this for us upon changes to the source code later on).

Visiting **localhost:3001** yields the exact result that using `json-server` to serve up our resource collection did earlier (with the exception of routes)

**localhost:3001**

![image-20191105214436220](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105214436220.png)

### Getting Started With Express

Now, you might have thought that writing all of this code to handle basic HTTP requests is quite inconvienent. This is where Express comes in. There are plenty of libraries, including Express, that ease the development process by providing excellent functionality and interfaces to use when building web applications.

We can install Express to our project by running the following command

```bash
npm install express
```

#### Dependencies

This will install the `express` module to our **node_modules** folder (should only be kept locally, this will be discussed later)  and add `express` to our `dependencies` object in our **package.json** file.

**/package.json**

```json
{
  "name": "1_project2_express",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1" // Here is the addition of the express module
  }
}

```

Next to the `"express"` property/key value, you'll find the string value of `"^4.17.1"`; this string represents the version of Express. Packages follow the **semantic versioning spec** which involves:

* The first release of a package starting with 1.0.0 (For example, 1.0.0)
* Increments of the third digit being bug fixes that **are** backwards compatible. (For example, 1.0.1)
* Increments of the second digit being new features that **are** backwards compatible. (For example, 1.1.0)
* Increments of the first digit and resets of the second and third being major releases and changes that break backwards compatability (For example, 2.0.0)

You might also wonder what the caret in front of the version number means. The caret basically signifies that the version number must be **at least** `4.17.1` but can have higher second and third numbers (versions with bug fixes and new features that are backwards compatible).

Practically, all of this boils down to the following statement:

As long as the first number (major version number) does not change, version changes should be backwards compatible.

Diving into the **/node_modules** folder, you'll find that the installation of `express` also required the installation of plenty of other dependencies. These dependencies are called **transitive dependencies**.

![image-20191105215233352](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105215233352.png)

 Typically when using a service like GitHub to keep track of our source code or transferring our source code in any manner, we would **not** upload our node_modules folder as it takes up way too much space and can be installed with a simple command. To exclude our folder from being tracked and pushed by git, we could create a **.gitignore** file in the root of our project directory that contained the following two records

**/.gitignore**

```text
.gitignore
node_modules
```

If we cloned our project to another computer, we could install all of the dependencies of the project by simply running

```bash
npm install
```

If we wanted to update the dependencies of our project, we could simply run the following command

```bash
npm update
```

#### Serving Resources Depending On URL/Route

Going back to our Node application, we can make the following changes in order to utilize Express.

**/index.js**

```jsx
const http = require('http')
const express = require('express')

const app = express()

let notes = [
  {
      id: 1,
      content: 'First Note',
      date: '2019-09-30T17:32:41.199Z',
      flagged: true
  },
  {
      id: 2,
      content: 'Second Note!',
      date: '2019-08-30T12:13:24.091Z',
      flagged: false
  },
  {
      id: 3,
      content: 'This is our third note.',
      date: '2019-08-30T12:20:14.998Z',
      flagged: true
  }
]

app.get('/', (req, res) => {
  res.send('<h1>Hello World!</h1>')
})

app.get('/notes', (req, res) => {
  res.json(notes)
})

const PORT = 3001
app.listen(PORT, () => {
  console.log(`Express: Server running on port ${PORT}`)
})
```

Here, you'll notice quite a few changes. First, we import `express` (which turns out to be a function) and store it in a constant, `express`. We then create another constant, `app` and call the `express()` function which is used to create an express application.

Then, moving down to the calls to the `get()` method of our `app`, you'll notice two route definitions. Routing involves the mechanism in deciding which functionality to call based on the provided URL and parameters of a request (mapping and figuring out what to do/return based off of a URL).

The first route that we register involves HTTP GET requests to the route of our application (/).

```jsx
app.get('/', (req, res) => {
  res.send('<h1>Welcome to our Notes Application Backend!</h1>')
})
```

You'll notice that we provide an event handler as the second parameter to our `get()` method call. This event handler takes in a `request` parameter which contains the information pertaining to the HTTP request and a `response` parameter which allows us to define how to respond to the request. `req` is short for `request` and `res` is short for `request`.

In the above snippet, we utilize the `send()` method of the `res` object in order to respond to the specified HTTP request with a body that contains the string `"<h1>Welcome to our Notes Application Backend!</h1>"`. We do not have to manually set the `Content-Type` header as Express automatically does so after seeing that the parameter of the `send()` method was a string. Additionally, we do not need to set the status code of the response as it defaults to 200 automatically. 

In the following code block, we define an event handler that handles HTTP GET requests to the '/notes' path.

```jsx
app.get('/notes', (req, res) => {
  res.json(notes)
})
```

Here, we respond to the request by using the `json()` method of the `res` object which automatically sends the notes array that was passed to it as a JSON formatted string with all the appropriate header values.

Note that we did not have to manually specify header values or transform our data into the JSON format (JSON is a data structure standard that can be converted to a string to be transmitted over the HTTP protocol; it is textual and is language-independent; as a result, to transmit our data, we must convert our JavaScript objects into JSON). This is because Express automatically makes this transformation for us.

It is worth noting that, in JavaScript, `typeof json` yields `'string'` as a result, but it is incorrect to say that it is solely a string. As stated earlier, JSON is a standard that allows data to be transmitted through encoded or raw strings over the HTTP protocol.

Now that we have made our changes and discussed them, let's restart our application to see what has changed.

**localhost:3001**

![image-20191106005226777](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106005226777.png)

Visiting the root directory yields the message embedded in `h1` tags as we had defined earlier.

**localhost:3001**

![image-20191106005311403](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106005311403.png)

Visiting the '/notes' paths now gives us a resource collection including all of the notes. We now are able to target different paths via routes and provide different responses.

### Additional Tools

There are plenty of tools, such as `nodemon`, that assist us in the development process. `nodemon` automatically restarts our application in order to see our changes upon a change to the applciation's code. `nodemon` does this by 'watching' the files in the directory that `nodemon` was started in where it will automatically restart our node application if any files change.

Let's install `nodemon` as a **developer dependency** (packages only needed for the development of the application) since we only need to automatically restart our application when it is under active development.

```bash
npm install nodemon --save-dev
```

We can now see a new property, `devDependencies`, that contains our `nodemon` dependency in **package.json**.

**/package.json**

```json
{
  //...
  "dependencies": {
    "express": "^4.17.1"
  },
  "devDependencies": {
    "nodemon": "^1.19.4"
  }
}
```

Now, in order for `nodemon` to actually do its job, we need to start our application using `nodemon`. One way to do this is to refer to it directly via its path in the **node_modules** folder as follows

```bash
node_modules/.bin/nodemon index.js
```

This above command starts **index.js** using `nodemon` via executing it directly. 

Now, as this command is quite long, we can write a **npm scrip** for it (as we did earlier for `start`) in our **package.json** file

**/package.json**

```jsx
{
  //...
  "scripts": {
    "start": "node index.js",
    "watch": "nodemon index.js", // New script
    "test": "echo \"Error: no test specified\" && exit 1"
  },
}
```

Here, we do not have to specify the exact file path for `nodemon` in our script as npm is able to find the package as it knows to search for the package in that directory whereas our terminal is not.

Now, we can launch our server using `nodemon` by simply executing the following command

```bash
npm run watch
```

We have to add `run` to the command for custom scripts other than `start` and `test`.

![image-20191106205021398](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106205021398.png)

Running the command should yield the above result. You can see that `nodemon` is operating correctly in this case.

### REST in Express

Let's go a step further in covering the REST functionality that `json-server` covered so that we can fully transition our notes app over into Express.

First, let's talk about creating a REST interface so that our frontend can operate on individual note resources. We can do this by creating a route to fetch a single given resource.

When we used `json-server` we could fetch individual notes by fetching the notes path followed by the  ID number of the individual note. For example, if we wanted to interact with the note of ID 2, we would make a request to `host` +`/notes/2`.

We can add this functionality through the use of parameters in Express routes that allow us to take in values from the URL. Let's add the following to our application

**/index.js**

```js
app.get('/notes/:id', (req, res) => {
  const id = req.params.id // Takes in the parameter from the URL
  const note = notes.find(note => note.id === id) // Finds the note that matches the parameter
  res.json(note)
})
```

Parameters are defined with the above colon syntax, `:PARAMETER`. This above block will handle all requests made to the path `/notes/ANY_STRING` where ANY_STRING can be any string where this string will be passed in as a parameter. These parameters can be accessed via the `params` property of the `req` object. We then find the particular note with the ID passed in as a parameter using the `find()` array method that we have used earlier. Finally, we convert the `note` object into JSON and add it to our response.

Now, if you tried to execute your application and interact with an individual note, you'd find that our application isn't actually returning any notes. Some `console.log()` debugging later and you might find that our application simply isn't finding any matching notes, even if we provide the parameter with an ID that we do have. This is because we're using a strict equality comparison to check if we have matching IDs which takes into account the data type of the object. This leads to a failed comparison as our request parameter is a string while our `id`s are numbers. We can easily fix this by casting our parameter string into a number before making the comparison as follows

```js
app.get('/notes/:id', (req, res) => {
  const id = Number(req.params.id) // Takes in the parameter from the URL and casts it to a number
  const note = notes.find(note => note.id === id) // Finds the note that matches the parameter
  res.json(note)
})
```

With this, we can now successfully fetch an individual note.

Let's go over some other problems that we should fix with the above block. Recall that making a call to the request's `json()` method returns the response with a response code of `200` by default. This response code means that the response had succeeded, but with our above implementation, even if we don't find a note, we return the response code of `200` as we do not handle `notes.find()` returning nothing.

Let's add a way to handle an invalid ID (not finding the note) and returning the correct response

```js
app.get('/notes/:id', (req, res) => {
  const id = Number(req.params.id) // Takes in the parameter from the URL and casts it to a number
  const note = notes.find(note => note.id === id) // Finds the note that matches the parameter
  
  if (note) { // If we actually found a note
    res.json(note)
  } else {
    res.status(404).end() // Sets the appropriate status for not finding the note
  }
})
```

Here, we add a if-else statement that sends `note` and the correct response code, `200`, if we actually find a note. If we do not find the note, we set the status of the response to `404` in which this response code signifies that the specified resource was not found. We then use the `end()` method to respond to the request without sending data (other than the actual response information itself).

We are able to just write `if (note)` as JavaScript objects are **truthy** (they evaluate to true in comparisons) whereas `undefined` is falsy (evaluates to false in comparisons). This means that if the node did not exist, `notes.find()` would return nothing, leading to the value of `note` being `undefined`; thus, `note` would evaluate to `false` and the appropriate control would be executed.

On our front end, we can display a user-friendly interface for when we encounter error response codes (such as 404).

## CONTINUED IN WEB DEVELOPMENT 2

