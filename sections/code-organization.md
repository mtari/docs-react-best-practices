#Code Organization
The class definition for can be organized as follows:

* class definition
  * defaultProps
  * propTypes
  * statedef
  * constructor
    * event-handlers
  * component life-cycle events
  * getters
  * setters

Let’s look at an example:

```
class Car extends React.Component {
  defaultProps = {
    make: 'Toyota'
  };
  
  propTypes = {
    make: React.PropTypes.string
  };

  constructor (props) {
    super(props);
 
    this.state = { running: false };
 
    this.handleClick = () => {
      this.setState({running: !this.state.running});
    };
  }
 
  componentWillMount () {
    // add event listeners (Flux Store, WebSocket, document, etc.)
  },
 
  componentDidMount () {
    // React.getDOMNode()
  },
 
  componentWillUnmount () {
    // remove event listeners (Flux Store, WebSocket, document, etc.)
  },
 
  get engineStatus () {
    return (this.state.running) ? "is running" : "is off";
  }
 
  render () {
    return (
      <div onClick={this.handleClick}>
        {this.props.make} {this.engineStatus}
      </div>
    );
  },
}
```

##Source 

(http://seanamarasinghe.com/developer/react-best-practices-patterns/)[http://seanamarasinghe.com/developer/react-best-practices-patterns/]