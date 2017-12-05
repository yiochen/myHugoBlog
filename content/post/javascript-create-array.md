+++
tags = []
bigimg = ""
subtitle = ""
draft = false
categories = []
date = "2017-12-04T23:11:33-08:00"
title = "JavaScript: Ways to create array of given length"

+++

How do you create an array in JavaScript and fill it with a constant? The most naive way would be:

```
const length = 10;
const arr = [];
for (let i = 0; i < length; i++ ) {
    arr.push(1);
}
```

But that seems too much work for such a simple task.

Dr. Axel Rauschmayer in his [blog](http://2ality.com/2013/11/initializing-arrays.html) discussed this problem. He gave some very interesting ways to accomplish this task. The original post was written in 2013. Since then, JavaScript has evolved a lot. This blog will discuss some other ways.

### Array.prototype.fill

This might be the easiest way to initialize an array with value. ES 6 provides [fill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/fill) method. So you can do:

```
new Array(3).fill('a') // ['a', 'a', 'a']
```

At the time of writing. All major browsers except IE supports this.

### Array.from
I saw [Ben Lesh](https://twitter.com/BenLesh) used this in one of his workshop and I think it's pretty cool. From MDN,

> The Array.from() method creates a new Array instance from an array-like or iterable object.

Well, what's `Array-like`?

In fact, it could be an just an Object with a `length` attribute. For example

```
Array.from({length: 3}); // [ undefined, undefined, undefined ]
```

With that we can initialize our array with `map`
```
Array.from({length: 3}).map(() => 'a');
```



