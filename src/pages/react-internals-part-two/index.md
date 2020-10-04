---

title: React Internals Part Two Component & PureComponent

date: '2020-10-04'

spoiler: Learn something from React.js source code.

---

> to be organized...

Here are some notes I took while learning React source code. 

**Note1**: I will focus on the contents of the [_**packages**_](https://sourcegraph.com/github.com/facebook/react/-/tree/packages) folder, rather than the overall architecture.

**Note2**: My analysis process will rely heavily on the sourcegraph plugin, which you can download from [_**here**_](https://docs.sourcegraph.com/integration/browser_extension). Also I'll post sourcegraph links of the code as I analyze it in case you want to follow with me.

**Note3**: This article will be updated as the official React repository is updated, at this time of writing React version is _**16.13.1**_.

***

`Component` and `PureComponent` are probably the most common React APIs that we use for development:

```js
class ComponentA extends React.Component {}

class ComponentB extends React.PureComponent {}
```

And as you may know the difference between them is that `PureComponent` makes a shallow comparison between old and new props or state to further figure out if a component needs an update or not, whereas `Component` doesn't help us for this.

As we mentioned in previous post, the only difference of their implementation details is:

```js
pureComponentPrototype.isPureReactComponent = true;
```

You can find the source code for this section at [_**packages/react/src/ReactBaseClasses.js**_](https://sourcegraph.com/github.com/facebook/react/-/blob/packages/react/src/ReactBaseClasses.js?utm_source=share#L20:10)


Notice: I'll omit some relatively unimportant code for simplicity.

- `Component`

```js
function Component(props, context, updater) {
  this.props = props;
  this.context = context;

  this.refs = emptyObject;

  this.updater = updater || ReactNoopUpdateQueue;
}
```
```js

Component.prototype.isReactComponent = {};

Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};
```

The code is pretty straightforward: function `Component`, which is the constructor of `Component` class, take couple parameters and assign them to the instance. And then the property `isReactComponent` and two methods `setState` and `forceUpdate` are added to the prototype of `Component`.

About `setState` and `forceUpdate`, as this part is platform related, it's implemented in `React-Dom`, so we'll skip it for now and explain later.


- `PureComponent`

```js
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;

  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = 
  (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;

Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true;
```

In order to understand the relationship between `PureComponent` and `Component`, let's briefly review inheritance in JavaScript(this part relys heavily on [Professional JavaScript for Web Developers 4th Edition](https://www.amazon.com/Professional-JavaScript-Developers-Matt-Frisbie/dp/1119366445/ref=pd_sbs_14_1/143-1717190-7644662?_encoding=UTF8&pd_rd_i=1119366445&pd_rd_r=0c8f661d-5bce-4c65-9d45-351df79c013e&pd_rd_w=889l7&pd_rd_wg=YsQpd&pf_rd_p=b65ee94e-1282-43fc-a8b1-8bf931f6dfab&pf_rd_r=21TN0J1M6BXM8PGDNFSF&psc=1&refRID=21TN0J1M6BXM8PGDNFSF)):

1. Prototype Chaining

   > What if the prototype were actually an instance of another type? That would mean the prototype itself would have a pointer to a different prototype that, in turn, would have a pointer to another constructor. If that prototype were also an instance of another type, then the pattern would continue, forming a chain between instances and prototypes. This is the basic idea behind prototype chaining.

    Problems with Prototype Chaining:

    - all instances of `subType` share the same property
    - no way to pass arguments into the `superType` constructor

```js{13}
function SuperType() { 
  this.property = true
}

SuperType.prototype.getSuperValue = function() { 
  return this.property;
};

function SubType() { 
  this.subproperty = false;
}

SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () { 
  return this.subproperty;
};

let instance = new SubType(); 

console.log(instance.getSuperValue()); // true
```


2. Constructor Stealing
  
   > The basic idea is quite simple: call the supertype constructor from within the subtype constructor. 

   Problems with Constructor Stealing:

   - Methods must be defined inside the supertype constructor, so that subType can inherit them.

```js{6}
function SuperType(name){ 
  this.name = name;
}

function SubType() {
  SuperType.call(this, "evan");
  this.hobby = 'music'; 
}

let instance = new SubType(); 

console.log(instance.name); // "evan"; 
console.log(instance.hobby); // 'music'
```

3. Combination Inheritance

   > The basic idea is to use prototype chaining to inherit properties and methods on the prototype and to use constructor stealing to inherit instance properties.

     Problem:
     
     - The supertype constructor is always called twice: once to create the subtype's prototype, and once inside the subtype constructor

```js{11,15}
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() { 
  console.log(this.name);
};

function SubType(name, age){ 
  SuperType.call(this, name);
  this.age = age; 
}

SubType.prototype = new SuperType();

SubType.prototype.sayAge = function() { 
  console.log(this.age);
};

let instance1 = new SubType("Nicholas", 29); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
instance1.sayName(); // "Nicholas"; 
instance1.sayAge(); // 29

let instance2 = new SubType("Greg", 27); 
console.log(instance2.colors); // "red,blue,green" 
instance2.sayName(); // "Greg"; 
instance2.sayAge(); // 27
```

4. Prototypal Inheritance
  > Essentially, `object()` performs a shadow copy of any object that is passed into it.

    Problems:
    - similar to using the prototype pattern, the properties containing reference values will always share those values

```js
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}
```

5. Parasitic Inheritance

   Problems:
   - adding functions to objects using parasitic inheritance leads to inefficiencies related to function reuse, similar to the constructor pattern.

```js
function createAnother(original){
  let clone = object(original); 
  clone.sayHi = function() { 
    console.log("hi"); 
  };
  return clone;
}
```

6. Parasitic Combination Inheritance

```js{15,16,17}
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}

function prototype(child, parent) {
  var prototype = object(parent.prototype);
  prototype.constructor = child;
  child.prototype = prototype;
}

prototype(Child, Parent);
```
Compared to Combination Inheritance, Parasitic Combination avoids calling the parent constructor when assigning a child prototype, thus eliminating the need to defined redundant properties on the child prototype.

And this is exacly how `PureComponent` inherit from `Component`. Also we noticed that there is a minor optimization of the prototype chain search:

```js
Object.assign(pureComponentPrototype, Component.prototype)
```

***


At this point, you‘re probably wondering the same thing I am: we can call life cycles like `componentDidMount`, `componentDidUpdate`, etc. in every instance generated from `Compoenent` or `PureComponent`, but where are they?


Further reading:
---

> [MDN: Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)