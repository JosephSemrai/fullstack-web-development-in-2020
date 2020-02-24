[TOC]



###BRING IN THE REST FROM GOOGLE DOCS

###Disclaimer: I would heavily recommend reading the documentation for these components for a deeper understanding. By the time that you have read this, the components might have changed. The documentation will provide you with the deepest understanding of these components should you want to learn more.



Moving components to other files gives us the benefit of being able to easily scale the components in functionality (for example, we can just add a `StyleSheet` specific to the component within the file) while also making having larger quantities of components more manageable. Having components in an external file also allows us to reuse the components in other files.

### Conditional Renders

When dealing with renders that may depend on a boolean flag, it is recommended to use a ternary operator to hide or show that component. For example,

```javascript
render() {
  return (
  	<View style={styles.container}>
    <Button title="toggle contacts" onPress={this.toggleContacts} />
    {this.state.showContacts ? (
     <ScrollView>
     	{contacts.map(contact => <Row {...contact} />)}
      </ScrollView>
     ) : null
		}
  )
}
```

Here you can see the ternary syntax in use. If ```this.state.showContacts``` turns out to be true, it will render the left half of the colon and vice versa.



The syntax for a ternary operator is as follows:

```javascript
condition ? thisExecutesIfTrue : thisExecutesifFalse
```

Here, if condition evaluates to be true, the "thisExecutesIfTrue" block will be returned and executed, whereas if it evaluates to be false, "thisExecutesifFalse" will be returned.

#### Logical Operator Trick

We can simplify the code block above even further.

```javascript
render() {
  return (
  	<View style={styles.container}>
    <Button title="toggle contacts" onPress={this.toggleContacts} />
    {this.state.showContacts && (
     <ScrollView>
     	{contacts.map(contact => <Row {...contact} />)}
      </ScrollView>
			)
		}
  )
}
```

We can write the block like this using the ```&&``` operator, which will only returnand render the contacts ```ScrollView``` if ```this.state.showContacts``` evaluates to be true. This shorthand returns the object in the second half of the expression if the expression in the first half (before the ```&&```) evaluates to be true.

### FlatList

Now, ```ScrollView```s are a pretty good introduction to lists in React Native, but when it comes to larger lists, it is extremely inefficient and can cause the user to have to wait several seconds for the list to be shown. This is due to teh nature of ```ScrollView```s having to render all of their children before being shown.

This brings us to a component called a ```FlatList```. ```FlatList```s are virtualized (a convinence wrapper around the component VirtualizedList) which means that it only renders what's needed on screen at the moment. This can be orders of magnitude faster than a standard ```ScrollView```. It is also important to consider that **rows are recycled** and when they leave visibility they can be unmounted. With this, you may have to figure out how to maintain the state of these components if they are unmounted in a ```FlatList```. We will talk about doing this later in the book.



###Getting Started with ```FlatList```

```FlatList```s require that you pass a renderItem function to them.

```javascript
{this.state.showContacts && (
	<FlatList>
  	renderItem={obj => <Row {...(obj.item)} />}
  	data={contacts}
  </FlatList>
)}
```

Here, for the ```renderItem``` function, we have it take an object passed into it from the data object which contains all our contacts, and then it accesses the data through ```obj.item``` and uses the spread syntax on it in order to pass it into the ```Row  ``` component.

**Important Note:** 

```javascript
sort = () => {
  this.setState(prevState => ({
    contacts: prevState.contacts.sort(compareNames),
  }))
}
```

With ```FlatList```s, they will only update if it's props are updated. This, when we call ```sort``` as implemented above, this will not immediately display the sorted list as the ```sort()``` function does not create a new array but rather just rearranges the elements within itself. With this, when we set the state, we are still referencing the exact same array and React does not know that an update has occured. Thus, the ```FlatList``` will not reflect the changes.

This brings us to the concept of **immutablility being extremely important**.

Here we can use the spread syntax once again to "spread" our existing array into a new array and then sort this new array as shown below:

```javascript
sort = () => {
  this.setState(prevState => ({
    contacts: [...prevState.contacts].sort(compareNames),
  }))
}
```

React will now see that there is a reference to a new array in its state and will update the FlatList accordingly. Please note that all of the objects within the array still reference the exact same objects, the only thing that has changed is that there is now a new array that contains all of these references.





