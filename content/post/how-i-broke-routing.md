+++
draft = false
categories = []
tags = []
bigimg = ""
subtitle = ""
date = "2017-12-23T22:50:27-05:00"
title = "How I Broke the Routing System of Our Enterprise App"

+++

Working in UIPlatform team, I was in charged of refactoring the legacy front-end routing system. When I finished, everything behaved the same as before (for the most part, except a dozen regression issues), except when a user download a document from our site, navigation will stop working after that.

How does downloading break the routing system? We were all confused. I will briefly explain our routing infrastructure first.

Our app is a giant single page app that uses JQuery, backbone and a home made routing system. We uses hashes in URL for routing. For example, a typical URL would be `https://domain.com/#currentpage`. Changing the hash part will not make a request to the server, because everything after the hash is ignored.

Each view is a Backbone component that extends a super class that we call `main_nav`. `main_nav` listens to browser hash change event. When a hash change event happens `main_nav`'s onHashChange function will be called. Other views that extends `main_nav` will override this function to do custom rendering.

As we are moving away from hashes in url, in favor of [HTML5 History API fallback](https://github.com/bripkens/connect-history-api-fallback#introduction). We wanted to use [`History.js`](https://github.com/ReactTraining/history) to ease the transition.

This is what I did: in `main_nav`, instead of directly listening to browser's hash change event, I changed it to listen to route change from `History.js`.

```
// main_nav
import History from 'history';

const history = History.createHistory(); 

const unlisten = history.listen(initView);
```

When I attach the listener, I get a function to unsubscribe. When should I unsubscribe to route change event? I ended up sticking it to [`window.onbeforeunload`](https://developer.mozilla.org/en-US/docs/Web/API/WindowEventHandlers/onbeforeunload). When window unload all the resource, we will unlisten for route changes. Make sense right? I learned from college to always clean up resource to prevent memory leak.

But the 