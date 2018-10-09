#Functional Components
##Smart and dumb component separation
Give yourself a pat on the back if you read Dan Abramov’s post about [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). It gives you a perfect introduction to the idea. The next example shows how to build a simple smart component that utilises two dumb components.

You first want to make a root component that will serve as the container. It will cover all the possible side effects, gather all the necessary data and set up all the possible callbacks that will be propagated to your dumb components.

Up in the hierarchy lies our root component:
```
import React, { PureComponent, PropTypes } from 'react';
import { connect } from 'react-redux';
import { Link } from 'react-router';
import { List } from 'immutable';

import TodoItem from './TodoItem';
import TodosForm from './TodosForm';
import todosSelector from './todosSelector';

import * as todoActions from '../../../universal/modules/todo/todoDuck';

const underline = { textDecoration: 'underline' };

class Todos extends PureComponent {
  static propTypes = {
    todos: PropTypes.instanceOf(List).isRequired,
    phase: PropTypes.string.isRequired,
    fetchTodos: PropTypes.func.isRequired,
    createTodo: PropTypes.func.isRequired,
    editTodo: PropTypes.func.isRequired,
    deleteTodo: PropTypes.func.isRequired,
  };

  constructor(props) {
    super(props);

    this.handleSubmit = this.handleSubmit.bind(this);
  }

  componentDidMount() {
    const { phase, fetchTodos } = this.props;

    if (phase !== 'success') {
      fetchTodos();
    }
  }

  handleSubmit({ todo }) {
    const { createTodo } = this.props;

    createTodo(todo);
  }

  render() {
    const { todos, editTodo, deleteTodo } = this.props;

    return (
      <div className="Todos">
        <div className="Todos-header">
          <h2>Todos:</h2>
          <span>Displaying {todos.size} todos.</span>
        </div>
        <div className="Todos-list">
          {todos.map((todo, index) => (
            <TodoItem
              key={index}
              todo={todo}
              index={index}
              onEdit={editTodo}
              onDelete={deleteTodo}
            />
          ))}
        </div>
        <div className="Todos-form">
          <TodosForm onSubmit={this.handleSubmit} />
          <div className="Todos-filters">
            <h4>Show:</h4>
            <Link to="/todos" activeStyle={underline}>all</Link>
            <Link to="/todos/active" activeStyle={underline}>active</Link>
            <Link to="/todos/done" activeStyle={underline}>done</Link>
          </div>
        </div>
      </div>
    );
  }
}

export default connect((state, props) => ({
  todos: todosSelector(state, props),
  phase: state.todo.phase,
}), todoActions)(Todos);
```

This example component has everything you will commonly see in smart components:
* Side effect via one of React’s [lifecycle hooks](https://reactjs.org/docs/react-component.html#the-component-lifecycle)
* Reacting to user inputs via handler functions
* Supplies dumb components with the necessary data and callbacks

A good analogy for this dumb and smart separation can be taken from Haskell — pure and impure. Pure components are the dumb ones that simply spit out markup, impure ones kind of lie in the IO monad, they take care of all the side effects and interactions.

Let’s see a dumb component in action:
```
import React, { PropTypes, PureComponent } from 'react';

import Todo from '../../../universal/containers/Todo';


const doneStyle = {
  textDecoration: 'line-through',
};

const notDoneStyle = {
  textDecoration: 'none',
};

export default class TodoItem extends PureComponent {
  static propTypes = {
    todo: PropTypes.instanceOf(Todo).isRequired,
    index: PropTypes.number.isRequired,
    onEdit: PropTypes.func.isRequired,
    onDelete: PropTypes.func.isRequired,
  };

  constructor(props) {
    super(props);

    this.handleToggle = this.handleToggle.bind(this);
    this.handleDelete = this.handleDelete.bind(this);
  }

  handleToggle() {
    const { todo, index, onEdit } = this.props;

    onEdit(todo.set('done', !todo.done), index);
  }

  handleDelete() {
    const { todo, index, onDelete } = this.props;

    onDelete(todo.id, index);
  }

  render() {
    const { todo, index } = this.props;

    return (
      <div
        className="TodoItem"
        style={todo.done ? doneStyle : notDoneStyle}
        onClick={this.handleToggle}
      >
        <span className="TodoItem-text">{todo.text}</span>
        <span className="TodoItem-delete-icon" onClick={this.handleDelete} />
      </div>
    );
  }
}
```

All it does is display todo’s text and an icon. There are no direct side effects, no lifecycle hooks, only content.

Although it isn’t a functional component because of the event handlers it provides, it’s still considered dumb because it only propagates the event upwards for its parent component to consume and cause effects.

You will often encounter dumb components that are functional. If we didn’t have any callbacks, our dumb component could look a lot dumber:
```
import React from 'react';

import Todo from '../../../universal/containers/Todo';

const doneStyle = {
  textDecoration: 'line-through',
};

const notDoneStyle = {
  textDecoration: 'none',
};

const DumbTodo = ({ todo }) => (
  <div
    className="TodoItem"
    style={todo.done ? doneStyle : notDoneStyle}
  >
    <span className="TodoItem-text">{todo.text}</span>
  </div>
);

export default DumbTodo;
```

Basically the only time you want your dumb components to be classes is when you have to partially apply a callback, but more on that later.

###Summary
Smart components are the brain, dumb components are the muscles! This kind of separation will allow you to build more reusable (pure) components, while keeping the interactions and side effects centralized.

##The library approach
The library approach, as I call it, consists of two important rules to remember.

###Functions first
When you design components, always start with the assumption that your component can be functional. Functional components are much more lightweight and will see an improved performance in future React versions.

You only need a class when:

* You need internal state
* You need to use one of React’s [lifecycle hooks](https://reactjs.org/docs/react-component.html#the-component-lifecycle)
* You need to supply callbacks with additional parameters

The first two are only really needed for your smart components. The only exception is if you need to have some kind of small UI state in your component. The third point is also seen more often in dumb components— you often want to propagate some kind of information to the overlord smart component via a callback, thus a class is suitable in these cases.

Writing many stateless components will also greatly ease your debugging sessions. How many times did you have to debug the state of a component for being wrong, something wild coming as a prop, lifecycle hook doing wild stuff, or for not doing anything at all?

On the contrary, only two things can actually go wrong with functional components. You either supplied bad props to the component, or your component rendered the wrong way. Both cases can easily be avoided using a [type checker](https://www.typescriptlang.org/) of your choice and, of course, [testing](https://github.com/airbnb/enzyme).

###Contextless
It should get clearer why I call this the library approach in this second rule. Since most of the time you’ll spend creating dumb components, it is a good idea to keep them only dependant on the properties they receive. This means no connect, no inject and ideally no touching the context at all.

The thing is, no matter what state container you use, your components will always behave the same. Another benefit is that when using something like a [storybook](https://github.com/storybooks/storybook), you don’t have to mock your context every time you want to add a new component. The list goes further:

* Better modularity
* Easier documentation
* Global state independence
* Testability

Let’s have ourselves an example, shall we?
```
import React from 'react';
import { connect } from 'react-redux';
import { injectIntl } from 'react-intl';

import messages from './messages';

const TodoList = props => (
  <div className="TodoList">
    <h3>{props.intl.formatMessage(message.listHeader)}</h3>
    {props.todos.map(todo => (
      <div className="TodoList-item" key={todo.id}>
        {todo.text}
      </div>
    ))}
  </div>
);

export default injectIntl(connect(state => ({
  todos: state.todos.list,
})(TextField));
```

A very simple, yet rigid component! Notice that the header will always be the same, as will the todo’s data source. One would have to mock two different contexts to test this component.

What can we do? Simply let smart components propagate the necessary data via props!
```
// @flow
import React from 'react';

import Todo from '../containers/Todo';

type Props = {
  header: string,
  todos: Todo[],
}

const TodoList = (props: Props) => (
  <div className="TodoList">
    <h3>{props.header}</h3>
    {props.todos.map(todo => (
      <div className="TodoList-item" key={todo.id}>
        {todo.text}
      </div>
    ))}
  </div>
);

export default TextField;
```

Now we don’t care if we use i18n or where the data comes from. We simply render the component as it is — no huge wrappers, no context mocking. Want to put it into a storybook for designers? Test it painlessly with Enzyme? Adjust header dynamically via props? Go ahead!

###Summary
The library approach is literally writing your components as if they were meant to be used as a library — contextless, reusable, small, performant. You will eventually develop a set of nice and simple components for your team to use.

##Mind your re-renders
In a previous section we discussed supplying callback functions with additional data. Many people tend to either use some kind of partial application (bind, Ramda, lodash…), or arrow functions, but this is a [big antipattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)! The same applies to creating new objects or arrays. They create a new reference every time, often causing unnecessary re-rendering.

Let’s have a look at a bad example first.
```
// WRONG!
// Always use a class/memoization for partial application
// and keep props flat, or implement 'shouldComponentUpdate'.
import React from 'react';

import CountryIcon from './CountryIcon';
import Box from './Box';

const BadTodo = props => (
  <Box onClick={() => props.onClick(props.todo.id)}>
    <h4>{props.text}</h4>
    <CountryIcon
      geo={{ lat: props.lat, lon: props.lon }}  
    />
  </Box>
);

export default BadTodo;
```

Notice that we create a new onClick function and a new geo object every render. If you used some kind of [performance library](https://github.com/garbles/why-did-you-update) it would eat you alive and ensure your government wiped away any details of your existence.

To fix the onClick function, simply transform the component to a class and bind in the construction phase by one of the [common ways](https://daveceddia.com/avoid-bind-when-passing-props/).

The geo object is a bit trickier to fix. If the component eating the prop is yours, just spread it flat to avoid any issues whatsoever — since shallow compare will be sufficient. However, if you must have it as an object, you will have to implement shouldComponentUpdate yourself if [it’s worth it](http://wiki.c2.com/?PrematureOptimization).

Note the big highlight on the “if it’s worth it” part of the sentence. When an optimization like this is done in a very difficult or awkward way, simply ignore this specific case and move on, unless you absolutely must make something turbo fast. Measure first!

A cleansed version of the previous component:
```
import React, { PureComponent } from 'react';

import CountryIcon from './CountryIcon';
import Box from './Box';

export default class TurboTodo extends PureComponent {
  handleClick = () => {
    const { todo, onClick } = this.props;
    
    onClick(todo.id);
  }
  
  render() {
    const { text, lat, lon } = this.props;
    
    return (
      <Box onClick={this.handleClick}>
        <h4>{text}</h4>
        <CountryIcon
          lat={lat}
          lon={lon}
        />
      </Box>
    )
  }
}
```

Now, a re-render of CountryIcon won’t be triggered unless some properties actually change.

If you really had to keep the geo prop, you could implement an update on the CountryIcon component yourself:
```
// ... code
shouldComponentUpdate(nextProps) {
  const { geo, ...rest } = this.props;
   
  const geoOk = geo.lat === nextProps.geo.lat && geo.lon === nextProps.geo.lon;
  return !(geoOk && Object.keys(rest)
    .reduce((acc, key) => acc && rest[key] === nextProps[key], true));
}
// ... code
```

If you couldn’t adjust the CountryIcon component (say it was an unoptimized library), there’s one more approach that’s worth mentioning:
```
import React, { PureComponent } from 'react';

import CountryIcon from './CountryIcon';
import Box from './Box';

export default class TurboTodo extends PureComponent {
  constructor(props) {
    super(props);
    
    this.geo = { lat: props.lat, lon: props.lon };
  }
  
  componentWillReceiveProps(nextProps) {
    const { lat, lon } = this.props;
    
    if (lat !== nextProps.lat || lon !== nextProps.lon) {
      this.geo = { lat: nextProps.lat, lon: nextProps.lon };
    }
  }
  
  handleClick = () => {
    const { todo, onClick } = this.props;
    
    onClick(todo.id);
  }
  
  render() {
    const { text, lat, lon } = this.props;
    
    return (
      <Box onClick={this.handleClick}>
        <h4>{text}</h4>
        <CountryIcon
          geo={this.geo}
        />
      </Box>
    )
  }
}
```

This way, the geo object is created at mount time, then re-created only when really necessary. This, in my opinion, is the best way of handling such situation in case you simply cannot change the component receiving non-primitive props (CountryIcon in this case).

###Summary
When it comes to performance, remember these rules:
* Make functions and static objects only during the construction phase
* Keep props flat, ideally comparable via ===
* Measure first, then optimize if it’s worth it
* Construct dynamic objects only when necessary, componentWillReceiveProps helps a lot here
* Implement shouldComponentUpdate if you must optimize, and you cannot change your prop signature

##Remove logic from components
I often encounter this when doing CRs. When you have a component that receives some properties, you often need to perform some operations in order to get the data you actually care about.

It may seem like an overhead, but trust me, it’s totally worth it in the long run. Keep your components like this:
```
import React, { Component } from 'react';

const FnComponent = props => (
  // ...your JSX
);

class ClassComponent extends Component {
  // ...lifecycle hooks
  
  // ...handlers
  
  render() {
    return (
      // ...your JSX
    );
  }
}
```

Notice I explicitly wrote what kind of functions to place in a class component. This is where many people tend to put a lot of ballast (including me in the past, don’t worry).

Anyway, my point is, no computations. No if-else chains, all you care about in a component is the markup (and the side effects in smart components). Where to locate stuff like data selection, computations and such, then?

Task: create a component that displays todos based on a filter in the url.

We now know not to put the filtering directly on the component. While there’s no cookie-cutter answer on where it should take place, my big recommendation is to simply follow the conventions. Meet [reselect](https://github.com/reduxjs/reselect):
```
import { createSelector } from 'reselect';

// sub-selectors
const allTodoSelector = state => state.todo.todos;  // an array of todos
const filterSelector = (_, props) => props.params.filter;  // a url parameter

// a selector
const todoSelector = createSelector(
  [allTodoSelector, filterSelector],
  (todos, filter = 'all') => {
    switch (filter) {
      case 'all':
        return todos;

      case 'active':
        return todos.filter(todo => !todo.done);

      case 'done':
        return todos.filter(todo => todo.done);

      default:
        return todos;
    }
  },
);

export default todoSelector;
```

Alright, we have ourselves a selector! A selector in this context is basically a memoized function that returns some data derived from its input. Check the documentation in case you’re not yet comfortable with the API.

Now, how do we actually use such selector? Since one of the most common use cases is to derive data from Redux’s state, it may be a good idea to place it into the connect function:
```
// ...your component

export default connect((state, props) => ({
  todos: todosSelector(state, props),
}), todoActions)(Todos);
```

Pretty straightforward. Not only is our code neater, we can test it a lot easier, too. But selectors don’t stop there! We can also create ourselves a selector for connect-less components, [selecting only from props](https://github.com/reduxjs/reselect#q-can-i-use-reselect-without-redux):
```
import { createSelector } from 'reselect';

const allTodoSelector = props => props.todos;  // an array of todos
const filterSelector = props => props.filter;  // a url parameter

const todoSelector = createSelector(
  [allTodoSelector, filterSelector],
  (todos, filter = 'all') => {
    switch (filter) {
      case 'all':
        return todos;

      case 'active':
        return todos.filter(todo => !todo.done);

      case 'done':
        return todos.filter(todo => todo.done);

      default:
        return todos;
    }
  },
);

export default todoSelector;
```

And this is how we use it:
```
// @flow
import React from 'react';

import Todo from '../containers/Todo';

import todoSelector from './todoSelector';

type Props = {
  filter: string,
  header: string,
  todos: Todo[],
}

const TodoList = (props: Props) => (
  <div className="TodoList">
    <h3>{props.header}</h3>
    {todoSelector(props).map(todo => (
      <div className="TodoList-item" key={todo.id}>
        {todo.text}
      </div>
    ))}
  </div>
);

export default TextField;
```

That’s all there is to it! Keep in mind you don’t have to use reselect, you can simply make your own functions that take props and spit out filtered todos. Or use some other library — it’s up to you.

It’s not only about selecting derived data. There are many more use cases of separating logic from components:
* API calls
* API response mappers
* Validation functions
* Different browser-based side effects (setting cookies, local storage, … )

###Summary
Let your components do what they are designed to do — spit out markup and handle user interactions. Place any logic in a separate file, where it can be tested a lot more easily, and allow for certain optimization techniques, such as memoization.

##Source 
[https://code.kiwi.com/the-top-5-of-effective-react-984e54cceac3](https://code.kiwi.com/the-top-5-of-effective-react-984e54cceac3)

#HOC
##Higher order components
Now that mixins are dead and not supported in ES6 Class components we should look for a different approach.

What is a higher order component?
```
PassData({ foo: 'bar' })(MyComponent)
```

Basically, you compose a new component from your original one and extend its behaviour. You can use it in various situations like authentication: requireAuth({ role: 'admin' })(MyComponent) (check for a user in higher component and redirect if the user is not logged in) or connecting your component with Flux/Redux store.

At RisingStack, we also like to separate data fetching and controller-like logic to higher order components and keep our views as simple as possible.

###Source
[https://blog.risingstack.com/react-js-best-practices-for-2016/](https://blog.risingstack.com/react-js-best-practices-for-2016/)

##Extract common aspects into higher-order components
Components can often become polluted by cross-cutting concerns that aren’t directly related to its main purpose.

Suppose you wanted to send analytics data whenever a link in a Document component is clicked. To further complicate things, the data needs to include information about the document such as its ID. The obvious solution might be to add code to Document’s componentDidMount and componentWillUnmount lifecycle methods, like so:
```
class Document extends React.Component {
  componentDidMount() {
    ReactDOM.findDOMNode(this).addEventListener('click', this.onClick);
  }
  
  componentWillUnmount() {
    ReactDOM.findDOMNode(this).removeEventListener('click', this.onClick);
  }
  
  onClick = (e) => {
    if (e.target.tagName === 'A') { // Naive check for <a> elements
      sendAnalytics('link clicked', {
        documentId: this.props.documentId // Specific information to be sent
      });
    }
  };
  
  render() {
    // ...
  }
}
```

There are a few problems with this:
1. The component now has an extra concern that obscures its main purpose: displaying a document.
2. If the component has additional logic in those lifecycle methods, the analytics code also becomes obscured.
3. The analytics code is not reusable.
4. Refactoring the component is made harder, as you’d have to work around the analytics code.

Decomposition of aspects such as this one can be done using [higher-order components](https://reactjs.org/docs/higher-order-components.html) (HOCs). In short, these are functions that can be applied to any React component, wrapping that component with a desired behaviour.
```
function withLinkAnalytics(mapPropsToData, WrappedComponent) {
  class LinkAnalyticsWrapper extends React.Component {
    componentDidMount() {
      ReactDOM.findDOMNode(this).addEventListener('click', this.onClick);
    }

    componentWillUnmount() {
      ReactDOM.findDOMNode(this).removeEventListener('click', this.onClick);
    }

    onClick = (e) => {
      if (e.target.tagName === 'A') { // Naive check for <a> elements
        const data = mapPropsToData ? mapPropsToData(this.props) : {};
        sendAnalytics('link clicked', data);
      }
    };
    
    render() {
      // Simply render the WrappedComponent with all props
      return <WrappedComponent {...this.props} />;
    }
  }
  
  return LinkAnalyticsWrapper;
}
```

It’s important to note that this function doesn’t mutate the component to add its behaviour, but returns a new wrapping component. It is this new wrapping component that is used in place of the original Document component.
```
class Document extends React.Component {
  render() {
    // ...
  }
}

export default withLinkAnalytics((props) => ({
  documentId: props.documentId
}), Document);
```

Notice that the specific detail of what data to send (documentId) can be extracted as configuration for the HOC. This keeps the domain knowledge of documents with the Document component, and the generic ability to listen to clicks in the withLinkAnalytics HOC.

Higher-order components show off the powerful compositional nature of React. The simple example here demonstrates how seemingly tightly-integrated code can be extracted into modules with single responsibilities.

HOCs are commonly employed in React libraries such as [react-redux](https://github.com/reduxjs/react-redux), [styled-components](https://github.com/styled-components/styled-components), and [react-intl](https://github.com/yahoo/react-intl). After all, these libraries are all about solving generic aspects of any React application. Another library, [recompose](https://github.com/acdlite/recompose/), goes a step further by using HOCs for everything from component state to lifecycle methods.

###Source
[https://medium.com/dailyjs/techniques-for-decomposing-react-components-e8a1081ef5da](https://medium.com/dailyjs/techniques-for-decomposing-react-components-e8a1081ef5da)