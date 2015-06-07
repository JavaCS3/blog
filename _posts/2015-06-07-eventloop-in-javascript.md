---
layout: post
category: javascript
title: Chrome Developer Tool 探究 Javascript 事件循环
---

### 前言
熟悉Javascript的朋友们对事件循环应该不陌生。目前绝大多数的Javascript的运行环境都是单线程的。要想实现一些异步回调的方法，就不得不用到事件循环了。然而，目前很多的文章中对Javascript的事件循环仅限于一些图解，为了方便大家更加深刻的理解事件循环，本文将利用Chrome Developer Tool从时间轴上证明事件循环的这一机制。

### 简单的栗子
先来看一段代码：

```html
<!DOCTYPE html>
<html>
<head>
  <title>EventLoop Sample</title>
  <script type="text/javascript">
    function onClick() {
      console.log('In onClick...');
    }
  </script>
</head>
<body>
  <button onclick="onClick()">Click</button>
</body>
</html>
```

这段代码的功能很简单，就是当鼠标点击按钮的时候调用`onClick()`函数打印一个`log`而已。

然而，如果把这段内嵌的Javascript看成别的运行脚本(`Python`, `Shell`...)。直觉会告诉你，当函数`onClick`被解析完后整个程序应该退出才对。为啥我点下按钮时还会再触发一次？这就是事件循环搞得“鬼”，我会在下文中给出解答。

#### Timeline
打开Chrome Developer Tool(CDT)中的Timeline功能并点击Record按钮。看看这段代码在时间轴是什么样子的吧。

![pic_simple_case](http://pic.yupoo.com/javacs3/EHD5GqKD/8YlEV.png)

从图中可以很容易发现，当鼠标点击的时候触发了两个事件`mouseup`和`click`。当触发`click`事件的时候，函数调用栈才进入了我们的`onClick()`函数并且执行了另一个函数`console.log(...)`。我们还可以发现另一个现象————函数发生调用的过程中，被调函数总是会在调用函数的运行时间区间之内。通过这个时间分布图，其实我们可以很容易的去分析一个系统的具体运行流程图。接下来我们看一下另一个例子。

### 函数的直接调用
上代码：

```html
<!DOCTYPE html>
<html>
<head>
  <title>EventLoop Sample</title>
  <script type="text/javascript">
    function onClick() {
      console.log('In onClick...');
      callback();
    }
    function callback() {
      console.log('In callback...');
    }
  </script>
</head>
<body>
  <button onclick="onClick()">Click</button>
</body>
</html>
```

这个例子功能同样比较简单，只不过在`onClick`的调用中增加了一个新的`callback()`函数。那么这个情况的时间分布图是怎样的呢？

![direct_call](http://pic.yupoo.com/javacs3/EHDj4uYi/OwTl0.png)

这里可以发现一个小细节：同一个调用栈内的函数的时间分别总是在同一个层级上的。

### “异步”回调
我们经常会在`Javascript`中看到类似这样的代码：

```javascript
setTimeout(callback, 0); // WTF
```

不了解事件循环的话，看这段代码会非常费解。为什么不直接调`callback()`而是触发一个定时为0的Timeout。看到下面这个图大概你就明白了原理了。

上代码：

```html
<!DOCTYPE html>
<html>
<head>
  <title>EventLoop Sample</title>
  <script type="text/javascript">
    function onClick() {
      console.log('In onClick...');
      setTimeout(callback, 0);
    }
    function callback() {
      console.log('In callback...');
    }
  </script>
</head>
<body>
  <button onclick="onClick()">Click</button>
</body>
</html>
```

时间图：

![async_call](http://pic.yupoo.com/javacs3/EHDj4NZP/B1fYv.png)

可以发现虽然`setTimeout`的定时参数为Zero，但是实际上Timeout的回调大概在2ms以后再执行。夹在当中是浏览器的一些绘制事件。

所以`setTimeout(callback, 0)`的真正意图是为了提高UI的响应速度，把任务拆分放在后面的事件循环去做。不知道大家有没有明白其中的奥妙。

### 后记
如果大家有兴趣了解Javascript的时间循环机制，可以看看[这篇文章](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)and[视频](https://vimeo.com/96425312)。
