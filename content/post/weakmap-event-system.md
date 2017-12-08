+++
bigimg = ""
date = "2017-12-07T23:46:42-08:00"
title = "Create a Event System Using ES6 WeakMap"
subtitle = ""
draft = false
categories = []
tags = []

+++

EcmaScript 6 came with WeakMap. [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) defined WeakMap as

> "a collection of key/value pairs in which the keys are weakly referenced.  The keys must be objects and the values can be arbitrary values."

`Weakly referenced` means that when garbage collector are counting the references to a WeakMap key (an object), it doesn't consider the WeakMap's reference to it.

In JavaScript, Object is very flexible. You can add a key to it, or you can delete the key. It accomplish pretty much everything a hashmap does in other languages. ES6 Map complements Object by allowing anything as the key. When I saw WeakMap, I feel confused about its usage. Is WeakMap just a poor-man Map? If so, why does it exist?

I spent some time thinking about it's usage and turns out that it is a very powerful tool.

WeakMap allows us to add metadata to an object. For example

```JavaScript
const objectFactory = {
    metadata: new WeakMap(),
    createObject() {
        const obj = {};
        this.metadata.set(obj, { created: new Date()}); // associate some metadata to the object.
        return obj;
    },
    getMetadata(obj) {
        return this.metadata.get(obj); // retrieve metadata
    }
}

let myObj = objectFactory.createObject();

console.log(objectFactory.getMetadata(myObj)); // {created: Fri Dec 08 2017 00:41:22 GMT-0800 (Pacific Standard Time)}

myObj = undefined; // remove the reference to the object, so that it will be garbage collected. 
```

WeakMap's key can be garbage collected. Using this property we can create an event system, similar to the one in the DOM.

In DOM, we usually add event listeners like:

```JavaScript
button.addEventListener('click', listenerFunction);
```

This simple line of code tells us couple things:

1. When `button` is garbage collected, all the listeners on the button should be detached and possibly garbage collected too.
2. 

