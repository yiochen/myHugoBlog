+++
draft = false
categories = []
date = "2017-03-30T00:59:40-04:00"
title = "Angular 2 State Aware Pipes"
tags = []
bigimg = ""
subtitle = "A brief discussion about impure pipes"

+++

> This post will discuss the usage of impure pipe, begin by some introduction. If you know what impure pipes are, go ahead and jump to [here](#tween-pipe)


In Angular 2, we can use pipes to transform data in the template. One of the builtin pies that I like the most is the `json` pipe. 

```xml
<div>{{myObject | json}}</div>
```

This will display the content of `myObject`, great for debugging.

A pipe is essentially just a `transform` function. This function takes a value, and returns a new value after transformation. very simple. But the class that exports this function can contain other information. What if it has a local variable, that preserve the state inside the pipe, such as remembering the times the transform function is called (not that useful from what I see, but might be a shortcut in some niche cases)? Things gets a bit interesting.

So I tried to create a pipe with local variables. 

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

1. Print `pipe created` every time it is created.
2. Increment the `count` when `transform` is called.

I use this pipe twice in the template.

```
{{123 | count}}
{{456 | count}}
```
But I only see one `pipe created` in the console. That means only one pipe instance was created, and shared across the template. The local variables of the pipe class is also shared. We can't count individually how many times each pipe is called.

To make sure Angular creates separate pipe instances, we need to use `impure` pipes. 

## Impure Pipe

To be honest, impure sounds like a word for describing smelly code. In fact, Angular documentation also advise the minimum usage of impure pipes. 

> implement an impure pipe with great care. An expensive, long-running pipe could destroy the user experience.

We should use Impure pipes with caution. Everytime when we need a pipe, we should first consider if the same thing can be done using pure pipes.

That being said, let's see how to create a impure pipe.

```typescript
import { Pipe, PipeTransform, OnDestroy, ChangeDetectorRef } from '@angular/core';

@Pipe({
  name: 'count',
  pure: false
})
export class CountPipe implements PipeTransform, OnDestroy {

  count = 0;
  interval: any;

  constructor(private _ref: ChangeDetectorRef) {
    this.interval = setInterval(()=> {
      this.count ++;
      this._ref.markForCheck();
    }, 1000);
  }

  transform(value: any): any {
    return value + this.count;
  }

  ngOnDestroy() {
    clearInterval(this.interval);
  }
}

```

This is an very simple impure pipe. Basically what it does is the following. It keeps a internal timer (count), and increment it every second. When used in the template, it will increment the value by the seconds passed. For example: 

```html
<div>
	{{0 | count}}
</div>
```

You will see the value inside the div will increment by 1 every second, starting from 0.

To make a pipe impure, we need to do a couple things.

1. in the metadata, we need to define the pipe as impure using `pure: false`. This will tell Angular to call this pipe during every change detection.  
2. A lot of time, impure pipe involves some internal data (it has to save the state somewhere). Make sure to free any resource and prevent memory leak when the pipe is not in use.
3. Notice the injected `ChangeDetectorRef`. Impure pipe by default will be checked every time during change detection. **Unless** the parent component sets the `ChangeDetectionStrategy` to `OnPush`. For example: 
```typescript
// parent component
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
```
`OnPush` tells Angular to do change detection only when the inputs change. If the inputs are unchanged, your impure pipe cannot cause the view to rerender.

	That's why if we have any asynchronous operations in the pipe, when it finishes, we need to notify the change detector to check again using `markForCheck`. Note that, by doing that, the change detection will propogate from the child (this pipe) to the parent (the component that uses this pipe in the view). This might sounds like defeating the purpose of  `ChangeDetectionStrategy.OnPush`. But think about it, it's only adding one more change detection every second (in this example), so it's not that bad.

## Tween Pipe

One interesting usage of impure pipe is tweening (I am looking into doing the same thing with Angular Animation, might write a new post later). Say we have a game UI that shows how many coins we currently have in numbers. When we gain some coin, we want the UI to reflect the change through a rapid digit changing effect. There is a naive way. We just maintain a `currentValue` in the component, representing the currenly displayed coin count. Whenever the input value change, we set a timer to change the `currentValue` every couple milliseconds, until it became the same as the input value. Then we will remove the timer. See the following structure. 
```typescript
// template

<div> Coins: {{currentValue}} </div>

// class

@Input() value: number;
private currentValue: number;
ngOnChange(changes: SimpleChanges) {

	if (changes['value']) {
    	// set timer to update currentValue 
    }

}

ngOnDestroy() {
	// clean any timer still running.
}
```

But this is just a simple visual effect. We added so many more states to this component just for this effect. If we ever want to change the behavior, we need to make an aweful lot of changes to this code.

So I thought, maybe we can keep all those states in a pipe. I call it a tween pipe.
```typescript
import { Pipe, PipeTransform, ChangeDetectorRef } from '@angular/core';
import * as D3 from 'd3';

@Pipe({
  name: 'tween',
  pure: false
})
export class TweenPipe implements PipeTransform {

  cached: string = 'null';
  currentValue: any;
  timer: D3.Timer;
  ease = D3.easeCubic;

  constructor(private _ref: ChangeDetectorRef) { }

  transform(value: any, time = 750): any {

    let newValue = JSON.stringify(value);

    if (newValue !== this.cached) {
      this.stop(); // stop previous tween if not finished
      let interpolator = this.startInterpolating(JSON.parse(this.cached), JSON.parse(newValue), time, this.ease);
      this.cached = newValue;
      this.currentValue = interpolator(0);
    }

    return this.currentValue;
  }

  startInterpolating(from: any, to: any, time: number, easeFunc: any) {
    let interpolator = D3.interpolate(JSON.parse(from), JSON.parse(to));

    this.timer = D3.timer((elapsed: number) => {
      if (elapsed < time) {
        this.currentValue = interpolator(easeFunc(elapsed / time));
        this._ref.markForCheck();
      } else {
        this.currentValue = interpolator(1); // when the time runs out, set the final value
        this._ref.markForCheck();
        this.stop();
      }
    });

    return interpolator;
  }

  stop() {
    if (this.timer) {
      this.timer.stop();
      this.timer = null;
    }
  }

}

```

Here I use D3 for the timer and interpolation. You can simply replace it with `setInterval` or `requestAnimationFrame`.
