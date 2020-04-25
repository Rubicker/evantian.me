---
title: React in and out (1)  What is React
date: '2020-04-25'
spoiler: Intoducing the main concepts of React
---

This is how React official definite React.js:

>  A JavaScript library for building user interfaces.

How to understand the real meaning behind this defination? And how React.js build and maintain UI components for us? Is this process quick enough?

Let's walk through the real React and dig out these answers. 

## Focus on UI

Nowadays web application has became much more complicated  than previous days. That's why some libraries / frameworks like Angular or Ember offers a whole tool sets, take Angular as an example, it provides the following "out of the box":

- `@angular/animation`
- `@angular/forms`
- `@angular/router`
- `@angular/common/http`
- ...

and we can almost develop any kind of application features with them. Seems like we already have the silver bullet, so why do we still need React and what's the diffrence bettwen React and these full-fledged frameworks? 

>  Flexibility

Yes, that's the first word cross my mind. React.js only focus on UI part, give us the biggest freedom to choose what we like to set up our applications. And it's just that simple:

`UI = render(data)`



---

## Virtual DOM

Let's be honest, it's such a painful memories to manipulate DOM in a traditional way.

Every time we want to update the page, we need to manually change the DOM:

![](./dom-update.png)

And we both know it's really an expensive move to update DOM, which always a performance killer in Frontend Developing. Also those part of code will ruin the readability of out codebase more or less. 

React uses Virtual DOM to solve this pain point, it transfer real DOM to JavaScript Object tree, which is the Virtual DOM. Every time the data has changed, React will compare the new Virual DOM with the old one and then figure out if the real DOM need a update and which part should be updated. All of this happened automatically, of course we need to do some stuffs to help React do it better(We'll discuss this part in the future chapter).

With virtual DOM:

![](./vdom-update.png)

---



## Functional Programming(FP)

If you're kinda not familar with FP, I strongly  recommend you read [this article](https://medium.com/@agm1984/an-overview-of-functional-programming-in-javascript-and-react-part-one-10d75b509e9e) before next step. 

Technically speaking,  FP is the core of React.  We can find FP everywhere in React: Virtual DOM, declaritive components concept etc. Benifit from FP, we can reuse our components, and even our logics(with React hooks, which we'll talk about later), this also keep our code simple and matainable. 

---



Hope you enjoy this ;)

In next chapter, we'll discuss about JSX, the first new concept that we'll learn in React world. See you there :)





