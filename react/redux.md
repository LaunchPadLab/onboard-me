
# Functional Programming in React | Redux

**State**

*What is `state`?*

`State` is a type of data that controls a component, similar to `props`. Where `props` is data set by the parent and fixed throughout the lifecycle of a component, `state` is data that changes during the component lifecycle.
<!-- add link to docs -->

*Component Lifecycle* 

The component lifecycle is triggered by the `render` method, and takes place between the moments a component is created and destroyed.

The component lifecycle provides us a window of opportunity to decide if a component should be updated, as well as react to changes in `state`.

**Redux, React-Redux, & Store**

As previously mentioned, our component functions will not be handling state.

That's where [Redux](https://github.com/reactjs/redux) comes in.

From the Github page:

> Redux is a predictable state container for JavaScript apps. It helps you write applications that behave consistently, run in different environments (client, server, and native), and are easy to test. On top of that, it provides a great developer experience, such as live code editing combined with a time traveling debugger.

There's a lot to read regarding the [Core Concepts] and [Principles] surrounding Redux (highly encouraged). 

For now, we really only care about the `store` that Redux provides.

The `store` is where we...store, our state. It is a read only object, and should be considered the "single source of truth" regarding the state of our React application.

Instead, we will rely on two packages:

	1. redux
	2. react-redux

TODO: What is redux
TODO: What is react-redux

TODO **Connect to Store**

TODO **mapStateToProps()**

TODO: What is `mapStateToProps()`?

	1. mapStateToProps()
	2. mapDispatchToProps()

For now let's look at `mapStateToProps()`:

```javascript
// person.js

	function mapStateToProps (state) {
		return {
			name: state.name
			age: state.age
		}
	}

```

You might be wondering: 

"Hey, I thought you said state was handled in the component lifecycle; i.e. when `render` is called in our component?"

That's correct - we will handle incorporating our stateful functions when we connect to the `store` at the end of our component file.












**Step ?: Rendering**
**Step ?: Handlers**
```  
// function reducer(state = initialState, action) {
//   // write comments describing handers
//   const handlers = {
//     [SET_FILTER]: (state, action) => set('filter', action.payload, state)
//   }
//
//   const myHandler = handlers[action.type]
//   return myHandler ? myHandler(state, action) : state
//   switch (action.type) {
//     case SET_FILTER:
//       return set('filter', action.payload, state)
//     default:
//       return state
//   }
// }  
```


**Step ?: React with Rails API**

Effects => send / fetch data to API
Actions => receive responses from API, update state
Reducer => absorbs both and figures out where to put things








Contributors
-

+ Nathan Jones
