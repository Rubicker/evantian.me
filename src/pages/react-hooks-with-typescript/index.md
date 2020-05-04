---

title: React Hooks with TypeScript

date: '2020-05-02'

spoiler: useTS()?

---

## `useState`

```js
// Here `isModalOpen` is infered to be Boolean,
// and `toggleModal` is infered to a func 
// that recieves a Boolean argument. 
const [isModalOpen, toggleModal] = React.useState(false)
```

Most time we might need to initial state value as `null`, then set it to other type value later:

```tsx
type IData = {
  date: Date,
  content: string
}
const [data, setData] = React.useState<IData>(null)
```

## `useRef`

```tsx
// This means inputEl is inmutable or read-only,
// and we'll pass it to `ref` attributes of one React element
const inputEl = useRef<HTMLInputElement>(null)
```

And also remember to check if inputEl is avaliable before using it, or React and Typescript will complain about it:

```tsx{7}
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

> ```tsx
> // we could use optional chaining to simplify handleClick
> const handleClick = () => {
>   inputEl?.current?.focus()
> }
> ```
>
> About [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)

## `useEffect`

You don't need to provide any extra typings. TypeScript will check that the method signature of the function you provide is correct. This function also has a return value(for cleanups). And TypeScript will check that you provide a correct function as well.

```tsx
useEffect(
  () => {
    const DARK_THEME_CLASS = 'dark-theme'
    const bodyClassList = window.document.body.classList
    if (enabled) {
      bodyClassList.add(DARK_THEME_CLASS)
    } else {
      bodyClassList.remove(DARK_THEME_CLASS)
    }
    return () => {
      bodyClassList.remove(DARK_THEME_CLASS)
    }
  }
)
```

## `useMemo` / `useCallback`

Still, you don't have to do too much, the React typings will do the best.

```tsx
const users = useMemo(() => getUsers(userData), [userData])

const clickCallback = useCallback((count: number) => {
  // ... 
}, [count])

// Then...
clickCallback(5)
```

## `useReducer`

```tsx
type Todo = {
  id: number,
  title: string,
  completed: boolean
}

type TodoState = Todo[]

type TodoAction = 
  | { type: 'ADD_TODO', payload: { title: string } }
  | { type: 'DELETE_TODO', payload: { id: number } }
  | { type: 'TOGGLE_TODO', payload: { id: number } }

const initialState = [
  { id: 1, title: 'read books', completed: false },
  { id: 2, title: 'drink water', completed: false },
  { id: 3, title: 'walk dog', completed: true }
]

let nextId = 4
const reducer = (state = initialState: TodoState, action: TodoAction) => {
  switch(action.type) {
    // here TypeScript make sure we don't get wrong with type name
    case 'ADD_TODO':
      return [
        ...state, 
        { id: nextId++, title: action.payload.title, completed: false }
      ];
    case 'DELETE_TODO': 
      return state.filter(todo => todo.id !== action.payload.id)
    case 'TOGGLE_TODO':
      return state.map(todo => 
        todo.id === action.payload.id 
          ? { ...todo, completed: !todo.completed }
          : todo
      )
  }
}
```

## Custom Hooks

All we need to notice is the return value of curtom hooks:

```tsx
// Code below come from: https://usehooks.com/useAsync/
const useAsync = (
  asyncFunction: Promise<any>, 
  immediate = true: boolean
) => {
  const [pending, setPending] = useState(false);
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  // The execute function wraps asyncFunction and
  // handles setting state for pending, value, and error.
  // useCallback ensures the below useEffect is not called
  // on every render, but only if asyncFunction changes.
  const execute = useCallback(() => {
    setPending(true);
    setValue(null);
    setError(null);
    return asyncFunction()
      .then(response => setValue(response))
      .catch(error => setError(error))
      .finally(() => setPending(false));
  }, [asyncFunction]);

  // Call execute if we want to fire it right away.
  // Otherwise execute can be called later, such as
  // in an onClick handler.
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { execute, pending, value, error };
  
  // if we return an array instead of object:
  return [ execute, pending, value, error ] as const
  // or 
  return [ execute, pending, value, error ] as [
    // here is the type you need to clarify
  ]
};
```





