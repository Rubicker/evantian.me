---
title: How to implement React lifecycle hooks with useEffect?
date: '2019-06-09'
spoiler: Aren’t Hooks Magic?
---

You heard about react hooks. You made a simulation of traditional `setState` in a functional component. You felt the power of react hooks.

Alright, "state" is clear, now you might be wondering about the lifecycle methods: how to implement them in a functional component with a "simple" `useEffect`?

Don't worry. In this article will go over all of them, including `componentDidMount`, `componentWillUnmount` and `componentDidUpdate`.

PS: if you haven't touched react hooks yet and want take a look, I strongly recommend you to go through the [official documentation](https://reactjs.org/docs/hooks-intro.html) first.

## useDidMount

```js
import { useEffect } from 'react';

const useDidMount = fn => useEffect(() => fn && fn(), []);

export default useDidMount;
```

Notice the second argument? That is the array of values that the effect depends on. if we pass an empty array, it means that we only need this function fire once.

## useDidUpdate

```js
import { useEffect, useRef } from 'react';

const useDidUpdate = (fn, conditions) => {
  const didMountRef = useRef(false);
  useEffect(() => {
    if (!didMountRef.current) {
      didMountRef.current = true;
      return;
    }

    return fn && fn();
  }, conditions);
};

export default useDidUpdate
```

This time we use `useRef` to record if this render is the first render. If it is, then we do nothing but make label `didMountRef` to be true.

## useWillUnmount

```js
import { useEffect } from 'react';

const useWillUnmount = fn => useEffect(() => () => fn && fn(), []);

export default useWillUnmount;
```

If we return a function at the first arguement, the component will fire it before unmount.

## Bonus

What if I just want a forceUpdate?

```js
const forceUpdate = () => useState(0)[1];
```
