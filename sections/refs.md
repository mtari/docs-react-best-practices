#Refs
* Avoid using refs as much as possible
  * Why? Using refs makes the implementation more brittle. The refd component can not be wrapped without custom logic.
  * Having a nested ref means that section of your render can not easily be refactored out into its own component.

* If you must use one (to get the dom node, for example), it must be to the top component in your render. Don't ever use refs of a nested component. If you need the ref of a child, extract it out to its own component, then pass a prop/callback to it.

* DO NOT use string refs. All refs should use the callback style.

* [WHY?](https://news.ycombinator.com/edit?id=12093234)
  * String refs are not composable. A wrapping component can’t “snoop” on a ref to a child if it already has an existing string ref. On the other hand, callback refs don’t have a single owner, so you can always compose them.
  * String refs don’t work with static analysis like Flow
  * The owner for a string ref is determined by the currently executing component. This means that with a common “render callback” pattern (e.g. <DataTable renderRow={this.renderRow} />), the wrong component will own the ref (it will end up on DataTable instead of your component defining renderRow).

* DO NOT use findDOMNode()
  * use callback refs instead [abramov tweet](https://twitter.com/dan_abramov/status/752936646602031104)

##Source
[https://github.com/kylpo/react-playbook/blob/master/best-practices/react.md](https://github.com/kylpo/react-playbook/blob/master/best-practices/react.md)

#Avoid Refs
Refs will only make your code harder to maintain. Plus when you use refs you are manipulating the virtual Dom directly. Which means that the component will have to re-render the whole Dom tree.

##Source
[https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21)