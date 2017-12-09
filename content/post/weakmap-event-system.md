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
button.addEventListener('click', listener);
```

This simple line of code tells us a couple things:

1. When `button` is garbage collected, all the listeners on the button should be detached and possibly garbage collected too.
2. listener will not be garbage collected as long as it is still listening to the event.
3. To detach the listener, we have to manually do it. In JavaScript, it is `button.removeEventListener('click', listener)`.

With those in mind, let's create our own event system. This is our API:

```JavaScript
const eventSystem = {
    add(target, event, listener) {
        // attach the listener to target and subscribe to the event
    },
    remove(target, event, listener) {
        // detach the listener
    },
    fire(target, event) {
        // calls all the listeners attached to the target that subscribed to the event
    }
};

// examples

// let's assume we have a microwave object
const microwave = new Microwave();

// we will do somthing when Microwave finish cooking
const takeOutTheFood = function(event, target) {
    console.log('Food is ready, lets take it out');
    // also remove the event listener
    eventSystem.remove(microwave, 'finished', takeOutTheFood);
};

// attach listener to microwave
eventSystem.add(microwave, 'finished', takeOutTheFood);


// let's simulate a event
eventSystem.fire(microwave, 'finished'); // We should see console logged 'Food is ready, lets take it out'
```

In this example, the target is the microwave, event is `'finished'`, and listener is `takeOutTheFood` function.

We need to design a data structure to store the connection between target, event and listener. One of the requirement is: We shouldn't stop target being garbage collected when we attach listener to it. In another word, target shouldn't know about the listeners that subscribe to its events. We can use a WeakMap for that. 

```JavaScript
const eventSystem = {
+   wm: new WeakMap(),
    add(target, event, listener) {
+       let listeners = this.wm.get(target);
        //  we will add the listener to listeners 

+       this.wm.add(target, listeners);    
    },

...

};

...
```

For all the listeners of a target, we also need to organize them by event. Since events are just strings in our example, we can just use a simple map. And for all the listeners of an specific event, we can use a ES 6 Set.

```JavaScript
const eventSystem = {
+   wm: new WeakMap(),
    add(target, event, listener) {
        let listeners = this.wm.get(target);
+       if (listeners === undefined) {
+           listeners = {};
+       }
+       let listenersForEvent = listeners[event];
+       if (listenersForEvent === undefined) {
+           listenersForEvent = new Set();
+       }
+       listenersForEvent.add(listener);
+       listeners[event] = listenersForEvent;
        this.wm.add(target, listeners);
    },
...

};

...
```

With this structure, it is very easy to remove a listener and call a listener. Let's fill up the `remove` and `fire function.

```JavaScript
const eventSystem = {
    wm: new WeakMap(),
    add(target, event, listener) {
        let listeners = this.wm.get(target);
        if (listeners === undefined) {
            listeners = {};
        }
        let listenersForEvent = listeners[event];
        if (listenersForEvent === undefined) {
            listenersForEvent = new Set();
        }
        listenersForEvent.add(listener);
        listeners[event] = listenersForEvent;
        this.wm.add(target, listeners);
    },

    remove(target, event, listener) {
+       let listeners = this.wm.get(target);
+       if (!listeners) return;
+       let listenersForEvent = listeners[event];
+       if (!listenersForEvent) return;
+       listenersForEvent.delete(handler);
    },
    
    fire(target, event) {
+       let listeners = this.wm.get(target);
+       if (!listeners) return;
+       let listenersForEvent = listeners[event];
+       if (!listenersForEvent) return;
+       for (let handler of handlers) {
+           setTimeout(handler, 0, event, target); // we use a setTimeout here because we want event triggering to be asynchronous. 
+       }
    }

}

```

That's it. Now we have a simple event system, built using WeakMap. When an event target is garbage collected, all the listeners will be detached thanks to the property of WeakMap.
