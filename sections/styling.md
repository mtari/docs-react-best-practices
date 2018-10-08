Avoid className and Style in Your Components
============================================

You want your application to have a consistent look and feel, so you don’t want to pass around classNames and styles in an ad-hoc manner. Having the possibility to pass any style or className to a component means that:

You have to consider what style it needs to make it look the way you want.
You have to know the default set of styles, in order to know the styles from first point.
You can pass any kind of styles that don’t make sense, and may break your UI.
Let’s see what I’m talking about with some examples.

Let’s render a ‘primary’ button to start:
```
<button type=”button” className=”btn btn-primary”>Press me!</button>
```

In the above example, we have to know that we need to pass the classes btn and btn-primary. Also, we could pass any other className that would not make sense, like make-this-look-like-a-pig.

Instead, consider the following code snippet:
```
<Button primary>Press me!</Button>
```

In the code example above we use a custom Button component that internally may use the first approach, but just exposes a primary Boolean property like this:
```
class Button extends React.Component {
  render() {
    // Using https://github.com/JedWatson/classnames
    var className = classNames(‘btn’, this.props.primary && ‘btn-primary’);
    return <button type=”button” className={className} onClick={this.props.onClick}>
      {this.props.children}
    </button>;
  }
}
```

Let’s consider another example. We want to render the main content on the left, and the sidebar on the right:
```
<div style={{float: ‘left’, width: ‘80%’}}>
  Main content
</div>
<div style={{float: ‘left’, width: ‘20%’}}>
  Sidebar content
</div>
```

What if we want to use flexbox? We need to come here and modify the styles we are passing to the div. Also, we can pass any other style that we want, including styles that don’t make any sense or that we don’t intend someone to use.

Now instead, take a look at the following code:
```
<Main>
  Main content
</Main>
<Sidebar>
  Sidebar content
</Sidebar>
```

We wrapped that piece of layout into separate components, which hide that style implementation detail away from us, and we just use them. This particular piece of code doesn’t even care if the sidebar is on the right or on the left.

Another example, showing a successful notification:
```
<div className=”notification notification-success”>
  A good notification
</div>
```

Same problem as the first example. We need to know that those classes need to be used.
```
<Notification type=”success”>
  A good notification
</Notification> 
```

Using a custom notification component that accepts a type, and which can be validated via React.PropTypes to just be any value of a predefined set makes the user of this component not need to remember any class convention.

One final example. Let’s see how we could simplify layouting a form by defining a Layout component that specifies the width of its children.

Here we first layout out three inputs in the same row, where the first two have equal size and the third one is half the size of the others, with the row taking full width of the container. Then we render another row with a button taking full width.
```
<Layout sizes={[1, 1, 0.5]}>
  <Input placeholder=”First Name” />
  <Input placeholder=”Last Name” />
  <Input placeholder=”Age” />
</Layout>
<Layout>
  <Button>Submit</Button>
</Layout>
```

Please notice how avoiding styles and classNames forces you to separate styling concerns like how it looks vs how it lays out, which means you’ll have different components to enforce each. In this last example, the Input and Buttons have no idea how they will be laid out. That’s the job for a parent component.

Think about components’ properties as their API. Exposing styles or classNames is like exposing an ‘options’ object which can have hundreds of options. Do you really want that for your own components? Do you really want to think about each possible option when you use them?

It’s better to leave styles and className to be used in your leaf-most components, and generate a set of components that will build your look and feel, and another set that will define your layout.

When you do this, it will be much more simple to create new components, views, and pages of your application, since you’ll just need to arrange your set of appearance and layout components and pass them the correct props without even thinking about CSS or styles.

And as an added benefit, if you want to tweak the styles or change the look and feel of your application, you will mostly just need to modify the library components. Also if you keep the same API for them, the components that use them will remain untouched.

##Source
[https://www.toptal.com/react/tips-and-practices](https://www.toptal.com/react/tips-and-practices)

Reuse through React instead of CSS
==================================
The final scaling tip that we want to share in this post is this: ensure that React components are your primary unit of reuse. Every one of our React components has an associated CSS file. Some of our components don’t even have any JS interactions or functionality. They’re just bundles of markup and styles.

We’ve been staying away from Bootstrap-like global styles through class names. You can definitely still use Bootstrap, but wrapping the Bootstrap components in your own React components will save you time in the long run. For example, having an Icon React component that encapsulates that markup and accepts the icon name as a prop is better than having to remember exactly what markup and class names to use for icons, which in turn makes refactoring easier. It also makes it easier to add functionality to these components later on.

Although we do define a few global styles for elements such as anchors and headings, as well as many global SCSS variables, we don’t really define global css classes. Being careful to reuse UI mostly through React components has made us more productive as a team because the code is more consistent and predictable.

And that’s about it! These have been some of the guiding principles that helped us build a robust React architecture that has scaled with the size of our engineering team and the complexity of the application. Please feel free to comment with your thoughts and experiences with the ideas in this post.

##Source
[https://engineering.siftscience.com/best-practices-for-building-large-react-applications/](https://engineering.siftscience.com/best-practices-for-building-large-react-applications/)