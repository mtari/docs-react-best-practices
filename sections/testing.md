Use shallow rendering
=====================
Testing React components is still a bit of a tricky topic. Not because it's hard, but because it's still an evolving area, and no single approach has emerged as the 'best' one yet. At the moment, my go-to method is to use shallow rendering and prop assertions.

Shallow rendering is nice, because it allows you to render a single component completely, but without delving into any of its child components to render those. Instead, the resulting object will tell you things like the type and props of the children. This gives us good isolation, allowing testing of a single component at a time.

There are three types of component unit tests I find myself most commonly writing:

##Render logic
Imagine a component that should conditionally display either an image, or a loading icon:

```
const Image = props => {
  if (props.loading) {
    return <LoadingIcon/>;
  }

  return <img src={props.src}/>;
};
```

We might test it like this:
```
describe('Image', () => {
  it('renders a loading icon when the image is loading', () => {
    const image = shallowRender(<Image loading={true}/>);

    expect(image.type).toEqual(LoadingIcon);
  });

  it('renders the image once it has loaded', () => {
    const image = shallowRender(<Image loading={false} src="https://example.com/image.jpg"/>);

    expect(image.type).toEqual('img');
  });
});
```

Easy! I should point out that the API for shallow rendering is slightly more complicated than what I've shown. The shallowRender function used above is our own helper, which wraps the real API to make it easier to use.

Revisiting our ListOfNumbers component above, here is how we might test that the map is done correctly:
```
describe('ListOfNumbers', () => {
  it('renders an item for each provided number', () => {
    const listOfNumbers = shallowRender(<ListOfNumbers className="red" numbers={[3, 4, 5, 6]}/>);

    expect(listOfNumbers.props.children.length).toEqual(4);
  });
});
```

##Prop transformations
In the last example, we dug into the children of the component being tested, to make sure that they were rendered correctly. We can extend this by asserting that not only are the children there, but that they were given the correct props. This is particulary useful when a component does some transformation on its props, before passing them on. For example, the following component takes CSS class names as an array of strings, and passes them down as a single, space-separated string:
```
const TextWithArrayOfClassNames = props => (
  <div>
    <p className={props.classNames.join(' ')}>
     {props.text}
    </p>
  </div>
);

describe('TextWithArrayOfClassNames', () => {
  it('turns the array into a space-separated string', () => {
    const text = 'Hello, world!';
    const classNames = ['red', 'bold', 'float-right'];
    const textWithArrayOfClassNames = shallowRender(<TextWithArrayOfClassNames text={text} classNames={classNames}/>);

    const childClassNames = textWithArrayOfClassNames.props.children.props.className;
    expect(childClassNames).toEqual('red bold float-right');
  });
});
```

One common criticism of this approach to testing is the proliferation of props.children.props.children... While it's not the prettiest code, personally I find that if I'm being annoyed by writing props.children too much in the one test, that's a sign that the component is too big, complex, or deeply nested, and should be split up.

The other thing I often hear is that your tests become too dependant on the component's internal implementation, so that changing your DOM structure slightly causes all of your tests to break. This is definitely a fair criticism, and a brittle test suite is the last thing that anyone wants. The best way to manage this is to (wait for it) keep your components small and simple, which should limit the number of tests that break due to any one component changing.

##User interaction
Of course, components are not just for display, they're also interactive:
```
const RedInput = props => (
  <input className="red" onChange={props.onChange} />
)
```

Here's my favourite way to test these:

```
describe('RedInput', () => {
  it('passes the event to the given callback when the value changes', () => {
    const callback = jasmine.createSpy();
    const redInput = shallowRender(<RedInput onChange={callback}/>);

    redInput.props.onChange('an event!');
    expect(callback).toHaveBeenCalledWith('an event!');
  });
});
```

It's a bit of a trivial example, but hopefully you get the idea.

##Integration testing
So far I've only covered unit testing components in isolation, but you're also going to want some higher level tests in order to ensure that your application connects up properly and actually works. I'm not going to go into too much detail here, but basically:

1. Render your entire tree of components (instead of shallow rendering).
2. Reach into the DOM (using the React TestUtils, or jQuery, etc) to find the elements you care about the most, and then
  * assert on their HTML attributes or contents, or
  * simulate DOM events and then assert on the side effects (DOM or route changes, AJAX calls, etc)

##Source

[https://camjackson.net/post/9-things-every-reactjs-beginner-should-know#use-shallow-rendering](https://camjackson.net/post/9-things-every-reactjs-beginner-should-know#use-shallow-rendering)

Use enzyme
==========
One of our favorite library for component testing is [enzyme](https://github.com/airbnb/enzyme) by AirBnb. With it's shallow rendering feature you can test logic and rendering output of your components, which is pretty amazing. It still cannot replace your selenium tests, but you can step up to a new level of frontend testing with it.
```
it('simulates click events', () => {
  const onButtonClick = sinon.spy()
  const wrapper = shallow(
    <Foo onButtonClick={onButtonClick} />
  )
  wrapper.find('button').simulate('click')
  expect(onButtonClick.calledOnce).to.be.true
})
```
Looks neat, isn't it?

Do you use chai as assertion library? You will like [chai-enyzime](https://github.com/producthunt/chai-enzyme)!

##Source

[https://blog.risingstack.com/react-js-best-practices-for-2016/](https://blog.risingstack.com/react-js-best-practices-for-2016/)

Redux testing
=============
Testing a reducer should be easy, it responds to the incoming actions and turns the previous state to a new one:
```
it('should set token', () => {
  const nextState = reducer(undefined, {
    type: USER_SET_TOKEN,
    token: 'my-token'
  })

  // immutable.js state output
  expect(nextState.toJS()).to.be.eql({
    token: 'my-token'
  })
})
```

Testing actions is simple until you start to use async ones. For testing async redux actions we recommend to check out redux-mock-store, it can help a lot.
```
it('should dispatch action', (done) => {
  const getState = {}
  const action = { type: 'ADD_TODO' }
  const expectedActions = [action]
 
  const store = mockStore(getState, expectedActions, done)
  store.dispatch(action)
})
```

For deeper [redux testing](http://redux.js.org/docs/recipes/WritingTests.html) visit the official documentation.

##Source

[https://blog.risingstack.com/react-js-best-practices-for-2016/](https://blog.risingstack.com/react-js-best-practices-for-2016/)

TDD
===

[https://semaphoreci.com/community/tutorials/getting-started-with-tdd-in-react?utm_source=javascriptweekly&utm_medium=email](https://semaphoreci.com/community/tutorials/getting-started-with-tdd-in-react?utm_source=javascriptweekly&utm_medium=email)