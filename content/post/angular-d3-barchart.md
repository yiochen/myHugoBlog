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
