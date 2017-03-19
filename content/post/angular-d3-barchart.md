+++
subtitle = "with the separation of concerns"
draft = false
categories = []
tags = ["Angular2", "D3"]
date = "2017-03-07T01:16:50-05:00"
title = "Let's make a barchart using Angular 2 and D3"
bigimg = ""

+++

[Angular 2](https://angular.io) is great, [D3](https://d3js.org/) is also great, but when they are together, uh, not so great. Angular is a framework for mordern front-end development. It promotes a declarative approach to construct document structure. One thing that Angular 2 doesn't like is modifying the DOM directly. D3 represents Data-Driven Documents, on its website, it describe itself as

> a Javascript library for manipulating documents based on data.

In other words, it loves modifying the Dom.

That's a problem, both framework/library do a great job in manipulating the Dom(although I prefer Angular's way), which one should have control? A lot of examples I saw online let D3 handles all of DOM manipulation. In this post, I will try to use another approach. I will let D3 handle mostly the calculation, while let Angular handle the DOM manipulation. Let's make a simple bar chart.

Some of the ideas are from this [awesome article](http://alexandros.resin.io/angular-d3-svg/)

## Setup

I use [Angular CLI](https://cli.angular.io/) to bootstrap the project. As the time of writing, it is 1.0.0-rc.1. For more information on how to use Angular CLI, please check ou the [doc](https://github.com/angular/angular-cli)

starting a new project with Angular CLI is very easy, after installing, just run

```bash
ng new BarChart
cd BarChart
ng serve
```

We also need D3. Angular 2 uses TypeScript, so we need the type definition for D3. We can install D3 using following commands:

```bash
npm install --save d3
npm install --save-dev @types/d3
```

For this demo, I am using Angular `2.4.0`, D3  `4.7.1`.

## Create BarChartComponent

Run generator to create a component skeleton

```
ng generate component bar-chart
```
There are some [life cycle hooks](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html) that will be important for intergrating Angular 2 and D3.  
- **OnChange** ngOnChange is called when a input value changes. This is when we call D3 to recalculate the values. Very important to note that, it will be called even before the dom for the component is created. If you want to use D3 to modify the Dom (as we will talk about later), you need to make sure the dom exists.
- **AfterViewInit** This is called when the Dom of the oomponent becomes available(the current component's Dom, not necessarily the children's Dom). If we need to use D3 to do some complex SVG generation, we need to make sure we do that after ngAfterViewInit is called.
- **OnDestroy** You might not need this, because Angular is smart enough to clear all the content even if it's created by D3. But if you have some on-going service upon destruction of the component, it might be safe to clean it up in ngOnDestroy to prevent memory leak.

One more thing to keep in mind is that, svg doesn't allow any properties that it doesn't understand. So we cannot bind attributes like `height`, `width` directly. If you do that, it will complain  
![error](http://res.cloudinary.com/yiou-me/image/upload/post-angular-barchart/error.png)  
So we need to bind everything using [attribute binding](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#other-bindings).

Ok, let's code this component.

###### d3-bar-chart.component.html
```xml
<svg [attr.height]="height" [attr.width]="width">
  <g [attr.transform]="transform">
    <rect 
      *ngFor="let item of data" 
      [attr.height]="barHeights[i]" 
      [attr.width]="barWidth" 
      [attr.x]="xCoordinates[i]"  
      [attr.y]="0" 
      fill="skyblue">
    </rect>
  </g>
</svg>
```
###### d3-bar-chart.component.ts
```typescript
import { Component, Input, OnChanges } from '@angular/core';
import * as D3 from 'd3';

export type Datum = {name: string, value: number};
@Component({
  selector: 'app-d3-bar-chart',
  templateUrl: './d3-bar-chart.component.html',
  styleUrls: ['./d3-bar-chart.component.css']
})
export class D3BarChartComponent implements OnChanges {
  
  @Input() height = 300;
  @Input() width = 600;
  @Input() data: Datum[] = [];
  @Input() range = 100;

  xScale: D3.ScaleBand<string> = null;
  yScale: D3.ScaleLinear<number, number> = null;
  transform = '';
  chartWidth = this.width;
  chartHeight = this.height;
  barHeights: number[] = [];
  barWidth = 0;
  xCoordinates: number[] = [];
  
  // Input changed, recalculate using D3
  ngOnChanges() {
	this.chartHeight = this.height;
    this.chartWidth = this.width;
    this.xScale = D3.scaleBand()
      .domain(this.data.map((item: Datum)=>item.name)).range([0, this.chartWidth])
      .paddingInner(0.5);
    this.yScale = D3.scaleLinear()
      .domain([0, this.range])
      .range([this.chartHeight, 0]);

    this.barWidth = this.xScale.bandwidth();
    this.barHeights = this.data.map((item: Datum) =>this.barHeight(item.value));
    this.xCoordinates = this.data.map((item: Datum) => this.xScale(item.name));
    
    // use transform to flip the chart upside down, so the bars start from bottom
    this.transform = `scale(1, -1) translate(0, ${- this.chartHeight})`;
  }

  clampHeight(value: number) {
    if (value < 0) {
      return 0;
    }
    if (this.chartHeight <= 0) {
      return 0
    }
    if (value > this.chartHeight) {
      return this.chartHeight;
    }
    return value;
  }

  barHeight(value) {
    return this.clampHeight(this.chartHeight - this.yScale(value));
  }

}
```
###### d3-bar-chart.component.css
```
:host {
    display: block;
    height: 300px;
    width: 600px;
}

svg {
    height: 100%;
    width: 100%;
}
```
We can use this component as follow in app.component.ts

```xml
<app-d3-bar-chart [data]="data"></app-d3-bar-chart>
```
For this example, I just generated some random data:
```typescript
data = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
    .map((month: string) => ({
      name: month,
      value: Math.random() * 100
    }));
```
The result will be similar to the following image

![barchart basic](http://res.cloudinary.com/yiou-me/image/upload/post-angular-barchart/barchart1.png)

I need to point out something here. As you can see below, I defined the yScale using an opposite domain and range. I also calculate the height as the difference between `chartHeight` and the return value of the scale.
```typescript
// inside ngOnChanges
this.yScale = D3.scaleLinear()
      .domain([0, this.range])
      .range([this.chartHeight, 0]);
// inside barHeight
return this.clampHeight(this.chartHeight - this.yScale(value));
```
<p id="axis-at-bottom">The reason for this is so that the origin(zero) of y-axis is at the bottom. Basically I flipped the bars upside down. When we change the value for each bar, we can see the top of the bar moving up or down, instead of the bottom. This might not be obvious, but when we add the transition, you will be able to see the effect.</p>

Another thing is, I tried to use local variable to cache some calculation results such as `transform` . By default, [angular change detection](https://blog.thoughtram.io/angular/2016/02/22/angular-2-change-detection-explained.html) will check all the template bindings every time. If a function or getter is used in the template, the function/getter will be called every time.

## Create Axes
Let's add the x and y axes to the chart.
D3 provides a very powerful tool for generating SVG axes called [d3-axis](https://github.com/d3/d3-axis). For axes, we would actually want to let D3 to handle the Dom manipulation, because it will be too much trouble to reinvent the wheel.

We cannot use custom components in svg elements, because svg is not html. It is xml. If it sees any tags that it doesn't understand, it will yell at you. But we can use Angular [directives](https://angular.io/docs/ts/latest/guide/attribute-directives.html).

Generate a template for directive
```bash
ng generate directive d3-axis
```
and fill in the directive
```typescript
import { Directive, Input, AfterViewInit, OnChanges, ElementRef } from '@angular/core';
import * as D3 from 'd3';
@Directive({
  selector: '[appD3Axis]'
})
export class D3AxisDirective {
  
  @Input() scale: any;
  @Input() orientation: 'vertical' | 'horizontal' = 'horizontal';
  initialized = false;
  constructor(private el: ElementRef) {}

  drawAxis() {
    switch (this.orientation) {
      case 'horizontal':
        D3.select(this.el.nativeElement).call(D3.axisBottom(this.scale));
        break;
      case 'vertical':
        D3.select(this.el.nativeElement).call(D3.axisLeft(this.scale));
    }
  }

  ngAfterViewInit() {
    // all the Inputs will be set before this gets called.
    // D3 needs to wait for view init to modify it
    this.initialized = true;
    this.drawAxis();
  }

  ngOnChanges() {
    if (this.initialized) {
      this.drawAxis();
    }
  }

}
```
This is very straight forward. We will use this directive on a `<g>` element. To get a reference to the native Dom, we inject it into the constructor. `D3.axisLeft` and `D3.axisBottom` both take a scale and generate the ticks and labels. We are allowed to call them every time input changes, because they will remove the old axes if there are any, and create new ones. Make sure we call `drawAxis` after `ngAfterViewInit` because that's when the Dom becomes available.

We can use this directive in our `BarChartComponent`, but before that, we need to make some changes. We need to leave some space at the left and bottom of the chart for the axes. let's add two inputs to the `BarChartComponent`  
```typescript
 @Input() paddingLeft = 30;
 @Input() paddingBottom = 20;
```
We should also change the calculation for `chartHeight` and `chartWidth`.
```typescript
// ngOnChanges in d3-bar-chart.component.ts

// chartWidth = this.width;
// chartHeight = this.height;
chartWidth = this.width - this.paddingLeft;
chartHeight = this.height - this.paddingBottom;
```

Now we will change the `transform` to make sure we leave room for the axes. We will also create two more transforms for the axes.

```typescript
// ngOnChanges in d3-bar-chart.component.ts
this.transform = `scale(1, -1) translate(${this.paddingLeft}, ${- this.chartHeight})`;
this.axisBottomTransform = `translate(${this.paddingLeft}, ${this.chartHeight})`;
this.axisLeftTransform = `translate(${this.paddingLeft}, 0)`;
```
Finally, we add the directive to the template of `BarChartComponent`.  
The final template is 
```xml
<svg [attr.height]="height" [attr.width]="width">
  <g [attr.transform]="transform">
      <rect 
        *ngFor="let item of data; let i=index" 
        [attr.height]="barHeights[i]" 
        [attr.width]="barWidth" 
        [attr.x]="xCoordinates[i]" 
        [attr.y]="0" 
        fill="skyblue">
      </rect>
  </g>
  <g class="axis"
    [attr.transform]="axisBottomTransform" 
    appD3Axis orientation="horizontal" [scale]="xScale"></g>
  <g class="axis"
    [attr.transform]="axisLeftTransform"
    appD3Axis orientation="vertical" [scale]="yScale"></g>
</svg>

```
Remember we said [before](#axis-at-bottom), I intentially set the `yScale` to start from the bottom. By doing that, the generated y axis will also start from bottom.

![chart2](http://res.cloudinary.com/yiou-me/image/upload/post-angular-barchart/barchart2.png)

## Make it responsive
Next step, we will make the chart auto-resizable when the container size change. Even though SVG stands for Scalable Vector Graphics, doesn't mean it can auto scale (we can use [viewbox](https://css-tricks.com/scale-svg/#article-header-id-4), but that will scale the text too). My approach here is to watch for container size using [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) and change the height and width inputs on the `BarChartComponent`. Let's make it into a directive, so that we can reuse it for other charts too.

Create an auto-resize directive
```bash
ng generate directive auto-resize
```
###### auto-resize.directive.ts
```typescript
import { Directive, AfterViewInit, ElementRef, OnDestroy } from '@angular/core';

@Directive({
  selector: '[appAutoResize]',
  exportAs: 'autoResize'
})
export class AutoResizeDirective implements AfterViewInit, OnDestroy {
  height = 0;
  width = 0;
  requestId = null;
  constructor(private el: ElementRef) { }

  ngAfterViewInit() {
    let checkDimension = () =>{
      this.height = this.el.nativeElement.clientHeight;
      this.width = this.el.nativeElement.clientWidth;
      this.requestId = window.requestAnimationFrame(checkDimension);
    }
    // If call the following line here, error will be thrown in debug mode
    // "Expression has changed after it was checked."
    // checkDimension();

    this.requestId = window.requestAnimationFrame(checkDimension);
  }

  ngOnDestroy() {
    if (this.requestId != null) {
      window.cancelAnimationFrame(this.requestId);
    }
  }
  
}
```

This directive will detect the size change of any container. The method `checkDimension` will be called during each animation frame. What is does is simply reading the height and width from dom, and then assigning them to the local variables. We can then bind the height and width to `BarChartComponent`.

Let's update the `app.component.html`
```xml
<div>
  <app-d3-bar-chart 
    [data]="data"
    appAutoResize
    #resizer="autoResize"
    [height]="resizer.height"
    [width]="resizer.width"
    ></app-d3-bar-chart>
</div>
```
We exported the directive as `autoResize`, so that we can reference it in the template.

The height and width will be used to bind to inputs, so we need to be very careful not to changed them when Angular finishes the change detection. Note that I didn't call `checkDimension` inside `ngAfterViewInit`, this is because in dev mode, Angular adds another check to make sure that no input has changed after the change detection. Angular will call `ngOnChanges`,`ngDoCheck` from parent(`BarChartComponent`) before `ngAfterViewInit` of child(`AutoResizeDirective`). If we change the height and width in child's `ngAfterViewInit`, Angular will complain.

But we are allowed to change them in `requestAnimationFrame`, because `requestAnimationFrame` is actually called before change detections. If you want to know more about Angular 2's change detection, there is a thorough [explanation](https://blog.thoughtram.io/angular/2016/02/22/angular-2-change-detection-explained.html) on that.

Now, if we set the width of the chart to 100%, we can see it auto-resizing. 
###### app.component.css
```css
app-d3-bar-chart {
    width: 100%;
}
```

## Transition? perhaps?
What about transitions, can we animate the bars when we change the value of it? Of course we can, but this is gonna be hard. Before we talk about transition, we need to talk about how we want to change the data. The input `data` is an object(array). For an object, by default Angular will only check the reference. That is, during change detection, Angular compare the new data and the old data with [strict equality](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness#Strict_equality_using) (===). If we changes the value of one element inside the array, `ngOnChanges` will not be called, and the heights of the bar will not be recalculated. To invoke `ngOnChange`, we need to make sure everytime we change the value, we make a shallow copy of the array. 
```typescript
// any event handler in app.component.ts
this.data[0].value = Math.random() * 100;
this.data = this.data.slice();
```
But even if we get the changes, we update the heights and widths of the bars, how do we add the transition? Can we just use css transition? It turns out we can, but only in chrome.
###### d3-bar-chart.component.css
```css
rect {
    transition: height 1s ease, width 1s ease;
}
```
To make it work cross platform, we need to use [D3-transition](https://github.com/d3/d3-transition). I will leave it to another post coming up. That's it for this post. There is still a lot to improve. But this is just a proof of concept. It shows that it's possible to separate the responsibilities when using Angular 2 and D3. I will write more about this in the future.

If you want to see the complete code. You can view this [github repo](https://github.com/yiochen/BarChart)
