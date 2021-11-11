# React component lifecycle methods

[Youtube video](https://www.youtube.com/watch?v=m_mtV4YaI8c&t=399s)

**Table of contents**

- [React component lifecycle methods](#react-component-lifecycle-methods)
  * [Introduction](#introduction)
  * [constructor](#constructor)
  * [componentDidMount](#componentdidmount)
  * [componentWillUnmount](#componentwillunmount)
  * [componentDidUpdate](#componentdidupdate)
  * [shouldComponentUpdate](#shouldcomponentupdate)
  * [getDerivedStateFromProps](#getderivedstatefromprops)
  * [getSnapshotBeforeUpdate](#getsnapshotbeforeupdate)
  * [componentDidCatch](#componentdidcatch)
  * [getDerivedStateFromError](#getDerivedStateFromError)
  * [Lifecycle visualised](#lifecycle-visualised)
  * [Playground](#playground)

## Introduction

There are **4** major categories of lifecycle hooks/methods in React:

1. **Mounting**
   1. **`constructor(props)`**
   2. **`componentDidMount()`**
2. **Updating**
   1. **`componentDidUpdate(prevProps, prevState)`**
   2. **`shouldComponentUpdate(nextProps, nextState)`**
   3. **`getDerivedStateFromProps(props, state)`**
   4. **`getSnapshotBeforeUpdate(prevProps, prevState)`**
3. **Error boundaries**
   1. **`componentDidCatch(error, info)`**
   2. **`getDerivedStateFromError(error)`**
4. **Unmounting**
   1. **`componentWillUnmount()`**

## constructor

The constructor is invoked the very first time the component is created. It is **not** invoked on re-renders (updates)!

The constructor does 3 things:

1. It invokes `super(props)` which sets the `this.props` to the passed in props
2. It sets the state with `this.state = {...}`
3. It sets instance variables and methods. Ex: `this.instanceMethod = () => {...}`

```jsx
class Counter extends React.Component {
  constructor(props) {
    console.log('Constructor')
    super(props);
    this.state = {
      count: 0
    };
    console.log('-----------')
  }
  render() {
    console.log('render')
    return <div>Count: {this.state.count}</div>;
  }
}
```

```
/* Output on initial script execution */

Constructor 
----------- 
render 
```

Note: 

1. The constructor is invoked **before** the first render
2. The constructor will be invoked (before the first render) everytime the parent re-mounts the component (i.e toggling component)

## componentDidMount

This method runs only once i.e **after the initial render (first render)**

If the parent component shows & hides the component then everytime it is shown, which means that it is mounted fresh, the componentDidMount method is invoked. However, it is **never** invoked on re-renders i.e updates

It is useful to *perform side effect tasks after initial rende*r. Ex: Fetching and loading data from an API and displaying it. If we load the API data before render (i.e in constructor) then the render will be blocked. Therefore, we want the DOM to render first, maybe with a loader in this case, and then re-render it when data becomes available

## componentWillUnmount

This method is invoked **before** the component will unmount!

It is useful to *perform clean up operations* when the component is getting destroyed (unmounted). For example, it is important to **remove any event listeners** in order to avoid **memory leaks**

```jsx
// App.js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      showCounter: true
    };
    this.toggleCounter = () =>
      this.setState({
        showCounter: !this.state.showCounter
      });
  }

  render() {
    const { showCounter } = this.state;
    return (
      <div className="App">
        <button onClick={this.toggleCounter}>Toggle Counter</button>
        {showCounter ? <Counter /> : null}
      </div>
    );
  }
}
```

```jsx
// Counter.js
class Counter extends React.Component {
  constructor(props) {
    console.log("Constructor");
    super(props);
    this.state = {
      count: 0
    };
    console.log("-----------");
  }

  componentDidMount() {
    console.log("Component Did Mount");
    console.log("-----------");
  }

  componentWillUnmount() {
    console.log("Component Will Unmount");
    console.log("-----------");
  }

  render() {
    console.log("render");
    return <div>Count: {this.state.count}</div>;
  }
}
```

```
/* Output on initial script execution */

Constructor 
----------- 
render 
Component Did Mount 
----------- 

/* After clicking toggle button (hide): */

Component Will Unmount 
-----------

/* After clicking toggle button again (show): */

Constructor 
----------- 
render 
Component Did Mount 
----------- 

/* After clicking toggle button a 3rd time (hide): */

Component Will Unmount 
-----------
```

## componentDidUpdate

This method **runs after time the component re-renders**

The component will re-render when:

1. *Props change* 
2. *State changes*

Like `componentDidMount`, `componentDidUpdate` too runs *after* the render takes place!

## shouldComponentUpdate

This method runs **before the component re-renders**. It has the power to determine if a given update (i.e re-render) must take place or not!

If the method returns `true`, update (re-render) takes place (& `componentDidUpdate` lifecycle method executes)

If the method returns `false`, there is no update (re-render) (`componentDidUpdate` does not execute as well)

It receives two parameters: 

1. Next props, &
2. Next state

These two are used in deciding whether we want an update to take effect (re-render) or not!

```jsx
// App.js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      showCounter: true,
      ignoreProp: 0
    };
    this.toggleCounter = () =>
      this.setState({
        showCounter: !this.state.showCounter
      });

    this.generateignoreProp = () => {
      this.setState({
        ignoreProp: Number.parseInt(Math.random() * 100, 10)
      });
    };
  }

  render() {
    const { showCounter, ignoreProp } = this.state;
    return (
      <div className="App">
        <button onClick={this.toggleCounter}>Toggle Counter</button>
        <button onClick={this.generateignoreProp}>
          Generate ignoreProp value
        </button>
        {showCounter ? <Counter ignoreProp={ignoreProp} /> : null}
      </div>
    );
  }
}
```

```jsx
// Counter.js
class Counter extends React.Component {
  constructor(props) {
    console.log("Constructor");
    super(props);
    this.state = {
      count: 0
    };

    this.incCounter = (e) =>
      this.setState({
        count: this.state.count + 1
      });
    this.decCounter = (e) =>
      this.setState({
        count: this.state.count - 1
      });

    console.log("-----------");
  }

  componentDidMount() {
    console.log("Component Did Mount");
    console.log("-----------");
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (nextState.count === this.state.count) {
      console.log("Should Component Update - No Update");
      return false;
    }
    console.log("Should Component Update - Yes, Update");
    return true;
  }

  componentDidUpdate(prevProps, prevState) {
    console.log("Component Did Update");
    console.log("-----------");
  }

  componentWillUnmount() {
    console.log("Component Will Unmount");
    console.log("-----------");
  }

  render() {
    console.log("render");
    return (
      <div>
        <button onClick={this.incCounter}>Increment</button>
        <button onClick={this.decCounter}>Decrement</button>
        <div>Count: {this.state.count}</div>
      </div>
    );
  }
}
```

```
/* Output on initial script execution */

Constructor 
----------- 
render 
Component Did Mount 
----------- 

/* After clicking generate ignoreProp value button: */

Should Component Update - No Update 

/* After clicking increment button: */

Should Component Update - Yes, Update 
render 
Component Did Update 
----------- 

/* After clicking decrement button: */

Should Component Update - Yes, Update 
render 
Component Did Update 
----------- 

/* After clicking generate ignoreProp value button: */

Should Component Update - No Update 
```

## getDerivedStateFromProps

This method runs **before the component re-renders but only when prop changes**

**Note**: This is a `static` method!

The object it returns gets merged with the state of the component. If you return `null` nothing gets merged

It is helpful when you need to update your state that derives its value (computed) from a prop. For example, it is useful to change counter value when the seed value for the counter, as a prop from the parent, changes! 

It accepts two parameters that help us create additional state:

1. props 
2. state

**Note**: This method executes **before the first render** & **before subsequent updates (re-renders)** (*even before the `shouldComponentUpdate` hook*)

```jsx
// App.js
 class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      showCounter: true,
      ignoreProp: 0
    };
    this.toggleCounter = () =>
      this.setState({
        showCounter: !this.state.showCounter
      });

    this.generateignoreProp = () =>
      this.setState({
        ignoreProp: Number.parseInt(Math.random() * 100, 10)
      });

    this.setSeed = (e) =>
      this.setState({
        seed: e.target.value
      });
  }

  render() {
    const { showCounter, ignoreProp, seed } = this.state;
    return (
      <div className="App">
        <button onClick={this.toggleCounter}>Toggle Counter</button>
        <button onClick={this.generateignoreProp}>
          Generate ignoreProp value
        </button>
        <input
          type="number"
          onBlur={this.setSeed}
          placeholder="Enter seed value"
        />
        {showCounter ? <Counter seed={seed} ignoreProp={ignoreProp} /> : null}
      </div>
    );
  }
}
```

```jsx
// Counter.js
 class Counter extends React.Component {
  constructor(props) {
    console.log("Constructor");
    super(props);
    this.state = {
      count: 0
    };

    this.incCounter = (e) =>
      this.setState({
        count: this.state.count + 1
      });
    this.decCounter = (e) =>
      this.setState({
        count: this.state.count - 1
      });

    console.log("-----------");
  }

  componentDidMount() {
    console.log("Component Did Mount");
    console.log("-----------");
  }

  static getDerivedStateFromProps(props, state) {
    if (props.seed && props.seed !== state.seed) {
      console.log("Get Derived State From Props - Modify state");
      return {
        seed: props.seed,
        count: props.seed
      };
    }

    console.log("Get Derived State From Props - No state modification");
    return null;
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (
      nextState.count === this.state.count &&
      nextProps.seed === this.props.seed
    ) {
      console.log("Should Component Update - No Update");
      return false;
    }
    console.log("Should Component Update - Yes, Update");
    return true;
  }

  componentDidUpdate(prevProps, prevState) {
    console.log("Component Did Update");
    console.log("-----------");
  }

  componentWillUnmount() {
    console.log("Component Will Unmount");
    console.log("-----------");
  }

  render() {
    console.log("render");
    return (
      <div>
        <button onClick={this.incCounter}>Increment</button>
        <button onClick={this.decCounter}>Decrement</button>
        <div>Count: {this.state.count}</div>
      </div>
    );
  }
}
```

```
/* Output on initial script execution */

Constructor 
----------- 
Get Derived State From Props - No state modification 
render 
Component Did Mount 
----------- 

/* After typing a seed value in parent's input (value: 50): */

Get Derived State From Props - Modify state 
Should Component Update - Yes, Update 
render 
Component Did Update 
----------- 

/* After typing a seed value in parent's input (value: 50 again!): */

Get Derived State From Props - No state modification 
Should Component Update - No Update 
```

## getSnapshotBeforeUpdate

This method helps us **capture data before the component updates (i.e re-renders)**. The data can be things like layout width prior to re-render, cursor location, etc. We would like to have these data points available to us a snapshot after re-render, that is, in `componentDidUpdate`

Whatever value we return in this method becomes available as the 3rd argument to `componentDidUpdate`

It receives two parameters:

1. Previous props, &
2. Previous state

```jsx
// App.js: NO CHANGES
// Counter.js
import React from "react";

export default class Counter extends React.Component {
  constructor(props) {
    console.log("Constructor");
    super(props);
    this.state = {
      count: 0
    };

    this.incCounter = (e) =>
      this.setState({
        count: this.state.count + 1
      });
    this.decCounter = (e) =>
      this.setState({
        count: this.state.count - 1
      });

    console.log("-----------");
  }

  componentDidMount() {
    console.log("Component Did Mount");
    console.log("-----------");
  }

  static getDerivedStateFromProps(props, state) {
    if (props.seed && props.seed !== state.seed) {
      console.log("Get Derived State From Props - Modify state");
      return {
        seed: props.seed,
        count: props.seed
      };
    }

    console.log("Get Derived State From Props - No state modification");
    return null;
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (
      nextState.count === this.state.count &&
      nextProps.seed === this.props.seed
    ) {
      console.log("Should Component Update - No Update");
      return false;
    }
    console.log("Should Component Update - Yes, Update");
    return true;
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log("Get Snapshot Before Update");
    return {
      snapshot: 'value'
    }
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log("Component Did Update");
    console.log('snapshot', snapshot)
    console.log("-----------");
  }

  componentWillUnmount() {
    console.log("Component Will Unmount");
    console.log("-----------");
  }

  render() {
    console.log("render");
    return (
      <div>
        <button onClick={this.incCounter}>Increment</button>
        <button onClick={this.decCounter}>Decrement</button>
        <div>Count: {this.state.count}</div>
      </div>
    );
  }
}
```

```
/* Output on initial script execution */

Constructor 
----------- 
Get Derived State From Props - No state modification 
render 
Component Did Mount 
----------- 

/* After clicking increment button: */

Get Derived State From Props - No state modification 
Should Component Update - Yes, Update 
render 
Get Snapshot Before Update
Component Did Update 
snapshot {snapshot: "value"}
----------- 
```

## componentDidCatch

Whenever there is a *downstream erro*r (i.e errors in child components) or *error inside component,* we can catch it and perform some action like displaying a fallback UI.

If it is not caught, our app can crash! So, it is important to use such lifecycle methods when we anticipate runtime errors.

The method receives two parameters:

1. An error object (contains `message` property), &
2. An info object

```jsx
// Error.js
export default (props) => {
  return <div>{props.nope.nonExistent}</div>;
};
```

```jsx
// Counter.js
class Counter extends React.Component {
  constructor(props) {
    console.log("Constructor");
    super(props);
    this.state = {
      count: 0,
      showFallback: false
    };

    this.incCounter = (e) =>
      this.setState({
        count: this.state.count + 1
      });
    this.decCounter = (e) =>
      this.setState({
        count: this.state.count - 1
      });

    console.log("-----------");
  }

  componentDidMount() {
    console.log("Component Did Mount");
    console.log("-----------");
  }

  static getDerivedStateFromProps(props, state) {
    if (props.seed && props.seed !== state.seed) {
      console.log("Get Derived State From Props - Modify state");
      return {
        seed: props.seed,
        count: props.seed
      };
    }

    console.log("Get Derived State From Props - No state modification");
    return null;
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (
      nextState.count === this.state.count &&
      nextProps.seed === this.props.seed &&
      nextState.showFallback === this.state.showFallback
    ) {
      console.log("Should Component Update - No Update");
      return false;
    }
    console.log("Should Component Update - Yes, Update");
    return true;
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log("Get Snapshot Before Update");
    return {
      snapshot: "value"
    };
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log("Component Did Update");
    console.log("snapshot", snapshot);
    console.log("-----------");
  }

  componentWillUnmount() {
    console.log("Component Will Unmount");
    console.log("-----------");
  }

  render() {
    console.log("render");
    const { showFallback, count } = this.state;

    if (showFallback) {
      return <div> Encountered an error </div>;
    } else {
      return (
        <div>
          <button onClick={this.incCounter}>Increment</button>
          <button onClick={this.decCounter}>Decrement</button>
          <div>Count: {count}</div>
          <Error />
        </div>
      );
    }
  }

  componentDidCatch(error, info) {
    console.log("Component Did Catch", error, info);
    this.setState({
      showFallback: true
    });
  }
}
```

```
/* Output on initial script execution */

Constructor 
----------- 
Get Derived State From Props - No state modification 
render 
Component Did Mount 
----------- 
The above error occurred in the <_default> component: ...
Component Did Catch 
TypeError: Cannot read property 'nonExistent' of undefined
{componentStack: "..."}
Get Derived State From Props - No state modification 
Should Component Update - Yes, Update 
render 
Get Snapshot Before Update 
Component Did Update 
```

## getDerivedStateFromError

This method appears in a component known as "***ErrorBoundary***". It helps us update the state whenever an error is encountered (runtime error that can lead to a crash)

**Note**: It is a `static` method (similar to `getDerivedStateFromProps`)

It is used in conjuction with `componentDidCatch`

It receives one parameter: The *error* object.

Whatever object it returns, merges with the state of the component.

Along with `getDerivedStateFromError`, we can provide a graceful degradation instead of an app crash (A concept called "error boundaries")

```js
static getDerivedStateFromError(error) {
  console.log("Get Derived State From Error", error);
  return {
    error
  };
}
```

## Lifecycle visualised

```
/*  Normal flow (On mount, update, and finally, unmount): */

> constructor
|
|
> getDerivedStateFromProps
|
|
> render
|
|
> componentDidMount
|
|
> getDerivedStateFromProps
|
|
> shouldComponentUpdate
|
|
render
|
|
> getSnapshotBeforeUpdate (Even though it runs after render, it has access to prev props & state as params)
|
|
> componentDidUpdate
|
|
> componentWillUnmount

/*  Outside of the flow (On an uncaught error): */

> getDerivedStateFromError
|
|
> componentDidCatch
```

## Playground

[Link to playground](https://codesandbox.io/s/quizzical-khorana-zl8fx?file=/src/Counter.js)
