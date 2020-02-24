

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
import React from 'react'
import StyledFirebaseAuth from 'react-firebaseui/StyledFirebaseAuth'
import firebase from 'firebase/app' // Allows us to create app instances
import 'firebase/auth' // Gives us Firebase authentication support

// Firebase configuration provided to us by the app creation process
const firebaseConfig = {
  apiKey: "AIzaSyDJf0kn4MHA8mMgolqu1DLh3YqrjSAt57Q",
  authDomain: "notes-app-bfbc5.firebaseapp.com",
  databaseURL: "https://notes-app-bfbc5.firebaseio.com",
  projectId: "notes-app-bfbc5",
  storageBucket: "notes-app-bfbc5.appspot.com",
  messagingSenderId: "768088207404",
  appId: "1:768088207404:web:abd4b64d40dbab447e6189",
  measurementId: "G-NBKG2S4VX5"
};

// Initializes Firebase and creates an app instance
firebase.initializeApp(firebaseConfig)

// Configure FirebaseUI.
const uiConfig = {
  // Popup signin flow rather than redirect flow.
  signInFlow: 'popup',
  // We will display Google and Facebook as auth providers. Also configures the component with the provider IDs.
  signInOptions: [
    firebase.auth.GoogleAuthProvider.PROVIDER_ID,
    firebase.auth.FacebookAuthProvider.PROVIDER_ID
  ],
  callbacks: {
    // `signInSuccessWithAuthResult` is called with the user object when the user has successfully logged in
    signInSuccessWithAuthResult: () => false // Empty function as we do not want to do anything just yet
  }
}

const App = () => {
  return (
    <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()}/>
  )
}

export default App
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
    const unsubscribe = onAuthStateChange() // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe()
  }, [])
```

Here, we subscribe to the authentication state and deal with it in `onChange`. The `onAuthStateChanged()` function returns the unsubscribe function for the observer (subscriber). We then return a function that will call this unsubscribe function, to which React will call this function upon unmounting.

#### Managing User State Through Subscriptions

We'll now observe/subscribe the authentication state of our user to which we'll manage our state dependent on what our provider (Firebase) tells us:

**/src/App.js**

```jsx
import React, { useEffect } from 'react'

//... EXISTING CONFIGURATION

const onAuthStateChange = () => {
  return firebase.auth().onAuthStateChanged(user => {
    if (user) {
      console.log("The user is logged in.")
    } else {
      console.log("The user is not logged in.")
    }
  })
}  

const App = () => {
  
  
  useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange() // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe()
  }, [])

  return (
    <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()}/>
  )
}

export default App
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
import React, { useEffect, useState } from 'react'

//... EXISTING CONFIGURATION

const onAuthStateChange = (setUser) => {
  return firebase.auth().onAuthStateChanged(user => {
    if (user) {
      console.log("The user is logged in.", user)
      setUser(user)
    } else {
      console.log("The user is not logged in.")
    }
  })
}

const App = () => {
  const [user, setUser] = useState()

  useEffect(() => {
    // Realtime authentication listener subscription
    const unsubscribe = onAuthStateChange(setUser) // Returns the unsubscribe function
    // Unsubscribes from the listener when unmounting
    return () => unsubscribe()
  }, [])

  return (
    <>
      {JSON.stringify(user)}
      <StyledFirebaseAuth uiConfig={uiConfig} firebaseAuth={firebase.auth()}/>
      <button onClick={() => firebase.auth().signOut()}>Sign-out</button>
    </>
  )
}

export default App
```

Here, we made a few changes. 