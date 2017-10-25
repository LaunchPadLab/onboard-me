<!-- TODO:

- [ ] Include links to relevant documentation for more robust explanations on topics.

-->

# Functional Programming in React | The Basics

This guide is intended as an introduction to how we build React front-end interfaces via the functional programming style.

We'll go over the basic requirements, patterns, and expectations of every LPL React component.

## Contents

- [Goal](#goal)
- [Basic Composition](#basic-composition)
    - [Import Packages](#import-packages)
    - [PropTypes](#proptypes)
    - [Component Declaration](#component-declaration)
    - [JSX & Props](#jsx-and-props)
    - [Assign PropTypes to Component](#assign-proptypes-to-component)
    - [Export Default](#export-default)
- [Final](#final)
- [Next Steps](#next-steps)

# Goal

**React components should be predictable**

A core tenant of functional programming is predictability. The inputs of my function should remain consistent, such that the expected return is predictable and testable.

Predictability in functional programming is typically achieved by writing functions that are:

 - Immutable (does not mutate data, or alter global state)
 - Uniform (does one thing really well, always)
 - Short (legible, concise, and well named)

You might be thinking - "Hey, React is supposed to update state!" - and you'd be correct. To achieve our FP goals while updating state, we'll send messages to Redux, which will take care of updating state for us.

This means we never have to write functions that mutate data, or directly update the global state.

Ultimately we feel this approach achieves a few things really well:

 - Allows quick and flexible product growth
 - Mitigates exception errors in production
 - Reusable components without side-effects

Note:

  Even if you've been writing JavaScript for a while, doing so in a functional programming style will introduce a learning curve. If you've never done ANY functional programming before, this curve can be quite steep.

  Fear not! We are here to help. It's important to give your mind time to adjust and to not be too hard on yourself. Functional programming can be abstract and quite challenging, but at the end of the day we feel it's worth the squeeze.

# Composition

Each of our React components follows a basic compositional pattern.

## Import Packages

At the top of your component file, you'll need to import the following packages:

```javascript
// person.js

import React from 'react'
import PropTypes from 'prop-types'
```

Depending on what type of component you're building, you many need to import additional packages. For now `react` and `prop-types` represent the bare minimum you'll need to get started with your first React component. We'll be adding more packages as we add complexity to our components.

## PropTypes

Next, let's define our `propTypes` object.
Simply put, `propTypes` allow us to declare data types for our incoming props.

```javascript
// person.js

...
import PropTypes from 'prop-types'
...

const propTypes = {
  name: PropTypes.string.isRequired
  age: PropTypes.number.isRequired
}
```

Here we've defined some propTypes for our `Person` component.
We're expecting a person to have a `name` value of type `string`, as well as an `age` value of type `number`.

If a `name` or `age` prop is passed to our component, but does not match the data types we've defined, React will throw an error.

Therefore,

 - name **MUST** be of data type **STRING**
 - age **MUST** be of data type **NUMBER**

By explicitly declaring our prop data types, PropTypes is able to monitor all incoming data and warn us against type conflicts.

## Component Declaration

Let's write our first component!

We're going to declare our Person component like so:

```javascript
// person.js

const Person = () => {
  return (
    <h1>Hello World!</h1>
  )
}
```

Whoa, what's going on here?!

If you've seen any examples of React online, you'll notice our declaration is very different.

Typically, we're taught to declare React components like so:

```javascript
// person.js

class Person extends React.Component {
  render () {
    return (
      <h1>Hello World!</h1>
    )
  }
}
```

However, our main goal is to ensure predictability in our React environment. Declaring components as `class Person extends React.Component`, typically indicates that `state` will be managed within our component.

Instead, we are going to manage `state` with `react-redux` and `redux` external to our component. Since we are not managing `state` within our components and simply passing `props`, there is no need to declare our components as React classes.

React allows us to declare our components as *stateless functional components*; i.e. pure functions.

There are numerous upsides to this approach, the primary benefit being our components are easy to understand; they just render stuff!

## JSX and Props

You'll notice in our component declaration we wrote some code that looks a lot like HTML markup.

```javascript
// person.js

...

return (
  <h1>Hello World!</h1>
)
```

This is called `JSX`. Simply put `JSX` is sytax extension to JavaScript which helps describe what our UI should look like and render React elements.

The example above is describing a `h1` element, with some text of "Hello World!". In our browser, this will produce a `h1` element which wraps the text specified.

What if we wanted to programmatically render information fetched from our server? This is where `props` come in:

```javascript
// person.js

function Person (props) {
  return (
    <h1>
      Hello { props.name }!
      You are { props.age } years old.
    </h1>
  )
}

```

`Props` are passed to our component the same way we would pass arguments to a function. Our function is a valid React component because it accepts a single `props` object argument with data and returns a React element.

In this example, we might expect our `props` object to look like this when we eventually render our component:

```javascript
const props = {
  name: "Sarah",
  age: 26,
}
```

`Props` is an object made available by React when it recognizes an element representing a user-defined component. We'll cover more about `props` later when we render our component.

Because `props` is the only argument we are passing to our React component, we can "destructure" our `props` object using ES6 syntax:

```javascript

// person.js

function Person ({ name, age }) {
  return (
    <h1>
      Hello { name }!
      You are { age } years old.
    </h1>
  )
}

```

This syntax completely eliminates the need to write `props.attribute`, and makes explicit which object attributes we are passing in.


## Assign PropTypes to Component

Assigning your `propTypes` object to your component is a short but crucial step. You might remember in `Step 2` we declared a `propTypes` object.

```javascript
// person.js

...

import PropTypes from 'prop-types'
...

const propTypes = {
  name: PropTypes.string.isRequired
  age: PropTypes.number.isRequired
}
```

Unfortunately, React doesn't automagically make this object available to our component.

Fortunately, it's easy to append:

```javascript

Person.propTypes = propTypes

```

## Export Default

Lastly, we need to export this component and make it available for import in other sections of our application.

```javascript
// person.js
...

export default Person

```

## Final

  If all went according to plan, you should have a `Person` that looks something like this:

```javascript

import React from 'react'
import PropTypes from 'prop-types'

const propTypes = {
  name: PropTypes.string.isRequired
  age: PropTypes.number.isRequired
}

const defaultProps = {
  name: "Sarah",
  age: 26
}

function Person ({ name, age }) {
  return (
    <h1>
      Hello { name }!
      You are { age } years old.
    </h1>
  )
}

Person.propTypes = propTypes

Person.defaultProps = defaultProps

export default Person

```

  You might be thinking, "This is a pretty basic component..." and you'd be correct. Currently, the only way to change what this component renders is to alter the `default_props` object or pass it new props on render.

  There is also no way for the component to change the state - i.e., stored information - of the application. This means we've succeeded in isolating this component to do only one job: take in data and render that data in a predictable manner.

# Next Steps

  In the next [section](./redux), we'll cover how to use `react-redux`, request data from our backend API, and update state in our React application.
