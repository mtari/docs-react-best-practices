#How to Use Dependency Injection in React.js Application?

Dependency injection is one of the best software design practices that we have at our disposal as JavaScript developers. By using dependency injections we can avoid a lot of duplicated code in the application, mostly regarding services initialization and configuration.

In the example below, I used a depenedecy injection library called [Jimple](https://github.com/fjorgemota/jimple) to manage the injection of my service layer code into components. But first, I need to make sure the Jimple container is injected correctly on each React.js component.

Let’s take a look first on container definition:
```
import Jimple from 'jimple';

import { UsersService } from './services/UsersService';

export let container = new Jimple();

container.set('API_URL', function (c) {
  return 'http://localhost:8080/api';
});

container.set('API_VERSION', function (c) {
  return '1.0.0';
});

container.set('USERS_SERVICE', function (c) {
  return new UsersService(c.get('API_URL'), c.get('API_VERSION'));
});
```

Next, on the Router component we need to inject the container on each Component rendered by its routes. For this we defined a function called createElement which allows us to forward the container to components as a property:
```
import { Router, IndexRedirect, Route, History } from 'react-router';

import { container } from './container';

class MoneyApp extends React.Component {
  render() {
    return <div>{this.props.children}</div>
  }
}

// Pass container as prop to children components
function createElement(Component, props) {
  return <Component container={container} {...props}/>
}

render((
<Router history={history} createElement={createElement}>
  <Route path="/" component={MyApp}>
    <Route path="/login" component={Login} />
    <Route path="/dashboard" component={Dashboard}>
      <Route path="/dashboard/overview" component={Overview} />
    </Route>
    <IndexRedirect to="dashboard" />
  </Route>
</Router>),
document.getElementById('app'));
```

In the end, we can retrieve the dependencies like in the example below:
```
class Overview extends React.Component {
  constructor() {
    super();
    this.state = { pending: [], lastMonths: [] }
  }
  componentDidMount() {
    let service = this.props.container.get('SERVICE');
    service.yourMethod();
  }
  render() {
	return <div><span>Some content</span></div>
  }
}
```

##Source
[https://www.toptal.com/react/tips-and-practices](https://www.toptal.com/react/tips-and-practices)

#Use React’s context for Global Configuration

React Reform relies on global configuration to make themes and validations easily accessible.
The old way of doing this looked like this:
```
const themes = {};

export function registerTheme(name, theme) {
  themes[name] = theme;
}

export function getTheme(name) {
  return themes[name];
}
```

I.e. creating a global register within the library module. This isn’t without disadvantages. Especially in the context of server-side-rendering.
When rewriting React Reform, I chose a path much closer to how a lot of other react libraries solve this issue: by creating a component that’s supposed to sit at the top of your app. You pass the configuration as props to this component and all it’s doing is to create a context that all your forms have access to.
So here’s the new code in a slightly simplified version:
```
import React from 'react'

export default class ReformContext extends React.Component {

  static childContextTypes = {
    reformRoot: React.PropTypes.object.isRequired
  }

  getChildContext() {
    return {
      reformRoot: {
        getTheme: (name) => this.props.themes[name],
        getValidation: (name) => this.props.validations[name],
      }
    }
  }

  render() {
    return this.props.children
  }
}
```

You apply it like this:
```
class App extends Component {

  render() {
    return (
      <ReformContext themes={themes} validations={validations}>
        <div>
          ...Your App Code...
        </div>
      </ReformContext>
    )
  }
}
```

This approach comes with some benefits:
* Server Side Rendering works out of the box since we’re not storing anything in some global register but only within the tree we’re rendering right now.
* Granularity: if you know that you’re going to use different types of default themes within different parts of of your application, you’re free to go further down your component tree and create several ReformContext’s at different branches.
* Composability: Should you want to overwrite some configuration settings for specific branches of your tree, it’s quite straight forward to create a ReformOverwriteContext component that takes the existing configuration, merges the changes, and pass it down to its subtree. (Note that this particular feature is not implemented in React Reform yet)

##Source
[https://www.codecks.io/blog/2017/lessons-learnt-rewriting-react-lib/](https://www.codecks.io/blog/2017/lessons-learnt-rewriting-react-lib/)