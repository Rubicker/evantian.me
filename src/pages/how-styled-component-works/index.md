---
title: How Styled-Component works?
date: '2020-05-07'
spoiler: The magic behind styled-component
---

[styled-component](https://styled-components.com/) is one of the most famous CSS-in-JS solutions, we could write out component like this with styled-component:

```jsx
const Button = styled.button`
  color: grey;
`
```

Let's walk through [the repo of styled-component](https://github.com/styled-components/styled-components) and learn something from it!


Note: to make things clear and easy to understand, I'll skip some unneccesary code that not related this article like code about React-Native, SSR and Babel plugins etc.


Start from styled constructor:

```js
// packages/styled-components/src/constructors/styled.js

const styled = tag => constructWithOptions(StyledComponent, tag);

// Shorthands for all valid HTML Elements
// we could use both styled(div) or styled.div
domElements.forEach(domElement => {
  styled[domElement] = styled(domElement);
});
```

`construnctWithOptions` is a function that allows us to merge some options into `StyledComponent`:

```js{14, 15}
// packages/styled-components/src/constructors/constructWithOptions.js

export default function constructWithOptions(
  componentConstructor,
  tag,
  options = EMPTY_OBJECT
) {
  // to verify tag
  if (!isValidElementType(tag)) {
    return throwStyledError(1, String(tag));
  }

  /* This is callable directly as a template function */
  const templateFunction = (...args) =>
    componentConstructor(tag, options, css(...args));


  /* skip some static methods like templateFunction.withConfig
      and templateFunction.attrs */

  return templateFunction;
}
```

`css` function is the key thing `styled-component` used to take care of `template string`, let's see the implementation of it:

```js
// packages/styled-components/src/constructors/css.js

export default function css(styles, ...interpolations) {

  // skip some boundary situations

  return flatten(interleave(styles, interpolations));
}
```

So `flatten` looks like handle a huge responsibility on compiling templete string to something that styledComponentConstructor can understand.

Before we check `flatten`, let's see what `interleave` returns:

```js
// packages/styled-components/src/utils/interleave.js

export default (
  strings,
  interpolations
) => {
  const result = [strings[0]];

  for (let i = 0, len = interpolations.length; i < len; i += 1) {
    result.push(interpolations[i], strings[i + 1]);
  }

  return result;
};

// Here is an example to show how interleave works:
const PLACE = 'styled-component'
const NAME = 'Evan'

const compileTemplateString = 
  (strings, ...interpolations) => interleave(strings, interpolations)

console.log(compileTemplateString`Hello, ${NAME}, welcome to ${PLACE}.`)
// output: 
// ["Hello, ", "Evan", ", welcome to ", "styled-component", "."]
```

Then `flatten` show up:

```js
// packages/styled-components/src/utils/flatten.js

export default function flatten(chunk, executionContext, styleSheet) {
  if (Array.isArray(chunk)) {
    const ruleSet = [];

    for (let i = 0, len = chunk.length, result; i < len; i += 1) {
      result = flatten(chunk[i], executionContext, styleSheet);

      if (result === '') continue;
      else if (Array.isArray(result)) ruleSet.push(...result);
      else ruleSet.push(result);
    }

    return ruleSet;
  }

  if (isFalsish(chunk)) {
    return '';
  }

  /* Handle other components */
  if (isStyledComponent(chunk)) {
    return `.${chunk.styledComponentId}`;
  }

  /* Either execute or defer the function */
  if (isFunction(chunk)) {
    if (isStatelessFunction(chunk) && executionContext) {
      const result = chunk(executionContext);

      if (process.env.NODE_ENV !== 'production' && isElement(result)) {
        console.warn(
          // some logs
        );
      }

      return flatten(result, executionContext, styleSheet);
    } else return chunk;
  }

  if (chunk instanceof Keyframes) {
    if (styleSheet) {
      chunk.inject(styleSheet);
      return chunk.getName();
    } else return chunk;
  }

  /* Handle objects */
  return isPlainObject(chunk) ? objToCssArray(chunk) : chunk.toString();
}
```

We can see, `flatten` help us to concat css styles as an array, it will transfer styledComponent to a className and deal with keyframes. In the end, there will be only static strings and functions left.

```js
const Link = styled.a`
  display: flex;
  align-items: center;
  padding: 5px 10px;
  background: papayawhip;
  color: palevioletred;
`;
const Icon = styled.svg`
  flex: none;
  transition: fill 0.25s;
  width: 48px;
  height: 48px;
  // here flatten will compile Link to .${Link.styledComponentId}
  // which is a className for Link
  ${Link}:hover & {    
    fill: rebeccapurple;
  }
`;
```

At runtime, functions will execute and get a value for some property:
```js
const Button = styled.button`
  width: ${props => props.width}px
`

// ...

<Button width={30} /> 
```


















> [The magic behind 💅 styled-components](https://mxstbr.blog/2016/11/styled-components-magic-explained/)