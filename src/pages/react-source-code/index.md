---

title: React Internals, Part One-Overview

date: '2020-10-03'

spoiler: Learn something from React.js source code.

---

> to be organized...

Here are some notes I took while learning React source code. 

**Note1**: I will focus on the contents of the [_**packages**_](https://sourcegraph.com/github.com/facebook/react/-/tree/packages) folder, rather than the overall architecture.

**Note2**: My analysis process will rely heavily on the sourcegraph plugin, which you can download from [here](https://docs.sourcegraph.com/integration/browser_extension). Also I'll post sourcegraph links to the code as I analyze it so you can find it.

**Note3**: This article will be updated as the official React repository is updated, at this time of writing React version is _**16.13.1**_.


## ReactElement

I guess this API is the most familiar and yet the strangest one. 

Most of us don't need to call this API directly when we're developing, but it plays an essential role in helping us convert JSX into react components.

To unravel its mysteries, let's get hands dirty from [__packages/react/src/React.js__](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/React.js?utm_source=share#L37):

```js
// Remove all the import declarations for simplicity

const createElement = __DEV__ 
  ? createElementWithValidation 
  : createElementProd;
const cloneElement = __DEV__ 
  ? cloneElementWithValidation 
  : cloneElementProd;
const createFactory = __DEV__ 
  ? createFactoryWithValidation 
  : createFactoryProd;

const Children = {
  map,
  forEach,
  count,
  toArray,
  only,
};

export {
  Children,
  createMutableSource,
  createRef,
  Component,
  PureComponent,
  createContext,
  forwardRef,
  lazy,
  memo,
  useCallback,
  useContext,
  useEffect,
  useImperativeHandle,
  useDebugValue,
  useLayoutEffect,
  useMemo,
  useMutableSource,
  useReducer,
  useRef,
  useState,
  REACT_FRAGMENT_TYPE as Fragment,
  REACT_PROFILER_TYPE as Profiler,
  REACT_STRICT_MODE_TYPE as StrictMode,
  REACT_DEBUG_TRACING_MODE_TYPE as unstable_DebugTracingMode,
  REACT_SUSPENSE_TYPE as Suspense,
  createElement,
  cloneElement,
  isValidElement,
  ReactVersion as version,
  ReactSharedInternals 
    as __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED,
  // Deprecated behind disableCreateFactory
  createFactory,
  // Concurrent Mode
  useTransition,
  startTransition,
  useDeferredValue,
  REACT_SUSPENSE_LIST_TYPE as SuspenseList,
  REACT_LEGACY_HIDDEN_TYPE as unstable_LegacyHidden,
  // enableBlocksAPI
  block,
  // enableFundamentalAPI
  createFundamental as unstable_createFundamental,
  // enableScopeAPI
  REACT_SCOPE_TYPE as unstable_Scope,
  useOpaqueIdentifier as unstable_useOpaqueIdentifier,
};
```
We can see that this file exists as an API export. Here I'll pick some common APIs and briefly review them below:

- `Children`

  This object contains several APIs, such as `React.Children.map`, `React.Children.only`, etc. to help you handle React children prop. These methods circumvent some drawbacks of using JavaScript's native array APIs directly, such as the fact that rendering children with `Array.prototype.map` will report an error when the children type is not an array, while `React.children.map` will just ignore it!

  You can explore more through links below:

  - [Official documentation: React.Children](https://reactjs.org/docs/react-api.html#reactchildren)
  - [A deep dive into children in React by max stoiber](https://mxstbr.blog/2017/02/react-children-deepdive/#enforcing-a-single-child)

- `createRef`
  

  creates a ref that can be attached to React elements via the ref attribute.

  You can explore more through links below(and findout why stringRef is legacy):

  - [Official documentation: Refs and the DOM](https://reactjs.org/docs/refs-and-the-dom.html)
  - [Implement Better Refs API ](https://github.com/facebook/react/issues/1373)

```js
// in class component
class App extends React.component {
  constructor() {
    this.demoRef = React.createRef()
  }

  render() {
    return <div ref={this.demoRef} />

    // or
    
    return <div ref={ref => this.demoRef = ref}>
  }
}
```

- `Component` & `PureComponent`
  
  From [_**packages/react/src/ReactBaseClasses.js**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/ReactBaseClasses.js?utm_source=share#L141:51), we know that there is no essential difference between the two classes, except that the `PureComponent` has an additional property(which is mounted on its prototype):

```js
pureComponentPrototype.isPureReactComponent = true;
```
  And this property is used to determine if the component's props and state should be shallowly compared or not, and thus whether the component needs to be updated:
```js
// packages/react-reconciler/src/ReactFiberClassComponent.new.js
if (ctor.prototype && ctor.prototype.isPureReactComponent) {
  return (
    !shallowEqual(oldProps, newProps) 
    || !shallowEqual(oldState, newState)
  );
}
```
  check [here](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react-reconciler/src/ReactFiberClassComponent.new.js?utm_source=share#L335)
  
- `createContext`

  You can explore more through links below:
  - [Official documentation: React.createContext](https://reactjs.org/docs/context.html#reactcreatecontext)
  - [How to use React Context effectively](https://kentcdodds.com/blog/how-to-use-react-context-effectively)

- `fowardRef`
 
  Sometimes we may need to pass a ref through a component to its children. This is exacly where `fowardRef` shines.

  - [Official documentation: Fowarding Refs](https://reactjs.org/docs/forwarding-refs.html)
  
- `useXXX` hooks
  
  Hooks are a really powerful addition in React 16.8. It makes resusing logic between defferent component a snap.

- `createElement`

  `createElement` is used to build `ReactElement`, usually we use Babel to compile JSX as a result of a call from the `React.createElement` function:
  ```js
  // JSX
  <p className="text">hello world</p>

  // compile result
  React.createElement('p', {className: 'text'}, 'hello world')
  ```
  