# React Context API

## Solves

Prop drilling through multiple levels of components in the render tree i.e data being consumed by deeply nested components

## Used for

Global state which is frequently updated and is used by deeply nested components

## Examples

Color theme, user details, language preferences, cart status in e-commerce applications, and so on

## Implementation

1. Create the context (`React.createContext`)
2. Provide the context (Wrap with a `<context>.Provider` component and provide a value attribute)

```typescript
type Theme = 'light' | 'dark';

interface ThemeContext {
  theme: Theme;
  toggleTheme: () => void;
}

const themeContext = createContext<ThemeContext | null>(null);

interface ThemeProviderProps {
  children: React.ReactNode;
}

const ThemeProvider = ({ children }: ThemeProviderProps) => {
  const [theme, setTheme] = useState<Theme>('light');

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return (
    <themeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </themeContext.Provider>
  );
};
```

3. Consume the context (useContext)

```typescript
function App() {
  return (
    <>
      <ThemeProvider>
        <MyButton />
    </>
  );
}

const MyButton = () => {
  const { theme, toggleTheme } = useContext(themeContext) as ThemeContext;

  const styles = {
    light: { color: 'black', background: 'white', cursor: 'pointer' },
    dark: { color: 'white', background: 'black', cursor: 'pointer' },
  };

  return (
    <button style={styles[theme]} onClick={toggleTheme}>
      Toggle theme! Number of renders = {renderRef.current}
    </button>
  );
};

```

## Problems with the Context API

Whenever we have multiple data points in a context and different components consume it, each component re-renders even if a data point that the component does not use changes! (Ex: context containing number of likes and number of comments incremented via buttons. Component displaying number of likes will still re-render when number of comments increases and vice-versa as long as it consume the context).
Note: Wrapping a consumer component with React.memo() will not help as it only optimises rendering for the props whereas context is not part of the props.
Solution: Manually break down a context in to smaller contexts i.e more granular (Ex: one context for likes and one for comments)

## Context vs Redux or similar libraries 

1. If you have a large state that is consumed by deeply nested components but not all together i.e only pick certain points only then should we use redux since we can point to only that data point from the store via a component. The component will not re-render if other data points in the store update. 
2. Only if you have proper granularity for contexts or do not mind the manual work of breaking down contexts then use the Context API (which requires more manual work to prevent re-renders when data not belonging to components updates inside the context being consumed by them). Also, if the large dataset is more or less static then we can use Context API.
Bottom-line: Use redux for global state with high frequency changes. For low frequency changes and more granular globaldata sets, use context. 

## Improving the performance of Context API

1. Functions as context properties: Make sure they are wrapped in a useCallback() so that their reference does not change unnecessarily. For components consuming the context, they re-render even when the ancestor with the provider re-renders since it causes the context value to be updated and even if nothing changes, the function reference changes (if not using useCallback()).
2. Wrap the context object in a useMemo: This is just an additional step to make sure the context object is only computer once and it does not change unless the dependencies we have mentioned in the useMemo does not change! i.e explicitly defining what changes should trigger the context object changes 

(Note: if there are many function and other properties of the context that we cannot guarantee will not update unnecessarily or that their references (Ex: function) will not change without good reason, it is probably time to split your context further!)

### Avoiding beginner mistakes with Context:
If the context values depend on state, make sure to use a provider component instead of the parent to store that state. Why? If the parent/ancestor stores state, and passes it to a context provider as value then when the state changes, the nested child components within the provider not depending on the context (i.e not consuming it) will also re-render since a parent/ancestor state changed

### Using a library to avoid Context API perf issues
1. We can manually split the context and wrap the function properties inside a useCallback and the whole context object within a useMemo. However, we can also do better:
2. Use an NPM library such as "use-context-selector" which help us pick up only the values we care about inside a component from the context and do not re-render for other changes in the context.

## Using a custom hook to wrap a context

Instead of `useContext(myContext)`, create a `useCustomContext`, and use `useCustomContext(myContext)` inside the components. 
Inside this wrapper, handle the default values and error handling. This makes sure that when we use the custom hook, we simply destructure it and use the context values instead of adding conditionals.

## Another use for React Context:
To mock API responses, save the value in a context, and we can mock the provider of that context with mock a data provider.


