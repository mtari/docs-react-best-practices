Split the render() method
=========================
This is the most obvious technique to apply: when a component renders too many elements, splitting those elements into logical sub-components is an easy way to simplify.

A common and quick way to split the render() method is to create additional “sub-render” methods on the same class:
```
class Panel extends React.Component {
  renderHeading() {
    // ...
  }

  renderBody() {
    // ...
  }

  render() {
    return (
      <div>
        {this.renderHeading()}
        {this.renderBody()}
      </div>
    );
  }
}
```

While this can have its place, it is not a true decomposition of the component itself. Any state, props, and class methods are still shared, making it difficult to identify which of those are used by each sub-render.

To really reduce complexity, entirely new components should be created instead. For simpler sub-components, [functional components](https://reactjs.org/docs/components-and-props.html#functional-and-class-components) can be used to keep boilerplate to a minimum:
```
const PanelHeader = (props) => (
  // ...
);

const PanelBody = (props) => (
  // ...
);

class Panel extends React.Component {
  render() {
    return (
      <div>
        // Nice and explicit about which props are used
        <PanelHeader title={this.props.title}/>
        <PanelBody content={this.props.content}/>
      </div>
    );
  }
}
```

There is another subtle but important difference achieved by splitting in this way. By replacing direct function calls with indirect component declarations, we produce smaller units for React to work with. This is because the return value of Panel’s render() is a tree of element descriptors that only go as far as PanelHeader and PanelBody, rather than all the elements beneath them too.

There are practical implications for testing: a [shallow render](https://reactjs.org/docs/shallow-renderer.html) can be used to easily isolate those units for independent testing. As a bonus, when React’s new [Fiber architecture](https://github.com/acdlite/react-fiber-architecture) arrives, the smaller units will allow it to perform incremental rendering more effectively.