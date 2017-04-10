+++
draft = true
categories = []
date = "2017-03-30T00:59:40-04:00"
title = "Angular 2 State Aware Pipes"
tags = []
bigimg = ""
subtitle = "A brief discussion about impure pipes"

+++
> This post will discuss the usage of impure pipe, begin by some introduction. If you know what impure pipes are, go ahead and jump to XX


In Angular 2, we can use pipes to transform data in the template. One of the builtin pies that I like the most is the `json` pipe. 

```xml
<div>{{myObject | json}}</div>
```

This will display the content of the object, great for debugging.

A pipe is essentially a `transform` function. This function takes a value, and returns a new value after transformation. very simple. But the class that exports this function can contain other information. What if it has a local variable, that remembers the times the transform function is called (not that useful from what I see, but might be a shortcut in some niche cases)? Things gets interesting.

So I went out and tried to create a pipe with local variable. 

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'count'
})
export class CountPipe implements PipeTransform {
  count = 0;
  constructor() {
    console.log('pipe created');
  }
  transform(value: any): any {
    this.count++;
    return value;
  }
}
```

This pipe will,
1. print `pipe created` every time it is created.
2. increment the `count` when `transform` is called.

I use this pipe twice in the template. 

```
{{123 | count}}
{{456 | count}}
```
But I only see one `pipe created` in the console. That means only one pipe instance is created, and shared across the template. The local variables of the pipe class is also shared. We can't count individually how many times each pipe is called.

To make sure Angular creates separate pipe instances, we need to use `impure` pipes. 

## Impure Pipe

To be honest, impure sounds like a word for describing smelly code. In fact, Angular documentation also advise the minimum usage of impure pipes. 

> implement an impure pipe with great care. An expensive, long-running pipe could destroy the user experience.

However, if implemented right, an impure pipe could greatly simplify the code we write.

Pipes are pure by default, which means they only get executed when the input value changes. Impure pipes on the other hands, are executed in every change detection. Angular is very smart about change detection. Change detection only happens when possible data modifications might happen. These include when input changes, when events get fired, when promise resolves, when async calls finish, etc. Which means, as long as we keep our pipes simple, and limit their usage in the same page, there shouldn't be too much performance drag.

## Tween Pipe

One interesting usage of impure pipe is tweening. Say we have a game UI that shows how many coins we currently have in numbers. When we gain some coin, we want the UI to reflect the change through a rapid digit changing effect. 