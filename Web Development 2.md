Continued



## Resource Deletion (HTTP DELETE)

We've gone over adding, modifying, and requesting resources, but now let's go over the act of deleting a resource. We can delete an object by making a HTTP DELETE request to a given resource. We would handle this in Express with a route as follows

```jsx
app.delete('/notes/:id', (req, res) => {
  const id = Number(req.params.id)
  notes = notes.filter(note => note.id !== id) // Filters out the specified note from `notes`

  res.status(204).end()
})
```

Here, upon a HTTP DELETE request to the above path, we take in the `id` parameter and set `notes` equal to a new array with the note that matches `id` filtered out. We then return the request with the response code `204` which signifies 'No Content' (we picked this obscure status code as there is not really a designated status code for a DELETE request). We also return no data with the response as shown by the call to `end()` without any parameters.

### Exploring the HTTP Standard

When building REST APIs, we must consider two very important properties of HTTP methods that are defined in HTTP's specification. You'll find HTTP methods that we have not explored yet mentioned in the following section; do not worry as understanding over these methods is not important to understanding the concepts surrounding these properties.

**Safe**

The first important property to consider is that GET and HEAD HTTP requests should be **safe**. This means that our request should not lead to any "side effects" in our server. For example, our HTTP GET request should not change our database in any way as we are simply requesting data. Additionally, our HTTP GET request should only return data that already exists on the server. These rules also apply to the HTTP HEAD request (similar to HTTP GET except that it only returns the status code and headers).

**Idempotent**

All HTTP requests (other than HTTP POST) be **idempotent**. This means that they should cause the exact same side effects regardless of how many times a given request is sent. For example, if we use a HTTP PUT request to update/replace a specific resource, duplicating this request would not give any other side effects other than replacing the resource for the first time. This is just to safeguard against duplicate requests so that we have a *fault-tolerant API* that doesn't leave our system unstable if there were a duplicate request.

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
  next()
}
```

We have already discussed the `request` and `response` objects. The `next` function, which is a new concept, simply gives control and execution over to the next middleware that was bound (in order of the `use()` calls in the program that we will discuss next). The main takeaway for middleware functions is that they allow us to do things with the `request` and `response` objects before our `route` handlers are actually called, allowing all `route` handlers to take advantage of their functionality.

We can use middleware in our app by binding them through our Express app's `use()` method as follows

```jsx
app.use(exampleMiddleware)
```

where we use the above middleware function that we have defined.

For every request, before the execution of our route handlers, middleware functions are called in the order they were bound through the `use()` method of our Express server. For example, if we bound the following middleware in this order

```jsx
app.use(bodyParser.json())
app.use(exampleMiddleware)
```

the `bodyParser.json()` middleware function will be executed before `exampleMiddleware` where `bodyParser.json()` hands control over to `exampleMiddleware` through a call to `next()`. This order also allows us to use the `body` property of `request`/`req` in `exampleMiddleware` as it was executed beforehand and defines the property.

For most cases, we would define and bind our middleware before our routes (placing the `use()` calls before our route definitions), but there are a few cases where we can put our `use()` calls after our route definitions to achieve a desired behavior. 

One practical use would be if we wanted to call a middleware function if no route had handled the HTTP request. The middleware function might look something like the following

```jsx
const notFound = (req, res) => {
  res.status(404).send({ error: 'Your request did not match any endpoint.' })
}
```

This function simply sends a response with the status code of `404` (not found) and gives an error message that the request did not match any endpoint covered by our routes.

We then can put the following snippet after all of our routes

```jsx
app.use(notFound)
```

so that the `notFound` middleware function will be called if no route handled the HTTP request.

### body-parser

We can import `body-parser` and lay out our HTTP POST route as follows

**/index.js**

```js
const express = require('express')
const app = express()
const bodyParser = require('body-parser')

app.use(bodyParser.json()) // Imports bodyParser

//...

app.post('/notes', (req, res) => {
  const note = req.body
  console.log(note)

  res.json(note)
})

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

const generateNoteId = () => { // Helper function to generate ID #s
  const maxId = notes.length > 0
    ? Math.max(...notes.map(note => note.id))
    : 0
  return maxId + 1
}

app.post('/notes', (req, res) => {
  const body = req.body

  if (!body.content) {
    return res.status(400).json({ 
      error: 'No content specified in note body' 
    })
  }

  const newNote = {
    content: body.content,
    flagged: body.flagged || false,
    date: new Date(),
    id: generateNoteId(),
  }

  notes = notes.concat(newNote)

  res.json(newNote)
})

//...
```

Here, you'll notice a few things. First, we have defined a helper function that will generate the ID for our new note by simply taking the largest ID number in the current list and then returning that `maxId` + 1 to give us a new 'maximum'. With 'actual' applications, this method of dealing with IDs is not recommended, but it will serve us just fine for now. Taking a closer look at the body of code, you'll notice the usage of a few new functions

```jsx
const generateNoteId = () => { // Helper function to generate ID #s
  const maxId = notes.length > 0
    ? Math.max(...notes.map(note => note.id))
    : 0
  return maxId + 1
}
```

Here, we call the `map` method of our `notes` array to create a new array with just the `id`s of our array of notes. We achieve this by passing in a function to `notes.map()` that takes in the `note` object and just returns its `id` property. Then, we use `Math.max()` upon the array that we have destructured to give us the maximum `id` value currently in our array. We destructured our array as `Math.max()` takes in many numbers as its parameters rather than an array. Finally, we return the value of `maxId` (the maximum `id` value that we have found) + 1. We also used the ternary operator, `expression ? ifTrue : ifFalse`, to give us a default value of `0` if `notes.length` is not greater than 0.

You'll also notice that we handle having a missing value for `body.content` by sending a response with the status code `400` which stands for 'Bad Request' as well as sending the error message `'content missing'`:

```jsx
if (!body.content) {
  return res.status(400).json({ 
    error: 'No content specified in note body' 
  })
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
  }
```

Here, you'll notice that we take in the `content` property of the `body` constant which was created via a reference to `req.body`. We know for certain that there is a `content` property via the logical check that we performed earlier in the handler.

You might have wondered, what do we do if another value was missing such as a the `flagged` property value or the `date` property value? We simply provide default values through the use of the logical OR operator, `||`. For example, in the following code snippet, we provide the `flagged` property with a default value of `false` if there is no value for `body.flagged` (it was not specified in the JSON of the body of the request)

```jsx
flagged: body.flagged || false,
```

The logic of this statement depends on the fact that `undefined` has a falsy value. Thus, if `body.flagged` is `undefined` it will evaluate to false and the value on the right, `false`, will be assigned to the `flagged` property. It may be helpful to think of the OR logical operator as a *selection* operator with the following rules:

* If the 1st expression (left side) evaluates to true, it will always be the one *selected* and 'returned' from the comparison
* If the 1st expression (left side) evaluates to false, the 2nd expression (right side) will be the value returned 'returned' from the comparison **even if the 2nd expression is falsy** (like in the above example)

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
const cors = require('cors')

app.use(cors())
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
const PORT = process.env.PORT || 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
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

Finally, we can run `git push heroku master` to 'push' our commit up to the remote Heroku server. The `git push` command is used to 'push' (transfer commits) from our local repository to a remote repository (in this case, our remote repository is heroku). Thus, in this command, the first argument is the remote repository/destination, `heroku`,  with the second argument being the name of the branch that we want to push, `master` (since we have committed all our changes to the `master` branch)

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

First, we'll need to create the *production build* of our React app. In the past, when we've been working with our React app, we've been running the code in *development mode* which displays clear error messages, automatically reloads and renders code changes, and more; however, the production version includes extra performance optimizations, a minified version of the code, and no detailed error messages (for security reasons).

`create-react-app` comes with a script that will create the production build for us with one simple command. Let's run the command at the root of our frontend project

```bash
npm run build
```

![image-20191110124758374](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110124758374.png)

We'll then see output similar to the above. You'll also notice a message describing how the project was built to be hosted as the server's root directory. We'll talk about what this means in a moment.

After taking a look at our project directory, you'll notice that our build script created a directory, **build**, in the root of our frontend directory (the directory from which we ran the build command). This directory contains a few files, the **index.html** file that provides some meta information and references the JavaScript files needed to run our application, and a **static** directory. This **static** directory contains the minified versions of our JavaScript code that was generated by the build process. Minified code is simply the result of removing all unnecessary characters from the source code without changing any  of the functionality; this allows for our files to be faster served over the Internet since there is less data to transmit. 

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
app.use(express.static('build'))
//...
```

The middleware takes in a root argumetn that specifies the root directory from which to serve static assets. In this case, we're serving our static assets from the **build** directory, so we'll pass in `'build'`. Now, since this middleware appears before our routes and route handlers, whenever our server receives an HTTP GET request, the middleware will check if our **build** directory contains any files that match as per the request's address. For example, visiting the domain of our application by default requests the **index.html** file; since there is a file called **index.html** in our **build** directory, Express will serve that file which serves up our React frontend.

We'll also need to update the `baseUrl` constant in our notes service as the URL from which we will be making resource requests to is different.

**/Frontend/src/services/notes.js**

```jsx
import axios from 'axios'
const baseUrl = '/notes'

//...
```

Here, we can remove the domain and just add the base path from which all of our resources will come from. We can do this as our frontend will now be hosted from the same domain/IP as the backend, so if the frontend was being served on `examplehost.com`, we can simply make requests to the **relative URL** `/notes` to which our browser will take that relative URL and assume to make requests to `examplehost.com/notes`. A relative URL is simply a URL that doesn't explicitly state the protocol (HTTP or HTTPS) or the domain, forcing the web browser (or Node) to assume that the path is appended upon the current URL.

### Automated build processes

Upon making these changes, we'll need to re-build our frontend and move over the **build** directory once again. Let's focus on automating this process with a script in the **package.json** of our backend directory (since this is the directory upon which everything is moving over to)

**/Backend/package.json**

```json
//...
{
  
}
//...
```

We can run multiple commands in a script by simply adding another command after `&&` in the sequence. For now, our scripts only work on *nix systems (unfortunately, some of these commands will not work on Windows unless you install `cygwin` or some other alternative). This is because we use  the `rm` command to remove the **build** directory in our backend (in preparation for the new directory that will be copied over) and the `cp` command to copy our **build** directory over to the root of our backend in our `build:ui` script.

Notice how we can run npm scripts from within a script as with our `deploy:full` script which runs our `build:ui ` and `deploy` scripts.

To sum the scripts up,

* The `build:ui` script first deletes the **build** directory in our backend (as it will be updated via a new copy). Then, it changes our directory (using the `cd` command) over to the **/Frontend** directory using a relative path. After this, since we're in the **/Frontend** directory, it'll run the `npm run build --prod` command to build our production React app. Finally, we use the `cp` command in order to copy the **build** directory over to the **/Backend** directory.
* The `deploy` script simply pushes our committed changes to the Heroku remote (assuming that we have already staged and committed some changes).
* The `deploy:full` script first executes the `build:ui` script as described earlier. Then, we stage all of the changes in our directory by executing `git add .`. After this, we commit our changes with the message `'UIBuild'` by executing `git commit -m UIBuild`. Finally, we run the `deploy` script as described earlier which simply pushes these committed changes to our Heroku remote.
* The `logs:prod` script simply executes the Heroku log command that we talked about earlier, allowing for easy access to the live logs of our Heroku container.

With this, if we make any updates and want the changes reflected on our remote/production server, we can just run `npm run deploy:full` which automates the entire process for us.

Upon running the above command, we can now access the address of our Heroku app and see the frontend for our app.

![image-20191110203212139](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110203212139.png)

When we visit the address of our Heroku app (in this case, `hidden-chamber-09544.herokuapp.com`), our express server simply returns the **index.html** in the **build** directory as we have specified with the usage of the `express.static()` middleware. This HTML file that we are served then causes us to load the JavaScript files referenced in the file to which these JavaScript files contain all of the React code, components, and logic that we have written. Then, when our browser receives all of these files, our app is then rendered and can request the notes via the JavaScript now running on our browser. Our React frontend makes the requests for the notes to the same address but with the '/notes' path appended to the end. You can see this process illustrated in the 'Network' tab of your chrome developer tools:

![image-20191110203717388](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191110203717388.png)

AFTER THIS, UPDATE THE CODE AND GO BACK TO WHEN THEY INITIALLY SERVED THE APP**\\

### API Conventions

In the above example, we pushed the code directly to the 'production server' in order to observe our changes. It is important that you never do this in practice as testing should be done thoroughly on your local machine before pushing your code to where it may affect others.

Currently, in our backend, our usage of routing methods for our API respond to endpoints defined at the root of our project. For example, visiting `exampleDomain.com/notes` would give us the collection of notes from our backend. The issue lies in the fact that we're serving our frontend from the same address as our backend, so now, we couldn't have a route for our frontend at `exampleDomain.com/notes` that served up a nice frontend page. As the backend's only purpose (other than serving up static assets in this case) is to provide an API for our frontend, it is best that we separate our API addresses out by adding a naming convention. In this case, we'll be adding `/api/` before all routes pertaining to our API (in this case, we should update all uses of routing methods as our middleware takes care of serving the static content).

Let's first update our backend routing method usage to reflect these new endpoints.

**/Backend/index.js**

For example, we would convert the update routing method handler function

```js
//...

app.get('/notes/:id', (req, res) => {
  const id = Number(req.params.id) // Takes in the parameter from the URL and casts it to a number
  const note = notes.find(note => note.id === id) // Finds the note that matches the parameter
  res.json(note)
})

//...
```

to

```js
//...

app.get('/api/notes/:id', (req, res) => {
  const id = Number(req.params.id) // Takes in the parameter from the URL and casts it to a number
  const note = notes.find(note => note.id === id) // Finds the note that matches the parameter
  res.json(note)
})

//...
```

We'll repeat this process for every handler function in **index.js**.

After this, we can update the `baseUrl` constant in our frontend in order to point our requests to the right endpoints.

**/Frontend/src/services/notes.js**

```jsx
import axios from 'axios'
const baseUrl = '/api/notes'

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

which is the *MongoDB URI*, where URI stands for Uniform Resource Identifier. A URI String of characters that identify resources on the Internet. For example, a URL is an example of a URI.

### Using MongoDB in Express

**Do not execute or add the code blocks in this section to any files. They are merely here to illustrate the concepts of creating and saving objects. We will work with Mongoose directly in our existing notes application later.**

Now that we have our database created and the URI for it, we can start using it in our Express application. In order to connect to our database and interface with it, we'll be using **Mongoose** which is an **object document mapper** that offers a higher level API over the official MongoDB driver (essentially makes things easier to understand and use at the expense of some performance and flexibility). It does this by adding an abstraction layer above the native driver.

To illustrate the higher level nature of Mongoose and how it might ease the development process. The following two blocks of code achieve the same task.

Using Mongoose:

```js
Book.find({ 'released_in_year': 2015 }, 'title author')
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

const mongoose = require('mongoose')

const url =
  'mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority'

mongoose.connect(url, { useNewUrlParser: true, useUnifiedTopology: true })

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
})

const Note = mongoose.model('Note', noteSchema)

//...
```

Let's take a look at this code.

First, we store the URI referencing the location of our database in the `url` const. We then call the `connect` method and pass in this `url` as well as an options object in order to actually establish a connection to our database. Notice how we have replaced '<password>' with the actual password for our user as well as changed 'test' following the forward slash (the database name) to 'note-app'. Later, we'll talk about how to hide our password as we are currently just hardcoding the full URI + password in a constant. This may become an issue if you were to ever upload your code or share it.

Then, we define the `noteSchema` by using a JavaScript object constructor to create a new `mongoose.Schema` object (based off a constructor function where construction functions are tasked with creating new JavaScript objects based off of parameters) with the appropriate fields and data types. Then we use our schema definition by converting our `noteSchema` into a model that we can actually work with. We do this with the following line

```jsx
const Note = mongoose.model('Note', noteSchema)
```

When you call `mongoose.model()` upon a schema, Mongoose will compile a model for you. The above code creates a model with the singular name of 'Note'. Mongoose will name the collection for these `Note` models as 'notes' by convention where Mongoose automatically names collections as the plural in lowercase with the schema referring to each document in the singular.

Keep in mind that document databases (like the one we're using now) are schemaless, meaning that they do not care how our data is structured in the database. These are just general guidelines and it is possible to store documents within a collection with completely different fields.

Additionally, you might also encounter the common practice of storing `mongoose.Schema()` within a variable or constant to make our code look slightly better with the usage of `mongoose.Schema()` as follows

```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const noteSchema = new Schema({
  content: String,
  date: Date,
  flagged: Boolean,
})
```

Notice how we were able to just use the constant `Schema` rather than directly calling `mongoose.Schema()`.

### Creating New Objects

Models are essentially 'fancy constructors' that are compiled from `mongoose.Schema()` definitions. They are responsible for creating and reading documents.

As our models are actually constructor functions, all of our instances of models have the same properties as provided by the constructor function, meaning that they all include methods that allow us to save our instance to the database and more. These instances of models will now be referred to as their more proper term, documents. The following example illustrates how we might go about creating a `Note` document that makes use of our `Note` model (which is actually a constructor function, giving all these documents the same set of built-in instance methods)

```js
const noteDocument = new Note({
  content: 'This document was created using the Note model',
  date: new Date(),
  flagged: false
})
```

We can now save this document to our database by calling its `save()` instance method which can be provided with an event handler that is called upon a fulfillment of its promise (if the note is successfully saved)

```js
noteDocument.save().then(result => {
  console.log('noteDocument has been saved')
  mongoose.connection.close()
})
```

Now, when the promise is fulfilled (the note being saved successfully), our event handler provided to the `then()` method of our promise will get called. This event handler prints a message and closes the database connection.

### Fetching Objects

In the context of our example application, we would find a given note by referencing the model, using its `find` method, and defining what we want to look for.

For example, if we wanted to find only notes with the `flagged` property set to true, we would write

```js
Note.find({ flagged: true }).then(result => {
  result.forEach(note => {
    console.log('Match found: ', note)
  })
  mongoose.connection.close()
})
```

Upon the fulfillment of this promise (the query being completed), our handler function prints out the results.

### Database Application

Let's introduce a database to our notes application so that its state is persistent. First, let's just copy the code that we used above to 'connect' to our database (and define our Note model) and paste it in **/Backend/index.js**

```js
const mongoose = require('mongoose')

const url =
  'mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority'

mongoose.connect(url, { useNewUrlParser: true, useUnifiedTopology: true })

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
})

const Note = mongoose.model('Note', noteSchema)
```

Let's also go ahead and delete our `notes` array that we store in memory.

Now, we can start rewriting our handler functions to use our database. For example, instead of having the following handler function

```js
app.get('/api/notes', (req, res) => {
  res.json(notes)
})
```

We would write the following handler to use our database as follows

```js
app.get('/api/notes', (req, res) => {
  Note.find({}).then(notes => {
    res.json(notes)
  })
})
```

Here, we use the `find` method on our `Note` model to find all notes that match the parameter that we passed to it. When we use an empty object as our parameter, we get all of the documents in the collection. Then, in our handler function for the promise that the `find` method had returned, we take in the collection of objects that it returns as our `notes` parameter. We then take the parameter variable and send it as our response using our `res` object's `json` method.

Upon converting this one routing method handler, we can observe the changes by using our browser and visiting **localhost:3001/api/notes**.

![image-20191112232145407](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112232145407.png)

Doing so yields us an empty page. This is because we don't actually have any notes in our database just yet. We can add some test data to our database by running the following code in a new file

**/Backend/create-test-data.js

```js
const express = require('express')
const mongoose = require('mongoose')
const app = express()

const url =
  'mongodb+srv://exampleuser:examplepassword@cluster0-nfeqs.mongodb.net/note-app?retryWrites=true&w=majority'

mongoose.connect(url, { useNewUrlParser: true, useUnifiedTopology: true })

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  flagged: Boolean,
})

const Note = mongoose.model('Note', noteSchema)

const firstTestNote = new Note({
    content: 'This is our first note in our database!',
    date: new Date(),
    flagged: false
})

firstTestNote.save().then(result => {
    console.log('Successfully saved!')
    mongoose.connection.close()
})

const secondTestNote = new Note({
    content: 'This is our second note in our database!',
    date: new Date(),
    flagged: true
})

secondTestNote.save().then(result => {
    console.log('Successfully saved!')
    mongoose.connection.close()
})

const thirdTestNote = new Note({
    content: 'This is our third note in our database!',
    date: new Date(),
    flagged: true
})

thirdTestNote.save().then(result => {
    console.log('Successfully saved!')
    mongoose.connection.close()
})
```

Upon running this 'application' with Node, we should see the following as console output

![image-20191112232829913](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112232829913.png)

This means that we have successfully created three test note objects that live on our database in our **notes** collection. If we take a peek at the **Collections** panel of our MongoDB Atlas cluster, we can see how our new `Note` documents lie under the `notes` collection.

![image-20191112233259744](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112233259744.png)

Notice how we did not have to provide a name for our collection. As stated earlier, Mongoose will automatically take our model name and make it lowercase and plural. For example, documents created under the 'Splotch' model would fall under the 'splotches' collection. If you need to, you can explicitly name the collection when creating a model by passing it as a third model. For example, if we wanted our notes to be stored in a collection called 'notesCollection', we would use the following to create our model

```js
const Note = mongoose.model('Note', noteSchema, 'notesCollection')
```

Going back to our application, after initializing our database with some test data, visiting **localhost:3001/api/notes** now yields the following result

![image-20191112233725952](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191112233725952.png)

Here, you'll notice the introduction of a few strange fields (`_id` and `__v`).

`__v` is simply the version key of our document. The version key is a property set on each document when it is created by Mongoose. It represents the *internal revision* of the document (basically keeps track of how many modifications had happened that may change the position of an element in an array); this mitigates the issue of referring to the wrong sub-document or document when dealing with arrays that may have elements with changing indexes. With this, we can refer to elements that match the version number  as well as the document we're looking for to mitigate some issues with targeting the wrong document and more.

Now, the frontend doesn't have much use for the `__v` property and it is looking for an `id` field rather than `_id`, so how might we go about sending objects in the correct 'format' to our frontend?

We can do this by updating the `toJSON` method of our documents of our note schema. We do this by the `set` method of a schema definition. This allows us to specify a built-in method of the documents that would be created under this schema definition and update it.

**/Backend/index.js**

```js
//...

noteSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString() // Just in case, we convert the _id field of our object to a string as it is in fact an object (even though it is displayed like a string)
    delete returnedObject._id
    delete returnedObject.__v
  }
})

//...
```

Here, we updated the `toJSON()` method of our `Note` documents by specifying the method as our first parameter and the updated object property as the second parameter to the `set()` method of `noteSchema`.

This `transform` property now has a function that takes in the  `returnedObject`, sets the `id` property to the `_id` property converted to a string, and deletes the `_id` and `__v` fields as its value.

This new `id` field is quite similar to a virtual. A virtual field is essentially an attribute that is nice to have (for sending data to the client, etc.) but isn't actually persisted (saved) to MongoDB. This applies to our `id` field as we aren't actually saving the `id` field to our database, but we are creating it when calling the `toJSON()` method in order to make things easier on the frontend.

We can then use this in our routing event handler by formatting every note with the `toJSON` method.

**/Backend/index.js**

```js
//...

app.get('/api/notes', (req, res) => {
  Note.find({}).then(notes => {
    res.json(notes.map(note => note.toJSON()))
  })
})


//...
```

Visiting **localhost:3001/api/notes** now yields the following result

![image-20191113000245903](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191113000245903.png)

You can now see how our updated `toJSON()` document method copies the `_id` field over to a new `id` field and strips the `__v` and `_id` fields.

#### Organization with Mongoose

Since things are getting pretty messy again, let's take advantage of our ability to move things into separate files and create a module for our Mongoose code. Most project structures follow the paradigm of moving Mongoose code and models into a folder called **models** that resides in our root backend directory.

**/Backend/models/note.js**

```js
const mongoose = require('mongoose')
const uri = process.env.MONGODB_URI

console.log('Connecting to: ', uri)

mongoose.connect(uri, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => {
    console.log('Connected to MongoDB')
  })
  .catch((error) => {
    console.log('Error connecting to MongoDB:', error.message)
  })

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
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

In this module, we simply define the schema for a note and assign it to the `noteSchema` constant. Then we update the note schema's `toJSON()` method as we did earlier. We also handle connecting to the database within this module. Take a look at the `url` constant. We no longer 'hardcode' the address, password and all, within the code as that is not wise with regard to security if we were to share our code or upload it. We now pull in the address through the use of an environment variable. We'll talk about defining environment variables in just a moment.

Finally, we export the model that is compiled for us based upon our `noteSchema`. Notice how these CommonJS modules are slightly different than ES6 modules with regard to importing and exporting. AS we only exported the model, we do not have access to anything else defined in this module (such as the schema, etc.).

Now, we can make use of this model (which happens to be the only thing we really need from that file for most tasks) by importing it as follows in **index.js**

**/Backend/index.js**

```js
//...
const Note = require('./models/note')
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

app.post('/api/notes', (req, res) => {
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
})

//...
```

Like we did earlier, we create a new note object using the `Note` model constructor function. Compared to the earlier block of code, we do something slightly different. Now, upon the fulfillment of our promise involving saving the note to our database, we send the response back containing the note (converted by our modified `toJSON()` method). We put this call to `response.json()` inside the `then()` method of our promise to ensure that we only send a response containing our note back if the note had indeed been saved. We can also get rid of our `generateNoteId()` function as Mongoose will now handle that for us.

We'll add functionality related to handling any errors that might occur with saving our note later.

Let's also quickly update our handler for fetching an individual note

**/Backend/index.js**

```js
//...

app.get('/api/notes/:id', (req, res) => {
  Note.findById(req.params.id).then(note => {
    res.json(note.toJSON())
  })
})

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

app.get('/api/notes/:id', (req, res) => {
  Note.findById(req.params.id)
    .then(note => {
    	res.json(note.toJSON())
  	})
  	.catch(error => {
    	console.log(error)
    	response.status(404).end()
  	})
})

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

app.get('/api/notes/:id', (req, res) => {
  Note.findById(req.params.id)
    .then(note => {
    	if (note) {
        res.json(note.toJSON())
      } else {
        res.status(404).end() // Condition fail
      }
  	})
  	.catch(error => {
    	console.log(error)
    	res.status(400).send({ error: 'Incorrect format for id parameter'})
  	})
})

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

app.get('/api/notes/:id', (req, res, next) => {
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

//...
```

In order to use the `next()` function, we need to pass it in as our third parameter. Here, you can see that we passed in our `error` that was caught to the `next()` routing function. If you pass anything to the `next()` function (except the string `'route'`), Express will regard the given request as an error and will skip any non-error handling routing and middleware functions. Thus, execution will skip to the error handling middleware.

Express comes with a built-in error handler that tries to take care of any errors that you might encounter. This automatically gets added at the end of the middleware function stack. Thus, if we pass an error to `next()` like we did above, the default error handler will handle it. We are also given the option of writing our own error-handling middleware functions. These are quite similar to other middleware function definitions, except error handlers have four arguments (the addition of the `err` (error) parameter) instead of three. We can write an error handler that pertains to our application as follows

**/Backend/index.js**

```js
//...

const errorHandler = (err, req, res, next) => {
  console.error(err.message)

  if (err.name === 'CastError' && err.kind === 'ObjectId') {
    return res.status(400).send({ error: 'Incorrect format for id parameter' })
  } 

  next(error)
}

app.use(errorHandler)

//...
```

Here, we simplified the logic involving checking for the type of error by simply just checking if the error was a `CastError` exception with a `kind` of `'ObjectId'` (meaning that our object `id` was not in the correct format). We then return the status code of `400` with an error in the response body upon the fulfillment of these conditions. If we get past this block of code, we simply forward the error to the default error handler that we talked about earlier. At the end of the code snippet, we simply load our new error handling middleware function.

Let's also define another middleware function that handles requests to endpoints that don't exist (for example, **localhost:3001/api/cars**).

**/Backend/index.js**

```js
//...

const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: 'Endpoint does not exist (Unknown Endpoint)' })
}

//...
```



With the addition of new middleware, it is important to be aware of the order that we are loading them into Express as they are executed in the same order. Thus, the order of loading middleware should resemble the following

**/Backend/index.js**

```js
//...

app.use(express.static('build'))
app.use(cors())
app.use(bodyParser.json())

//...
// ROUTING METHODS
//...

const unknownEndpoint = (req, res) => {
  res.status(404).send({ error: 'Endpoint does not exist (Unknown Endpoint)' })
}

app.use(unknownEndpoint)

const errorHandler = (err, req, res, next) => {
  console.error(err.message)

  if (err.name === 'CastError' && err.kind === 'ObjectId') {
    return res.status(400).send({ error: 'Incorrect format for id parameter' })
  } 

  next(error)
}

app.use(errorHandler)

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
app.delete('/api/notes/:id', (req, res, next) => {
  Note.findByIdAndRemove(req.params.id)
    .then((result) => {
      res.status(204).end()
    })
    .catch(error => next(error))
})
```

Here, even if we attempt to delete a note that does not exist (via an invalid `id` parameter), we respond with the status code `204` (No Content) unless the promise is rejected. If we wanted to provide two different status codes depending on if we actually deleted a note or if there is no note that matches the provided `id`, we could make use of the `result` object that is passed into our callback function provided to the `then()` method. You can also see how we pass on our error to the `next()` function were our promise rejected.

We can employ a similar function in order to update the `flagged` property of a given `Note` document in our database. Using the `findByIdAndUpdate` method, we can fetch a specific note and update it by writing the following

```js
app.put('/api/notes/:id', (req, res, next) => {
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
```

Here, we create a new `note` object using the request object's body. This note object was not created using the `Note` model constructor function as this note is not directly involved with the database, but rather is used as a regular JavaScript object to replace an object in our database. Then, we find the note object in our `notes` collection that matches the `id` request parameter and replace it with our `note` object.

Taking a look at our call to `findByIdAndUpdate`, the first parameter is the value of `_id` of the document that we want to replace/update. The second parameter is the actual object that we want to replace the object that we have selected with. Finally, the third parameter is the options object.

In the above example, we take advantage of this options object by giving the `new` boolean property a value of `true`. This returns the modified document (the document in our database after we have replaced it) rather than the original document. This emulates the behavior that we had originally in our application where we returned the updated `note` rather than the original `note` that Mongoose would have were it not provided with this option.

Upon testing these new features with Postman (or VSCode) we can see that most aspects of our server work just fine. Unfortunately, when attempting to update a note using our HTTP PUT routing method handler (to update the flagged property of a note object) or to delete a note using our HTTP DELETE routing method handler, we encounter the following error

![image-20191115113347499](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191115113347499.png)

Upon looking at the error message and the relevant documentation provided, we can fix this issue simply by adding the following line to **note.js**

**/Backend/models/note.js**

```js
const mongoose = require('mongoose')
mongoose.set('useFindAndModify', false)

//...
```

As Mongoose had a `findOneAndUpdate()` function before the native MongoDB driver had its own `findOneAndUpdate()` function, Mongoose will default to using the native driver's `findAndModify()` function instead. We can make Mongoose use the `findOneAndUpdate()` function by setting the `useFindAndModify` global option to `false` as we did with the command above.

As this is a global option, instead of adding the above code snippet to **note.js** we can also just add it to our connection options as follows

```js
const mongoose = require('mongoose')
const uri = process.env.MONGODB_URI

console.log('Connecting to: ', uri)

mongoose.connect(uri, {
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
    required: true
  },
  date: { 
    type: Date,
    required: true
  },
  flagged: Boolean,
})
```

Validation is middleware. Delving deeper into Mongoose, there are four types of middleware in Mongoose: document middleware, model middleware, aggregate middleware, and query middleware. Mongoose comes with several built-in validators that we can define in the `SchemaType`. We use the `required` and `minlength` validators above. The `required` validator ensures that the value of this property cannot be missing, meaning that it must be defined. The `minlength` validator sets the minimum length that the string value can have, where we set it at 1. This also implicitly defines the `required` validator as the minimum length logically sets that there should be a definition; thus, we would not have to explicitly define the `required` validator.

Strings come with `enum`, `match`, `minlength`, and `maxlength` validators. Numbers come with `min` and `max` validators. All schema types come with the `required` validator. Mongoose also allows for the creation of custom validators if we need validators that aren't satisfied by the built in ones.

Now, attempting to store an object that breaks any of the above restrictions in our code will throw an exception. Let's `catch` this exception and pass it to our error handler middleware as follows

**/Backend/index.js**

```js
//...

app.post('/api/notes', (req, res) => {
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

//...
```

Let's also add a condition in our error handler to provide a specific response for this type of error

**/Backend/index.js**

```js
//...

const errorHandler = (err, req, res, next) => {
    console.error(err.message)

    if (err.name === 'CastError' && err.kind === 'ObjectId') {
        return res.status(400).send({ error: 'Incorrect format for id parameter' })
    } else if (err.name === 'ValidationError') {
        return res.status(400).json({ error: err.message })
    }

    next(err)
}

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
app.post('/api/notes', (req, res) => {
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

  newNote.save()
  .then(savedNote => savedNote.toJSON())
  .then(formattedSavedNote => {
    response.json(formattedSavedNote)
  })
  .catch(error => next(error))
})
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

``` bash
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
    'env': {
        'commonjs': true,
        'es6': true,
        'node': true
    },
    'extends': 'eslint:recommended',
    'globals': {
        'Atomics': 'readonly',
        'SharedArrayBuffer': 'readonly'
    },
    'parserOptions': {
        'ecmaVersion': 2018
    },
    'rules': {
        'indent': [
            'error',
            4
        ],
        'linebreak-style': [
            'error',
            'unix'
        ],
        'quotes': [
            'error',
            'single'
        ],
        'semi': [
            'error',
            'never'
        ]
    }
}
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

![image-20191117132605255](/Users/josephsemrai/Library/Application Support/typora-user-images/image-20191117132605255.png)