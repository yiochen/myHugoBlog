+++
subtitle = ""
date = "2017-11-04T13:25:38-07:00"
title = "Custom Jasmine Asymmetric Matchers"
draft = false
categories = []
tags = []
bigimg = ""

+++

Jasmine provides some very convenient asymmetric matchers such as `any`, `anything` and `objectContaining`. Using them can make your test more robust.

For example:

```JavaScript
var foo = {
      a: 1,
      b: 2,
      bar: "baz"
    };

// check if object foo has a field bar and if foo.bar === "baz"
expect(foo).toEqual(jasmin.objectContaining({
    bar: "baz"
}));
```

Just like `objectContaining`, all the asymmetric matchers use custom logic to compare the test element and the expected result instead of just basic equality check. Jasmine has very good [documentation](https://jasmine.github.io/2.2/introduction?spec=jasmine.any#section-Matching_Anything_with_<code>jasmine.any</code>)for the usage of their matchers, so I will not try to explain them here.

As far as I know, Jasmine doesn't have any official documentation on creating custom asymmetric matcher. I looked inside the code and found that it is actually very easy to implement one.

These are the requirement: 

1.  An asymmetric matcher needs to be a function.
2.  The asymmetric matcher needs to expose a field `asymmetricMatch`, which is a function that takes two arguments. The first argument is the test element (the one you wrap inside `expect`); the second argument is `customTesters`, which is [another way](https://jasmine.github.io/2.2/custom_equality.html) to achieve custom comparison. For the scope of this article, we will not use the second argument.
3.  You can provide optional reporting when test fails by providing a `jasmineToString` method.

Let's say we need to test phone numbers, but the phone numbers we get are in different formats. For example (I am using American phone numbers), `123-456-7890`, or `(123)456-7890`. We can't simply `toEqual` because that will do string comparison and the two numbers will be considered not equal. 

We can create a custom asymmetric matcher that normalize the difference before doing the comparison. 

<p data-height="425" data-theme-id="dark" data-slug-hash="wPGxZp" data-default-tab="js,result" data-user="yiochen" data-embed-version="2" data-pen-title="Jasmine Asymmetric Matcher" class="codepen">See the Pen <a href="https://codepen.io/yiochen/pen/wPGxZp/">Jasmine Asymmetric Matcher</a> by Yiou Chen (<a href="https://codepen.io/yiochen">@yiochen</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>