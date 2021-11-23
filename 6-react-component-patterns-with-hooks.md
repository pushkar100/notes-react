# React component patterns with hooks

[Course link](https://www.udemy.com/course/the-complete-guide-to-advanced-react-patterns)

## Using custom hooks

Hooks are better than classes. Here's why:
1. Hooks let us organize the logic inside a component into reusable isolated units
2. Hooks let you always use functions instead of having to constantly switch between functions, classes, higher-order components, and render props.
	- Unlike patterns like render props or higher-order components, Hooks don’t introduce unnecessary nesting into your component tre
3. It’s also likely that **adopting Hooks could  _reduce_  your bundle size** because code using Hooks tends to minify better than equivalent code using classes.
4. Since Hooks are regular JavaScript functions, you can combine built-in Hooks provided by React into your own “custom Hooks”. This lets you turn complex problems into one-liners and share them across your application or with the React community.
	- They’re not a way to share _state_ — but a way to share _stateful logic_.
	- They also follow the **rules of hooks**.

**Pattern:** If you have a higher order component (HOC), you can now **refactor the HOC into a hook**!
- Anytime you see an H.O.C, think about replacing it with a custom hook
- When you see a class-based lifecycle methods in your H.O.C, think about built-in hooks that can replace those too! For example, `useEffect` can replace a `componentDidMount` and a `componentDidUpdate`

The convention for using a custom hook is the same as for the other hooks: prefix it with the **`use`**.

**Before refactor**

```jsx
// H.O.C:
var withLogger = (ComposedComponent) =>
  class extends Component {
    componentDidMount() {
      this.setState({
        meta: getLogMetaData()
      });
    }

    log = (id, event) => {
      console.log(`id: ${id} | event: ${event}`);
      console.log(`source: ${this.state.meta.source}`);
    }

    render() {
      return <ComposedComponent {...this.props} logger={this.log} />;
    }
  };

// Component that needs H.O.C:
// First convert this to a function if it already isn't one
const Greetings = ({ logger }) => {
  return (
    <div>
      <h1>Hello</h1>
      <button onClick={() => logger(1, 'enter')}>Enter</button>
    </div>
  );
};

export default withLogger(Greetings);
```

```
// Output
id: 1 | event: enter 
source: www 
```

**After refactor**

```jsx
import React, { useState, useEffect } from "react";

// Mock data
const getLogMetaData = () => ({
  source: "www"
});

// Hook:
const useLogger = () => {
  const [meta, setMeta] = useState({});
  const logger = (id, event) => {
    console.log(`id: ${id} | event: ${event}`);
    console.log(`source: ${meta.source}`);
  };

  useEffect(() => {
    setMeta(getLogMetaData());
  }, []); // Mimic "componentDidMount"

  return logger;
};

// Component uses the hook: A simple function call!
const Greetings = () => {
  // No props injected like with an H.O.C
  const logger = useLogger();

  return (
    <div>
      <h1>Hello</h1>
      <button onClick={() => logger(1, "enter")}>Enter</button>
    </div>
  );
};

export default Greetings;
```

