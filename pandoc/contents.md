[TOC]

# Book Todo

- Add Images/Diagrams written with the iPad Pro for
  - Document Object Model
  - HTTP Requests (Submitting New Data Using HTTP POST and Forms)
- Add section about style guides, linting, etc.
- Replace mentions of 'I' in first person
- Explain what the global object is in the JavaScript section

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

| HTTP Methods | What does it do?                                                                                                                                                                                                        |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GET          | Requests data from a specified resource                                                                                                                                                                                 |
| POST         | Sends data to a server to update or create something                                                                                                                                                                    |
| PUT          | Sends data to a server to update or create smething, but is _idempotent_ (calling the same PUT request will always produce the same result, but multiple POST requests may produce duplicate results)                   |
| HEAD         | Pretty much identical to GET, but does not include the response body (does not return the contents of the response - often used for checking what a GET request will return before actuallying downloading it)          |
| DELETE       | Deletes the specified resources                                                                                                                                                                                         |
| PATCH        | Update/Changes a resource. PATCH requests only contain the changes to the resource, not the entire resource. Thus, they should be in a patch language (JSON Patch, XML Patch, etc.) (not safe or idempotent on its own) |
| OPTIONS      | Describes/Outlines the comuncation options for the resource that is being targeted.                                                                                                                                     |

These HTTP methods all have their own response codes which we'll talk about later. Also be aware that these methods all have different behaviors when cached, reloaded, called upon via the user pressing the back button, etc.

Now, visit a website and open up your developer tools panel (this book will reference the Chrome Development Tools) and click on the 'Network' tab. Upon refreshing or loading the page, you will see everything that the browser has requested. Click one of these requests to see more information about the request and response. Here you can see information such as the Content-Type of the page which indicates the format and type of the document returned.

If you take a look at the reponse tab, you can see what was actually returned from the server from that GET request.

Further along the requests, you may see references to images that may have been loaded after receiving a HTML response with tags (such as the <img> tag) telling the browser to request other resources. Upon receiving an initial response from the server, a browser will make additional requests to load any additional resources that may be specified in the HTML.

## Delving into Dynamic Web Pages

Here we'll take a look at a code snippet for a dynamic web page that makes use of the Express framework for Node.js. We'll talk more about this later, so it's okay if you have a vague idea of what's going on here.

```javascript
const getHtml = (listLength) => {
  return `
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
`;
};

app.get("/", (req, res) => {
  const page = getHtml(list.length);
  res.send(page);
});
```

Here is an example of a dynamic web page that we generate on the server side. We take advantage of this fact by passing the page some data to display that we would not be able to do traditionally with a static file. Upon a request to the server, the server will take note of the length of a list stored in memory and return a HTML file with the value inserted in place of `${listLength}`.

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
};
```

is an event handler defined on the `xhttp` object involved with the request that is called when the state of the object changes. Thus, if we assign a function to this event handler, it will be called when the state of this object changes.

When the request is completed, the state changes and calls this function. From within the function, we check if `readyState == 4 && this.status == 200` to see if the operation is complete (readyState 4) and if the HTTP status code of the response is 200 (HTTP Status OK).

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
li.setAttribute("class", "list-item");
todoList.setAttribute("id", "list");

document.getElementById("list").innerHTML = "example";
document.getElementsByClassName("list-item")[0].innerHTML = "example";
```

Here, `getElementById()` returns the one element with that ID of an element's child elements. The `document` object is the topmost node in the hierarchy of the DOM tree. As we are looking in the document's child elements, we are looking through all elements.

`getElementsByClassName()` returns a collection of an element's child elements with the specified class name. In the above example, we adjust the innerHTML of the first and only element in that collection. InnerHTML relates to the HTML content of an element (the content within element tags).

## Styling with CSS

In order to provide our web page styling information, we can take advantage of Cascading Style Sheets (CSS for short) which allow us to define the apperance of web pages using a markup language.

Within the head element of the HTML code, we can create a `<link>` tag to fetch a CSS file from a particular address as such.

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

You can write CSS rules to apply styling information to elements that contain these classes. Similarly, to use ID selectors, you would write `#class-name` as seen above with the 'list' ID.

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
  <input type="text" name="todo" />
  <br />
  <input type="submit" value="Save" />
</form>
```

Inserting this in our page gives us a text box and a "Save" button that allows the user to type in a todo and save it. Please note how the form tag takes in the attributes of `action` and `method` allowing us to specify the HTTP method and the address to point to.

When the button is clicked, the browser will send the values in the inputs (the textbox) to the server using a HTTP POST method directed at the "/new_todo" endpoint.

### ADD IMAGE HERE OF REQUESTS

Now, if you take a look at the Network tab when you submit this form, you might be surprised that it causes five HTTP requests. The reason for this being that the first event is the HTTP POST request to the server with the data to be submitted to which the server responds with the 302 HTTP status code that describes a URL redirect. This URL redirect tells the browser to do a new HTTP GET request to the address mentioned in the header of the response; as a result, the browser redirects to the location and loads the page bringing along more HTTP GET requests for each of the HTML, CSS, JavaScript, and raw data files (data.json that is fetched as part of the client side logic).

Now, how might we handle this HTTP POST request on the server side of things?

On the server, we would write a few lines of code to handle this route.

```javascript
app.post("/new_todo", (req, res) => {
  todos.push({
    content: req.body.todo,
    date: new Date(),
  });

  return res.redirect("/todos");
});
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
  <input type="text" name="todo" />
  <br />
  <input type="text" value="Save" />
</form>
```

Here, you'll notice that we got rid of the action and method attributes while also adding an ID to the form so that we can programatically select it using the DOM API.

Now, to send the form data the "SPA way", we must write some more JavaScript on the client side.

```javascript
var form = document.getElementById("todos_form");
form.onsubmit = function (e) {
  e.preventDefault(); // Overrides the default behavior of some browers that may lead to a redirect, etc.

  var todo = {
    content: e.target.elements[0].value, // Gets the first input of the form and it's value (text content)
    date: new Date(),
  };

  todos.push(todo); // Adds the todo to the todos list so we can use it to rerender the page with it
  e.target.elements[0].value = ""; // Clears the form text box
  redrawTodos(); // Function that will cause all of the todos to be rerendered
  sendToServer(todo); // Function that will send the new todo to be sent to the server using the XMLHttpRequest API
};
```

Notice how we are creating functions that provide reusable functionality to make our code easier to maintain and read, especially as our projects continue to grow. Now, we'll go over the implementation of the `sendToServer()` method.

```javascript
var sendToServer = function (todo) {
  var xhttpForPost = new XMLHttpRequest();

  xhttpForPost.open("POST", "/new_todo", true);
  xhttpForPost.setRequestHeader("Content-type", "application/json");
  xhttpForPost.send(JSON.stringify(todo));
};
```

In this method, we create a new `XMLHttpRequest` and set it's request header to match the type of data that we are sending. In this case, we are sending json objects, so we update the request header and use `JSON.stringify()` to convert our JavaScript `todo` object into a JSON string that can be sent to the server.

# JavaScript

This is just going to be a quick overview of some features of the JavaScript langauge and tips that will prove useful during development. This section will teach you everything that you will need to know to continue in this book, but by no means should be used to learn JavaScript. For the purpose of learning JavaScript, I heavily recommend the _You Don't Know JavaScript_ book series.

## JavaScript Background

You should have experience with some language prior to going through this book; if that language was Java, great, you'll recognize some of the syntax. Do not mistake JavaScript as being similar to Java, however, as they are completely different in how they are used. Some may try to use JavaScript as if it were Java with regard to design patterns and features, but this is highly disapproved.

Another thing to know about JavaScript is that it is often transpiled down to older versions as browsers may not support all of the new features in newer versions of JavaScript. This is accomplished by using transpilers such as Babel which transpile JavaScript written in newer versions to older versions that are more compatible. When we break into creating our first React project with `create-react-app`, Babel will automatiically be configed to transpile our code.

JavaScript is widely known as a client-side langauge that enables interactivity on websites, but it can also run on anything from embedded machines to servers. Node.js, a JavaScript runtime environment based off of Chrome's V8 engine, allows for JavaScript to be executed outside of a browser.

## Variables

## Arrays

You can store an array in a const as follows

```js
const exampleArray = [5, 0, 3];
```

If you would like to add an item to the array, you can use

```js
exampleArray.push(6);
```

If you would like to execute a function for each item in an array, you can iterate through items in an array by using the `forEach()` function as follows

```js
exampleArray.forEach((valueFromArray) => {
  console.log(valueFromArray); // prints 4 lines containing 5, 0, 3, and 6
});
```

Here, you can see something called an 'arrow function' being defined and passed to the `forEach()` function. Upon receiving this function as a parameter, `forEach()` will execute that function for each of the items in the array, making sure to pass the individual element (for the current iteration) as a parameter for arrow function that we passed to it.

### Immutability and Concat

Now, when we eventually visit React, you'll soon find that a common feature is the use of immutable data structures. Thus, if we were to ever need to add another item to an array, we wouldn't add it to the same array reference but rather create a new array with our new value by using the `concat()` function.

```js
const newArray = exampleArray.concat(5);
console.log(exampleArray); // [5, 0, 3] is printed
console.log(newArray); // [5, 0, 3, 5] is printed
```

Here, you can see how using the `contcat()` method did not mutate our original array in any way but rather created and returned a new array with the function argument added on to it.

### The Map Expression

Expanding on this concept of immutability, when we want to modify or pass every value from an array into a function, the `map()` function (which will be used quite extensively in React) returns the results of an interable (array, list, etc.) after applying a given function. What this means for our arrays is that we can create completely new arrays where the function given as a parameter is used to create them based off of our original array. This is done without mutating our original array as it returns a completely new array.

```js
const exampleMap = [5, 3, 1];

const newExample = exampleMap.map((value) => value * 2);
console.log(newExample); // [10, 6, 2] is printed
```

### Destructuring Assignment Expression

You can assign individual items in an array to variables by using **destructuring assignment**.

```js
const toBeDestructured = [5, 6, 7, 8, 9];

const [firstItem, secondItem, ...everythingElse] = toBeDestructured;

console.log(firstItem, secondItem); // 5, 6 is printed
console.log(everythingElse); // [7, 8, 9] is printed
```

Here, you can see that the variables `firstItem` and `secondItem` receive the first two integers of the array. Then the rest of the values get collected into a new array which is then assigned to the variable `everythingElse`. You can use any variable name and any amount of variables to get the values that you desire from the array.

## Objects and JavaScript Object Notation (JSON)

JavaScript contains objects that follow the JavaScript object notation. One way to create these objects is by using _object literals_ in which you literally write the object in your code as follows

```js
const examplePersonObject = {
  name: {
    first: "ExampleFirst",
    last: "ExampleLast",
  },
  jobs: ["Teacher", "Software Engineer"],
  workplace: "Microsoft",
};
```

Within an object you have things called properties. These are essentially the key values that you would use in order to get the values of the properties. The syntax is, as you can see above, `property: value`. The values of these properties can be anything ranging from integers to full blown objects.

### Referencing Properties

In order to use values within objects, we can use dot notation or brackets to reference these properties and receive these values back.

```js
console.log(examplePersonObject.workplace); // Using dot notation to reference the property, 'Microsoft' is printed
console.log(examplePersonObject["workplace"]); // Using brackets to reference the property, 'Microsoft' is printed
```

With brackets, we also gain the ability to reference properties based off of a variable instead of having to rely on string literals or hardcoded properties.

```js
const exampleField = "workplace";
console.log(examplePersonObject[exampleField]); // Using bracket notation and a variable to reference the property specified in the variable, 'Microsoft' is printed
```

In addition, we can use the concepts from above to dive deeper into our nested object. Recall that our `name` property within our `examplePersonObject` object has an object of its own. We can access properties of that object as follows

```js
const requestedName = "last";
console.log(examplePersonObject["name"][requestedName]); // Using bracket notation and a variable to reference the property within the object of the 'name' property, 'ExampleLast' is printed
console.log(examplePersonObject.name.first); // Using 'stacked' dot notation, we reference the name property and the 'first' property of the value of the name property leading to 'ExampleFirst' being printed
```

Here, you'll notice that we can 'stack' brackets or the use of dot notation multiple times to delve within objects. It may also be helpful to think of your objects as any file structure that you would find on your computer and the dot notation/brackets as the file path.

> Please note, you _cannot_ use variables with dot notation as of ES9. You also cannot reference properties with spaces or create new properties with spaces in their names when using dot notation as in the following example.

### Adding Properties

We can also add properties to any object after the fact by using brackets or dot notation as well.

```js
examplePersonObject.age = 204;
examplePersonObject["favorite color"] = "Blue";
```

Here, you'll notice, as mentioned earlier, we must use brackets to create the `'favorite color'` property as the name has a space in it.

Whenever we encounter these odd cases for property names, we can use string syntax when assigning our object literals as follows.

```js
const exampleObject2 = {
  "some random property": 25,
};
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
const referenceToTalk = exampleObject3.talk;
referenceToTalk("Example"); // Prints "Called from within an object with the argument: Example"
```

If you use the `this` keyword in any of your object methods, please note that `this` is defined based on how the method is called with these regular functions (explained in further detail below in the 'JavaScript Functions' section). Thus, if you call the function via reference, `this` becomes the global object and can lead to many errors.

For example, if we tried calling the `bindDemo` method from a reference, we will get an error

```js
const referenceToBindDemo = exampleObject3.bindDemo;
referenceToBindDemo(); // Error printed as there is no property in the global object
```

We can create new functions with `this` bound manually regardless of what is calling the method.

```js
const boundFunction = exampleObject3.bindDemo.bind(exampleObject3); // Bound to exampleObject3 so 'this' will work
```

With this, it is important to always keep track of `this` when writing JavaScript as it can lead to a plethora of issues.

## INSERT INFORMATION ABOUT OBJECT METHODS IN JAVASCRIPT

### Constructor Functions

The use of a constructor may remind you of classes coming from a Java background or any of the many other languages that make use of this mechanism. I wouldn't get my hopes up too much, though, as JavaScript doesn't really have classes that behave in the same was as these other langauges.

We can write constructors as follows

```js
function Person(first, last, age, occupation) {
  this.firstName = first;
  this.lastName = last;
  this.age = age;
  this.occupation = occupation;
  this.changeAge = function (age) {
    this.age = age;
  };
}
```

Here, you can see the constructor function for the creation of a `Person` object for which we can use to create many objects of the `Person` "type". You'll also notice the use of a keyword `this` which is a substitute for the new object which is to be created. Upon the creation of this object, this binds itself to that object and the appropriate properties are created.

We can also add object methods to our constructor which will be applied to the objects that are created. You'll also notice how we use `this` to reference the object that we created using the constructor so that we can just call `changeAge()` in order to update a given `Person` object's age.

> It is recommended to name constructor functions with an upper-case first letter.

```js
var examplePerson = new Person("Jake", "Flannn", 25, "Software Engineer");
```

The above is how we would go about creating a object of the `Person` "type". If we wanted to go ahead and change the age of our `Person` object, we can call the object method stored in one of its properties.

```js
examplePerson.changeAge(25);
```

Here, the object method mentioned in the constructor has bound `this` to the `examplePerson` object, to which we are able to reference the object's age by `this.age` in the method and update it.

### Class Syntax

With version ES6 came the introduction of the _class syntax_ which can help us structure objects and object-oriented classes. Now, this is where things might get familiar to those with a background in languages such as Java. Please keep in mind, although they behave similarly to Java objects, at their core, they are still JavaScript objects that are based on prototype inheritance.

#### Class Declarations

We can define a class by using a _class declaration_ as follows

```js
class Person {
  constructor(firstName, lastName, age, occupation) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
    this.occupation = occupation;
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
    this.height = dimension;
    this.width = dimension;
  }
  area() {
    return this.height * this.width;
  }
};

console.log(new Square(5).area()); // Prints 25
```

You'll notice that we omited the class name here, which you cannot do with class declarations. The constructor property is also optional for these classes with redeclaration being possible.

As mentioned earlier, the body of a class is automatically put into "strict mode".

#### Static Methods in Classes

When using the `static` keyword to define a method for a class, we create a static method which can be called without instantiating the class (creating an instance of the class/the object created after using the constructor to create a new object). These methods **cannot** be called by class instances.

```js
class Example {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  static add(a, b) {
    return a + b;
  }
}
const exampleInstance = new Example(5, 5);
console.log(Example.add(5, 5)); // Prints 10
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
  return a + b;
};

// To call our function
const sum = add(5, 5);
console.log(sum); // Prints 10
```

There are also shortcuts that we can take. If we just have one parameter that we need for our function, we can get rid of the parentheses around our parameters as follows

```js
const square = (d) => {
  return d * d;
};
```

Going even further, if there's just one expression inside the body of our function, we can keep everything in one line and get rid of the brackets.

```js
const square = (d) => d * d;
```

This shortened syntax often finds its use when dealing with the map method as it looks quite clean and can fit everything into one line.

```js
const toBeSquared = [5, 2, 3];
const squaredNumbers = toBeSquared.map((d) => d * d);
```

Arrow functions behave differently than regular functions. For example, they bind the keyword `this` to the lexical scope instead of binding it based on context. Usually, regular functions bind this to the object that calls the function (the context), while arrow functions take the value of `this` in the scope that the function was defined. In other words, arrow functions bind the value of `this` to the value of `this` where it was defined. This makes them unsuitable for being used as an object method.

The differences between arrow functions and regular functions is something that you should definitely read up on later in your career, but for now, just go with the general rule that you must using function expressions for object methods while arrow functions are often better when dealing with many callbacks or methods such as `map()`, `reduce()` or `forEach()`.

### Function Expressions and Declarations (Regular Functions)

Before arrow functions, there were function expressions and function declarations. When we give a regular function a name, it is called a function declaration.

```js
function add(a, b) {
  return a + b;
}
const sum = add(2, 9);
```

When we do not give a function a name, it is called a function expression:

```js
const add = function (a, b) {
  return a + b;
};
const sum = add(2, 9);
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
import React from "react";
import ReactDOM from "react-dom";

const App = () => (
  <div>
    <p>My first React app!</p>
  </div>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

You can also go ahead and delete `App.js`, `App.css`, `App.test.js`, `logo.svg`, and `serviceWorker.js` as they are not needed for this project.

## Components

Within our `index.js` file, you'll find an item called a React-component with the name of `App`.

There is also a line that renders this component into view at the bottom of the file as follows

```jsx
ReactDOM.render(<App />, document.getElementById("root"));
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
  );
};
```

Where we now explicitly return the component that we want to render. Upon doing this, we can now render dynamic content as we can execute any JavaScript statement before the return statement.

For example, we can now write

```jsx
const App = () => {
  const now = new Date();
  const a = 50;
  const b = 50;

  return (
    <div>
      <p>My first React clock! It is {now.toString()}</p>
      <p>
        {a} plus {b} is {a + b}
      </p>
    </div>
  );
};
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
  return React.createElement(
    "div",
    null,
    React.createElement(
      "p",
      null,
      "My first React clock! It is ",
      now.toString()
    ),
    React.createElement("p", null, a, " plus ", b, " is ", a + b)
  );
};
```

in pure JavaScript. If you were to write this component in pure JavaScript, you would have to write this.

JSX isn't exactly like HTML though, it is "XML-like", meaning that every tag written needs a closing tag.

For example, when writing HTML, you can write a line break element as follows

```html
<br />
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
  );
};

const App = () => {
  return (
    <div>
      <h1>Here's my React App!</h1>
      <ExampleComponent />
      <ExampleComponent />
      <ExampleComponent />
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

Here, you can see that we mentioned `<ExampleComponent />` multiple times in our `App` component. This allows us to break our project into many components designed for a purpose that can be reused in other components, allowing us to keep our project maintainable as it grows larger.

## Components With Data (Props)

Now, let's combine the concept of using JavaScript in our JSX and the modular structure of React. Doing so requires the use of _props_ which allow us to pass data to components.

For example, we can create the following components that pass data to each other and use it in a meaningful way.

```jsx
const Welcome = (props) => {
  return (
    <div>
      <p>
        Welcome, {props.title}
        {props.name}!
      </p>
    </div>
  );
};

const App = () => {
  const josephTitle = "Mr. ";
  const josephAge = 37;

  return (
    <div>
      <h1>Members: </h1>
      <Welcome name="Grace" title="Ms. " age={5 + 20} />
      <Welcome name="Joseph" title={josephTitle} age={josephAge} />
    </div>
  );
};
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
  const josephTitle = "Mr. ";
  const josephAge = 37;

  return (
    <>
      <h1>Members: </h1>
      <Welcome name="Grace" title="Ms. " age={5 + 20} />
      <Welcome name="Joseph" title={josephTitle} age={josephAge} />
    </>
  );
};
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
  );
};

const App = () => {
  const exampleName = "Jake";
  const exampleAge = 15;

  return (
    <div>
      <Greet name="Jason" age={36 + 10} />
      <Greet name={exampleName} age={exampleAge} />
    </div>
  );
};
```

### Component Helper Functions

Now, if we wanted to write a component that also wrote about how many days the person has been alive based off of their age, we could make use of what is called a **component helper function**.

```jsx
const Greet = (props) => {
  const ageDays = () => {
    return props.age * 365;
  };

  return (
    <div>
      <p>
        Welcome {props.name}, we have your age as: {props.age}
      </p>
      <p>You are close to {ageDays()} days old</p>
    </div>
  );
};
```

As the function is defined within the component function, it has access to all the props passed to the components. Thus, we do not need to pass the age prop to the function. We then use the function that we defined within the `Greet` component in order to get, rougly, the amount of days that the user has been alive for.

The result of this component should look like this:

![image-20190915002753027](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20190915002753027.png)

### Destructuring in JavaScript

We can streamline our references to our props by using something called destructuring to extract the values of an object's properties into separate variables without needing to reference them via brackets or dot notation.

```jsx
const Greet = (props) => {
  const { name, age } = props; // Assigns the name property to the variable 'name' and the age property to the variable 'age'

  const ageDays = () => {
    return age * 365;
  };

  return (
    <div>
      <p>
        Welcome {name}, we have your age as: {age}
      </p>
      <p>You are close to {ageDays()} days old</p>
    </div>
  );
};
```

Here, you'll see that we no longer have to reference these properties through dot notation/brackets upon the object, we can just simply reference the properties by their names as they have been assigned to constants when they were destructured.

In order to further streamline this, we can destructure the props into the variables in the argument list when we define our arrow function.

```jsx
const Greet = ({ name, age }) => {
  // The props are destructured in the argument list
  const ageDays = () => {
    return age * 365;
  };

  return (
    <div>
      <p>
        Welcome {name}, we have your age as: {age}
      </p>
      <p>You are close to {ageDays()} days old</p>
    </div>
  );
};
```

Here, instead of passing the entire `props` object into the component function, we destructure the props into the two variables `name` and `age` which are then passed into our component without passing in the entire `props` object. Thus, any references to the `props` object will lead to a 'Failed to compile' error. With this, it is important to remember that you will not be able to reference the `props` object and will have to destructure any new props that you pass into your component.

### Component Re-Renders

Let's bring some interactivity into our components.

```jsx
const App = (props) => {
  const { count } = props;
  return (
    <div>
      <p>The current count is: {count}</p>
    </div>
  );
};

let count = 1;

const refresh = () => {
  ReactDOM.render(<App count={count} />, document.getElementById("root"));
};

setInterval(() => {
  refresh();
  count += 1;
}, 1000);
```

Now, you'll see here that we have a function called `refresh` that will call upon `ReactDOM.render` to render our component to the root element in our document. Within this `refresh` function, we pass the current value of the variable count to our `App` component which will then be rendered and added to the root element.

Moving down to the bottom of our snippet, you'll also notice that we have a `setInterval` method that calls our `refresh` function and then adds 1 to the count variable every 1000 milliseconds (1 second). This effectively renders our component every second: first with the count variable as 1, second with the count variable as 2, and so on.

This actually isn't how things are typically done with React though, we shouldn't need to call `ReactDOM.render` in order to re-render components. We'll discuss better ways to re-render our components next.

### Stateful Components

React recently had the addition of 'React Hooks' in React 16.8. We'll now be taking advantage of React's state hook. React Hooks allow us to add state to function components wihtout converting it to a class.

Let's rewrite our above app to make use of `useState`.

```jsx
import React, { useState } from "react";
import ReactDOM from "react-dom";

const App = (props) => {
  const [count, setCounter] = useState(0);

  setTimeout(() => setCounter(count + 1), 1000);

  return (
    <div>
      <p>The current count is: {count}</p>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

There are quite a few new concepts and changes to this application, so let's take a look into what has changed and what these new functions do.

You'll first notice that we imported the `useState` function from 'react' in order to take advantage of React's state hook.

Next, draw your attention to the following line

```jsx
const [count, setCounter] = useState(0);
```

Here, we make use of the destructuring syntax to assign the elements of the array that `useState` returns to the constants `count` and `setCounter`. When you call `useState`, it creates this 'state variable' which will be preserved by React and not 'disappear' (due to the scope of our variables) when the function exists unlike normal variables. It also creates a function that we can use to update our state.

The first element in the array that is returned from the `useState` function call is the 'state variable'. The second element in the array that is returned from the call is our function that we can use to update our state. We then utilize array destucturing (the bracket syntax) in order to store the first element in the `count` constant and the update function `setCounter` constant.

One common point of confusion is simply just passing an expression to update the state in a `setTimeout` function or anywhere. You typically will have to create a function to change state and pass that to any function which will update your state. For example, in our application above, we wrote

```jsx
setTimeout(() => setCounter(count + 1), 1000);
```

In this snippet, we call the `setTimeout` function and pass it an arrow function (with shortcut syntax) to increment the counter variable in our state and with a timeout duration of 1000 milliseconds. Every time this state modifying function is called, React will go ahead and re-render the component. When dealing with functional components, the entire function's body will be re-executed upon a re-render. Upon calling `setCounter`, the `count` variable will be updated and we can reference it as such.

Just to provide some contrast, if we were to not use React Hooks and convert it to a class, our component would look something like this.

```jsx
import React from "react";
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
  }

  render() {
    return (
      <div>
        <p>The current count is {this.state.count}</p>
      </div>
    );
  }

  componentDidMount() {
    setInterval(() => {
      this.setState({
        count: this.state.count + 1,
      });
    }, 1000);
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
    console.log("A button has been clicked!");
  };

  render() {
    return <button onClick={this.handleClick}>Click Me!</button>;
  }
}
```

Here, we took the 'onClick' event handler and set the handler to our `handleClick` function in our class. Notice how follows the syntax of a property and is passed like a prop would be to our component.

Moving back to React Hooks and our functional component, here's an example of a more complex component that increments our `count` value upon a button press.

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => setCounter(count + 1)}>Increment</button>
    </div>
  );
};
```

Here, we have defined a function inside the onClick event handler which takes the current value of the `count` variable in our component's state, adds one to that value, and uses `setCounter` to set its value to the incremented value.

In the interest of keeping things more manageable, we can declare our functions outside of the event handler as helper functions similarly to what we did with our React component class above.

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  const incrementOne = () => setCounter(count + 1);

  return (
    <div>
      <div>{count}</div>
      <button onClick={incrementOne}>Increment</button>
    </div>
  );
};
```

With this, it is fair to say that event handlers (handlers) are just functions. However, when setting our `onClick` property/event handler to a function, we can make a fatal mistake very easily by calling the function rather than a reference to the function.

### React Re-Rendering Error

```jsx
<button onClick={incrementOne()}>Increment</button>
```

Writing this with the parentheses makes `incrementOne()` a **function call** which is not the reference to the function that it is expecting. This is an issue, because if we have a direct function call, the function will be executed upon the rendering of the component rather than just upon the event. This creates an infinite cycle as the `incrementOne()` function calls the update function `setCounter` which causes the component to be re-rendered executing the `incrementOne` function, starting the cycle once again.

Now, for this case, we could just have omitted the parentheses in order to make it a reference to the `incrementOne` constant once again, but what if we needed to pass parameters into the function?

Let's explore a function that sets the counter to a specific function.

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  const incrementOne = () => setCounter(count + 1);

  const setCounterValue = (value) => setCounter(value);

  return (
    <div>
      <div>{count}</div>
      <button onClick={incrementOne}>Increment</button>

      <button onClick={setCounterValue(0)}>Set Counter to 0</button>
    </div>
  );
};
```

Here, we have written a function that should take in a parameter and set the count variable to that given variable. We also have created a button that **incorrectly** makes a function call to `setCounterVariable()` rather than passing in a reference. This is because we only want to tell React the function to execute upon an event (in this case, the onClick event). When we pass a function call as the handler, we do not do this but rather execute the function leading to unwanted behavior. This will cause the same issue as described earlier, so how can we fix this and still retain the ability to pass in parameters to our function?

One quick and easy way to do this is to make the event handler a function that calls our `setCounterValue` function.

```jsx
<button onClick={() => setCounterValue(0)}>
```

Now, `setCounterValue(0)` will not be executed when our component is rendered. Wrapping our function call inside an arrow function makes our function call behave similarly to a function reference and fixes our problem. This works because this syntax in which we wrap our actual function call inside an arrow function with shortened syntax that **returns** the function that we want to execute as a result of its call. So, when React goes ahead and executes this function as a result of us mentioning a function as our event handler, it is essentially given the function that we want to execute upon the event without executing it.

Another way we can solve this problem is by defining a function that ends up returning a function.

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  const incrementOne = () => setCounter(count + 1);

  const setCounterValue = (value) => {
    return () => {
      setCounter(value);
    };
  };
  return (
    <div>
      <div>{count}</div>
      <button onClick={incrementOne}>Increment</button>

      <button onClick={setCounterValue(0)}>Set Counter to 0</button>
    </div>
  );
};
```

Here, we edited the `setCounterValue` function to take in the value and return a function that calls the `setCounter` function in order to update our state variable. We can now write the "function call" without wrapping it in an arrow function or encountering an error as this "function call" behaves like a reference to the function with the call only returning the function to be exectuted rather than executing the function. Thus, when the component renders, the `setCounter` function will not be called, avoiding the issue of infinite re-renders. This is similar to our above solution in that, upon a call, it returns the function that we actually want to execute upon an event which behaves like a reference to the function.

With this implementation, you might have also noticed that we now no longer have a need for a dedicated `incrementOne` helper function. We can replace its reference in the "Increment" button as seen in the following implementation of our component

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  const setCounterValue = (value) => {
    return () => {
      setCounter(value);
    };
  };
  return (
    <div>
      <div>{count}</div>
      <button onClick={setCounterValue(count + 1)}>Increment</button>

      <button onClick={setCounterValue(0)}>Set Counter to 0</button>
    </div>
  );
};
```

This new component makes use of the `setCounterValue` function to both increment and set our counter to 0.

When dealing with React, you may also see things called "double arrow functions", these functions are a result of the above scenarios in which the first function call "configures" the second function through definining its parameters in which we are left with a returned function call with the parameter "inserted". For example, with this function

```jsx
const setCounterValue = (value) => {
  return () => {
    setCounter(value);
  };
};
```

Calling `setCounterValue(100)` returns

```jsx
() => {
  setCounter(100);
};
```

as `value` is passed in to the second arrow function "configuring" it with the value that we desire so that we do not have to pass in parameters to this function and can just take the function as is. This illustrates how these functions return other functions that contain our parameters inserted so they can be used almost as a reference and are not executed upon their initial call (they must be called two times in order to get our final result, React does this by first "unwrapping" the desired function through calling it upon a render and then by calling the desired function upon the specified event).

Since we're dealing with arrow functions, we can take advantage of the shortened syntax that it offers in order to achieve the same result more elegantly.

The same function can be written as

```jsx
const setCounterValue = (value) => () => {
  setCounter(value);
};
```

since we can omit the return statement and the surrounding curly braces as there is only one expression for the surrounding/initial function.

We can take this one step further and omit the curly braces surrounding our returned function as we are only returning one expression once again.

This shortens our function that returns a function to

```jsx
const setCounterValue = (value) => () => setCounter(value);
```

which is a "double arrow function" with compact syntax. This way of dealing with functions is called "currying" and it involves nesting functions for each possible argument so that the inner components do not need to have all the arguments passed to them but rather get retain/get them through the natural closure created by the nested functions.

## Child Components

One of React's core principles is to write components that can be reused with maintainability in mind. Thus, it is recommended to keep components quite small to make things easier, especially as your project scales. Let's take a look at our application and split it up into three components that are logical with regard to what they are used for and achieve.

First, we can create a `Display` component that allows us to actually display the value of the counter. As you continue to familiarize yourself with React, you'll find that it is best to "lift the state up" as high as possible in the component hierarchy (i.e. keeping state accessible to all the components that reflect the same changing data through keeping the state above their closest common ancestor).

```jsx
const Display = ({ counter }) => <div>{counter}</div>;
```

Here, our `Display` component gets the application state passed ot it and is able to display it in terms of returning the appropriate `div` which contains everything to be shown on the page.

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  const setCounterValue = (value) => () => setCounter(value); // Making use of double arrow functions described earlier
  return (
    <div>
      <Display counter={count} />{" "}
      {/* Using our display component mentioned earlier */}
      <button onClick={setCounterValue(count + 1)}>Increment</button>
      <button onClick={setCounterValue(0)}>Set Counter to 0</button>
    </div>
  );
};
```

This new App component makes use of the Display component that we wrote earlier and passes the current `count` value from our state to the component.

We can also make a component for our buttons to simplify things a bit.

Here is the implementation of our `CounterButton` component.

```jsx
const CounterButton = ({ onClick, text }) => (
  <button onClick={onClick}>{text}</button>
);
```

Here's the updated `App` component that takes advantage of our `CounterButton` component.

```jsx
const App = (props) => {
  const [count, setCounter] = useState(0);

  const setCounterValue = (value) => () => setCounter(value); // Making use of double arrow functions described earlier
  return (
    <div>
      <Display counter={count} />{" "}
      {/* Using our display component mentioned earlier */}
      <CounterButton onClick={setCounterValue(count + 1)} text="Increment" />
      <CounterButton onClick={setCounterValue(0)} text="Set Counter to 0" />
    </div>
  );
};
```

After this, we have three components with the `App` component making use of the `CounterButton` component in order to update the `count` state variable and the `Display` component in order to display this variable.

> Note that since our `CounterButton` component is custom, we do not have to name our `onClick` attibute 'onClick' as we are not using the default DOM `<button>` element. Thus, we are free to give any name to `CounterButton`'s `onClick` property. However, please remember that it is conventional to name props that represent events 'on[Event]' and to name methods that handle the events 'handle[Event]'.

## Effective State Practices

As your applications get more complex, your projects will progressively contain more components with more variables in their states. Most of the time, you'll find that just adding more uses of the `useState` function to create more state variables will do.

Let's start with an application designed to track the amount of times that two buttons labeled with 'no' and 'yes' were pressed by a user.

```jsx
const App = (props) => {
  const [noCount, setNo] = useState(0);
  const [yesCount, setYes] = useState(0);

  return (
    <div>
      <div>
        {noCount}
        <button onClick={() => setNo(noCount + 1)}>No</button>
        <button onClick={() => setYes(yesCount + 1)}>Yes</button>
        {yesCount}
      </div>
    </div>
  );
};
```

Here, we used `useState` two times to create two state 'variables' and functions that we can use to update our state variables. Now, there aren't really any rigid rules toward what a component's state can consist of, if we wanted to organize more complex state situations, we could throw these variables into an object as properties and update this object whenever we wanted to update the state rather than the indivdual variables. We would achieve this as follows

```jsx
const App = (props) => {
  const [clicksObject, setClicks] = useState({
    yes: 0,
    no: 0,
  });

  const handleYesClick = () => {
    const updatedClicksObject = {
      yes: clicksObject.yes + 1,
      no: clicksObject.no,
    };
    setClicks(updatedClicksObject);
  };

  const handleNoClick = () => {
    const updatedClicksObject = {
      yes: clicksObject.yes,
      no: clicksObject.no + 1,
    };
    setClicks(updatedClicksObject);
  };

  return (
    <div>
      <div>
        {clicksObject.no}
        <button onClick={handleNoClick}>No</button>
        <button onClick={handleYesClick}>Yes</button>
        {clicksObject.yes}
      </div>
    </div>
  );
};
```

This achieves the same result as the application described earlier, just with a slightly different implementation. Let's break it down a bit.

In this application, we created the object `clicksObject` using the `useState` method. This means that our state consists of `clicksObject` and we can use the `setClicks` constant that we passed to `useState` as a function reference to update our state object. When we press the 'No' button, the `onClick` event is invoked and the `handleNoClick` handler function is called.

```jsx
const handleNoClick = () => {
  const newClicks = {
    yes: clicksObject.yes,
    no: clicksObject.no + 1,
  };
  setClicks(updatedClicksObject);
};
```

This function creates a new object that takes the properties from the `clicksObject` object in our application state and puts it into our new object, `updatedClicksObject`, we then add one to the `no` property in order to reflect the updated count. Finally, we call` setClicks` and pass in our new object to set as our application state, updating everything and showing our new count. The `handleYesClick` handler functions the same way, only referencing the `yes` property of the object instead to be incremented by one.

Now, later on, as your objects grow in size and your application state becomes complex, you may write needlessly long, complex code in order to handle simple state updates with this solution. Fortuneately, there's a better way to handle things in a neater, more organized fashion. We can make use of the _object spread_ syntax described earlier in order to copy the properties of our original object without having to specify each property as follows

```jsx
const handleYesClick = () => {
  const updatedClicksObject = {
    ...clicksObject,
    yes: clicksObject.yes + 1,
  };
  setClicks(updatedClicksObject);
};

const handleNoClick = () => {
  const updatedClicksObject = {
    ...clicksObject,
    no: clicksObject.no + 1,
  };
  setClicks(updatedClicksObject);
};
```

As a reminder, the object spread syntax essentially creates a copy of the object that follows the `...`, thus we can use it to create a copy of an object and update only the property that we need to by explicitly referencing it.

This implementation also allows us to add new properties to our state object without needing to add it to all of our handler functions as they would create a new object that would not reflect these additional properties.

Now, in the nature of making our code clean, consise, and easy to read, we can just create the object inside the `setClicks` call and pass it in directly as follows

```jsx
const handleYesClick = () =>
  setClicks({ ...clicksObject, yes: clicksObject.yes + 1 });

const handleNoClick = () =>
  setClicks({ ...clicksObject, no: clicksObject.no + 1 });
```

Here's a point in which many may encounter an issue when dealing with React. In pursuit of the most simple solution, one may try to just update the state directly as such

```jsx
const handleYesClick = () => {
  clicksObject.yes++;
};
```

or

```jsx
const handleYesClick = () => {
  clicksObject.yes++;
  setClicks(clicksObject);
};
```

Neither of these solutions are acceptable as **React does not allow for state to be mutated directly**. Objects in state should never be mutated (as in updating the object that is referenced by a variable or constant) but rather replaced with a new object. If you tried the above incorrect solutions for the handler methods, you would find that the counters would never be updated as React would not trigger a rerender due to it thinking that nothing had changed. React thinks nothing has changed, because even though the object was updated, the object reference has not changed (a new object was not passed), thus React is none the wiser to any updates.

> Storing our two separate variables in one object for our application state is not a good application design approach. This was only meant to serve as an example for the instances where data structures involving objects may be more appropriate for your component.

## Arrays in State

Let's take the above application and adapt it to use a single array to keep track of our 'Yes' and 'No' clicks. We'll use this array to display a history of all clicks.

```jsx
const App = (props) => {
  const [clicksObject, setClicks] = useState({
    yes: 0,
    no: 0,
  });

  const [clicksArray, setArray] = useState([]);

  const handleYesClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = {
      yes: clicksObject.yes + 1,
      no: clicksObject.no,
    };
    setClicks(updatedClicksObject);

    // Handles updating the clicksArray
    setArray(clicksArray.concat("Yes")); // We use concat here to create a new array and pass it to setArray as we cannot mutate the original array in React.
  };

  const handleNoClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = {
      yes: clicksObject.yes,
      no: clicksObject.no + 1,
    };
    setClicks(updatedClicksObject);

    // Handles updating the clicksArray
    setArray(clicksArray.concat("No"));
  };

  return (
    <div>
      <div>
        {clicksObject.no}
        <button onClick={handleNoClick}>No</button>
        <button onClick={handleYesClick}>Yes</button>
        {clicksObject.yes}
      </div>

      {/* Displays the contents of clicksArray */}
      <p>{clicksArray.join(" ")}</p>
    </div>
  );
};
```

Please take a look at the comments in our new application to see the changes that we have made in order to reflect this new array and our display of our click history. In order to create our history array, the `useState` hook was used to initialize our state with a blank array. We then use the updater method in order to update our array when one of the buttons is clicked as follows

```jsx
setArray(clicksArray.concat("Yes"));
```

Here, we use the `setArray` updater method to update our array in our component's state in order to reflect our new click.

Now, you might think about using the `push()` method in order to add the elements to the array. You should **never** do this; recall that we discussed the nature of React and how state must not be mutated. Mutating state directly almost always leads to undesired behavior and issues later down the line. While this may be hard to grasp initially, it is best to reinforce these practices early on.

Looking at how we display the content of our clicksArray (the history of our clicks), we simply use the `join()` method in order to take the contents of our array and convert it into a string with a space between every entry which is then rendered to the page.

### Dynamic Content

In this section, _conditionals_ will be introduced which allow us to conditionally (who would've thought?) render items in our component.

First, let's take the history section in our previous section and componentize it.

```jsx
const ClickHistory = (props) => {
  if (props.clicksArray.length === 0) {
    return <div>Please press one of the buttons!</div>;
  }

  return <div>History: {props.clicksArray.join(" ")}</div>;
};
```

Here, you'll notice that we introduced an `if` statement in our component. This allows us to **conditionally** render items in our component depending on what our expression evaluates to. In this case, if the array in our state has no items within it, the component will display a message to the user, 'Please press one of the buttons!', otherwise, we'll display the history of `clicksArray` to the user.

Let's now update the `App` component in order to take advantage of this new `ClickHistory` component.

```jsx
const App = (props) => {
  const [clicksObject, setClicks] = useState({
    yes: 0,
    no: 0,
  });

  const [clicksArray, setArray] = useState([]);

  const handleYesClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = {
      yes: clicksObject.yes + 1,
      no: clicksObject.no,
    };
    setClicks(updatedClicksObject);

    // Handles updating the clicksArray
    setArray(clicksArray.concat("Yes")); // We use concat here to create a new array and pass it to setArray as we cannot mutate the original array in React.
  };

  const handleNoClick = () => {
    // Handles updating the clicksObject
    const updatedClicksObject = {
      yes: clicksObject.yes,
      no: clicksObject.no + 1,
    };
    setClicks(updatedClicksObject);

    // Handles updating the clicksArray
    setArray(clicksArray.concat("No"));
  };

  return (
    <div>
      <div>
        {clicksObject.no}
        <button onClick={handleNoClick}>No</button>
        <button onClick={handleYesClick}>Yes</button>
        {clicksObject.yes}
      </div>

      {/* Displays the contents of clicksArray */}
      <ClickHistory clicksArray={clicksArray} />
    </div>
  );
};
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
<button onClick={(objectCount = 5)}> Press Me! </button>
```

If we tried to do something like this, we'll get thrown an error relating to being passed the wrong type of object as the handler for this property.

```js
index.js:1 Warning: Expected `onClick` listener to be a function, instead got...
```

Now, you might wonder, what would happen if we passed a statement as our event handler. Let's explore the following implementation

```jsx
<button onClick={console.log("I've Been Clicked!")}>Press Me!</button>
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
  );

  const printSomething = (value) => {
    console.log(value);
  };

  return (
    <div>
      <CustomButton onClick={() => printSomething(50)} content="Press Me!" />
    </div>
  );
};
```

Here, you'll see in our `CustomButton` component, we have received an event handler from the parent and passed it to a child `button` component through referencing the property via dot notation on the `props` object. Keep in mind, you must use the correct names when referencing the props of the parent component. For example, if we changed the property to `handleFunction` we would have to reference it as `props.handleFunction` as follows

```jsx
const CustomButton = (props) => (
  <button onClick={props.handleFunction}>{props.content}</button>
);

const App = () => {
  const printSomething = (value) => {
    console.log(value);
  };

  return (
    <div>
      <CustomButton
        handleFunction={() => printSomething(50)}
        content="Press Me!"
      />
    </div>
  );
};
```

With this, we can continue to pass props down (or event handlers in this case) further and further down from child to child simply by referencing the `props` object of the parent component.

#### Common Mistake: Defining Components Within Components

When developing, you may think to define components within components. For example, it might make some sense to define the `CustomButton` component inside our `App` component as follows

```jsx
const App = () => {
  const CustomButton = (props) => (
    <button onClick={props.handleFunction}>{props.content}</button>
  );

  const printSomething = (value) => {
    console.log(value);
  };

  return (
    <div>
      <CustomButton
        handleFunction={() => printSomething(50)}
        content="Press Me!"
      />
    </div>
  );
};
```

People initially make this mistake with the idea of keeping things organized. If you're only using this custom component within this custom component, why not just define the component within the component that uses it? While, in this case, the application will appear to work, you should **never** do this as it can lead to unexpected issues with your components.

### Web Development Tips & Debugging

Typically, when developing, you should always leave your console open to monitor what's going on. Of course there are exceptions to this such as when you need all your screen real estate to test a layout, but this is just a general rule to follow. Additionally, developer workflows are highly aided when you have your code and your browser window open at the same time (especially when combined with things like live reload, which automatically refresh your browser window to allow you to see the changes you are making in real time).

When dealing with debugging, there are a variety of tools that you will gradually learn over the years, but old school print debugging can always solve problems in a pinch. They allow you to see what's going on within your code with a few simple lines of code, so don't be afraid to write a few `console.log` statements in your code to see what's going wrong inside your code.

In order to make the most out of print debugging, be sure to not concatenate objects that you want to monitor using the `+` operator. Instead, log to the console by passing another argument as follows

```jsx
console.log("This is the History object: ", props.clicksArray);
```

Do **not** concatenate the object to the string using the `+` operator as follows

```jsx
console.log("This is the History object: " + props.clicksArray);
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
import React from "react";
import ReactDOM from "react-dom";

const notes = [
  {
    id: 1,
    content: "First Note",
    date: "2019-09-30T17:32:41.199Z",
    flagged: true,
  },
  {
    id: 2,
    content: "Second Note!",
    date: "2019-08-30T12:13:24.091Z",
    flagged: false,
  },
  {
    id: 3,
    content: "This is our third note.",
    date: "2019-08-30T12:20:14.998Z",
    flagged: true,
  },
];

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
  );
};

ReactDOM.render(<App notes={notes} />, document.getElementById("root"));
```

Here, you'll notice that we simply create three `li` elements and reference the content of our three different notes objects. However, we can take this a step forward and rewrite our `App` component to use the `map` function in order to create list items for every object within our `notes` array.

```jsx
const App = ({ notes }) => {
  return (
    <div>
      <h1>Notes</h1>
      <ul>
        {notes.map((note) => (
          <li>Content: {note.content}</li>
        ))}
      </ul>
    </div>
  );
};
```

This will generate the same result as the code snippet above.

We can go even further with regard to reusability by separating this `map` call into its own method.

```jsx
const App = ({ notes }) => {
  const noteList = () => notes.map((note) => <li>Content: {note.content}</li>);

  return (
    <div>
      <h1>Notes</h1>
      <ul>{noteList()}</ul>
    </div>
  );
};
```

Everything seems good with this application, but we still have some work to do; if you take a look in your console, you'll find that three errors are thrown related to each `<li>` element that we have in our app.

![image-20191016195508067](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191016195508067.png)

This is because, when dealing with collections/lists/arrays, React needs to identify whcih items have changed, are added, are removed, etc. for various performance and internal state reasons. Thus, each list item should have a unique key that gives the element a stable identity. This is often done by assigning an ID to our data and then using that as the key. We assigned an ID to our data with the `id ` property of each 'note' item in our notes collection.

We can now update our `noteList` method to use our key values.

```jsx
const noteList = () =>
  notes.map((note) => <li key={note.id}>Content: {note.content}</li>);
```

The reason why our app works fine without assigning keys initially is that, by default, React uses indexes as keys as in the following example

```jsx
<li key={index}>Content: {note.content}</li>
```

Sometimes, in order to quickly resolve this error, some developers may manually set the key to the index via a solution similar to the one below

```jsx
const noteList = () =>
  notes.map((note, i) => <li key={note.id}>Content: {note.content}</li>);
```

> Note how the map function passes the value of the index to `i` when called in this manner.

It is not recommended to use indexes for keys if the order of the list items changes as there are various performance issues and issues that arise with doing this as React isn't really able to track which element has a particular key as if an element is added or items are rearranged, the components will have different indexes. For example, if you had a list of input boxes with the keys assigned as their index and you typed something into the second input box (key value of 3) and then added an input box to the top of the list, the previously second input box would now become the third input box (key value of 4), but it wouldn't have the content that you typed in the text box as React believes the text content should belong to the input box with a key value of 3 (the one now above the desired input box).

### Componetizing

We've been exploring the idea of making things more manageable by splitting our project up into components since we've started talking about React, but you might have noticed that we have stuck everything in one file, posing potential managability issues. We can remedy this by using exports/imports as well as separate files for our components. Taking advantage of imports/exports and splitting our components into separate files will become increasingly necessary as your projects increase in complexity.

#### Project Structures

Now, React projects range in terms of _project structure_ (the layout of the files and folders for a given application), but typically the files relating to components will find themselves in a folder called 'components' in which that folder is found in a 'src' directory (source directory).

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

- 'public' is where static files typically reside. These files are not imported by the JavaScript application and will maintain their file name (files imported in 'src' will be given hashed names so that you always get the latest file rather than a cached vrsion of that file). Ultimately, this means that in production, the names for these files will be served up exactly as is; as a result, the client will usually cache these files and not fetch newer versions of it.
- 'node_modules' is where dependencies/packages installed by NPM or Yarn will go.
- 'build' is where the production-ready build is built to and typically will not exist until after you run `npm build` or `yard build`.

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
import React from "react";
```

is an example of a ES6-module import where the 'react' module is assignd to the variable `React`. There are two types of exports:

#### Default Exports

A given file can only contain **one** default export. Default exports are written as follows

**./components/ExampleComponent.js**

```jsx
// Default exporting 'ExampleComponent'
const ExampleComponent = () => {};

export default ExampleComponent;
```

An example of importing a default export is as follows

**./index.js**

```jsx
// Importing the above component in another file
import ICanUseAnyNameSinceItsADefaultExport from "./components/ExampleComponent";
```

> Note, we used a _relative path_ to refer to our component and did not have to mention the '.js' file type in our path.

As a result of being able to have only one default export per file, you can use any name in the import of the file as illustrated by our random variable name in the above example.

In reality, the default export is actually just a standardized named input with the default name "default" allowing clients to understand what you're referring to without having to provide a name.

### Named Exports

Named exports are useful when we need to export multiple objects in a given file. The drawback being that we lose the ability to indepently name the variable that contains our imported object without special syntax. Named exports are written as follows

**./components/ExampleComponent.js**

```jsx
// Using named exports to export the two following components
export const ExampleComponent = () => {};
export const ExampleComponent2 = () => {};
```

An example of importing multiple named exports is as follows

**./index.js**

```jsx
// Importing multiple named exports at once
import {
  ExampleComponent,
  ExampleComponent2,
} from "./components/ExampleComponent";
```

Notice how this is similar to the destructuring syntax in both form and function in that we are only taking certain objects from the source file via reference by name.

If you wanted to import a named export with your own custom name for the variable that it is stored in, you can write

```jsx
import { ExampleComponent as HereIsARandomName } from "./components/ExampleComponent";
```

Additionally, if you wanted to import every named export from a file, you can write

```jsx
import * from "./components/ExampleComponent"
```

If you wanted all of those named exports stored within one object, you could also write

```jsx
import * as ComponentsObject from "./components/ExampleComponent";
```

Where you would now reference the components as properties of this object, `ComponentsObject.ExampleComponent` and `ComponentsObject.ExampleComponent2`.

### Using Imports and Exports in React

We can write our first ES6-module default **export** for our note app as follows

**./components/Note.js**

```jsx
import React from "react";

const Note = ({ note }) => {
  return <li>{note.content}</li>;
};

export default Note;
```

For every file that contains a React component, we must import React. Additionally, if we would like to actually use the component, we must export the component (it can be a named export or default export, just remember to import it correctly when the time comes). Now, we can use this `Note` component in **Note.js** by referencing it in our `App` component

**./App.js**

```jsx
import React from "react";
import Note from "./components/Note";

const App = ({ notes }) => {
  const noteList = () =>
    notes.map((note) => <Note key={note.id} note={note} />);

  return (
    <div>
      <h1>Notes</h1>
      <ul>{noteList()}</ul>
    </div>
  );
};

export default App;
```

Now, our `App` component makes use of the imported `Note` component that we exported earlier. We can now reference our overall `App` component in **index.js** as follows

**./index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

const notes = [
  // Here is where our note data comes from
];

ReactDOM.render(<App notes={notes} />, document.getElementById("root"));
```

Notice how we imported the `App` component for use in our `index.js` file which renders this component to our root document element. This relationship is quite common in that components contain other components that all leads up to a component being rendered by the `ReactDOM.render()` function. This all serves as an example to how components would be split into separate files and used.

### Forms

Submitting data is a crucial aspect of almost any web application, with this we can use forms to submit input to our applications and components.

Let's take a look at our note taking application and figure out a way to add the ability for users to add new notes.

**./App.js**

```jsx
import React, { useState } from "react";
import Note from "./components/Note";

const App = (props) => {
  const [notes, setNotes] = useState(props.notes);

  const noteList = () =>
    notes.map((note) => <Note key={note.id} note={note} />);

  return (
    <div>
      <h1>Notes</h1>
      <ul>{noteList()}</ul>
    </div>
  );
};

export default App;
```

> Notice how the import for `useState` is a named import

In this new `App` component, we use the `useState` function in order to add state to our function. We initialize it with `props.notes` which contains the array of notes that we passed into the `App` component in our **index.js** file. Now that we have a way to keep our component state, we can create a way to add notes to our state

```jsx
import React, { useState } from "react";
import Note from "./components/Note";

const App = (props) => {
  const [notes, setNotes] = useState(props.notes);

  const noteList = () =>
    notes.map((note) => <Note key={note.id} note={note} />);

  const addNote = (event) => {
    event.preventDefault(); // Prevents the page from being refreshed when we click the submit button
    console.log("Note form submitted", event.target);
  };

  return (
    <div>
      <h1>Notes</h1>
      <ul>{noteList()}</ul>
      <form onSubmit={addNote}>
        <input />
        <button type="submit">Add Note</button>
      </form>
    </div>
  );
};

export default App;
```

In this code snippet, you'll notice that we have written an `addNote` function which functions as an event handler upon the event that is the submission of the form that we have included in the JSX of our component. This function takes in a parameter called `event` which is the particular event that had triggered the call to this function.

As a refresher, events are "things" that happen to HTML elements in which you can write event handlers that react upon these "things" that happen. There are plenty of DOM Events that are used to notify our event handlers that something has taken place. You can read more about these on a resource such as 'MDN web docs' if you're curious.

> If you've worked with HTML before, you might notice a difference with how we describe our events in React. React events are named using camelCase rather than just being undercase, so events such as `onclick="eventHandler()"` would be `onClick={eventHandler}` in React.

Circling back to the discussion of our above component, from the `event` parameter that this function takes in, it logs the target of the event `event.target` to the console so that we can see what triggered this event handler. In this case, when the submit button is pressed, `event.target` would be the form that contains the submit button defined in our component.

The next logical step would be to access the text that we put in our `input` element so that we can append it to our notes array. This brings about numerous solutions to this problem. The first solution involves **controlled components** which are components that maintain their own state and update it based upon user input.

We can accomplish this by adding a new piece of state that keeps track of the value of our input field as follows

```jsx
import React, { useState } from "react";
import Note from "./components/Note";

const App = (props) => {
  const [notes, setNotes] = useState(props.notes);
  const [noteField, setNoteField] = useState("Enter a new note here...");

  const noteList = () =>
    notes.map((note) => <Note key={note.id} note={note} />);

  const addNote = (event) => {
    event.preventDefault(); // Prevents the page from being refreshed when we click the submit button
    console.log("Note form submitted", event.target);
  };

  return (
    <div>
      <h1>Notes</h1>
      <ul>{noteList()}</ul>
      <form onSubmit={addNote}>
        <input value={noteField} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  );
};

export default App;
```

This is only a partial solution as this leaves us with a read-only field as our `input` element since upon every render, the value of our input field is set to `noteField`. Opening the console will also give you an error relating to providing a value to a form field without its own 'onChange' handler which leads to a field that is read-only.

To fix this, as our error message recommended to us, we can write an 'onChange' event handler that will sync any changes that the user makes to the input box with the component's state. This way, the change will be updated in the state so that the change is actually reflected upon the rendering of the component rather than the change being undone as a result of the `input`'s value being reset to the state's value every time.

```jsx
import React, { useState } from "react";
import Note from "./components/Note";

const App = (props) => {
  const [notes, setNotes] = useState(props.notes);
  const [noteField, setNoteField] = useState("Enter a new note here...");

  const noteList = () =>
    notes.map((note) => <Note key={note.id} note={note} />);

  const addNote = (event) => {
    event.preventDefault(); // Prevents the page from being refreshed when we click the submit button
    console.log("Note form submitted", event.target);
  };

  const handleInputChange = (event) => {
    console.log("Input form changed", event.target.value);
    setNoteField(event.target.value);
  };

  return (
    <div>
      <h1>Notes</h1>
      <ul>{noteList()}</ul>
      <form onSubmit={addNote}>
        <input value={noteField} onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  );
};

export default App;
```

Let's take a look at what we added.

```jsx
const handleInputChange = (event) => {
  console.log("Input form changed", event.target.value);
  setNoteField(event.target.value);
};
```

First, we wrote an event handler that would update our `noteField` state to reflect the changes in our input box. We can get the value of the input box by retrieving the `value` property of the `target` of the `event` following the syntax that we had discussed earlier.

```jsx
<form onSubmit={addNote}>
  <input value={noteField} onChange={handleInputChange} />
  <button type="submit">Add Note</button>
</form>
```

Then, we updated our form to include the 'onChange' event that would reference our `handleInputChange` handler. As the name would imply, on every change to our input box, the event handler passed to the 'onChange' property/event will be executed. As discussed earlier, our event handler function will receive the event object as a parameter to which we can reference our controlled `input` element via the `target` property and its corresponding value via the `value` property of the `target` property.

Another thing to note is that we don't need to prevent the default behavior of the `input` element by calling `event.preventDefault()` since (as of writing) input changes do not have a default action.

Since we included a `console.log` statement in our 'onChange' event handler, we can see how our handler is called as we make changes to our input box.

Now that we resolved this issue, `noteField` accurately reflects the value of the input box, so we can finish up our `addNote` function in order to add a note to our list of notes with the content set to the value of the input box.

```jsx
const addNote = (event) => {
  event.preventDefault(); // Prevents the page from being refreshed when we click the submit button
  console.log("Note form submitted", event.target);

  const newNoteObject = {
    id: notes.length + 1, // May cause issues with duplicate ids if this array were to ever have elements deleted from it
    content: noteField,
    date: new Date().toISOString(),
    flagged: Math.random() > 0.75,
  };

  setNotes(notes.concat(newNoteObject));
  setNoteField("Enter a new note here...");
};
```

In the completion of this function, you'll notice a few things. We create the new note object by grabbing the current date through creating a new instance of `Date`, the content by looking at our component state in order to see the value of our input box, the important value by randomly generating a number and evaluating it which gives us a 25% chance of the note being marked as flagged, and an ID number that is generated by adding one to the length value of the array. If we were to have the feature of deleting notes, this ID solution would not work as we could run into issues with duplicate IDs given that a previous note was deleted, leading to the length calculation to providing us with an ID that is in use.

Then, we take this new note and concatenate it to our notes array. Recall that concat takes a new object and creates a brand new array with the new object appended. This allows us to follow the rule of never mutating the state directly but rather passing new objects with different memory locations. We then use `setNotes` to set our state to this new array.

After we update our `notes` piece of state, we set the `noteField` piece of state to our placeholder text. In practice, you would not set an input box's value with placeholder text but rather use a placeholder property.

### Filtering Elements

Let's make use of that `flagged` property in each one of our `notes` objects.

We can first track whether we're actually supposed to show all the notes or just the flagged notes by adding a piece of state using `useState`.

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
const final = conditionToEvaluate ? caseIfTrue : caseIfFalse;
```

Thus, if `conditionToEvaluate` is true, `final` will be set to the value of `caseIfTrue` and if it's false, it'll be set to `caseIfFalse`. This operator is usually used as a shortcut in place of `if` statements.

You can also see the usage of the `filter` method here where we filter for only notes where the `filtered` property is true. The syntax for the `filter` method typically resembles the following

```jsx
const newArray = originalArray.filter(function (arrayItem) {
  return condition; // If condition is true, then `arrayItem` will be included in the new array
});
```

What this does is take each individual item of the array, pass it into the function specified as an argument of the `filter` method in order to get a condition. If the condition is true, then the given element will be included in the creation of the new array.

Our implementation is quite similar, the difference being that we used an arrow function with shortened syntax as opposed to a function expression.

```jsx
const filteredNotes = notes.filter((note) => note.flagged); // Assuming that `showAll` is always true
```

Here, you can see our arrow function where each element in `notes` is passed into the function as `note` to which the condition that is returned is the `flagged` property of the element.

Now, to finish everything up, let's add the ability to toggle our filter.

```jsx
import React, { useState } from "react";
import Note from "./components/Note";

const App = (props) => {
  // ...
  return (
    <div>
      <h1>Notes</h1>
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          {" "}
          {/* "Flips" the `showAll` state */}
          Show {showAll ? "Flagged" : "All"}{" "}
          {/* Changes button text depending on state */}
        </button>
      </div>
      <ul>{noteList()}</ul>
      <form onSubmit={addNote}>
        <input value={noteField} onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  );
};
```

Here, we added a button with its `onClick` property set to toggle the `showAll` piece of state. Additionally, we put a conditional (ternary) operator in order to determine the text that was shown inside the button.

The code for our project thusfar is as follows (you can always refer to the project GitHub in order to see the source code for all of these projects)

**./index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

const notes = [
  {
    id: 1,
    content: "First Note",
    date: "2019-09-30T17:32:41.199Z",
    flagged: true,
  },
  {
    id: 2,
    content: "Second Note!",
    date: "2019-08-30T12:13:24.091Z",
    flagged: false,
  },
  {
    id: 3,
    content: "This is our third note.",
    date: "2019-08-30T12:20:14.998Z",
    flagged: true,
  },
];

ReactDOM.render(<App notes={notes} />, document.getElementById("root"));
```

**./App.js**

```jsx
import React, { useState } from "react";
import Note from "./components/Note";

const App = (props) => {
  const [notes, setNotes] = useState(props.notes);
  const [noteField, setNoteField] = useState("Enter a new note here...");
  const [showAll, setShowAll] = useState(true);

  const filteredNotes = showAll ? notes : notes.filter((note) => note.flagged);

  const addNote = (event) => {
    event.preventDefault(); // Prevents the page from being refreshed when we click the submit button
    console.log("Note form submitted", event.target);

    const newNoteObject = {
      id: notes.length + 1, // May cause issues with duplicate ids if this array were to ever have elements deleted from it
      content: noteField,
      date: new Date().toISOString(),
      flagged: Math.random() > 0.75,
    };

    setNotes(notes.concat(newNoteObject));
    setNoteField("Enter a new note here...");
  };

  const handleInputChange = (event) => {
    console.log("Input form changed", event.target.value);
    setNoteField(event.target.value);
  };

  const noteList = () =>
    filteredNotes.map((note) => <Note key={note.id} note={note} />);

  return (
    <div>
      <h1>Notes</h1>
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          {" "}
          {/* "Flips" the `showAll` state */}
          Show {showAll ? "Flagged" : "All"}{" "}
          {/* Changes button text depending on state */}
        </button>
      </div>
      <ul>{noteList()}</ul>
      <form onSubmit={addNote}>
        <input value={noteField} onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>
    </div>
  );
};

export default App;
```

**./components/Note.js**

```jsx
import React from "react";

const Note = ({ note }) => {
  return <li>{note.content}</li>;
};

export default Note;
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
const xhttp = new XMLHttpRequest();

xhttp.onreadystatechange = function () {
  if (this.readyState == 4 && this.status == 200) {
    const data = JSON.parse(this.responseText); // Parses the response into an object
  }
};

xhttp.open("GET", "/data.json", true);
xhttp.send();
```

Here, we use a new `XMLHttpRequest` object to interact with our server, allowing us to retrieve data from our URL without refreshing the page. `XMHttpRequest()` is a constructor that initializes our new XMLHttpRequest via the `new` keyword. We then assign a function expression to the `onreadystatechange` property of the object which is an `EventHandler` that is called whenever the `readySatate` attribute changes. In other words, whenever the `xhttp` object changes in state, the `onreadystatechange` event handler is executed indicating that the response to the request has arrived.

Inside the function, we initially check the `readyState` and `status` properties of the response in order to see if we actually were returned the correct response. You'll learn more about what these status codes mean later in this book. We then get the actual content of our response (the JSON) by parsing the `responseText` property of our response.

You may have noticed that our code is _asynchronous_ in this case, in that our code is not executed from "top to bottom" but rather is executed out of order with our event handler being executed only once our request has returned after being sent.

#### JavaScript's Asynchronous Model

JavaScript is a single-threaded language meaning that, in actuality, only one thing can happen at once. Thus, JavaScript runtimes or engines can only execute one statement at a time per thread (although, today, it is possible to run parallelized code on more than one thread wit h web workers; this does not make the event loop multithreaded).

Since JavaScript is single-threaded and can only make one 'thing' happen at once, we have to make sure that things that may take a while do not block interaction (such as making a web request like the one we did above but instead synchronously). This means that almost all I/O in JavaScript is non-blocking whether it be HTTP requests, reading/writing to the disk, fetching something from a database, etc. All of these actions would be done via the thread of execution providing a callback function that will be executed when this non-blocking operation is done so that the rest of the program can continue.

For now, you can think of these concepts as having a list of chores: to wash your clothes, cook your dinner, and brush your teeth. The synchronous way of completing these tasks would be to put your clothes in the washer, wait until the washer is done, then start cooking your dinner, then wait until you finished cooking your dinner, then start brushing your teeth.

The asynchronous way of doing this would be to start washing, then while the washer is running, you can go ahead and start cooking your dinner, after you start cooking your dinner, you can go ahead and brush your teeth. Then, you'll eventually get the callback function that will be executed for your dinner and washer (when those tasks are done, you'll have to tend to them), so a callback function for the washer might be to pick up the clothes, and a callback function for your dinner might be to put it on a plate and serve it to yourself. These callback functions are called once the task at hand is done. The event loop simply just keeps track of the order that everything is in and what needs to be executed.

# DESCRIBE THE EVENT LOOP IN DETAIL LATER

### The Axios Library

Now, let's take a look at using the _axios_ library in order to fetch data from our server. It functions quite like fetch, a promise based function, but offers some additional features such as its wide browser support (for example, `fetch()` is not supported on IE11, but Axios functions just fine with its backwards compatability) and convenience features such as a timeout property, automatic JSON data transformation, and much more.

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
    "production": [">0.2%", "not dead", "not op_mini all"],
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
  }
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
import axios from "axios";

const examplePromise = axios.get("http://localhost:3001/notes");
console.log(examplePromise);

const examplePromise1 = axios.get("http://localhost:3001/randomtext");
console.log(examplePromise1);
```

A promise can have three different states

- **Pending**: Was not yet fulfilled nor rejected (initial state, the value has not been made available yet)
- **Fulfilled**: The async operation has completed successfully (the final value of the promise was successfully determined after the operation had completed)
- **Rejected**: The async operation has failed (the final value of the promise was unable to be determined)

Taking a look at your console to see the result of these `console.log` statements, you should find the following printed

![image-20191024205638403](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191024205638403.png)

> This image displays the Chrome Developer Tools Console in the native dark mode (supported as of Chrome 78). It may be beneficial to use Chrome in its dark mode depending on your environment.

Here, you can see how our first promise returned a promise status of being "resolved" as it was pointed to an actual valid URL, the notes route of our server. The second promise displays a status of "rejected" as we just gave it a random route to fetch something from.

If we wanted to get the result that our promise resolves to, we can provide a function to the `then` method of our promise.

```jsx
const examplePromise = axios.get("http://localhost:3001/notes");

examplePromise.then((response) => {
  console.log(response);
});
```

What we're doing here is providing a callback function to the `then` method of the promise (which is included in the `Promise` prototype with the responsibility of appending fulfillment and rejection handlers to the given promise). This callback function with then be called when the operation has been completed/resolved and pass in the response object as the `response` parameter. This response object includes the typical response data that would come with an HTTP GET request response including the headers, status code, and the actual content/data.

The above code snippet would produce the following result in the console

![image-20191024214425345](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191024214425345.png)

The server returns the data as plain text, but we're able to parse the plaintext data into a JavaScript array since we know the data type as mentioned by the `content-type` property inside the `headers` property of our response as shown below.

![image-20191026173817715](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191026173817715.png)

In most cases, we wouldn't actually need to store the entire object in a constant or variable, so we can get rid of the assignment operator and variable declaration. We can also clean our code up by using a code formatting style in that we _chain_ our methods

```jsx
axios.get("http://localhost:3001/notes").then((response) => {
  const notes = response.data;
  console.log(notes);
});
```

Now, let's take a look at an attempt to actually use the data returned from the server in our notes app from earlier.

**./index.js**

```jsx
import ReactDOM from "react-dom";
import React from "react";
import App from "./App";

import axios from "axios";

axios.get("http://localhost:3001/notes").then((response) => {
  const notes = response.data;
  ReactDOM.render(<App notes={notes} />, document.getElementById("root"));
});
```

Here, upon starting our app, we'll call the `get` method of `axios` and, with `then`, assign the callback function to get the data from the response and render our `App` component to the root element of our web page with the response passed in as the `notes` property.

This will go ahead and, upon resolving the promise involving our call to our server, render our `App` component with the `notes` property being assigned the data of our response, thus rendering our app with the notes. Notice how we're only fetching the data upon our app being loaded and rendering our entire React app inside the callback function; this could lead to some issues in the future, so let's explore another way to achieve our desired result

### Effect Hooks

Another type of hook that proves to be quite useful is the effect hook. Effect hooks allows for side effects (data fetching, setting up subscriptions, manually changing the DOM, etc.) to be performed in function components.

We'll be using the effect hook in our `App` component to fetch the notes data from our server, so we can get rid of our axios method call in **index.js** and the mention of a `notes` property in the usage of our `App` component.

**./index.js**

```jsx
import ReactDOM from "react-dom";
import React from "react";
import App from "./App";

ReactDOM.render(<App />, document.getElementById("root"));
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
  console.log("Reached Effect ");
  axios.get("http://localhost:3001/notes").then((response) => {
    console.log("The promise has been fulfilled :)");
    setNotes(response.data);
  });
};

useEffect(effectHook, []);
console.log("Rendered", notes.length, "notes");
```

Here, you'll notice how `useEffect` takes in a function to which we can use to fetch our data (this function will be run right after the component is rendered). `useEffect` also takes in a second parameter for which if you pass in an empty array ([]), like we did in this example, the effect will never be re-run as it tells React that it does not depend on any props or state (thus eliminating the need that it would need to be re-run since it's not dependent on anything that changes). If we were to omit this empty array as the second argument and use React's default, the effect would be run after every single completed render; that means, if something changes in your component to where it has to be re-rendered, this request would be made which may not be a good idea for performance reasons.

Taking a look at the console, assuming that everything works and the server is up, the following statements would be printed

![image-20191027003328132](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191027003328132.png)

- 'Rendered 0 notes' is printed first as the component is rendered for the first time without there being anything in our `notes` piece of state, simply because the function passed to `useEffect` had not been executed yet
- Then, after the components first render, the `effectHook` function is executed (which was passed into `useEffect`) which causes 'Reached Effect' to be printed.
- After our function had been executed, eventually (assuming that our promise is actually fulfilled), the fucntion that we passed into the `then()` method of our response will be executed with 'The promise has been fulfilled :)' being printed to the console.
- Finally, we set our `notes` piece of state to our response's data using the `setNotes` function that was supplied to us via the use of `useState`. This update to our state causes our component to re-render which leads ot the `console.log` statement involving our rendered components to be executed again; this time, however, our `notes` piece of state actually has three objects inside it, so 'Rendered 3 notes' is printed to the console.

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

Here, we made the changes to store the promise that axios returns in a constant so we can reference it later on when using our event handler defined in another constant to be set by the `then` method of the promise. This way of writing things, while it may allow for cleaner statements, is not usually the best way to go about these promises, the formatting of _chaining_ usually works just fine.

With regard to taking various approaches to solve problems, a good rule to follow involves writing your code to be easily read. Don't strive to have the "fanciest" solution with crazy one-liners, just write code in the most concise, human-readable form that you can. This makes maintaing your own projects and having other people work on them much easier; keep it stupid simple (KISS) is an acronym that you may want to have in mind when working on larger projects.

## Working With Data

### REST Terminology

If you've ever decided to look up what a REST (Representational State Transfer) API is supposed to do, you might have come across the four functions which use HTTP requests to: GET, PUT, POST, and DELETE data. REST APIs consist of data objects, called "resources" in REST terms, which all have unique addresses (the URL). For example, with the notes app that we've been working with, the URL **/notes/3** would return the note in the notes collection with an ID of 3 whereas a request to the URL **/notes** would return a resource collection of _all_ the notes. Forward slashes (/) typically indicate hierarchical relationships. Additionally, is it good practice to not use trailing forward slashes (/ at the end of URLs) as it does not have any semantic value.

Resources are fetched via HTTP GET requests to the server. In our above app, we retrieved a resource collection containing all the notes through a HTTP GET request to our /notes URL. If we wanted to store a new note, instead of making a HTTP GET request, we would have to make an HTTP POST request. The following update to our `addNote` handler is an example of how we might make a HTTP POST request to a server

```jsx
const addNote = (event) => {
  event.preventDefault();
  const newNoteObject = {
    content: noteField,
    date: new Date(),
    flagged: Math.random() > 0.5,
  };

  axios.post("http://localhost:3001/notes", newNoteObject).then((response) => {
    console.log("Received Response", response);
    setNotes(notes.concat(response.data));
  });
  setNoteField("Enter a new note here...");
};
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
  );
};
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
  console.log(`To change 4's flagged property`);
};
```

Another thing you might have noticed is the use of the _template strings_ or _template literal_ in our `toggleFlagged` event handler. With this, you can embed expressions into your strings through the use of simple syntax as follows `` `Here is a template literal ${expression}` ``. Make note of how you must wrap your template literals in grave accents '`' rather than quotation marks.

Now, when trying to toggle the checkboxes in our app, you'll notice that the specific event handler functions for each given `Note` component is executed, telling you that a given note's flagged property is to be chained. You'll also notice that our checkboxes aren't actually changing with regard to being checked or not. This is because they depend on the `Note`'s `flagged` property, and we actually aren't updating this property in our `toggleFlagged` event handler.

Let's update our event handler function to make a HTTP POST request to our server and update the given `note`'s `flagged` property.

```jsx
const toggleFlagged = (id) => {
  const noteURL = `http://localhost:3001/notes/${id}`;
  const referencedNote = notes.find((note) => note.id === id);
  const updatedNote = { ...referencedNote, flagged: !referencedNote.flagged };

  axios.put(noteURL, updatedNote).then((response) => {
    setNotes(notes.map((note) => (note.id === id ? response.data : note))); // Explained
    console.log(`Changed ${id}'s flagged property`);
  });
};
```

Here, you'll see the combination of a few concepts that we have talked about previously. First, we reference the specific resource URL of the `note` that we want to update by creating a `noteURL` constant that makes use of the `id` parameter variable in order to access the `note` resource that we want.

We then use the array's `find` method in order to find the note with the `id` that we're looking for (we could've passed the entire `note` in as a parameter, but that makes for a messier solution when it comes to the comparisons that we'll do later). We then store the `note` with the desired `id` in the `referencedNote` constant.

From this, we create a new object that "copies" everything from the old `note` object using the spread syntax (...) except the `flagged` property from which we apply a logical negation `!` to toggle the property. We store this updated note object in the `updatedNote` constant.

> Remember: We should not mutate state in React; thus, we create a new constant that has our updated `note` rather than changing the `flagged` property of the `note` referenced by the `referencedNote` constant as such
>
> `referencedNote = !referencedNote.important`

### Shallow Copies vs Deep Copies

#### Shallow Copies/Clones

When we create a new `note` in this fashion with regard to using the spread syntax. We create a _shallow copy/clone_. Shallow copying only copies over field values over to our `updatedNote` object, so that the values of the properties of our new object still reference the same objects that `referencedNote` referenced. The only difference being that there is a new object that contains all of these fields and that we can make changes without mutating the original object by assigning new objects in place of these properties. In short, this means that if our original note had a reference to an object in one of its properties and we created a new note, `copiedNote` that utilized the spread syntax, the same property would reference the exact same object that the original note referenced. This is something to consider, because if you accidentally mutate the state of the referenced object, you mutate the object of the original note's property. Consider the following code snippet

```jsx
const obj1 = {
  enabled: true,
  size: {
    width: 100,
    height: 100,
  },
};

const obj2 = { ...obj1 }; // Shallow copy of obj1 using the spread operator

obj2.enabled = false; // Updates the enabled property of the original object to false

obj1.size.width = 200; // Changes a property of an object that the original object refers to

console.log(obj2);
```

![image-20191104223920660](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191104223920660.png) Seeing the output of `console.log(obj2)` and diving into the `size` property of that object, you'll see how updating it via `obj1` still affected the object referenced by a property of `obj2` since both `obj1` and `obj2` refer to the exact same object with only the references being copied over to our new, distinct object (shallow copy). In the above snippet, you'll notice that the output of `obj2` uncovers that the `width` property of the `size` property of `obj2` is set to 200, even though we made the changes to `obj1`.

#### Deep Copies/Clones

Deep cloning involves going though all of an object's properties and cloning the objects and any objects that they might refer to in order to create a brand new object that does not share any references with the original object. This would be done in a recursive manner so that the cloning would continue from object reference to object reference until there are no more referred objects to be cloned. Unfortuneately, there is not a native way in JavaScript to deep clone an object which may prove useful in React where you may have to create objects that are entirely new. There are various custom solutions to this problem that you can find online. They all come with various benefits and disadvantages (for example, some do not support function object values or objects that contains a self reference to itself).

### Toggles Continuted

Going back to what our event handler does here, we make a HTTP PUT request to our backend server using `axios` telling it to replace the resource at `noteURL` with `updatedNote`.

Then, in our callback function from our HTTP PUT request, we use `setNotes()` to update our `notes` piece of state to a new array created by mapping over our original array and replacing the note that matches the `id` in our parameter variable with the `data` property of our response, essentially updating our state to reflect our backend and the change that we had just made.

```jsx
setNotes(notes.map((note) => (note.id === id ? response.data : note))); // Explained
```

Here, you can see how we map over the `notes` array in order to create a new array. Notice how we use a conditional (ternary) operator in order to see whether we should be replacing a given object with the updated object we had received in our response. If our `id` parameter variable (the ID of the note that we had just updated) matches the `id` of the note that we are currently mapping over in our `notes` array, then we will replace it with the updated object.

This method of dealing with updates also gives us the ability to verify that our change had indeed been reflected by our backend. If our state never updates, we know that we had not fulfilled our promise and that something has went wrong. We will soon go over how to handle these errors.

After this, we just log our changes to the console, explaining that we had successfully changed the `flagged` property of a given note.

### Organzing Backend Communication Services

An important computer programming principle to keep in mind while making these applications is the _single responsibility principle_ which states that every single module, class, or function should have responsiblility over only a single part of the functionality of our overall application. Looking at our `App` component, you might have noticed that we're cramming quite a lot of code in this singular component and that it violates the single responsibility principle in that it also does the job of communicating with the backend. Let's look at moving our code involved with communicating with our backend server into its own module.

In our project structure, we'll put all of our services in the directory **src/services**.

Within **src/services**, we'll create a **notes.js** file

**./services/notes.js**

```jsx
import axios from "axios";
const baseUrl = "http://localhost:3001/notes";

const getAll = () => {
  const request = axios.get(baseUrl);
  return request.then((response) => response.data);
};

const create = (newObject) => {
  const request = axios.post(baseUrl, newObject);
  return request.then((response) => response.data);
};

const update = (id, newObject) => {
  const request = axios.put(`${baseUrl}/${id}`, newObject);
  return request.then((response) => response.data);
};

export default {
  getAll: getAll,
  create: create,
  update: update,
};
```

Here, you can see how we used a default export to export an object that contains three functions defined earlier in this file (`getAll`, `create`, and `update`) with these functions representing the actions that we have used in our application already to communicate with our backend.

Also, note how we're returning the `then` method of our original promise; this will allow us to write cleaner solutions later in our `App` component so that we won't have to reference the `data` property but rather just the returned object. We accomplish this by storing our promise in the `request` constant and then using the `then` method to assign a callback function that returns the `data` property of the `response` and then returning this new promise. When the promise returned by the `then()` method is resolved, it executes the respective handler function an returns a promise that resolves to the returned value. In short, this behaves as if we received only the data as a response and are returning that response instead of the entire response in a promise.

Now, in our `App` component, we can import this service and use it to talk to our backend rather than writing all of the functions within our `App` component as follows

```jsx
import noteService from "./services/notes";

//...

const App = () => {
  //...

  useEffect(() => {
    noteService.getAll().then((initialNotes) => {
      // Promise Chaining, explained
      setNotes(initialNotes);
    });
  }, []);

  const toggleFlagged = (id) => {
    const referencedNote = notes.find((n) => n.id === id);
    const updatedNote = { ...referencedNote, flagged: !referencedNote.flagged };

    noteService.update(id, updatedNote).then((responseNote) => {
      // Promise Chaining, explained
      setNotes(notes.map((note) => (note.id === id ? updatedNote : note)));
    });
  };

  const addNote = (event) => {
    event.preventDefault();
    const newNoteObject = {
      content: noteField,
      date: new Date(),
      flagged: Math.random() > 0.5,
    };

    noteService.create(newNoteObject).then((responseNote) => {
      // Promise Chaining, explained
      setNotes(notes.concat(responseNote));
      setNoteField("Enter a new note here...");
    });
  };
};
```

Here, we transitioned from making direct calls to `axios` methods to using our note service module to make requests for us and provide a layer of abstraction. You'll also notice instances of 'Promise Chaining' (demarcated by `// Promise Chaining, explained`). Here, we 'chain' another `then()` method to the previous promise returned by the functions in our notes function.

#### Promise Chaining Explained

The following diagram illustrates the act of a function creating an `axios` promise, calling its `then()` method in which upon resolving, the callback function that we have specified is executed, returning only the `response`'s data property rather than the entire response object in the form of a promise. When we call any of these functions in our `App` component, we get the `response`'s `data` property returned in the form of a promise that resolves to the `data`; we then 'chain' onto this and use its `then()` method that resolves and executes a callback that performs the appropriate operations with the returned data.

TODO: Replace with actual diagram

To explain things as simply as possible and to provide an example, a call to a promise's `then()` method returns a promise, so that we can also call its `then()` method and use the return value of its handler in our next promise. The following example illustrates how we access and use the return values of previous promises in subsequent promises

```js
new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000); // (*)
})
  .then((result) => {
    console.log(result); // Alerts 1
    return result * 2;
  })
  .then((result) => {
    console.log(result); // Alerts 2
    return result * 2;
  })
  .then((result) => {
    console.log(result); // Alerts 4
    return result * 2;
  });
```

Here, the first promise resolves and in the handler that we specified in our first `then()` call, the number 1 is printed to the console. Then at the end of the handler, we return the result that our first promise resolved to multiplied by 2. The return is returned in the form of a promise that resolves to the value that we returned. As the returned value was a promise, we can make another `then()` call upon this new handler, only this new promise resolved to the value that was returned in handler of the promise mentioned one before in the chain. We then use this returned value in our new handler and return it again in the form of another promise so that we can use it in another `then() ` handler.

The topic of promises is quite confusing, so if you don't understand what's going on here, don't worry. There are plenty of resources online that go in depth with regard to the explanation of promises. Again. the 'You Do Not Know JavaScript' book series provides an excellent (but quite lengthy) explanation of promises.

#### Cleaner Code

Looking back to the code snippet, notice how we no longer have to specify the URL of our resources in our `App` component among other things, making the code look a lot cleaner.

We can also make our code a bit cleaner in respect to using shortened syntax. There are many instances in JavaScript where we can take advantage of the object-property value shorthand introduced in ES6. This object-property value shorthand allows us to take something like

```js
const year = 1995;
const model = "apple";

const car = {
  year: year,
  model: model,
};
```

and turn it into

```js
const year = 1995;
const model = "apple";
const car = { year, model };
```

Thus, whenever both the property and variable reference name is the same, you can just write the name of the variable reference so that the variable reference will be stored in a property with the same name.

Applying this shorthand to our 'notes' service that we wrote earlier, we can now use the following in place of the object that we default exported as follows

**./services/notes.js**

```jsx
import axios from "axios";
const baseUrl = "http://localhost:3001/notes";

const getAll = () => {
  const request = axios.get(baseUrl);
  return request.then((response) => response.data);
};

const create = (newObject) => {
  const request = axios.post(baseUrl, newObject);
  return request.then((response) => response.data);
};

const update = (id, newObject) => {
  const request = axios.put(`${baseUrl}/${id}`, newObject);
  return request.then((response) => response.data);
};

export default { getAll, create, update }; // Taking advantage of the shorthand here
```

## Dealing With Promise Errors

If we ever encounter unexpected behavior in our web apps, we should be able to handle error messages gracefully and make the user aware that their request had not succeded. The console informing us of an error is simply not enough. Recall that there are three different 'states' of a promise, we have already discussed handling fulfilled promises, but now let's talk about when a given promise is rejected.

One thing we can do here is terminate our `json-server` process so that when our React app tries to retrieve notes or interface with our backend in any way, it is unable to. Do this by closing the associated terminal/console window that you have ran the `npm run server` script in or by pressing `control` + `c` in the associated window.

Now that our backend server is offline, we can refresh the page of our React app and observe how it fails to retrieve our notes.

![image-20191103224302657](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191103224302657.png)

Here, you can see how we do not handle this error in any way. When our HTTP request fails, the promise that is returned is rejected. The only way a user would be able to observe this error is to notice unexpected behavior or check the console which is undesirable.

We can handle the rejection of a promise by providing our promise chain with a `catch()` method which returns a promise and deals with rejected cases.

```jsx
examplePromise.catch(onRejected); // where onRejected is a function that is called when the promise is rejected
```

In terms of an `axios` request, using the `catch()` method would probably look somewhat like the following

```jsx
axios
  .put("totallyfakeurl.com/will_fail")
  .then((response) => {
    console.log("This actually didn't fail");
  })
  .catch((error) => {
    console.log("This failed as expected!");
  });
```

Here, when the promise is rejected, the callback function provided to the `catch()` method in our promise chain will be execute with whatever return value that the promise had being passed in as `error`.

Putting this into practice, let's write a rejection handler for when we can't retrieve any notes from the server as follows

**./App.js**

```jsx
//...
const effectHook = () => {
  noteService
    .getAll()
    .then((initialNotes) => {
      setNotes(initialNotes);
    })
    .catch((error) => {
      alert(
        `${error}: Something went wrong while trying to retrieve data from the server.`
      );
    });
};
//...
```

Here, we simply added a `then()` method call to our promise chain involved with fetching all of our notes. The callback function we provided to the `then()` method call simply takes the error passed into it and alerts the user via the default alert box while also displaying a message.

![image-20191103225332897](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191103225332897.png)

When we lay everything out in our promise chain as such in the following code snippet, it becomes easier to see how the object returned by the promise previous in the chain is passed in as the argument to the next callback function in the chain.

```jsx
axios
  .put(`${baseUrl}/${id}`, newObject)
  .then((response) => response.data)
  .then((responseNote) => {
    // ...
  })
  .catch((error) => {
    console.log(`${Error}: An error has occured.`);
  });
```

With this, you can now start writing error handlers for your promises that may get rejected.

### Styling in React

There are many ways to style your components in React. We'll dive into some of the more practical approaches later, but for now, we'll talk about styling in an old-fashioned sense in that we'll be adding CSS files to our application.

First, let's create a CSS file in the **/src** directory and name it **index.css**.

We can our CSS file to our application by simply importing it similarly to how we would import a module (we imported it in our **index.js** file so that these changes are reflected across all components)

**./index.js**

```jsx
import "./index.css";
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
import React from "react";

const Note = ({ note, toggleFlagged }) => {
  return (
    <li className="mainNote">
      <input type="checkbox" checked={note.flagged} onChange={toggleFlagged} />
      {note.content}
    </li>
  );
};

export default Note;
```

> Here, we use camelCase for our class names; it is common in the CSS world to use kebab-case (putting dashes between tokens). Most naming conventions are enabled by React, however, so it is up to your preferences and the style guide that you (and/or your team) are following.

Then, we can add the following snippet to our **index.css** file to target this new class as follows

```css
.mainNote {
  font-size: large;
  color: red;
}
```

We can also add multiple classes to a given element as follows

**./components/Note.js**

```jsx
import React from "react";

const Note = ({ note, toggleFlagged }) => {
  return (
    <li className="mainNote card">
      <input type="checkbox" checked={note.flagged} onChange={toggleFlagged} />
      {note.content}
    </li>
  );
};

export default Note;
```

We can now add the CSS class to target this new class that we have added to the `li` element above

```css
.card {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
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
import React from "react";

const Notification = ({ message }) => {
  if (message === null) {
    return null;
  }

  return <div className="error">{message}</div>;
};

export default Notification;
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
  color: #d8000c;
  background-color: #ffd2d2;
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
  color: #d8000c;
  background-color: #ffd2d2;
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
import React from "react";

const Separator = () => {
  const separatorStyle = {
    color: "white",
    padding: 15,
    fontStyle: "bold",
    fontSize: 16,
    backgroundColor: "#05386B",
  };

  return (
    <div style={separatorStyle}>
      <p>This is the end of the above content</p>
    </div>
  );
};

export default Separator;
```

Notice how we have passed the inline style object, `separatorStyle`, into the `style` property of the `div` element.

Now, we can simply add our `Separator` component into the `div` element that our `App` component returns.

```jsx
import React, { useState, useEffect } from "react";
import Note from "./components/Note";
import Notification from "./components/Notification";
import Separator from "./components/Separator"; // Separator Import

import axios from "axios";

import noteService from "./services/notes";

const App = (props) => {
  //...

  return (
    <div>
      <h1>Notes</h1>

      <Notification message={notificationMessage} />

      <div>
        <button onClick={() => setShowAll(!showAll)}>
          Show {showAll ? "Flagged" : "All"}
        </button>
      </div>

      <ul>{noteList()}</ul>

      <form onSubmit={addNote}>
        <input value={noteField} onChange={handleInputChange} />
        <button type="submit">Add Note</button>
      </form>

      <Separator />
    </div>
  );
};
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
console.log("Hello World, this is my Node application!");
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
const http = require("http");

const webApp = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello World, this is my Node application!");
});

const port = 3001;
webApp.listen(port);
console.log(`Web server running on port ${port}`);
```

Looking at this code, you'll see how we use Node's built-in web server via 'importing' it.

```js
const http = require("http");
```

This syntax probably looks slightly odd to you. This is because most applications in Node.js still use CommonJS modules (since Node applications needed a solution for modules before the official specification with ES6 modules came out). Recently, with Node.js 12, support for ESM modules was released. Unfortuneately, it still is quite a pain to deal with over the support we have for it in browsers as (at the time of writing) you still have to enable the ``--experimental-modules` flag and use either the `.mjs` extension or add `"type": "module"` to **package.json**. Because of this, we will still stick to using CommonJS modules.

Moving to the next block of code

```jsx
const webApp = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello World, this is my Node application!");
});
```

Here, we call the `createServer()` method of the `http` module that we have imported to create our server. This method takes in an event handler as a parameter which is called upon every single HTTP request to the server.

This event handler takes in objects pertaining to the request, `req`, and the response, `res`. We then assign the response code and other header information (the content-type) through a call to the `writeHead()` method of `res`.

Finally, we pass in a string argument to the `end()` method which adds the string to the response body. Our response will then be sent.

Moving to the next block of code

```jsx
const PORT = 3001;
webApp.listen(PORT);
console.log(`Web server running on port ${PORT}`);
```

We assign the port that we want our server to listen on to the `PORT` constant. We make our HTTP server listen to requests on the port 3001 instead of 3000 as create-react-app typically runs the React development server on the port 3000. We then call the `listen()` method of the HTTP server that we stored in the `webApp` constant and pass in the port that we wish to use as a parameter. After this, we simply log that we have ran the web server listening on `PORT` successfully.

Visiting **localhost:3001** in our web browser yields the following result

![image-20191105213153375](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191105213153375.png)

Here, you can start to see how this is similar to how `json-server` would serve up a response if we just returned a string in the `response-body`.

Let's make another stride toward our goal of writing our notes app in Node by hardcoding our notes array in **index.js** and returning it in our response.

**index.js**

```js
const http = require("http");

let notes = [
  {
    id: 1,
    content: "First Note",
    date: "2019-09-30T17:32:41.199Z",
    flagged: true,
  },
  {
    id: 2,
    content: "Second Note!",
    date: "2019-08-30T12:13:24.091Z",
    flagged: false,
  },
  {
    id: 3,
    content: "This is our third note.",
    date: "2019-08-30T12:20:14.998Z",
    flagged: true,
  },
];

const webApp = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify(notes));
});

const PORT = 3001;
webApp.listen(PORT);
console.log(`Web server running on port ${PORT}`);
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

This will install the `express` module to our **node_modules** folder (should only be kept locally, this will be discussed later) and add `express` to our `dependencies` object in our **package.json** file.

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

- The first release of a package starting with 1.0.0 (For example, 1.0.0)
- Increments of the third digit being bug fixes that **are** backwards compatible. (For example, 1.0.1)
- Increments of the second digit being new features that **are** backwards compatible. (For example, 1.1.0)
- Increments of the first digit and resets of the second and third being major releases and changes that break backwards compatability (For example, 2.0.0)

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
const http = require("http");
const express = require("express");

const app = express();

let notes = [
  {
    id: 1,
    content: "First Note",
    date: "2019-09-30T17:32:41.199Z",
    flagged: true,
  },
  {
    id: 2,
    content: "Second Note!",
    date: "2019-08-30T12:13:24.091Z",
    flagged: false,
  },
  {
    id: 3,
    content: "This is our third note.",
    date: "2019-08-30T12:20:14.998Z",
    flagged: true,
  },
];

app.get("/", (req, res) => {
  res.send("<h1>Hello World!</h1>");
});

app.get("/notes", (req, res) => {
  res.json(notes);
});

const PORT = 3001;
app.listen(PORT, () => {
  console.log(`Express: Server running on port ${PORT}`);
});
```

Here, you'll notice quite a few changes. First, we import `express` (which turns out to be a function) and store it in a constant, `express`. We then create another constant, `app` and call the `express()` function which is used to create an express application.

Then, moving down to the calls to the `get()` method of our `app`, you'll notice two route definitions. Routing involves the mechanism in deciding which functionality to call based on the provided URL and parameters of a request (mapping and figuring out what to do/return based off of a URL).

The first route that we register involves HTTP GET requests to the route of our application (/).

```jsx
app.get("/", (req, res) => {
  res.send("<h1>Welcome to our Notes Application Backend!</h1>");
});
```

You'll notice that we provide an event handler as the second parameter to our `get()` method call. This event handler takes in a `request` parameter which contains the information pertaining to the HTTP request and a `response` parameter which allows us to define how to respond to the request. `req` is short for `request` and `res` is short for `request`.

In the above snippet, we utilize the `send()` method of the `res` object in order to respond to the specified HTTP request with a body that contains the string `"<h1>Welcome to our Notes Application Backend!</h1>"`. We do not have to manually set the `Content-Type` header as Express automatically does so after seeing that the parameter of the `send()` method was a string. Additionally, we do not need to set the status code of the response as it defaults to 200 automatically.

In the following code block, we define an event handler that handles HTTP GET requests to the '/notes' path.

```jsx
app.get("/notes", (req, res) => {
  res.json(notes);
});
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

When we used `json-server` we could fetch individual notes by fetching the notes path followed by the ID number of the individual note. For example, if we wanted to interact with the note of ID 2, we would make a request to `host` +`/notes/2`.

We can add this functionality through the use of parameters in Express routes that allow us to take in values from the URL. Let's add the following to our application

**/index.js**

```js
app.get("/notes/:id", (req, res) => {
  const id = req.params.id; // Takes in the parameter from the URL
  const note = notes.find((note) => note.id === id); // Finds the note that matches the parameter
  res.json(note);
});
```

Parameters are defined with the above colon syntax, `:PARAMETER`. This above block will handle all requests made to the path `/notes/ANY_STRING` where ANY_STRING can be any string where this string will be passed in as a parameter. These parameters can be accessed via the `params` property of the `req` object. We then find the particular note with the ID passed in as a parameter using the `find()` array method that we have used earlier. Finally, we convert the `note` object into JSON and add it to our response.

Now, if you tried to execute your application and interact with an individual note, you'd find that our application isn't actually returning any notes. Some `console.log()` debugging later and you might find that our application simply isn't finding any matching notes, even if we provide the parameter with an ID that we do have. This is because we're using a strict equality comparison to check if we have matching IDs which takes into account the data type of the object. This leads to a failed comparison as our request parameter is a string while our `id`s are numbers. We can easily fix this by casting our parameter string into a number before making the comparison as follows

```js
app.get("/notes/:id", (req, res) => {
  const id = Number(req.params.id); // Takes in the parameter from the URL and casts it to a number
  const note = notes.find((note) => note.id === id); // Finds the note that matches the parameter
  res.json(note);
});
```

With this, we can now successfully fetch an individual note.

Let's go over some other problems that we should fix with the above block. Recall that making a call to the request's `json()` method returns the response with a response code of `200` by default. This response code means that the response had succeeded, but with our above implementation, even if we don't find a note, we return the response code of `200` as we do not handle `notes.find()` returning nothing.

Let's add a way to handle an invalid ID (not finding the note) and returning the correct response

```js
app.get("/notes/:id", (req, res) => {
  const id = Number(req.params.id); // Takes in the parameter from the URL and casts it to a number
  const note = notes.find((note) => note.id === id); // Finds the note that matches the parameter

  if (note) {
    // If we actually found a note
    res.json(note);
  } else {
    res.status(404).end(); // Sets the appropriate status for not finding the note
  }
});
```

Here, we add a if-else statement that sends `note` and the correct response code, `200`, if we actually find a note. If we do not find the note, we set the status of the response to `404` in which this response code signifies that the specified resource was not found. We then use the `end()` method to respond to the request without sending data (other than the actual response information itself).

We are able to just write `if (note)` as JavaScript objects are **truthy** (they evaluate to true in comparisons) whereas `undefined` is falsy (evaluates to false in comparisons). This means that if the node did not exist, `notes.find()` would return nothing, leading to the value of `note` being `undefined`; thus, `note` would evaluate to `false` and the appropriate control would be executed.

On our front end, we can display a user-friendly interface for when we encounter error response codes (such as 404).

## CONTINUED IN WEB DEVELOPMENT 2

Continued

## Resource Deletion (HTTP DELETE)

We've gone over adding, modifying, and requesting resources, but now let's go over the act of deleting a resource. We can delete an object by making a HTTP DELETE request to a given resource. We would handle this in Express with a route as follows

```jsx
app.delete("/notes/:id", (req, res) => {
  const id = Number(req.params.id);
  notes = notes.filter((note) => note.id !== id); // Filters out the specified note from `notes`

  res.status(204).end();
});
```

Here, upon a HTTP DELETE request to the above path, we take in the `id` parameter and set `notes` equal to a new array with the note that matches `id` filtered out. We then return the request with the response code `204` which signifies 'No Content' (we picked this obscure status code as there is not really a designated status code for a DELETE request). We also return no data with the response as shown by the call to `end()` without any parameters.

### Exploring the HTTP Standard

When building REST APIs, we must consider two very important properties of HTTP methods that are defined in HTTP's specification. You'll find HTTP methods that we have not explored yet mentioned in the following section; do not worry as understanding over these methods is not important to understanding the concepts surrounding these properties.

**Safe**

The first important property to consider is that GET and HEAD HTTP requests should be **safe**. This means that our request should not lead to any "side effects" in our server. For example, our HTTP GET request should not change our database in any way as we are simply requesting data. Additionally, our HTTP GET request should only return data that already exists on the server. These rules also apply to the HTTP HEAD request (similar to HTTP GET except that it only returns the status code and headers).

**Idempotent**

All HTTP requests (other than HTTP POST) be **idempotent**. This means that they should cause the exact same side effects regardless of how many times a given request is sent. For example, if we use a HTTP PUT request to update/replace a specific resource, duplicating this request would not give any other side effects other than replacing the resource for the first time. This is just to safeguard against duplicate requests so that we have a _fault-tolerant API_ that doesn't leave our system unstable if there were a duplicate request.

HTTP POST is not safe nor idempotent. We can send an HTTP POST request and it'll cause a side effect in our database in that it'll create a resource. We can also duplicate an HTTP POST request and watch it have side effects for every duplicate request. For example, if we sent 5 HTTP POST requests requesting that we create a new note, we'll end up with 5 new notes in our notes collection.

### Backend Development Tools

#### Postman

We can test simple HTTP get requests via the browser, but testing other types of requests is not so easy. We could write an application that implements these functions and makes the appropriate requests, but that is not the ideal situation when in development (especially in team settings). There are various tools that allow us to test backends; one of these tools is **Postman**.

First, let's install Postman from 'getpostman.com' or via a package manager. If you're running macOS and have `brew` installed, simply run the following command

```bash
brew cask install postman
```

Then, open Postman up and press `Cmd`/`CTRL` + `T` to create a new request.

![image-20191106234000147](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106234000147.png)

Upon creating a new request, we'll be displayed the following form. Here, all we need to do is point Postman to the URL that we want to make the request upon and define what type of request we want to make.

Just to test things out, let's define a GET request to `http://localhost:3001/notes' which should return a resource collection consisting of all our notes.

![image-20191106234204758](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106234204758.png)

Here, we can verify that things are working as expected. Now, let's try doing something that wouldn't be too easy to do in a browser.

Let's change the HTTP method to DELETE and point it to a specific note resource. In the following example, we'll send a HTTP DELETE request to 'http://localhost:3001/notes/1' which should delete the note with an ID of 1.

![image-20191106234746460](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106234746460.png)

Here, you can see that we received the expected response code, `204`. Now, if we make a HTTP GET request to the '/notes' path, we should receive a resource collection that does not include note 1.

![image-20191106234909026](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106234909026.png)

The image above shows that we were successful in deleting the resource with our HTTP DELETE request.

Now, since we defined and initialized the `notes` variable within our server code (we hardcoded the `notes` array in **index.js**), every time we restart the server, our changes will revert. If we want persistent changes, we will have to work with a database which we will discuss later.

#### VSCode REST Client

Visual Studio Code, as mentioned earlier, has tons of extensions that go on to make VSCode (Visual Studio Code) an extremely powerful tool for web development. If you happen to use VSCode, you can install the 'REST Client' extension to make requests instead of Postman (although Postman does make complex requests much easier). You can install the plugin by navigating to the 'Extensions' tab of VSCode and then typing in 'Rest Client'. The first result by Huachao Mao is the extension that we'll be using. Upon opening the extension's page, just press install and you should be all set.

![image-20191106235528641](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191106235528641.png)

You should see that the extension is enabled globally under the extension information.

We use this function by creating a `FILE_NAME.rest` file anywhere in our project directory and then making our requests from there. For the sake of keeping things organized, let's create a **/requests** directory where we can store all of our `.rest` files in. Let's then create a **get_notes.rest** file in our **/requests** directory.

**/requests/get_notes.rest**

```HTTP
get http://localhost:3001/notes
```

Upon viewing this file in VSCode, we'll se a 'Send Request' button appear above this request that we've written

![image-20191107000124682](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191107000124682.png)

After clicking the 'Send Request' button, we'll see the response displayed right in VSCode complete with syntax highlighting and header information.

![image-20191107000225620](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191107000225620.png)With this, we have explored two options for making HTTP requests to our backend during development.

## Receiving Data in the Backend

Let's now add the ability to add new notes to our Express server. Recall that we would do this by making a HTTP POST request to our notes resource collection with the request body containing our new note in JSON format.

In order to make accessing the request body of a given request easier, we'll be using `body-parser` which is a Node.js body parsing middleware that parses incoming request bodies before we handle the request. It essentially takes the JSON data of a given request, translates it into a JavaScript object and attaches it to the `body` property of the request object before the actual handler is called. This allows us to access the request body via the `body` property of `req`/our request object in our handler function.

### Middleware

`body-parser` is an example of middleware that we'll be using in a moment. Middleware generally involves software that connects different components or applications allowing them to interact. In the Express world, middleware are functions that execute during the lifecycle of a request, meaning that they are used for handling the `request`/`req` and `response`/`res` objects. In fact, Express is entirely composed of middleware functions which is how it makes handling requests much easier.

To illustrate this, we'll take the example of `body-parser` which takes the raw data from the request and translates it into a JavaScript object that can be assigned to the `body` property of the `request` object. It does all of this before the route handler is actually called so that we can interact with whatever the middleware had done for us.

Middleware functions receive three parameters: the `request` object, the `response` object, and the `next` function. For example, a middleware function may look like the following

```jsx
const exampleMiddleware = (request, response, next) => {
  //...
  next();
};
```

We have already discussed the `request` and `response` objects. The `next` function, which is a new concept, simply gives control and execution over to the next middleware that was bound (in order of the `use()` calls in the program that we will discuss next). The main takeaway for middleware functions is that they allow us to do things with the `request` and `response` objects before our `route` handlers are actually called, allowing all `route` handlers to take advantage of their functionality.

We can use middleware in our app by binding them through our Express app's `use()` method as follows

```jsx
app.use(exampleMiddleware);
```

where we use the above middleware function that we have defined.

For every request, before the execution of our route handlers, middleware functions are called in the order they were bound through the `use()` method of our Express server. For example, if we bound the following middleware in this order

```jsx
app.use(bodyParser.json());
app.use(exampleMiddleware);
```

the `bodyParser.json()` middleware function will be executed before `exampleMiddleware` where `bodyParser.json()` hands control over to `exampleMiddleware` through a call to `next()`. This order also allows us to use the `body` property of `request`/`req` in `exampleMiddleware` as it was executed beforehand and defines the property.

For most cases, we would define and bind our middleware before our routes (placing the `use()` calls before our route definitions), but there are a few cases where we can put our `use()` calls after our route definitions to achieve a desired behavior.

One practical use would be if we wanted to call a middleware function if no route had handled the HTTP request. The middleware function might look something like the following

```jsx
const notFound = (req, res) => {
  res.status(404).send({ error: "Your request did not match any endpoint." });
};
```

This function simply sends a response with the status code of `404` (not found) and gives an error message that the request did not match any endpoint covered by our routes.

We then can put the following snippet after all of our routes

```jsx
app.use(notFound);
```

so that the `notFound` middleware function will be called if no route handled the HTTP request.

### body-parser

We can import `body-parser` and lay out our HTTP POST route as follows

**/index.js**

```js
const express = require("express");
const app = express();
const bodyParser = require("body-parser");

app.use(bodyParser.json()); // Imports bodyParser

//...

app.post("/notes", (req, res) => {
  const note = req.body;
  console.log(note);

  res.json(note);
});

//...
```

Note that we did not have to install `body-parser` via npm as it is included with Express in versions 4.16 and up.

Upon adding this to our **index.js** file, we can then turn to Postman (or REST Client) to make sure that our route is set up correctly and to verify that the server is actually receiving our HTTP POST requests correctly.

Recall that we need to send the note that we wish to add in the request body of our request. We can do this by visiting the 'Body' tab of our Postman request and filling in the form as follows (make sure that you have closed out of the previous request or else you may not see this tab).

![image-20191107002734243](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191107002734243.png)

Here, we selected the **raw** type of data and changed our content type from **Text** to **JSON**.

Upon sending this request, we should see the note that we sent in our request body printed to the console as specified in our handler for the given route.

![image-20191107003254891](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191107003254891.png)

When using Postman, it is very important to make sure that you are setting the correct headers (`Content-Type`, etc.) as the server may not know how to parse the data correctly if not set correctly. For example, if we set the above request's `Content-Type` to `text/plain`, the server would parse the data as an empty object as it wouldn't think its a JSON object.

We can use REST Client to make the same HTTP POST request as follows

**/requests/create_note.rest**

```HTTP
post http://localhost:3001/notes
Content-Type: application/json

{
	"content": "This was sent using REST Client!",
	"flagged": true
}
```

Sending this request yields the following result

![image-20191107004016132](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191107004016132.png)

The explorer bar on the right provides some detail as to our current project structure.

At the expense of some user friendliness, REST Client offers some huge advantages involving collaboration and availability. These requests are available right in the project directory and do not require opening up another application. Additionally, these requests can be easily shared between a team (included in the project directory and do not require additional setup other than installing the extension). The documentation for REST Client could prove to be very helpful and powerful should this be the tool that you choose to use.

One thing to keep in mind is that blank lines can cause issues with how the REST Client interprets the file (for example, a blank line before the row specifying the HTTP headers can cause the client to interpret this as the headers being empty). With these potential issues surrounding incorrect headers, it may be wise to save some headache by logging the `headers` property (`console.log(req.headers)`) of the `req` object whenever you use it, allowing you to quickly verify whether the headers are correct.

Now, going back to adding the ability to add new notes to our server, we can finish up our handler and a helper function that together will actually create a new note and add it to `notes` as follows

**/index.js**

```jsx
//...

const generateNoteId = () => {
  // Helper function to generate ID #s
  const maxId =
    notes.length > 0 ? Math.max(...notes.map((note) => note.id)) : 0;
  return maxId + 1;
};

app.post("/notes", (req, res) => {
  const body = req.body;

  if (!body.content) {
    return res.status(400).json({
      error: "No content specified in note body",
    });
  }

  const newNote = {
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
    id: generateNoteId(),
  };

  notes = notes.concat(newNote);

  res.json(newNote);
});

//...
```

Here, you'll notice a few things. First, we have defined a helper function that will generate the ID for our new note by simply taking the largest ID number in the current list and then returning that `maxId` + 1 to give us a new 'maximum'. With 'actual' applications, this method of dealing with IDs is not recommended, but it will serve us just fine for now. Taking a closer look at the body of code, you'll notice the usage of a few new functions

```jsx
const generateNoteId = () => {
  // Helper function to generate ID #s
  const maxId =
    notes.length > 0 ? Math.max(...notes.map((note) => note.id)) : 0;
  return maxId + 1;
};
```

Here, we call the `map` method of our `notes` array to create a new array with just the `id`s of our array of notes. We achieve this by passing in a function to `notes.map()` that takes in the `note` object and just returns its `id` property. Then, we use `Math.max()` upon the array that we have destructured to give us the maximum `id` value currently in our array. We destructured our array as `Math.max()` takes in many numbers as its parameters rather than an array. Finally, we return the value of `maxId` (the maximum `id` value that we have found) + 1. We also used the ternary operator, `expression ? ifTrue : ifFalse`, to give us a default value of `0` if `notes.length` is not greater than 0.

You'll also notice that we handle having a missing value for `body.content` by sending a response with the status code `400` which stands for 'Bad Request' as well as sending the error message `'content missing'`:

```jsx
if (!body.content) {
  return res.status(400).json({
    error: "No content specified in note body",
  });
}
```

Calling `return` causes the code execution in this route to stop so that we never reach the point to where our `newNote` object is created and added to our `notes` array.

We then create a new note via the following block of code

```jsx
const newNote = {
  content: body.content,
  flagged: body.flagged || false,
  date: new Date(),
  id: generateNoteId(),
};
```

Here, you'll notice that we take in the `content` property of the `body` constant which was created via a reference to `req.body`. We know for certain that there is a `content` property via the logical check that we performed earlier in the handler.

You might have wondered, what do we do if another value was missing such as a the `flagged` property value or the `date` property value? We simply provide default values through the use of the logical OR operator, `||`. For example, in the following code snippet, we provide the `flagged` property with a default value of `false` if there is no value for `body.flagged` (it was not specified in the JSON of the body of the request)

```jsx
flagged: body.flagged || false,
```

The logic of this statement depends on the fact that `undefined` has a falsy value. Thus, if `body.flagged` is `undefined` it will evaluate to false and the value on the right, `false`, will be assigned to the `flagged` property. It may be helpful to think of the OR logical operator as a _selection_ operator with the following rules:

- If the 1st expression (left side) evaluates to true, it will always be the one _selected_ and 'returned' from the comparison
- If the 1st expression (left side) evaluates to false, the 2nd expression (right side) will be the value returned 'returned' from the comparison **even if the 2nd expression is falsy** (like in the above example)

> This way of providing defaults can have some unintended outcomes. For example, if you wanted to assign 0 or an empty string as the value for a property with a default specified with the logical OR operator, it would actually fall back to the default as empty strings and 0 are falsy.
>
> Explained further, due to `||` being a boolean operator, the left operator must be coerced to a boolean to be evaluated with 0 and "" among other falsy values being coerced into false, and as a result not being returned.
>
> This is being solved with the Nullish Coalescing Operator (`??`) which returns the left operand if the value provided is a falsy value that is not `null` or `undefined`, so it will only return the right hand expression if the left expression is `null` or `undefined`.

Finally, we concatenate the new note object that we have created to our `notes` array and send the new note that we have created in the response through a call to `res.json()`.

# Connecting to the Backend

As we enter the upcoming chapters, there will be a change in the file structure of our project files (available for download). The project files will now be structured with the backend files in the `/Backend` directory and the `create-react-app` files in the `/Frontend` directory as pictured below

![image-20191109174054578](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191109174054578.png)

Now, upon starting both our backend and frontend, visiting our React app fails to render any notes. A peek at the console reveals the following error

![image-20191109174450193](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191109174450193.png)

You might be wondering why our browser is failing to interact with the server when it making the same exact requests that we were doing earlier with Postman or REST Client. This issue lies within something called **Cross-Origin Resource Sharing** (CORS).

CORS is a mechanism that allows restricted resources (fonts, etc.) to be requested from a domain/IP address outside of the domain from which the initial resource was served.

In our case, we're trying to load resources from a different origin from the origin that our React app was served from. Our React app was served from `localhost:3000` while we're trying to request the notes from `localhost:3001`.

By default, the same-origin security policy is enabled and does not allow us to request resources from other domains. However, we can easily allow requests from other origins by using the `cors` middleware.

This middleware does not come with Express, so we'll have to install it with npm:

```jsx
npm install cors
```

Then we can bind the middleware by adding the following to **index.js**

**/Backend/index.js**

```jsx
//...
const cors = require("cors");

app.use(cors());
//...
```

Now, upon reloading our backend server (or saving the file if you ran the server with `npm run watch` since it'll restart automatically), you'll see that our React frontend is able to successfully make requests and retrieve resources from our server as our server now allows requests to come from other origins.

## Deploying our App to the Internet

Now that we have our frontend and backend talking to each other, how might we emulate what's happening on our local machine on the internet? Let's first move our backend to the internet using Heroku.

Heroku is a cloud platform as a service (provides infrastructure) that allows developers to deploy, manage, and scale modern apps using a variety of services. If you wish to follow along, create an account on Heroku and then continue (or just keep reading and just see what we'll be doing).

### Installing the Heroku Command Line Interface

The Heroku Command Line Interface (CLI) is a super easy way to manage our apps from the terminal. Updated directions for installation can always be found in the Heroku Devcenter, but the following commands should work for most cases.

**(macOS only) Install Homebrew**

If you are running macOS, Homebrew is a massive time saver for many tasks as installing new pieces of software or installing packages becomes as simple as running a command. It becomes even more helpful if you ever have to install software that does not come pre-built, to which you can just use a Homebrew formula to easily install the program. You can install Homebrew by running the following command

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

**macOS with Homebrew Installed**

```bash
brew tap heroku/brew && brew install heroku
```

**Ubuntu 16+**

```bash
sudo snap install --classic heroku
```

If you are running Windows or did not install Homebrew, you can install the Heroku CLI via the installers available in the Heroku Devcenter.

### Moving our app onto Heroku

First, when we upload our files to Heroku, Heroku needs to know how to start our application. We do this by providing them with a `Procfile` in the root of our directory.

The `Procfile` is just a text file located in the root directory of our app that explicitly declares what command(s) should be executed in order to start our app.

**/Backend/Procfile**

```text
web: node index.js
```

Heroku will try to dynamically configure an application port via the definition of an environment variable (it assigns our app a port, so we can't define it to a fixed number beforehand). An environment variable is a variable whose value is set outside of a program (usually in the operating system or micro-service). Thus, we should update our assignment of `PORT` in **index.js** to use the `PORT` environment variable on the remote Heroku server, if available.

**/Backend/index.js**

```jsx
const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Here, we try to use the `PORT` environment variable, but if it is undefined, we fallback to the default port of `3001`.

Finally, add a `.gitignore` file with the following content to the root of the backend directory that we will be uploading

**/Backend/.gitignore**

```text
node_modules/
```

After this preparation, we can begin moving our project over to Heroku by creating a Git repository and interacting with the Heroku CLI.

From within our project directory, let's run `git init` in order to create a local Git repository (Git repositories essentially track the changes that we make to our code over time).

```bash
git init
```

Upon running this command, we should see confirmation via a console statement along the lines of 'Initialized empty Git repository in ...'.

Then, we can add all of our files to the 'stage' (essentially preparing it for a commit/save of changes) by running `git add .` (where `.` just specifies that we wish to add the entire directory to the stage)

```bash
git add .
```

Finishing with our local repository for now, we can run `git commit -m 'Initial Commit'` to commit/save the changes in our staging area with the message 'Initial Commit'

```bash
git commit -m 'Initial Commit'
```

![image-20191110011555756](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110011555756.png)

Then, let's run `heroku create` to create our app

```bash
heroku create
```

You might encounter the following error:

![image-20191110010518525](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110010518525.png)

Upon encountering this error, simply press any key and sign into the Heroku account that you have created. After this, you should see the successful creation of your app as follows

![image-20191110013156770](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110013156770.png)

This creates a Heroku-hosted **remote** and automatically sets it as the remote for our local repository. **Git remotes** are versions of our repository that live on other computers/servers.

Finally, we can run `git push heroku master` to 'push' our commit up to the remote Heroku server. The `git push` command is used to 'push' (transfer commits) from our local repository to a remote repository (in this case, our remote repository is heroku). Thus, in this command, the first argument is the remote repository/destination, `heroku`, with the second argument being the name of the branch that we want to push, `master` (since we have committed all our changes to the `master` branch)

![image-20191110012319712](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110012319712.png)

Here, in the console output, you can see that Heroku has detected that we have pushed a `Node.js` app and is creating a runtime environment in order to run our application. It will then run our application using the command that we have specified in our `Procfile`.

The process should finish with output akin to the following

![image-20191110012938116](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110012938116.png)

We can look at our application by visiting the URL of our application that was provided when we ran `heroku create`.

![image-20191110014617365](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110014617365.png)

To further verify that our application is working, we can make an HTTP GET request to the `notes` collection by visiting the appropriate URL as follows

![image-20191110014720708](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110014720708.png)

If we ever run into any issues with an application running on Heroku, we can take a peek at the logs that Heroku provides us by running `heroku logs --tail` (or `heroku logs -t` for short) which gives us a log of everything happening on the server as it happens. For example, if we failed to use the `PORT` environment variable (as we can't set the port to a fixed number on Heroku), we'll see the following displayed when we visit the URL of our application in a browser

![image-20191110013902528](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110013902528.png)

We can then run the `heroku logs -t` command in order to look at the Heroku logs and see what may be wrong. The command should yield output similar to the following

![image-20191110014013076](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110014013076.png)

Upon analyzing the log statements, we can see that a particular error is causing our process to stop and change to a 'crashed' state.

![image-20191110014136975](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110014136975.png)

With this information provided by the logs, we can identify the root of the error and remedy the issue. Thus, it may be wise to always take a look at the logs of a new application in case there may be an issue that is not immediately apparent.

Once we have fixed the error, it's simply a matter of re-staging our changes, committing the stages, and pushing our `master` branch (the branch that we have committed our changes to) to our Heroku remote.

![image-20191110014536054](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110014536054.png)

We can then observe from the logs that our application is indeed 'up'.

### Building and Deploying our React Frontend

Now, let's talk about how we would move and host our frontend on the Internet.

First, we'll need to create the _production build_ of our React app. In the past, when we've been working with our React app, we've been running the code in _development mode_ which displays clear error messages, automatically reloads and renders code changes, and more; however, the production version includes extra performance optimizations, a minified version of the code, and no detailed error messages (for security reasons).

`create-react-app` comes with a script that will create the production build for us with one simple command. Let's run the command at the root of our frontend project

```bash
npm run build
```

![image-20191110124758374](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110124758374.png)

We'll then see output similar to the above. You'll also notice a message describing how the project was built to be hosted as the server's root directory. We'll talk about what this means in a moment.

After taking a look at our project directory, you'll notice that our build script created a directory, **build**, in the root of our frontend directory (the directory from which we ran the build command). This directory contains a few files, the **index.html** file that provides some meta information and references the JavaScript files needed to run our application, and a **static** directory. This **static** directory contains the minified versions of our JavaScript code that was generated by the build process. Minified code is simply the result of removing all unnecessary characters from the source code without changing any of the functionality; this allows for our files to be faster served over the Internet since there is less data to transmit.

### Serving Our Static Files from the Backend

Now, as we discussed earlier, we can serve the frontend from our server's root directory. Thus, one (although messy) way to do this would be to configure the backend to serve the frontend's main page (**/Frontend/build/index.html**) as its main page with it pulling the rest of the static files as well. Static files are simply files that clients directly download from the server as they are.

First, let's copy over the **build** directory from our React app over to the root of our **/Backend** directory.

![image-20191110131359893](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110131359893.png)

Here, you can see that we now have the **build** directory in the root of our **/Backend** directory.

Now, let's make the appropriate changes to our Express server in order to serve up these static files.

First, we need to add a middleware called `express.static` in order to be able to serve static content (**index.html**, the minified JavaScript files, etc.). We do not need to install it as it is built-in with Express.

**/Backend/index.js**

```js
//...
app.use(express.static("build"));
//...
```

The middleware takes in a root argumetn that specifies the root directory from which to serve static assets. In this case, we're serving our static assets from the **build** directory, so we'll pass in `'build'`. Now, since this middleware appears before our routes and route handlers, whenever our server receives an HTTP GET request, the middleware will check if our **build** directory contains any files that match as per the request's address. For example, visiting the domain of our application by default requests the **index.html** file; since there is a file called **index.html** in our **build** directory, Express will serve that file which serves up our React frontend.

We'll also need to update the `baseUrl` constant in our notes service as the URL from which we will be making resource requests to is different.

**/Frontend/src/services/notes.js**

```jsx
import axios from "axios";
const baseUrl = "/notes";

//...
```

Here, we can remove the domain and just add the base path from which all of our resources will come from. We can do this as our frontend will now be hosted from the same domain/IP as the backend, so if the frontend was being served on `examplehost.com`, we can simply make requests to the **relative URL** `/notes` to which our browser will take that relative URL and assume to make requests to `examplehost.com/notes`. A relative URL is simply a URL that doesn't explicitly state the protocol (HTTP or HTTPS) or the domain, forcing the web browser (or Node) to assume that the path is appended upon the current URL.

### Automated build processes

Upon making these changes, we'll need to re-build our frontend and move over the **build** directory once again. Let's focus on automating this process with a script in the **package.json** of our backend directory (since this is the directory upon which everything is moving over to)

**/Backend/package.json**

```json
//...
{}
//...
```

We can run multiple commands in a script by simply adding another command after `&&` in the sequence. For now, our scripts only work on \*nix systems (unfortunately, some of these commands will not work on Windows unless you install `cygwin` or some other alternative). This is because we use the `rm` command to remove the **build** directory in our backend (in preparation for the new directory that will be copied over) and the `cp` command to copy our **build** directory over to the root of our backend in our `build:ui` script.

Notice how we can run npm scripts from within a script as with our `deploy:full` script which runs our `build:ui ` and `deploy` scripts.

To sum the scripts up,

- The `build:ui` script first deletes the **build** directory in our backend (as it will be updated via a new copy). Then, it changes our directory (using the `cd` command) over to the **/Frontend** directory using a relative path. After this, since we're in the **/Frontend** directory, it'll run the `npm run build --prod` command to build our production React app. Finally, we use the `cp` command in order to copy the **build** directory over to the **/Backend** directory.
- The `deploy` script simply pushes our committed changes to the Heroku remote (assuming that we have already staged and committed some changes).
- The `deploy:full` script first executes the `build:ui` script as described earlier. Then, we stage all of the changes in our directory by executing `git add .`. After this, we commit our changes with the message `'UIBuild'` by executing `git commit -m UIBuild`. Finally, we run the `deploy` script as described earlier which simply pushes these committed changes to our Heroku remote.
- The `logs:prod` script simply executes the Heroku log command that we talked about earlier, allowing for easy access to the live logs of our Heroku container.

With this, if we make any updates and want the changes reflected on our remote/production server, we can just run `npm run deploy:full` which automates the entire process for us.

Upon running the above command, we can now access the address of our Heroku app and see the frontend for our app.

![image-20191110203212139](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110203212139.png)

When we visit the address of our Heroku app (in this case, `hidden-chamber-09544.herokuapp.com`), our express server simply returns the **index.html** in the **build** directory as we have specified with the usage of the `express.static()` middleware. This HTML file that we are served then causes us to load the JavaScript files referenced in the file to which these JavaScript files contain all of the React code, components, and logic that we have written. Then, when our browser receives all of these files, our app is then rendered and can request the notes via the JavaScript now running on our browser. Our React frontend makes the requests for the notes to the same address but with the '/notes' path appended to the end. You can see this process illustrated in the 'Network' tab of your chrome developer tools:

![image-20191110203717388](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110203717388.png)

AFTER THIS, UPDATE THE CODE AND GO BACK TO WHEN THEY INITIALLY SERVED THE APP\*\*\\

### API Conventions

In the above example, we pushed the code directly to the 'production server' in order to observe our changes. It is important that you never do this in practice as testing should be done thoroughly on your local machine before pushing your code to where it may affect others.

Currently, in our backend, our usage of routing methods for our API respond to endpoints defined at the root of our project. For example, visiting `exampleDomain.com/notes` would give us the collection of notes from our backend. The issue lies in the fact that we're serving our frontend from the same address as our backend, so now, we couldn't have a route for our frontend at `exampleDomain.com/notes` that served up a nice frontend page. As the backend's only purpose (other than serving up static assets in this case) is to provide an API for our frontend, it is best that we separate our API addresses out by adding a naming convention. In this case, we'll be adding `/api/` before all routes pertaining to our API (in this case, we should update all uses of routing methods as our middleware takes care of serving the static content).

Let's first update our backend routing method usage to reflect these new endpoints.

**/Backend/index.js**

For example, we would convert the update routing method handler function

```js
//...

app.get("/notes/:id", (req, res) => {
  const id = Number(req.params.id); // Takes in the parameter from the URL and casts it to a number
  const note = notes.find((note) => note.id === id); // Finds the note that matches the parameter
  res.json(note);
});

//...
```

to

```js
//...

app.get("/api/notes/:id", (req, res) => {
  const id = Number(req.params.id); // Takes in the parameter from the URL and casts it to a number
  const note = notes.find((note) => note.id === id); // Finds the note that matches the parameter
  res.json(note);
});

//...
```

We'll repeat this process for every handler function in **index.js**.

After this, we can update the `baseUrl` constant in our frontend in order to point our requests to the right endpoints.

**/Frontend/src/services/notes.js**

```jsx
import axios from "axios";
const baseUrl = "/api/notes";

//...
```

Now, whenever we make request to our API, we'll be making requests to `exampleDomain.com/api`. For example, the endpoint for our notes collection would be `exampleDomain.com/api/notes`.

If you have tried to employ the practice of not deploying to our production server in order to test our changes, you might have encountered the issue of our frontend not being able to retrieve the notes when running our application locally.

This is because our `baseUrl` now refers to a relative path to which our browser assumes that the frontend is being served from the same location (same domain, port, etc.), so it just adds the relative path to the current path. This doesn't work when running our application locally, since, in development mode, the backend runs on a different port, `localhost:3001`, while our frontend is being served from `localhost:3000`. When our browser encounters this relative path, it assumes that the path described is `localhost:3000/api/notes` which is incorrect.

We can easily fix this issue by adding a `proxy` value to the **package.json** file in our frontend directory as follows

```json
{
  //...

  "proxy": "http://localhost:3001"

  //...
}
```

This change tells React to proxy API requests. A proxy is simply tasked with forwarding requests to other servers for further communication. Essentially, what this declaration does is tell the React application to redirect any requests that aren't managed by React (fetching other JavaScript files or CSS files) to the following address, effectively making this address the 'current address' that will be appended to with relative addresses. For example, when we use `baseUrl` in cases where it is evaluated as a path, React will recognize that it is not a reference to a static asset and will proxy the request to the specified proxy (with whatever relative path appended onto it).

This is just one way to manage deploying both a frontend and backend to the Internet (as well as how to work on the project locally). There are a few disadvantages to this approach, but we'll explore some options that are potentially better in certain cases later.

### Debugging in the backend

You might have wondered how we would go about debugging in the backend since we don't have access to Chrome Developer Tools when using Node.

We can always print to the console to debug in certain situations. This is often a quick and easy way to see what's going on with our code and potentially identify a potential source of errors.

Beyond printing to the console, Visual Studio code offers a built-in debugger that we can use to set breakpoints, step through our code, etc.

Under the **Debug** menu, there is an option to start our current file in debugging mode:

![image-20191111193747574](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111193747574.png)

This makes a variety of tools available to us. For example, if we set a breakpoint at this line in our `/notes` handler function as follows

![image-20191111194101156](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111194101156.png)

Upon visiting **localhost:3001/api/notes**, we will reach this breakpoint and we can inspect the state of our application at this point in execution

![image-20191111194311862](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111194311862.png)

Another option for debugging is using the Chrome Developer Tools (the tools we've been using with our frontend) with Node itself.

We can do this by starting our application with the `--inspect` flag as in the following command

```bash
node --inspect index.js
```

We'll see the following messages in the console

![image-20191111194504675](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111194504675.png)

We can then open up the console in Chrome on any webpage and click the green Node logo in the upper left of the panel.

![image-20191111194815877](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111194815877.png)

Upon clicking this button, the following window should appear

![image-20191111194915154](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111194915154.png)

These addresses point to the address given to us in the console output earlier. We can view the **Console** tab or the **Sources** tab as we would regularly with Chrome Developer Tools. We can also set breakpoints like we did earlier with VSCode by simply clicking on a line number under the **Sources** tab

![image-20191111195252627](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111195252627.png)

It is important to learn how to effectively debug as our programs grow in complexity (more components, the introduction of a database, interacting with APIs, etc.). With this, always be aware of the tools at your disposal and learn to step through your code and question the function of blocks of your code in order to attempt to figure out problems occurring (or problems that may occur) within your code.

## Introduction to Databases (MongoDB)

Now, if we wanted our data to actually have persistence, we would have to save our data in a database. We'll be using MongoDB which is a flexible, document based database.

You may have used a relational database before. As MongoDB is a document database, it differs in how data is organized and how you would query for that data. These types of databases are usually referred as to NoSQL (not only SQL) databases where SQL is a querying language.

MongoDB stores BSON documents (binary representation of JSON documents that contains more data types than JSON supports) in things called collections. Collections are analogous to tables in relational databases in that they store documents (akin to a table storing 'rows' in a relational database). What is referred to as 'Databases' in MongoDB simply hold collections to which these collections hold documents.

**Database -> Collections -> Documents**

You have the option of running MongoDB locally on your computer, but for the sake of not having to work through installing and configuring our database, we'll simply use a cloud provider as there are a few free options.

Let's take a look at using **MongoDB Atlas** to provide our database.

First, create an account on MongoDB Atlas. Upon signing in, you'll be prompted to create a cluster via a prompt. If you do not see the prompt, you can press the 'Build a New Cluster' button in the top right corner

![image-20191111233957886](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111233957886.png)

Then, we can select AWS as our cloud provider and choose a region with a free tier available in order to create a free tier cluster. Leave the rest of the options unchanged.

![image-20191111234108951](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111234108951.png)

After pressing create, we should see a cluster provisioning on our dashboard.

![image-20191111234247951](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111234247951.png)

**After about 10 minutes**, we should be ready to get started.

Once our cluster is created, we can dive into the **Database Access** tab in order to create some credentials for our server to connect to our database with.

![image-20191111235154600](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111235154600.png)

Press 'Add New User' to proceed. Upon pressing 'Add New User', we are presented with a modal allowing us to provide the username, password, and privileges for our user. You can use anything as your username and password, just keep in mind that we'll have to enter these credentials at some point in our server. We then give the user the ability to read and write to any database.

![image-20191111235510018](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111235510018.png)

Then, we have to change our security policy to tell our database to allow requests from our backend server's IP address. We do this by navigating to the **Network Access** tab and clicking the **Add IP Address** button. For simplicity, we'll just press the **Allow Access From Anywhere** button which will allow any IP to connect.![image-20191111235858389](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191111235858389.png)

Then, press confirm.

After this, we can finally connect to our database from our backend server. Let's navigate to the **Clusters** tab and press the **Connect** button.![image-20191112000024698](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112000024698.png)

Then, select **Connect Your Application**.

![image-20191112000156137](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112000156137.png)

This should finally yield us our **Connection String** which is the URI for our database as pictured below.![image-20191112000332470](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112000332470.png)

In this case, the connection string that MongoDB Atlas provided us with is

```
mongodb+srv://exampleuser:<password>@cluster0-nfeqs.mongodb.net/test?retryWrites=true&w=majority
```

which is the _MongoDB URI_, where URI stands for Uniform Resource Identifier. A URI String of characters that identify resources on the Internet. For example, a URL is an example of a URI.

### Using MongoDB in Express

**Do not execute or add the code blocks in this section to any files. They are merely here to illustrate the concepts of creating and saving objects. We will work with Mongoose directly in our existing notes application later.**

Now that we have our database created and the URI for it, we can start using it in our Express application. In order to connect to our database and interface with it, we'll be using **Mongoose** which is an **object document mapper** that offers a higher level API over the official MongoDB driver (essentially makes things easier to understand and use at the expense of some performance and flexibility). It does this by adding an abstraction layer above the native driver.

To illustrate the higher level nature of Mongoose and how it might ease the development process. The following two blocks of code achieve the same task.

Using Mongoose:

```js
Book.find({ released_in_year: 2015 }, "title author");
```

Using the native driver

```js
db.book.find({released_in_year:{'2015'}}, {title: true, author: true})
```

The differences become more pronounced (with the native driver code becoming much more complex) as our schema extends in complexity beyond this simple example.

With this, let's install Mongoose by running the following command in our backend directory

```bash
npm install mongoose
```

Then add the following to **/Backend/index.js**

```js
//...

const mongoose = require("mongoose");

const url =
  "mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority";

mongoose.connect(url, { useNewUrlParser: true, useUnifiedTopology: true });

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
});

const Note = mongoose.model("Note", noteSchema);

//...
```

Let's take a look at this code.

First, we store the URI referencing the location of our database in the `url` const. We then call the `connect` method and pass in this `url` as well as an options object in order to actually establish a connection to our database. Notice how we have replaced '<password>' with the actual password for our user as well as changed 'test' following the forward slash (the database name) to 'note-app'. Later, we'll talk about how to hide our password as we are currently just hardcoding the full URI + password in a constant. This may become an issue if you were to ever upload your code or share it.

Then, we define the `noteSchema` by using a JavaScript object constructor to create a new `mongoose.Schema` object (based off a constructor function where construction functions are tasked with creating new JavaScript objects based off of parameters) with the appropriate fields and data types. Then we use our schema definition by converting our `noteSchema` into a model that we can actually work with. We do this with the following line

```jsx
const Note = mongoose.model("Note", noteSchema);
```

When you call `mongoose.model()` upon a schema, Mongoose will compile a model for you. The above code creates a model with the singular name of 'Note'. Mongoose will name the collection for these `Note` models as 'notes' by convention where Mongoose automatically names collections as the plural in lowercase with the schema referring to each document in the singular.

Keep in mind that document databases (like the one we're using now) are schemaless, meaning that they do not care how our data is structured in the database. These are just general guidelines and it is possible to store documents within a collection with completely different fields.

Additionally, you might also encounter the common practice of storing `mongoose.Schema()` within a variable or constant to make our code look slightly better with the usage of `mongoose.Schema()` as follows

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const noteSchema = new Schema({
  content: String,
  date: Date,
  flagged: Boolean,
});
```

Notice how we were able to just use the constant `Schema` rather than directly calling `mongoose.Schema()`.

### Creating New Objects

Models are essentially 'fancy constructors' that are compiled from `mongoose.Schema()` definitions. They are responsible for creating and reading documents.

As our models are actually constructor functions, all of our instances of models have the same properties as provided by the constructor function, meaning that they all include methods that allow us to save our instance to the database and more. These instances of models will now be referred to as their more proper term, documents. The following example illustrates how we might go about creating a `Note` document that makes use of our `Note` model (which is actually a constructor function, giving all these documents the same set of built-in instance methods)

```js
const noteDocument = new Note({
  content: "This document was created using the Note model",
  date: new Date(),
  flagged: false,
});
```

We can now save this document to our database by calling its `save()` instance method which can be provided with an event handler that is called upon a fulfillment of its promise (if the note is successfully saved)

```js
noteDocument.save().then((result) => {
  console.log("noteDocument has been saved");
  mongoose.connection.close();
});
```

Now, when the promise is fulfilled (the note being saved successfully), our event handler provided to the `then()` method of our promise will get called. This event handler prints a message and closes the database connection.

### Fetching Objects

In the context of our example application, we would find a given note by referencing the model, using its `find` method, and defining what we want to look for.

For example, if we wanted to find only notes with the `flagged` property set to true, we would write

```js
Note.find({ flagged: true }).then((result) => {
  result.forEach((note) => {
    console.log("Match found: ", note);
  });
  mongoose.connection.close();
});
```

Upon the fulfillment of this promise (the query being completed), our handler function prints out the results.

### Database Application

Let's introduce a database to our notes application so that its state is persistent. First, let's just copy the code that we used above to 'connect' to our database (and define our Note model) and paste it in **/Backend/index.js**

```js
const mongoose = require("mongoose");

const url =
  "mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority";

mongoose.connect(url, { useNewUrlParser: true, useUnifiedTopology: true });

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
});

const Note = mongoose.model("Note", noteSchema);
```

Let's also go ahead and delete our `notes` array that we store in memory.

Now, we can start rewriting our handler functions to use our database. For example, instead of having the following handler function

```js
app.get("/api/notes", (req, res) => {
  res.json(notes);
});
```

We would write the following handler to use our database as follows

```js
app.get("/api/notes", (req, res) => {
  Note.find({}).then((notes) => {
    res.json(notes);
  });
});
```

Here, we use the `find` method on our `Note` model to find all notes that match the parameter that we passed to it. When we use an empty object as our parameter, we get all of the documents in the collection. Then, in our handler function for the promise that the `find` method had returned, we take in the collection of objects that it returns as our `notes` parameter. We then take the parameter variable and send it as our response using our `res` object's `json` method.

Upon converting this one routing method handler, we can observe the changes by using our browser and visiting **localhost:3001/api/notes**.

![image-20191112232145407](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112232145407.png)

Doing so yields us an empty page. This is because we don't actually have any notes in our database just yet. We can add some test data to our database by running the following code in a new file

\*\*/Backend/create-test-data.js

```js
const express = require("express");
const mongoose = require("mongoose");
const app = express();

const url =
  "mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority";

mongoose.connect(url, { useNewUrlParser: true, useUnifiedTopology: true });

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
});

const Note = mongoose.model("Note", noteSchema);

const firstTestNote = new Note({
  content: "This is our first note in our database!",
  date: new Date(),
  flagged: false,
});

firstTestNote.save().then((result) => {
  console.log("Successfully saved!");
  mongoose.connection.close();
});

const secondTestNote = new Note({
  content: "This is our second note in our database!",
  date: new Date(),
  flagged: true,
});

secondTestNote.save().then((result) => {
  console.log("Successfully saved!");
  mongoose.connection.close();
});

const thirdTestNote = new Note({
  content: "This is our third note in our database!",
  date: new Date(),
  flagged: true,
});

thirdTestNote.save().then((result) => {
  console.log("Successfully saved!");
  mongoose.connection.close();
});
```

Upon running this 'application' with Node, we should see the following as console output

![image-20191112232829913](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112232829913.png)

This means that we have successfully created three test note objects that live on our database in our **notes** collection. If we take a peek at the **Collections** panel of our MongoDB Atlas cluster, we can see how our new `Note` documents lie under the `notes` collection.

![image-20191112233259744](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112233259744.png)

Notice how we did not have to provide a name for our collection. As stated earlier, Mongoose will automatically take our model name and make it lowercase and plural. For example, documents created under the 'Splotch' model would fall under the 'splotches' collection. If you need to, you can explicitly name the collection when creating a model by passing it as a third model. For example, if we wanted our notes to be stored in a collection called 'notesCollection', we would use the following to create our model

```js
const Note = mongoose.model("Note", noteSchema, "notesCollection");
```

Going back to our application, after initializing our database with some test data, visiting **localhost:3001/api/notes** now yields the following result

![image-20191112233725952](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112233725952.png)

Here, you'll notice the introduction of a few strange fields (`_id` and `__v`).

`__v` is simply the version key of our document. The version key is a property set on each document when it is created by Mongoose. It represents the _internal revision_ of the document (basically keeps track of how many modifications had happened that may change the position of an element in an array); this mitigates the issue of referring to the wrong sub-document or document when dealing with arrays that may have elements with changing indexes. With this, we can refer to elements that match the version number as well as the document we're looking for to mitigate some issues with targeting the wrong document and more.

Now, the frontend doesn't have much use for the `__v` property and it is looking for an `id` field rather than `_id`, so how might we go about sending objects in the correct 'format' to our frontend?

We can do this by updating the `toJSON` method of our documents of our note schema. We do this by the `set` method of a schema definition. This allows us to specify a built-in method of the documents that would be created under this schema definition and update it.

**/Backend/index.js**

```js
//...

noteSchema.set("toJSON", {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString(); // Just in case, we convert the _id field of our object to a string as it is in fact an object (even though it is displayed like a string)
    delete returnedObject._id;
    delete returnedObject.__v;
  },
});

//...
```

Here, we updated the `toJSON()` method of our `Note` documents by specifying the method as our first parameter and the updated object property as the second parameter to the `set()` method of `noteSchema`.

This `transform` property now has a function that takes in the `returnedObject`, sets the `id` property to the `_id` property converted to a string, and deletes the `_id` and `__v` fields as its value.

This new `id` field is quite similar to a virtual. A virtual field is essentially an attribute that is nice to have (for sending data to the client, etc.) but isn't actually persisted (saved) to MongoDB. This applies to our `id` field as we aren't actually saving the `id` field to our database, but we are creating it when calling the `toJSON()` method in order to make things easier on the frontend.

We can then use this in our routing event handler by formatting every note with the `toJSON` method.

**/Backend/index.js**

```js
//...

app.get("/api/notes", (req, res) => {
  Note.find({}).then((notes) => {
    res.json(notes.map((note) => note.toJSON()));
  });
});

//...
```

Visiting **localhost:3001/api/notes** now yields the following result

![image-20191113000245903](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191113000245903.png)

You can now see how our updated `toJSON()` document method copies the `_id` field over to a new `id` field and strips the `__v` and `_id` fields.

#### Organization with Mongoose

Since things are getting pretty messy again, let's take advantage of our ability to move things into separate files and create a module for our Mongoose code. Most project structures follow the paradigm of moving Mongoose code and models into a folder called **models** that resides in our root backend directory.

**/Backend/models/note.js**

```js
const mongoose = require("mongoose");
const uri = process.env.MONGODB_URI;

console.log("Connecting to: ", uri);

mongoose
  .connect(uri, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => {
    console.log("Connected to MongoDB");
  })
  .catch((error) => {
    console.log("Error connecting to MongoDB:", error.message);
  });

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
});

noteSchema.set("toJSON", {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
  },
});

module.exports = mongoose.model("Note", noteSchema);
```

In this module, we simply define the schema for a note and assign it to the `noteSchema` constant. Then we update the note schema's `toJSON()` method as we did earlier. We also handle connecting to the database within this module. Take a look at the `url` constant. We no longer 'hardcode' the address, password and all, within the code as that is not wise with regard to security if we were to share our code or upload it. We now pull in the address through the use of an environment variable. We'll talk about defining environment variables in just a moment.

Finally, we export the model that is compiled for us based upon our `noteSchema`. Notice how these CommonJS modules are slightly different than ES6 modules with regard to importing and exporting. AS we only exported the model, we do not have access to anything else defined in this module (such as the schema, etc.).

Now, we can make use of this model (which happens to be the only thing we really need from that file for most tasks) by importing it as follows in **index.js**

**/Backend/index.js**

```js
//...
const Note = require("./models/note");
//...
```

where the `Note` constant now holds whatever was exported in the **./models/note** module.

Now, let's talk about how we would go about defining the value of an environment variable so we can actually start our application.

One simple way to go about defining an environment variable is to just pass it in when starting the application

```bash
MONGODB_URI=example_address npm start
```

Now, this may not be ideal as it will become cluttered with just a few environment variables, and we have to input it every time we want to run our application. This brings us to the next method: using the `dotenv` library.

Let's install `dotenv` with the following command

```bash
npm install dotenv --save
```

Then, we can create the file that `dotenv` will pull our environment variables from. Let's create a file called **.env** at the root of our backend directory. Then, we can specify the value of our `MONGO_URI` environment variable which would just be our connection string.

![image-20191113105819413](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191113105819413.png)

Recall how our `PORT ` constant in **index.js** takes in an environment variable and defaults to 3001. We can now manually assign our `PORT` constant in **.env** by adding a line to our file specifying the value so that our file now looks like

**/Backend/.env**

```env
MONGO_URI=mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority
PORT=3001
```

We should also add our **.env** file to **.gitignore** since we don't want it being uploaded to any cloud repository where it could potentially be exposed.

**/Backend/.gitignore**

```text
node_modules
.env
```

Now, let's just import `dotenv` into our **index.js** file so that we can use our environment variables. Let's be sure to import `dotenv` before our models and any imports that may use environment variables as we need to make sure that the environment variables are available globally before anything tries to use it.

### Using the Database with Other HTTP Methods

Let's start transitioning the other routing handlers to use our database. Let's first update our POST routing handler that allows us to create a new note

**/Backend/index.js**

```js
//...

app.post("/api/notes", (req, res) => {
  const body = req.body;

  if (!body.content) {
    return res.status(400).json({
      error: "No content specified in note body",
    });
  }

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  newNote.save().then((savedNote) => {
    res.json(savedNote.toJSON());
  });
});

//...
```

Like we did earlier, we create a new note object using the `Note` model constructor function. Compared to the earlier block of code, we do something slightly different. Now, upon the fulfillment of our promise involving saving the note to our database, we send the response back containing the note (converted by our modified `toJSON()` method). We put this call to `response.json()` inside the `then()` method of our promise to ensure that we only send a response containing our note back if the note had indeed been saved. We can also get rid of our `generateNoteId()` function as Mongoose will now handle that for us.

We'll add functionality related to handling any errors that might occur with saving our note later.

Let's also quickly update our handler for fetching an individual note

**/Backend/index.js**

```js
//...

app.get("/api/notes/:id", (req, res) => {
  Note.findById(req.params.id).then((note) => {
    res.json(note.toJSON());
  });
});

//...
```

This code block opens the possibility for a severe error.

### Error Handling

Upon trying to visit the URL for a note that doesn't exist in our database, for example **localhost:3001/api/notes/this_id_does_not_exist**, our browser will wait indefinitely for our server to respond. Looking at the console, we're presented with the following error:

![image-20191113200254754](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191113200254754.png)

This error points to us not handling the rejection of our promise (in this case, involving the promise created when we attempt to retrieve a particular note of a certain ID from our database). We can easily fix this by using the `catch()` method of our promise to define an error handler.

**/Backend/index.js**

```js
//...

app.get("/api/notes/:id", (req, res) => {
  Note.findById(req.params.id)
    .then((note) => {
      res.json(note.toJSON());
    })
    .catch((error) => {
      console.log(error);
      response.status(404).end();
    });
});

//...
```

Upon the rejection of our promise, we respond to the request with the `404` status code while also printing out information relating to the error to the console.

We actually go a bit further with our error messages here as we can encounter two potential errors with regard to the `id` parameter.

One involves the `id` parameter in our URL not matching the Mongo identifier format (the one we experienced above) where we'll get the following error message

```error
{ CastError: Cast to ObjectId failed for value ...
```

Another type of error occurs when the `id` parameter is indeed in the correct format, but our database is not able to find a note with that `id`. For example, if we made a request to **localhost:3001/api/notes/5dccb35145b29e3cca6633bf** where a note actually has the `_id` '5dccb35145b29e3cca6633bf'. When this happens, we'll get the following message

```error
TypeError: Cannot read property 'toJSON' of null ...
```

Let's now change our code to take advantage of these two different error types in order to provide our user or frontend with additional information.

**/Backend/index.js**

```js
//...

app.get("/api/notes/:id", (req, res) => {
  Note.findById(req.params.id)
    .then((note) => {
      if (note) {
        res.json(note.toJSON());
      } else {
        res.status(404).end(); // Condition fail
      }
    })
    .catch((error) => {
      console.log(error);
      res.status(400).send({ error: "Incorrect format for id parameter" });
    });
});

//...
```

Here, if our initial promise is fulfilled, this means that the formatting of our `id` was correct. With this, our handler for the fulfillment of our initial promise just checks if the note actually exists by checking if `note` is not undefined (truthy); if this check fails, then the else block demarcated by 'Condition fail' will execute sending an appropriate response code back (`404`) explaining that the note was not found.

If our initial promise is rejected, this means that the the format of our `id` parameter was incorrect. As a result of this, our handler provided to the `catch()` method of our initial promise is executed which prints the `error` object to the console and sends a response back with the `400` status code meaning 'Bad Request'. We also send an error message in the `error` property of the object we sent back telling the user that there was an 'Incorrect format for id parameter'. It's often a good idea to log the entire object as there always may be error cases that we have not accounted for.

####Error Handling Middleware

There are some cases where it might be better to implement everything related to error handling in a single place for organization or reporting errors to an error tracking system.

We can do this by making use of the `next()` function that we have discussed earlier. Recall that the `next()` function executes the middleware succeeding the current middleware.

**/Backend/index.js**

```js
//...

app.get("/api/notes/:id", (req, res, next) => {
  Note.findById(req.params.id)
    .then((note) => {
      if (note) {
        res.json(note.toJSON());
      } else {
        res.status(404).end(); // Condition fail
      }
    })
    .catch((error) => next(error));
});

//...
```

In order to use the `next()` function, we need to pass it in as our third parameter. Here, you can see that we passed in our `error` that was caught to the `next()` routing function. If you pass anything to the `next()` function (except the string `'route'`), Express will regard the given request as an error and will skip any non-error handling routing and middleware functions. Thus, execution will skip to the error handling middleware.

Express comes with a built-in error handler that tries to take care of any errors that you might encounter. This automatically gets added at the end of the middleware function stack. Thus, if we pass an error to `next()` like we did above, the default error handler will handle it. We are also given the option of writing our own error-handling middleware functions. These are quite similar to other middleware function definitions, except error handlers have four arguments (the addition of the `err` (error) parameter) instead of three. We can write an error handler that pertains to our application as follows

**/Backend/index.js**

```js
//...

const errorHandler = (err, req, res, next) => {
  console.error(err.message);

  if (err.name === "CastError" && err.kind === "ObjectId") {
    return res.status(400).send({ error: "Incorrect format for id parameter" });
  }

  next(error);
};

app.use(errorHandler);

//...
```

Here, we simplified the logic involving checking for the type of error by simply just checking if the error was a `CastError` exception with a `kind` of `'ObjectId'` (meaning that our object `id` was not in the correct format). We then return the status code of `400` with an error in the response body upon the fulfillment of these conditions. If we get past this block of code, we simply forward the error to the default error handler that we talked about earlier. At the end of the code snippet, we simply load our new error handling middleware function.

Let's also define another middleware function that handles requests to endpoints that don't exist (for example, **localhost:3001/api/cars**).

**/Backend/index.js**

```js
//...

const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: "Endpoint does not exist (Unknown Endpoint)" });
};

//...
```

With the addition of new middleware, it is important to be aware of the order that we are loading them into Express as they are executed in the same order. Thus, the order of loading middleware should resemble the following

**/Backend/index.js**

```js
//...

app.use(express.static("build"));
app.use(cors());
app.use(bodyParser.json());

//...
// ROUTING METHODS
//...

const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: "Endpoint does not exist (Unknown Endpoint)" });
};

app.use(unknownEndpoint);

const errorHandler = (err, req, res, next) => {
  console.error(err.message);

  if (err.name === "CastError" && err.kind === "ObjectId") {
    return res.status(400).send({ error: "Incorrect format for id parameter" });
  }

  next(error);
};

app.use(errorHandler);
```

We define and use the `unknownEndpoint` and `errorHandler` after all of our routing methods since we don't want them to be loaded before them since they'll be executed before all of our routing method requests leading to undesirable behavior. For example, if we wrote the following loading order

```js
const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: 'Endpoint does not exist (Unknown Endpoint)' })
}

app.use(unknownEndpoint)

// ROUTING METHODS
app.get('/api/notes', (req, res) => {
//...
```

Our `unknownEndpoint` middleware would be added and executed before the routing method handler. As this middleware responds to all requests, none of our routing handlers would work, thus we need to order this piece of middleware after every specific endpoint that we want to target since this function will handle every endpoint.

As per the Express documentation, we must define error-handling middleware last (after all other `app.use()` and routes calls)

### Deleting and Updating Documents

We can now go ahead and add the functionality to delete and update our notes (the flagged property).

The `findByIdAndRemove()` method provides us with an easy way to find a specific document (a note) and delete it. We can update our routing handler as follows

**/Backend/index.js**

```js
app.delete("/api/notes/:id", (req, res, next) => {
  Note.findByIdAndRemove(req.params.id)
    .then((result) => {
      res.status(204).end();
    })
    .catch((error) => next(error));
});
```

Here, even if we attempt to delete a note that does not exist (via an invalid `id` parameter), we respond with the status code `204` (No Content) unless the promise is rejected. If we wanted to provide two different status codes depending on if we actually deleted a note or if there is no note that matches the provided `id`, we could make use of the `result` object that is passed into our callback function provided to the `then()` method. You can also see how we pass on our error to the `next()` function were our promise rejected.

We can employ a similar function in order to update the `flagged` property of a given `Note` document in our database. Using the `findByIdAndUpdate` method, we can fetch a specific note and update it by writing the following

```js
app.put("/api/notes/:id", (req, res, next) => {
  const body = req.body;

  const note = {
    content: body.content,
    flagged: body.flagged,
  };

  Note.findByIdAndUpdate(req.params.id, note, { new: true })
    .then((updatedNote) => {
      res.json(updatedNote.toJSON());
    })
    .catch((error) => next(error));
});
```

Here, we create a new `note` object using the request object's body. This note object was not created using the `Note` model constructor function as this note is not directly involved with the database, but rather is used as a regular JavaScript object to replace an object in our database. Then, we find the note object in our `notes` collection that matches the `id` request parameter and replace it with our `note` object.

Taking a look at our call to `findByIdAndUpdate`, the first parameter is the value of `_id` of the document that we want to replace/update. The second parameter is the actual object that we want to replace the object that we have selected with. Finally, the third parameter is the options object.

In the above example, we take advantage of this options object by giving the `new` boolean property a value of `true`. This returns the modified document (the document in our database after we have replaced it) rather than the original document. This emulates the behavior that we had originally in our application where we returned the updated `note` rather than the original `note` that Mongoose would have were it not provided with this option.

Upon testing these new features with Postman (or VSCode) we can see that most aspects of our server work just fine. Unfortunately, when attempting to update a note using our HTTP PUT routing method handler (to update the flagged property of a note object) or to delete a note using our HTTP DELETE routing method handler, we encounter the following error

![image-20191115113347499](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191115113347499.png)

Upon looking at the error message and the relevant documentation provided, we can fix this issue simply by adding the following line to **note.js**

**/Backend/models/note.js**

```js
const mongoose = require("mongoose");
mongoose.set("useFindAndModify", false);

//...
```

As Mongoose had a `findOneAndUpdate()` function before the native MongoDB driver had its own `findOneAndUpdate()` function, Mongoose will default to using the native driver's `findAndModify()` function instead. We can make Mongoose use the `findOneAndUpdate()` function by setting the `useFindAndModify` global option to `false` as we did with the command above.

As this is a global option, instead of adding the above code snippet to **note.js** we can also just add it to our connection options as follows

```js
const mongoose = require("mongoose");
const uri = process.env.MONGODB_URI;

console.log("Connecting to: ", uri);

mongoose
  .connect(uri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
  })
  .then(() => {
    console.log("Connected to MongoDB");
  })
  .catch((error) => {
    console.log("Error connecting to MongoDB:", error.message);
  });

//...
```

Upon testing note deletion and updating notes, we no longer receive the error message mentioned above.

## Validation with Mongoose

Now, we don't want 'bad' data entering our database as that could lead to a plethora of problems later on. With this, we can use validation to apply constraints to data entering our database to handle these issues.

Previously, we had done a little bit of validation in our routing method handler by just checking if the note object was empty and returning the appropriate response

```js
//...

app.post('/api/notes', (req, res) => {
  const body = req.body

  if (!body.content) {
    return res.status(400).json({
      error: 'No content specified in note body'
    })
  }

//...
```

However, this is not nearly the best way to validation. One better way of handling validation is by using Mongoose to validate our data before storing it. We can do this by providing validation rules in the schema for our models. Let's update our `noteSchema` object with some validation rules

**/Backend/models/note.js**

```js
const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    minlength: 1,
    required: true,
  },
  date: {
    type: Date,
    required: true,
  },
  flagged: Boolean,
});
```

Validation is middleware. Delving deeper into Mongoose, there are four types of middleware in Mongoose: document middleware, model middleware, aggregate middleware, and query middleware. Mongoose comes with several built-in validators that we can define in the `SchemaType`. We use the `required` and `minlength` validators above. The `required` validator ensures that the value of this property cannot be missing, meaning that it must be defined. The `minlength` validator sets the minimum length that the string value can have, where we set it at 1. This also implicitly defines the `required` validator as the minimum length logically sets that there should be a definition; thus, we would not have to explicitly define the `required` validator.

Strings come with `enum`, `match`, `minlength`, and `maxlength` validators. Numbers come with `min` and `max` validators. All schema types come with the `required` validator. Mongoose also allows for the creation of custom validators if we need validators that aren't satisfied by the built in ones.

Now, attempting to store an object that breaks any of the above restrictions in our code will throw an exception. Let's `catch` this exception and pass it to our error handler middleware as follows

**/Backend/index.js**

```js
//...

app.post("/api/notes", (req, res) => {
  const body = req.body;

  if (!body.content) {
    return res.status(400).json({
      error: "No content specified in note body",
    });
  }

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  newNote
    .save()
    .then((savedNote) => {
      res.json(savedNote.toJSON());
    })
    .catch((error) => next(error));
});

//...
```

Let's also add a condition in our error handler to provide a specific response for this type of error

**/Backend/index.js**

```js
//...

const errorHandler = (err, req, res, next) => {
  console.error(err.message);

  if (err.name === "CastError" && err.kind === "ObjectId") {
    return res.status(400).send({ error: "Incorrect format for id parameter" });
  } else if (err.name === "ValidationError") {
    return res.status(400).json({ error: err.message });
  }

  next(err);
};

//...
```

![image-20191117114024042](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117114024042.png)

Upon trying to send the above request that breaks the required validation rule for the content property, we get the following response

![image-20191117114009587](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117114009587.png)

If you look at the above code for saving our note, it is quite messy. We can make it look cleaner by making use of promise chains as discussed earlier.

Recall then `then()` method of a promise returns its own promise which will return the expression after the `return` keyword upon its fulfillment as its value. We can then access this value in the callback function provided to the `then` method of this new promise. This makes it possible to return items within the function passed into `then()` in which these returned items will be passed down to the next promise in the chain.

We can format the above code with this in mind as follows

**/Backend/index.js**

```js
app.post("/api/notes", (req, res) => {
  const body = req.body;

  if (!body.content) {
    return res.status(400).json({
      error: "No content specified in note body",
    });
  }

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  newNote
    .save()
    .then((savedNote) => savedNote.toJSON())
    .then((formattedSavedNote) => {
      response.json(formattedSavedNote);
    })
    .catch((error) => next(error));
});
```

This makes it much easier to follow along and see what's happening in a linear fashion. Here, we simply save the note and take in the `savedNote` object that was returned by Mongoose. This `savedNote` object is then formatted by our `toJSON()` function in the callback function and is returned (taking advantage of the compact arrow function syntax that returns the evaluation of the expression to the right). This creates a new promise who receives the formatted note as its value. We then access the `then()` method of this new promise and in its handler, we actually send the response using `response.json()`.

#### Deploying to Production with Environment Variables

Now, to deploy our application with our new changes to production, we just need to make one small change. As our **.env** file is not tracked by Git, it will not be sent to our remote Heroku server. Thus, Heroku has no idea what the values for our environment variables are.

If we tried to run the server without changing anything using our deployment script:

```bash
npm run deploy:full
```

> You must set Heroku as the remote if you are initializing a new Git repository in a different directory
>
> 1. Run `git init .`
> 2. Run `heroku create`
> 3. Run `npm run deploy:full`

You'll notice the following in our logs

![image-20191117123245641](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117123245641.png)

It looks like the variable for our connection string is evaluating to `undefined`. This is because, as mentioned earlier, our **.env** file is not tracked, so our remote does not have it. Thus, the server on the Heroku remote tries to pull its value from the file when it runs into it in the code but is unable to. We can remedy this by specifying environment variables system-wide using the Heroku CLI:

```bash
heroku config:set MONGODB_URI=mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority
```

With this, our application starts successfully and we can access it as we did before.

### Linting

A linter is a very useful and important tool in software development. Linting (or static-checking) involves checking your code for programmatic and stylistic errors. A lint or linter is simply a program that supports linting. Linting is especially useful for interpreted languages such as JavaScript (although this classification isn't 100% correct as modern browsers support JIT compilation for JavaScript) or Python as these languages don't have to be compiled to display linting errors.

Let's make use of a popular JavaScript linter, ESLint. First, let's install ESLint using npm as follows:

```bash
npm install eslint --save-dev
```

We use the `--save-dev` flag as we do not need `eslint` to be installed on our production server(s) as its only use is for checking our code during development.

Then, we can initialize ESLint by running it with the `--init` flag

```bash
node_modules/.bin/eslint --init
```

We then specify the following options

![image-20191117130650549](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117130650549.png)

Note how we specify that our code only runs with Node, and that we are not using any frameworks since we are initializing only in our backend directory for now.

This creates a file called **.eslintrc.js** within the directory that we have executed this command.

**/Backend/.eslintrc.js**

```js
module.exports = {
  env: {
    commonjs: true,
    es6: true,
    node: true,
  },
  extends: "eslint:recommended",
  globals: {
    Atomics: "readonly",
    SharedArrayBuffer: "readonly",
  },
  parserOptions: {
    ecmaVersion: 2018,
  },
  rules: {
    indent: ["error", 4],
    "linebreak-style": ["error", "unix"],
    quotes: ["error", "single"],
    semi: ["error", "never"],
  },
};
```

Let's also add a few additional rules to make our code cleaner

```js
//...

		'eqeqeq': 'error',
    'no-trailing-spaces': 'error',
    'object-curly-spacing': [
      'error',
      'always'
    ],
    'arrow-spacing': [
      'error',
      { 'before': true, 'after': true }
    ],
    'no-console': 0
  }

```

We can individually evaluate a file using ESLint via the command line by running ESLint followed by a path to a file as follows

```bash
node_modules/.bin/eslint index.js
```

We can also evaluate entire directories. Let's check our entire backend by running

```js
node_modules/.bin/eslint .
```

![image-20191117131202469](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117131202469.png)

We immediately are presented with a ton of issues. Most of them are related to spacing, so we can easily remedy that by running the following command

```bash
node_modules/.bin/eslint . --fix
```

This reduces the number of errors that we have to deal with dramatically.

![image-20191117131335166](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117131335166.png)

Let's deal with these errors progressively. A smarter way of dealing with these errors is to install a lint plugin that allows us to see our errors immediately in our editor.

Let's install the 'ESLint' plugin available on Visual Studio Code.

![image-20191117131526334](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117131526334.png)

We can now see various problems in our code underlined. Upon hovering, we can see additional information relating to the linter rule that was violated and potential fixes.

![image-20191117132605255](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117132605255.png)Web Development 3

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

You'll notice the addition of a few new folders and files in our root directory here. Let's start discussing their functions and the various changes that we'll have to make to our notes application.

First, one change that we'll make is to put everything that isn't involved with starting our application into **app.js**. Then, **index.js** will import our `app` and start the server using that component as follows

**/Backend/index.js**

```js
const app = require("./app"); // Our `app` module that is started by this file
const http = require("http");
const config = require("./utils/config");

const server = http.createServer(app); // Creates the server using the `app` module

server.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT}`);
});
```

Here, you'll notice references to a `config` constant that seems to be imported from the file **./utils/config**.

**/Backend/utils/config.js**

```js
require("dotenv").config();

let PORT = process.env.PORT;
let MONGODB_URI = process.env.MONGODB_URI;

module.exports = {
  MONGODB_URI,
  PORT,
};
```

Here is where we handle our environment variables and export them for use in other modules via importing this configuration module.

We also moved the routes dealing with our `notes` end point to its own file under the **/controllers** directory. The handlers for our route methods are known as _controllers_, thus they will now fall under the **/controllers** directory for the sake of organization. We also take advantage of a few more Express features here as shown below

**/Backend/controllers/notes.js**

```js
const notesRouter = require("express").Router();
const Note = require("../models/note");

notesRouter.post("/", (req, res) => {
  const body = req.body;

  if (!body.content) {
    return res.status(400).json({
      error: "No content specified in note body",
    });
  }

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  newNote
    .save()
    .then((savedNote) => {
      res.json(savedNote.toJSON());
    })
    .catch((error) => next(error));
});

notesRouter.get("/", (req, res) => {
  res.send("<h1>Welcome to our Notes Application Backend!</h1>");
});

notesRouter.get("/", (req, res) => {
  Note.find({}).then((notes) => {
    res.json(notes.map((note) => note.toJSON()));
  });
});

notesRouter.delete("/:id", (req, res, next) => {
  Note.findByIdAndRemove(req.params.id)
    .then((result) => {
      res.status(204).end();
    })
    .catch((error) => next(error));
});

notesRouter.get("/:id", (req, res, next) => {
  Note.findById(req.params.id)
    .then((note) => {
      if (note) {
        res.json(note.toJSON());
      } else {
        res.status(404).end(); // Condition fail
      }
    })
    .catch((error) => next(error));
});

notesRouter.put("/:id", (req, res, next) => {
  const body = req.body;

  const note = {
    content: body.content,
    flagged: body.flagged,
  };

  Note.findByIdAndUpdate(req.params.id, note, { new: true })
    .then((updatedNote) => {
      res.json(updatedNote.toJSON());
    })
    .catch((error) => next(error));
});

modules.exports = notesRouter;
```

Here, you'll notice how we took in the `Note` model so that we could reference our database from within these controllers. We also took advantage of `Router` in Express. A `Router` is like its own mini Express application by providing us with context-specific routing APIs (like `get()` and `put()`). Routers are actually a piece of middleware which can be used for defining and associating related routes.

```js
const notesRouter = require("express").Router();

//...
```

This allows us to use routing methods of our `notesRouter` rather than `app`. Additionally, as we can define a base URL for our `notesRouter`, we can shorten our endpoints to reflect them being extensions of a specific base URL. For example, `/api/notes/:id` would turn into `/:id`.

We can then make use of this router in our `app` module where we also register the base URL for this router upon which all these endpoints are appended onto.

**/Backend/app.js**

```js
const config = require("./utils/config");
const express = require("express");
const bodyParser = require("body-parser");
const app = express();
const cors = require("cors");
const notesRouter = require("./controllers/notes");
const middleware = require("./utils/middleware");
const mongoose = require("mongoose");

console.log("Connecting to:", config.MONGODB_URI);

mongoose
  .connect(config.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
  })
  .then(() => {
    console.log("Connected to MongoDB");
  })
  .catch((error) => {
    console.log("Error connecting to MongoDB:", error.message);
  });

// Middleware
app.use(cors());
app.use(express.static("build"));
app.use(bodyParser.json());
app.use(middleware.requestLogger);

// Routers
app.use("/api/notes", notesRouter);

// After router middleware
app.use(middleware.unknownEndpoint);
app.use(middleware.errorHandler);

module.exports = app;
```

In our app component, you'll notice quite a few changes from our app's last checkpoint. First, you'll notice that we now import all of our middleware from **/utils/middleware.js** to which it contains:

**/Backend/utils/middleware.js**

```js
const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: "Endpoint does not exist (Unknown Endpoint)" });
};

const errorHandler = (err, req, res, next) => {
  console.error(err.message);

  if (err.name === "CastError" && err.kind === "ObjectId") {
    return res.status(400).send({ error: "Incorrect format for id parameter" });
  } else if (error.name === "ValidationError") {
    return response.status(400).json({ error: error.message });
  }

  next(error);
};

module.exports = {
  unknownEndpoint,
  errorHandler,
};
```

Then, you'll notice how we now connect to our database from within our `app` module as we only need to maintain one connection to the database for our entire app for now. Our other approach which involved connecting to the database for each model is not ideal. This results in an updated **/models/note.js** file since we no longer need to connect to our database within each module:

**/Backend/models/note.js**

```js
const mongoose = require("mongoose");

const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    minlength: 1,
    required: true,
  },
  date: {
    type: Date,
    required: true,
  },
  flagged: Boolean,
});

noteSchema.set("toJSON", {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
  },
});

module.exports = mongoose.model("Note", noteSchema);
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
const palindrome = (string) => {
  return string.split("").reverse().join("");
};

const average = (array) => {
  const reducer = (sum, addition) => {
    return sum + addition;
  };

  return array.reduce(reducer, 0) / array.length;
};

module.exports = {
  palindrome,
  average,
};
```

You might not recognize the `reduce()` array method used above, with this, let's talk about and review some concepts in functional programming (with JavaScript).

## What is functional programming?

Functional programming typically allows for the easier reasoning of logic and higher reuse of code.

### Higher-Order Functions

In JavaScript, all functions are 'values' to which you can create an anonymous function and store it in a variable like so:

```js
var double = (x) => {
  return x * 2;
};

var test = double;

test(2); // returns 4
```

Here, you can see how we manipulate this function as if it were a value, because it is in fact a value just like strings, numbers, etc. This also allows us to **pass functions into other functions**.

Functions that take functions as parameters are typically referred to as _higher-order functions_. This allows for **composition** to where we can make a bunch of functions that achieve small tasks and _compose_ them into bigger functions that achieve a bigger task.

We've dealt with higher order functions in the past, `filter()`, `map()`, etc.

For example, let's just say that we wanted to filter an array by a property's value, in this case, we'll be looking to filter for only people whose favorite color is red.

Attempting that filter with a regular for-loop would look something like this:

```js
var filtered = [];
for (var i = 0; i < people.length; i++) {
  if (people[i].favoriteColor === "red") {
    filtered.push(people[i]);
  }
}
```

Now, attempting to do the above in a functional way using a higher-order function would be written as follows:

```js
var filtered = people.filter((person) => person.favoriteColor === "red");
```

> Recall that the `filter()` higher-order function appends the current iterable object if the function (passed in as a property) returns true. It will pass in every single object through the function provided as a property.

These functions allow us to write much cleaner code than we would usually be able to with using a regular for-loop. It is also much easier to follow what's going on with regard to logic.

Taking advantage of composition (although its benefits are not too apparent in this example as these functions are quite simple), we can separate out the callback and compose everything together later in the code as follows:

```js
var isFavoriteColorRed = (person) => person.favoriteColor === "red";
var filtered = people.filter(isFavoriteColorRed);
```

Aside from being easier to read, this also allows for the easier reuse of logic and functionality. For example, let's say that we wanted to now filter **out** people with the favorite color red (instead of only including them). We could pass in the `isFavoriteColorRed` function to the `reject()` higher-order function to achieve that as follows

```js
var filteredOut = people.reject(isFavoriteColorRed);
```

Using the traditional for-loop approach would require rewriting multiple lines of code and would result in logic that is harder to follow. Looking at the `isFavoriteColorRed` function, you'll also notice that we never write `return` but rather take advantage of implicit returns to which if our logic for our function fits in one line, we can implicitly return the evaluation of the expression to the right of our 'arrow'.

### Reduce

Now, let's talk about a higher-order function that you have not been previously introduced to (except for the code block above where we make use of this function). `reduce()` is used for any array transformation that you cannot express with `map()`, `filter()`, etc. This is the function to fall back on if you can't find a prebuilt array transformation.

For example, let's just say we wanted to transform an array of number values (in an array called `numbers`) into one figure that represented the sum of the array. Let's look into doing that with a traditional for-loop first:

```js
var total = 0;

for (var i = 0; i < numbers.length; i++) {
  total += numbers[i];
}
```

Using `reduce()`, we can convert the above functionality into:

```js
var total = numbers.reduce((sum, number) => sum + number, 0);
```

Just like `map()` and `filter()`, `reduce()` still takes in a callback function, but it also takes in an object to be used as a 'starting point'. Basically, what's happening here is that our first parameter of our callback function, `sum`, is being initialized with the value provided as the second parameter of the `reduce()` method. Then, for each iteration upon `numbers` we are passing the current object pertaining to our iteration into the second parameter of our callback function, to which this number is added to `sum`. This continues throughout the array, to which `sum` is added to every time. Every time an iteration is completed, the new `sum` value will be passed to the callback function for the next iteration with the next object in the array passed as the second parameter.

This initially can be hard to understand coming from more traditional approaches to programming, but it is important to attempt to understand these concepts as there are some clear advantages to functional programming in web development.

With this, we can now circle back to our test application to which we make use of the `reduce()` higher-order function.

#### Testing, continued

Looking at our previous test code, you'll now be able to understand how our `average` function works with the `reduce()` higher-order function.

```js
//...

const average = (array) => {
  const reducer = (sum, addition) => {
    return sum + addition;
  };

  return array.reduce(reducer, 0) / array.length;
};

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
const palindrome = require("../utils/test_example").palindrome; // Grabs the `palindrome` function from our 'for_testing' module

test("Palindrome of z", () => {
  const result = palindrome("z");

  expect(result).toBe("z");
});

test("Palindrome of test", () => {
  const result = palindrome("test");

  expect(result).toBe("tset");
});

test("Palindrome of palindrome", () => {
  const result = palindrome("palindrome");

  expect(result).toBe("emordnilap");
});
```

Upon viewing this code in the editor, we're faced with the following:

TODO: Find image image-20191126131154686.png

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
  const result = palindrome("z");

  expect(result).toBe("z");
};
```

This function performs the function that we want to test, `palindrome`, upon a predetermined value. We then store this result so that we can compare and verify the result with an expected value. We do this by calling the `expect()` function which takes in the result of our test. We then can compare it by using **matcher functions** (made available to us via methods upon the `expect()` function). In this case, since we're just comparing two strings together, we can just use the `toBe()` matcher function to which we pass in our expected value.

With this, we can execute all our tests and see if our application behaves as expected.

TODO: Find image image-20191126134650097.png

From this, we can see that all of our unit tests have passed.

Now, you might be wondering what Jest gives us were a test to fail. Let's provide a false expectation to see what it gives us by changing the expectation in our first function to 'a':

```js
() => {
  const result = palindrome("z");

  expect(result).toBe("a"); // Incorrect
};
```

Running our tests now yields the following output

TODO: Find image-20191126141051765.png

Here, you can see that Jest provides us with rich error messages relating to the test that failed, the received value, the expectation involved, and more. These error messages are particularly useful when dealing with errors that may not be easy to identity. Let's look into an example of this in our tests relating to the `average` function.

Let's create a test file **average.test.js** and add the following to the file

**/Backend/tests/average.test.js**

```js
const average = require("../utils/test_example").average;

describe("Average", () => {
  test("of one value is the same value", () => {
    expect(average([1])).toBe(1);
  });

  test("of many", () => {
    expect(average([1, 2, 3, 4, 5, 6])).toBe(3.5);
  });

  test("of an empty array is 0", () => {
    expect(average([])).toBe(0);
  });
});
```

Here, we introduce a new function, `describe`, which wraps tests into collections to which we can give the group a name

```js
describe("Name", () => {
  // Test 1
  // Test 2
  // Test ...
});
```

You'll also notice the descriptions of our unit tests now start with 'of' as Jest in its output will prepend the name of the collection before the test name. This just makes everything much more readable.

Let's try running these tests. The output that Jest gives us uncovers one error

![image-20191126152516561](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191126152516561.png)

Here, Jest's detailed output comes in handy as we can see that our function, in this case, is returning NaN rather than the expected value of 0 for when averaging an empty array. This is because our array is being divided by the length of our array (0), and, in JavaScript, dividing an empty array by 0 yields NaN (through our reducer, it actually ends up dividing an empty object by 0 which still yields NaN).

We can fix this by updating our `average` function as follows

```js
const average = (array) => {
  const reducer = (sum, addition) => {
    return sum + addition;
  };

  return array.length === 0 ? 0 : array.reduce(reducer, 0) / array.length;
};
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

One issue that we have is that this method of setting environment variables does not work on Windows. Most Windows command prompts won't accept environment variables being set like `NODE_ENV=production` as they require them to be set using `set` or `$env:node_env = 'production'`. We can fix this by installing `cross-env` which allows for the cross-platform setting of environment variables.

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
require("dotenv").config();

let PORT = process.env.PORT;
let MONGODB_URI = process.env.MONGODB_URI;

if (process.env.NODE_ENV === "test") {
  MONGODB_URI = process.env.TEST_MONGODB_URI;
}

module.exports = {
  MONGODB_URI,
  PORT,
};
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
const mongoose = require("mongoose");
const supertest = require("supertest");
const app = require("../app");

const api = supertest(app);

test("notes returned as json", async () => {
  await api
    .get("/api/notes")
    .expect(200)
    .expect("Content-Type", /application\/json/);
});

afterAll(() => {
  mongoose.connection.close();
});
```

Here, you'll notice quite a few new concepts. We'll go through these new concepts one at a time. First, we import our `app` module (as we need this to 'start' our application so that we can interact with it to perform our integration tests). We pass our `app` module into the `supertest()` function which transforms our `app` module into a **superagent** (basically transforms the module into a lightweight, client-side HTTP server that supports HTTP requests). We then assign this superagent to the `api` variable. Now, our tests can use `api` to make HTTP requests to the backend.

Throughout this code, you'll see various references to the `async`/`await` keywords. This refers to another way to write asynchronous code (essentially syntactic sugar around promises). We will discuss this syntax later on.

You can see how we make these requests and compare the results in the following block of code:

```js
test("notes returned as json", async () => {
  await api
    .get("/api/notes")
    .expect(200)
    .expect("Content-Type", /application\/json/);
});
```

Here, wrapped in a test function, we have a function that performs a HTTP GET request to our backend (/api/notes) through calling the `get()` method upon our `api` variable with the parameter being the endpoint that we want to make our request to.

Then, to 'test' our response, we make the expectation that the response code should be `200` through the use of the `expect()` method. We also do the same to make sure that our `Content-Type` header is set to `application/json` (notice how we have to escape the second forward slash as it must be included).

Finally, after all of our tests have run, we want to close our database connection so that our application can terminate and so that we don't leave any unneeded connections open to our database.

We do this by utilizing the `afterAll()` method to which a callback is provided as a parameter to which it is executed after all of the tests have ran (hence the name).

```js
afterAll(() => {
  mongoose.connection.close();
});
```

Here, were we not to initialize the `testEnvironment` property with the value of `node` in the `jest` object of **package.json** as we did earlier, we would have encountered the following error:

![image-20191127012415162](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191127012415162.png)

This line essentially just tells Jest to use a node-like environment for testing rather than the default `jsdom` environment that simulates a browser-like environment. This could lead to some unexpected behaviors with the execution of our code were it left to its defaults.

Also, take note with how we passed in our `app` module rather than make use of **index.js** somehow. This is because we don't need to manually start our Express application or define ports as `supertest` will bind our server to an ephemeral port (ports allocated automatically by software).

Let's increase the complexity of our tests by adding two more tests to **notes_api.test.js**:

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require("mongoose");
const supertest = require("supertest");
const app = require("../app");

const api = supertest(app);

test("notes returned as json", async () => {
  await api
    .get("/api/notes")
    .expect(200)
    .expect("Content-Type", /application\/json/);
});

test("there are three notes", async () => {
  const response = await api.get("/api/notes");

  expect(response.body.length).toBe(3);
});

test("the first note is about it being the first note", async () => {
  const response = await api.get("/api/notes");

  expect(response.body[0].content).toBe("This is the first note!");
});

afterAll(() => {
  mongoose.connection.close();
});
```

You'll notice that our tests not depend on the state of our database. This is not good, because the 'state' of our database persists across separate test runs. This means that the results of one test run could differ from the other without changing any of the tests.

#### Initializing Test Databases

In order for our tests to be consistent and not dependent on the state of our database, let's reset and initialize our database prior to each test run.

First, let's initialize our database before the tests run by making use of the `beforeEach()` function. Similarly to how the `afterAll()` allowed us to execute a callback function after all of the tests had ran, the `beforeEach()` function allows us to execute a function prior to running any test.

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require("mongoose");
const supertest = require("supertest");
const app = require("../app");
const api = supertest(app);
const Note = require("../models/note");

const initializationNotes = [
  {
    content: "This is the first note!",
    date: new Date(),
    flagged: true,
  },
  {
    content: "This is a note that happens to be the second note.",
    date: new Date(),
    flagged: false,
  },
  {
    content: "This is the third note.",
    date: new Date(),
    flagged: true,
  },
];

beforeEach(async () => {
  await Note.deleteMany({});

  let noteObject = new Note(initializationNotes[0]);
  await noteObject.save();

  noteObject = new Note(initializationNotes[1]);
  await noteObject.save();

  noteObject = new Note(initializationNotes[2]);
  await noteObject.save();
});

// ...
```

Here, we created an `initialNotes` array which contains the three notes. Then, in the `beforeEach()` callback function, we wipe the database using the `deleteMany()` method of the `Note` model. The `deleteMany()` method deletes all of the documents that match the conditions passed into it, to which we had passed an empty object, deleting all of the documents in the collection.

We then create two new note objects by using the `Note` model constructor function and passing in the respective note to that function. We then preform `save()` upon the `noteObject` variable which holds the respective note to be saved that was just created by our `Note` model. This note is then saved by calling hte `save()` method upon the document.

After this, our database should be set up to perform our tests. With this, let's make a few updates to our last two tests so that they make use of the information now granted to us about our database due to its state being set up predictably.

**/Backend/tests/notes_api.test.js**

```js
//...

test("there are three notes", async () => {
  const response = await api.get("/api/notes");

  expect(response.body.length).toBe(initializationNotes.length);
});

test("the first note is about it being the first note", async () => {
  const response = await api.get("/api/notes");

  const allContents = response.body.map((n) => n.content);

  expect(allContents).toContain("This is the first note!");
});

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
  if (process.env.NODE_ENV !== "test") {
    console.log(...params);
  }
};

const error = (...params) => {
  console.error(...params);
};

module.exports = {
  info,
  error,
};
```

Here, in our `logger` module, we export two functions, the `info` function which contains a check to prevent these log statements from printing if our application is in test mode, and the `error` function which will `console.error` whatever parameter(s) is passed to it and will execute regardless of the environment mode.

Let's also build a `logRequest` middleware that takes advantage of these new functions.

```js
const logger = require("./logger");

const logRequest = (req, res, next) => {
  logger.info("Method:  ", req.method);
  logger.info("Path:    ", req.path);
  logger.info("Body:    ", req.body);
  logger.info("--------");
  next();
};

const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: "Endpoint does not exist (Unknown Endpoint)" });
};

const errorHandler = (err, req, res, next) => {
  logger.error(err.message);

  if (err.name === "CastError" && err.kind === "ObjectId") {
    return res.status(400).send({ error: "Incorrect format for id parameter" });
  } else if (err.name === "ValidationError") {
    return res.status(400).json({ error: err.message });
  }

  next(err);
};

module.exports = {
  logRequest,
  unknownEndpoint,
  errorHandler,
};
```

Here, you'll notice how we build the `logRequest` function that logs various bits of information about the request using `logger.info()`. We also updated our `errorHandler` middleware to use `logger.error()` function.

We also need to make the appropriate changes to **app.js** involving our new **logger.js** module, as well as adding `logRequest` among our middleware.

**/Backend/app.js**

```js
//...

const logger = require("./utils/logger");

logger.info("Connecting to:", config.MONGODB_URI); // Now uses logger

mongoose
  .connect(config.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
  })
  .then(() => {
    logger.info("Connected to MongoDB"); // Now uses logger
  })
  .catch((error) => {
    logger.error("Error connecting to MongoDB:", error.message); // Now uses logger
  });

// Middleware
app.use(cors());
app.use(express.static("build"));
app.use(bodyParser.json());
app.use(middleware.logRequest); // The addition of our `logRequest` middleware

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
Note.find({})
  .then((notes) => {
    return notes[0].remove();
  })
  .then((res) => {
    console.log("successfully removed the first note");
  });
```

Here, we use the `find()` method upon the `Note` model in order to get all of the notes, then we access the result of this promise through registering a callback to the `then()` method which we then remove the first note in. This is an example of a promise chain (which we have discussed in-depth earlier). Another approach to making multiple asynchronous function calls in sequence would be to register callbacks within callbacks, but this is undesirable as it quickly becomes hard to read and leads to what is called _callback hell_.

As syntactic sugar around these promises, **generator functions** were introduced in ES6 as well that provided a way, similar to promises, to write asynchronous code that resembled synchronous code. However, generator functions were never widely used with its awkward syntax. Although, it is interesting to know that generators and promises formed the foundation for the async/await expressions.

Now, let's dive in to doing the above using the asynx/await keywords. We can use the `await` operator to _wait_ for a promise (like the one `Note.find({})` returns).

```js
const notes = await Note.find({});
```

Now, the execution of our code pauses at this line until the promise (the expression following the `await` keyword) is fulfilled. Once this promise is fulfilled, the execution of our code will continue. Here, you can see how our code looks like synchronous code.

Now, let's add the removal of the first note.

```js
const notes = await Note.find({});
const res = await notes[0].remove();
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
notesRouter.get("/", (req, res) => {
  Note.find({}).then((notes) => {
    res.json(notes.map((note) => note.toJSON()));
  });
});
```

This is what the controller looks like using `async`/`await`:

**/Backend/controllers/notes.js**

```js
notesRouter.get("/", (req, res) => {
  const notes = await Note.find({});
  res.json(notes.map((note) => note.toJSON()));
});
```

Now, with refactoring our code, we must test to see if everything works as encountering **regression** (the breaking of existing functionality) is always a risk when refactoring.

In this case, we can test our route just by visiting the respective path or by using testing tools, but this can become tedious when dealing with many controllers and routes as we're about to. As an alternative, we can use our testing suite, Jest, in order to automate this process for us.

Let's write a test for each route of our API.

First, let's write a test that tests our HTTP POST controller that creates a new note.

**/Backend/tests/notes_api.test.js**

```js
//...

test("a valid note can be added ", async () => {
  const newNote = {
    content: "This is a new note!",
    flagged: true,
  };

  await api
    .post("/api/notes")
    .send(newNote)
    .expect(201)
    .expect("Content-Type", /application\/json/);

  const response = await api.get("/api/notes");

  const contents = response.body.map((r) => r.content);

  expect(response.body.length).toBe(initializationNotes.length + 1);
  expect(contents).toContain("This is a new note!");
});

//...
```

Here, you'll notice how we send the note and then await for the response. Upon receiving the response, we check to see if the length of the notes array is one higher than the initial array length that we set our database state to earlier. We also check if the collection of notes contains the text of our note that we had added. Additionally, we just check if the status code of the response and content-type header are correct.

Upon running this test, we encounter a few issues. Our test does not pass, so we'll have to make some changes to our controller as we had not set the correct status all along. We never set our status code before sending our response using the `json()` method, so we'll just call the `status()` method to do so beforehand. Our updated controller looks like this:

```js
notesRouter.post("/", (req, res, next) => {
  const body = req.body;

  if (!body.content) {
    return res.status(400).json({
      error: "No content specified in note body",
    });
  }

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  newNote
    .save()
    .then((savedNote) => {
      res.status(201).json(savedNote.toJSON());
    })
    .catch((error) => next(error));
});
```

Now, our test passes! Let's also test our edge case by writing a test that checks if a note with no content will not be saved into the database.

**/Backend/tests/notes_api.test.js**

```js
//...

test("note without content is not saved", async () => {
  const newNote = {
    flagged: true,
  };

  await api.post("/api/notes").send(newNote).expect(400);

  const response = await api.get("/api/notes");

  expect(response.body.length).toBe(initializationNotes.length);
});

//...
```

Here, we just expect that the status code returned will be `400` and that the length of `response` (which contains the results of a HTTP GET request to our notes collection) is the same as the length of the array that we had initialized our database with.

Note, when we run our tests serially, the previous tests do not affect the outcome of subsequent tests as we reset the state of our database before each and every test. Otherwise, the previous test where we added a note to our database would affect our test results here as the length of `response` would be one higher than it is supposed to be.

Now, you'll notice that we're starting to repeat some code between our tests. This is usually a good indicator that there is something better to do with our code. Let's abstract around our code and create a few helper functions in their own module that we can use in our tests.

**/Backend/tests/test_helper.js**

```js
const Note = require("../models/note");

const initializationNotes = [
  {
    content: "This is the first note!",
    date: new Date(),
    flagged: true,
  },
  {
    content: "This is a note that happens to be the second note.",
    date: new Date(),
    flagged: false,
  },
  {
    content: "This is the third note.",
    date: new Date(),
    flagged: true,
  },
];

const deletedNoteId = async () => {
  const note = new Note({
    content: "This note was created as part of a test and will be deleted",
    date: new Date(),
  });

  await note.save();
  await note.remove();

  return note._id.toString();
};

const dbNotes = async () => {
  const notes = await Note.find({});
  return notes.map((note) => note.toJSON());
};

module.exports = {
  initializationNotes,
  deletedNoteId,
  dbNotes,
};
```

Here, we simply created two helper functions and moved the definition of our `initializationNotes` to this file. In `deletedNoteId`, we create a new note, save the note to our database, and then delete the not. After this, we fetch the `_id` property of the note object that we created using the `Note` model constructor function and return that, representing the ID of a database object that has since been deleted. In `dbNotes` we simply fetch all of the notes and create an array of notes mapped through `toJSON()`.

Now, we can use these helper functions in our tests as follows:

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require("mongoose");
const supertest = require("supertest");
const app = require("../app");
const api = supertest(app);

const Note = require("../models/note");

const helper = require("./test_helper");

beforeEach(async () => {
  await Note.deleteMany({});

  let noteObject = new Note(helper.initializationNotes[0]);
  await noteObject.save();

  noteObject = new Note(helper.initializationNotes[1]);
  await noteObject.save();

  noteObject = new Note(helper.initializationNotes[2]);
  await noteObject.save();
});

test("notes returned as json", async () => {
  await api
    .get("/api/notes")
    .expect(200)
    .expect("Content-Type", /application\/json/);
});

test("there are three notes", async () => {
  const response = await api.get("/api/notes");

  expect(response.body.length).toBe(helper.initializationNotes.length);
});

test("the first note is about it being the first note", async () => {
  const response = await api.get("/api/notes");

  const allContents = response.body.map((n) => n.content);

  expect(allContents).toContain("This is the first note!");
});

test("a valid note can be added ", async () => {
  const newNote = {
    content: "This is a new note!",
    flagged: true,
  };

  await api
    .post("/api/notes")
    .send(newNote)
    .expect(201)
    .expect("Content-Type", /application\/json/);

  const allNotes = await helper.dbNotes();

  const contents = allNotes.map((r) => r.content);

  expect(allNotes.length).toBe(helper.initializationNotes.length + 1);
  expect(contents).toContain("This is a new note!");
});

test("note without content is not saved", async () => {
  const newNote = {
    flagged: true,
  };

  await api.post("/api/notes").send(newNote).expect(400);

  const allNotes = await helper.dbNotes();

  expect(allNotes.length).toBe(helper.initializationNotes.length);
});

afterAll(() => {
  mongoose.connection.close();
});
```

We can delete our `initializationNotes` constant since we can reference it from the helper file. In the above block, we changed all references to `initializationNotes` to `helper.initializationNotes`. Additionally, in our 'a valid note can be added' test, we use our `dbNotes()` helper function to get all the notes rather than making a HTTP GET request for all the notes and using the response. We also do the same for the ''note without content is not saved" test.

Upon running our test script, we'll find that all of our tests pass. With this, we can now begin to refactor our code to use `async`/`await`.

We can change our HTTP POST handler (that handles the adding of notes) to use `async`/`await` as follows:

**/Backend/controllers/notes.js**

```js
//...

notesRouter.post("/", async (req, res, next) => {
  const body = req.body;

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  const savedNote = await newNote.save();
  res.status(201).json(savedNote.toJSON());
});

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

notesRouter.post("/", async (req, res, next) => {
  const body = req.body;

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
  });

  try {
    const savedNote = await newNote.save();
    res.status(201).json(savedNote.toJSON());
  } catch (exception) {
    next(exception);
  }
});

//...
```

This is read as 'trying' the code in the first block, and if there is an exception in the first block, the second block will 'catch' the exception. Here, in the catch block, we simply call the `next` function to pass the exception down to our error handling middleware.

Now, moving on to the functionality regarding fetching a specific note, we can write the test as follows:

**/Backend/tests/notes_api.test.js**

```js
//...

test("a specific note can be fetched", async () => {
  const allNotes = await helper.dbNotes();

  const noteToFetch = allNotes[0];

  // Convert date value to string as the response will contain a string
  noteToFetch.date = noteToFetch.date.toISOString();

  const fetchedNote = await api
    .get(`/api/notes/${noteToFetch.id}`)
    .expect(200)
    .expect("Content-Type", /application\/json/);

  expect(fetchedNote.body).toEqual(noteToFetch);
});

//...
```

Additionally, we can write the test surrounding deleting a specific note as follows:

**/Backend/tests/notes_api.test.js**

```js
//...

test("a specific note can be deleted", async () => {
  const unalteredNotes = await helper.dbNotes();
  const noteToDelete = unalteredNotes[0];

  await api.delete(`/api/notes/${noteToDelete.id}`).expect(204);

  const alteredNotes = await helper.dbNotes();

  expect(alteredNotes.length).toBe(helper.initializationNotes.length - 1);

  const contents = alteredNotes.map((r) => r.content);

  expect(contents).not.toContain(noteToDelete.content);
});

//...
```

Here, these tests simply perform the operation and a comparison to the original state of the database (whether that be the database state being stored in a variable via a call to `dbNotes()` or via a direct comparison to the `initializationNotes` array).

Now, we can go ahead and refactor the routes involving deleting and fetching a specific note as follows:

```js
//...

notesRouter.delete("/:id", async (req, res, next) => {
  try {
    await Note.findByIdAndDelete(req.params.id);
    res.status(204).end();
  } catch (exception) {
    next(exception);
  }
});

notesRouter.get("/:id", async (req, res, next) => {
  try {
    const note = await Note.findById(req.params.id);

    if (note) {
      res.json(note.toJSON());
    } else {
      res.status(404).end();
    }
  } catch (exception) {
    next(exception);
  }
});

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
  await Note.deleteMany({});
  console.log("Deleted all notes!");

  helper.initializationNotes.forEach(async (note) => {
    let newNote = new Note(note);
    await newNote.save();
    console.log("Saved the new note!");
  });
  console.log("Done with all of the operations!");
});

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
  await Note.deleteMany({});

  const allNotes = helper.initializationNotes.map((note) => new Note(note));
  const promiseArray = allNotes.map((note) => note.save());
  await Promise.all(promiseArray);
});

//...
```

Here, we map over our `initializationNotes` to create an array of Mongoose objects (created using the `Note` constructor function). Then, we take this array of Mongoose objects and map over it, within the map function, we call `save()` upon the Mongoose object which returns a promise to be fulfilled. This gives us an array of promises that are to be fulfilled. Note that these promises are in the process of being fulfilled/rejected as we have already executed the `save()` operation, but we can ensure the fulfillment/rejection of all of these promises before ending the function by passing this array to `Promise.all()` with the `await` operator in front of it. The `Promise.all()` method essentially creates a "master" promise that we can `await` since it will only fulfill when all of the promises have fulfilled and is located in `beforeEach()` so the function will actually wait for its execution to be completed.

> To clarify, promises cannot be "executed", promises themselves are wrappers around a created task and represent only the results of that task. Thus, when we call `note.save()` above, the task of saving the note has started and we are returned with a promise that represents the result of that task which can be passed in to `Promise.all()`. Execution simply refers to the underlying task being performed, which must already have been performed for the promise to be created. In short, `Promise.all()` is not executing anything, but rather just ensuring that all of these promises are fulfilled (or one is rejected) before continuing.

Now, as these promises are executed within the function provided to `allNotes.map()`, all of these promises are executed in parallel being that we can't execute them sequentially with no guarantees as to the order of the promises being finished. As we can iterate through our note objects and create our promises on-demand, we can ensure a specific execution order by creating a promise, waiting until its fulfillment and then proceeding as follows:

```js
beforeEach(async () => {
  await Note.deleteMany({});

  for (let note of helper.initializationNotes) {
    let newNote = new Note(note);
    await newNote.save();
  }
});
```

Here, for each note in our `initializationNotes` we just create a new note, save it, and await that promise before continuing to the next iteration.

Finally, with this handled, we can go over tests for invalid IDs and non-existent as follows:

```js
test("fails with status code 404 if ID does not exist", async () => {
  const validNonexistentId = await helper.deletedNoteId();

  await api.get(`/api/notes/${validNonexistentId}`).expect(404);
});

test("fails with statuscode 400 if id is invalid", async () => {
  const invalidId = "thisisaninvalidid";

  await api.get(`/api/notes/${invalidId}`).expect(400);
});
```

With this, our test coverage covers most major functionality of our application and some edge cases. The following is the entire test file with the culmination of all of our tests that we've written:

```js
const mongoose = require("mongoose");
const supertest = require("supertest");
const app = require("../app");
const api = supertest(app);

const Note = require("../models/note");

const helper = require("./test_helper");

beforeEach(async () => {
  await Note.deleteMany({});

  const allNotes = helper.initializationNotes.map((note) => new Note(note));
  const promiseArray = allNotes.map((note) => note.save());
  await Promise.all(promiseArray);
});

test("notes returned as json", async () => {
  await api
    .get("/api/notes")
    .expect(200)
    .expect("Content-Type", /application\/json/);
});

test("there are three notes", async () => {
  const response = await api.get("/api/notes");

  expect(response.body.length).toBe(helper.initializationNotes.length);
});

test("the first note is about it being the first note", async () => {
  const response = await api.get("/api/notes");

  const allContents = response.body.map((n) => n.content);

  expect(allContents).toContain("This is the first note!");
});

test("a valid note can be added ", async () => {
  const newNote = {
    content: "This is a new note!",
    flagged: true,
  };

  await api
    .post("/api/notes")
    .send(newNote)
    .expect(201)
    .expect("Content-Type", /application\/json/);

  const allNotes = await helper.dbNotes();

  const contents = allNotes.map((r) => r.content);

  expect(allNotes.length).toBe(helper.initializationNotes.length + 1);
  expect(contents).toContain("This is a new note!");
});

test("note without content is not saved", async () => {
  const newNote = {
    flagged: true,
  };

  await api.post("/api/notes").send(newNote).expect(400);

  const allNotes = await helper.dbNotes();

  expect(allNotes.length).toBe(helper.initializationNotes.length);
});

test("a specific note can be fetched", async () => {
  const allNotes = await helper.dbNotes();

  const noteToFetch = allNotes[0];

  // Convert date value to string as the response will contain a string
  noteToFetch.date = noteToFetch.date.toISOString();

  const fetchedNote = await api
    .get(`/api/notes/${noteToFetch.id}`)
    .expect(200)
    .expect("Content-Type", /application\/json/);

  expect(fetchedNote.body).toEqual(noteToFetch);
});

test("a specific note can be deleted", async () => {
  const unalteredNotes = await helper.dbNotes();
  const noteToDelete = unalteredNotes[0];

  await api.delete(`/api/notes/${noteToDelete.id}`).expect(204);

  const alteredNotes = await helper.dbNotes();

  expect(alteredNotes.length).toBe(helper.initializationNotes.length - 1);

  const contents = alteredNotes.map((r) => r.content);

  expect(contents).not.toContain(noteToDelete.content);
});

test("fails with status code 404 if ID does not exist", async () => {
  const validNonexistentId = await helper.deletedNoteId();

  await api.get(`/api/notes/${validNonexistentId}`).expect(404);
});

test("fails with statuscode 400 if id is invalid", async () => {
  const invalidId = "thisisaninvalidid";

  await api.get(`/api/notes/${invalidId}`).expect(400);
});

afterAll(() => {
  mongoose.connection.close();
});
```

We could improve these tests, provide additional code coverage, and improve the readability of these tests (adding describe blocks as an example), but this is a good starting point and should demonstrate how to write integration/unit tests.

Web Development 4

## Authentication

Almost every application on the Internet requires some form of user authentication (whether that be through a Google account, phone number, email, etc.) in order to restrict access to certain resources. With this, let's begin to add a way to authenticate users in our application.

To authenticate users, we first need a way to keep track of them and store user information in a persistent manner. Our database fits these requirements perfectly, so we will store our users in the database.

As an overview of what we'll be doing: our database will hold information pertaining to the user's credentials and the notes that they "own". Users will only be able to modify the notes that they own (similar to how users can only manage their own posts on social media platforms).

Thinking about the relationship between our user database objects and the notes, we would have a one-to-many relationship where one user has many notes. Now, you might have worked with relational databases in the past where implementing the above relationship is quite straightforward. Due to the nature of NoSQL databases (recall that the NO stands for "not only"), we are not restricted to a subset of implementations but rather are given many different ways to model this situation. One approach would be to have a collection of _notes_ and a collection of _users_. There are quite a few features found in relational databases that document databases (like MongoDB) do not support to which ODMs such as Mongoose try to emulate. This statement is becoming less true over time, however, as MongoDB and other document databases add more features.

### Referencing Objects in Other Collections

In document databases, like we did earlier, we refer to documents via object IDs (similar to referring to a foreign key in a relational database that provides a link between two objects in two different tables).

> Please note that the IDs in the following examples are not representative of anything, but rather just illustrate the relationships between objects.

**Users** collection:

```js
[
  {
    username: "josuff",
    _id: 123456,
  },
  {
    username: "markus",
    _id: 111111,
  },
];
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
    username: "josuff",
    _id: 123456,
    notes: [225533],
  },
  {
    username: "markus",
    _id: 111111,
    notes: [223266, 123456],
  },
];
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
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  username: String,
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Note",
    },
  ],
});

userSchema.set("toJSON", {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
    // For security purposes, the passwordHash of the user should not be sent with the user object (even though it is not the plaintext password, it can still be decrypted)
    delete returnedObject.passwordHash;
  },
});

const User = mongoose.model("User", userSchema);

module.exports = User;
```

Here, you'll notice that we specify the schema of the user object as having a `username` of type String, a `name` of type String, a `passwordHash` (hashed version of their password, we'll talk about this more later) of type String, and a `notes` array of a complex type. This `notes` array is specified to contain objects of the type `ObjectId` which simply refers to it containing `ObjectId`s of other database objects; additionally, we provide the model that these `ObjectId`s will reference to as the value of the `ref` parameter.

Now, motivated by the desire to have the ability to lookup the user that owns a note from a given note, we'll add a reference to the `User` model within our `Note` model's schema:

**/Backend/models/note.js**

```js
const mongoose = require("mongoose");

const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    minlength: 1,
    required: true,
  },
  date: {
    type: Date,
    required: true,
  },
  flagged: Boolean,
  user: {
    // Reference to the user that "owns" the note
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
  },
});

noteSchema.set("toJSON", {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
  },
});

module.exports = mongoose.model("Note", noteSchema);
```

In this approach, the user model can reference the notes that "it" owns, and the notes can reference the user who "owns" them. This is in contrast to conventions of most relational databases, but is possible and acceptable with document databases.

## User Controller

Now, let's actually take advantage of our `User` model by creating a controller that will actually create our users.

Recall the mention of `passwordHash` from earlier and how it refers to a hashed version of a user's password. The value of this property will simply be assigned the output of a _one-way hash function_ (essentially means that it will be close to impossible to derive the original text from this hashed string). We store this value as it is not proper with regard to security practices to store plaintext passwords. The topic of encryption and hashing as a subset of cryptography are very interesting topics and deserve a read if you are interested in the how and why with regard to how computers take advantage of cryptography and why it is used.

As with most things in the JavaScript ecosystem, we do not have to write our own functions (especially so in this case as we do not have the acumen nor the resources to maintain all of these functions) to generate our password hashes. We can utilize the `bcrypt` package for doing exactly this:

```bash
npm install bcrypt
```

As we are _creating_ a new resource when we create a new user, we'll be making and handling POST requests when creating these users as per RESTful conventions. Let's create the file **users.js** within our **/controllers** directory.

**/Backend/controllers/users.js**

```js
const bcrypt = require("bcrypt");
const usersRouter = require("express").Router();
const User = require("../models/user");

usersRouter.post("/", async (req, res, next) => {
  try {
    const body = req.body;

    const saltRounds = 10;
    const passwordHash = await bcrypt.hash(body.password, saltRounds);

    const user = new User({
      username: body.username,
      name: body.name,
      passwordHash,
    });

    const savedUser = await user.save();

    res.json(savedUser);
  } catch (exception) {
    next(exception);
  }
});

module.exports = usersRouter;
```

Now, let's just make use of our new controller by binding the exported router to our application through a call to `app.use()`.
**/Backend/app.js**

```js
//...

const usersRouter = require("./controllers/users");

//...

app.use("/api/users", usersRouter);

//...
```

In the above controller, we never store the plaintext password that was sent over the request. Instead, we hash it and store the hash.

You'll notice our call to `bcrypt.hash()` and its parameter `saltRounds`. We specified the value for `saltRounds` earlier in the file as 10. We won't go into too much detail regarding the technical bits of hashing as that would far out-scope the content of this book (while interesting and potentially useful to know, knowledge of the workings of hashing is not required).

Basically, you might have heard of hash functions like MD5, SHA1, SHA2, etc., these are all _general purpose hash functions_ that place an emphasis on hashing large amounts of data in a fast manner. This fast manner makes them almost useless for passwords as a modern server (especially one that is GPU accelerated) can calculate the hash extremely fast, making it possible for someone to try every single password combination (given certain constraints). `bcrypt` (based on the Blowfish block cipher cryptomatic algorithm) solves this by introducing a work factor to which we can adjust. `bcrypt` does not simply just hash passwords (as the exact same passwords would have the exact same hash going through the algorithm with the same work factor), but it instead passes in a _salt_ (a randomized string) along with the password to hash, so that every single hash is truly unique, giving us an extra layer of security and preventing the use of rainbow tables (described below).

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

Here, you can see that we go from 71 milliseconds for 10 salt rounds to over 67 _seconds_ for 20 salt rounds. This increase in time resembles a exponential relationship which is a good match to counteract the exponential increase of computing power that we experience (for the most part, as the increase in computing power does not rigidly follow Moore's Law as of about a decade).

Finally, it is important to realize that, even with relatively high work values, it is still feasible that a single password could be cracked with enough computing powers. Thus, in the event of a breach where password hashes are released, you should take care to have users reset their passwords (thus producing another hash).

#### Testing the User Controller

Let's write a few initial tests for our new endpoints. We'll add these tests to our existing test file for now, but as you continue to add tests, it may be wise to components out these tests into multiple files:

**/Backend/tests/notes_api.test.js**

```js
const mongoose = require("mongoose");
const supertest = require("supertest");
const app = require("../app");
const api = supertest(app);

const Note = require("../models/note");
const User = require("../models/user");

const helper = require("./test_helper");

describe("when there are a few initial notes saved to the db", () => {
  beforeEach(async () => {
    await Note.deleteMany({});

    const allNotes = helper.initializationNotes.map((note) => new Note(note));
    const promiseArray = allNotes.map((note) => note.save());
    await Promise.all(promiseArray);
  });

  test("notes returned as json", async () => {
    await api
      .get("/api/notes")
      .expect(200)
      .expect("Content-Type", /application\/json/);
  });

  test("there are three notes", async () => {
    const response = await api.get("/api/notes");

    expect(response.body.length).toBe(helper.initializationNotes.length);
  });

  test("the first note is about it being the first note", async () => {
    const response = await api.get("/api/notes");

    const allContents = response.body.map((n) => n.content);

    expect(allContents).toContain("This is the first note!");
  });

  test("a valid note can be added ", async () => {
    const newNote = {
      content: "This is a new note!",
      flagged: true,
    };

    await api
      .post("/api/notes")
      .send(newNote)
      .expect(201)
      .expect("Content-Type", /application\/json/);

    const allNotes = await helper.dbNotes();

    const contents = allNotes.map((r) => r.content);

    expect(allNotes.length).toBe(helper.initializationNotes.length + 1);
    expect(contents).toContain("This is a new note!");
  });

  test("note without content is not saved", async () => {
    const newNote = {
      flagged: true,
    };

    await api.post("/api/notes").send(newNote).expect(400);

    const allNotes = await helper.dbNotes();

    expect(allNotes.length).toBe(helper.initializationNotes.length);
  });

  test("a specific note can be fetched", async () => {
    const allNotes = await helper.dbNotes();

    const noteToFetch = allNotes[0];

    // Convert date value to string as the response will contain a string
    noteToFetch.date = noteToFetch.date.toISOString();

    const fetchedNote = await api
      .get(`/api/notes/${noteToFetch.id}`)
      .expect(200)
      .expect("Content-Type", /application\/json/);

    expect(fetchedNote.body).toEqual(noteToFetch);
  });

  test("a specific note can be deleted", async () => {
    const unalteredNotes = await helper.dbNotes();
    const noteToDelete = unalteredNotes[0];

    await api.delete(`/api/notes/${noteToDelete.id}`).expect(204);

    const alteredNotes = await helper.dbNotes();

    expect(alteredNotes.length).toBe(helper.initializationNotes.length - 1);

    const contents = alteredNotes.map((r) => r.content);

    expect(contents).not.toContain(noteToDelete.content);
  });

  test("fails with status code 404 if ID does not exist", async () => {
    const validNonexistentId = await helper.deletedNoteId();

    await api.get(`/api/notes/${validNonexistentId}`).expect(404);
  });

  test("fails with statuscode 400 if id is invalid", async () => {
    const invalidId = "thisisaninvalidid";

    await api.get(`/api/notes/${invalidId}`).expect(400);
  });
});

describe("when there is one initial user in the db", () => {
  beforeEach(async () => {
    await User.deleteMany({});
    const user = new User({ username: "root", password: "password" });
    await user.save();
  });

  test("creation of a new user succeeds", async () => {
    const initialUsersState = await helper.usersInDb();

    const newUser = {
      username: "josuff",
      name: "Joseph Semrai",
      password: "thisisapassword123",
    };

    await api
      .post("/api/users")
      .send(newUser)
      .expect(200)
      .expect("Content-Type", /application\/json/);

    const finalUsersState = await helper.dbUsers();
    expect(finalUsersState.length).toBe(initialUsersState.length + 1);

    const usernames = finalUsersState.map((u) => u.username);
    expect(usernames).toContain(newUser.username);
  });
});

afterAll(() => {
  mongoose.connection.close();
});
```

The above code represents our entire test file. It is quite a bit of code, so let's step through it and provide an overview of what's going on and what has changed.

Firstly, we wrapped our original tests pertaining to our notes routes in a `describe` block so that we could register a different `beforeEach()` function. This allows us to set up our initial database state for our notes test and users test where we are expecting different initial states. Then, we specified our 'creation of a new user succeeds' test by comparing our user collection's state after we have created a new user to the original state when the test started. We do this by checking to see if the collection is one greater in length than the starting state and if the collection contains a username matching the user that we have just created.

You'll also notice that we had called a `dbUsers()` helper function. We elected to make this helper function as we would likely need to fetch all of the users in multiple cases and tests. The function is as follows:

**/Backend/tests/test_helper.js**

```js
const Note = require("../models/note");
const User = require("../models/user");

const initializationNotes = [
  {
    content: "This is the first note!",
    date: new Date(),
    flagged: true,
  },
  {
    content: "This is a note that happens to be the second note.",
    date: new Date(),
    flagged: false,
  },
  {
    content: "This is the third note.",
    date: new Date(),
    flagged: true,
  },
];

const deletedNoteId = async () => {
  const note = new Note({
    content: "This note was created as part of a test and will be deleted",
    date: new Date(),
  });

  await note.save();
  await note.remove();

  return note._id.toString();
};

const dbNotes = async () => {
  const notes = await Note.find({});
  return notes.map((note) => note.toJSON());
};

const dbUsers = async () => {
  const users = await User.find({});
  return users.map((u) => u.toJSON());
};

module.exports = {
  initializationNotes,
  deletedNoteId,
  dbNotes,
  dbUsers,
};
```

Now, what would happen if we wrote a test testing our ability to create a user with an existing username?

**/Backend/tests/notes_api.test.js**

```js
//...

describe("when there is one initial user in the db", () => {
  //...

  test("user creation fails with statuscode and message if username taken", async () => {
    const initialUsersState = await helper.dbUsers();

    const newUser = {
      username: "root",
      name: "some other person",
      password: "anotherexamplepassword",
    };

    const result = await api
      .post("/api/users")
      .send(newUser)
      .expect(400)
      .expect("Content-Type", /application\/json/);

    expect(result.body.error).toContain("`username` taken");

    const finalUsersState = await helper.dbUsers();
    expect(finalUsersState.length).toBe(initialUsersState.length);
  });
});
```

Here, our test expects that we'll receive an error message and a `400` status code response if we create a new user with a taken username. This test will not pass as we have not written the actual logic behind checking if a username is taken in our controller.

This practice of writing tests which defines the requirements of our functionality before writing the code for the functionality is described as _test-driven development_. In test-driven development, we turn our requirements into specific test cases and then improve our code so that our tests pass.

### Unique Validation

Let's improve our code so that the previous test will pass. To ensure that our username is unique, we'll take advantage of Mongoose validators. Recall that Mongoose validators comes with a very specific set of validators with uniqueness not being one; however, Mongoose validators are extensible and there is a package that provides exactly what we're looking for.

We can install this package by running:

```bash
npm install mongoose-unique-validator
```

We can now add this validator to our `user` model.

**/Backend/models/user.js**

```js
const mongoose = require("mongoose");
const uniqueValidator = require("mongoose-unique-validator");

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    unique: true,
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Note",
    },
  ],
});

userSchema.set("toJSON", {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString();
    delete returnedObject._id;
    delete returnedObject.__v;
    delete returnedObject.passwordHash;
  },
});

userSchema.plugin(uniqueValidator);

const User = mongoose.model("User", userSchema);

module.exports = User;
```

Here, you'll see how we set the `username` property with an object that sets the type of the property to `String` and makes use of the `unique` validator that is given to us by binding the `uniqueValidator` module as a plugin to our schema.

Finally, we can write a GET route for our users controller so that we can see what we've done so far:

**/Backend/controllers/users.js**

```js
//...

usersRouter.get("/", async (req, res, next) => {
  try {
    const users = User.find({});
    res.json(users.map((u) => u.toJSON()));
  } catch (exception) {
    next(exception);
  }
});

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

const User = require("../models/user");

//...

notesRouter.post("/", async (req, res, next) => {
  const body = req.body;

  const user = User.findById(body.userId);

  const newNote = new Note({
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
    user: user._id,
  });

  try {
    const savedNote = await newNote.save();
    user.notes = user.notes.concat(savedNote._id); // Adds the ID of the note we have just saved to the user's notes
    await user.save();
    res.status(201).json(savedNote.toJSON());
  } catch (exception) {
    next(exception);
  }
});

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

Now, one downside to this is that Mongoose accomplishes this join through multiple queries. In relational databases, join queries are _transactional_, ensuring that the state of the database cannot change while the query is executing. This means that we cannot guarantee that the state of the collections remains the same during our 'join' query, leading to some edge cases where the data that we receive may not match the true state of our database.

With this, let's actually perform the join using the `populate()` method:

**/Backend/controllers/users.js**

```js
//...

usersRouter.get("/", async (req, res) => {
  const users = await User.find({}).populate("notes");

  res.json(users.map((u) => u.toJSON()));
});

//...
```

Here, we chain the `populate()` method after 'finding' all of our users, thus performing the populate operation on every single user document before the data is stored in the `users` constant. We pass in the property where we want references to be replaced with the actual documents. What this means is that any reference to an object in a collection found in the specified property will be replaced by the actual document, so that all the IDs found in the `notes` properties of all of the users will be replaced by the actual documents to which they refer to.

Mongoose knows that we are referring to the `notes` collection as we had provided the `ref` option earlier in our `User` schema. Without this, the database would not know that these IDs that we want to populate actually refer to documents in the `notes` collection:

```js
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    unique: true,
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Note", // We provide information as to what collection this ObjectId will refer to here
    },
  ],
});
```

Now, upon visiting this endpoint in our browser, we can see the actual note documents from within our user object:

![image-20191211194006275](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211194006275.png)

Now, let's improve on this implementation. We do not need the `user` field in the note object(s) as we have the relationship visible to us as the notes are now embedded in the user document. Let's filter out this field by specifying the fields that we want to include in an object that is used as the second parameter:

**/Backend/controllers/users.js**

```js
//...

usersRouter.get("/", async (req, res) => {
  const users = await User.find({}).populate("notes", {
    content: 1,
    flagged: 1,
    date: 1,
  });

  res.json(users.map((u) => u.toJSON()));
});

//...
```

Here, you can see how we are no longer given the `user` field.

![image-20191211194356591](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211194356591.png)

Let's apply these steps to our root notes endpoint:

```js
//...

notesRouter.get("/", async (req, res) => {
  const notes = await Note.find({}).populate("user", { username: 1, name: 1 });

  res.json(notes.map((note) => note.toJSON()));
});

//...
```

Visiting this endpoint (**http://localhost:3001/api/notes**) in the browser, we receive the following response:

![image-20191211195040360](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191211195040360.png)

Some of the content (the content that was not filtered out) is now included under the `user` property of our note's object rather than just its ID.

## Token-Based Authentication

Now, you might be wondering, how do we know which user is actually using our app and submitting the notes? Previously, we had just been manually inserting in the desired ID, but we need to add a way for users to log in to our application and for our application to know that this given user is currently signed in from its requests.

With this, we will add _token-based authentication_ to our application. Tokens are one of the best ways to handle authentication for multiple users as it is easy to implement across various platforms, allows for stateless and scalable servers, extra security, and much more.

As the HTTP protocol is stateless, even after we authenticate a user, our backend would not know who the user is for every subsequent request unless we tell it so via the request. With this, we will expand upon this concept and create a token that will be passed to the client to which the client can use in every one of its requests.

The process of token-based authentication generally follows this sequence:

1. Server serves website to client.
2. User logs in with username and password on the client.
3. Client sends request to the server to authenticate.
4. The server takes the authentication request, and if the information is correct, it creates a signed token that is sent back to the client.
5. Client stores the token to be used with its requests.
6. **Subsequent requests to the server carries the token as a HTTP header** (the server will now respond to sensitive requests if this token is valid)

Here, nothing regarding a user's session state or records of the user's authentication is stored on our server which is a major benefit of token-based authentication. With what is called _server-based authentication_, the server would save user info in a session variable to which the client and server have to store information that is consistently compared for every request. This increases overhead with regard to having to create records of authentications, needing a designated area of session memory for users, having to account for CORS, and having to protect against CSRF (to a greater extent).

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
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");
const loginRouter = require("express").Router();

const User = require("../models/user");

loginRouter.post("/", async (req, res) => {
  const body = req.body;

  const user = await User.findOne({ username: body.username });
  const passwordCorrect =
    user === null // Checks if there is actually a user with the given username
      ? false
      : await bcrypt.compare(body.password, user.passwordHash);

  if (!user || !passwordCorrect) {
    return res.status(401).json({
      error: "Invalid username or password",
    });
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  };

  const token = jwt.sign(userForToken, process.env.SECRET);

  res.status(200).send({ token, username: user.username, name: user.name });
});

module.exports = loginRouter;
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

const loginRouter = require("./controllers/login");

//...

app.use("/api/login", loginRouter);

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

const jwt = require("jsonwebtoken");

//...

const getTokenFrom = (req) => {
  const authorization = req.get("authorization");
  if (authorization && authorization.toLowerCase().startsWith("bearer ")) {
    return authorization.substring(7);
  }
  return null;
};

//...

notesRouter.post("/", async (req, res, next) => {
  const body = req.body;

  const token = getTokenFrom(req);

  try {
    const decodedToken = jwt.verify(token, process.env.SECRET);
    if (!token || !decodedToken.id) {
      return res.status(401).json({ error: "missing or invalid token" });
    }

    const user = await User.findById(decodedToken.id);

    const newNote = new Note({
      content: body.content,
      flagged: body.flagged || false,
      date: new Date(),
      user: user._id,
    });

    const savedNote = await newNote.save();
    user.notes = user.notes.concat(savedNote._id); // Adds the ID of the note we have just saved to the user's notes
    await user.save();
    res.status(201).json(savedNote.toJSON());
  } catch (exception) {
    next(exception);
  }
});
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
  logger.error(err.message);

  if (err.name === "CastError" && err.kind === "ObjectId") {
    return res.status(400).send({ error: "Incorrect format for id parameter" });
  } else if (err.name === "ValidationError") {
    return res.status(400).json({ error: err.message });
  } else if (err.name === "JsonWebTokenError") {
    return res.status(400).json({ error: "Invalid token" });
  }

  logger.error(err);

  next(err);
};
```

### Practicality

While going over the process required to verify and use the object within our token, you might have wondered if we could abstract this process to a middleware so that we could make use of it in multiple controller. This could be done through the use of a package like `express-jwt` which does the above task of validating a `JsonWebToken` to which it attaches the resulting object to the `user` property of the `req` object, allowing us to authenticate HTTP requests without writing the code that we did above for every controller.

## Frontend User Management

Let's now implement signing in and adding tokens to our requests so that we can add notes from our React app (as we updated our backend to require authentication).

If we try launching our frontend, you'll find that we can no longer add notes (since we implemented and required authentication in the controller for posting notes), yet we can still update the flagged property as we had not required or implemented authentication there.

Let's add a form so that we can sign in:

**/Frontend/src/app.js**

```js
import React, { useState, useEffect } from "react";
import Note from "./components/Note";
import Notification from "./components/Notification";
import Separator from "./components/Separator";

import axios from "axios";

import noteService from "./services/notes";

const App = (props) => {
  const [notes, setNotes] = useState([]);
  const [noteField, setNoteField] = useState("Enter a new note here...");
  const [showAll, setShowAll] = useState(true);
  const [notificationMessage, setNotificationMessage] = useState();
  const [username, setUsername] = useState(""); // Add username piece to our state
  const [password, setPassword] = useState(""); // Add password piece to our state

  //...

  const handleLogin = (event) => {
    // Form event handler, prevents page submission upon submitting the form
    event.preventDefault();
    console.log("logging in:", username, password);
  };

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
  );
};

export default App;
```

Here, we first add two pieces of state to hold our username and password. Then, we add a form, similarly to how we added a form for submitting notes earlier.

You'll notice that our `input` elements within our form have handlers assigned to the `onChange` events. This is so that we can update our pieces of state, `username` and `password`, to reflect the changes that the user is attempting to make in the input box. Since these `input` elements have their `value` property set to a piece of state, they are now considered _controlled components_ where it takes its current value through props and depends on an external state. Thus, for the user to be able to type and make changes in the `input` element, we have to specify a handler to the `onChange` event so that we can modify the state as such:

```js
onChange={({ target }) => setPassword(target.value)}
```

This event handler simply gets the object that it is given and accesses the value property to call our state setter upon.

We also provided an event handler to the `onSubmit` event of our `form` element. Right now, the event handler just makes use of the `preventDefault()` method of the event in order to prevent the page from refreshing upon a submission (just like we did with the note submission form). The `handleLogin` handler also logs the `username` and `password` pieces of state upon being calld:

```js
const handleLogin = (event) => {
  // Form event handler, prevents page submission upon submitting the form
  event.preventDefault();
  console.log("logging in:", username, password);
};
```

Recall that we must make a HTTP POST request to **/api/login** in order to authenticate. We must also keep track of the token that is returned from a successful login, with this, as we may reuse this request multiple times, and write a considerable amount of code surrounding the request, we'll create a separate module for this request in **services**:

**/Frontend/services/login.js**

```js
import axios from "axios";
const baseUrl = "/api/login";

const login = async (credentials) => {
  const res = await axios.post(baseUrl, credentials);
  return res.data;
};

export default { login };
```

You may notice that we're able to send objects using `axios` by simply passing it as the second parameter.

Now, over in our `App` component, let's make use of this service in our `handleLogin()` function:

**/Frontend/src/App.js**

```jsx
//...

import loginService from "./services/login";

//...

const [user, setUser] = useState(null);

//...

const handleLogin = async (event) => {
  event.preventDefault();
  try {
    const user = await loginService.login({
      username,
      password,
    });

    setUser(user);
    setUsername("");
    setPassword("");
  } catch (exception) {
    console.log(exception);
    setNotificationMessage("Incorrect credentials: " + exception);
    setTimeout(() => {
      setNotificationMessage(null);
    }, 3000);
  }
};

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
import axios from "axios";
const baseUrl = "/api/notes";

let token = null;

const setToken = (userToken) => {
  token = `bearer ${userToken}`;
};

const getAll = () => {
  const req = axios.get(baseUrl);
  return req.then((response) => response.data);
};

const create = async (newObject) => {
  const config = {
    headers: { Authorization: token },
  };

  const res = await axios.post(baseUrl, newObject, config);
  return res.data;
};

const update = (id, newObject) => {
  const req = axios.put(`${baseUrl}/${id}`, newObject);
  return req.then((res) => res.data);
};

export default {
  setToken: setToken,
  getAll: getAll,
  create: create,
  update: update,
};
```

Here, we made a few updates, including updating our variable names to their shortened form to better reflect our Express application. With regard to updating our token, we created a `setToken()` function that updates the value of `token` (which is a private variable that belongs to the module).

Our `create()` function now uses `async`/`await` syntax and sets the appropriate `Authorization` header through an object given as the third parameter of the `axios` method call. Let's also update our `handleLogin()` function to set the token in our `notes` service module:

**/Frontend/src/App.js**

```js
//...

const handleLogin = async (event) => {
  event.preventDefault();
  try {
    const user = await loginService.login({
      username,
      password,
    });

    setUser(user);
    setUsername("");
    setPassword("");
    noteService.setToken(user.token);
  } catch (exception) {
    console.log(exception);
    setNotificationMessage("Incorrect credentials: " + exception);
    setTimeout(() => {
      setNotificationMessage(null);
    }, 3000);
  }
};

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
  event.preventDefault();
  try {
    const user = await loginService.login({
      username,
      password,
    });

    setUser(user);
    setUsername("");
    setPassword("");
    window.localStorage.setItem(
      // Saves user information to local storage
      "noteAppUser",
      JSON.stringify(user)
    );
    noteService.setToken(user.token);
  } catch (exception) {
    console.log(exception);
    setNotificationMessage("Incorrect credentials: " + exception);
    setTimeout(() => {
      setNotificationMessage(null);
    }, 3000);
  }
};

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
  event.preventDefault();
  try {
    window.localStorage.removeItem("noteAppUser");
    noteService.setToken(null);
    setUser(null);
  } catch (exception) {
    console.log(exception);
  }
};

//...
```

Here, we simply created a handler function for our logout button that removes the user information from local storage so that we are no longer automatically signed in upon loading the page. Additionally, we set our `user` piece of state and the token to `null` to 'log out' in our current session. Now, we just have to add the button to the JSX that our component returns:

**/Frontend/src/App.js**

`````````jsx
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
`````````

Here, we conditionally render the sign out button if the user is signed in. This button calls the `handleLogout` handler function upon the `onClick` event (when the button is clicked) which logs the user out by removing the user information from local storage.

## Composition and React `props.children`

Although we have been writing in a mostly functional side, React resembles an object oriented nature with many parallels such as allowing the defining of objects that contain and directly manipulate data without defining how they are used. Components are logically separated and can be used as building blocks for bigger purposes.

This brings us to two ideas: composition and inheritance. In basic terms, composition is when an object/class utilizes another object to provide some or all of its functionality. Problems are broken down into modular solutions to which components build on top of each other to provide the ultimate solution. Inheritance involves creating a basic structure to which other subclasses inherit the implementation and augment/reuse this basic structure.

With this, React has a powerful composition model and there are almost no cases where one would benefit from using inheritance over composition to reuse code between components. With this, in order to utilize composition, we have to build upon smaller components.

Our current way of dealing with components does not work in some cases as some components don't know their children ahead of time. This is where `props.children` comes in.

Let's update our application so that the login form is only displayed when the user wants to sign in. We'll achieve this by creating a button that causes the form to appear and also by creating a button that hides the form. For the sake of organization and maintainability, as our **App.js** file is growing too large, we'll extract the login form into its own dedicated component:

**/Frontend/src/components/LoginForm.js**

```jsx
import React from "react";

const LoginForm = ({
  handleLogin,
  handleUsernameChange,
  handlePasswordChange,
  username,
  password,
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
  );
};

export default LoginForm;
```

As our state is outside of this component and we have not yet moved it over, we created two new functions `handleUsernameChange` and `handlePasswordChange` that will update our state for us from within our component. We'll talk about the definition of these functions in a moment.

Additionally, we can keep the JSX that our component returns the same for the most part for now. We're able to keep these references, even though that we're now referring to props, as we destructure these objects from the `props` object in the parameter list of our function.

For example, instead of writing `this.props.handleLogin` in a class component or `props.handleLogin` when `props` is passed in as the sole parameter of our function as follows:

```jsx
const example = (props) => (
  <form onSubmit={props.handleLogin}>{/* ... */}</form>
);
```

We can instead destructure the `handleLogin` function from the `props` object in the parameter list, so that we can instead write:

```jsx
const example = ({ handleLogin }) => (
  <form onSubmit={handleLogin}>{/* ... */}</form>
);
```

Doing this comes with a number of advantages. Assigning objects to a local variable (doesn't have to just be through destructuring) can allow your code to be minified better when the value is used repeatedly (since the reassigned name can be mangled).

Destructuring also prevents having to do repeated property lookups. Property lookups can be "slow" for objects like `this.props` or `this.state`, although, in practice, this might not have too much of an effect.

Let's proceed with adding a way to track the visibility of our `LoginForm` in our state as well as control the visibility of certain elements based on this state in a new component:

**/Frontend/src/App.js**

```jsx
import LoginForm from "./components/LoginForm";

//...

const App = () => {
  //...

  const [loginVisible, setLoginVisible] = useState(false);

  //...

  const loginForm = () => {
    const hideWhenVisible = { display: loginVisible ? "none" : "" };
    const showWhenVisible = { display: loginVisible ? "" : "none" };

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
    );
  };

  //...
};

//...
```

Here, we create a new "component" that makes use of our `LoginForm` component in it.

Stepping through the changes, we added a piece of state called `loginVisible` that contains a boolean value pertaining to whether the login form should be displayed.

Then, from within our `loginForm` "component" that returns JSX that takes advantage of our `LoginForm` component, we define two style objects that contain properties that depend on the `loginVisible` state.

```jsx
const hideWhenVisible = { display: loginVisible ? "none" : "" };
const showWhenVisible = { display: loginVisible ? "" : "none" };
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
<Togglable buttonLabel="Login">
  <LoginForm
    handleLogin={handleLogin}
    handleUsernameChange={({ target }) => setUsername(target.value)}
    handlePasswordChange={({ target }) => setPassword(target.value)}
    username={username}
    password={password}
  />
</Togglable>
```

Previously, most of our components had been _singleton tags_ or _self-closing tags_ where we do not have an opening and closing tag, but rather just one, single tag in the format of `<Element />`.

Here, `Togglable` has both an opening and closing tag. With this, any elements that fall between the tags `<Togglable>` and `</Togglable>` are considered child components of `Togglable`.

The functionality of `Togglable` involves creating a button with its label set to the `buttonlabel` prop. Upon clicking this button, all child elements will be visible and a new "Cancel" button will appear, allowing the user to hide all of the elements again.

Let's actually write our `Togglable` component:

**/Frontend/src/components/Togglable.js**

```jsx
import React, { useState } from "react";

const Togglable = ({ buttonLabel, children }) => {
  const [visible, setVisible] = useState(false);

  const hideWhenVisible = { display: visible ? "none" : "" };
  const showWhenVisible = { display: visible ? "" : "none" };

  const toggleVisibility = () => {
    setVisible(!visible);
  };

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
  );
};

export default Togglable;
```

Here, our logic involving toggling the visibility of elements has mostly stayed the same with the exception of a dedicated function that now toggles the `visibility` piece of state, and the movement of the piece of state that tracks the visibility of our component into the `Togglable` component itself.

Everything here seems familiar up until the reference to `children` which has been destructured from `props`.

This entire time, React, in the background, has secretly been adding a `children` property to our `props` object. `props.children` is used in order to reference the child components of our component, so that we can actually include it in our JSX that our component returns. Recall that we discussed some potential limitations with components that might depend on their children or need to know what their children return ahead of time prior to rendering. This is what `props.children` solves.

The `children` property contains an array of all the elements that were defined between the opening and closing brackets of the component. If the component is a singleton, `props.children` will just contain an empty array.

We can now update our `App` component to simplify our code and make use of this new `Togglable` component:

**/Frontend/src/App.js**

```jsx
import Togglable from "./components/Togglable";

//...

const App = () => {
  //...

  const loginForm = () => {
    return (
      <Togglable buttonLabel="Login">
        <LoginForm
          handleLogin={handleLogin}
          handleUsernameChange={({ target }) => setUsername(target.value)}
          handlePasswordChange={({ target }) => setPassword(target.value)}
          username={username}
          password={password}
        />
      </Togglable>
    );
  };

  //...
};

//...
```

With this, we have drastically simplified the `loginForm` function in our `App` component, increasing maintainability and readability. Let's also apply this to our notes form. First let's move the form into its own component:

**/Frontend/src/components/NoteForm.js**

```js
const noteForm = ({ addNote, noteField, handleInputChange }) => (
  <form onSubmit={addNote}>
    <input value={noteField} onChange={handleInputChange} />
    <button type="submit">Add Note</button>
  </form>
);

export default noteForm;
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
);

//...
```

Now, with `Togglable`, we control the visibility of the component through state that is found within the component rather than within our `App` component. This is better for organization, but it limits us in some ways. For example, how might we go about hiding the note form after submitting a form (handled by a different function) if the visibility of the form is controlled by state found within the component?

We can solve this problem by using refs which gives us a reference to the rendered elements (DOM nodes) to which we can interact with manually. This is often not the best solution as there is often a solution involving a better place to "own" state higher up in the component hierarchy, allowing us to do things purely through state. For now, though, we'll just be using `ref`s:

**/Frontend/src/App.js**

```jsx
//...

const noteFormRef = React.createRef();

const noteForm = () => (
  <Togglable buttonLabel="Create Note" ref={noteFormRef}>
    <NoteForm
      addNote={addNote}
      noteField={noteField}
      handleInputChange={handleInputChange}
    />
  </Togglable>
);

//...
```

Here, we use the `createRef` method to create a ref that we then assign to the `Togglable` wrapping component that wraps the note form. With this, `noteFormRef` is a reference to our `Togglable` component.

To do this, you'll notice that we had passed in the ref as an attribute. Let's update our `Togglable` component to support refs:

**/Frontend/src/components/Togglable.js**

```jsx
import React, { useState, useImperativeHandle } from "react";

const Togglable = React.forwardRef(({ buttonLabel, children }, ref) => {
  const [visible, setVisible] = useState(false);

  const hideWhenVisible = { display: visible ? "none" : "" };
  const showWhenVisible = { display: visible ? "" : "none" };

  const toggleVisibility = () => {
    setVisible(!visible);
  };

  useImperativeHandle(ref, () => {
    return {
      toggleVisibility,
    };
  });

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
  );
});

export default Togglable;
```

Here, we made a few changes. We first wrapped our functional component inside of a call to `forwardRef` which will allow our component to access the reference that was passed as an attribute to it.

We also call the `useImperativeHandle` hook which takes in the reference and a function that returns an object containing all of the functions that we might want to make accessible via the ref. In this example, we give it an object containing the `toggleVisibility` function, allowing us to call it through the reference available outside of the component.

Being able to call `toggleVisibility` within that component from outside of the component using the `noteFormRef` that we had created allows us to hide the form after creating a new note. We can do this by updating our `addNote` handler to use this ref:

**/Frontend/src/App.js**

```js
//...

const addNote = (event) => {
  event.preventDefault();
  noteFormRef.current.toggleVisibility();
  const newNoteObject = {
    content: noteField,
    date: new Date(),
    flagged: Math.random() > 0.5,
  };

  noteService.create(newNoteObject).then((data) => {
    setNotes(notes.concat(data));
    setNoteField("Enter a new note here...");
  });
};

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
import React from "react";
import PropTypes from "proptypes";

const LoginForm = ({
  handleLogin,
  handleUsernameChange,
  handlePasswordChange,
  username,
  password,
}) => {
  return {
    /*...*/
  };
};

LoginForm.propTypes = {
  handleLogin: PropTypes.func.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired,
};

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
import React from "react";
import "@testing-library/jest-dom/extend-expect";
import { render } from "@testing-library/react";
import Note from "./components/Note";

test("note renders content", () => {
  const note = {
    content: "This is a test note",
    flagged: true,
  };

  const noteComponent = render(<Note note={note} toggleFlagged={jest.fn()} />);

  expect(noteComponent.container).toHaveTextContent("This is a test note");
});
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
import React from "react";
import "@testing-library/jest-dom/extend-expect";
import { render } from "@testing-library/react";
import Note from "./components/Note";

test("note renders content", () => {
  const note = {
    content: "This is a test note",
    flagged: true,
  };

  const noteComponent = render(<Note note={note} toggleFlagged={() => {}} />);

  noteComponent.debug(); // Logs the HTML of the component to the console

  expect(noteComponent.container).toHaveTextContent("This is a test note");
});
```

Upon running our tests, we receive the following output with the HTML of our component:

![image-20191226194358291](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191226194358291.png)

`debug()` is simply a shortcut for `console.log(prettyDOM())` to which `prettyDOM` comes from the `DOM Testing Library` and just lays out the HTML of our component in a readable manner.

Let's import and use `prettyDOM ` manually to inspect the contents of a smaller element from within our component:

```js
import React from "react";
import "@testing-library/jest-dom/extend-expect";
import { render } from "@testing-library/react";
import { prettyDOM } from "@testing-library/dom"; // prettyDOM import
import Note from "./components/Note";

test("note renders content", () => {
  const note = {
    content: "This is a test note",
    flagged: true,
  };

  const noteComponent = render(<Note note={note} toggleFlagged={() => {}} />);

  noteComponent.debug(); // Logs the HTML of the component to the console
  const listElement = noteComponent.container.querySelector("li"); // use querySelector to get our list element

  console.log(prettyDOM(listElement)); // logs the prettyDOM object

  expect(noteComponent.container).toHaveTextContent("This is a test note");
});
```

Now, as we begin to write extensive tests, you'll find that things can become quite repetitive with all of the patterns involving checking for an element's attributes, text content, finding by CSS class, etc.

`jest-dom` attempts to provide a partial solution by providing custom jest matchers that test the state of the "DOM".

We could import the matchers by adding the following line to our test files:

```jsx
import "@testing-library/jest-dom/extend-expect";
```

You'll notice that we are not assigning it to a variable or anything. With this, to prevent the repetitive nature of having to import all of these modules in every test file that we write, let's create a configuration file that imports these modules for us (in newer versions of `create-react-app` the file may already be created):

**/Frontend/src/setupTests.js**

```jsx
import "@testing-library/jest-dom/extend-expect";
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

test("checking the checkbox fires event handler once", () => {
  const note = {
    content: "This is a test note",
    flagged: true,
  };

  const mockHandler = jest.fn();

  const { getByAltText } = render(
    <Note note={note} toggleFlagged={mockHandler} />
  );

  const checkbox = getByAltText("Toggle Flagged");
  fireEvent.click(checkbox);

  expect(mockHandler.mock.calls.length).toBe(1);
});

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
import React, { useState, useImperativeHandle } from "react";
import PropTypes from "proptypes";

const Togglable = React.forwardRef(({ buttonLabel, children }, ref) => {
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
  );
});

//...

export default Togglable;
```

Now, we can go ahead and write tests for our `Togglable` component:

**/Frontend/src/Togglable.test.js**

```jsx
import React from "react";
import { render, fireEvent } from "@testing-library/react";
import Togglable from "./Togglable";

describe("<Togglable /> Component Tests", () => {
  let component;

  beforeEach(() => {
    component = render(
      <Togglable buttonLabel="Show Children">
        <div className="childDiv" />
      </Togglable>
    );
  });

  test("children are rendered", () => {
    component.container.querySelector(".childDiv");
  });

  test("children are not displayed when not shown", () => {
    const div = component.container.querySelector(".togglableContent");

    expect(div).toHaveStyle("display: none");
  });

  test("children are displayed after clicking show button", () => {
    const button = component.getByText("Show Children");
    fireEvent.click(button);

    const div = component.container.querySelector(".togglableContent");
    expect(div).not.toHaveStyle("display: none");
  });
});
```

Here, we simply wrap all of our tests in a describe block and employ other concepts that we have already been exposed to. Before each test, we render the `Togglable` component to the `component` variable of which the `Togglable` component has a `div` element as its child. We then check if it is rendered, if the child div is not shown (has `display: 'none'`), and if the child div is shown after clicking on the button (does not have `display: 'none'`).

Now, something to note is that `querySelector` returns the first match that it gets based on the query. In our case, it returned the first button in our component and not the second "cancel" button that is currently hidden.

This leads to the question of retrieving elements that are not the first query match. This is done through the query using a selector like we would in CSS. Let's define a test that tests our components ability to hide the content upon clicking the cancel button:

**/Frontend/src/Togglable.test.js**

```jsx
//...

test("children are hidden after clicking cancel button", () => {
  const showButton = component.getByText("Show Children");
  fireEvent.click(showButton);

  const cancelButton = component.container.querySelector("button:nth-child(2)");
  fireEvent.click(cancelButton);

  const div = component.container.querySelector(".togglableContent");
  expect(div).toHaveStyle("display: none");
});

//...
```

Here, we simply use the query `button:nth-child(2)` to find the second element that matches the query. Finding components in this manner is not recommended and should be identified based on text, alt-text, or ID:

**/Frontend/src/Togglable.test.js**

```jsx
//...

test("children are hidden after clicking cancel button", () => {
  const showButton = component.getByText("Show Children");
  fireEvent.click(showButton);

  const cancelButton = component.getByText("Cancel");
  fireEvent.click(cancelButton);

  const div = component.container.querySelector(".togglableContent");
  expect(div).toHaveStyle("display: none");
});

//...
```

#### Form and Input Testing

Let's expand our tests to test our `NoteForm` component.

In order to properly test this component, we would have to enter some value into our `input` element and check the outcome. We can do this by taking advantage of `fireEvent` once again to fill out our input element:

**/Frontend/src/components/NoteForm.test.js**

```jsx
import React from "react";
import { render, fireEvent } from "@testing-library/react";
import NoteForm from "./NoteForm";

const Wrapper = (props) => {
  const handleInputChange = (event) => {
    props.state.noteField = event.target.value;
  };

  return (
    <NoteForm
      addNote={props.addNote}
      noteField={props.state.noteField}
      handleInputChange={handleInputChange}
    />
  );
};

test("<NoteForm /> updates through handleInputChange and calls addNote", () => {
  const addNote = jest.fn();
  const state = {
    noteField: "",
  };

  const component = render(<Wrapper addNote={addNote} state={state} />);

  const input = component.container.querySelector("input");
  const form = component.container.querySelector("form");

  fireEvent.change(input, { target: { value: "Test form input" } });
  fireEvent.submit(form);

  expect(addNote.mock.calls.length).toBe(1);
  expect(state.noteField).toBe("Test form input");
});
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
import "@testing-library/jest-dom/extend-expect";

let savedItems = {};

const localStorageMock = {
  setItem: (key, item) => {
    savedItems[key] = item;
  },
  getItem: (key) => savedItems[key],
  clear: () => {
    savedItems = {};
  },
};

Object.defineProperty(window, "localStorage", { value: localStorageMock });
```

Here, we declared a `savedItems` varaibles which will act as the storage for our mock. We then have an object `localStorageMock` that contains two functions which will allow us to modify this object. Finally, we use `Object.defineProperty` to add this `localStorageMock` object to the `window` object under the property `localStorage`.

Now, to address the problem of fetching notes from our MongoDB database, we can simply replace it with a hardcoded object that represents the collection of our notes. The use of hardcoded data also allows increased speed and stability as we do not have to access a database whose state may change over time.

With this, let's mock our `noteService` module with a module that makes use of and returns hardcoded data. In Jest, manual mocks of modules are defined through writing a module in a **\_\_mocks\_\_** subdirectory in the same directory of the module. As our `noteService` module (**/Frontend/src/services/notes.js**) lies within the **services** directory, we will make a **\_\_mocks\_\_** directory within it and place our mock service within it:

**/Frontend/src/services/\_\_mocks\_\_/notes.js**

```jsx
const notes = [
  {
    content: "This is a mock note!",
    flagged: true,
    date: "2019-12-23T04:32:47.154Z",
    user: {
      username: "jassuf",
      name: "joe",
      id: "5df151019f8679503a3de183",
    },
    id: "5e00436f4001db92b2ab0a00",
  },
  {
    content: "This is another mock note.",
    flagged: false,
    date: "2019-12-23T04:33:21.129Z",
    user: {
      username: "jassuf",
      name: "joe",
      id: "5df151019f8679503a3de183",
    },
    id: "5e0043914001db92b2ab0a01",
  },
  {
    content: "This is the third mock note.",
    flagged: false,
    date: "2019-12-23T07:11:38.841Z",
    user: {
      username: "jassuf",
      name: "joe",
      id: "5df151019f8679503a3de183",
    },
    id: "5e0068aa4001db92b2ab0a02",
  },
];

const getAll = () => {
  return Promise.resolve(notes);
};

export default { getAll };
```

Here, we hardcoded the data that our database might have. We also created a mock `getAll` function that we would find within our real `noteService`. This function actually returns a promise that will resolve to the list of notes. We do this because, in our actual application, we return a promise as a result of the `then()` method of the promise that `axios.get()` gives us.

We can now go ahead and write our integration test relating to our `App` component:

**/Frontend/src/App.test.js**

```jsx
import React from "react";
import { render, waitForElement } from "@testing-library/react";
jest.mock("./services/notes");
import App from "./App";

describe("<App /> Integration Tests", () => {
  test("renders all notes from service", async () => {
    const component = render(<App />);
    component.rerender(<App />);
    await waitForElement(() => component.container.querySelector(".mainNote"));

    const notes = component.container.querySelectorAll(".mainNote");
    expect(notes.length).toBe(3);

    expect(component.container).toHaveTextContent("This is a mock note!");
    expect(component.container).toHaveTextContent("This is another mock note.");
    expect(component.container).toHaveTextContent(
      "This is the third mock note."
    );
  });
});
```

Through a call to `jest.mock()`, `jest` will find the import of the referenced module and check if there is a mock folder with the appropriate mock module in the same directory of the original module. If there is, it will use the manual mock, but if not, it will go ahead and create an _automatic mock_ with functions that typically return `undefined`.

`jest` allows for automatic mocks and extending actual modules from within manual mocks, but given the size of our current project, we are not going to discuss this at this time. As your projects scale, it is recommended to read more of the `jest` documentation in order to take advantage of newer features.

Looking at our test, we first call a `rerender` and await the rendering of all of our elements that have the `.mainNote` class. We rerender our component just to ensure that our effects are triggered, even though our current events should execute after the first render.

### Determining Test Coverage

As our projects begin to grow larger and the amount of tests that we write increases, it would be helpful to be able to see what parts of our project do not have tests written for them. We can determine the _test coverage_ of our project with `jest`'s integrated coverage reported which typically requires no configuration.

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

Jest also offers another form of testing with regard to _snapshot_ testing. Snapshot testing is useful for when you want to ensure that the user interface hasn't change. WIth this, the core principle of snapshot testing is very simple: upon creating the test, a snapshot will be generated of a given component, etc. to which this snapshot is compared to the live rendering of a component.

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

- Hooks can only be called at the top level of the component
  - This means that they should not be called inside loops, conditional blocks, nested functions, etc.
    - This ensures that the hooks are called in the exact same order every time the component renders, allowing React to hold on and preserve the state between multiple calls.
    - As for background, when taking the `useState()` hook as an example, you might wonder how React knows which piece of state corresponds to each `useState()` call. It does this through tracking the order of which the hooks are called. This is why it is important that hooks are called within the top level of the component, because if they are executed conditionally or multiple times, it alters the order and leads to the behavior of our hooks to shift.
- Hooks can only be called from React functions
  - We can only call hooks from React function components and within other custom hooks.
    - This allows us to see all stateful logic related to a component from standardized areas in its source code.

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
  plugins: ["react", "jest", "react-hooks"],
  rules: {
    "react-hooks/rules-of-hooks": "error",
    //...
  },
};
```

We will now gain access to syntax highlighting and underlining for hooks within VSCode and will get thrown an error if we violate these rules upon linting our project.

As an introduction to hooks, let's just create a simple counter using custom hooks. Previously, we might have created a counter as follows:

```jsx
import React, { useState } from "react";
const App = (props) => {
  const [counter, setCounter] = useState(0);

  return (
    <div>
      <div>{counter}</div>
      <button onClick={() => setCounter(counter + 1)}>plus</button>
      <button onClick={() => setCounter(counter - 1)}>minus</button>
      <button onClick={() => setCounter(0)}>zero</button>
    </div>
  );
};
```

With React Hooks, we can first abstract the logic involving the counter and its state to its own hook:

```jsx
const useCounter = () => {
  const [value, setValue] = useState(0);

  const increase = () => {
    setValue(value + 1);
  };

  const decrease = () => {
    setValue(value - 1);
  };

  const reset = () => {
    setValue(0);
  };

  return {
    value,
    increase,
    decrease,
    reset,
  };
};
```

Notice how we call the `useState()` hook from within our own custom hook. Here, our hook behaves similarly to how an abstraction over our mock local storage module behaved. This hook, upon being called, will return an object containing functions that allow us to set the internal state of this component, as well as the internal state itself.

Now, we can make use of this hook within a component:

```jsx
const App = (props) => {
  const counter = useCounter();

  return (
    <div>
      <div>{counter.value}</div>
      <button onClick={counter.increase}>Increase</button>
      <button onClick={counter.decrease}>Decrease</button>
      <button onClick={counter.zero}>Reset</button>
    </div>
  );
};
```

Here, in our component, we just lay out the elements of our counter and make use of the `useCounter()` hook for all of our logic and state relating to the counter.

In this case, unlike what we did earlier with `useState()`, we did not destructure the object that our hook returned.

This code is arguably more manageable than the solution mentioned previously that did not make use of a custom hook, but the true potential of custom hooks is realized when we reuse this logic.

If we wanted a component that required two counters within it, rather than making a specific counter component and then a parent wrapper component, we can easily take advantage of our custom hook in one component. For example, let's create an component that has counters for the amount of times a button has been clicked/side has been voted on:

```jsx
const App = () => {
  const leftVote = useCounter();
  const rightVote = useCounter();

  return (
    <div>
      {leftVote.value}
      <button onClick={leftVote.increase}>Vote for Left</button>
      <button onClick={rightVote.increase}>Vote for Right</button>
      {rightVote.value}
    </div>
  );
};
```

Here, our component contains two separate counters with their logic in separate instances of the `useCounter()` hook. We then call the respective functions of the hooks and display their states to provide the interaction and display of the two counters.

Now, let's look into a more applicable hook for our application. From our previous experiences, dealing with forms in React can be quiet unorganized sometimes, especially when we need to assume control of multiple input elements (making them into controlled components so that we can track their state).

Thinking about this, the logic behind creating a controlled form element can be reused across multiple instances in a form. With this, let's create a hook that will hold and manage the state for a given controlled element:

```jsx
const useField = (type) => {
  const [value, setValue] = useState("");

  const onChange = (event) => {
    setValue(event.target.value);
  };

  return {
    type,
    value,
    onChange,
  };
};
```

Here, we create a piece of state so that we can keep track of what is typed in our input element. We then return a function that takes in an event and sets our internal state to the target value of that event (this is what we can pass to the `onChange` property of our actual form element). We also keep track of the element type and return that as well as the state.

Later, if you end up using Formik in an application (form library for React), `useField()` is a custom React hook that will allow you to hook up inputs to Formik. We emulate this behavior here.

As a quick example, we can use the hook as such:

```jsx
const App = () => {
  const userEmail = useField("text");
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
  );
};
```

Here, our `input` element is controlled by the logic in the `userEmail` hook. In this case, since our key values are the same as the key values in the object that we're trying to pass values from, we can use the spread operator to break down our object into the correct fields:

```jsx
const App = () => {
  const userEmail = useField("text");
  // ...

  return (
    <div>
      <form>
        <input {...userEmail} />
      </form>
    </div>
  );
};
```

This produces the exact same result as manually referencing the properties of the object and matching them up with the appropriate property of the element.

As per the React documentation, the following two components are equivalent:

```jsx
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = { firstName: "Ben", lastName: "Hector" };
  return <Greeting {...props} />;
}
```

You can see how the `props` object contains the exact same properties that the `Greeting` component takes in, to which when we use the spread operator upon the object, it is broken down and the properties of it are assigned to their respective fields.

With this, many libraries are starting to come with their own custom hooks. Additionally, you can find plenty of hooks and examples of real-world usage of hooks on the Internet. One especially good source for this content would be the `awesome-react-hooks` repository on GitHub.

# State Management with Redux

So far, we've just dealt with simple component state where we just followed the convention of lifting state up to the greatest common ancestor component and passed it down through props. This isn't too manageable with large, complex applications. To address this, Facebook created an alternative approach to state management.

### Flux Architecture

Flux is the application architecture that Facebook uses (they are transitioning to Redux, however) to build client-side web applications. It utilizes a unidirectional data flow, similar to how passing down state through components is unidirectional. Rather than a full framework, Flux is essentially a pattern of state management, to which there are several implementations (see Redux below).

One key difference is that the state is separated completely from React components into their own _stores_. This state is then modified (not directly, but updated) through the use of _actions_. Typically, when an action changes the state of a store, they emit an event that causes the views (components) to be rerendered.

![img](https://miro.medium.com/max/2600/1*Ek68XwgLgxwlAZl6hhxx1w.png)

> Annotated Flux Overview by Facebook: [facebook.github.io/flux/docs/in-depth-overview.html](https://facebook.github.io/flux/docs/in-depth-overview.html)

Here, you can see how state is updated and triggers events in components while being separate from the components. For example, if you clicked a button that incremented a piece of state in a store, the change would have to be made with an action. This action is then provided to the dispatcher who sends the action to all stores. These stores will then update themselves and their state and emit a _change event_ once they are done. Then, the _controller-views_ consisting of components listen for these changes and update their state, triggering a rerender (usually).

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
  type: "INCREMENT";
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

Here, dependent on the action type, we update our state correspondingly. Reducers will take in the original state and the action as inputs and will return the updated state as the output. Now, we must create a store that will bring our actions and reducers together. We also use a _default function parameter_ to set `state = 0` if no value or `undefined` is passed to `state`.

#### Store

The Store is the object that brings the actions (which essentially describe "what happened") and the reducers (which update the state according to the actions) together.

It is important to realize that a reducer should never (except in some very odd cases) be called directly from the application code. This is what the store is for.

The store is responsible for:

- Holding the application state
- Allowing access to the state through a call to `getState()`
- Allowing updates to the state through a call to `dispatch(action)`
- Registering listeners through a call to `subscribe(listener)`
- Handling unregistering of listeners through the function returned by `subscribe(listener)`

The last 2 responsibilities may sound very unfamiliar. Do not worry as we will soon cover these topics. Additionally, it is important to note that we'll only have a singular store in our entire application. Eventually, when we want to use multiple reducers to split our data handling logic, we'll use _reducer composition_ to combine our separate reducers into one single reducer that we can give to our store.

Moving back to our counter application, we can create the store by passing the reducer to `createStore()`:

```jsx
import { createStore } from "redux";

const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    case "RESET":
      return 0;
    default:
      return state;
  }
};

const store = createStore(counterReducer);
```

The store now has our `counterReducer` to handle any actions that we _dispatch_ (send) to it using the `dispatch()` method:

```js
store.dispatch({ type: "INCREMENT" });
```

We can call `dispatch(action)` from anywhere in our application. If we wanted to get the state of our store, we can just call `getState()` upon the store.

#### Registering Listeners via Subscribe

For the purposes of testing our application for now, let's register a listener upon our store that listens to every single change upon it:

```jsx
//...

const store = createStore(counterReducer);

store.subscribe(() => {
  const currentStore = store.getState();
  console.log(currentStore);
});
```

Here, we listen for any change upon the store. Upon a change, the function that we had provided will be executed. In this case, the function simply just logs the store object to the console.

Let's add the following to our application:

```jsx
store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "DECREMENT" });
store.dispatch({ type: "RESET" });
store.dispatch({ type: "INCREMENT" });
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
import React from "react";
import ReactDOM from "react-dom";
import { createStore } from "redux";

const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    case "RESET":
      return 0;
    default:
      return state;
  }
};

const store = createStore(counterReducer);

store.subscribe(() => {
  const currentStore = store.getState();
  console.log(currentStore);
});

const App = () => {
  return (
    <div>
      <div>{store.getState()}</div>
      <button onClick={(e) => store.dispatch({ type: "INCREMENT" })}>
        Increment
      </button>
      <button onClick={(e) => store.dispatch({ type: "DECREMENT" })}>
        Decrement
      </button>
      <button onClick={(e) => store.dispatch({ type: "RESET" })}>Reset</button>
    </div>
  );
};

const renderDOM = () => {
  console.log("Rendering application...");
  ReactDOM.render(<App />, document.getElementById("root"));
};

renderDOM();
store.subscribe(renderDOM);
```

Here, we create an `App` component that simply calls the appropriate dispatcher for the button that emits the `onClick` event. We display the current counter value by grabbing the state of the store using `getState()`. Finally, at the end, we create a function that will call `ReactDOM.render()` to render our `App` component. We also subscribe to the store and pass the `renderDOM` function to it so that our component will be rerendered upon every change to our state.

> Note that we have to manually call `renderDOM()` as the function would not execute otherwise, leading to the component not being displayed.

### Redux + Notes Application

With the general idea of Redux and Flux Architecture down, let's first start making a version of our notes application that uses Redux but does not connect to the backend (for the sake of simplicity):

**/src/index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { createStore } from "redux";

import noteReducer from "./reducers/noteReducer";

const store = createStore(noteReducer);

store.dispatch({
  type: "NEW_NOTE",
  data: {
    content: "This note is stored in the Redux store",
    flagged: true,
    id: 1,
  },
});

store.dispatch({
  type: "NEW_NOTE",
  data: {
    content: "This is a new note created with Redux",
    flagged: false,
    id: 2,
  },
});

const App = () => {
  return (
    <div>
      <ul>
        {store.getState().map((note) => (
          <li key={note.id}>
            <input
              type="checkbox"
              alt="Toggle Flagged"
              checked={note.flagged}
            />
            {note.content}
          </li>
        ))}
      </ul>
    </div>
  );
};

const renderDOM = () => {
  console.log("Rendering application...");
  ReactDOM.render(<App />, document.getElementById("root"));
};

renderDOM();
store.subscribe(renderDOM);
```

Our initial app does not have any project organization with separate modules, but it does lay out the groundwork for the structure of our notes and being able to send out note actions to add notes to our store using our `noteReducer`.

The two `dispatch(action)` calls create two `NEW_NOTE` actions to which the payload has a `data` property which contains our actual note "object". This payload is received by the `noteReducer` reducer to which it concatenates the data if the action was of the type `NEW_NOTE`. We use `concat()` rather than `push()` (just as we did with React components) on the data as reducers are meant to be _pure functions_ (functions that take in an input and return a value without modifying anything) to which we do not modify the store/state directly, but rather return a new state with the updated information. Pure functions, due to their nature of not being dependent on state or affecting state will return the same response every single time they are called with the same parameters.

Another thing to note is the manner of which we map over our store's state in order to render an unordered list of our store's notes.

### Testing Reducers

Let's employ some aspects of test-driven development by creating a test for our `noteReducer`'s current features as well as its ability to toggle the flagged property of a note, keeping in mind that we had not implemented this yet.

Before creating our tests, let's move our reducer over to its own dedicated directory, **reducers**, for the sake of organization:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
  switch (action.type) {
    case "NEW_NOTE":
      return state.concat(action.data);
    default:
      return state;
  }
};

export default noteReducer;
```

For our test, we'll be utilizing a library, `deep-freeze`, that will allow us to freeze objects recursively. This is important, because we can freeze our state and ensure that nothing (our reducers, etc.) is directly modifying it. We can install `deep-freeze` by running:

```bash
npm install --save-dev deep-freeze
```

Similar to React components, we define the tests for a given reducer in the same directory as the reducer, just with a **.test.js** file ending:

**/src/reducers/noteReducer.test.js**

```jsx
import noteReducer from "./noteReducer";
import deepFreeze from "deep-freeze";

describe("noteReducer Unit Tests", () => {
  test("action NEW_NOTE returns new state", () => {
    const state = [];
    const action = {
      type: "NEW_NOTE",
      data: {
        content: "This should update the store state",
        flagged: false,
        id: 1,
      },
    };

    deepFreeze(state);
    const newState = noteReducer(state, action);

    expect(newState.length).toBe(1);
    expect(newState).toContainEqual(action.data);
  });
});
```

Here, we simply freeze the state so that we can't modify it directly, allowing us to see via a thrown error if we are not defining reducers correctly. With this, we just attempt to get the updated state using the `NEW_NOTE` action to which we store it in `newState` and expect it to contain 1 object as well as the notes data.

Let's change our `noteReducer` to utilize `array.push()` instead of `array.concat()` so that it attempts to directly modify the state:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
  switch (action.type) {
    case "NEW_NOTE":
      state.push(action.data);
      return state;
    default:
      return state;
  }
};

export default noteReducer;
```

Upon running our tests, we receive the following error as a result of directly mutating the state that has been frozen:

![image-20191230182443818](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191230182443818.png)

Let's now revert our changes and use `array.concat()` once again. With this, the test surrounding the `NEW_NOTE` action of `noteReducer` passes.

As stated earlier, we'll go ahead and create a test for toggling the flagged property of our notes before actually implementing the feature (test-driven development):

**/src/reducers/noteReducer.test.js**

```jsx
//...

test("action TOGGLE_FLAGGED returns new state with note flagged changed", () => {
  const state = [
    {
      content: "This is stored in the store",
      flagged: false,
      id: 1,
    },
    {
      content: "This state will be updated via actions",
      flagged: true,
      id: 2,
    },
  ];

  const action = {
    type: "TOGGLE_FLAGGED",
    data: {
      id: 2,
    },
  };

  deepFreeze(state);
  const newState = noteReducer(state, action);

  expect(newState.length).toBe(2);

  expect(newState).toContainEqual(state[0]);

  expect(newState).toContainEqual({
    content: "This state will be updated via actions",
    flagged: false,
    id: 2,
  });
});
```

Here, we write a test surrounding the `TOGGLE_FLAGGED` action of `noteReducer`. The action that we perform in the test is:

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
const originalNote = state.find((n) => n.id === noteId);
```

Recall that this method returns the object being passed in the current iteration when the return value of the specified function is truthy.

We then create the updated note object with the `flagged` property:

```js
const updatedNote = {
  ...originalNote,
  flagged: !originalNote.flagged,
};
```

The object spread operator (`...`) is used here to copy the properties over (shallowly) so that we don't have to manually copy over each of the properties. We also made use of this in our other application.

Finally, we return the updated state by mapping over the old state and replacing the referenced nte with the updated note:

```jsx
return state.map((n) => (n.id === noteId ? updatedNote : n));
```

We do this by mapping over the array with a function that checks if the current note that is being iterated over matches the `id` of the action. If the `id` matches, then we simply replace the note with the `updatedNote`. This line makes use of the conditional (ternary) operator.

#### Array Spread Operator

After taking a look at the object spread operator in this reducer, let's talk about the _array spread operator_ which is quite similar conceptually.

The array spread operator allows for an iterable to be expanded into areas that take in more than one argument. Basically, this operator breaks an array into its individual objects, to which these objects are passed in, one by one, into the area where it is called. Let's take a look at an example:

```js
const alphabet = ["a", "b", "c", "d"];

const moreLetters = [...alphabet, "e", "f"];

const lettersAndNumbers = [1, 2, ...alphabet];

const [first, second, ...rest] = moreLetters;

console.log(alphabet);
console.log(moreLetters);
console.log(lettersAndNumbers);
console.log(first);
console.log(second);
console.log(rest);
```

This code outputs the following:

![image-20191230204841434](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191230204841434.png)

The first use of the spread operator in the assignment of `moreLetters` breaks up the `alphabet` array into its individual letters and puts it inside the new array that we are creating. We then add a few more letters to the end of the array.

The use of the array spread operator for `lettersAndNumbers` follows the same concept, only we add two numbers before the use of the spread operator.

Arguably, the most interesting bit of the code is the destructuring of the array and the use of the spread operator in the line:

```js
const [first, second, ...rest] = moreLetters;
```

Here, we destructure based on position. The variable specified in index 0 of the variable declaration side of the expression will be assigned the value of index 0 in the array and so on. Then, the variable following the array spread operator will receive the rest of the array. In other words, when using the array spread operator in assignment, anything that has not been destructured will go into the variable prefixed by the array spread operator.

> Note that you should not use the spread operator as the first variable in an attempt to destructure variables at the end of the array. This essentially just copies the array, and you will not be able to destructure the variables at the end.

In our simplified notes application, the array spread operator actually has an application:

**/src/reducers/noteReducer.js**

```jsx
const noteReducer = (state = [], action) => {
  switch (action.type) {
    case "NEW_NOTE":
      return [...state, action.data];
    case "TOGGLE_FLAGGED": {
      const noteId = action.data.id;
      const originalNote = state.find((n) => n.id === noteId);
      const updatedNote = {
        ...originalNote,
        flagged: !originalNote.flagged,
      };
      return state.map((n) => (n.id === noteId ? updatedNote : n));
    }
    default:
      return state;
  }
};

export default noteReducer;
```

As our state object is just an array, we can use the array spread operator to create a copy of the original state array in our new state array and just add our new note to the end of the array.

#### Forms with Redux

Now, moving on to the ability to actually add notes via the frontend of our application, let's first create various UI elements that dispatch actions for us:

**/src/index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { createStore } from "redux";

import noteReducer from "./reducers/noteReducer";

const store = createStore(noteReducer);

const generateId = () => Math.round(Number(Math.random() * 10000));

const App = () => {
  const addNote = (event) => {
    event.preventDefault();
    const textContent = event.target.noteField.value;
    store.dispatch({
      type: "NEW_NOTE",
      data: {
        textContent,
        important: false,
        id: generateId(),
      },
    });
    event.target.noteField.value = "";
  };

  const toggleFlagged = (id) => () => {
    store.dispatch({
      type: "TOGGLE_FLAGGED",
      data: { id },
    });
  };

  return (
    <div>
      <form onSubmit={addNote}>
        <input name="noteField" />
        <button type="submit">Create Note</button>
      </form>

      <ul>
        {store.getState().map((note) => (
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
        ))}
      </ul>
    </div>
  );
};

const renderDOM = () => {
  console.log("Rendering application...");
  ReactDOM.render(<App />, document.getElementById("root"));
};

renderDOM();
store.subscribe(renderDOM);
```

In the above block of code, you'll find comments describing most of the major additions to our previous iteration of our app. Here, we create a wrapper function around our `TOGGLE_FLAGGED` action dispatch call that takes in an `id` of a note to be updated. Then, this function is set as the `onClick` handler in the JSX for our checkboxes.

In `toggleFlagged`, we utilize function currying once again to return a function, unique to the `id` passed to it, that will go ahead and toggle the flagged property upon being called via a click. We do this as React (if given a function call) calls the function passed to it once when being rendered, and we want the actual function to be assigned to the `onClick` property rather than whatever the function actually returns. We do not have to utilize function currying for the `addNote` function as we do not have to pass a parameter to it, so we can just pass the function without calling it (via the variable name without parentheses).

We also created a function, `addNote`, that is simply the `onSubmit` handler of our form that we created. It just takes in the data from the note input field and creates a note using that information and a randomly generated `id` created by the helper function `generateId`.

One thing to note, unlike our previous note application without Redux, our form is not controlled, meaning that the internal state of the form fields (our `input` element in this case) are not bound to a piece of state, leaving only the DOM to handle the form data (although we can retrieve the data in our event handlers). The React documentation refers to these as uncontrolled components.

It is generally recommended to use controlled components to implement forms as we can later add dynamic validation of inputs, etc. but this implementation will serve us just fine for now.

### Synchronous Action Creators

In the block of code above, we wrapped the `TOGGLE_FLAGGED` action dispatch call into its own function. These functions are called _action creators_. As React components do not actually need to observe the implementation of actions, including their type, we can just abstract these calls into their own functions and simply call the functions from within our components.

With this, putting our action dispatch calls into action creator functions would look something like this:

**/src/index.js**

```jsx
//...

const createNote = (textContent) => {
  return {
    type: "NEW_NOTE",
    data: {
      textContent,
      important: false,
      id: generateId(),
    },
  };
};

const toggleFlaggedOf = (id) => {
  return {
    type: "TOGGLE_FLAGGED",
    data: { id },
  };
};

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
    case "NEW_NOTE":
      return [...state, action.data];
    case "TOGGLE_FLAGGED": {
      const noteId = action.data.id;
      const originalNote = state.find((n) => n.id === noteId);
      const updatedNote = {
        ...originalNote,
        flagged: !originalNote.flagged,
      };
      return state.map((n) => (n.id === noteId ? updatedNote : n));
    }
    default:
      return state;
  }
};

const generateId = () => Math.round(Number(Math.random() * 10000));

export const createNote = (textContent) => {
  return {
    type: "NEW_NOTE",
    data: {
      textContent,
      flagged: false,
      id: generateId(),
    },
  };
};

export const toggleFlaggedOf = (id) => {
  return {
    type: "TOGGLE_FLAGGED",
    data: { id },
  };
};
export default noteReducer;
```

First, we moved the action creators to the `noteReducer` module as we may use it in multiple components, and it makes logical sense for these action creators to be with the reducer. We also moved over the `generateId` helper function, as `createNote` relies on it. We then named exported these two functions. Next, let's reflect these changes in the `App` component:

**/src/App.js**

```jsx
import React from "react";
import { createNote, toggleFlaggedOf } from "./reducers/noteReducer";

const App = ({ store }) => {
  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id));
  };

  return (
    <div>
      <ul>
        {store.getState().map((note) => (
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
        ))}
      </ul>
    </div>
  );
};

export default App;
```

Then, we move the `App` component into its own module from which we take in the store from the props by destructuring `store` from the object. We also have to import `createNote` and `toggleFlaggedOf` from the `noteReducer` module.

Finally, we're left with a simplified **index.js** file:

**/src/index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { createStore } from "redux";
import App from "./App";

import noteReducer from "./reducers/noteReducer";

const store = createStore(noteReducer);

const renderApp = () => {
  ReactDOM.render(<App store={store} />, document.getElementById("root"));
};

renderApp();
store.subscribe(renderApp);
```

This is a good start, but we can go even further by moving everything involved with creating a note into its own component:

**/src/components/NewNote.js**

```jsx
import React from "react";

const NewNote = ({ store }) => {
  const addNote = (event) => {
    event.preventDefault();
    const textContent = event.target.noteField.value;
    store.dispatch(createNote(textContent));
    event.target.noteField.value = "";
  };

  return (
    <form onSubmit={addNote}>
      <input name="noteField" />
      <button type="submit">Create Note</button>
    </form>
  );
};

export default NewNote;
```

Here, we moved the function that essentially bridged our React part of our application to the Redux part of our application into a component that renders the new note form. This makes sense as this form is the only element that makes use of the `addNote` function.

For further simplification, let's create a singular note component and a component that encompasses a bunch of notes in a list:

**/src/components/Note.js**

```jsx
import React from "react";

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
  );
};

export default Note;
```

Here, we have a component for a singular note. We just take in the `toggleFlagged` `onChange` handler and the actual note content so that we can compose the individual note row. This component will be the child component of the `Notes` component which will compose a list of these `Note` components:

**/src/components/Notes.js**

```jsx
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import Note from "./Note";

const Notes = ({ store }) => {
  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id));
  };

  return (
    <ul>
      {store.getState().map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

export default Notes;
```

The `Note` component actually takes in the store as a prop and gets the current state of the store to which we map over it with a function. This function renders a `Note` component for each note in an unordered list with us passing in the note, a key, and the actual `toggleFlagged` function.

As `Note` simply returns JSX and does not contain any logic within it (all logic is passed in through props), we refer to it as a _presentational component_.

`Notes` is referred to as a _container component_, not because it contains multiple instances of another component, but rather because it contains some form of application logic and configures presentational components. In this case, it defines some logic surrounding the event handlers of the `Note` component (upon which the `Note` component will use for a `onChange` event).

All of these changes leave us with a very simple `App` component:

```jsx
import React from "react";
import Notes from "./components/Notes";
import NewNote from "./components/NewNote";

const App = ({ store }) => {
  return (
    <div>
      <NewNote store={store} />
      <Notes store={store} />
    </div>
  );
};

export default App;
```

You'll notice that, even though we receive the `store` prop, we do not actually use it within our `App` component exclusively. Herein lies a downside to forwarding the store to all components, going down the hierarchy. We will explore another approach to accessing the store in just a bit.

### Filtering the Store

Now, like in our previous notes application, let's add the ability to only show notes that have been flagged and vice versa.

First, let's implement the user interface that will call a placeholder handler function:

**/src/App.js**

```jsx
import React from "react";
import Notes from "./components/Notes";
import NewNote from "./components/NewNote";

const App = ({ store }) => {
  let showAll = true; // placeholder

  const filterNotes = (value) => () => {
    alert(value);
    showAll = !showAll;
  };

  return (
    <div>
      <NewNote store={store} />

      <div>
        <button onClick={filterNotes(showAll ? "FLAGGED" : "ALL")}>
          Show {showAll ? "Flagged" : "All"}
        </button>
      </div>

      <Notes store={store} />
    </div>
  );
};

export default App;
```

Here, we create a placeholder `showAll` piece of state (just a variable) so that we can better visualize the logic here. We have a `filterNotes` placeholder function that will be called by the button that we had added.

The button passes in a different string literal to `filterNotes` dependent on the current value of `showAll` using a ternary operator.

With this, we'd like our `filterNotes` handler to trigger an action creator that creates an action that creates a `filter` property upon our store:

```js
{
  notes: [
    //...
  ];
  filter: "ALL";
}
```

Our state object not has two properties: `notes` which contains an array of objects representing the data of our notes, and `filter` which contains a string describing what kind of nots should be displayed.

### Multiple Reducers

For the sake of organization, we'll define a separate module representing the reducer for the `filter` property. This is logical separation in which each property has its own reducer.

Recall that the `createStore(reducer)` function takes in only one reducer, yet we are now creating multiple reducers. We'll explore the solution to this problem in just a moment by combining our reducers.

First, let's implement the reducer that simply returns a new state that contains only the notes that we want displayed:

**/src/filterReducer.js**

```jsx
const filterReducer = (state = "ALL", action) => {
  switch (action.type) {
    case "SET_FILTER":
      return action.filterType;
    default:
      return state;
  }
};

export const filterNotesBy = (filterType) => {
  return {
    type: "SET_FILTER",
    filterType,
  };
};

export default filterReducer;
```

Note how this reducer seems to only be concerned about its "part" of the state object (the `filter` property, rather than the rest of the state). We'll discuss how this is possible in a moment.

We also have defined an action creator that will create `SET_FILTER` actions.

With this new reducer and action creator defined, let's actually try to use it. Since `createStore(reducer)` only takes in one reducer, we can use the `combineReducer()` function to combine the reducers before calling `createStore(reducer)`:

**/src/index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { createStore, combineReducers } from "redux";
import App from "./App";

import noteReducer from "./reducers/noteReducer";
import filterReducer from "./reducers/filterReducer";

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
});

const store = createStore(reducer);

const renderDOM = () => {
  console.log("Rendering application...");
  ReactDOM.render(<App store={store} />, document.getElementById("root"));
};

renderDOM();
store.subscribe(renderDOM);
```

Here, you'll see how we use `combineReducers()` to combine the reducers into one reducer that we can then pass to `createStore(reducer)`. Herein this block of code also explains how these reducers only have to worry about their part/property of the state object.

Within `combineReducers()`, we specify the property relating to the actual property in our state object that we want to manage with its respective reducer as its value. In this case, `noteReducer` will manage the `notes` property of our state object while `filterReducer` manages the `filter` property of our state.

The state that is returned by our reducers registered to the specific properties will replace the value of the property rather than the entire state object. So, if our `filterReducer` returned `'ALL'`, the state object's `filter` property would have the value `'ALL'` rather than the state being replaced with `'ALL'`.

For the following explanation, we'll add a manual dispatch call so that we can observe an interesting behavior:

**/src/index.js**

```jsx
//...

store.dispatch({ type: "SET_FILTER", filterType: "ALL" });

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
      {store.getState().notes.map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

export default Notes;
```

Upon accessing the `notes` property of the state object instead of the state object itself and mapping over that object, our application will now start.

Again, looking at our `App` component, we can further componentize our application by extracting the note filter into its own component:

**/src/components/NoteFilter.js**

```jsx
import React from "react";
import { filterNotesBy } from "../reducers/filterReducer";

const NoteFilter = ({ store }) => {
  const filterNotes = (value) => () => {
    store.dispatch(filterNotesBy(value));
  };

  const showAll = store.getState().filter === "ALL";

  return (
    <div>
      <button onClick={filterNotes(showAll ? "FLAGGED" : "ALL")}>
        Show {showAll ? "Flagged" : "All"}
      </button>
    </div>
  );
};

export default NoteFilter;
```

Here, we moved the JSX and helper function related to the filtering aspect of our application into its own module. Additionally, we replaced the placeholder helper function with a function that will actually create the appropriate action and dispatch the action. We also set `showAll` to appropriate value, dependent on the value of `store.filter`.

Now, for our application to work, all we have to do is update the `Notes` component to take into account the value of the `filter` property of our state object:

**/src/components/Notes.js**

```jsx
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import Note from "./Note";

const Notes = ({ store }) => {
  const { notes, filter } = store.getState();

  const noteList = () => {
    if (filter === "FLAGGED") {
      return notes.filter((n) => n.flagged);
    } else {
      return notes;
    }
  };

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id));
  };

  return (
    <ul>
      {noteList().map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

export default Notes;
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
import React from "react";
import ReactDOM from "react-dom";
import { createStore, combineReducers } from "redux";
import { Provider } from "react-redux";
import App from "./App";

import noteReducer from "./reducers/noteReducer";
import filterReducer from "./reducers/filterReducer";

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
});

const store = createStore(reducer);

ReactDOM.render(
  <Provider store={store}>
    <App store={store} />
  </Provider>,
  document.getElementById("root")
);
```

Here, we got rid of the `renderDOM()` function and the subscription that took it in as a parameter as we no longer need a listener to manually trigger rerenders when our store changes. Instead, we just call the `render()` method of `ReactDOM` directly on a `Provider` which wraps our `App` component.

Now that our `App` component is wrapped in a `Provider`, every single nested component, including `App`, has access to the `connect()` function. Let's first make our `Notes` component into a connected component:

**/src/components/Notes.js**

```jsx
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import { connect } from "react-redux";
import Note from "./Note";

const Notes = ({ store }) => {
  const { notes, filter } = store.getState();

  const noteList = () => {
    if (filter === "FLAGGED") {
      return notes.filter((n) => n.flagged);
    } else {
      return notes;
    }
  };

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id));
  };

  return (
    <ul>
      {noteList().map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

const ConnectedNotes = connect()(Notes);
export default ConnectedNotes;
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
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import { connect } from "react-redux";
import Note from "./Note";

const Notes = ({ notes, filter, store }) => {
  const noteList = () => {
    if (filter === "FLAGGED") {
      return notes.filter((n) => n.flagged);
    }
    return notes;
  };

  const toggleFlagged = (id) => () => {
    store.dispatch(toggleFlaggedOf(id));
  };

  return (
    <ul>
      {noteList().map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter,
  };
};

const ConnectedNotes = connect(mapStateToProps)(Notes);
export default ConnectedNotes;
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
  return x + y;
};

const addOne = add(1);

const testOne = addOne(5); // 1 + 5 === 6

const testTwo = add(1)(5); // 1 + 5 === 6
```

Here, with `addOne`, we assign it with a function call to `add` while only providing one set of parentheses. This returns a function that we can later provide the second set of parentheses to which the first value was placed within the function (`x` in the function declaration was replaced with the value `1`). This is simply the practice of currying functions that we've been exposed to in the past, only with variables that are not immediately curried (return/curried functions that take in parameters).

With `testTwo`, our first parentheses creates a function with `x` replaced with `1` just like we did with `addOne`, only this time, we immediately execute the function that was returned with the parameter(s) in the second parameter and store the return value of that function instead.

TODO: Replace with actual diagram

The above diagram illustrates how `connect()()` returns a "connected" component that has access to certain parts of the store via its props dependent on the assignments specified in `mapStateToProps()`.

### `mapDispatchToProps`

Looking back to our Notes component, you'll notice that, even though we eliminated the need to directly call the store with regard to retrieving state, we still need to call `dispatch()` upon the `store` object in order to dispatch our actions.

React Redux provides a similar solution for dispatching actions: `mapDispatchToProps`. We can pass an object to the second parameter of the first call to `connect()()` that specifies the action creators that we wish to use in our component to which we then will dispatch the actions. Let's make these changes:

**/src/components/Notes.js**

```jsx
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import { connect } from "react-redux";
import Note from "./Note";

const Notes = ({ notes, filter, toggleFlagged }) => {
  const noteList = () => {
    if (filter === "FLAGGED") {
      return notes.filter((n) => n.flagged);
    }
    return notes;
  };

  return (
    <ul>
      {noteList().map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter,
  };
};

const mapDispatchToProps = {
  toggleFlagged: toggleFlaggedOf,
};

const ConnectedNotes = connect(mapStateToProps, mapDispatchToProps)(Notes);
export default ConnectedNotes;
```

Here, we made a few changes to support dispatching from props using `mapDispatchToProps` rather than calling `dispatch()` upon our store directly. Here, we specify the `mapDispatchToProps` object which contains the action creators that we want to be passed to the connected component as props. We don't need to call `dispatch()` manually here, as `connect()()` will create a function that dispatches a given action creator for each one. Within the `mapDispatchToProps` object, we specify the name of the prop that we want for our dispatcher as the property name. The value for that property will be the action creator that is used to create the function that eventually dispatches this action.

With everything handled on the React Redux side, we can just destructure the new `toggleFlagged` prop that we have access to in our component. This prop calls a function that will dispatch the action created by the `toggleFlaggedOf` action creator as specified in our `mapDispatchToProps` object.

Now, instead of having a `toggleFlagged` function that calls `dispatch` directly on the store using the action creator (`store.dispatch(toggleFlagged(note.id))`), we can just call `toggleFlagged(note.id)` to dispatch the action. With this, we no longer need to pass the store into our components.

You might have noticed that the value of `mapDispatchToProps` is not a function unlike `mapStateToProps`. This is the more declarative method of just passing an object as the value of `mapDispatchToProps`. We can provide a function as well and will explore this method in a moment.

#### Side-Effects of Calling Store Directly

We can also go ahead and use `connect()()` to fix a bug that had been introduced. Previously, we would not be able to toggle the filter with 0 elements as our `NoteFilter` component did not rerender when our store's `filter` property changed. As a result, the button would contain text that did not reflect the current state and handler logic that would not affect our state. Using `connect()()` will allow our component to subscribe to the store and rerender when necessary. Let's make the same changes that we did for `Notes` to `NoteFilter`:

**/src/components/NoteFilter.js**

```jsx
import React from "react"; // eslint-disable-line no-unused-vars
import { filterNotesBy } from "../reducers/filterReducer";
import { connect } from "react-redux";
const NoteFilter = ({ filter, filterNotes }) => {
  const showAll = filter === "ALL";

  return (
    <div>
      <button onClick={() => filterNotes(showAll ? "FLAGGED" : "ALL")}>
        Show {showAll ? "Flagged" : "All"}
      </button>
    </div>
  );
};

const mapStateToProps = (state) => {
  return {
    filter: state.filter,
  };
};

const mapDispatchToProps = {
  filterNotes: filterNotesBy,
};

const ConnectedNoteFilter = connect(
  mapStateToProps,
  mapDispatchToProps
)(NoteFilter);
export default ConnectedNoteFilter;
```

#### The Two Forms of `mapDispatchToProps`

The `mapDispatchToProps` parameter of `connect()()` can have either a function or an object as its value:

- Function form: allows us to access `dispatch()` and make the call to it ourselves, giving us more flexibility in customizing its behavior.
- Object form: easier to use as it will automatically convert the action creators specified in the object into functions that call `dispatch()` upon these actions.

We have just explored the object form where:

```jsx
//...

const mapDispatchToProps = {
  filterNotes: filterNotesBy,
};

const ConnectedNoteFilter = connect(
  mapStateToProps,
  mapDispatchToProps
)(NoteFilter);
export default ConnectedNoteFilter;
```

Allows us to access `filterNotes()` from within our component, to which the action creator, `filterNotesBy`, is automatically dispatched via a call to our prop.

Alternatively, we can utilize the function form in order specify the implementation of the dispatch call:

```jsx
//...

const mapDispatchToProps = (dispatch) => {
  return {
    filterNotes: (filter) => {
      dispatch(filterNotesBy(filter));
    },
  };
};

const ConnectedNoteFilter = connect(
  mapStateToProps,
  mapDispatchToProps
)(NoteFilter);
export default ConnectedNoteFilter;
```

Here, we create a function that will eventually take in the `dispatch()` function (provided by `connect()()`) and will return an object containing functions where the keys will be the names of the props passed to the component. This function form achieves the same result as the object form, but gives us some more flexibility with its implementation.

The practicality of this more complex implementation is realized when you need to access the component's props to create a particular action. We can do this through taking in the optional `ownProps` argument.

For example, let's say that we needed to call an action creator `removePostById()` and pass in the component's ID stored in its props. We would do this as follows:

```jsx
const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    removePost: () => {
      dispatch(removePostById(ownProps.postid));
    },
  };
};
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

\*\*/

```jsx
//...

const Notes = ({ notes, filter, toggleFlagged }) => {
  const noteList = () => {
    if (filter === "FLAGGED") {
      return notes.filter((n) => n.flagged);
    }
    return notes;
  };

  return (
    <ul>
      {noteList().map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleFlagged(note.id)}
        />
      ))}
    </ul>
  );
};

//...
```

Instead, we can employ the "separation of concerns" design principle (to a certain extent) by moving the logic that determines what notes will be rendered outside of the function:

```jsx
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import { connect } from "react-redux";
import Note from "./Note";

const Notes = ({
  notes, // Subset of notes returned by the `noteList` selector
  toggleFlagged,
}) => {
  return (
    <ul>
      {notes.map(
        (
          note // We map over the `notes` prop now as it will contain only the notes that we want to display
        ) => (
          <Note
            key={note.id}
            note={note}
            toggleFlagged={() => toggleFlagged(note.id)}
          />
        )
      )}
    </ul>
  );
};

const noteList = ({ notes, filter }) => {
  // Destructures `notes` and `filter` from the state object passed to it
  if (filter === "FLAGGED") {
    return notes.filter((n) => n.flagged);
  }
  return notes;
};

const mapStateToProps = (state) => {
  return {
    notes: noteList(state), // Call to `noteList` returns only the notes that we want to display dependent on the value of `filter` in our state
  };
};

const mapDispatchToProps = {
  toggleFlagged: toggleFlaggedOf,
};

const ConnectedNotes = connect(mapStateToProps, mapDispatchToProps)(Notes);
export default ConnectedNotes;
```

Read the comments in the code above to get a general idea of the changes that we had made. Here, instead of taking in the entire `notes` piece of state and then calling `noteList` upon the prop to filter out the notes that we don't want to display, we moved the logic for filtering the notes outside of the component. `noteList`, which lies outside of our functional component, now takes in the entire `state` object that will be given to it in our connected component. From this `state` object, it filters or _selects_ a subset of the store and returns it. These simple functions that return a subset of data passed to it are called _selectors_.

Within `mapStateToProps()`, we call the `noteList` selector upon the `state` object and pass the resulting object to the `notes` prop. This allows our component to just receive the `notes` prop and map over it without any consideration of logic. All of this culminates in us creating a component that is closer to a _presentational component_.

Recall that a presentational component is not concerned about any of the logic surrounding it (that's left to _container components_), rather, it just takes in values and returns JSX to be _presented_.

### Expanding Upon Presentational and Container Components

Dan Abramov, co-author of Redux and Create React App, states the following regarding presentational components:

"Presentational components:

- Are concerned with _how things look_.
- May contain both presentational and container components\*\* inside, and usually have some DOM markup and styles of their own.
- Often allow containment via _this.props.children_.
- Have no dependencies on the rest of the app, such as Flux actions or stores.
- Don’t specify how the data is loaded or mutated.
- Receive data and callbacks exclusively via props.
- Rarely have their own state (when they do, it’s UI state rather than data).
- Are written as [functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) unless they need state, lifecycle hooks, or performance optimizations."

Our above `Notes` functional component follows most of these guidelines. When we pass this functional component into `connect()()`, it becomes a container component. Dan Abramov also states that container components:

- "Are concerned with _how things work_.
- **May contain both presentational and container components** inside but usually don’t have any DOM markup of their own except for some wrapping divs, and never have any styles.
- Provide the data and behavior to presentational or other container components.
- Call Flux actions and provide these as callbacks to the presentational components.
- Are often stateful, as they tend to serve as data sources.
- Are usually generated using [higher order components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) such as _connect()_ from React Redux, _createContainer()_ from Relay, or _Container.create()_ from Flux Utils, rather than written by hand."

Usually, with separating our components in this manner, we gain a handful of benefits. Dan Abramov also lists the benefits of his separation criteria (listed above) as:

- "Better separation of concerns. You understand your app and your UI better by writing components this way.
- Better reusability. You can use the same presentational component with completely different state sources, and turn those into separate container components that can be further reused.
- Presentational components are essentially your app’s “palette”. You can put them on a single page and let the designer tweak all their variations without touching the app’s logic. You can run screenshot regression tests on that page.
- This forces you to extract “layout components” such as _Sidebar, Page, ContextMenu_ and use _this.props.children_ instead of duplicating the same markup and layout in several container components."

Splitting up components like this may be a good idea sometimes, but it is not good to force this design upon your project if it does not fit well, especially in the case of using the React Hooks API. React Hooks allow us to "separate" stateful logic from the component without splitting the component up.

Dan Abramov mentions a term, "higher-order component", that we had not discussed previously. A higher-order function is simply a function that takes in a component and returns a new component (similar to how a higher-order function takes in a function). Higher-order components are not part of the React API but instead are an advanced pattern that emerges due to the compositional nature of React.

These higher-order components provide an alternative to using mixins, objects that are used to mix additional functionality into React components, with regard to reusing generic functionality. Mixins were determined to be more trouble than they were worth by Facebook and are being removed from React and its documentation.

This concept of defining generic functionality that can be applied to components using a function (for example, `connect()()` applied the connected functionality that our component needs to talk to our store) also loosely relates to the concept of inheritance in object-oriented programming.

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
import axios from "axios";

const baseUrl = "http://localhost:3001/notes";

const getAll = async () => {
  const res = await axios.get(baseUrl);
  return res.data;
};

export default { getAll };
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

import noteService from "./services/noteService"; // Import our Notes Service

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
});

const store = createStore(reducer);

noteService.getAll().then(
  (
    notes // Gets all notes using the Notes Service and dispatches a NEW_NOTE action for each note
  ) =>
    notes.forEach((note) => {
      store.dispatch({
        type: "NEW_NOTE",
        data: note,
      });
    })
);

ReactDOM.render(
  <Provider store={store}>
    <App store={store} />
  </Provider>,
  document.getElementById("root")
);
```

This gets the job done, but we can do better. Let's add support for a new `INIT_NOTES` action in our notes reducer so that we can initialize our notes by dispatching only one action:

**/src/reducers/noteReducer.js**

```jsx
//...

const noteReducer = (state = [], action) => {
  switch (action.type) {
    case "INIT_NOTES":
      return action.data; // Simply sets the `notes` part of the store to the action's data
    case "NEW_NOTE":
      return [...state, action.data];
    case "TOGGLE_FLAGGED": {
      const noteId = action.data.id;
      const originalNote = state.find((n) => n.id === noteId);
      const updatedNote = {
        ...originalNote,
        flagged: !originalNote.flagged,
      };
      return state.map((n) => (n.id === noteId ? updatedNote : n));
    }
    default:
      return state;
  }
};

//...
```

We can also create an action creator for using this new action type:

**/src/reducers/noteReducer.js**

```jsx
//...

export const initNotes = (notes) => {
  return {
    type: "INIT_NOTES",
    data: notes,
  };
};

//...
```

Finally, we can make use of this new action in **index.js** where our initialization simplifies to:

**/src/index.js**

```jsx
//...

import noteReducer, { initNotes } from "./reducers/noteReducer";

//...

noteService.getAll().then(
  (
    notes // We get all notes and use the `initNotes` action creator with `notes` passed to it to dispatch an action that will initialize the `notes` part of our store
  ) => store.dispatch(initNotes(notes))
);

//...
```

We don't make use of `mapDispatchToProps` here since we are not in a React component. Let's move this initialization logic into our `App` component:

**/src/App.js**

```jsx
import React, { useEffect } from "react"; // Import useEffect which will call initNotes
import { connect } from "react-redux"; // Import connect so we can create our connected component

import Notes from "./components/Notes";
import NewNote from "./components/NewNote";
import NoteFilter from "./components/NoteFilter";

import { initNotes } from "./reducers/noteReducer"; // Imports our `initNotes` action creator

import noteService from "./services/noteService"; // Imports our notes service so that we can fetch our notes

const App = ({ initializeNotes }) => {
  // Destructures `initNotes` provided to our connected component through its props

  useEffect(() => {
    noteService.getAll().then((notes) => initializeNotes(notes)); // Retrieves all notes and then calls the `initNotes` prop upon the object it retrieves
  }, [initializeNotes]);

  return (
    <div>
      <NewNote />
      <NoteFilter />
      <Notes />
    </div>
  );
};

const ConnectedApp = connect(null, { initializeNotes: initNotes })(App); // Creates a connected component with a mapDispatchToProps object
export default ConnectedApp;
```

Read the comments in the code above to see every single change that we had made and why we made them. In short, we make use of `react-redux` to create a connected component that has the `initNotes` action creator passed to it as a dispatcher function through its props. This `initNotes` dispatcher (created automatically by `connect()()` from our action creator) is called in the function specified to `useEffect()` upon the fulfillment of the `noteService.getAll()` promise. We make use of the effect hook as we do not want to fetch the data from the server every single time our component renders (we register `initializeNotes` as a dependency of the effect hook as to make sure that we call the function if `initializeNotes` is ever updated and to silence React warnings).

Let's extend this logic to creating new notes so that our notes can be reflected in our "database" as well:

**/src/services/noteService.js**

```jsx
import axios from "axios";

const baseUrl = "http://localhost:3001/notes";

const getAll = async () => {
  const res = await axios.get(baseUrl);
  return res.data;
};

const create = async (textContent) => {
  // New service that posts a note object to our "database"
  const note = {
    textContent,
    flagged: false,
  };
  const res = await axios.post(baseUrl, note);
  return res.data;
};

export default {
  getAll,
  create,
};
```

We can then update the `addNote` method in the `NewNote` component to make use of this service:

**/src/components/NewNote.js**

```jsx
import React from "react";
import { connect } from "react-redux";

import { createNote } from "../reducers/noteReducer"; // Relevant imports

import noteService from "../services/noteService"; // Relevant imports

const NewNote = ({ createNewNote }) => {
  const addNote = async (event) => {
    event.preventDefault();
    const textContent = event.target.noteField.value;
    event.target.noteField.value = ""; // Moved before the awaited method call to preserve atomic updates
    const receivedNote = await noteService.create(textContent); // Uses the create method of our service
    createNewNote(receivedNote); // Dispatches the `createNote` action using the received note from the post request
  };

  return (
    <form onSubmit={addNote}>
      <input name="noteField" />
      <button type="submit">Create Note</button>
    </form>
  );
};

//...
```

The comments above describe the changes that we had made to the code. Essentially, we just send a post request using the `create()` method of our notes service and use the returned note with the `createNewNote()` dispatcher to update our state (instead of the text content).

As we are now passing in the entire note object rather than just the text content to our `createNewNote` dispatcher, we can update the `createNote` action creator to just take in the note without generating an ID (since that's handled for us in the backend):

**/src/reducers/noteReducer.js**

```jsx
//...

export const createNote = (data) => {
  return {
    type: "NEW_NOTE",
    data,
  };
};

//...
```

Finally, we'll add the ability to update a note in our "backend":

**/src/services/noteService.js**

```jsx
import axios from "axios";

const baseUrl = "http://localhost:3001/notes";

const getAll = async () => {
  const res = await axios.get(baseUrl);
  return res.data;
};

const create = async (textContent) => {
  const note = {
    textContent,
    flagged: false,
  };
  const res = await axios.post(baseUrl, note);
  return res.data;
};

const update = async (id, newObject) => {
  // Added update method that takes in the id and the object to replace it with
  const res = await axios.put(`${baseUrl}/${id}`, newObject);
  return res.data;
};

export default {
  getAll,
  create,
  update,
};
```

Now, let's make use of this new service method that we had created in our `Notes` component where we'll create a `toggleNoteFlagged()` wrapper function around our `toggleFlagged()` dispatcher that will both make the appropriate API calls and update our local state:

**/src/components/Notes.js**

```jsx
import React from "react";
import { toggleFlaggedOf } from "../reducers/noteReducer";
import { connect } from "react-redux";
import Note from "./Note";
import noteService from "../services/noteService";

const Notes = ({ notes, toggleFlagged }) => {
  const toggleNoteFlagged = async (note) => {
    const toggledNote = {
      // Create the updated note with the toggled flag
      ...note,
      flagged: !note.flagged,
    };
    await noteService.update(note.id, toggledNote); // Update the note with the updated note
    toggleFlagged(note.id); // Toggle the flagged property in our local state (in a proper solution, we should update the entire object upon receiving a response)
  };

  return (
    <ul>
      {notes.map((note) => (
        <Note
          key={note.id}
          note={note}
          toggleFlagged={() => toggleNoteFlagged(note)}
        />
      ))}
    </ul>
  );
};

const noteList = ({ notes, filter }) => {
  if (filter === "FLAGGED") {
    return notes.filter((n) => n.flagged);
  }
  return notes;
};

const mapStateToProps = (state) => {
  return {
    notes: noteList(state),
  };
};

const mapDispatchToProps = {
  toggleFlagged: toggleFlaggedOf,
};

const ConnectedNotes = connect(mapStateToProps, mapDispatchToProps)(Notes);
export default ConnectedNotes;
```

### Abstraction with Redux-Server Communication

From the code that we have written so far, you might have been wondering if we could refactor the code into something cleaner and easier to understand (especially in the case of functions like `toggleNoteFlagged()`).

Currently, we're communicating with the server using functions that reside inside of our functions. Let's apply the "separation of concerns" design pattern and attempt to abstract these functions away from the insides of our components.

`redux-thunk` allows us to do this by enabling us to write action creators that return a function instead of an action creator. These functions, called _thunks_, receive `dispatch()` as an argument and can call it asynchronously. This allows us to communicate with the server from within the action creators that are created instead of writing an entire wrapper function that calls the dispatcher and the relevant statements to communicate with the server (like we did with `toggleNoteFlagged()`).

To illustrate this, our initialization of notes would be simplified from this:

```jsx
useEffect(() => {
  noteService.getAll().then((notes) => initializeNotes(notes));
}, [initializeNotes]);
```

to this:

```jsx
useEffect(() => {
  initializeNotes(notes);
}, [initializeNotes]);
```

This is because we would be able to perform the server communication involving fetching the notes in the function that is returned (as we are no longer just returning the action).

Let's start by installing `redux-thunk` by executing:

```bash
npm install redux-thunk
```

`redux-thunk` is a _redux middleware_. Recall that middleware in Express is simply some piece of code that you can put between some code during the lifecycle of a request to the server. Redux middleware implements this concept by providing a third-party extension point between dispatching an action and reaching the reducer. Thus, you have an extension point where you can execute code that will be executed after you dispatch an action, but before the reducer receives it (like how you can use Express middleware to modify the `req` object before it actually reaches the controller).

Before we work with `redux-thunk`, let's organize our project further by placing the initialization of our store and combining of our reducers in its own module:

**/src/store.js**

```jsx
import { createStore, combineReducers, applyMiddleware } from "redux";
import thunk from "redux-thunk";

import filterReducer from "./reducers/filterReducer";
import noteReducer from "./reducers/noteReducer";

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
});

const store = createStore(reducer, applyMiddleware(thunk));

export default store;
```

Here, we create our store with our combined reducers and add the `thunk` middleware using `applyMiddleware()`. `applyMiddleware()` is a _store enhancer_ which is a higher-order function that composes a store creator to return a new, enhance store creator. It is unlikely that you will have to work store enhancers, but it is helpful to understand what `applyMiddleware()` actually is.

We can then just import `store` from the module in **index.js**, dramatically simplifying the file:

**/src/index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";
import App from "./App";

import store from "./store";

ReactDOM.render(
  <Provider store={store}>
    <App store={store} />
  </Provider>,
  document.getElementById("root")
);
```

Let's write our first _thunk_ involving the initialization of notes in our application:

**/src/reducers/noteReducer.js**

```jsx
//...

export const initNotes = () => {
  return async (dispatch) => {
    const notes = await noteService.getAll();
    dispatch({
      type: "INIT_NOTES",
      data: notes,
    });
  };
};

//...
```

Here, `initNotes` (the _thunk_ or wrapper function) returns an asynchronous function that awaits the retrieval of notes from the backend and dispatches a `INIT_NOTES` action with the notes in order to initialize our store.

In our `App` component, we can go ahead and use the simplified code that we talked about earlier:

**/src/App.js**

```jsx
import React, { useEffect } from "react";
import { connect } from "react-redux";

import Notes from "./components/Notes";
import NewNote from "./components/NewNote";
import NoteFilter from "./components/NoteFilter";

import { initNotes } from "./reducers/noteReducer";

const App = ({ initializeNotes }) => {
  useEffect(() => {
    initializeNotes();
  }, [initializeNotes]);

  return (
    <div>
      <NewNote />
      <NoteFilter />
      <Notes />
    </div>
  );
};

const ConnectedApp = connect(null, { initializeNotes: initNotes })(App);
export default ConnectedApp;
```

Here, we can just call `initNotes` and watch as it updates our store using data that has been fetched in the thunk. One thing to note is that: when we call `connect()()`, Redux will no longer add the `dispatch` logic for us as we are now defining it in the function form rather than the object shorthand form.

The logic for initializing our notes now resides outside of any of our components. We can apply the same logic to `createNote`:

```jsx
//...

export const createNote = (textContent) => {
  return async (dispatch) => {
    const newNote = await noteService.create(textContent);
    dispatch({
      type: "NEW_NOTE",
      data: newNote,
    });
  };
};

//...
```

With thunks, our wrapper function (the actual thunk) returns a curried function with any of the parameters that we had passed in, to which Redux will pass in `dispatch()` and execute the function when we call the wrapper function.

> The function returned by our asynchronous action creators (thunks) also receive the store's `getState()` method which allows us to read values from the current state of the store by calling `getState()`.

Our `NewNote` component now simply just calls the `createNewNote` thunk rather than making the API requests itself in `addNote()`:

```jsx
import React from "react";
import { connect } from "react-redux";

import { createNote } from "../reducers/noteReducer";

const NewNote = ({ createNewNote }) => {
  const addNote = (event) => {
    event.preventDefault();
    const textContent = event.target.noteField.value;
    event.target.noteField.value = "";
    createNewNote(textContent);
  };

  return (
    <form onSubmit={addNote}>
      <input name="noteField" />
      <button type="submit">Create Note</button>
    </form>
  );
};

const mapDispatchToProps = {
  createNewNote: createNote,
};

const ConnectedNewNote = connect(null, mapDispatchToProps)(NewNote);
export default ConnectedNewNote;
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
import { createStore, combineReducers, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import { composeWithDevTools } from "redux-devtools-extension";

import filterReducer from "./reducers/filterReducer";
import noteReducer from "./reducers/noteReducer";

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
});

const enhancer = composeWithDevTools(applyMiddleware(thunk));

const store = createStore(reducer, enhancer);

export default store;
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
  const [page, setPage] = useState("home"); // State tracking the page that we should render

  const toPage = (page) => (event) => {
    // Updates the state
    event.preventDefault();
    setPage(page);
  };

  const content = () => {
    // Returns the component that we should render dependent on the state
    if (page === "home") {
      return <Home />;
    } else if (page === "notes") {
      return <Notes />;
    } else if (page === "users") {
      return <Users />;
    }
  };

  return (
    <div>
      <div>
        {" "}
        {/* Start navigation bar */}
        <a href="" onClick={toPage("home")}>
          Home
        </a>
        <a href="" onClick={toPage("notes")}>
          notes
        </a>
        <a href="" onClick={toPage("users")}>
          users
        </a>
      </div>{" "}
      {/* End navigation bar */}
      {content()} {/* Content of "page" */}
    </div>
  );
};
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
  Route,
  Link,
  Redirect,
  withRouter,
} from "react-router-dom";

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
  );
};
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

Something interesting to note is that React Router takes advantage of _dynamic routing_. Static routing (found in Express, Ember, Angular, and older versions of React Router) has routes declared before the app listens for any requests (declaring routes up front before the app renders).

React Router with its dynamic routing has routing taking place as the app renders. Routes do not lie within a configuration or convention running outside of the app. This gives us the advantage of dynamically changing routes depending on screen size, state, etc. For example, we might want to render a dashboard view instead of individual sections of the dashboard on larger screens. With this declarative method, on a larger screen, we could automatically redirect the user to the dashboard route and "remove" the routes relating to the smaller screen sizes. This would not be possible with static routing.

#### Transitioning

> From this point on, we'll be going back to our project with an Express backend since we will need a backend server for certain features related to authentication in the future. This is also contingent on having navigation in our application which is why we are making the changes now.

Our old project is in a horrendous state. Tons of logic can be found all in our `App` component where we have a ton of functional components cobbled together to which our app consists of. Let's try to refactor our application a tiny bit so that it is somewhat manageable (please note that this none of the following practices should be looked to as an example). Let's first create a `LoginContainer` component that contains all of the logic that we need surrounding login functionality:

**/Frontend/src/components/LoginContainer.js**

```jsx
import React, { useState } from "react";
import loginService from "../services/login";
import noteService from "../services/notes";
import Togglable from "./Togglable";
import LoginForm from "./LoginForm";

const LoginContainer = ({ user, setUser, setNotificationMessage }) => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const handleLogin = async (event) => {
    event.preventDefault();
    try {
      const user = await loginService.login({
        username,
        password,
      });

      setUser(user);
      setUsername("");
      setPassword("");
      window.localStorage.setItem("noteAppUser", JSON.stringify(user));
      noteService.setToken(user.token);
    } catch (exception) {
      console.log(exception);
      setNotificationMessage("Incorrect credentials: " + exception);
      setTimeout(() => {
        setNotificationMessage(null);
      }, 3000);
    }
  };

  const handleLogout = async (event) => {
    event.preventDefault();
    try {
      window.localStorage.removeItem("noteAppUser");
      noteService.setToken(null);
      setUser(null);
    } catch (exception) {
      console.log(exception);
    }
  };

  const loginForm = () => {
    return (
      <Togglable buttonLabel="Login">
        <LoginForm
          handleLogin={handleLogin}
          handleUsernameChange={({ target }) => setUsername(target.value)}
          handlePasswordChange={({ target }) => setPassword(target.value)}
          username={username}
          password={password}
        />
      </Togglable>
    );
  };

  return user === null ? (
    loginForm()
  ) : (
    <div>
      <p>Welcome, {user.name}</p>{" "}
      <button onClick={handleLogout}>Sign Out</button>
    </div>
  );
};

export default LoginContainer;
```

Note how `LoginContainer` takes in `user`, `setUser`, and `setNotificationMessage` in from its props. We did not move these functions and pieces of state into this component as multiple components rely on this state (lifting state up).

When we eventually render this component in `App`, we will pass the props as follows:

```jsx
<LoginContainer
  user={user}
  setUser={setUser}
  setNotificationMessage={setNotificationMessage}
/>
```

We'll delete the excess code from `App` and continue with moving note creation into its own component:

**/Frontend/src/components/NoteFormContainer.js**

```jsx
import React, { useState } from "react";
import noteService from "../services/notes";
import Togglable from "./Togglable";
import NoteForm from "./NoteForm";

const NoteFormContainer = ({ notes, setNotes }) => {
  const [noteField, setNoteField] = useState("Enter a new note here...");

  const noteFormRef = React.createRef();

  const handleInputChange = (event) => {
    console.log("Input form changed", event.target.value);
    setNoteField(event.target.value);
  };

  const addNote = (event) => {
    event.preventDefault();
    noteFormRef.current.toggleVisibility();
    const newNoteObject = {
      content: noteField,
      date: new Date(),
      flagged: Math.random() > 0.5,
    };

    noteService.create(newNoteObject).then((data) => {
      setNotes(notes.concat(data));
      setNoteField("Enter a new note here...");
    });
  };

  return (
    <Togglable buttonLabel="Create Note" ref={noteFormRef}>
      <NoteForm
        addNote={addNote}
        noteField={noteField}
        handleInputChange={handleInputChange}
      />
    </Togglable>
  );
};

export default NoteFormContainer;
```

As with `LoginContainer `, `NoteFormContainer` takes in state through its props since this state needs to be shared between components (`notes` piece of state). When we use this component, we would pass in the props as follows:

```jsx
<NoteFormContainer notes={notes} setNotes={setNotes} />
```

Finally, let's create a component for the displaying of notes and the toggling of `flagged` for particular notes:

**/Frontend/src/components/NoteContainer.js**

```jsx
import React, { useState } from "react";
import noteService from "../services/notes";
import Note from "./Note";

const NoteContainer = ({ notes, setNotes }) => {
  const [showAll, setShowAll] = useState(true);

  const toggleFlagged = (id) => {
    const referencedNote = notes.find((n) => n.id === id);
    const updatedNote = { ...referencedNote, flagged: !referencedNote.flagged };

    noteService.update(id, updatedNote).then((responseNote) => {
      setNotes(notes.map((note) => (note.id === id ? updatedNote : note)));
    });
  };

  const filteredNotes = showAll ? notes : notes.filter((note) => note.flagged);

  const noteList = () =>
    filteredNotes.map((note) => (
      <Note
        key={note.id}
        note={note}
        toggleFlagged={() => toggleFlagged(note.id)}
      />
    ));

  return (
    <>
      <div>
        <button onClick={() => setShowAll(!showAll)}>
          Show {showAll ? "Flagged" : "All"}
        </button>
      </div>

      <ul>{noteList()}</ul>
    </>
  );
};

export default NoteContainer;
```

As with `NoteFormContainer `, `NoteContainer` takes in the `notes` piece of state since it needs to be shared between components. When we use this component, we would pass in the props as follows:

```jsx
<NoteContainer notes={notes} setNotes={setNotes} />
```

With all of this, our `App` component is dramatically simplified and only contains the shared state and effects for grabbing the user's information and loading the notes:

**/Frontend/src/App.js**

```jsx
import React, { useState, useEffect } from "react";
import Notification from "./components/Notification";
import Separator from "./components/Separator";
import noteService from "./services/notes";
import LoginContainer from "./components/LoginContainer";
import NoteFormContainer from "./components/NoteFormContainer";
import NoteContainer from "./components/NoteContainer";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

const App = (props) => {
  const [notes, setNotes] = useState([]);
  const [notificationMessage, setNotificationMessage] = useState();
  const [user, setUser] = useState(null);

  const loadNotesEffect = () => {
    noteService
      .getAll()
      .then((initialNotes) => {
        setNotes(initialNotes);
      })
      .catch((error) => {
        setNotificationMessage(
          `${error}: Something went wrong while trying to retrieve data from the server.`
        );
      });
  };

  const signedInEffect = () => {
    const loggedInUserJSON = window.localStorage.getItem("noteAppUser");
    if (loggedInUserJSON) {
      const user = JSON.parse(loggedInUserJSON);
      setUser(user);
      noteService.setToken(user.token);
    }
  };

  useEffect(loadNotesEffect, []);
  useEffect(signedInEffect, []);

  const linkStyle = { padding: 10 };

  return {
    /*...*/
  };
};

export default App;
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
          <Link style={linkStyle} to="/">
            Home
          </Link>
          <Link style={linkStyle} to="/notes">
            Notes
          </Link>
          <Link style={linkStyle} to="/login">
            Login
          </Link>
          {user === null ? null : (
            <Link style={linkStyle} to="/create">
              Create
            </Link>
          )}
        </div>
        <Notification message={notificationMessage} />
        <div>
          <Route exact path="/" render={() => <h2>Home</h2>} />
          <Route
            path="/login"
            render={() => (
              <LoginContainer
                user={user}
                setUser={setUser}
                setNotificationMessage={setNotificationMessage}
              />
            )}
          />
          <Route
            path="/notes"
            render={() => <NoteContainer notes={notes} setNotes={setNotes} />}
          />
          <Route
            path="/create"
            render={() => (
              <NoteFormContainer notes={notes} setNotes={setNotes} />
            )}
          />
        </div>
      </div>
    </Router>
    <Separator />
  </div>
);

//...
```

Here, we establish three links that will always be displayed near the top of the page (links to `/`, `/login`, and `/notes`). We conditionally render the `/create` route since we don't want to display the page that allows a user to create a note when the user cannot do so. This outlines an advantage to the dynamic nature of React Router: if we wanted to apply the conditional logic to the `Route` component, the `/create` route would not "exist" when the user is not signed in as it is dynamically created only when the user is created:

```jsx
{
  user ? (
    <Route
      path="/create"
      render={() =>
        user ? (
          <NoteFormContainer notes={notes} setNotes={setNotes} />
        ) : (
          <Redirect to="/login" />
        )
      }
    />
  ) : null;
}
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
        <Link style={linkStyle} to="/">
          Home
        </Link>
        <Link style={linkStyle} to="/notes">
          Notes
        </Link>
        <Link style={linkStyle} to="/login">
          Login
        </Link>
        {user === null ? null : (
          <Link style={linkStyle} to="/create">
            Create
          </Link>
        )}
      </div>
      <Notification message={notificationMessage} />
      <div>
        <Route exact path="/" render={() => <h2>Home</h2>} />
        <Route
          path="/login"
          render={() => (
            <LoginContainer
              user={user}
              setUser={setUser}
              setNotificationMessage={setNotificationMessage}
            />
          )}
        />
        <Route
          exact
          path="/notes"
          render={() => <NoteContainer notes={notes} setNotes={setNotes} />}
        />
        <Route
          path="/notes/:id"
          render={({ match }) => (
            <Note notes={getNoteById(match.params.id)} setNotes={setNotes} />
          )}
        />
        <Route
          path="/create"
          render={() => <NoteFormContainer notes={notes} setNotes={setNotes} />}
        />
      </div>
    </div>
  </Router>
);
```

Notice how we had to specify `exact` for the `/notes` route since the start of the route would also apply to the `/notes/:id` route that we want to target.

Here, similar to how parameters are defined in Express, we specify parameters in the form of `:PARAMETER`. In our `/notes/:id` route, we make use of parameters as well as a new `getNoteById()` helper:

```jsx
<Route
  path="/notes/:id"
  render={({ match }) => <Note notes={getNoteById(match.params.id)} />}
/>
```

You'll notice that we're able to grab our parameters from the `params` property of `match`. The `match` object simply contains information on how the `Route` _matched_ the URL.

Our helper is simply defined as:

```jsx
const getNoteById = (id) => notes.find((note) => note.id === id);
```

This helper gives us the information for the note with the id specified in our URL, allowing us to pass it in as a prop to the `Note` component.

Now, we can dive into our `Note` component and modify it so that the text content of the note links to the about route with the appropriate ID:

**/Frontend/src/components/Note.js**

```jsx
import React from "react";
import { Link } from "react-router-dom";

const Note = ({ note, toggleFlagged }) => {
  if (!note) {
    note = {
      flagged: false,
      content: "Note content not found",
    };
  }
  return (
    <li className="mainNote card">
      <input
        type="checkbox"
        alt="Toggle Flagged"
        checked={note.flagged}
        onChange={toggleFlagged}
      />
      <Link to={`/notes/${note.id}`}>{note.content}</Link>
    </li>
  );
};

export default Note;
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
<Route
  path="/create"
  render={() =>
    user ? (
      <NoteFormContainer notes={notes} setNotes={setNotes} />
    ) : (
      <Redirect to="/login" />
    )
  }
/>
```

If we are not signed in and attempt to visit `/create`, we will be taken to `/login`.

Looking at the `App` component, you can see the various components that we had used to accomplish our navigation:

**/Frontend/src/App.js**

```jsx
import React, { useState, useEffect } from "react";
import Notification from "./components/Notification";
import Separator from "./components/Separator";
import noteService from "./services/notes";
import LoginContainer from "./components/LoginContainer";
import NoteFormContainer from "./components/NoteFormContainer";
import Note from "./components/Note";
import NoteContainer from "./components/NoteContainer";
import {
  BrowserRouter as Router,
  Route,
  Link,
  Redirect,
} from "react-router-dom";

const App = (props) => {
  const [notes, setNotes] = useState([]);
  const [notificationMessage, setNotificationMessage] = useState();
  const [user, setUser] = useState(null);

  const loadNotesEffect = () => {
    noteService
      .getAll()
      .then((initialNotes) => {
        setNotes(initialNotes);
      })
      .catch((error) => {
        setNotificationMessage(
          `${error}: Something went wrong while trying to retrieve data from the server.`
        );
      });
  };

  const signedInEffect = () => {
    const loggedInUserJSON = window.localStorage.getItem("noteAppUser");
    if (loggedInUserJSON) {
      const user = JSON.parse(loggedInUserJSON);
      setUser(user);
      noteService.setToken(user.token);
    }
  };

  useEffect(loadNotesEffect, []);
  useEffect(signedInEffect, []);

  const getNoteById = (id) => notes.find((note) => note.id === id);

  const linkStyle = { padding: 10 };

  return (
    <div>
      <h1>Notes</h1>
      <Router>
        <div>
          <div>
            <Link style={linkStyle} to="/">
              Home
            </Link>
            <Link style={linkStyle} to="/notes">
              Notes
            </Link>
            <Link style={linkStyle} to="/login">
              Login
            </Link>
            {user === null ? null : (
              <Link style={linkStyle} to="/create">
                Create
              </Link>
            )}
          </div>
          <Notification message={notificationMessage} />
          <div>
            <Route exact path="/" render={() => <h2>Home</h2>} />
            <Route
              path="/login"
              render={() => (
                <LoginContainer
                  user={user}
                  setUser={setUser}
                  setNotificationMessage={setNotificationMessage}
                />
              )}
            />
            <Route
              exact
              path="/notes"
              render={() => <NoteContainer notes={notes} setNotes={setNotes} />}
            />
            <Route
              path="/notes/:id"
              render={({ match }) => (
                <Note note={getNoteById(match.params.id)} />
              )}
            />
            <Route
              path="/create"
              render={() =>
                user ? (
                  <NoteFormContainer notes={notes} setNotes={setNotes} />
                ) : (
                  <Redirect to="/login" />
                )
              }
            />
          </div>
        </div>
      </Router>
      <Separator />
    </div>
  );
};

export default App;
```

You'll also notice that the "Notes" header is rendered regardless of our route. This is because it is found outside of the `Router` and any `Route`s and will not be conditionally rendered.

There are plenty of features in React Router, and when you get into more complex applications, these features may come into much use. It is recommended to read up on the latest React Router documentation on _reacttraining.com_.

## Passport.js and OAuth

Previously, we talked about using JSON Web Tokens (JWT) for authentication and implemented it manually in our application using `bcrypt` and `jsonwebtoken`. We'll now transition into using Passport.js which gives us the ability to integrate with login strategies for Facebook, Google, LinkedIn, Twitch, Twitter, and many more.

If you just need username/password-like authentication, it is recommended to just implement the feature manually, but if you need to take advantage of OAuth, OpenID, etc. you'll usually need to take advantage of Passport.js.

Recall that we had implemented token-based authentication. As a refresher and to draw another difference, the three main options for authentication are:

- **Session-Based Authentication:** Associates a "session" with the client through storing the user state in the server's memory. When the user logs in, the session ID is stored in a cookie in the user's browser. This ID is used to identify the user and is used to be compared to the stored session data in the server's memory in order to authenticate.
- **Cookie-Based Authentication:** Identifier for the user is stored in a cookie (usually a token) and is used to automatically identify API requests. This is easier to deal with, but it comes with some security flaws as cookies are automatically included in all suitable requests to the host.
- **Token-Based Authentication:** Like cookie-based authentication (the definition that we explored earlier in this book), but you must manually include it with requests (in the `Authorization` header).

CodeRealm (https://github.com/alex996) provides a very good presentation highlighting the differences in authentication methods (skip to #Passport Modules if you wish to skip to right with working with token-based authentication):

- **Session Flow**: user submits email/password (credentials) -> server verifies credentials from the data it has stored in the database -> server creates a temporary user **session** -> server takes the session's ID and sends it back -> user sends the cookie with the ID stored with each request -> server validates the cookie against the session store and grants access if correct -> user logs out and the server destroys the session.
  - Stateful (sessions stored on the server whether it's a database or in memory)
- **Token Flow**: user submits email/password (credentials) -> server verifies credentials from the data it has stored in the database -> server creates a temporary **token** and embeds the user's data in it -> server sends the token back -> client stores the token -> client sends the token along with requests -> server validates the token and grants access if correct -> user logs out and the client deletes the token
- **Stateless JWT** (JWT token contains the session data encoded right in the token)
- **Stateful JWT** (JWT token just contains a reference to the session and the session data is still stored server-side)

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
require("dotenv").config();
const logger = require("./logger");

let isRequired = (name, ...values) => {
  values.forEach((value) => {
    if (value == null || value === "") {
      // Takes advantage of coercing to check if the value is undefined or null or blank
      logger.error(`${name} is missing a value!`);
    }
  });
};

// Express Configuration
let PORT = process.env.PORT;
isRequired("Express Configuration", PORT);

// Database Configuration
let MONGODB_URI = process.env.MONGODB_URI;
isRequired("MongoDB Configuration", MONGODB_URI);

// Google API Configuration
let GOOGLE_CLIENTID = process.env.GOOGLE_CLIENTID;
let GOOGLE_CLIENTSECRET = process.env.GOOGLE_CLIENTSECRET;
isRequired("Google API Configuration", GOOGLE_CLIENTID, GOOGLE_CLIENTSECRET);

// Facebook API Configuration
let FACEBOOK_CLIENTID = process.env.FACEBOOK_CLIENTID;
let FACEBOOK_CLIENTSECRET = process.env.FACEBOOK_CLIENTSECRET;
isRequired(
  "Facebook API Configuration",
  FACEBOOK_CLIENTID,
  FACEBOOK_CLIENTSECRET
);

if (process.env.NODE_ENV === "test") {
  MONGODB_URI = process.env.TEST_MONGODB_URI;
}

module.exports = {
  MONGODB_URI,
  PORT,
  GOOGLE_CLIENTID,
  GOOGLE_CLIENTSECRET,
  FACEBOOK_CLIENTID,
  FACEBOOK_CLIENTSECRET,
};
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
require("dotenv").config();
const convict = require("convict");

const config = convict({
  port: {
    doc: "Port that Express listens on",
    format: "port",
    default: 3000,
    env: "PORT",
  },
  database: {
    doc: "MongoDB URI for database connection",
    format: String,
    default: "",
    env: "MONGODB_URI",
  },
  authentication: {
    google: {
      clientId: {
        doc: "Google Auth Client ID",
        default: "",
        env: "GOOGLE_CLIENTID",
      },
      clientSecret: {
        doc: "Google Auth Client Secret",
        default: "",
        env: "GOOGLE_CLIENTSECRET",
      },
    },
    facebook: {
      clientId: {
        doc: "Facebook Auth client ID",
        default: "",
        env: "FACEBOOK_CLIENTID",
      },
      clientSecret: {
        doc: "Facebook Auth client secret",
        default: "",
        env: "FACEBOOK_CLIENTSECRET",
      },
    },
    token: {
      secret: {
        doc: "Secret key for JWT (signer)",
        default: "jDGWbgiUDHGIWOGE&G8DUGVHUI",
        env: "SECRET",
      },
      issuer: {
        doc: "JWT Issuer",
        default: "Test App",
      },
      audience: {
        doc: "JWT Audience",
        default: "Test App",
      },
    },
  },
});

// Perform validation
config.validate({ allowed: "strict" });

module.exports = config.getProperties();
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

const passport = require("passport");

//...

app.use(passport.initialize());

//...
```

The Passport documentation instructs us to add the `passport.session()` middleware, but since we have no need for persistent login sessions (as we're using stateless JWT), we can skip its binding.

### JWT Token Authentication Utility

We can now go ahead and set up JWT token authentication with Passport. For this strategy, we'll be using the `passport-jwt` module which allows us to look for an "Authorization" header with the JWT inside it and will decode the JWT so that we can manipulate the payload of the token however we wish. This module will also reject the request for us automatically if the token is invalid.

For the sake of organization, we'll be putting the configuration for the strategy in a `jwt` module within a new directory, **authentication**, found in the **utils** folder:

**/Backend/utils/authentication/jwt.js**

```jsx
const passport = require("passport");
const passportJwt = require("passport-jwt");
const config = require("../config");
const User = require("../../models/user");

const jwtOptions = {
  // Reads JWT from the HTTP Authorization header with the 'bearer' scheme
  jwtFromRequest: passportJwt.ExtractJwt.fromAuthHeaderWithScheme("Bearer"),
  // Secret used to sign the JWT
  secretOrKey: config.authentication.token.secret,
  // Issuer stored in JWT
  issuer: config.authentication.token.issuer,
  // Audience stored in JWT
  audience: config.authentication.token.audience,
};

passport.use(
  new passportJwt.Strategy(jwtOptions, async (payload, done) => {
    // In production, you should check to see if the token has not expired here
    console.log(payload.sub);
    const user = await User.findById(payload.sub);
    console.log(user);
    if (user) {
      return done(null, user, payload);
    }
    return done(null, false, { message: "User not found." });
  })
);
```

Here, we simply configure the JWT decoder with our knowledge of the secret, issuer, and audience. We also configure the JWT decoder to get the JWT from the Authorization header as a bearer token. If the issuer or audience does not match with what we have in our configuration, the authentication will fail.

When our token is decoded by `passport-jwt`, the token data is passed as a `payload` to the callback that we specify as the second argument to `passportJwt.strategy()`.

We then use the information from the token via the `payload` object to find the associated user from which the request was sent. If the user is found, we provide it to Passport who will then make it available to any following middleware/controllers under the `user` property of `req` (`req.user`).

In a production app, we should add the ability to revoke tokens and check if a token is revoked before granting access. This would require a JWT ID (explained below) to be issued.

### JWT Token Generation Utility

Now that we had implemented the ability to decode our JWTs, let's create a dedicated function for generating an authentication token for a given user (which will decode to reveal the user ID in its `subject` to which we can use the user ID to fetch the user for more information).

**/Backend/utils/authentication/token.js**

```jsx
const jwt = require("jsonwebtoken");
const config = require("../config");

// Generate an Access Token for the given User ID
function generateAccessJWT(userId) {
  // Time until token expires
  const expiresIn = "1 hour";
  // Issuer of token
  const issuer = config.authentication.token.issuer;
  // Intended audience/service for token
  const audience = config.authentication.token.audience;
  // Signing key for token
  const secret = config.authentication.token.secret;

  const token = jwt.sign({}, secret, {
    expiresIn: expiresIn,
    audience: audience,
    issuer: issuer,
    subject: userId.toString(),
  });

  return token;
}

module.exports = {
  generateAccessJWT,
};
```

Here, `generateAccessJWT()` takes in a user's ID and creates a JWT that will allow them to authenticate as that user. Notice how this token does not contain any other information in its payload where we previously were adding the user's username and id (instead we take the ID from the subject instead of other payload properties):

```jsx
const userForToken = {
  username: user.username,
  id: user._id,
};

const token = jwt.sign(userForToken, process.env.SECRET);
```

We set the `subject` option that `jwt.sign()` takes in its third parameter to the userID so that we can simply check the token's payload for the `sub` property (subject is stored within `sub` in the payload of a token) for our user's ID.

Within a token's payload, there are numerous _registered claims_ which provide a set of useful, interoperable claims such as `iss` (issuer), `exp` (expiration time), `sub` (subject), and `aud` (audience).

See "Payload" of https://jwt.io/introduction/ for more information.

With our above generation of a token, we do not implement the generation or specification of a JWT ID. This means that our tokens do not have a unique identifier to which we could use to check if the token had been revoked. Expiration of the token will still work as it depends on time, but if we programmatically or manually revoke a token, we must have specified a JWT ID so that we could create a store of revoked IDs. We would then check the tokens that come with requests to see if they are among the tokens in the store. JWT IDs usually come in the form of UUIDs to which we can randomly assign the IDs without worrying about collisions (since the probability is so low, think one in one trillion trillion trillion trillion).

### Social Login Provider Strategies

With the ability to generate and decode tokens completed, we now just need to create controllers that will actually handle the initiation of our authentication process.

When using social login providers, the user is redirected to a social login provider to which, upon success, they are redirected back to us where we can complete the process.

#### Google Authentication

Let's create a `google` module that will configure Passport with `passport-google-oauth` and add a new strategy to which we create a new user (if one doesn't already exist):

**/Backend/utils/authentication/google.js**

```jsx
const passport = require("passport");
const passportGoogle = require("passport-google-oauth");
const config = require("../config");
const User = require("../../models/user");

const passportConfig = {
  clientID: config.authentication.google.clientId,
  clientSecret: config.authentication.google.clientSecret,
  callbackURL: "http://localhost:3000/api/authentication/google/redirect",
};

if (passportConfig.clientID) {
  passport.use(
    new passportGoogle.OAuth2Strategy(passportConfig, function (
      request,
      accessToken,
      refreshToken,
      profile,
      done
    ) {
      // Creates user if this user does not already have an account with this googleId
      User.findOrCreate(
        { name: profile.name.givenName, providerId: profile.id },
        function (err, user) {
          return done(err, user);
        }
      );
    })
  );
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
const authRouter = require("express").Router();
const passport = require("passport");
const token = require("../utils/authentication/token");
require("../utils/authentication/jwt");
require("../utils/authentication/google");
require("../utils/authentication/facebook");

function generateUserJWTAndRedirect(req, res) {
  const accessJWT = token.generateAccessJWT(req.user.id);
  res.status(200).redirect(`http://localhost:3001/login?token=${accessJWT}`);
}

// Google Routes
authRouter.get(
  "/google/start",
  passport.authenticate("google", {
    session: false,
    scope: ["openid", "profile", "email"],
  })
);
authRouter.get(
  "/google/redirect",
  passport.authenticate("google", { session: false }),
  generateUserJWTAndRedirect
);

module.exports = authRouter;
```

Here, we chain our controllers/middleware.

We'll need to bind our router to our Express application in **App.js** as follows:

**/Backend/App.js**

```js
//...

const notesRouter = require("./controllers/notes");
const usersRouter = require("./controllers/users");
const loginRouter = require("./controllers/login");
const authRouter = require("./controllers/auth");

//...

// Routers
app.use("/api/authentication", authRouter);
app.use("/api/notes", notesRouter);
app.use("/api/users", usersRouter);
app.use("/api/login", loginRouter);

//...
```

#### Frontend Changes

Now, let's delve into the frontend and add a simple "Sign In with Google" button that should trigger this entire authentication flow using Google OAuth:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

const authenticateWithProvider = (provider) => {
  window.location =
    "http://localhost:3000/api/authentication/" + provider + "/start";
};

const loginForm = () => {
  return (
    <>
      <Togglable buttonLabel="Sign Up With Email">
        <LoginForm
          handleLogin={handleLogin}
          handleUsernameChange={({ target }) => setUsername(target.value)}
          handlePasswordChange={({ target }) => setPassword(target.value)}
          username={username}
          password={password}
        />
      </Togglable>
      <button onClick={() => authenticateWithProvider("google")}>
        Sign In with Google
      </button>
    </>
  );
};

//...
```

Here, we simply create an event handler that opens a new window that loads the start URL of whatever provider we are using. While user authentication and creation is successful at this moment. We need to do a few more things to make our frontend work with our new authentication system. Right now, our browser is redirected to the OAuth provider, to which the OAuth provider redirects us to a callback URL on our backend server. In the controller for this URL, we take the information given to us by Google and generate a token that the user can use to authenticate. We then take this token and redirect the user to the address of the frontend with the token embedded in it as a URL parameter.

This means that instead of getting the token through an `axios` request like the following:

```jsx
const user = await loginService.login({
  username,
  password,
}); // Gives us username, name, and token in the user object
```

We have to fetch it from the URL:

```jsx
const params = new URL(document.location).searchParams;
const token = params.get("token");
```

Our backend route will redirect us to the login route of our frontend with the token as a URL parameter to which we need to access and set the appropriate tokens in our modules/add it to local storage. This presents a perfect use case for `useEffect()` as we need something to occur after the component has rendered as a side effect:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

import authenticatedState from "../helpers/authenticatedState";

const LoginContainer = () => {
  //...

  const tokenEffect = () => {
    const params = new URL(document.location).searchParams;
    const token = params.get("token");
    if (token) {
      setUser(authenticatedState(token));
    }

    // For production cases, you should remove the parameters from the URL so the user cannot accidentally copy and send their token when trying to share their link
  };

  useEffect(tokenEffect, []);
};
```

Here, our effect simply takes the token (if there is one) from the URL upon initial render of the page. We also set our user to the return value of `authenticatedState(token)`:

Since we are no longer receiving our `token`, `username`, and `name` all in one response when we login with passport (unlike logging in through our `/login` route), we must take this token that we receive and use it to fetch our user information. With this, we created our first _helper_ in our frontend that will go ahead and do everything for us regarding setting the tokens for our services, getting our `user` object, setting `localStorage`, setting state state, etc.

**/Frontend/src/helpers/authenticatedState.js**

```jsx
import noteService from "../services/notes";
import loginService from "../services/login";

const authenticatedState = async (token) => {
  noteService.setToken(token);
  loginService.setToken(token);
  let user = await loginService.profile();
  user = {
    token,
    ...user,
  };
  window.localStorage.setItem("noteAppUser", JSON.stringify(user));
  return user;
};

export default authenticatedState;
```

Here, you'll see that we take advantage of a method of `loginService`, `profile()` to which we'll add as the following:

**/Frontend/src/services/login.js**

```jsx
import axios from "axios";
const baseUrl = "/api/login";

let token = null;

const setToken = (userToken) => {
  token = `bearer ${userToken}`;
};

const login = async (credentials) => {
  const res = await axios.post(baseUrl, credentials);
  return res.data;
};

const profile = async () => {
  const config = {
    headers: { Authorization: token },
  };

  const res = await axios.get(`${baseUrl}/profile`, config);
  return res.data;
};

export default { setToken, login, profile };
```

This route does not yet exist in our backend, so we'll add the following to our `login` controller:

**/Backend/controllers/login.js**

```js
//...

const passport = require("passport");
require("../utils/authentication/jwt");

//...

loginRouter.get(
  "/profile",
  passport.authenticate("jwt", { session: false }),
  async (req, res) => {
    const user = req.user;
    const userForProfile = {
      username: user.username,
      id: user._id,
    };
    res.json(userForProfile);
  }
);

module.exports = loginRouter;
```

Here, since the goal is to simply provide an endpoint that returns the information that our frontend needs regarding the user, we can just return a JSON object containing the user information. Passport, upon authentication, makes the user object representing the user that is authenticated available under `req.user`. **We can "authenticate"/protect our controller by chaining the `passport.authenticate()` middleware before our actual controller.** This will ensure that the request is authenticated before we run any logic and will allow us to access `req.user`.

We'll also replace the manual logic for verifying a JWT with a simple call to `passport.authenticate()` before the controller for creating a note:

**/Backend/controllers/notes.js**

```jsx
//...

const passport = require("passport");
require("../utils/authentication/jwt");

//...

notesRouter.post(
  "/",
  passport.authenticate("jwt", { session: false }),
  async (req, res, next) => {
    const body = req.body;

    try {
      const user = await User.findById(req.user.id);

      const newNote = new Note({
        content: body.content,
        flagged: body.flagged || false,
        date: new Date(),
        user: user._id,
      });

      const savedNote = await newNote.save();
      user.notes = user.notes.concat(savedNote._id); // Adds the ID of the note we have just saved to the user's notes
      await user.save();
      res.status(201).json(savedNote.toJSON());
    } catch (exception) {
      next(exception);
    }
  }
);

//...
```

We can also make a few updates to our `App` component so that we set the token for our login service as well:

**/Frontend/src/App.js**

```jsx
//...

const signedInEffect = () => {
  const loggedInUserJSON = window.localStorage.getItem("noteAppUser");
  if (loggedInUserJSON) {
    const storageUser = JSON.parse(loggedInUserJSON);
    console.log("user");
    setUser(storageUser);
    console.log(storageUser);
    noteService.setToken(storageUser.token);
    loginService.setToken(storageUser.token);
  }
};

//...
```

With this, our app should work as it did before (except for username and password authentication which we'll add in a moment). We could make use of our `authenticatedState` helper here, but we don't need to re-request for our profile information as `authenticatedState` does since we have no way of changing this information currently. Additionally, with this project lacking Redux, the organization of our project will not be optimal.

#### Facebook Authentication

We can now easily add Facebook authentication to our app by creating a `facebook` auth module (that is almost exactly like our `google` auth module), the appropriate controllers (again, very similar to the Google setup), and a button in our frontend that makes the initial request.

Let's create the `facebook` strategy as follows:

**/Backend/utils/authentication/facebook.js**

```js
const passport = require("passport");
const passportFacebook = require("passport-facebook");
const config = require("../config");
const User = require("../../models/user");

const passportConfig = {
  clientID: config.authentication.facebook.clientId,
  clientSecret: config.authentication.facebook.clientSecret,
  callbackURL: "http://localhost:3000/api/authentication/facebook/redirect",
};

if (passportConfig.clientID) {
  passport.use(
    new passportFacebook.Strategy(passportConfig, function (
      request,
      accessToken,
      refreshToken,
      profile,
      done
    ) {
      // Creates user if this user does not already have an account with this facebookId
      User.findOrCreate(
        { name: profile.displayName, providerId: profile.id },
        function (err, user) {
          return done(err, user);
        }
      );
    })
  );
}
```

We can then add the following routes to our `auth` controller:

**/Backend/controllers/auth.js**

```js
const authRouter = require("express").Router();
const passport = require("passport");
const token = require("../utils/authentication/token");
require("../utils/authentication/jwt");
require("../utils/authentication/google");
require("../utils/authentication/facebook");

function generateUserJWTAndRedirect(req, res) {
  const accessJWT = token.generateAccessJWT(req.user.id);
  res.status(200).redirect(`http://localhost:3001/login?token=${accessJWT}`);
}

// Google Routes
authRouter.get(
  "/google/start",
  passport.authenticate("google", {
    session: false,
    scope: ["openid", "profile", "email"],
  })
);
authRouter.get(
  "/google/redirect",
  passport.authenticate("google", { session: false }),
  generateUserJWTAndRedirect
);

// Facebook Routes
authRouter.get(
  "/facebook/start",
  passport.authenticate("facebook", { session: false })
);
authRouter.get(
  "/facebook/redirect",
  passport.authenticate("facebook", { session: false }),
  generateUserJWTAndRedirect
);

module.exports = authRouter;
```

Finally, let's just add a button in the frontend that makes the first request for Facebook authentication:

```jsx
//...

const loginForm = () => {
  return (
    <>
      <Togglable buttonLabel="Sign Up With Email">
        <LoginForm
          handleLogin={handleLogin}
          handleUsernameChange={({ target }) => setUsername(target.value)}
          handlePasswordChange={({ target }) => setPassword(target.value)}
          username={username}
          password={password}
        />
      </Togglable>
      <button onClick={() => authenticateWithProvider("google")}>
        Sign In with Google
      </button>
      <button onClick={() => authenticateWithProvider("facebook")}>
        Sign In with Facebook
      </button>
    </>
  );
};

//...
```

### Fixing Username/Password Authentication

We can fix username/password authentication with the help of `passport-local`. We'll create a new authentication configuration module as follows:

**/Backend/utils/authentication/local.js**

```jsx
const passport = require("passport");
const LocalStrategy = require("passport-local").Strategy;
const User = require("../../models/user");
const bcrypt = require("bcrypt");

passport.use(
  new LocalStrategy(function (username, password, done) {
    // Find user with matching username
    User.findOne({ username: username }, async function (err, user) {
      if (err) {
        return done(err);
      }
      if (!user) {
        return done(null, false, { message: "Username not found." });
      }

      // Checks to see if hashed password matches hash of provided password
      const passwordCorrect =
        user === null
          ? false
          : await bcrypt.compare(password, user.passwordHash);

      if (!passwordCorrect) {
        return done(null, false, { message: "Incorrect password." });
      }
      return done(null, user);
    });
  })
);
```

Here, you can see how we're pretty much left to ourselves in terms of describing what should authenticate and what shouldn't. We even describe how to check passwords. Passport does not implement signup methods as it is just an authorization library, so we are given the freedom with regard to how we create users as well. For now, our previous logic found in the `users` controller will work.

Now, we just have to make use of our new `LocalStrategy` in our `auth` controller that will authenticate us and then generate a token that we can use for further authentication around our apps:

```jsx
const authRouter = require("express").Router();
const passport = require("passport");
const token = require("../utils/authentication/token");
require("../utils/authentication/jwt");
require("../utils/authentication/google");
require("../utils/authentication/facebook");
require("../utils/authentication/local"); // Local strategy import

function generateUserJWTAndRedirect(req, res) {
  const accessJWT = token.generateAccessJWT(req.user.id);
  res.status(200).redirect(`http://localhost:3001/login?token=${accessJWT}`);
}

// Google Routes
authRouter.get(
  "/google/start",
  passport.authenticate("google", {
    session: false,
    scope: ["openid", "profile", "email"],
  })
);
authRouter.get(
  "/google/redirect",
  passport.authenticate("google", { session: false }),
  generateUserJWTAndRedirect
);

// Facebook Routes
authRouter.get(
  "/facebook/start",
  passport.authenticate("facebook", { session: false })
);
authRouter.get(
  "/facebook/redirect",
  passport.authenticate("facebook", { session: false }),
  generateUserJWTAndRedirect
);

// Local Authentication Routes
authRouter.get(
  "/local/start",
  passport.authenticate("local", { session: false }),
  generateUserJWTAndRedirect
);

module.exports = authRouter;
```

Here, we don't need a redirect route that we then authenticate with as we can simply authenticate upon our initial request (as we have all the information that we need), generate a token, and redirect the user.

Finally, we just need to change our login form to actually submit a request to the `/api/authentication/local/start` route:

**/Frontend/src/components/LoginContainer.js**

```jsx
//...

const loginForm = () => {
  return (
    <>
      <Togglable buttonLabel="Sign Up With Email">
        <LoginForm
          loginAddress={"http://localhost:3000/api/authentication/local/start"}
          handleUsernameChange={({ target }) => setUsername(target.value)}
          handlePasswordChange={({ target }) => setPassword(target.value)}
          username={username}
          password={password}
        />
      </Togglable>
      <button onClick={() => authenticateWithProvider("google")}>
        Sign In with Google
      </button>
      <button onClick={() => authenticateWithProvider("facebook")}>
        Sign In with Facebook
      </button>
    </>
  );
};

//...
```

Here, we pass in a new prop `loginAddress` which replaced our `handleLogin` handler. Additionally, we removed the extraneous imports of our application.

Now, we can just update the JSX for our login form to make a POST request to the `loginAddress` which triggers the local authentication process:

**/Frontend/src/components/LoginForm.js**

```jsx
import React from "react";
import PropTypes from "proptypes";

const LoginForm = ({
  loginAddress,
  handleUsernameChange,
  handlePasswordChange,
  username,
  password,
}) => {
  return (
    <form action={loginAddress} method={"post"}>
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
  );
};

LoginForm.propTypes = {
  loginAddress: PropTypes.string.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired,
};

export default LoginForm;
```

We also changed the names of our input fields to lowercase as that is what Passport is expecting.

Our app will now redirect instead of simply remaining on the application page and making HTTP requests. We could implement a solution that allows us to read information from a request or open another window for the sign in process to which we send the information back to our main window, but we'd like to keep the logic surrounding all three authentication methods as close as possible for now.## Module Bundling and Handling JS Files

It's time to learn about what's going on behind the scenes and uncover a certain degree of abstraction. We had bootstrapped our application by utilizing `create-react-app` which handled all of the configuration for us.

However, it was not always like this, in the past, creating a React app was considered daunting with many tools requiring configuration.

`create-react-app` uses "webpack" to bundle images, assets, scripts, etc. The image provided on the homepage of `webpack.js.org` provides a good overview of what "webpack" does:

![Webpack Image](https://i.imgur.com/Y0ilaHv.png)

> Image retrieved from https://webpack.js.org/ on 02/19/20

### Bundling Modules

Even though ES6 modules have been standard for quite a while now, most browsers cannot handle code that's been split up into modules. This means that if we just tried to serve an application with one initial JavaScript file as an entry point and tried importing other files, our application would simply not work.

To solve this issue, we have _bundlers_ that will _bundle_ our modules into single files that contain all of the necessary application code. For example, when we run `npm run build`, our bundler (webpack) will end up building the production version of the single JavaScript files. Our bundler will also manage other types of files for us (CSS, images, etc.). Bundling and building our application has previously resulted in the following files:

![Webpack](https://i.imgur.com/NAUdw9f.png)

Essentially, webpack takes all of our import statements, grabs the content from those modules, and dumps it all into "one" file. It continues to do so "recursively" for all imports.

Here, you'll see how our application is reduced into a few files by our bundler. We access these bundled JavaScript files in our **index.html** file to which our browser can now understand how to run our application:

```html
<link href="/static/css/main.ebd3d852.chunk.css" rel="stylesheet" />

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
const path = require("path");

const config = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "main.js",
  },
};
module.exports = config;
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
  console.log("Hello World!");
};
```

After this setup, we can finally bundle and build our 'vanilla' JavaScript application by running `npm run build`:

![webpack output](https://i.imgur.com/PGrotIZ.png)

In the output, we can see how our **index.js** file is picked up by webpack and is bundled into the **main.js** asset. Let's create our first imported module so that we can see how webpack handles modules:

**/src/App.js**

```js
const App = () => {
  return "Hello World!";
};

export default App;
```

We'll also make the following changes to our **index.js** file so that we're making use of importing modules:

**/src/index.js**

```js
import App from "./App";

const test = () => {
  App();
};

test();
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
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(<App />, document.getElementById("root"));
```

This is extremely similar to how our previous React applications were created with regard to their entry points.

**/src/App.js**

```jsx
import React from "react";

const App = () => <div>Hello World!</div>;

export default App;
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
const App = () => <div>Hello World!</div>;
```

Using JSX leads to an error as webpack, by default, only can bundle "vanilla" JavaScript. We can fix this by following the suggestion in the error message.

#### Webpack Loaders

webpack enables the use of loaders to allow us to bundle resources beyond "vanilla" JavaScript. We can even write our own loaders using Node.js. These loaders simply process our files before they are bundled, allowing webpack to proceed with bundling.

We can add a loader that transforms our JSX into "vanilla" JavaScript by adding the following to our webpack configuration:

**/webpack.config.js**

```js
const path = require("path");

const config = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "main.js",
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: "babel-loader",
        options: {
          presets: ["@babel/preset-react"],
        },
      },
    ],
  },
};
module.exports = config;
```

Here, we define a loader under the `module` property in its `rules` array.

In order to define a loader, we need to provider a `test` property, a `loader` property, and a `options` property:

- The `test` property tells the file endings that our loader should target. It takes in a regular expression and our files must match this condition.
- The `loader` property simply allows us to specify a loader to be used for the processing of files that match our `test`.
- The `options` property just allows us to configure our loader. Here we just set the our `babel-loader` loader's preset to the default preset for React.

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
const path = require("path");

const config = {
  entry: ["core-js", "./src/index.js"],
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "main.js",
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: "babel-loader",
        options: {
          presets: ["@babel/preset-react"],
        },
      },
    ],
  },
};
module.exports = config;
```

Here, we changed the value of `entry` to an array with `'core-js'` as its first value.

This is not a devDependency (`--save-dev`) as we need it to run before our source code for the actual application, even when its not in development.

If we serve our **index.html** file with a HTTP server and open it in our browser, we'll see the following result as expected:

![Expected Application](https://i.imgur.com/rW5iTus.png)

When we "convert" this JSX code into "vanilla" JavaScript code, we are _transpiling_ the code. **Transpilers are tools that read source code written in one programming language (or superset/subset) and produce the equivalent in another language.**

In our configuration, we utilize `babel` which takes care of transpiling our code. Now, most browsers do not support the ES6 and ES7 features that we make use of (we did not use polyfills for every single feature). We can solve this by using a preset that will transpile our code written in newer JavaScript standards to code that operates in older standards. This allows us to use newer features without losing any compatibility.

In order to do this, we'll simply add a preset that will allow us to use the latest JavaScript without micromanaging polyfills and syntax transforms; this also comes with the benefit of making our JavaScript bundles smaller in some cases.

Install the preset by running the following command:

```bash
npm i --save-dev @babel/preset-env
```

As this package is only used by `babel` when transpiling our application, we only need to install it as a devDependency.

Now, we can just add it to our `presets` property in our loader options for `babel-loader`:

```js
const path = require("path");

const config = {
  entry: ["core-js", "./src/index.js"],
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "main.js",
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: "babel-loader",
        options: {
          presets: ["@babel/preset-env", "@babel/preset-react"], // New Preset
        },
      },
    ],
  },
};
module.exports = config;
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

import "promise-polyfill/src/polyfill";

//...
```

Here, if our window object (global) does not have a `Promise` property, this `import` will add a global Promise object (including support for Node which does not have a `window` object).

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
import React from "react";
import "./index.css";

const App = () => <div className="test">Hello World!</div>;

export default App;
```

This yields the following result when served by a HTTP server:

![Styles with Webpack](https://i.imgur.com/ZXbD47w.png)

Hopefully this clears up any confusion regarding how our app actually receives its styles (we import the styles to which they are made available in the DOM).

### Development Tools for webpack

Previously, when working with `create-react-app`, our app would automatically recompile whenever its code changed. This is not the case in our current configuration.

There are three options with webpack that will automatically compile our code:

- webpack Watch Mode
- `webpack-dev-server`
- `webpack-dev-middleware`

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
import React, { useState } from "react";
import "./index.css";

const App = () => {
  const [testArray, setTestArray] = useState(); // Defined with no value

  const handleArrayAdd = () => {
    setTestArray(testArray.concat("Added!")); // Tries to concat 'Added!' to an object that isn't an array
  };

  return (
    <>
      {testArray}
      <button onClick={handleArrayAdd} className="test">
        Add value to array
      </button>
    </>
  );
};

export default App;
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

With larger projects, it may become important to monitor our package size. _BundlePhobia_ (bundlephobia.com) is a great tool for monitoring the cost of adding a package to our application.

We can also upload our **package.json** file to BundlePhobia to get some information on the cost of each package.

Upon viewing a package, we can view the _minified_ size of the package (+ gzipped) as well the estimated times it would take to download these packages:

![Package](https://i.imgur.com/WTLXuWu.png)

You may not have heard of the term, _minification_, just yet. It simply refers to the compression of our files, optimizing the formatting by removing whitespace, shortening variable names, replacing verbose functions with more concise functions, etc.

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

As we mentioned earlier, `create-react-app` uses webpack for its bundling. Its configuration is abstracted behind-the-scenes and is configured for us. If we wish to change the default configuration, we must _eject_ from the tool. This essentially makes `create-react-app` a boilerplate generator to which we are given all of the configuration files needed for the project. We are essentially on our own after ejecting.

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
import React from "react";

class App extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <div>
        <h1>Note App</h1>
      </div>
    );
  }
}

export default App;
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
import React from "react";
import "promise-polyfill/src/polyfill";

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      notes: [],
    };
  }

  render() {
    if (this.state.notes.length === 0) {
      return <p>There are no notes</p>;
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>{this.state.notes.map((n) => n.content)}</div>
        <button>Remove Notes</button>
      </div>
    );
  }
}

export default App;
```

Notice how we keep our `notes` piece of state as a property of the state object (`this.state`). `this.state.notes` is like the `notes` constant given to us by `const [notes, setNotes] = useState([])`. Within our component, we must also refer to our pieces of state as properties of `this.state`.

Here, we simply return a paragraph containing "There are no notes" if the length of our `notes` piece of state is 0. If this condition is not satisfied (meaning that there are notes), we simply return the application, rendering a heading, the notes, and a button. This is all done in the `render()` method of the class.

This yields the following result:

![no notes](https://i.imgur.com/0YfB20h.png)

This is rendered as we do not have any notes in our state. Let's go about fetching some notes to demonstrate this component.

Previously, we had used the `useEffect()` hook in order to perform side effects in our function components. Among these side effects were network requests to fetch data. We can achieve the same result with class components through the use of _lifecycle methods_.

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
    this.state = { someValue: "test" };
  }
  static getDerivedStateFromProps(props, state) {
    return { someValue: props.valuePassedInFromProps };
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
import React from "react";
import "promise-polyfill/src/polyfill";
import axios from "axios";

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      notes: [],
    };
  }

  componentDidMount() {
    axios.get("http://localhost:3001/notes").then((res) => {
      this.setState({ notes: res.data });
    });
  }

  render() {
    if (this.state.notes.length === 0) {
      return <p>There are no notes</p>;
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>{this.state.notes.map((n) => n.content)}</div>
        <button>Remove Notes</button>
      </div>
    );
  }
}

export default App;
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
    console.log(this);
    this.setState({
      notes: [],
    });
  }

  render() {
    if (this.state.notes.length === 0) {
      return <p>There are no notes</p>;
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>{this.state.notes.map((n) => n.content)}</div>
        <button onClick={this.handleClick}>Remove Notes</button>
      </div>
    );
  }
}

export default App;
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
import React from "react";
import "promise-polyfill/src/polyfill";
import axios from "axios";

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      notes: [],
    };

    this.handleClick = this.handleClick.bind(this);
  }

  componentDidMount() {
    axios.get("http://localhost:3001/notes").then((res) => {
      this.setState({ notes: res.data });
    });
  }

  handleClick() {
    this.setState({
      notes: [],
    });
  }

  render() {
    if (this.state.notes.length === 0) {
      return <p>There are no notes</p>;
    }

    return (
      <div>
        <h1>Notes App</h1>
        <div>{this.state.notes.map((n) => n.content)}</div>
        <button onClick={this.handleClick}>Remove Notes</button>
      </div>
    );
  }
}

export default App;
```

The "same" component written as a functional component looks like the following:

```jsx
import React, { useState, useEffect } from "react";
import "promise-polyfill/src/polyfill";
import axios from "axios";

const App = () => {
  const [notes, setNotes] = useState([]);

  useEffect(() => {
    axios.get("http://localhost:3001/notes").then((res) => {
      setNotes(res.data);
    });
  }, []);

  const handleClick = () => {
    setNotes([]);
  };

  if (notes.length === 0) {
    return <p>There are no notes</p>;
  }

  return (
    <div>
      <h1>Notes App</h1>
      <div>{notes.map((n) => n.content)}</div>
      <button onClick={handleClick}>Remove Notes</button>
    </div>
  );
};

export default App;
```

Here, you can somewhat see the differences between functional and class components. The advantages of functional components become much more realized with larger components and projects.

`useEffect()` often becomes much more effective for dealing with side effects rather than the many lifecycle methods found in class components. We also do not have to deal with referencing `this` or being restricted to one object as our component's state.

One potential use for class components that still has not been resolved is the use of class components as _error boundaries_, these components wrap other components and display a fallback UI upon catching an error in its child components.

Error boundaries depend on the `componentDidCatch(error, info)` lifecycle method, to which these error boundaries can only be class components (although they may still wrap functional components).

Functional components can try to display fallback UIs and catch errors using the `try`/`catch` syntax, but this only works for imperative JavaScript code:

```jsx
try {
  renderComponent();
} catch (error) {
  displayFallbackUI();
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
    <link
      rel="stylesheet"
      href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap"
    />
    <!-- Font Link -->

    <!-- ... -->
  </head>
</html>
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
import { createMuiTheme } from "@material-ui/core";
import { orange } from "@material-ui/core/colors";

const baseTheme = {
  status: {
    danger: orange[500],
  },
};

const theme = createMuiTheme(baseTheme);

export default theme;
```

Here, we leverages `createMuiTheme` in order to create a theme object that we'll pass to a theme provider which will apply our styles specified in `baseTheme` to our application.

We'll then use `ThemeProvider` to inject our `theme` object into our application:

**/Frontend/src/index.js**

```jsx
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import { ThemeProvider } from "@material-ui/core";
import theme from "./theme";
import "./index.css";

ReactDOM.render(
  <ThemeProvider theme={theme}>
    <App />
  </ThemeProvider>,
  document.getElementById("root")
);
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

Here, we use `makeStyles()` in order to access our theme object and generate a hook to which we can call within our component to get `classes`. From this, we can access the CSS properties stored in the properties of the object returned by the function passed to `makeStyles()`. We access the class through providing the reference to `className`.

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
import React from "react";
import { Link } from "react-router-dom";
import {
  Card,
  CardContent,
  Checkbox,
  Typography,
  makeStyles,
} from "@material-ui/core";

const useStyles = makeStyles({
  root: {
    marginBottom: 10,
  },
});

const Note = ({ note, toggleFlagged }) => {
  const classes = useStyles();

  if (!note) {
    note = {
      flagged: false,
      content: "Note content not found",
    };
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
  );
};

export default Note;
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
import React from "react";
import PropTypes from "proptypes";
import { Input, Button, Typography } from "@material-ui/core";

const LoginForm = ({
  loginAddress,
  handleUsernameChange,
  handlePasswordChange,
  username,
  password,
}) => {
  return (
    <form action={loginAddress} method={"post"}>
      <div>
        <Typography>Username</Typography>
        <Input
          type="text"
          value={username}
          name="Username"
          onChange={handleUsernameChange}
        />
      </div>
      <div>
        <Typography>Password</Typography>
        <Input
          type="password"
          value={password}
          name="Password"
          onChange={handlePasswordChange}
        />
      </div>
      <Button variant="contained" type="submit" color="primary">
        Login
      </Button>
    </form>
  );
};

LoginForm.propTypes = {
  loginAddress: PropTypes.string.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired,
};

export default LoginForm;
```

Here, we just replaced our `input` elements with `Input` elements from Material UI. We also wrapped our text in `Typography` components and replaced our standard `button` with a `Button` component from Material UI with the extra props `variant` and `color`.

Our result is much better looking than before (but not nearly good enough for a production application):

![Better example](https://i.imgur.com/MQ2eeUw.png)

In a production application, we would have to take much more consideration with regard to the color palette of our application, the spacing and distribution of components/elements, etc. With this, we have explored the basics of using a UI framework. There are plenty of other components that you can take advantage of in your future applications, but we have covered just about every component that we would be able to use in our application here.

With this, we'll explore one other method of styling React components. This method is often considered the best way to style components (to which the many advantages of styled components are inspired and found in Material UI's theming system).

#### Styled Components

Styled components allow us to create reusable components that we can use throughout our application with styling information baked in. For example, if we wanted to create a styled regular `button` element, we would do the following:

```jsx
import styled from "styled-components";

const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;
`;
```

Here, we create a styled version of `button` using the style information provided between the backticks. This syntax may look awkward with it being a relatively niche feature introduced in ES6. Our styles here were defined using tagged template literals (enclosed by the backticks ``).

Tagged template literals take the template literal (the text in the backticks) and call the `tag` expression (the function, expression, etc. preceding the template literal) with the value of the template literal.

Let's just render this button to see its result:

```jsx
const TestButton = () => <Button>This is a test of Styled Components!</Button>;
```

This yields the following result:

![Styled Components Test](https://i.imgur.com/4MHQqlj.png)

Material UI allows for it to be styled using `styled-components` to which we use the `styled()` method provided by `styled-components` in order to create new styled components.

For example, if we wanted to create a styled `Button` component (not a plain `button`) with the `Button` component coming from Material UI, we would do the following:

```jsx
import React from "react";
import styled from "styled-components";
import Button from "@material-ui/core/Button";

const StyledButton = styled(Button)`
  background-color: #6772e5;
  color: #fff;
  box-shadow: 0 4px 6px rgba(50, 50, 93, 0.11), 0 1px 3px rgba(0, 0, 0, 0.08);
  padding: 7px 14px;
  &:hover {
    background-color: #5469d4;
  }
`;
```

Here, we call the `styled()` method with the `Button` component that we want to style, to which we put the template literal containing the styling information to add after.

## Firebase

Throughout the course of this book, we've been creating our own backends to server our React apps. Sometimes, there is a need for bootstrapping a quick prototype without worrying about backend logic and infrastructure. Firebase is just a set of tools that Google offers to help developers build scalable applications using cloud infrastructure.

Among these tools include a scalable NoSQL database, a authentication suite, a machine learning toolkit, web app hosting, cloud storage, powerful analytics tools, messaging tools, and much more.

Firebase is increasingly becoming a substitute for an Express backend, with it being presented as an alternative in most cases. It is an extremely powerful tool for certain use cases and should be considered when creating new projects.

We'll first explore creating our notes application (complete with user authentication) using Firebase and then transition into looking at the power of Firebase, especially when using tools like React Native (to be discussed later).

### Firebase Configuration

In order to use Firebase, we'll need to sign up for Firebase at https://firebase.google.com using a Google account. Then, in the Firebase console (https://console.firebase.google.com), we can create a project by pressing "Add Project":

![Create Project Firebase](https://i.imgur.com/lm01So1.png)

We'll then be prompted to give a project name to which we can enter anything we want (Firebase will generate an ID for us depending on this name). There will also be a few steps to walk through afterwards, but none of them should affect anything that we're about to do.

Upon creating our project, we'll be presented with a dashboard that looks like this:

![Firebase Dashboard](https://i.imgur.com/wgyCo6F.png)

We'll want to press the `</>` web logo in order to add a web app to our FIrebase project:

![Web App](https://i.imgur.com/2qPDeKv.png)

Here, we'll be prompted to enter a nickname (does not matter). We'll also be prompted to enable Firebase Hosting. This is not required, so feel free to leave it unchecked. Upon the creation of our app, we'll be given the following code block containing the `firebaseConfig` variable which we'll need later:

![Firebase Config](https://i.imgur.com/cHaEyGN.png)

> The Firebase credentials will be invalidated upon this book's release. Please do not try to use these credentials in your application.

We won't need to add the SDK scripts into our HTML files since we'll be installing them through npm and importing them into our application.

### Using Firebase in React

We'll first use `create-react-app` to create an entirely new project:

```bash
npx create-react-app firebase-project
```

We'll also need to install `firebase`:

```bash
npm i firebase
```

We'll now get started with user creation and authentication. Firebase offers a drop in UI module that will handle all of the UI flows regarding authentication for us whether it be authentication with email/password, phone number, or OAuth/federated identity providers. This component will implement the best practices for authentication without us having to do much work.

We can install the React wrapper for FirebaseUI by running the following command:

```bash
npm i react-firebaseui
```

Now, let's create our `App` component:

**/src/App.js**

```jsx
import React from "react";
import StyledFirebaseAuth from "react-firebaseui/StyledFirebaseAuth";
import firebase from "firebase/app"; // Allows us to create app instances
import "firebase/auth"; // Gives us Firebase authentication support

// Firebase configuration provided to us by the app creation process
const firebaseConfig = {
  apiKey: "AIzaSyDJf0kn4MHA8mMgolqu1DLh3YqrjSAt57Q",
  authDomain: "notes-app-bfbc5.firebaseapp.com",
  databaseURL: "https://notes-app-bfbc5.firebaseio.com",
  projectId: "notes-app-bfbc5",
  storageBucket: "notes-app-bfbc5.appspot.com",
  messagingSenderId: "768088207404",
  appId: "1:768088207404:web:abd4b64d40dbab447e6189",
  measurementId: "G-NBKG2S4VX5",
};

// Initializes Firebase and creates an app instance
firebase.initializeApp(firebaseConfig);

// Configure FirebaseUI.
const uiConfig = {
  // Popup signin flow rather than redirect flow.
  signInFlow: "popup",
  // We will display Google and Facebook as auth providers. Also configures the component with the provider IDs.
  signInOptions: [
    firebase.auth.GoogleAuthProvider.PROVIDER_ID,
    firebase.auth.FacebookAuthProvider.PROVIDER_ID,
  ],
  callbacks: {
    // `signInSuccessWithAuthResult` is called with the user object when the user has successfully logged in
    signInSuccessWithAuthResult: () => false, // Empty function as we do not want to do anything just yet
  },
};

const App = () => {
  return (
    <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()} />
  );
};

export default App;
```

Read the comments in the code block to get a detailed explanation of what's going on in the component.

Here, we simply imported the required Firebase packages (the base app package and the Firebase auth package to give us authentication support), created the configuration constants `firebaseConfig` and `uiConfig`, initialized Firebase with `initializeApp(firebaseConfig)`, and rendered `<StyledFirebaseAuth />` with our `uiConfig` and `firebase.auth()` method passed into it.

`firebaseConfig` simply comes from the code block that was provided to us from Google earlier. `uiConfig` simply contains a bunch of options (explained above in the comments) that our component will take in.

There are plenty of other features and options that come with `react-firebaseui`, but this knowledge should be enough for now.

Our application should look like this now:

![Application](https://i.imgur.com/rOyUkY1.png)

Before we actually manage our state with regard to handling the creation/retrieval of the user, we have to learn more about the `useEffect()` hook as well as the concept of subscriptions.

### Subscriptions and `useEffect()` Clean Up

Subscriptions, conceptually, are a concept of reactive programming. Subscriptions are connections between a subscriber and a publisher. A publisher will simply create a subscription for every single subscriber that is subscribed to it, to which requests and data are sent over this subscription. The subscriber that had "subscribed" to the publisher will react to changes in data sent down the subscription.

We will have to employ this concept when working with Firebase, as we will have to "subscribe" to the authentication state of our user in order to perform actions. For example, we will subscribe to the state and perform actions on a success, failure, etc.

Previously, we had talked about lifecycle methods in React class components. One lifecycle method was `componentWillUnmount` where we could clean up subscriptions

With effects, we can clean up subscriptions by returning a function from our effect function. If our effect returns a function, React will run it when its time to clean it up:

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    API.subscribeToStatus(props.user.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      API.unsubscribeToStatus(props.user.id, handleStatusChange);
    };
  });

  //...
```

React will "clean up" an effect when the component unmounts (as the name for the lifecycle method points to).

Our usage of these concepts with Firebase is as follows:

```jsx
useEffect(() => {
  // Realtime authentication listener subscription
  const unsubscribe = onAuthStateChange(); // Returns the unsubscribe function
  // Unsubscribes from the listener when unmounting
  return () => unsubscribe();
}, []);
```

Here, we subscribe to the authentication state and deal with it in `onChange`. The `onAuthStateChanged()` function returns the unsubscribe function for the observer (subscriber). We then return a function that will call this unsubscribe function, to which React will call this function upon unmounting.

#### Managing User State Through Subscriptions

We'll now observe/subscribe the authentication state of our user to which we'll manage our state dependent on what our provider (Firebase) tells us:

**/src/App.js**

```jsx
import React, { useEffect } from "react";

//... EXISTING CONFIGURATION

const onAuthStateChange = () => {
  return firebase.auth().onAuthStateChanged((user) => {
    if (user) {
      console.log("The user is logged in.");
    } else {
      console.log("The user is not logged in.");
    }
  });
};

const App = () => {
  useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange(); // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe();
  }, []);

  return (
    <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()} />
  );
};

export default App;
```

Here, we define a function `onAuthStateChange()` which simply calls `firebase.auth().onAuthStateChanged()` internally. `firebase.auth().onAuthChanged()` takes in a callback function to which it'll pass in the `user` object upon a successful authentication. We take advantage of this fact to log to the console information about whether we have logged in or not.

`firebase.auth().onAuthStateChanged()` also returns the unsubscribe function that we need to call upon unmounting, so we'll return the value of this call in `onAuthStateChange()` so that the effect that we had described earlier can return it to be called upon unmounting.

#### Enabling Authentication Methods

Now, our initial setup for authentication is just about complete. If we try to log in using a provider right now, we'll receive the following error:

![Identity provider disabled](https://i.imgur.com/yeJtXOM.png)

This is simply because we had not enabled these identity providers in our Firebase dashboard just yet. We can visit the "Authentication" tab of the dashboard and toggle the Google sign-in method in just a few clicks.

![Authentication](https://i.imgur.com/EJerjV2.png)

> Make sure to select a project support email address

Facebook is almost as simple. As we did with Passport authentication, Firebase asks us to enter in a App ID and App secret as well as allow the redirect URL.

Now, we can attempt to sign into our application:

![Successful console](https://i.imgur.com/jGjgura.png)

Here, we see the console log that we are expecting upon logging in. The callback function that we had provided to `firebase.auth().onAuthStateChanged()` will be called every single time our user's authentication state changes (Firebase will notify our application).

Now, one change we might want to make is to store our user in our component's state and render a UI for the user when they're signed in. We'll eventually extend this UI to support creating/viewing notes using more of Firebase's features.

**/src/App.js**

```jsx
import React, { useEffect, useState } from "react";

//... EXISTING CONFIGURATION

const onAuthStateChange = (setUser) => {
  return firebase.auth().onAuthStateChanged((user) => {
    if (user) {
      console.log("The user is logged in.", user);
      setUser(user);
    } else {
      console.log("The user is not logged in.");
    }
  });
};

const App = () => {
  const [user, setUser] = useState();

  useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange(setUser); // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe();
  }, []);

  return (
    <>
      {JSON.stringify(user)}
      <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()} />
      <button onClick={() => firebase.auth().signOut()}>Sign-out</button>
    </>
  );
};

export default App;
```

Here, we made a few changes.
