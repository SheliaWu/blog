---
title: js事件循环机制
date: 2019-04-14
tags: microtask macrotask
---
Javascript是一个单线程的脚本语言，所谓的单线程是代码的执行是按顺序执行的，前面的代码执行后才能执行后面的代码。这样的同步执行在实际使用中很不方便，如果有些代码执行了大量运算，会导致后续代码难以执行，甚至网页出现假死。异步的出现，解决了这个问题。通常异步操作会注册一个回调函数，当异步操作完成后，会通过回调函数通知我们，这样我们就不用一直去轮询操作是否完成。但是如果程序在执行其他事件函数，异步完成了也需要等待程序空 下来才能执行。

## macrotask和microtask
在js中，浏览器的任务分为两种，一种是宏任务(macrotask)，另一种是微任务(microtask)。

宏任务通常是普通的js代码，通常是以下几种 ：
* script整体代码
* 浏览器事件
* 定时事件(setTimeout setInterval setImmediate)
*  I/O操作
* ui渲染

微任务主要是一下的代码操作：
* Promise
* Object.observe(已弃用)
* MutaitionObserver

在学习vue的源代码中，在vue.$nextTick()中，优先使用了微任务的操作推入一个timerFunc,在当前栈执行完毕以后执行nexttick传入的function。

## Event Loop，执行栈，任务队列

执行栈，当一个函数调用时就会形成一个调用帧并压入栈中，当函数返回时，则帧弹栈。

任务队列，每一个任务都包含一个处理该任务的函数，当任务产生时，任务及其处理函数会被作为一个整体推入任务队列中。任务队列按照先进先出的顺序执行。当任务队列中的任务需要被处理时，将会被移出队列，调用其处理函数，这是形成一个调用帧，并压入执行栈。当执行栈唯恐时，会接着处理任务队列中的接下来一个任务。

事件循环值的是主线程从任务队列中循环读取事件并调用执行。

在事件循环模型中，首先从task queue中选择最先进入的task执行，每执行完一个task都会检查microtask queue是否为空，若不为空则执行完microtsk queue中的所有任务。然后再选择task queue中最先进入的task执行，以此循环。

![事件模型](../uploads/eventloop.jpeg)

## async和await
async函数是promise的一个语法糖，简单理解为：await中的语句相当于在promise.resolve()中；await后面的语句相当于.then中的语句