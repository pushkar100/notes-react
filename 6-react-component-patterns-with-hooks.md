# React component patterns with hooks

[Course link](https://www.udemy.com/course/the-complete-guide-to-advanced-react-patterns)

[Article](https://kentcdodds.com/blog/control-props-vs-state-reducers)

[Article](https://kentcdodds.com/blog/the-state-initializer-pattern)

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

## Props collection

When you know that there are certain props that are likely to be supplied by a user of a component, we can group them and pre-define them as a collection that the user can then pass in as a single prop.

Consider this example:
```jsx
<SomeComponent prop1={1} prop2={2} prop3={3} uniquePropA />
<SomeComponent prop1={1} prop2={2} prop3={3} uniquePropB />
```
Like in the example above, do you have a case where you constantly find yourself passing in the same, multiple props to the component whenever you instantiate it? What if you need to render it multiple times? In such a scenario, it helps to group these props into just one prop and pass that prop in as shown below:

```jsx
const collection = { prop1: 1, prop2: 2, prop3: 3 }
// ...
<SomeComponent collection uniquePropA />
<SomeComponent collection uniquePropB />
```

It is a very useful pattern when:
1. You need to use the same set of props every time
2. When the prop values are the same every time you use them (which is why having a collections object that has defined them makes sense)

The perfect example of a good place to use this pattern is when passing **accessibility** props (`aria-*`). All semantic HTML elements need this and depending on the type of element, many aria attributes can be the same.

```jsx
import React from "react";

const Button = ({ text, onClick, tabIndex, ariaDisabled }) => {
  return (
    <button 
	    onClick={onClick} 
	    tabIndex={tabIndex} 
	    role="button" 
	    aria-disabled={ariaDisabled} 
	>
      {text}
    </button>
  );
};

export default Button;
```

```jsx
// usage
<Button
  text="send"
  onClick={() => console.log("Clicked")}
  role="button"
  tabIndex={0}
  ariaDisabled="false"
/>
<Button
  text="open"
  onClick={() => console.log("Clicked")}
  role="button"
  tabIndex={1}
  ariaDisabled="false"
/>
```

We can use the *props collection* pattern to make this component easier to work with by supplying a single prop for all the accessibility declarations:

```jsx
const buttonAriaPropCollection = {
  tabIndex: 0,
  role: "button",
  "aria-disabled": false
};
```

```jsx
const Button = ({ text, onClick, tabIndex, role, 'aria-disabled': ariaDisabled }) => {
  return (
    <button onClick={onClick} tabIndex={tabIndex} role={role} aria-disabled={ariaDisabled}>
      {text}
    </button>
  );
};
```

```jsx
<Button
  text="click me"
  onClick={() => console.log("Clicked")}
  {...buttonAriaPropCollection}
/>
```

## Props getter 

This pattern is exactly the same as a props collection method but it allows for some **flexibility**. What if for one of the buttons, you want to change one of the accessibility props from the collection? A props collection does not let you do that but a props getter can!

- It allows us to override some props from the collection with our own i.e the user's 
- We define a function that returns an object of props instead of just an object containing props
- The arguments to the functions will define the override of certain props
- For methods, we probably want to invoke additional methods defined & NOT override the default. For this purpose, creating a helper function that invokes the default as well as the passed in functions is useful.

```jsx
const logClick = () => console.log("button clicked");
```

```jsx
const callAll = (...fns) => (...args) => fns.forEach((fn) => fn(...args));
```

```jsx
export const buttonPropsGetter = ({ customOnClick, ...props }) => ({
  onClick: callAll(logClick, customOnClick),
  tabIndex: 0,
  role: "button",
  "aria-disabled": false,
  text: "Button",
  ...props // The overrides
});

const Button = ({ text, buttonProps }) => {
  return <button {...buttonProps}>{text}</button>;
};
```

```jsx
// Usage:
<Button
  {...buttonPropsGetter({
    customOnClick: () => console.log("Sent email"),
    text: "Send"
  })}
/>
```

## State reducers

This pattern has a goal similar to that of control props. That is, it gives more control to the user of the component. Both are based on the *Inversion of Control* principle.

For components with just a prop and an event handler, a control prop pattern where the parent maintains the state and the event handler method is sufficient. An input element that is controlled is an example of this.

What happens when the control mechanism gets more complicated? What if we use control props but are faced with the problem of multiple props & multiple event handlers to manage. It gets too complicated: The user of the component needs to *manage the entire state of the child component*. This is a *drawback* of control props.

State reducer pattern allows us to do what control props did but by using a reducer function:
1. Easier to manage 
2. Has the benefit of the user not having to manage the entire state for the child
	- Instruction based (Provide actions similar to redux)
3. Can pass in a custom reducer when needed!
	- Built in such a way that it overrides the state change from the default reducer *only when needed*

### Implementation using hooks

1. Utilize a `useReducer` in your component
2. Keep a default reducer function
3. Build a custom reducer in such a way that it overrides the default reducer in required scenarios

A simple example with 3 elements:
- A controlled div that toggles the text 'on' & 'off' on clicking itself
- A button to 'switch on' the div
- Another button to 'switch off' the div

```jsx
function useToggle() {
  const [on, setOnState] = React.useState(false);

  const toggle = () => setOnState((o) => !o);
  const setOn = () => setOnState(true);
  const setOff = () => setOnState(false);

  return { on, toggle, setOn, setOff };
}
```
```jsx
function Toggle() {
  const { on, toggle, setOn, setOff } = useToggle();

  return (
    <div>
      <button onClick={setOff}>Disable</button>
      <button onClick={setOn}>Enable</button>
      <div onClick={toggle}>
        {on ?  "On"  :  "off"}
      </div>
    </div>
  );
}
```
```jsx
// usage
<Toggle />
```

What if a new requirement comes in like so:
> Allow the user to toggle the div's on/off status a maximum of 4 times by clicking on it. After 4 clicks, only the buttons have the ability to change the div's status

In such a scenario, we can go ahead and edit `useToggle` to include this logic. However, it is not very good for composability. What if there are other components that do not want to `useToggle` to change its original logic? 

A better solution is to use the State Reducer pattern. We can give the control of the state to the user of the component while maintaining a default reducer

*The component we are trying to apply this pattern on here is actually the hook, **`useToggle`***

**Step 1**: Change the `useToggle` hook to use a reducer

```jsx
const actionTypes = {
  on: "on",
  off: "off",
  toggle: "toggle"
};

const toggleReducer = (state, action) => {
  switch (action.type) {
    case actionTypes.on:
      return { on: true };
    case actionTypes.off:
      return { on: false };
    case actionTypes.toggle:
      return { on: !state.on };
    default:
      return state;
  }
};

function useToggle() {
  const [{ on }, dispatch] = React.useReducer(toggleReducer, { on: false });

  const toggle = () => dispatch({ type: actionTypes.toggle });
  const setOn = () => dispatch({ type: actionTypes.on });
  const setOff = () => dispatch({ type: actionTypes.off });

  return { on, toggle, setOn, setOff };
}

function Toggle() {
  const { on, toggle, setOn, setOff } = useToggle();

  return (
    <div>
      <button onClick={setOff}>Disable</button>
      <button onClick={setOn}>Enable</button>
      <div onClick={toggle}>
        {on ?  "On"  :  "off"}
      </div>
    </div>
  );
}
```

**Step 2**: Allow users to define a custom reducer that overrides the default one

```jsx
const actionTypes = {
  on: "on",
  off: "off",
  toggle: "toggle"
};

const toggleReducer = (state, action) => {
  switch (action.type) {
    case actionTypes.on:
      return { on: true };
    case actionTypes.off:
      return { on: false };
    case actionTypes.toggle:
      return { on: !state.on };
    default:
      return state;
  }
};

function useToggle(reducer = toggleReducer) {
  const [{ on }, dispatch] = React.useReducer(reducer, { on: false });

  const toggle = () => dispatch({ type: actionTypes.toggle });
  const setOn = () => dispatch({ type: actionTypes.on });
  const setOff = () => dispatch({ type: actionTypes.off });

  return { on, toggle, setOn, setOff };
}

function Toggle() {
  const [clicks, setClicks] = React.useState(0);
  const tooManyClicks = clicks > 4;

  const { on, toggle, setOn, setOff } = useToggle((state, action) => {
    const defaultChanges = toggleReducer(state, action);

    if (tooManyClicks && action.type === actionTypes.toggle) {
      return { ...defaultChanges, on: state.on };
    }

    return defaultChanges;
  });

  return (
    <div>
      <button onClick={setOff}>Disable</button>
      <button onClick={setOn}>Enable</button>
      <div
        onClick={() => {
          toggle();
          setClicks((clicks) => clicks + 1);
        }}
      >
        {on ? "On" : "off"}
      </div>
    </div>
  );
}
```

Therefore, state reducers:
- Are better than control props for inverting control of state
- Allow the user to manage only the state updation that needs to be different from the default updation flow

## Which pattern to use when?

### Basic decision-making

1. ***Are we adding pure logic?***
	- Use a **custom hook**! Examples include adding logging mechanism, detecting copy paste, providing context (use `useContext`), fetching data from/dispatching action to redux (`useSelector`, `useDispatch`), making ajax requests, etc.
2. ***Are we building components purely for rendering purposes?***
	- ***Do the components have a parent-child relationship?***
		- If ***yes***, use the **Compound Components pattern**. It avoids bloated components & prop drilling. Better for composability.
		- If ***no***, use simple components (**no pattern**). If a single component is too bloated, split them out into more reusable components that do not have a parent-child relationship.

If you need both logic & presentation, you can combine hooks with simple or compound components i.e build hooks that will be used by a component/compound set of components.

### Further decision-making

1. ***Do we need to reset state?***
	- It is a good idea to use the **state initializer pattern**
2. ***Does it make sense to group props?*** (or) ***Are we going to reuse a set of props together or multiple times?***
	- Use the **Prop Collections** or **Prop Getters** pattern. Use the latter if you need to be able to customize the set of props.
3. ***Are we handling multiple state elements?*** (or) ***Do we want to give the user of the component more control?***
	- Use the **State Reducer** pattern. If we do not want to give control to the user of the component but we still have many state elements for it, a simple `useReducer` should suffice
	- If the parent can manage *state* as well as *event handlers* (Ex: for input elements) we could also use **control props**
4. ***Should we allow the user to override the default styles of the component?***
	- Use the **Reusable Styles** pattern
