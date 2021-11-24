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

**When should we write a custom hook?**

1. When we want to refactor an H.O.C as shown above!
    - Originally, an H.O.C was used to enhance the functionality of a component. Therefore, we can use a hook to enhance a functional component.
2. Reusability: Create custom hooks when you notice a common pattern between your components and would like to extracted & generalized into cleaner and simplified code. You probably don't need a hook if it's going to used by just one component
3. Separation of concerns and decouples the logic from UI (abstract complexity): Custom hooks can be thought of as headless components - they do stuff but don't really render anything. They return values. Therefore, it is a good idea to build a hook to contain stateful logic that goes beyond just a single `useEffect`/`useState` since a custom hook can encapsulate all this complex logic.

## Compound components

When a component gets bloated, it is a good idea to separate the logic out into smaller components. This helps preserve the Single Responsibility Principle (SRP) as well as improve maintainability.

Another reason to break down a component into more components is **customizability**. The following example shows a `Menu` component that does everything related to displaying the menu items and buttons by itself:

```jsx
import React, { useState } from "react";

const Menu = ({ hideButtons, menuButtons, menuItems }) => {
  return (
    <div>
      {!hideButtons &&
        menuButtons.map(({ text, action }) => (
          <button key={text} onClick={action}>
            {text}
          </button>
        ))}
      {menuItems.map((text) => (
        <div key={text}>{text}</div>
      ))}
    </div>
  );
};

export default Menu;
```

The component could get bloated in the future but more importantly, it is not customizable. What if we wanted to place the buttons below the items? Even worse, what if in one use case we want the buttons to be above the items and in another, below them? Should we duplicate this component for each different purpose? 

The **compound component pattern** solves this problem!
- It introduces a *parent-child* relationship where you define a parent and it encloses different child components. We can move the child components around, remove them, or even add some more. Definitely customizable!
```jsx
<Parent {...parentProps}>
    <Child {...childProps} />
    <Child {...childProps} />
</Parent>
```
- It also makes the code more **readable / understandable** i.e When you look at the JSX, you know how the component is composed and what each one does based on a readable name for the component.
- The patterns also **avoids props overload** by exposing children to render. The `Menu` component took all the props for the elements it had to render. However, what if we just had a `MenuItem` child component and only that received the `menuItems` prop? Right prop for the right UI component.

An HTML example of this pattern is the following:
```jsx
<select>
  <option value="volvo">Volvo</option>
  <option value="mercedes">Mercedes</option>
  <option value="audi">Audi</option>
</select>
```

**Implementation**
- Identify the `children` prop in the parent. Use it in the render.
- Create a context in the parent component to store shared data, if any
- Wrap the children in a context provider so that they may access it later

```jsx
import React, { useState } from "react";

export const MenuContext = React.createContext({ hideButtons: false });
const Menu = ({ children, hideButtons }) => {
  return (
    <MenuContext.Provider value={{ hideButtons }}>
      {children}
    </MenuContext.Provider>
  );
};

export default Menu;
```

```jsx
import React, { useContext } from "react";
export const MenuButton = ({ text, action }) => {
  const { hideButtons } = useContext(MenuContext);

  if (hideButtons) {
    return null;
  }

  return (
    <button key={text} onClick={action}>
      {text}
    </button>
  );
};
```

```jsx
import React from "react";
export const MenuItem = ({ text }) => <div key={text}>{text}</div>;
```

```jsx
<Menu hideButtons>
  <MenuButton text="Exit" action={() => console.log("Exiting")} />
  <MenuButton text="Settings" action={() => console.log("settings")} />
  <MenuItem text="News" />
  <MenuItem text="Serials" />
  <MenuItem text="Sports" />
</Menu>
```

**A matter of preference:** We may even create static properties of the parent component and assign component to them. This approach enables the user to see how the parent is related to a child. It also keeps all related component together. 

We need to export only one component from a module when using this pattern

```jsx
// ...
const Menu = () => { /* ... */ }
// ...
Menu.Button = () => { /* ... */ }
Menu.Item = () => { /* ... */ }
// ...
export default Menu;
```

```jsx
<Menu hideButtons>
  <Menu.Button text="Exit" action={() => console.log("Exiting")} />
  <Menu.Button text="Settings" action={() => console.log("settings")} />
  <Menu.Item text="News" />
  <Menu.Item text="Serials" />
  <Menu.Item text="Sports" />
</Menu>
```

**Making a compound component even more extensible:** Add a *callback* prop to the parent. This allows us to many any state changes or other invoke other actions that the outside world can listen to! 

```jsx
<Menu hideButtons onFadeAway={onFadeAway}>{/* ... */}</Menu>
```

## Reusable styles

Sometimes, for **extensibility** & **customizability**, we can provide an API for altering or adding styles to our component

The common methods to add styles are:
1. Via a class name
2. Via a style attribute (This is especially helpful if you need to pass in emotion styles or something similar)

Consider the following example:

```jsx
/* styles.css */
.btn-default {
  background: #fff;
  border: 0;
  cursor: pointer;
  box-shadow: 0 1px 1px 1px #000;
}
```

```jsx
// Button.js
import React from "react";
import "./styles.css";

export const Button = ({ text, onClick }) => {
  return (
    <button key={text} onClick={onClick} className="btn-default">
      {text}
    </button>
  );
};

export default Button;
```

```jsx
// Usage
<Button  text="click me"  onClick={() =>  console.log("Clicked")} />
```

**Should we allow the user to change the properties of a component?**

It depends. If you want to enforce a strict style on a component, it is probably not a good idea to not allow this. However, sometimes you may let a user choose from various customizations by the use of a class name. Ex: A class for a particular type of border.

**Tip:** To import a stylesheet into a *variable* to tap into the styles, use a CSS module. Ex: `styles.module.css` and `import ButtonStyles from 'styles.module.css`. With these two changes, you can use `ButtonStyles.btn` or `ButtonStyles['btn']` where `.btn` is a css class declaration.

If you really want to enable custom styles (`style` attribute) to be passed in (such as CSS properties or emotion styles) then you may also expose a prop for the same.

Both the options are shown in the below example:

```css
/* styles.module.css */
.btn {
  background: #fff;
  border: 0;
  cursor: pointer;
  box-shadow: 0 1px 1px 1px #000;
}

.btn-border-blue {
  border: 1px solid blue;
}

.btn-border-teal {
  border: 1px solid teal;
}
```

```jsx
// Button.js
import React from "react";
import ButtonStyles from "./styles.module.css";

export const Button = ({ text, onClick, classes = [], styles }) => {
  const classNames = [ButtonStyles["btn"], ...classes].join(" ");

  return (
    <button key={text} onClick={onClick} className={classNames} style={styles}>
      {text}
    </button>
  );
};

export default Button;
```

```jsx
// Usage
import  ButtonStyles  from  "./styles.module.css";
// ...
<button key={text} onClick={onClick} className={classNames} style={styles}>
  {text}
</button>
```

> Note that using a `.module.css` is optional. You can always just use a plain `.css` stylesheet and use strings for class names!

## Control props

This is a very **powerful pattern** even though it looks simple. The objective is to *control the state of the child components* from a parent. In order to do this:
1. We need to *expose a prop* from the child to the parent that enables this control
2. To change the values supplied to this prop, the parent has to be *stateful*!
    - If not, then changes in the parent do not update the children
4. *Optional:* You may also want to expose a callback to the parent in order to give it data that helps change the value of the control prop.

The most common example of a control prop is that of an input element in react:
```jsx
const [value, setValue] = useState('');
//...
<input value={value} onChange={(e) => setValue(e.target.value)} />
```
The above pattern is for controlling DOM elements, specifically the HTML form elements. What about controlling components we build? We can do the same for them as well.

**When should we use control props on components?**

1. When we want to **synchronize** data between and instruction to different components.
2. When we want to give **absolute control** over a component to the parent / consumer.

Imagine a scenario where two buttons increment and decrement a counter. If they are not synchronised, they will be running two counters independent of each other:

```jsx
export const IncrementButton = ({ text }) => {
  const [count, setCount] = useState(0);
  const onClick = () => setCount(count + 1);

  return (
    <button key={text} onClick={onClick}>
      {text} ({count})
    </button>
  );
};

export const DecrementButton = ({ text }) => {
  const [count, setCount] = useState(0);
  const onClick = () => setCount(count - 1);

  return (
    <button key={text} onClick={onClick}>
      {text} ({count})
    </button>
  );
};

const Counter = () => {
  return (
    <div>
      <IncrementButton text="+" />
      <DecrementButton text="-" />
    </div>
  );
};
```

We can use control props to synchronise them: The parent will have a state for the counter and pass the values down. The children will invoke a callback on click to allow parent to update the count value. In this way, both the buttons will have the same counter value i.e as expected.

```jsx
export const IncrementButton = ({ text, count, onIncrement }) => (
  <button key={text} onClick={onIncrement}>
    {text} ({count})
  </button>
);

export const DecrementButton = ({ text, count, onDecrement }) => (
  <button key={text} onClick={onDecrement}>
    {text} ({count})
  </button>
);

const Counter = () => {
  const [count, setCount] = useState(0); // Lifted state into the parent
  const onIncrement = () => setCount(count + 1);
  const onDecrement = () => setCount(count - 1);

  return (
    <div>
      <IncrementButton text="+" count={count} onIncrement={onIncrement} />
      <DecrementButton text="-" count={count} onDecrement={onDecrement} />
    </div>
  );
};
```

`count` was the state value that was moved up into `Counter`. The control prop exposed by the buttons was `count` and the callbacks that helped update the count's state were `onIncrement` and `onDecrement`.

You might not always need callbacks as the state can update from outside the children too. In this example,m the callbacks were, in fact, needed.


