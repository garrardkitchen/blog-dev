---
title: "How to Create React App"
date: 2020-04-17T09:13:19+01:00
draft: true
tags: [react, nodejs]
---

# Why this blog?

I recently started on a project to replace our current Twilio telephony implementation with Twilio Flex.  Flex is written with React.  It also comes with a set of core UI React components.  It doesn't yet support React Hooks but this is on the horizon!  I have used React a few times over the last 5ish years.  I had researched it and introduced it as a core UI front-end stack technology in a previous employment.  

Reusable UI components is a very powerful productivity enabler when done correctly.  Hooks, a new feature introduced with **React 16.8**, has consolidated my approach when building a React app. To capture new learnings, I have decided to create a blog post to share these and also to remind myself, possibly at a later date on how to do something.  It would not be the first time that I have searched for how to do something, to find the solution in an old blog post of mine!

## React Requirements

- npm
- nodejs
- react (create-react-app)

To create your first react app, run:

```script
npm i npx create-react-app
npx create-react-app <name-of-app>
cd <name-of-app>
```

The above, installs npx. This is used to execute a `command` from node_modules/.bin folder.  By default, it will look in the `$PATH` or local project for the `command`.

npx create-react-app create a project that includes the default framework for your SPA.  Finally, you cd into the name of your project.

```
npm i @material-ui/core --save-dev
npm install @material-ui/icons --save-dev
```

Added this to before the </head> element:
```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons" />
```

## Pushing state change down into child components

Let's say we have a parent (App) and 2 child functon components:

```js
import React, {useState} from "react"

export default function App(prop) {
    const [items, setItems] = useState(prop.rebuildList)
    const onClick = () => {
        //     
    }
    return (
        <App>
            <Button onClick={onClick}></Button>
            <Child1 list={items}/>
            <Child2 list={items}/>
        </App>
    }
}
```

If we want state (`list`) to propagate down into the 2 Child function components we need to do something like this:

```js
const onClick = () => {
    items.push({id:1})
    setRebuilds([...items])
}
```

This would see those nested components automatically updated (re-rendered) when referencing that prop (`list`).

However, if you did this:
```js
const onClick = () => {
    items.push({id:1})
    setRebuilds(items)
}
```

you would see the props in the child function components change but any components referring these props would not re-render.

Is this odd behaviour?  Good question.  I believe that as we're only passing an object literal reference, React doesn't see it as a changed value.  It does however, if we pass in a `value` (new state) and not a Object reference.  This `value` can also be a computed function (`param => param + 1`)

## What are Hooks?

Hooks is a new way of hooking into React functionality without having to write a class.  Hooks were introduced in React 16.8.  These Hooks can only be used from React Functions.  There's a new function called `useState`.  You use javascript destructuring to make available a new state variable and a `setter`. You can use the State Hook multiple times in a React Function For example:

```js
import React, {useRef, useState} from "react"
...
const [item, setItems] = useState([])
const [current, setCurrent] = useState(0)
```


## To render JSON object

There is a 3rd parameter that you can set when using the [`JSON.stringify`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) method.  This is the number of spaces to use for indentation.  For this to render correctly though, you need to wrap it in `<pre>` element as illustrated here:

```html
<pre>
{JSON.stringify(prop.rebuildList[0], null, 2)}
</pre>
```

## How to access input field value with a Hook

Prior to Function Components you would use react.createRef() to establish a reference to a html element and Function Component code.  With React Hooks you can use this short hand:

```js
let myInput = useRef()
```

Full example:
```js
import React, {useRef} from "react";

const Input = (props) => {

    const myInput = useRef()    

    const onClick = () => {
        props.onSave(myInput.current.value)
    }

    return (
        <div>
            <input type="text" ref={myInput} />
            <button onClick={onClick}>Save</button>
        </div>
    )
}

export default Input
```

## What is a custom Hook?

convention includes prefixing function name with `use`.  For example, `use`Whatever


## What are the Rules of Hooks

2 Rules:

- Must be at top level of Function, ergo, no int any nested arrow function.
- Must not be called from within loops, conditions or nested functions.

## What is `Dispatch`?

`Dispatch` relates for `Redux`.  Although, often used together, `Redux` is not part of `React`.



## References

- [create-a-new-react-app](https://reactjs.org/docs/create-a-new-react-app.html)
- [npx](https://www.npmjs.com/package/npx)
- [create-react-app](https://www.npmjs.com/package/create-react-app)
- [hooks](https://reactjs.org/docs/hooks-overview.html)
- [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
- [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
- [Kata](https://github.com/garrardkitchen/katas/blob/master/react-function-components.md)



