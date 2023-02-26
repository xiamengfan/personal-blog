---
title: 浅谈JS的事件循环
date: 2021-01-30 23:06:31
tags: JavaScript
---

# 为什么有事件循环
首先，JS中的同步任务和异步任务是按怎样的顺序去执行，这依赖于事件循环，因为事件循环本质上就是为了协调多线程任务的进行；
然而我们要知道，事件循环本身并不是JS引擎负责的工作，JS作为单线程的脚本语言，是没有能力去协调多线程的任务执行顺序的；
回想JS最初的作用，其实就是静态网页为了实现和用户交互所引入进来的，通过键盘、鼠标的输入来执行一段代码，而监听这些输入事件，其实是浏览器的行为，而不是JS的；
所以这么一看就很明显了，所谓JS的事件循环，其实是JS所嵌入的环境来定义并实现的，而且根据JS所嵌入的环境，具体的实现也不一样，比如浏览器端和Node服务端的差别就很大；
这里主要介绍一下，在浏览器端，如何通过事件循环来与多个事件源进行交互；
也就是如何协调用户交互（鼠标、键盘）、脚本（如 JavaScript）、渲染（如 HTML DOM、CSS 样式）、网络等行为等事件的执行顺序；
（而在Node端，是没有鼠标、键盘等事件，也没有渲染，多了文件I/O事件，详情可以查看[Node官网](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)）

# 浏览器包含的进程：
- **Browser进程(浏览器的主进程，负责协调、主控，只有一个)：**
负责浏览器的界面显示，与用户交互，如前进，后退等
负责各个页面的管理，创建和销毁其它进程
将Rendered进程得到的内存中的Bitmap,绘制到用户界面上
网络资源的管理，下载
- **第三方插件进程：**
每种类型的插件对应一个进程，仅当使用该插件时才创建。
- **GPU进程：**
最多一个，用于3D绘制等。
浏览器渲染进程（浏览器内核）（Render进程，内部是多线程的）：
默认每个Tab页面一个进程，互不影响。主要作用为：页面渲染，脚本执行，事件处理等
在浏览器中打开一个网页相当于新起了一个进程（进程内有自己的多线程）

# 重点是浏览器内核（渲染进程）
对于普通的前端操作来说，最重要的渲染进程：页面的渲染，JS的执行，事件的循环等都在这个进程内执行;
浏览器是多进程的，浏览器的渲染进程是多线程的；

## GUI渲染线程
负责渲染浏览器界面，解析HTML,CSS,构建DOM树和RenderObject树，布局和绘制等。
当界面需要重绘或由于某种操作引发回流时，该线程就会执行。
注意，GUI渲染线程与JS引擎线程是互斥的，当JS引擎执行时GUI线程会被挂起（相当于冻结了）,GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。

## JS引擎线程
也称为JS内核，负责处理JavaScript脚本程序。（例如V8引擎）。
JS引擎线程负责解析JavaScript脚本，运行代码。
JS引擎一直等待着任务队列中任务的到来，然后加以处理，一个Tab页（render进程）中无论什么时候都只有一个JS线程在运行JS程序。
同样注意，GUI渲染线程与JS引擎线程是互斥的，所以如果JS执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。

## 事件触发线程
归属于浏览器而不是JS引擎，用来控制事件循环（可以理解成JS引擎自己都忙不过来，需要浏览器另开线程协助）。
JS的同步任务都在主线程上执行，形成一个执行栈，主线程之外，事件触发线程管理着一个任务队列，只要异步任务有了运行结果，就在任务队列之中放置一个事件
当JS引擎执行代码块如setTimeout时（也可来自浏览器内核的其它线程，如鼠标点击，AJAX异步请求等），会将对应任务添加到事件线程中。
当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎空闲时再处理。

一旦执行栈中的所有同步任务执行完毕（此时JS引擎空闲），系统就会读取任务队列，将可运行的异步任务添加到可执行栈，开始执行。

## 定时触发器线程
传说中的setTimeout和setInterval所在的线程
浏览器定时计数器并不是由JavaScript引擎计数的，（因为JavaScript引擎是单线程的，如果处于阻塞线程状态就会影响计时的准确）
因此通过单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）
注意，W3C在HTML标准中规定，规定要求setTimeout中低于4ms的时间间隔算为4ms。

## 异步http请求线程
在XMLHttpRequest在连接后是通过浏览器新起一个线程请求
将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中，再由JavaScript引擎执行

# 总结浏览器的渲染流程
Browser主进程收到用户请求（输入url），首先需要获取页面内容（通过网络下载资源）,随后将该任务通过RendererHost接口传递给Render渲染进程
Render渲染进程的Renderer接口收到消息，简单解释后，交给渲染线程GUI，然后开始渲染
GUI渲染线程接收请求，加载网页并渲染网页，这其中可能需要Browser主进程获取资源和需要GPU进程来帮助渲染
当然可能会有JS线程操作DOM（这可能会造成回流并重绘）
最后Render渲染进程将结果传递给Browser主进程
Browser主进程接收到结果并将结果绘制出来

# 回到浏览器的渲染进程（render进程）
到此时，已经是属于浏览器页面初次渲染完毕后的事情，接下来，详细介绍JS代码是如何去执行的；

各种浏览器事件同时触发时，肯定有一个先来后到的排队问题。决定这些事件如何排队触发的机制，就是事件循环。这个排队行为以 JavaScript 开发者的角度来看，主要是分成两个队列：
- 一个是 JavaScript 外部的队列。外部的队列主要是浏览器协调的各类事件的队列，标准文件中称之为 Task Queue。下文中为了方便理解统一称为外部队列。
- 另一个是 JavaScript 内部的队列。这部分主要是 JavaScript 内部执行的任务队列，标准中称之为 Microtask Queue。下文中为了方便理解统一称为内部队列。

## 外部队列
外部队列（Task Queue），顾名思义就是 JavaScript 外部的事件的队列，这里我们可以先列举一下浏览器中这些外部事件源（Task Source），他们主要有：
- DOM 操作 (页面渲染)
- 用户交互 (鼠标、键盘)
- 网络请求 (Ajax 等)
- 定时器 (setTimeout 等)

可以观察到，这些外部的事件源可能很多，为了方便浏览器厂商优化，HTML 标准中明确指出一个事件循环由一个或多个外部队列，而每一个外部事件源都有一个对应的外部队列。不同事件源的队列可以有不同的优先级（例如在网络事件和用户交互之间，浏览器可以优先处理鼠标行为，从而让用户感觉更加流程）

## 内部队列
内部队列（Microtask Queue），即 JavaScript 语言内部的事件队列，在 HTML 标准中，并没有明确规定这个队列的事件源，通常认为有以下几种：
- Promise 的成功 (.then) 与失败 (.catch)
- MutationObserver

## 处理模型
![事件循环处理模型](/images/JSEventLoop/event-loop.jpeg "事件循环处理模型")

### 示例

``` JavaScript
setTimeout(() => {
    console.log(0);
})

var fn = function () {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            console.log(1);
            resolve(2);
        })
    }).then((value) => {
        console.log(value)
    })
}

fn();
```

``` HTML
<html>

<body>
    <pre id="main"></pre>
</body>
<script>
    const main = document.querySelector('#main');
    const callback = (i, fn) => () => {
        console.log(i)
        main.innerText += fn(i);
    };
    let i = 1;
    while (i++ < 5) {
        setTimeout(callback(i, (i) => {
            return '\n' + i + '<';
        }))
    }

    while (i++ < 10) {
        Promise.resolve().then(callback(i, (i) => {
            return i + ',';
        }))
    }
    console.log(i)
    main.innerText += '[end ' + i + ' ]\n'
</script>

</html>
```

``` JavaScript
async _sleep(time = 0) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve();
        }, time);
    });
}
```
``` JavaScript
this.isLoader = true;
await this._sleep();
// other异步任务
this.isLoader = false;
```

### Promise 和 async/await
[Promise 源码分析](https://zhuanlan.zhihu.com/p/58428287)
[async/await 源码分析](http://www.ruanyifeng.com/blog/2015/04/generator.html)

其实，script 本身就是是一个外部队列事件，先执行所有内部事件，接着执行渲染，再去执行一个外部事件，满足模型的执行流程；