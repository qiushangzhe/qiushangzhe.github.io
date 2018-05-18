---
layout: post
title: egg源码窥视之cluster
date: 2018-05-18 00:00:00
category: code
tags: [web-server,egg.js]
---

## 这是什么

在看egg-cluster源码过程中的笔记。

<!-- more -->

## 知识点

#### 进程间通信（IPC）

IPC的方式通常有管道（包括无名管道和命名管道）、消息队列、信号量、共享存储、Socket、Streams等

1. 无名管道 UNIX 系统IPC最古老的形式 特点：半双工，具有固定的读端和写端。
2. FIFO，也称为命名管道
	- 可以在无关的进程之间交换数据。
	- FIFO有路径名与之相关联，它以一种特殊设备文件形式存在于文件系统中。
3. 消息队列，是消息的链接表，存放在内核中。一个消息队列由一个标识符（即队列ID）来标识。
	- 消息队列是面向记录的，其中的消息具有特定的格式以及特定的优先级。
	- 消息队列独立于发送与接收进程。进程终止时，消息队列及其内容并不会被删除。
	- 消息队列可以实现消息的随机查询,消息不一定要以先进先出的次序读取,也可以按消息的类型读取。

4. 不想了解了。

so： egg利用node的events模块作为IPC的通讯底层。具体node的events模块是那种IPC方式，不感兴趣。


## egg-cluster 架构

在diamante的lib/utils/messenger.js 中看到了这么一段图形的注释
```
/*
 *             ┌────────┐
 *             │ parent │
 *            /└────────┘\
 *           /     |      \
 *          /  ┌────────┐  \
 *         /   │ master │   \
 *        /    └────────┘    \
 *       /     /         \    \
 *     ┌───────┐         ┌───────┐
 *     │ agent │ ------- │  app  │
 *     └───────┘         └───────┘
 */
```
这里面明显提到了四个关键词：parent、master、agent、app

但是通过看代码，我只看到了三个关键词：agent、app、master

至于parent是什么东西。不得而知。感觉可能是node进程,因为在messenger调用sendToParent的方法的时候，函数内部的代码是`process.send(data)`

抛弃掉parent关键字，我来结合egg-cluster代码看看其他三种东西是什么。

### master

是整个egg进程中的主要管理进程。职责上，目前我看代码看到的有：

- 处理配置信息。 egg-cluster专门有一个options.js 去处理，并把结果返回给master。
- 创建并监听管理agent和app进程，会及时重启死掉的app进程和agent进程。并会根据cpu的个数创建对应的app进程。
- 监听系统事件，比如关闭进程等等。

总之是一个起调度作用的主进程。调度的对象为agent和app进程。

### agent

呐，代码中有这么一段注释：

```
/**
 * agent worker is child_process forked by master.
 *
 * agent worker only exit in two cases:
 *  - receive signal SIGTERM, exit code 0 (exit gracefully)
 *  - receive disconnect event, exit code 110 (maybe master exit in accident)
 */
```

从中可以看到agent进程退出的时机。

然后、然后就没了。整个agent_worker.js 里面能得到的信息就这么点。那agent进程到底是干嘛的。也许从这句代码可以看出来
```
const Agent = require(options.framework).Agent;
```
其他的agent逻辑都在外部framework中编写的。

so，egg-cluster中的agent只是一个进程对象的基类。具体的业务逻辑是通过类似插件或者装饰器的模式，插到egg-cluster的agent进程中。

那我有种预感。app_worker.js 中会不会也是像这个一样，仅仅是实现了master对app_worker的调度，而没有任何其他的业务代码。

### app

果不其然 上来就是这段代码 ：
```
const Application = require(options.framework).Application;
```
不过app_worker.js中多了一些对http和https服务创建的逻辑。

### 总结一下

首先系统通过运行master进程，来创建agent和app进程。
而顺序是
```
/*
 * [startCluster] -> master -> agent_worker -> new [Agent]       -> agentWorkerLoader
 *
*/                         `-> app_worker   -> new [Application] -> appWorkerLoader
```
这个顺序是类似同步调用函数一样的，就是前一个完成后一个才开始的顺序。而实现方式就是通过master的调度。master收到上一个进程的创建成功消息，才会开始fork新的进程。直到最后的app_worker被fork成功。就结束了整个的egg程序启动。

1. master 调度进程
2. agent 不知道是干啥的。之后看看egg的其他模块，看看这个`options.framework`到底传了一个什么进来。
3. app 这个应该就是业务进程，所有的http请求处理啦，数据库读写逻辑，都是在这个进程中。也就是用户开发的代码都在这个进程里面跑

所以，egg在特点是单进程，单线程，只支持单核CPU，不能充分的利用多核CPU服务器的node框架上，利用一些node提供的模块，加上自己的一些封装，编写出了egg-cluster，扩展出了一套多进程，可以充分的利用多核CPU服务器的web-service `egg.js`

如果想学习node的多进程调度，多进程实现。那么阅读egg-cluster源码不失为一个很好的方式。

## 一些小惊喜

### Array.from()

从一个类似数组或可迭代对象中创建一个新的数组实例。

```
const bar = ["a", "b", "c"];
Array.from(bar);
// ["a", "b", "c"]

Array.from('foo');
// ["f", "o", "o"]
```

### Map.keys()

```
const map = new Map();
map.set('key1','key1-content');
map.set('key2','key2-content');
map.keys();
// MapIterator {"key1", "key2"}
Array.from(map.keys());
// ["key1", "key2"]
```

### Array.protype.some()

some() 方法用于检测数组中的元素是否满足指定条件（函数提供）。
some() 方法会依次执行数组的每个元素：

- 如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测。
- 如果没有满足条件的元素，则返回false。

some() 不会对空数组进行检测。
some() 不会改变原始数组。

```
var ages = [3, 10, 18, 20];

function checkAdult(age) {
    return age >= 18;
}

const result = ages.some(checkAdult);
console.log(result);
// true
```

### process.cwd()

process模块用来与当前进程互动，可以通过全局变量process访问，不必使用require命令加载。它是一个EventEmitter对象的实例。
这里的process.cwd()
表示返回运行当前脚本的工作目录的路径

### 轮询调度算法(Round-Robin Scheduling)

简单的说就是一共ABC三个机器做事情，事情来了，先给A，又来个事情，这次给B，又来个事情，这次给C，又来个事情，这次给A，然后这样一直循环ABCABCABC。

- 算法的优点是其简洁性，它无需记录当前所有连接的状态，所以它是一种无状态调度。
- 所以此种均衡算法适合于服务器组中的所有服务器都有相同的软硬件配置并且平均服务请求相对均衡的情况。


#### 权重轮询调度算法(Weighted Round-Robin Scheduling)

简单的说就是一共ABC三台机器，权重分别是3：1：1 那么还是刚才的例子，做事情的顺序就变成AAABCAAABCAAABC。

- 由于权重轮询调度算法考虑到了不同服务器的处理能力，所以这种均衡算法能确保高性能的服务器得到更多的使用率，避免低性能的服务器负载过重。所以，在实际应用中比较常见。


- 轮询调度算法以及权重轮询调度算法的特点是实现起来比较简洁，并且实用。目前几乎所有的负载均衡设备均提供这种功能。

### 无栈式开发

https://enhancer.io/

不需要编写业务代码，只写数据库存储就可以生成后台。没研究过。

### 一些工具包

#### sendmessage

- 进程间发送数据的工具包
- 自己有对接受进程是否存活的判断和错误处理
- 底层也是通过node的事件驱动特性，`event.emit`的方式实现的。

#### is-type-of

- 方便好用的类型判断工具包。

#### graceful-process

- 优雅的进程退出工具。

#### cfork

- 简单的方式fork进程，restart进程。
- egg-cluster对进程的管理主要是通过这个工具来实现。

#### get-ready

- 不是特别理解这是一个什么工具。有机会用用看。


