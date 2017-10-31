---
layout:       post
title:        "Learn React Note - Day1"
subtitle:     "Take notes anything are important"
date:         2017-10-30 12:00:00
author:       "Luyi"
header-img:   "img/in-post/post-eleme-pwa/eleme-at-io.jpg"
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - React
---

#### Using State Correctly 

```js
// Wrong
this.state.comment = 'Hello';
```
Instead, use `setState()` 
```js
// this will trigger rerender
// Correct
this.setState({comment: 'Hello'});
```

#### Do Not Modify State Directly
```js
// Wrong
this.state.comment = 'Hello';
```
```js
//Correct
this.setState({comment: 'Hello'});
```

##### State Updates May Be Asynchronous
```js
// state could be Asynchronous, so the counter just waiting for each other
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

```js
// use function rather than object, it will receive prevState for update
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

#### Bind `this` on callback
 
 You have to be careful about the meaning of this in JSX callbacks. In JavaScript, class methods are not bound by default. If you forget to bind this.handleClick and pass it to onClick, this will be undefined when the function is actually called.
 
```js
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }
```