---

title: React Internals Part One Overview

date: '2020-10-03'

spoiler: Learn something from React.js source code.

---

> to be organized...

Here are some notes I took while learning React source code. 

**Note1**: I will focus on the contents of the [_**packages**_](https://sourcegraph.com/github.com/facebook/react/-/tree/packages) folder, rather than the overall architecture.

**Note2**: My analysis process will rely heavily on the sourcegraph plugin, which you can download from [_**here**_](https://docs.sourcegraph.com/integration/browser_extension). Also I'll post sourcegraph links of the code as I analyze it in case you want to follow with me.

**Note3**: This article will be updated as the official React repository is updated, at this time of writing React version is _**16.13.1**_.


## ReactElement

I guess this API is the most familiar and yet the strangest one. 

Most of us don't need to call this API directly when we're developing, but it plays an essential role in helping us convert JSX into react components.

To unravel its mysteries, let's get hands dirty from [_**packages/react/src/React.js**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/React.js?utm_source=share#L37):

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

  - [_**Official documentation: React.Children**_](https://reactjs.org/docs/react-api.html#reactchildren)
  - [_**A deep dive into children in React by max stoiber**_](https://mxstbr.blog/2017/02/react-children-deepdive/#enforcing-a-single-child)

- `createRef`
  

  creates a ref that can be attached to React elements via the ref attribute.

  You can explore more through links below(and findout why stringRef is legacy):

  - [_**Official documentation: Refs and the DOM**_](https://reactjs.org/docs/refs-and-the-dom.html)
  - [_**Implement Better Refs API**_](https://github.com/facebook/react/issues/1373)

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
  check [_**here**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react-reconciler/src/ReactFiberClassComponent.new.js?utm_source=share#L335) for the source code.
  
- `createContext`

  You can explore more through links below:
  - [_**Official documentation: React.createContext**_](https://reactjs.org/docs/context.html#reactcreatecontext)
  - [_**How to use React Context effectively**_](https://kentcdodds.com/blog/how-to-use-react-context-effectively)

- `fowardRef`
 
  Sometimes we may need to pass a ref through a component to its children. This is exacly where `fowardRef` shines.

  - [_**Official documentation: Fowarding Refs**_](https://reactjs.org/docs/forwarding-refs.html)
  
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
  
So now we have a basic understanding of React APIs, let's get back to the original topic: `React.createElement`. If you have a sharp eye, you probably noticed that the `createElement` exported by [_**packages/react/src/React.js**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/React.js?utm_source=share#L63:33) can be different depending on the environment variables:

```js
const createElement = __DEV__ 
  ? createElementWithValidation 
  : createElementProd
```

As the name implies, if your code is in a development environment, the `createElement` will do some additional checks on the given parameters, such as `type` etc. To make this part easy to read, let's just interpret production version `createElement`, which is `createElementProd` in the above code.

According to the entry file, we can find the code for the `createElement` located at [_**packages/react/src/ReactElement.js**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/ReactElement.js?utm_source=share#L348:17), which can be roughly divided into three sections:

- Gather component props from the second argument `config` as a new object named `props`, excluding the four special properties(`RESERVED_PROPS`) `key`, `ref`, `self`, and `source` (these are handled separately and will be passed directly to the next function `ReactElement`).
  
```js
//...
let propName;

const props = {};

let key = null;
let ref = null;
let self = null;
let source = null;

if (config != null) {
  if (hasValidRef(config)) {
    ref = config.ref;

    if (__DEV__) {
      warnIfStringRefCannotBeAutoConverted(config);
    }
  }
  if (hasValidKey(config)) {
    key = '' + config.key;
  }

  self = config.__self === undefined ? null : config.__self;
  source = config.__source === undefined ? null : config.__source;

  for (propName in config) {
    if (
      hasOwnProperty.call(config, propName) &&
      !RESERVED_PROPS.hasOwnProperty(propName)
    ) {
      props[propName] = config[propName];
    }
  }
}
//...
```
- Handle the `children` prop(The third and subsequent parameters are all children prop)

```js
const childrenLength = arguments.length - 2;
if (childrenLength === 1) {
  props.children = children;
} else if (childrenLength > 1) {
  const childArray = Array(childrenLength);
  for (let i = 0; i < childrenLength; i++) {
    childArray[i] = arguments[i + 2];
  }
  if (__DEV__) {
    if (Object.freeze) {
      Object.freeze(childArray);
    }
  }
  props.children = childArray;
}
```
- Handle `defaultProps`

```js
if (type && type.defaultProps) {
  const defaultProps = type.defaultProps;
  for (propName in defaultProps) {
    if (props[propName] === undefined) {
      props[propName] = defaultProps[propName];
    }
  }
}
```

In the end, `createElement` just pass all of these into `ReactElement` funtion.

```js
return ReactElement(
  type,
  key,
  ref,
  self,
  source,
  ReactCurrentOwner.current,
  props,
);
```
So what did `ReactElement` do with these parameters? 

The answer is really simple. `ReactElement` simply returns an object containing all the parameters except `self` and `source`, with an additional property `$$typeof` with a value of `REACT_ELEMENT_TYPE`:

```js
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  // omit the dev code

  return element;

}
```

So this is it. In short, `createElement` creates an element object for us, which contains some key attributes such as `$$typeof`, `type`, `props`, `key`, `ref` and so on. As for what this object is used for and how it helps to update the React Dom, I'll explain that later.

Further reading:
> [Why Do React Elements Have a $$typeof Property? by Dan Abramov](https://overreacted.io/why-do-react-elements-have-typeof-property/)

> [RFC: createElement changes and surrounding deprecations](https://github.com/reactjs/rfcs/pull/107)