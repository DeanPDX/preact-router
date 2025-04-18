# preact-cancellable-router

[![NPM](https://img.shields.io/npm/v/preact-cancellable-router.svg)](https://www.npmjs.com/package/preact-cancellable-router)
[![Build status](https://github.com/preactjs/preact-cancellable-router/actions/workflows/node.js.yml/badge.svg)](https://github.com/preactjs/preact-cancellable-router/actions/workflows/node.js.yml)

> [!WARNING]
> `preact-cancellable-router` unfortunately no longer sees active development! It's completely stable and so you can rely upon it for all existing apps, but for newer ones, we'd recommend using [`preact-iso`](https://github.com/preactjs/preact-iso) for your routing needs instead. It offers a very similar API while integrating a bit better Suspense and lazy loading, with potentially more useful hooks. Thanks to all the contributors and users over the years!

Connect your [Preact](https://github.com/preactjs/preact) components up to that address bar.

`preact-cancellable-router` provides a `<Router />` component that conditionally renders its children when the URL matches their `path`. It also automatically wires up `<a />` elements to the router.

> 💁 **Note:** This is not a preact-compatible version of React Router. `preact-cancellable-router` is a simple URL wiring and does no orchestration for you.
>
> If you're looking for more complex solutions like nested routes and view composition, [react-router](https://github.com/ReactTraining/react-router) works great with preact as long as you alias in [preact/compat](https://preactjs.com/guide/v10/getting-started#aliasing-react-to-preact).

#### [See a Real-world Example :arrow_right:](https://jsfiddle.net/developit/qc73v9va/)

---

### Usage Example

```js
import Router from 'preact-cancellable-router';
import { h, render } from 'preact';
/** @jsx h */

const Main = () => (
  <Router>
    <Home path="/" />
    <About path="/about" />
    // Advanced is an optional query
    <Search path="/search/:query/:advanced?" />
  </Router>
);

render(<Main />, document.body);
```

If there is an error rendering the destination route, a 404 will be displayed.

#### Caveats

Because the `path` and `default` props are used by the router, it's best to avoid using those props for your component(s) as they will conflict.

### Handling URLS

:information_desk_person: Pages are just regular components that get mounted when you navigate to a certain URL.
Any URL parameters get passed to the component as `props`.

Defining what component(s) to load for a given URL is easy and declarative.
Querystring and `:parameter` values are passed to the matched component as props.
Parameters can be made optional by adding a `?`, or turned into a wildcard match by adding `*` (zero or more characters) or `+` (one or more characters):

```js
<Router>
  <A path="/" />
  <B path="/b" id="42" />
  <C path="/c/:id" />
  <C path="/d/:optional?/:params?" />
  <D path="/e/:remaining_path*" />
  <E path="/f/:remaining_path+" />
  <F default />
</Router>
```

### Lazy Loading

Lazy loading (code splitting) with `preact-cancellable-router` can be implemented easily using the [AsyncRoute](https://www.npmjs.com/package/preact-async-route) module:

```js
import AsyncRoute from 'preact-async-route';
<Router>
  <Home path="/" />
  <AsyncRoute
    path="/friends"
    getComponent={() => import('./friends').then(module => module.default)}
  />
  <AsyncRoute
    path="/friends/:id"
    getComponent={() => import('./friend').then(module => module.default)}
    loading={() => <div>loading...</div>}
  />
</Router>;
```

### Active Matching & Links

`preact-cancellable-router` includes an add-on module called `match` that lets you wire your components up to Router changes.

Here's a demo of `<Match>`, which invokes the function you pass it (as its only child) in response to any routing:

```js
import Router from 'preact-cancellable-router';
import Match from 'preact-cancellable-router/match';

render(
  <div>
    <Match path="/">{({ matches, path, url }) => <pre>{url}</pre>}</Match>
    <Router>
      <div default>demo fallback route</div>
    </Router>
  </div>
);

// another example: render only if at a given URL:

render(
  <div>
    <Match path="/">{({ matches }) => matches && <h1>You are Home!</h1>}</Match>
    <Router />
  </div>
);
```

`<Link>` is just a normal link, but it automatically adds and removes an "active" classname to itself based on whether it matches the current URL.

```js
import { Router } from 'preact-cancellable-router';
import { Link } from 'preact-cancellable-router/match';

render(
  <div>
    <nav>
      <Link activeClassName="active" href="/">
        Home
      </Link>
      <Link activeClassName="active" href="/foo">
        Foo
      </Link>
      <Link activeClassName="active" href="/bar">
        Bar
      </Link>
    </nav>
    <Router>
      <div default>this is a demo route that always matches</div>
    </Router>
  </div>
);
```

### Default Link Behavior

Sometimes it's necessary to bypass preact-cancellable-router's link handling and let the browser perform routing on its own.

This can be accomplished by adding a `data-native` boolean attribute to any link:

```html
<a href="/foo" data-native>Foo</a>
```

### Detecting Route Changes

The `Router` notifies you when a change event occurs for a route with the `onChange` callback:

```js
import { render, Component } from 'preact';
import { Router, route } from 'preact-cancellable-router';

class App extends Component {
  // some method that returns a promise
  isAuthenticated() {}

  handleRoute = async e => {
    switch (e.url) {
      case '/profile':
        const isAuthed = await this.isAuthenticated();
        if (!isAuthed) route('/', true);
        break;
    }
  };

  render() {
    return (
      <Router onChange={this.handleRoute}>
        <Home path="/" />
        <Profile path="/profile" />
      </Router>
    );
  }
}
```

### Redirects

Can easily be implemented with a custom `Redirect` component;

```js
import { Component } from 'preact';
import { route } from 'preact-cancellable-router';

export default class Redirect extends Component {
  componentWillMount() {
    route(this.props.to, true);
  }

  render() {
    return null;
  }
}
```

Now to create a redirect within your application, you can add this `Redirect` component to your router;

```js
<Router>
  <Bar path="/bar" />
  <Redirect path="/foo" to="/bar" />
</Router>
```

### Custom History

It's possible to use alternative history bindings, like `/#!/hash-history`:

```js
import { h } from 'preact';
import Router from 'preact-cancellable-router';
import { createHashHistory } from 'history';

const Main = () => (
  <Router history={createHashHistory()}>
    <Home path="/" />
    <About path="/about" />
    <Search path="/search/:query" />
  </Router>
);

render(<Main />, document.body);
```

### Programmatically Triggering Route

It's possible to programmatically trigger a route to a page (like `window.location = '/page-2'`)

```js
import { route } from 'preact-cancellable-router';

route('/page-2'); // appends a history entry

route('/page-3', true); // replaces the current history entry
```

### Nested Routers

The `<Router>` is a self-contained component that renders based on the page URL. When nested a Router inside of another Router, the inner Router does not share or observe the outer's URL or matches. Instead, inner routes must include the full path to be matched against the page's URL:

```js
import { h, render } from 'preact';
import Router from 'preact-cancellable-router';

function Profile(props) {
  // `props.rest` is the rest of the URL after "/profile/"
  return (
    <div>
      <h1>Profile</h1>
      <Router>
        <MyProfile path="/profile/me" />
        <UserProfile path="/profile/:user" />
      </Router>
    </div>
  );
}
const MyProfile = () => <h2>My Profile</h2>;
const UserProfile = props => <h2>{props.user}</h2>;

function App() {
  return (
    <div>
      <Router>
        <Home path="/" />
        <Profile path="/profile/:rest*" />
      </Router>
      <nav>
        <a href="/">Home</a>
        <a href="/profile/me">My Profile</a>
        <a href="/profile/alice">Alice's Profile</a>
      </nav>
    </div>
  );
}

render(<App />, document.body);
```

### `Route` Component

Alternatively to adding the router props (`path`, `default`) directly to your component, you may want to use the `Route` component we provide instead. This tends to appease TypeScript, while still passing down the routing props into your component for use.

```js
import { Router, Route } from 'preact-cancellable-router';

function App() {
  let users = getUsers();

  return (
    <Router>
      <Route path="/" component={Home} />
      {/* Route will accept any props of `component` type */}
      <Route path="/users" component={Users} users={users}   />
    </Router>
  );
}
```

### License

[MIT](./LICENSE)
