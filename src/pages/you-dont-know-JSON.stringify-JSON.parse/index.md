---
title: You don't know JSON.stringify / JSON.parse
date: '2019-06-02'
spoiler: It's fun to go deep with some API that you thought you are already familar with.
---

[`JSON.stringify`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) is absolutely one of my favorite JavaScript's native methods.
It's really helpful when you want to do a deep copy with some objects. Of course, there
are some "Gocha" moments during this. So let's take a look.

## Getting Started

Before going deep into it, let's take a overview first.

> The JSON.stringify() method converts a JavaScript object or value to a JSON string, optionally replacing values if a replacer function is specified or optionally including only the specified properties if a replacer array is specified.

“converts a JavaScript object or value to a JSON string”, hmmm, sounds normal. 

“replacing values if a replacer function is specified”, ok, cool. 

“including only the specified properties if a replacer array is specified”, wait, what?

I believe some of my readers might have the same feeling above when first checking out the definition of `JSON.stringify`. Most of us got the impression that the only job it can do is a deep clone(with the help of `JSON.parse` of course). Like this below:

```js
const obj1 = { name: 'Ross' }

const obj2 = JSON.parse(JSON.stringify(obj1)) // { name: 'Ross' }
```

Actually it can do a better job than this, even more! Let's check out the siganature first:

```js
JSON.stringify(value[, replacer[, space]])
```

See, `JSON.stringify` can accept 3 parameters in fact: `value` for the thing you want to convert, `replacer` for 
the function or **array** if you need to do some filter or replace job when converting, `space` for the separate marker.

## value

Let's talk it straight, what kind of values are non-stringifiable?

**circular structure**, **Function**, **undefined**, **Symbol**, **Infinity**, **NaN**.

For circular structure, JavaScript will throw an error when you trying to convert it using `JSON.stringify`:

```js{2,8}
const foo = {}
foo.content = foo
JSON.stringify(foo)
// Uncaught TypeError: Converting circular structure to JSON

// Same as array
const bar = []
bar[0] = bar
JSON.stringify(bar)
// Uncaught TypeError: Converting circular structure to JSON
```

A Best practice is that you need to wrap `JSON.stringify` with `try/catch` because the circular structure could happen anywhere in production.

For Function, undefined and Symbol, Object just drop them and Array replace them with `null`; For Infinity and NaN, they will all be treated as `null`:

```js
const foo = {
  a: function() {},
  b: undefined,
  c: Symbol(),
  d: Infinity,
  e: NaN
}
JSON.stringify(foo)  // "{"d":null,"e":null}"

const bar = [ function(){}, undefined, Symbol(), Infinity, NaN ]
JSON.stringify(bar) // "[null, null, null, null, null]"
```

Btw, I didn't figure out why the special values get dropped in Object and get replaced with `null` in Array. I mean to keep the shape(no strip out) is always a good choice, and it can help us know which value is transformed.

## replacer

Although `JSON.stringify` deal with some special value in a "bad" way, we could format the output in a human-readable way by replacer :

```js
const foo = {
  a: Symbol('foo'),
  b: 1
}

JSON.stringify(foo, (_, v) => typeof v === 'symbol' ? 'A symbol' : v)
// "{"a":"A symbol","b":1}"
```

Here's the definition:

> A function that alters the behavior of the stringification process, or an array of String and Number objects that serve as a whitelist for selecting/filtering the properties of the value object to be included in the JSON string.

Pretty clear, right?

By replacer, we can deal with Functions or undefined etc as the same way.

Here is another use case for replacer as a whitelist:

```js
const foo = {
  name: 'Chandler',
  age: 29,
  sex: 'male',
  job: 'satistical analysis and data reconfiguration'  // Yeah, I know it 🤷‍♂
}

const whiteList = [ 'name', 'job' ]

JSON.stringify(foo, whiteList)
// "{"name":"Chandler","job":"satistical analysis and data reconfiguration"}"
```

## space

space is used to control space of the output. We can give it a string or a number.

> If it is a number, successive levels in the stringification will each be indented by this many space characters (up to 10).
If it is a string, successive levels will be indented by this string (or the first ten characters of it).

## toJSON()

In fact, we can customize JSON stringification behavior by giving a property named `toJSON` to an object as below(copied from MDN):

```js
var obj = {
  data: 'data',
  
  toJSON(key){
    if(key)
      return `Now I am a nested object under key '${key}'`;
    else
      return this;
  }
};

JSON.stringify(obj);
// '{"data":"data"}'

JSON.stringify({ obj })
// '{"obj":"Now I am a nested object under key 'obj'"}'

JSON.stringify([ obj ])
// '["Now I am a nested object under key '0'"]'
```


> To be continued...

