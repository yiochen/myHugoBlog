+++
subtitle = "experiment about the separation of concerns"
draft = false
categories = []
tags = ["Angular2", "D3"]
date = "2017-03-07T01:16:50-05:00"
title = "Let's make a barchart using Angular 2 and D3"
bigimg = ""

+++

[Angular 2](https://angular.io) is great, [D3](https://d3js.org/) is also great, but when they are together, uh, not so great. Angular is a framework for mordern front-end development. It promotes a declarative approach to construct document structure. One thing that Angular 2 doesn't like is modifying the DOM directly. D3 represents Data-Driven Documents, on its website, it describe itself as

> a Javascript library for manipulating documents based on data.

In another word, it loves modifying dom.

That's a problem, both framework/library do a great job in manipulating the DOM (although I prefer Angular's way), which one should have control? A lot of examples I saw online let D3 handles all of DOM manipulation. In this post, I will try to use another approach. I will let D3 handle mostly the calculation, while let Angular handle the DOM manipulation. Let's make a simple bar chart.

Some of the ideas are referencing from this [awesome article](http://alexandros.resin.io/angular-d3-svg/)

## Setup

I use [Angular CLI](https://cli.angular.io/) to bootstrap the project. As the time of writing, it is 1.0.0-rc.1. For more information on how to use Angular CLI, please check ou the [doc](https://github.com/angular/angular-cli)

starting a new project with Angular CLI is very easy, after installing, just run

```bash
ng new BarChart
cd BarChart
ng serve
```

We also need D3. Angular 2 uses TyoeScript, so we need the type definition for D3. We can install D3 using following commands:

```bash
npm install --save d3
npm install --save-dev @types/d3
```

For this demo, I am using Angular `2.4.0`, D3  `4.7.1`.

## Create BarChartComponent

run generator to create a component skeleton

```
ng generate component bar-chart
```
There are some [life cycle hooks](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html) that will be important for intergrating Angular 2 and D3.  
- **OnChange** ngOnChange is called when a input value changes. This is when we call D3 to recalculate the values. Very important to note that, it will be called even before the dom for the component is created. If you want to use D3 to modify the Dom (as we will talk about later), you need to make sure the dom exists.
- **AfterViewInit** If we need to use D3 to do some complex SVG generation, we need to make sure we do that after ngAfterViewInit is called.
- **OnDestroy** You might not need this, because Angular is smart enough to clear all the content even if it's created by D3. But if you have some on going service upon destruction of the component, it might be safe to clean it up in ngOnDestroy to prevent memory leak.
- 