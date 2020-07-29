---

title: Hooks, Tiny but powerful

date: '2020-07-29'

spoiler: Here's some React hooks recipes you may need.

---


### useOnClickOutside

```js
const useOnClickOutside = (ref, callback) => {
  useEffect(() => {
    const eventTrigger = event => {
      if (!ref.current || ref.current.contains(event)) {
        return
      }

      callback(event)
    }

    document.addEventListener('mousedown', eventTrigger)
    document.addEventListener('touchStart', eventTrigger)

    return () => {
      document.removeEventListener('mousedown', eventTrigger)
      document.removeEventListener('touchStart', eventTrigger)
    }

  }, [ref, callback])
}
```