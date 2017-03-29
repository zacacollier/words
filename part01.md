# Redux: a Gradual Approach: Part 1

Redux is, so you've been told, 'a library for managing application state'. Philosophically speaking, Redux is very simple, but when you're first starting out it's not always clear how and when to start implementing Redux as you're building a project.

We're going to start with an approachable, albeit slightly overwrought analogy, and gradually moving through the fiddly bits as we build a simple React Application.

Redux itself is something you gradually move towards, out of necessity - as such, every aspect of this tutorial will be based in gradual movements.

I suppose this is one reason why it's best to be familiar with vanilla React first - there comes a time where you need *something*, anything to help you structure and manage your data.

> "Necessity is the mother of invention."
> *-Plato*

Two modern predecessors - Flux and Elm - first aimed to address this problem, and later inspired Redux.

The two aren't necessary prerequisites for learning Redux - however, I **highly** encourage you to take the time to learn Elm at some point. As you pick up the language, you'll actually start to *intuit* the same design principles that are mirrored in Redux - and learn essential functional programming techniques that you can use to improve your JavaScript code quality.

Alright, let's get down to it.

## Splitting hairs

I'd like to modify our initial, hearsay impression of Redux by suggesting one that is a little more verbose: Redux is a library for 'managing *changes to* application state'.

To me, this implies that:
1. An application's state starts at **nothing**, and later it is updated - it goes from **nothing** to **something**.
2. Redux creates the **nothing**, and helps us understand how to update, by describing the state as **something**.

The initial **nothing** state becomes **something** when it is updated, that much is abundantly clear.

**But** the **update itself** represents the application's **new state**. That's the key takeaway here - an **update to the application's state** ***should model*** **the state itself**.

In the past, we've learned to use an *imperative* style of programming to "order around" our application state. We update state by telling it what to do. This is especially the case in jQuery, and still somewhat the case in React, although React is certainly an elegant approach that tends *away* from imperative programming.

Redux asks us to adapt a more *declarative* style of programming. Our state, from the get-go, is *declared*, and the updates are themselves are *described* as new versions of the state. We move away from telling our state what to do, towards *telling it how to do it*.

These updates are called **Actions**, and they do nothing more than what I've just described. **Actions** are objects that contain:
1. A `type` of change we'd like to see in our state
2. A piece of data (Optional - we don't always a value to tell update our state)

In order to actually **use** an Action, we'll write a function that takes in the action and uses it to *describe* an **updated model of the state**.

This function is called a **Reducer** - because under the hood a reducer function is principled according to JavaScript's native `.reduce()` method. The reducer will take a plain Object passed to it by an Action and return an Object that *describes* the updated version of the state by sandwiching together the *original state* and the *new, updated version of the state*.

There's no magic to this operation - we'll see how to use Plain JavaScript to achieve this result. Much of Redux's simplicity can be attributed to the fact that it relies on simple JS techniques we're already familiar with.

As far as a *gradual* introduction goes, I'll admit I may have jumped ahead a little. However, I felt it was appropriate in this case because Redux is really simple - so much so that we're actually all done examining Redux' core.

Seriously, there's really nothing more to the essence of Redux than what we've just described. Granted it's easier said than done, but I think you'll find that there's a kind of *zen* to Redux project design which you'll learn over time.

So with that, let's start walking through your first Redux application:

## Brass Tacks

We're going to build our own take on the Counter example taken from the [Redux docs](http://redux.js.org). You can also watch Dan Abramov build it on Egghead.io.

Before getting started, we should ask ourselves: what's the first thing we need to do? Never mind knowing **how** we'll do it, just answer the question: *"What's the first step in creating a Redux app?"*

There's no single answer - however, I would argue that we need an initial **nothing** state. Gotta have a **state** before you can start managing it, right? Besides, it's always nice to start from nothing and get something rendered as soon as possible.

We'll start minimally, adding React and Redux functionality as we go.

### Store

Redux **stores** the entire application's state in a single, aptly-named **store**. So let's make one now, using `.createStore()` provided by Redux.

```javascript
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';

import { createStore } from 'redux';

// import App from './App';

// 1: Create the Store
const initialState = {
  min: 0
}
const store = createStore();


ReactDOM.render(
  <h1>
    { store.getState().min }
  </h1>,
  document.getElementById('root')
);

```

Save that, hot reload the page and check out the result...uh-oh, an Error in the DevTools!

`Uncaught Error: Expected the reducer to be a function.`

But wait, we haven't even *written* a reducer! Why Redux, why?

Dig into the stack trace for yourself: click on the link in your DevTools, and you'll see why `createStore()` shit the bed:

```javascript
...
if (typeof reducer !== 'function') {
  throw new Error('Expected the reducer to be a function');
}
...
```

Makes sense right? Also notice how `createStore()` takes 3 parameters, the first of which is a `reducer`; the way we implemented `createStore()` a second ago, we didn't pass anything in. The first parameter - all of them, in fact - is simply `undefined`.

Before we fix this, we need to set up our Reducer to use the Redux DevTools extension (it's one of the best parts of Redux development - an absolute must-have).

Change `store` to:
```javascript
const store = createStore(
  reducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ ? window.__REDUX_DEVTOOLS_EXTENSION__() : f => f
);
```

Very nice - if the extension is available in the browser's global `window` object, it will have access to our App - otherwise, it will simply return an innocuous dummy function.

It won't work quite yet, we still have to write the `reducer` that `createStore` expects. Let's do that now:

### Reducer(s)

Reducers, you'll recall, take in a plain Object and return the new state.

> *Wait, I thought Reducers were supposed to take in Actions...*

Yes, that's true: but to be sure - Actions are simply Objects that alert the reducer of a particular change to our Application state. Actions bring a **type of state change** to a Reducer, and the Reducer hands off **a description of the state with the Action's changes applied** to the **Store**.

> In larger projects, you'll write functions that return Actions (called **Action Creators**). For now, we can do fine just passing around raw Objects.

Actions don't just magically appear at a Reducer's doorstep, they have to be delivered - rather, **dispatched**, using another aptly-named Redux method: `dispatch`.

> In the real world, mail doesn't just magically appear on your doorstep, it has to be *addressed* and *delivered* to you, right? Remember the *address* part, we're going to get to that in a sec.

All good? Let's continue by **modelling** the initial state and passing it into a `reducer`. We'll also add a simple `click` listener to trigger a `dispatch`:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

import { createStore } from 'redux';

// import App from './App';

// 1: Create the Store
const initialState = {
  min: 0
}
const reducer = (state = initialState) => {
  return state;
}
const store = createStore(reducer);


ReactDOM.render(
  <h1>
    { store.getState().min }
  </h1>,
  document.getElementById('root'),
  document.addEventListener('click', () => store.dispatch())
);
```

Try the example as above, and read the error in the console (that is, go to the section of the Redux source code where the error was thrown). You'll that `dispatch()` expects a plain object as its first argument.

Once you've done that, go ahead and pass in a valid argument to the `.dispatch()` call:

```javascript
  ...
  document.addEventListener('click', () => store.dispatch({ type: 'ADD' }))
  ...
```

You'll see the Actions being registered in the Redux DevTools, but in order to see the changes in our UI, we'll have to see to it that our `reducer` knows what to do with the Action - that our "mail" is being properly "addressed".

```javascript
...
const rootReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'UP':
      return { ...state, min: ++state.min }
    default:
      return state;
  }
}
...
```

> Notice how the default argument provided to `rootReducer` is now the entire `initialState` object, **not** just `initialState.min`. (You'll recall that during my talk I fumbled over this minor detail).

We're using a lovely `...` [spread operator]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) to return an object model of our current `state`, and the property that changed (`state.min`). You can imagine that this returned Object is a copy of the original state, with the new property "spread" over it.

> In ES5, this would be performed with `Object.Assign()`.

If you save and re-render, you'll see that this time our State is being updated in the UI!

Now that this is all wired up, try the next part yourself: add another property `max` to the `initialState` and render it to the DOM (instead of `min`). Then, see if you can make the `click` listener dispatch an Action that *decrements* `initialState.max`.

In Part 2 of this tutorial, we'll start with this, and then start turning this into a real React application.
