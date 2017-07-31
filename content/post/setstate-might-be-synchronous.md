+++
draft = false
categories = []
tags = ["React"]
date = "2017-07-30T00:11:16-07:00"
title = "React: setState Might Be Synchronous"
bigimg = ""
subtitle = ""

+++

> This post is targeted toward React 15. React 16 (fiber) is around the corner and the API might change.

For a while, I always believed that `setState` in React is asynchronous. If I have two `setState`s in the same function, they should be batched, and be applied after the function returns. For example, 
```
handleClick = () => {
    console.log(this.state.count); // assume it is 0
    this.setState({count: this.state.count + 1});
    this.setState({count: this.state.count + 1});
}
```

Since the two `setState`s are asynchronous, the first `setState` doesn't change `this.state.count` right away, and the result will be `this.state.count === 1`. Right? 

Not quite. If the `setState` is called outside of React's lifecycles, that include any native event callbacks from DOM listeners (added through `addEventListner`), `setTimeout`, `setInterval`, `requestAnimationFrame` and any server fetch callbacks, will trigger a rerender immediately. This is because react is idle most of the time. When a `setState` is called, React will first check if there is an update batching going on, if yes, then add the current update to the batch. If not, React will go ahead and execute the update. 

See the example below. When click on the page, it is expected, that the state will change from 0 to 1, but instead it will change to 2.

<iframe src="https://codesandbox.io/embed/lGR4m0LM?autoresize=1&hidenavigation=1" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

In most cases, synchronous `setState` is not a bad thing. A lot of beginners are baffled by the async nature of `setState`. If you still want to batch all the `setState`s when you call them outside of React. you can use a hidden API of ReactDOM, `unstable_batchedUpdates`. I know, this API sounds scary with that `unstable` prefix. But if you create a wrapper, and only use the wrapper in your project, you can easily change it if any api changes in the future. 

This is how you would use it: 

```
import ReactDOM from 'react-dom';

...

handleClick = () => {
    ReactDOM.unstable_batchedUpdates(() => {
        this.setState({count: this.state.count + 1});
        this.setState({count: this.state.count + 1});
    });
}
```

<iframe src="https://codesandbox.io/embed/Q0lqAlBzl?autoresize=1&hidenavigation=1" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

### Bonus point

Angular's answer for events happening outside of lifecycle is through a library called [`Zone.js`](https://github.com/angular/zone.js/). It cleverly patchs all browser APIs such as `setTimeout`, `addEventListener`, etc, to interact with Angular. With that, developers would never (unless you want to squeeze more juice out of Angular's performance) need to worry about where a change occurs and whether Angular can catch it.

