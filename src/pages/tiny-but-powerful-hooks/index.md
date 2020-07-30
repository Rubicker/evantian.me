---

title: Hooks, Tiny but powerful

date: '2020-07-29'

spoiler: Here's some React hooks recipes you may need.

---


### useOnClickOutside

```js
const useOnClickOutside = (ref, callback) => {
  useEffect(() => {
    const listener = event => {
      if (!ref.current || ref.current.contains(event)) {
        return
      }

      callback(event)
    }

    document.addEventListener('mousedown', listener)
    document.addEventListener('touchStart', listener)

    return () => {
      document.removeEventListener('mousedown', listener)
      document.removeEventListener('touchStart', listener)
    }

  }, [ref, callback])
}
```

### useInterval

> inspried by [Making setInterval Declarative with React Hooks](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)

```js
const useInterval = (
  callback, 
  delay, 
  runOnload = false, 
  effectDeps = []
) => {
  const savedCallback = useRef()

  useEffect(() => {
    if (runOnload) callback()
  }, [...effectDeps])

  // update callback when needed
  useEffect(() => {
    savedCallback.current = callback
  }, [callback])

  // this savedCallback can be persisi across the re-renders with ref
  useEffect(() => {
    if (delay !== null) {
      const id = setInterval(() => savedCallback.current(), delay)
      return () => clearInterval(id)
    }
  }, [delay, ...effectDeps])
}
```

### useWhyDidYouUpdate

```js
const useWhyDidYouUpdate = (name, props) => {
  const previousProps = useRef()

  useEffect(() => {
    if (previous.current) {
      const allKeys = Object.keys({...previousProps.current, ...props})
      const changeObj = {}

      allKeys.forEach(key => {
        if (previousProps.current[key] !== props[key]) {
          changesObj[key] = {
            from: previousProps.current[key],
            to: props[key]
          }
        }
      })

      if (object.keys(changesObj).length) {
        console.log('why-did-you-update', name, changesObj)
      }
    }

    previousProps.current = props
  })
}
```

### usePrevious

```js
const usePrevious = value => {
  const ref = useRef()

  useEffect(() => {
    ref.current = value
  }, [value])

  return ref.current
}
```