#Do more in componentDidUpdate
React is all about turning the imperative task of updating the DOM into a declarative one. It turns out that declaring other types of imperative behavior as a function of props and state can be beneficial as well. Here’s an example:

Let’s say that we’re building an interface to view and edit contacts. In the screenshot to the right, “contact #3” has unsaved changes. We would like the form to automatically save if the user navigates to another contact, say Contact #2, at this point.

A reasonable way to implement this functionality would be to have a method on our main component that looks like this:
```
navigateToContact: function(newContactId) {
  if (this._hasUnsavedChanges()) {
    this._saveChanges();
  }
  
  this.setState({
    currentContactId: newContactId
  });
}
...
navigateToContact(‘contact2’)
```

This setup is kind of fragile though. We have to make sure that the click handlers for both the “contact #2” sidebar item and the “< prev contact” link in the bottom left corner use the navigateToContact method instead of directly setting the currentContactId state.

Here’s what it would look like if we implemented this declaratively using componentDidUpdate.
```
componentDidUpdate: function(prevProps, prevState) {
  if (prevState.currentContactId !== state.currentContactId) {
    if (this._hasUnsavedChanges()) {
      this._saveChanges();
    }
  }
}
```

In this version, the functionality to save the previous contact when navigating to a new one is baked into the component lifecycle. It’s much harder to break now since all event handlers can directly call this.setState({currentContactId: ‘contact2’}) instead of having to know to use a special method.

This example is oversimplified of course. Invoking navigateToContact in both event handlers doesn’t seem too bad here, but the problem becomes more apparent as the component grows in complexity. Declaring actions to be invoked based on prop and state changes will make your components more autonomous and reliable. This technique is especially useful for components that manage a lot of state, and it has made refactorings much more pleasant for us.

##Source
[https://engineering.siftscience.com/best-practices-for-building-large-react-applications/](https://engineering.siftscience.com/best-practices-for-building-large-react-applications/)

#Optimize by understanding the component’s lifecycle
Optimize by understanding the component’s lifecycle
When a component is rendered, React tries to optimize in a way that only the altered state gets modified in the DOM (see virtual DOM). But this doesn’t mean the work is over. You still need to care about how often your render method will be called. Updating the state will always trigger a re-render, and you can prevent unnecessary render calls by adding conditional rendering logic inside of shouldComponentUpdate(). This method is one of several hooks React provides to intercept events that happen in a component’s lifecycle. Knowing these hooks is vital to understanding React, and can help you make better decisions when structuring your component’s logic.

Understanding the lifecycle can help:
* Optimize the rendering process
* Know where to fetch external data and avoid race conditions
* Update the state at the right time
* Take action upon receiving new data from a parent component
* Execute code after an update
* Know when to dispose of a component
* Initialize a container component with state

![Component Lifecycle](/../images/component-lifecycle.png)

##Source
[https://ignaciochavez.com/helpful-principles-starting-react/](https://ignaciochavez.com/helpful-principles-starting-react/)

#Use ShouldComponentUpdate for performance optimization
React is a templating language that renders EVERY TIME the props or the state of the component changes. So imagine having to render the entire page every time there in an action. That takes a big load on the browser. That’s where [ShouldComponentUpdate](https://reactjs.org/docs/react-component.html#updating-shouldcomponentupdate) comes in, whenever React is rendering the view it checks to see if shouldComponentUpdate is returning false/true. So whenever you have a component that’s static do yourself a favor and return false. Or if is not static check to see if the props/state has changed.

If you want to read more on performance optimization read [my article on React Perf](https://medium.com/@nesbtesh/react-performance-optimization-28ec5b61fff3)

##Source
[https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21)