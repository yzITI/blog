---

title: 【浏览器的特技】Javascript Workers
date: 2020-04-13 00:00:00
categories:

- 科普

tags:

- Web

- 前端

toc: true

---

> "浏览器本质上是很简单的，现在浏览器这么复杂是因为加了很多特技，想让所有应用都能在浏览器里跑。"——Nick
>
> 本文基本由Nick大佬讲述，我记录整理成文。

<!-- more -->

## 简介

Web worker, service worker和worklet是浏览器提供的可以让javascript多线程运行的框架，我就自作主张统称他们为Javascript workers。他们的工作原理类似但在使用上重叠不多。

众所周知，javascript在浏览器中表面上都是单线程的，通过事件循环减少阻塞异步加载，这样使JavaScript非常容易上手（多线程调试是非常，非常，非常狗屎的经验），但也制约了javascript发展，浏览器厂商就想给JavaScript加入多线程支持，但受制于历史糟粕又不能直接给javascript加入线程库，所以他们通过WHATWG在2009年左右提出了web worker的标准。

## Web Worker

我们先看第一种也是最朴素的web worker，他基本上只能用来处理计算。Web worke简单的说就是在浏览器里开另一个JavaScript虚拟机进程，这个进程上可以单独运行一个JavaScript脚本，处理一些耗时的操作，然后交给主线程渲染。例如很多html5游戏都是使用web worker处理一些计算操作，然后把具体的数据交给主线程呈现在网页上。

如果web worker使用wasm写就可以跑得起一些非常复杂的应用，例如一些3D游戏，或者在浏览器里训练神经网络

#### 使用

调用`Worker()`来建造一个新的worker。

```javascript
/* main.js */
const myWorker = new Worker('worker.js');
```

和其他类型workers一样，web workers不能直接操作DOM，所以需要使用`windows.postMessage`就可以传递消息

```javascript
/* main.js */
myWorker.postMessage("Hello World")
// worker处理接收消息
myWorker.onmessage (r) => {console.log(r.data)}
```

停止使用

```javascript
myWorker.terminate();
```

MDN给了一个简单的webworker脚本的例子https://github.com/mdn/simple-web-worker/blob/gh-pages/worker.js 。

在脚本里使用self表示worker本身，所以接收从主线程发来的消息是用`self.onmessage`和`self.postMessage`。

```javascript
/* worker.js */

// Receive message from main file
self.onmessage = function(e) {
  console.log(e.data);

  // Send message to main file
  self.postMessage(workerResult);
}
```

Web worker也可以有很多个。如果想知道最大数量可以在`firefox`里可以通过`about:config`查看。这个值可以调整，不过web worker一般执行计算密集型应用，太多也没什么意义 。

![F9357BFA5DCBD3B31940E9D07C6A6053.png](https://i.loli.net/2020/04/13/JzKLdrEDcApeUBj.png)



使用Web worker的好处是在计算密集型应用上，不会占用主线程进行计算，试想你在做游戏的时候如果主线程一遍渲染一遍计算那么你会觉得痛不欲生的，因为会很卡很慢，或者你在做一个图片处理应用，一旦开始处理图片用户就会发现网页失去相应，这都不是我们想要的效果。



## Service Worker

第二种要讲的是service worker，相比于web worker，它多了一个重要的能力：可以访问网络和浏览器缓存。Service Worker是作为一个浏览器，缓存以及网络直接的代理使用的。

因为前面讲到的那种是最早的worker，完全就是为了缓解主线程的负担而生，所以是完全用来计算的，关闭一个网页那么对应的web worker也会关闭，所有的数据都应该由主线程保存，所以人们又提出了service worker，service worker可以安装在浏览器里，自己发出请求然后储存在浏览器里。



打开浏览器开发者工具的application页，左边可以看到service workers这一项，里面就有不同的网站安装的service worker。

![serviceWorker2.png](https://i.loli.net/2020/04/13/ONZT6XsaJWwCPAb.png)



有一个非常著名的应用https://devdocs.io/ ，一个运行在浏览器里的文档查看器，点击左上角三个点，有一个offline data，你可以在浏览器里安装这些离线数据，这些数据会存在IndexedDB里，然后断网后service worker可以继续提供服务

比如这张图就是from ServiceWorker，在Network页可以看到两个不同的请求，一个是主线程的请求被ServiceWorker重定向到了ServiceWorker上，另一个是ServiceWorker从cache里取出实际的数据，service worker的请求会有一个齿轮标记在前面。



![seviceWorker.png](https://i.loli.net/2020/04/13/4tBVcGAMoY7dzJf.png)



实现也很简单，service worker通过监听三个事件完成，分别是`install`，代表安装进浏览器，`activate`，代表service worker被启用，`fetch`，是实际用来拦截主线程的请求的事件。

这个文档网站实际用来拦截的代码只有这么多。



![serviceWorker3.png](https://i.loli.net/2020/04/13/mIwjCNcLokylZxd.png)



#### 使用

方法与第一个类似，依然需要一个main和一个worker文件。

```javascript
/* main.js */

navigator.serviceWorker.register('/service-worker.js');
```

```javascript
/* service-worker.js */

// Install 
self.addEventListener('install', function(event) {
    // ...
});

// Activate 
self.addEventListener('activate', function(event) {
    // ...
});

// Listen for network requests from the main document
self.addEventListener('fetch', function(event) {
    // ...
});

self.addEventListener('fetch', function(event) {
    // Return data from cache
    event.respondWith(
        caches.match(event.request);
    );
});
```



## Worklet

第三种是worklet，worklet是最新的标准，还在测试，不过chrome 65以上实现了，worklet是一个专用的轻量级web worker，主要是用来调整渲染流水线。浏览器引擎需要通过多级流水线逐步渲染一个网页，分别是DOM，CSSOM，渲染树和布局，然后还有一个绘制。

worklet就是在这几个步骤上做钩子（钩子就是用来有限制的修改一个程序的内部状态用的，属于API的一种），添加进一些开发者需要的内容，有四种不同的worklet，分别是PaintWorklet，AudioWorklet，AnimcationWorklet和LayoutWorklet。

#### PaintWorklet

PaintWorklet操作的是图片，比如在PaintWorklet中可以像操作canvas一样画一个图，然后在css里用paint函数将在PaintWorker里定义的东西画出来，比如`background-image:paint(myPaintWorklet)`，myPaintWorklet是一段js代码。

使用这个worker我们依然需要main和myWorker两个文件

```javascript
/* main.js */

CSS.paintWorklet.addModule('myWorklet.js');
```

```javascript
/* myWorklet.js */

registerPaint('myPaintWorklet', class {
  paint(ctx, size, properties) {/* some shapes*/};
});
```

最后在CSS里面使用

```css
.div {
  background-image:paint(myPaintWorklet);
}
```



#### AudioWorklet

第二种AudioWorklet从名字就可以看出是用来操作音频用的，可以用它做出一些声音然后在网页里播放。具体可以看https://developers.google.com/web/updates/2017/12/audio-worklet 。

#### Animation Worklet

第三种Animation Worklet钩在了js的动画上，虽然这不是浏览器的渲染流水线的一部分但也是为了提高网页动画速度，可以做一些非常复杂的动画同时不占用主线程的时间。

这是一个草案中的提案，是web audio API的一部分，它可以在浏览器里做出mixer的效果，目标是游戏的音频引擎。

#### LayoutWorklet

最后一个是layoutWorklet，就是可以操作浏览器里的一些css 布局，这个没什么好说的，就是可以让javascript相应布局变化之类的，worklet都是最新标准，只有chrome和用chromium65以上的客户端支持，这些就是在浏览器里给javascript加速的一些组件。

另外还有一个重要的API，webGL，可以在浏览器里调用GPU绘制图像，和OpenGL紧密相连，有这些东西浏览器就可以运行游戏了，另外所有这些APIwasm都会支持，另外所有这些APIwasm都会支持，所以可以非常快的运行，在我刚刚给的那个devdocs.io里可以查到这些API的文档，如果觉得浏览器里用不太舒服，那就用nativefier把它变成一个应用程序https://www.npmjs.com/package/nativefier  就是个自动electron打包器。



## 结语

~~Nick大佬真的是太强了！~~此文只是对Javascript workers一个简单的介绍，想要在实际中运用得当，还是需要去查阅资料勤加练习。