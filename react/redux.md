
# Redux

In this section we're going to walk through Redux, the library we use to manage state within our React apps.

**State**

To begin, we need to understand state.

*What is `state`?*

`State` is a type of data that controls a component, similar to `props`. Where `props` is data set by the parent and fixed throughout the lifecycle of a component, `state` is data that changes during the component lifecycle.


**Redux and React-Redux**

As previously mentioned, our component functions will not be handling state. That's where [Redux](https://github.com/reactjs/redux) comes in.

From the [Redux Github page](https://github.com/reactjs/redux):

> Redux is a predictable state container for JavaScript apps. It helps you write applications that behave consistently, run in different environments (client, server, and native), and are easy to test. On top of that, it provides a great developer experience, such as live code editing combined with a time traveling debugger.

There's a lot to read regarding the [Core Concepts](http://redux.js.org/docs/introduction/CoreConcepts.html) and [Principles](http://redux.js.org/docs/introduction/ThreePrinciples.html) surrounding Redux (highly encouraged), but we will touch on these concepts in this guide.

To use Redux in our apps, we will include two more packages:

	1. redux
	2. react-redux

The *Redux* package provides the Redux API to use functions like `createStore`, `combineReducers`, and `compose` (you will see these and others a bit later...).

*React-redux* is a helpful library that provides React bindings for Redux.

The basic elements of the Redux Framework you will need to familiarize yourself with are:

- *Store* - Contains the application's state
- *Actions* - Simple JS objects that send data from your application to your store
- *Reducer* - Pure JS functions that determine how the state is affected by an action
- *Middleware* - Middleman between the actions and the reducer to determine if the action should be modified or canceled before reaching the reducer

**Store**

To start, we will need to set up a `store`.

The `store` is where we...store, our state. It is a read-only object, and should be considered the "single source of truth" regarding the state of our React application.

To be clear, there is only one store in our application. To split out data management logic throughout the app, we will rely on the reducers.

To create a store, you simply call `createStore(reducer, [preloadedState], [enhancer])`.

At LPL, we create our store within an `initializeStore` function like so:

```JSX
import { applyMiddleware, compose, createStore } from 'redux'
import thunkMiddleware from 'redux-thunk'
import { middleware as apiMiddleware } from './api'

function initializeStore () {

  const reducer = {}
  // We'll add some logic to create the reducer here

  // Add support for the Redux Dev Tools in chrome.
  const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose

  // We'll talk about middleware in a later section
  const enhancers = composeEnhancers(
    applyMiddleware(
      apiMiddleware,
      thunkMiddleware,
    )
  )

  const intialState = {}

  return createStore(reducer, initialState, enhancers)
}

export default initializeStore
```

**Actions**

Before we dig in to our reducers, we'll need to define our actions. As noted above, actions are just JS objects that pass data along to your store (through reducers) with an indication of what should be done with the data.

A simple example would look like this:

```JSX
const ADD_TODO = {
  type: 'ADD_TODO',
  text: 'Finish this guide'
}
```

There are few ways set up your actions in React apps, but at LPL we make use of the `createAction` function in the `redux-actions` package to define the actions.

We then combine this with `effects` to be able to connect to our APIs. Effects functions are responsible for sending and fetching data to or from an API. There will be more on making these requests later, but in general, our `todo/actions.js` might look like this:

```JSX
import { createAction } from 'redux-actions'

export const createItem = createAction('CREATE_ITEM')
export const editItem = createAction('EDIT_ITEM')
export const destroyItem = createAction('DESTROY_ITEM')
```

With a root-level `effects.js` like this:

```JSX
import { api } from './api' // this includes logic around the middleware when making API calls with our actions

export function createItem ({ text }) {
  return api.post('/todos', { todo: { text, completed: false } })
}

export function editItem ({ id, text }) {
  return api.patch(`/todos/${ id }/update_text`, { todo: { text } })
}

export function destroyItem ({ id }) {
  return api.destroy(`/todos/${ id }`)
}
```

**Reducer**

Once we've created some actions, we can build our reducers, both for the store and at the model level.

For the store:

```JSX
import { applyMiddleware, compose, createStore, combineReducers } from 'redux' // added combineReducers to make use of multiple reducers
import thunkMiddleware from 'redux-thunk'
import { middleware as apiMiddleware } from './api'

// new imports
import { routerReducer } from 'react-router-redux' // special reducer provided by react-router-redux
import { reducer as formReducer } from 'redux-form' // special reducer provided by redux-form
import { reducer as apiReducer } from '@launchpadlab/lp-redux-api' // special reducer created internally
import { reducer as rootReducer, reducerKey as rootKey } from './reducer'

function initializeStore () {
  /*
   * Combine the reducers into one that Redux can handle. The keys below are
   * important as data in the store will be namespaced and each reducer
   * only receives the slice of state according to the key.
   */
  const reducer = combineReducers({
    form: formReducer,
    [rootKey]: rootReducer,
    routing: routerReducer,
    api: apiReducer,
  })

  // Add support for the Redux Dev Tools in chrome.
  const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose

  const enhancers = composeEnhancers(
    applyMiddleware(
      apiMiddleware,
      thunkMiddleware,
    )
  )

  const initialState = {}

  return createStore(reducer, initialState, enhancers)
}

export default initializeStore

```

Where our main `reducer.js` looks like:

```JSX

import { combineReducers } from 'redux';

// we would add each of our models' reducers here
import {
  reducer as todoReducer,
  reducerKey as todoReducerKey
} from './todo'


const reducerKey = 'root' // unique name to distinguish from named reducers we imported into our store above

const reducer = combineReducers({
  [todoReducerKey]: todoReducer, // and add our models' reducers here
})

export { reducer, reducerKey }
```

Finally, the reducer for our model is where all the model-specific data logic lives, and you will see how it ties back to our actions:

```JSX
// todo/reducer.js:

import * as actions from './actions'
import { set, getOr } from 'utils'
import { handleActions } from 'redux-actions'
import { setFromRequest } from '@launchpadlab/lp-redux-api'

// Reducer Keys
const reducerKey = 'todo'
const slice = 'root.todo'

const initialState = {
  todos: {}
}

const reducer = handleActions({
  ...setFromRequest(REQ_FETCH_TODOS, 'todos'), // have been leaving this stuff out for simplicity...
  [actions.createItem]: (state, { payload: newItem }) => {
    const itemCollection = getOr([], 'todos.success', state)
    return set('todos.success', [ ...itemCollection, newItem ], state)
  },
  [actions.editItem]: (state, { payload: { id, text } }) => {
    const itemCollection = getOr([], 'todos.success', state)
    const newItemCollection = itemCollection.map(item => {
      return (item.id === id) ? { ...item, text } : item
    })
    return set('todos.success', newItemCollection, state)
  },
  [actions.destroyItem]: (state, { payload: id }) => {
    const itemCollection = getOr([], 'todos.success', state)
    const newItemCollection = itemCollection.filter(item => item.id !== id)
    return set('todos.success', newItemCollection, state)
  }
}, initialState)

export { reducer, reducerKey }
```

The key code to note here is that the reducer will look at the action given to it (using `dispatch` on store, which we will get to shortly), take the current state, and return a NEW state based on action.

If you look closely, you will also note that the reducer isn't returning the entire state back, it's only returning the *slice* that changed.

This is another way using Redux helps you minimize interactions between unrelated code. This model's reducer will only affect this model's slice of the state.

**Mapping to Props**

The final element to tie all these pieces together is to surface our state and our actions to our components.

We do this through a function we write: `mapStateToProps()`, and a constant: `mapDispatchToProps`, written for each of our models, at the `view` level. We then make use of the Redux `connect()` function to connect these state elements to our component.

*mapStateToProps( )*

`mapStateToProps` takes the `state` of your app, and returns the pieces of state that you need aligned to the props for you component. It looks something like this for `views/todo.js`:

```JSX
function mapStateToProps (state) {
  return {
    displayedItems: selectors.displayedItems(state)
  }
}
```

*mapDispatchToProps*

Similar to `mapStateToProp()`, `mapDispatchToProps` aligns the actions for your model to the props it will receive:

```JSX
const mapDispatchToProps = {
  createItem: actions.createItem,
  editItem: actions.editItem,
  destroyItem: actions.destroyItem,
}
```

We can then export our composed component:

```JSX
export default compose(
  connect(mapStateToProps, mapDispatchToProps)
)(Todo)
```


**Summary**

Phew! This was a long one. There are a lot of moving pieces, but the main takeaways are understanding how Redux utilizes the `store`, `actions`, and `reducers` to manage the state of your app, and familiarizing yourself with how we implement these elements at LPL.

Remember, the `store` is the container for your `state`, which is an immutable JavaScript object with all of your data. State never changes, but you can update the store to reflect changes to your data.

You can update the store through a series of steps. First, `effects` are used to initiate API requests to modify data in some way. When the effect is successful, `actions` are dispatched to deliver the resulting payload to the reducers. The `reducer` then decides how to update the store, by returning a *new* state based on the dispatched action.

You can read more about each of these elements in their respective guides:
- [effects](./effects)
- [actions](./actions)
- [reducer](./reducer)

Contributors
-

+ Nathan Jones
+ Ifat Ribon
