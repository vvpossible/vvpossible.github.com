---
layout: post
category : technical
tagline: "a simple web socket implementation"
tags : [tcp/ip, c, programming]
---
{% include JB/setup %}


最近一直在忙些杂七杂八的东西，code几乎没怎么写，感到手有些生疏了。于是决定找个小程序锻炼一下，保持状态。websocket看着挺有意思的，协议本身不复杂，浏览器不需要任何插件就可以和server用tcp通信，所以决定实现一个简单的websocket server。

需求很简单，就是能够和浏览器实现websocket的通信，对上层的应用来说，看到的就是client端送的raw数据。

于是每天下了班就写上一两个小时，断断续续的弄了两个星期，这个周末终于弄了个可以work的原型，用chrome浏览器测试了一下，可以很好的work，下一步决定写个上层的简单的聊天应用，顺便学习一下javascript的编程。

这次写了这个东西，最大的收获不是程序本身，而是学习了git和github，以及trigger了我要开始写技术博客的冲动。希望这是个好的开始，以后争取每天写点东西。

source code上传到了git hub上，地址是[https://github.com/vvpossible/toy_websocket](https://github.com/vvpossible/toy_websocket)。





