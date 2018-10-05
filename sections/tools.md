#Use Webpack
The decision to use Webpack is simple: Hot reloading, minified files, node modules :), and you can split your applications into small pieces and lazy load them.

If you are planing on building a large scale applcation I recommend reading [this article](https://medium.com/@ryanflorence/welcome-to-future-of-web-application-delivery-9750b7564d9f) to understand how lazy loading works.

##Source

[https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21)

#Use JSX
If you come from a web development background [JSX](https://reactjs.org/docs/jsx-in-depth.html) will feel very natural. But if your background is not in web develop don’t worry too much, JSX is very easy to learn. Note that if don’t you don’t use JSX the application will be harder to maintain.

##Source

[https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21)

#Static analysis is your friend
In a world of constantly growing single page applications written in a language that lets you do all kinds of crazy stuff, the need to statically check different kinds of code quality is higher than ever. Also, how satisfying is this?!

##Source

[https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21)

##ESlint
ESlint is the hot one now. With Airbnb having one of the most widely accepted set of rules, and more and more editors directly supporting error highlighting, we are getting to a spot where writing high quality code is totally painless.

There’s often a problem, though. With so many rules available, it’s hard to actually agree on what style should be accepted as the correct one. That’s why, in my humble opinion, the best approach is just follow the most widely accepted standard, which is the aforementioned Airbnb styleguide.

They have rules for React, import, ES6+ syntax, JSX … basically all you would ever want to use in this stack. It even checks for potential HTML issues!

Whether your team consists solely of you, or you work in a company of 100 front-end developers, having a unified, widely accepted code style goes a long way.

##Flow/TypeScript
I don’t want to be biased here, so I mentioned both options. Types are a godsend. If you’ve ever programmed in a language with a good type system, you know how much they can help make sure your program works before you actually run the code.

There’s a common saying that if it compiles, it probably runs. Although Flow itself doesn’t involve any compilation, you can check for type errors ahead of time. This means a lot less runtime errors, debugging sessions and headaches!

One huge, although less frequently mentioned, advantage of static typing is that the code you write will actually have some structure. This especially involves getting rid of weird data mappings so typical for JS functions, however, totally unpredictable and impossible to type.
All kinds of common errors that are very hard to track will be caught in advance:

* Mistyped object property
* Mistyped string constant
* Unexpected null or undefined
* Accessing a non-existent nested property
* Functions returning unexpected results

When considering whether types are worth it for you, the answer is almost always a big “Yes”. Starting with types is a lot less painful than trying to incorporate them into an already huge codebase (that’s what I am currently doing, lol). The better the coverage, the better the results!

Now, when it comes to propTypes vs. Flow or TypeScript, personally I’d just go with the static version. propTypes are awesome if you currently don’t use any kind of static typing, however, in a well covered code, they become obsolete.

##Source

[https://code.kiwi.com/the-top-5-of-effective-react-984e54cceac3](https://code.kiwi.com/the-top-5-of-effective-react-984e54cceac3)

#DevTools for React/Redux as Browser plugins
There are developer plugins for both Chrome and Firefox:

React Developer Tools for Chrome
Redux DevTools for Chrome
React Developer Tools for Firefox
Redux Devtools for Firefox
It adds itself as a new tab in the “inspect element” area that you most likely are very familiar with React devtools.

From there you can inspect all your components visually and for each component you can see the props and state. Very convenient!

Both React and Redux devtools are a must haves if you are a React/Redux developer.

So what can you do with them? I use them for mainly two things: debugging and inspecting data.

##Source 

[https://dev.to/jakoblind/how-to-become-a-more-productive-react-developer](https://dev.to/jakoblind/how-to-become-a-more-productive-react-developer)