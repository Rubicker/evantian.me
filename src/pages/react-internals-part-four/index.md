---

title: React Internals Part Four Context

date: '2020-10-06'

spoiler: Learn something from React.js source code.

---

> to be organized...

Here are some notes I took while learning React source code. 

**Note1**: I will focus on the contents of the [_**packages**_](https://sourcegraph.com/github.com/facebook/react/-/tree/packages) folder, rather than the overall architecture.

**Note2**: My analysis process will rely heavily on the sourcegraph plugin, which you can download from [_**here**_](https://docs.sourcegraph.com/integration/browser_extension). Also I'll post sourcegraph links of the code as I analyze it in case you want to follow with me.

**Note3**: This article will be updated as the official React repository is updated, at this time of writing React version is _**16.13.1**_.

***

In a typical React application, data is passed from parent to child via props from top to bottom. Sometimes this is extremely cumbersome for certain global or near-global attributes such as UI themes, language or regional preferences. `Context` provides an easy way to share properties between components without having to pass props through the component tree level by level.


Let's say, we need to provide a global used variables: `isDark` to determine if we should use dark theme UI:

```js
const themes = {
  isDark: true
}

const ThemeContext = React.createContext(themes)
```

```js
const App = () => {
  return (
    <ThemeContext.Provider>
      <Header />
    </ThemeContext.Provider>
  )
}
```

#### Context without hooks

```js
const Header = () => {
  <ThemeContext.Consumer>
    {
      themes => <Header.Base isDark={themes.isDark} />
    }
  </ThemeContext.Consumer>
}
```

#### Context with hooks

```js
const Header = () => {
  const { themes } = useContext(ThemeContext)
  return <Header.Base isDark={themes.isDark} />
}
```

Let's see the implementation of `createContext`:

```js
export function createContext<T>(
  defaultValue: T,
  calculateChangedBits: ?(a: T, b: T) => number,
): ReactContext<T> {
  if (calculateChangedBits === undefined) {
    calculateChangedBits = null;
  } 

  const context: ReactContext<T> = {
    $$typeof: REACT_CONTEXT_TYPE,
    _calculateChangedBits: calculateChangedBits,
    _currentValue: defaultValue,
    _currentValue2: defaultValue,
    _threadCount: 0,

    Provider: (null: any),
    Consumer: (null: any),
  };

  context.Provider = {
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context,
  };

  context.Consumer = context;

  return context;
}
```

Pretty straightfoward. Remind that `context` itself and `context.Provider` have diffrent `$$typeof`.

As for how values are passed and updated, we'll explain that in a later article.

Further reading:
---

>  [New context API](https://github.com/facebook/react/pull/11818)