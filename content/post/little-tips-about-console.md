+++
draft = false
categories = []
tags = ['JavaScript', 'Browser']
bigimg = ""
subtitle = ""
date = "2017-07-11T20:35:50-07:00"
title = "Little Tips about Console"

+++

I just read an [article](https://christianheilmann.com/2017/07/08/debugging-javascript-console-loggerheads/) by Christian Heilmann. He did a poll on Twitter and it showed that 67% of developers uses `console.log` to debug their JavaScript instead of some more advanced tools such as debugger or breakpoints. I will shamelessly say that, I am one of those 67%. In this short post, I will share some tips about console that I discovered recently.

### [console.clear](https://developer.mozilla.org/en-US/docs/Web/API/Console/clear)

[`console.clear()`](https://developer.mozilla.org/en-US/docs/Web/API/Console/clear) clear everything from the console. This is extremely useful when you are using some online editors such as [CodePen](https://codepen.io/). CodePen will keep running the JavaScript everytime you make a change, while all the error messages and console logs are just appended to the console. Having a `console.clear()` at the top of your JS panel will make debugging much easier.

![console.clear](../../post-images/console_clear.PNG)

Note that in Node's interactive console, there is no `console.clear`. To clear the screen, you can use

```JavaScript
process.stdout.write('\033c');
```

`033c` is an escape code for reseting the terminal. This might or might not work depending on your platform. For more explanation, see this [StackOverflow answer](https://stackoverflow.com/a/5367075/3429675).

### [console.group](https://developer.mozilla.org/en-US/docs/Web/API/Console/group)

You can group your console logs, and each group will appear as a collapsible section in dev console. 
The syntax for group is
```JavaScript
console.group([label]); // label is an optional argument for the name of the group
// your console logs
console.groupEnd(); // groupEnd will close the current group.
```

Using `group` will initially render the groups expanded. If you want them collapsed, just replace `console.group` with [`console.groupCollapsed`](https://developer.mozilla.org/en-US/docs/Web/API/Console/groupCollapsed).

You can also nest groups to create some really complex console outputs (Really? I would rather spend my time feeding cats).

![console.group](../../post-images/console_group.png)

### Substitution in console.log

Before we discuss substitution, do you know `console.log` can take multiple arguments? Like you can do `console.log(firstNumber, '+', secondNumber, '=', sum)`, so you don't need to use string concatenations.

I just found that `console.log` supports substitution. 
```JavaScript
console.log(msg [, subst1, ..., substN]);
```
Here is an example
```JavaScript
console.log('PI is %f', Math.PI); // PI is 3.141592653589793
```
For more information on substitution, check [here](https://developer.mozilla.org/en-US/docs/Web/API/console#Using_string_substitutions).

However, when I test, chrome doesn't respect some more advanced formatting, for example `console.log('PI is %.2f', Math.PI)` will give the same result, instead of truncating the number.