#Use PropTypes
PropTypes help you set data validation for components. This is very useful when debugging and when working with multiple developers. Anyone working with a large team should read [this article](https://reactjs.org/docs/typechecking-with-proptypes.html).

#Use Prop validation
[PropTypes](https://reactjs.org/docs/components-and-props.html) will make your life a lot better when working with a large team. They allow you to seemly debug your components. They will also make your debugging a lot easier. In a way you are setting standard requirements for a specific component.

#PropValidation
Always write PropValidation as precise as you can. Don’t just validate that a prop is an object, validate if the object has the necessary properties. This way a user does not have checkout the source code if an invalid property has been passed, but the warning will state everything necessary.

```
const Coworker = ({ worker }) => (
  <div>
    <h1>{worker.id}</h1>
    <span>{worker.name}</span>
    <p>{worker.duties}</p>
  </div>
)
// bad
const { object } = React.PropTypes
Coworker.propTypes = {
  worker: object.isRequired
}
// better
const { shape, number, string, arrayOf } = React.PropTypes
Coworker.propTypes = {
  worker: shape({
    id: number.isRequired,
    name: string,
    duties: arrayOf(string)
  }).isRequired
}
```

##Source
[https://thoughts.travelperk.com/writing-a-good-react-component-59624ed40b8e](https://thoughts.travelperk.com/writing-a-good-react-component-59624ed40b8e)

#Always use propTypes
propTypes offer us a really easy way to add a bit more type safety to our components. They look like this:
```
const ListOfNumbers = props => (
  <ol className={props.className}>
    {
      props.numbers.map(number => (
        <li>{number}</li>)
      )
    }
  </ol>
);

ListOfNumbers.propTypes = {
  className: React.PropTypes.string.isRequired,
  numbers: React.PropTypes.arrayOf(React.PropTypes.number)
};
```

When in development (not production), if any component is not given a required prop, or is given the wrong type for one of its props, then React will log an error to let you know. This has several benefits:

* It can catch bugs early, by preventing silly mistakes
* If you use isRequired, then you don't need to check for undefined or null as often
* It acts as documentation, saving readers from having to search through a component to find all the props that it needs
The above list looks a bit like one you might see from someone advocating for static typing over dynamic typing. Personally, I usually prefer dynamic typing for the ease and speed of development it provides, but I find that propTypes add a lot of safety to my React components, without making things any more difficult. Frankly I see no reason not to use them all the time.

One final tip is to make your tests fail on any propType errors. The following is a bit of a blunt instrument, but it's simple and it works:
```
beforeAll(() => {
  console.error = error => {
    throw new Error(error);
  };
});
```

##Source
[https://camjackson.net/post/9-things-every-reactjs-beginner-should-know#always-use-proptypes](https://camjackson.net/post/9-things-every-reactjs-beginner-should-know#always-use-proptypes)