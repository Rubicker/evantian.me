---

title: React Hooks with TypeScript

date: '2020-05-02'

spoiler: useTS()?

---

## `useState`

```js
// Here `isModalOpen` is infered to be Boolean,
// and `toggleModal` is infered to a func that recieves a Boolean argument. 
const [isModalOpen, toggleModal] = React.useState(false)
```

Sometimes we might need to initial state value as `null`, then set it to other type value later:

```tsx
type IData = {
  date: Date,
  content: string
}
const [data, setData] = React.useState<IData>(null)
```

## `useRef`

This means React will help us to set a new value:

```tsx
// This means inputEl is inmutable or read-only,
// and we'll pass it to `ref` attributes of one React element
const inputEl = useRef<HTMLInputElement>(null)
```

And also remember to check if inputEl is avaliable:

```tsx
const PasswordInput = () => {
  const inputEl = useRef<HTMLInputElement>(null)
  const handleClick = () => {
    // Here we check if inputEl & inputEl.current exist
    // if yes, then it must be a HTMLInputElement, which 
    // has some specific methods like `focus()`
    if (inputEl && inputEl.current) {
      inputEl.current.value = ''
    }
  }
  
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={handleClick}>Click me to clear input area!</button>
    </>
  )
} 
```

> Tips:
>
> ```tsx
> // we could use optional chaining to simplify handleClick
> const handleClick = () => {
>   inputEl?.current?.focus()
> }
> ```
>
> 