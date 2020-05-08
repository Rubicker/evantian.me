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


Note: to make things easier to understand, I'll skip some unneccesary code that not related this article like code about React-Native, SSR and Babel plugins etc.


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

`css` function is the key thing that `styled-component` used to take care of `template string`, let's see the implementation of it:

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

From above code, `flatten` help us to concat css styles as an array, it will transfer styledComponent to a className and deal with keyframes. In the end, there will be only static strings and functions left.

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

Ok, let's go back to `StyledComponent`, what features does this constructor bring in?

Break down this into several parts:

**`generateId`**

Like we talk before, `flatten` need an unique ID to identify if it encounter a component. `generateId` can help us for this.

```js
const identifiers = {};

function generateId(displayName, parentComponentId) {
  const name = typeof displayName !== 'string' 
    ? 'sc' 
    : escape(displayName);
  // Ensure that no displayName can lead to duplicate componentIds
  identifiers[name] = (identifiers[name] || 0) + 1;

  // generateComponentId actually return the hash value of the argument 
  // check it at utils/genetateComponentId.js
  const componentId = `${name}-${
    generateComponentId(name + identifiers[name])
  }`;
  return parentComponentId 
    ? `${parentComponentId}-${componentId}` 
    : componentId;
}
```

**`useResolvedAttrs`**

`useResolvedAttrs` is a React Hook used to calculate attrs at runtime under context(theme and pass-in props)

```js
function useResolvedAttrs(theme = EMPTY_OBJECT, props, attrs) {
  // returns [context, resolvedAttrs]
  // where resolvedAttrs is only the things 
  // injected by the attrs themselves
  const context = { ...props, theme };
  const resolvedAttrs = {};

  attrs.forEach(attrDef => {
    let resolvedAttrDef = attrDef;
    let key;

    if (isFunction(resolvedAttrDef)) {
      resolvedAttrDef = resolvedAttrDef(context);
    }

    for (key in resolvedAttrDef) {
      context[key] = resolvedAttrs[key] =
        key === 'className'
          ? joinStrings(resolvedAttrs[key], resolvedAttrDef[key])
          : resolvedAttrDef[key];
    }
  });

  return [context, resolvedAttrs];
}

```

**`useInjectedStyle`** 

This React hook help us to get a specific className for this component's style.

```js
function useInjectedStyle(
  componentStyle,
  hasAttrs,
  resolvedAttrs,
  warnTooManyClasses?
) {
  const styleSheet = useStyleSheet();
  const stylis = useStylis();

  // statically styled-components don't need to 
  // build an execution context object,
  // and shouldn't be increasing the number of class names
  const isStatic = componentStyle.isStatic && !hasAttrs;

  const className = isStatic
    ? componentStyle
        .generateAndInjectStyles(EMPTY_OBJECT, styleSheet, stylis)
    : componentStyle
        .generateAndInjectStyles(resolvedAttrs, styleSheet, stylis);

  useDebugValue(className);

  if (process.env.NODE_ENV !== 'production' 
    && !isStatic
    && warnTooManyClasses
  ) {
    warnTooManyClasses(className);
  }

  return className;
}
```

> [stylis](https://github.com/thysultan/stylis.js)


**`useStyledComponentImpl`**

This react hook returns a styledComponent, which pass props as context to ComponentStyle at rendering and then create a className.

```js
function useStyledComponentImpl(
  forwardedComponent,
  props,
  forwardedRef
) {
  const {
    attrs,
    componentStyle,
    defaultProps,
    foldedComponentIds,
    shouldForwardProp,
    styledComponentId,
    target,
  } = forwardedComponent;

  useDebugValue(styledComponentId);

  const theme = 
    determineTheme(props, useContext(ThemeContext), defaultProps);

  const [context, attrs] = 
    useResolvedAttrs(theme || EMPTY_OBJECT, props, componentAttrs);

  const generatedClassName = useInjectedStyle(
    componentStyle,
    componentAttrs.length > 0,
    context,
    process.env.NODE_ENV !== 'production' 
      ? forwardedComponent.warnTooManyClasses 
      : undefined
  );

  const refToForward = forwardedRef;

  const elementToBeCreated: Target = 
    attrs.$as || props.$as || attrs.as || props.as || target;

  const isTargetTag = isTag(elementToBeCreated);
  const computedProps = attrs !== props ? { ...props, ...attrs } : props;
  const propFilterFn = shouldForwardProp || (isTargetTag && validAttr);
  const propsForElement = {};

  for (const key in computedProps) {
    if (key[0] === '$' || key === 'as') continue;
    else if (key === 'forwardedAs') {
      propsForElement.as = computedProps[key];
    } else if (!propFilterFn || propFilterFn(key, validAttr)) {
      // Don't pass through non HTML tags through to HTML elements
      propsForElement[key] = computedProps[key];
    }
  }

  if (props.style && attrs.style !== props.style) {
    propsForElement.style = { ...props.style, ...attrs.style };
  }

  propsForElement.className = Array.prototype
    .concat(
      foldedComponentIds,
      styledComponentId,
      generatedClassName !== styledComponentId ? generatedClassName : null,
      props.className,
      attrs.className
    )
    .filter(Boolean)
    .join(' ');

  propsForElement.ref = refToForward;

  return createElement(elementToBeCreated, propsForElement);
}

```

**`createStyledComponent`**

This HOC packages our styledComponent:

```js
export default function createStyledComponent(
  target,
  options,
  rules
) {
  const isTargetStyledComp = isStyledComponent(target);
  const isCompositeComponent = !isTag(target);

  const {
    displayName = generateDisplayName(target),
    componentId = generateId(
      options.displayName, 
      options.parentComponentId
    ),
    attrs = EMPTY_ARRAY,
  } = options;

  const styledComponentId =
    options.displayName && options.componentId
      ? `${escape(options.displayName)}-${options.componentId}`
      : options.componentId || componentId;

  // fold the underlying StyledComponent attrs up (implicit extend)
  const finalAttrs =
    isTargetStyledComp && target.attrs
      ? Array.prototype.concat(target.attrs, attrs).filter(Boolean)
      : attrs;

  let shouldForwardProp = options.shouldForwardProp;

  if (isTargetStyledComp && target.shouldForwardProp) {
    if (shouldForwardProp) {
      // compose nested shouldForwardProp calls
      shouldForwardProp = (prop, filterFn) =>
        target.shouldForwardProp(prop, filterFn) 
        && options.shouldForwardProp(prop, filterFn);
    } else {
      shouldForwardProp = target.shouldForwardProp;
    }
  }

  const componentStyle = new ComponentStyle(
    isTargetStyledComp
      ? // fold the underlying StyledComponent rules up (implicit extend)
        target.componentStyle.rules.concat(rules)
      : rules,
    styledComponentId
  );

  /**
   * forwardRef creates a new interim component, 
   * which we'll take advantage of instead of 
   * extending ParentComponent to create _another_ interim class
   */
  let WrappedStyledComponent;

  const forwardRef = (props, ref) => 
    useStyledComponentImpl(WrappedStyledComponent, props, ref);

  forwardRef.displayName = displayName;

  // this is a forced cast to merge it StyledComponentWrapperProperties
  WrappedStyledComponent = React.forwardRef(forwardRef);

  WrappedStyledComponent.attrs = finalAttrs;
  WrappedStyledComponent.componentStyle = componentStyle;
  WrappedStyledComponent.displayName = displayName;
  WrappedStyledComponent.shouldForwardProp = shouldForwardProp;

  // this static is used to preserve the cascade of static classes 
  // for component selector purposes; 
  // this is especially important with usage of the css prop
  WrappedStyledComponent.foldedComponentIds = isTargetStyledComp
    ? Array.prototype.concat(
        target.foldedComponentIds, 
        target.styledComponentId
      )
    : EMPTY_ARRAY;

  WrappedStyledComponent.styledComponentId = styledComponentId;

  // fold the underlying StyledComponent 
  // target up since we folded the styles
  WrappedStyledComponent.target = isTargetStyledComp
    ? target.target
    : target;

  WrappedStyledComponent.withComponent = function withComponent(tag) {
    const { componentId: previousComponentId, ...optionsToCopy } = options;

    const newComponentId =
      previousComponentId &&
      `${previousComponentId}-${
        isTag(tag) 
          ? tag 
          : escape(getComponentName(tag))
      }`;

    const newOptions = {
      ...optionsToCopy,
      attrs: finalAttrs,
      componentId: newComponentId,
    };

    return createStyledComponent(tag, newOptions, rules);
  };

  Object.defineProperty(WrappedStyledComponent, 'defaultProps', {
    get() {
      return this._foldedDefaultProps;
    },

    set(obj) {
      this._foldedDefaultProps = isTargetStyledComp 
        ? merge({}, target.defaultProps, obj) 
        : obj;
    },
  });

  if (process.env.NODE_ENV !== 'production') {
    checkDynamicCreation(displayName, styledComponentId);

    WrappedStyledComponent.warnTooManyClasses = createWarnTooManyClasses(
      displayName,
      styledComponentId
    );
  }

  WrappedStyledComponent.toString = () => 
    `.${WrappedStyledComponent.styledComponentId}`;

  if (isCompositeComponent) {
    hoist(WrappedStyledComponent, (target: any), {
      // all SC-specific things should not be hoisted
      attrs: true,
      componentStyle: true,
      displayName: true,
      foldedComponentIds: true,
      shouldForwardProp: true,
      self: true,
      styledComponentId: true,
      target: true,
      withComponent: true,
    });
  }

  return WrappedStyledComponent;
}
```

> [React-official docs - Static Methos Must Be Copied Over](https://reactjs.org/docs/higher-order-components.html#static-methods-must-be-copied-over)
>
> [hoist-non-react-statics](https://www.npmjs.com/package/hoist-non-react-statics)
>
> [The magic behind 💅 styled-components](https://mxstbr.blog/2016/11/styled-components-magic-explained/)
> 
> [深入浅出 标签模板字符串 和 💅styled-components 💅](https://juejin.im/post/5cf23f35e51d45598611b911)
