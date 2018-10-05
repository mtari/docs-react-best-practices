#Routing
Almost every client side application has some routing. If you are using React.js in a browser, you will reach the point when you should pick a library.

Our chosen one is the [react-router](https://github.com/ReactTraining/react-router) by the excellent [rackt](https://github.com/rackt) community. Rackt always ships quality resources for React.js lovers.

To integrate react-router check out their [documentation](https://github.com/ReactTraining/react-router), but what's more important here: if you use Flux/Redux we recommend to keep your router's state in sync with your store/global state.

Synchronized router states will help you to control router behaviors by Flux/Redux actions and read router states and parameters in your components.

Redux users can simply do it with the [redux-simple-router](https://github.com/reactjs/react-router-reduxs) library.

##Code splitting, lazy loading
Only a few of webpack users know that it's possible to split your application’s code to separate the bundler's output to multiple JavaScript chunks:

```
require.ensure([], () => {
  const Profile = require('./Profile.js')
  this.setState({
    currentComponent: Profile
  })
})
```
It can be extremely useful in large applications because the user's browser doesn't have to download rarely used codes like the profile page after every deploy.

Having more chunks will cause more HTTP requests - but that’s not a problem with [HTTP/2 multiplexed](https://http2.github.io/faq/#why-is-http2-multiplexed).

Combining with [chunk hashing](https://survivejs.com/webpack/optimizing/adding-hashes-to-filenames/) you can also optimize your cache hit ratio after code changes.

The next version of react-router will help a lot in code splitting.

For the future of react-router check out this blog post by [Ryan Florence](https://twitter.com/ryanflorence): [Welcome to Future of Web Application Delivery](https://medium.com/@ryanflorence/welcome-to-future-of-web-application-delivery-9750b7564d9f).

##Source
[https://blog.risingstack.com/react-js-best-practices-for-2016/](https://blog.risingstack.com/react-js-best-practices-for-2016/)