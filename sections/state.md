Stateless components
====================

##Write stateless components
I would have to say that by far, the vast, vast majority of pain that I've felt when writing React-based apps, has been caused by components that have too much state.

State makes components difficult to test
There's practically nothing easier to test than pure, data-in data-out functions, so why would we throw away the opportunity to define our components in such a way by adding state to them? When testing stateful components, now we need to get our components into "the right state" in order to test their behaviour. We also need to come up with all of the various combinations of state (which the component can mutate at any time), and props (which the component has no control over), and figure out which ones to test, and how to do so. When components are just functions of input props, testing is a lot easier. (More on testing later).

State makes components difficult to reason about
When reading the code for a very stateful component, you have to work a lot harder, mentally, to keep track of everything that's going on. Questions like "Has that state been initialised yet?", "What happens if I change this state here?", "How many other places change this state", "Is there a race condition on this state?", to mention a few, all become very common. Tracing through stateful components to figure out how they behave becomes very painful.

State makes it too easy to put business logic in the component
Searching through components to determine behaviour is not really something we should be doing anyway. Remember, React is a view library, so while render logic in the components is OK, business logic is a massive code smell. But when so much of your application's state is right there in the component, easily accessible by this.state, it can become really tempting to start putting things like calculations or validation into the component, where it does not belong. Revisiting my earlier point, this makes testing that much harder - you can't test render logic without the business logic getting in the way, and vice versa!

State makes it difficult to share information to other parts of the app
When a component owns a piece of state, it's easy to pass it further down the component hierarchy, but any other direction is tricky.

Of course, sometimes it makes sense for a particular component to totally own a particular piece of state. In which case, fine, go ahead use this.setState. It's a legitimate part of the React component API, and I don't want to give the impression that it should be off-limits. For example, if a user is typing into a field, it might not make sense to expose every keypress to the whole app, and so the field may track its own intermediate state until a blur event happens, whereupon the final input value is sent outwards to become state stored somewhere else. This scenario was mentioned to me by a colleague recently, and I think it's a great example.

Just be very conscious of every time you add state to a component. Once you start, it can become very easy to add 'just one more thing', and things get out of control before you know it!

##Avoid state when possible
State is the number one cause for overly complex components. Instead of just understanding what output a given input produces, a developer now has to think about what state a component is in, how it is mutated, and how it affects the output. In a component rich of state, testing might turn into a tedious task, where many state configurations have to be considered.

Component States
================

##Bump it up
[Here](https://reactjs.org/docs/state-and-lifecycle.html#what-components-should-have-state)’s another must-read section of the React docs. It advises us to keep our components as stateless as possible. If you find yourself duplicating/synchronizing state between a child and parent component, then move that state out of the child component completely. Have the parent manage state and pass it in as a prop to the child.

Consider a Select component, a custom implementation of the HTML <select> tag. Where should the “currently selected option” state live? The Select component usually represents some kind of data in the outside world such as the value of a specific field in a model. If we make a state called selectedOption live in the Select component then we would have to update both the model and the selectedOption state when the user chooses a new option. This kind of state duplication can be avoided by making Select accept a prop called selectedOption from the parent instead of managing it’s own state.

Intuitively it makes sense that this state belongs on the model, since that’s what the component represents. The Select component itself is just a (mostly) stateless UI control and the model is the backend. Select is mostly stateless because it can actually contain one piece of state: whether or not it is currently expanded. This state can live directly on Select because it’s a UI detail and usually not something that the parent is concerned with. In the next section we will demonstrate how the isCurrentlyExpanded state can be delegated to an even lower level component to keep true to the single responsibility principle.

###Source
[https://engineering.siftscience.com/best-practices-for-building-large-react-applications/](https://engineering.siftscience.com/best-practices-for-building-large-react-applications/)

##Keep your state flat
API's often return nested resources. It can be hard to deal with them in a Flux or Redux-based architecture. We recommend to flatten them with a library like [normalizr](https://github.com/paularmstrong/normalizr) and keep your state as flat as possible.

Hint for pros:
```
const data = normalize(response, arrayOf(schema.user))

state = _.merge(state, data.entities)
```
(we use [isomorphic-fetch](https://www.npmjs.com/package/isomorphic-fetch) to communicate with our APIs)

###Source
[https://blog.risingstack.com/react-js-best-practices-for-2016/](https://blog.risingstack.com/react-js-best-practices-for-2016/)

Immutable data
==============

##Immutable data representation
When your data representation is mutable, then you’ll find it very difficult to maintain simple components. Individual components will become more complex by detecting and handling the transition states when data changes, rather than handling this at a higher-level component dedicated to re-fetching the data. Additionally, in React, immutable structures often lead to better performance: when data changes in a mutable representation, you’ll likely need to rely on React’s virtual DOM to determine whether components should update; alternatively, in an immutable representation, you can use a basic strict equality check to determine whether an update should occur.

Any time we have deviated from this and used a mutable object in props, it has resulted in regret and refactoring.

See here for more general benefits of using immutable data structures.

###Source
[https://medium.com/building-asana/designing-simpler-react-components-13a0061afd16](https://medium.com/building-asana/designing-simpler-react-components-13a0061afd16)

##Use immutable states

Shared mutable state is the root of all evil - Pete Hunt, React.js Conf 2015

Immutable logo for React.js Best Practices 2016

Immutable object is an object whose state cannot be modified after it is created.

Immutable objects can save us all a headache and improve the rendering performance with their reference-level equality checks. Like in the shouldComponentUpdate:
```
shouldComponentUpdate(nexProps) {
 // instead of object deep comparsion
 return this.props.immutableFoo !== nexProps.immutableFoo
}
```

How to achieve immutability in JavaScript?
The hard way is to be careful and write code like the example below, which you should always check in your unit tests with deep-freeze-node (freeze before the mutation and verify the result after it).
```
return {
  ...state,
  foo
}

return arr1.concat(arr2)
```

Believe me, these were the pretty obvious examples.

The less complicated way but also less natural one is to use Immutable.js.
```
import { fromJS } from 'immutable'

const state = fromJS({ bar: 'biz' })
const newState = foo.set('bar', 'baz')
```

Immutable.js is fast, and the idea behind it is beautiful. I recommend watching the Immutable Data and React video by Lee Byron even if you don't want to use it. It will give deep insight to understand how it works.

###Source
[https://blog.risingstack.com/react-js-best-practices-for-2016/](https://blog.risingstack.com/react-js-best-practices-for-2016/)



#Use Redux.js
In point #1, I said that React is just for views. The obvious question then is, "Where do I put all my state and logic?" I'm glad you asked!

You may already be aware of Flux, which is a style/pattern/architecture for designing web applications, most commonly ones that use React for rendering. There are several frameworks out there that implement the ideas of [Flux](https://facebook.github.io/flux/), but without a doubt the one that I recommend is [Redux.js](https://redux.js.org/).

I'm thinking of writing a whole separate blog post on the features and merits of Redux, but for now I'll just recommend you read through the official tutorial at the above link, and provide this very brief description of how Redux works:

Components are given callback functions as props, which they call when a UI event happens
Those callbacks create and dispatch actions based on the event
Reducers process the actions, computing the new state
The new state of the whole application goes into a single store.
Components receive the new state as props and re-render themselves where needed.
Most of the above concepts are not unique to Redux, but Redux implements them in a very clean and simple way, with a [tiny API](http://redux.js.org/docs/api/index.html). Having switched a decent sized project over from Alt.js to Redux, some of the biggest benefits I've found are:

The reducers are pure functions, which simply do oldState + action = newState. Each reducer computes a separate piece of state, which is then all composed together to form the whole application. This makes all your business logic and state transitions really easy to test.
The API is smaller, simpler, and better-documented. I've found it much quicker and easier to learn all of the concepts, and therefore much easier to understand the flow of actions and information in my own projects.
If you use it the recommended way, only a very small number of components will depend upon Redux; all the other components just receive state and callbacks as props. This keeps the components very simple, and reduces framework lock-in.
There are a few other libraries that complement Redux extremely well, which I also recommend you use:

* [Immutable.js](https://facebook.github.io/immutable-js/) - Immutable data structures for JavaScript! Store your state in these, in order to make sure it isn't mutated where it shouldn't be, and to enforce reducer purity.
* [redux-thunk](https://github.com/reduxjs/redux-thunk) - This is used for when your actions need to have a side effect other than updating the application state. For example, calling a REST API, or setting routes, or even dispatching other actions.
* [reselect](https://github.com/reduxjs/reselect) - Use this for composable, lazily-evaluated, views into your state. For example, for a particular component you might want to:
  * inject only the relevant part of the global state tree, rather than the whole thing
  * inject extra derived data, like totals or validation state, without putting it all in the store

You don't necessarily need all of these from day one. I would add Redux and Immutable.js as soon you have any state, reselect as soon you have derived state, and redux-thunk as soon as you have routing or asynchronous actions. Adding them sooner rather than later will save you the effort of retro-fitting them in later on.

https://camjackson.net/post/9-things-every-reactjs-beginner-should-know#use-redux-js