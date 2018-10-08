#Templatize components by passing React elements as props

Templatize components by passing React elements as props
If a component is becoming too complex due to multiple variations or configurations, consider turning the component into a simple “template” component with one or more open “slots”. This allows a dedicated parent component to focus on configuration only.

For example, a Comment component may have different actions and metadata displayed depending on whether you are the author, whether it was successfully saved, or what permissions you have. Rather than mix the structure of the Comment component — where and how its content is rendered — with logic to handle all the possible variations, these two concerns can be considered independently. Utilise the ability of React to pass elements, not just data, as props, to create a flexible template component.
```
class CommentTemplate extends React.Component {
  static propTypes = {
    // Declare slots as type node
    metadata: PropTypes.node,
    actions: PropTypes.node,
  };
  
  render() {
    return (
      <div>
        <CommentHeading>
          <Avatar user={...}/>
          
          // Slot for metadata
          <span>{this.props.metadata}</span>
          
        </CommentHeading>
        <CommentBody/>
        <CommentFooter>
          <Timestamp time={...}/>
          
          // Slot for actions
          <span>{this.props.actions}</span>
          
        </CommentFooter>
      </div>
    );
  }
}
```

Another component can then have the sole responsibility of figuring out what to fill the metadata and actions slots with.
```
class Comment extends React.Component {
  render() {
    const metadata = this.props.publishTime ?
      <PublishTime time={this.props.publishTime} /> :
      <span>Saving...</span>;
    
    const actions = [];
    if (this.props.isSignedIn) {
      actions.push(<LikeAction />);
      actions.push(<ReplyAction />);
    }
    if (this.props.isAuthor) {
      actions.push(<DeleteAction />);
    }
    
    return <CommentTemplate metadata={metadata} actions={actions} />;
  }
}
```

Keep in mind that in JSX, anything that’s between the opening and closing tags of a component is passed as the special children prop. This can be particularly expressive when used correctly. To be idiomatic, it should be reserved for the main content area of a component. In the comments example, this would likely be the text of the comment itself.
```
<CommentTemplate metadata={metadata} actions={actions}>
  {text}
</CommentTemplate>
```

##Source 
[https://medium.com/dailyjs/techniques-for-decomposing-react-components-e8a1081ef5da](https://medium.com/dailyjs/techniques-for-decomposing-react-components-e8a1081ef5da)

#Think about inmutability

If you are coming from Scala or other high performance languages inmutability is a concept that you are probably really familiar with. But if you are not familiar with the concept think of immutability like having twins. They are very much alike and they look the same but they are not equal. For example:
```
var object1 = {color: "blue", size: 12};
var object2 = object1
object2.color = "red"

object1 === object2 //true

object3 = Object.assign({}, object1)
object3.color = "red"

object1 === object3 //false
```

What just happened? Object2 was created as a reference of object1 that means that in every sense of the word object2 is another way of referencing object1. When I created object3 I created a new object that has the same structure as object1. The Object.assign function takes a new object and then clones the structure of object1 therefore creating a new reference so when you compare object1 to object3 they are different. Why is this significant? Think of performance optimization, I mentioned above that React renders everytime the state of a component changes. When using the ShouldComponentUpdate function instead of doing a deep check to see if all the attributes are different you can simply compare the objects. If you want to know more keep reading [this article](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/).

##Source
[https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21)