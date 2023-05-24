---
title: 大厂面试最爱问的Event Loop，一次性弄懂它！
date: 2023-05-05 12:33:21
tags: 基础知识
---

> 由于 node.js 里的 Event Loop 跟浏览器端不太一样，一两句说不清。篇幅有限，这里就暂且只探讨面试常问的浏览器端的 Event Loop 了 ：）

## 缘起
最近在准备换工作，也大概看了下平时掌握不深的 JavaScript 事件循环机制（Event Loop）。

但是在今天字节面的笔试中，一道题还是扫到了我的知识盲区。

先看题，输出以下代码的打印结果：
``` js
async function async1() {
    console.log('1');
    await async2();
    console.log('2');
}

async function async2() {
    console.log('3');
}

console.log('4');
async1();

setTimeout(() => {
    console.log('5');
}, 0)

new Promise((resolve, reject) => {
    console.log('6');
    resolve();
}).then(() => {
    console.log('7');
})

console.log('8');
```

看到这里，你可能已经在内心有了你的答案。在继续往下看之前，你也可以先去浏览器控制台验证下你的答案。

如果你全对，那么恭喜你，这块已经难不倒你了，你甚至可以直接跳过本文；如果你像我一样，谨小慎微还是错了一两个地方，那么继续往下看还是很有必要的。

这道题目大家一看就知道是在考 Event Loop，那么下面就先讲述下我自己对Event Loop的理解，然后再跟大家对下这道题的思路。如果我理解有误，也欢迎在评论区告诉我，我会及时弄清楚并纠正。

## Event Loop的背景：JS 单线程

> JS 是单线程的，这是因为设计之初它主要就是为了用来与用户互动，以及操作DOM的，而不是有现在这么大的作用。

由于**JavaScript是单线程语言**的特性，所以 JS 同一时间只能做一件事情。如果有很多任务就必须排队，等一个任务处理完才能处理下一个任务。

这种处理任务的方式，我们称之为`同步`。

常见的同步任务和异步任务我们分下类：
-   `同步任务`：比如声明语句、for、赋值等，读取后依据从上到下从左到右，立即执行
-   `异步任务`：比如 ajax 网络请求，setTimeout 定时函数等都属于异步任务。异步任务会通过事件任务队列(Event Queue)机制（先进先出的队列机制）来进行协调执行

如果所有 JS 代码都当做同步任务处理不仅效率低，还会引发很多问题。例如：协调事件、用户交互、脚本执行、UI 渲染和网络处理等行为被阻塞了，就会影响页面加载效率，造成页面卡顿甚至假死。

为了优化 JS 单线程阻塞的问题，于是就引入了`异步`任务处理机制。异步任务的特点是，暂缓返回任务执行结果。

> 如果一段代码，在执行时还不能够得到预期结果，而是需要在将来通过一定的手段拿到，那么这块代码就是异步任务。

JS 处理同步任务、异步任务的流程大致如下：

![彻底搞懂JavaScript事件循环 \- 掘金](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b1d13b5c77746f783045cd2f9dc1815~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

也就是说，异步任务不进入JS单线程，而是放在`任务队列`中。若有多个异步任务，也是需要在任务队列中排队等待的。

> 任务队列类似于缓冲区，任务下一步会被移到执行栈然后JS线程执行调用栈的任务。

前面说到：如果所有代码不分同步异步都放在主线程依次排队执行，会影响页面的渲染效率。那异步任务依次排队执行，有没有类似的问题呢？

答案是有的。

## Event Loop的基础：宏任务 & 微任务

异步任务，各个任务执行时间长短也差别很大。有的是 setTimeout 这种耗时很久的，有的是 promise 这种耗时较短的。

当异步任务很多的时候，耗时久的就会阻塞后面所有的异步任务，包括一些很快可以执行完的也被阻塞。

于是 JS 引擎就将异步任务分类管理，划分成两个队列：**宏任务队列** 和 **微任务队列**。

宏任务（macroTask）有：
- `<script async>`标签中的运行代码（异步任务）
- setTimeout、setInterval的回调函数
- 事件触发的回调函数，例如`DOM Events`、`I/O`、`requestAnimationFrame`、Ajax、UI交互等

微任务（microTask）有：
- Promise的回调函数：then、catch、finally
- MutationObserver：[使用方式](http://javascript.ruanyifeng.com/dom/mutationobserver.html)
- queueMicrotask：[使用方式](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/queueMicrotask)
- process.nextTick：Node独有

为了方便记忆，我们可以这么理解：宏任务是JavaScript 引擎（浏览器或 Node.js）发起的任务，微任务是 JavaScript 代码本身发起的任务

宏任务、微任务都是异步任务，而同步任务立即执行，不需要放入任务队列。

## Event Loop的原理：宏任务与微任务交替执行

为了既不阻塞JS单线程的执行，同时又保障JS执行的效率，前辈们设计了出了一套`事件循环机制`：
- 将 document 下 script 标签中的**所有同步代码都放入执行栈**，立即执行
- 执行过程中如果产出新的宏任务/微任务，就将他们**推入相应的任务队列**
- 等执行栈没有代码可以执行**之后再执行微任务队列**
- 微任务队列都执行完以后，又开始执行宏任务队列（执行微任务队列）
- 如此循环，**不断重复的过程就叫做 Event Loop(事件循环)** 

![一次搞懂\-JS事件循环之宏任务和微任务 \- 掘金](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52bec546cf0748f9b89f5ca537d77baa~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

串起来的过程为：
> 可执行代码 -> 微任务队列 -> 宏任务 -> 微任务队列 -> 宏任务 ......

讲到这里，稍微也提下 Node.js 里的 Event Loop 其实跟浏览器端的不太一样。区别是：
-   Node端：microTask 在事件循环的各个阶段之间执行
-   浏览器端：microTask 在事件循环的 macrotask 执行完之后执行

![diff.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dcbef5d30ea448684c98f38b5cf3d6d~tplv-k3u1fbpfcp-watermark.image?)

概念大家已经理解了，建议大家可以结合自己日常代码来研究下 Event Loop 的过程来巩固掌握。

由于我们开头已经抛出过一个比较经典的Event Loop面试题了，这里就不另外举例了。

## 回到开始的题目

这道平平无奇的题目，除了主要考事件循环，还考了 promise 的链式调用、async/await 函数的返回值。

有了上面的知识储备，我们再看开头的题目就显得思路清晰了。

> 先揭晓打印结果：4 => 1 => 3 => 6 => 8 => 2 => 7 => 5

一起来仔细盘下这道题 JS 事件循环的执行过程（建议打开右侧控制台对照题目进行）：
- 首先，所有的代码都会被推到执行栈执行。如果遇到异步任务会根据宏/微任务类型将其推到相应的任务队列
- 首先打印 4，因为 `async1`、`async2` 两个函数开始只是声明，没有调用
- 然后就是打印 1。看到 async 不要慌，async函数里的内容大都是同步执行的（除了 await 后面是作为 promise 来执行的）
- 然后就是打印 3。调用函数 async2，里面的代码会立即执行，但不会立即返回结果到async1 的 await 那里
- 然后就是打印 6。为什么不是 2，因为打印 2 这个任务在 await 后面，是会被推进了微任务队列的
- 为啥是 6 也解释下，Promise的函数体是同步立即执行的，只有 then、catch、finally 这些回调才是异步调用的（见上文）
- 此时的回调函数打印 7，也被推进了微任务队列，放到打印 2 后边异步执行
- 然后就是打印 8，毫无疑问是同步执行
- 同步任务至此已经全部完成，宏任务执行完了便开始执行微任务队列的微任务了
- 首先是打印 2，它最开始被推进微任务队列
- 其次是打印 7，初始化 promise 的时候将它推进微任务队列的（因为回调是异步执行的），这个时候微任务队列也已经执行完
- 最后是打印 5，根据前面所讲 setTimeout 是一个标准的宏任务，微任务队列执行完了就要执行宏任务队列

总结了下，有几个重点需要关注下：
- 同步任务是会立即执行的，不会被加入到任务队列中，更不会进行Event Loop
- 微任务总是优先于宏任务执行，每个宏任务执行完就一定会立即执行微任务队列
- 宏任务都是逐个来执行的，微任务都是以队列的方式来执行的（参考结尾的面试题结果）
- new Promise 中的代码是同步的，但是回调函数则是异步的（微任务）
- `async/await` 是 JavaScript 中使用同步代码来处理异步的一种方式，它本身并不是宏任务或微任务，但 `async` 函数将返回一个 Promise 对象。执行 async 函数执行时，一旦遇到await就会先返回一个Promise对象，等到await后的操作完成后，再接着执行async函数体内的语句
- setTimeout() 的第2个参数是为了告诉 JavaScript 再过多长时间把当前任务添加到宏任务队列中

## 缘灭
因为下份工作不想太卷，所以本来也没打算冲刺字节的，甚至说是对字节这个面试毫无准备。

但不知道是哪位猎头不讲武德，偷偷摸摸给我投了。

我只能说，当时大意了，没有闪！

没想到简历还过了，我就当顺便练练手，为后续面试积累下经验吧！

---
再补充一道网上看到的面试题，这个题含多个宏任务的执行，能帮助你更清晰的理解 Event Loop：
``` js
Promise.resolve().then(function() {
    console.log(1);
})
console.log(2); 
setTimeout(function () { 
    console.log('setTimeout1'); 
    Promise.resolve().then(function () { 
        console.log('promise'); 
    }); 
});
setTimeout(function () { 
    console.log('setTimeout2'); 
    Promise.resolve().then(function () { 
        console.log('promise2'); 
    }); 
});
// 输出 2 1 setTimeout1 promise setTimeout2 promise2
```

有兴趣大家也可以在评论区讨论交流下~
