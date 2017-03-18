+++
title = "TypeScipt, Where is my This"
subtitle = "TypeScript `this` undefined error"
draft = false
categories = []
tags = ["Typescript", "Object Oriented"]
bigimg = ""
date = "2017-03-18T00:27:33-04:00"

+++

I run into a bug while developing TypeScript recently. The code can boil down into the following snippet.

```typescript
class MyClass {
	color = "red";
    loop() {
        console.log(this.color);
        requestAnimationFrame(this.loop);
    }
}

let obj = new MyClass();
obj.loop();
```

If you are not familiar with [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame), it's just a method that takes a callback. This callback will be executed in the near future when the browser is ready to redraw.

The code looks right from an class-based Object Oriented point of view. The newly created MyClass instance has a loop method, it gets called, and print the color of the instance, which is `red`, it schedule a time for this method to run again, so you expect to see console flooded with the word `red` right? Not quite.

Console prints

```
red
undefined
Uncaught TypeError: Failed to execute 'requestAnimationFrame' on 'Window': The callback provided as parameter 1 is not a function.
    at MyClass.loop (<anonymous>:7:9)
```

The reason is that, even though TypeScript has class syntax (same for ES6), under the hood, it still uses prototype in JavaScript. A method is loosely tied to the prototype of an object. Object knows about the method, but method doesn't know who its owner is, until its invocation.

When a method is invoked, it's finally told about it's owner, owner will be assigned to the `this`. Note that `this` is a keyword, you cannot assign anything to it.

If you invoke the method using an object, for example `obj.loop()`, `this` for `loop` will be `obj`. If a method is invoked by itself, `this` by default will be set to a Global Object. In the context of HTML document, the Global Object is `window`. The Simplest example to tryout is

```Javascript
// Javascript
window.color = "red";
function printColor() {
	console.log(this.color);
}
printColor();
```

In JavaScript, we all learned that `this` is very dangerous. Sometimes this would change. Traditionally, we would do somthing like

```Javascript
...
var self = this; // save a reference to 'this'
setTimeout(function () {
	console.log(self.color); // in anonymous function, 'this' is changed
}, 1000);
...
```

But when we have all the goodies such as class declaration, we (at least myself) tend to treat JavaScript like a class-based Object Oriented language. That's why it caused this bug. So what's the fixed? Back to the first code snippet, there are multiple ways to fix this. 

One way is to `bind` the `this` context to the function.

```javascript
requestAnimationFrame(this.loop.bind(this));
```

Or we can use the new arrow function, which preserves the context.
```JavaScript
requestAnimationFrame(() => this.loop());
```

Or we do it the old fashion way, by saving a reference to the context.

```javascript
let self = this;
requestAnimationFrame(function () {
    self.loop();
});
```

If you are curious about whether the same thing happens in class based Object Oriented languages, such as C#, I did a little experiment.

C# has a similar concept of function reference call [delegate](https://msdn.microsoft.com/en-us/library/900fyy8e.aspx).

```cs
public delegate void MyDelegate();
public class Program
{
    public string color = "red";
    public MyDelegate del = null;

    public void Run()
    {
        this.del();
    }

    public static void Main(string[] args)
    {
        Apple apple = new Apple();
        Program pro = new Program();
        pro.del = apple.PrintColor;
        pro.Run();
    }
}

public class Apple{

    public string color = "yellow";

    public void PrintColor()
    {
        Console.WriteLine(this.color);
    }

}
```

The program prints `yellow`. which means that delegates in C# remember it's owner.