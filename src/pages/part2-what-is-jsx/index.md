---
title: React in and out (2)  What is JSX
date: '2020-04-26'
spoiler: Intoducing the JSX syntax and some traps
---

For the people who're really famillar with "old school" JavaScript Application, may heard or used templating languages like [Pug](https://pugjs.org/api/getting-started.html) or [Handlebars](https://handlebarsjs.com/). And sure, those languages help us a lot to write "elegant HTML" in an easy and maintainable way.

> Another templating languages? Really?

I'm sure this is the first impression for most of react newbies when learning JSX. Yes, their jobs look pretty similar, right? Actually, JSX play a significant role in React world. Let's learn some syntaxes of JSX and feel its power.

```tsx
const Button: ButtonTypeProps = ({
  htmlType, 
  classes, 
  handleClick, 
  buttonRef, 
  children
}) => (
  <button
    type={htmlType}
    className={classes}
    onClick={handleClick}
    ref={buttonRef}
  >
    { children }
  <button>
)
```



This is how we define a Button component with JSX in React. 

### Naming

Like the Button component we defined above, in convention, component names should also always start with a capital letter. For element, you should use lower-case letter. 

> What's the diffrence between `element` and `component`?
>
> "React elements are the building blocks of React applications. An element describes what you want to see on the screen. React elements are immutable."
>
> "React components are small, reusable pieces of code that return a React element to be rendered to the page. "

Some times we may use name spaces pattern to name our component to avoid conflicts like this:

```jsx
const NewHead = () => (
	<Head title="new head">
    <Head.Button />
    <Head.Tabs />
  </Head>
) 
```



### Comments

In fact, JSX is just JavaScript. So we could still use JavaScript comment style in JSX, but we need to insert comments into curly brackets:

```jsx
const App = () => (
	<>
  	{/* comments here */}
  	<Head />
  	{/* comments 
  	here */}
  	<Footer />
  </>
)
```



### Props

In JSX, the name of props respect JavaScript name convention, which is camelcase. And if the prop name consists keywords in JavaScript world, we need to use another one instead:

- `class` => `className`

- `for` => `htmlFor`

For some props which have boolean value, if we omit the value, JSX will set it true automatically. 



### Escaped in HTML

> By default, React DOM [escapes](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html) any values embedded in JSX before rendering them. Thus it ensures that you can never inject anything that’s not explicitly written in your application. Everything is converted to a string before being rendered. This helps prevent [XSS (cross-site-scripting)](https://en.wikipedia.org/wiki/Cross-site_scripting) attacks.

To avoid content that we pass in be escaped:

```jsx
<div dagerouslySetInnerHTML={{__html: 'cc &copy; 2020'}}>
```

Alright, this is it. Next chapter we'll talk about component, which is the most exciting part in React world. See you there :)