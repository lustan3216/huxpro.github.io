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


3.14 
15 92 65 一隻鸚鵡站在酒店小二身上手上拿著尿壺躲在鞋櫃裡面 
35 89 79 珊瑚上面有著芭蕉圖案的氣球掛在沙發上  
32 38 46 嫦娥和山胞一起坐在客廳裝上吃糧食
26 43 38 溜冰鞋和濕傘被山胞拿著掛在電視上
32 79 50 窗外看出去嫦娥拿著氣球跟武林高手打架
 
28 84 19 惡霸坐在巴士裡面後面跟著一台救護車想要衝去煮飯
71 69 39 奇異果和牛角麵包三角褲磁鐵在冰箱上面
93 75 10 軍人丟完垃圾拿著十字架坐在餐桌上
58 20 97 我爸愛吃鵝蛋，爸爸也有香港腳趴在電磁爐上面
49 44 59 有一個石臼和石獅子印在棺材上面放在洗碗朝裡面
 
23 07 81 駱駝背007騎著去找護士聊天躲在書櫃裡面
64 06 28 螺絲放在桌上突然就有一隻逗留跑過去逛惡霸
62 08 99 我躲在衣櫃拉著牛耳旁邊圍著籬笆看著99乘法表
86 28 03 躺在床上吃芭樂旁邊有隻惡狗發出鈴當聲
48 25 34 坐在沙發上玩骰子拉二胡喝沙士
 
21 17 06 鱷魚躲在洗手曹再用儀器看著遠方的鬥牛
79 82 14 氣球被白鵝叼著 醫師躲在浴缸
80 86 51 淋浴間裡面有個巴黎鐵塔上面長著芭樂還有烏魚子
32 82 30 嫦娥坐在馬桶上旁邊白鵝後面撞來了山石
66 47 09 鏡子旁邊掛著溜溜球，看到一個司機坐在菱角