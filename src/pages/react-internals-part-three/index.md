---

title: React Internals Part Three Refs

date: '2020-10-05'

spoiler: Learn something from React.js source code.

---

> to be organized...

Here are some notes I took while learning React source code. 

**Note1**: I will focus on the contents of the [_**packages**_](https://sourcegraph.com/github.com/facebook/react/-/tree/packages) folder, rather than the overall architecture.

**Note2**: My analysis process will rely heavily on the sourcegraph plugin, which you can download from [_**here**_](https://docs.sourcegraph.com/integration/browser_extension). Also I'll post sourcegraph links of the code as I analyze it in case you want to follow with me.

**Note3**: This article will be updated as the official React repository is updated, at this time of writing React version is _**16.13.1**_.

***

First of all, let's talk about how to create a DOM reference of a node in React:

#### string ref(legacy method)

```js{3}
class ComponentA extends React.Component {
  handleSpanRef = () => {
    this.refs.spanRef.textContent = 'clicked'
  }

  render() {
    return <span ref="spanRef" onClick={this.handleSpanRef}>hello</span>
  }
}
```

Note the line 3, recall how Component is implemented：

```js
function Component(props, context, updater) {
  // ...
  this.refs = emptyObject;
  //...
}
```
As the note says:
> If a component has string refs, we will assign a different object later.

**String Ref** Seems pretty easy to write, but the [_**downsides**_](https://news.ycombinator.com/edit?id=12093234) are obvious:
  - String refs are not composable. A wrapping component can’t “snoop” on a ref to a child if it already has an existing string ref. On the other hand, callback refs don’t have a single owner, so you can always compose them.
  - String refs don’t work with static analysis like Flow. Flow can’t guess the magic that framework does to make the string ref “appear” on this.refs, as well as its type (which could be different). Callback refs are friendlier to static analysis.
  - The owner for a string ref is determined by the currently executing component. This means that with a common “render callback” pattern (e.g. <DataTable renderRow={this.renderRow} />), the wrong component will own the ref (it will end up on DataTable instead of your component defining renderRow).
  - String refs force React to keep track of currently executing component. This is problematic because it makes react module stateful, and thus causes weird errors when react module is duplicated in the bundle.


#### callback ref

```js
class ComponentA extends React.Component {
  handleSpanRef = () => {
    this.spanRef.textContent = 'clicked'
  }

  render() {
    return (
      <span 
        ref={el => this.spanRef = el} 
        onClick={this.handleSpanRef}>
        hello
      </span>
    )
  }
}
```
#### createRef


```js
class ComponentA extends React.Component {
  constructor() {
    super();
    this.spanRef = createRef();
  }
  handleSpanRef = () => {
    this.spanRef.current.textContent = "clicked";
  };

  render() {
    return (
      <span ref={this.spanRef} onClick={this.handleSpanRef}>
        hello
      </span>
    );
  }
}
```
I guess you won't believe how simple the `createRef` implementation is:

```js
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  return refObject;
}
```
source code at [_**packages/react/src/ReactCreateRef.js**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/ReactCreateRef.js?utm_source=share#L14:1)

Just hold on, how it can hold an immutable value during lifecycle will be covered in following articles.

#### FowardRef

It's ok to use `ref` attribute **inside** a functional component as long as you refer to a DOM element or a class component, but it can't be used **on** functional components because they don't have instances... 

```js
const MyFunctionalComponent = () => <p>hello I'm functional component</p>

class ComponentA extends React.component {
  constructor() {
    this.funcRef = createRef()
  }
  render() {
    return (
      // 🚨 this will not work!
      // Function components cannot be given refs. 
      // Attempts to access this ref will fail.
      <MyFunctionalComponent ref={this.funcRef}>
    )
  }
}
```


Until React offers an API called `fowardRef`:

```js
const MyFunctionalComponent = React.forwardRef((props, ref) => (
  <p ref={ref}>hello I'm functional component</p>
));
```

The implemantation is as simple as `createRef`:
```js
export function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
  const elementType = {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };

  return elementType;
}

```

We can see that `forwardRef` actually return a type, take the above function component `<MyFunctionalComponent ref={this.funcRef}>` as an example, after being compiled by `React.createElement`, it will throw out an object like this:

```js
{
  $$typeof: REACT_ELEMENT_TYPE,
  type: {
    $$typeof: REACT_FIBER_REF_TYPE,
    render: (props, ref) => 
      <p ref={ref}>hello I'm functional component</p>
  },
  key: null,
  ref: this.funcRef,
  props: {},
  ...
}
```

Further reading:
---

>  [Forwarding Refs](https://reactjs.org/docs/forwarding-refs.html)