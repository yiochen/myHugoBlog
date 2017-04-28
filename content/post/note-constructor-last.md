+++
categories = []
tags = []
date = "2017-04-20T21:22:16-04:00"
title = "ES6 Class Constructors are Called Last"
bigimg = ""
subtitle = ""
draft = false

+++

> I found that writing long posts are very time consuming, and I am too lazy to do that. I will try some of this small notes more often.

I started to learn React. Coming from a Angular 2 background, I immediately fell in love with the powerful functional way of writing template in JSX. I think React is much better than Angular in this aspect. I might write a comparison of this two framework/library when I get more familiar with React. 

When I was reading React's [tutorial](https://facebook.github.io/react/docs/handling-events.html). I found it using ES6 class constructor in a very interesting way.

```javascript
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    ...
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    ...
  }
  ...
}
```

This class has a method call `handleClick`. When we created a instance of this class, for example `let toggle = new Toggle(props);`, we can reference it using `toggle.handleClick()`. But in the constructor, it sets the the `handleClick` method again. Which definition of `handleClick` takes precedence?

This has to do with how ES6 class is compiled into ES5 functions. I tried using the [TypeScript Playground](https://www.typescriptlang.org/play/) and found that, a simple class like the following: 

```javascript
class Greeter {
    constructor() {
        this.greet = this.greet.bind(this);
    }
    greet() {
        return "Hello world";
    }
}
```

will be compiled into the following ES5.

```javascript
var Greeter = (function () {
    function Greeter() {
        this.greet = this.greet.bind(this);
    }
    Greeter.prototype.greet = function () {
        return "Hello world";
    };
    return Greeter;
}());
```

The class `Greeter` is actually the returned value of a [Immediately-invoked function expression](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression). 
When the above code runs, the following steps happen in order.

1. A function `Greeter` is defined, but not called.
2. the `Greeter` function gets assigned a `greet` property, which is a function.
3. the `Greeter` function is returned and assigned to the global `Greeter` variable.

Note that the `Greeter` function is not called, so `this.greet = this.greet.bind(this);` never happened.

Finally, when we call `let greeter = new Greeter();`, we override the object's `greet` method with a new definition.

How is this useful? As used in React's tutorial, it could be a way to modify the class function, or from what I see, it could be used to provide a default function and let users provide a custom one if they wish. See the following example:

```javascript
class Reporter {
    constructor(reportFunc) {
        if (reportFunc) {
            this.report = reportFunc;
        }
    }
    report(message) {
        console.log(message);
    }
}

let normalReporter = new Reporter();

let reportFunc = (message) => console.log(`***FANCY MESSAGE: ${message}***`);
let fancyReporter = new Reporter(reportFunc);

normalReporter.report('Hello world'); // Hello world
fancyReporter.report('Hello world');  // ***FANCY MESSAGE: Hello world***
```