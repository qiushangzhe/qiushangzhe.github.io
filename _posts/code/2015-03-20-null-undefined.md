---
layout: post
title: undefined && null
date: 2015-03-20
category: code
tags: [javascript]
---

## undefined和null的区别详解

就拿C++来说，构造函数初始化变量,如果没有初始值的话，可以使用null关键字来赋初值。

```c++
Object(){
    this.name = null;
    this.age = null;
    this.sex = null;
}
```
<!-- more -->
而在JavaScript中，如果想赋初值为空的话，可以使用两种方式。

```javascript
var number = null;
var number = undefined;
```

而这两种写法，并没有什么太大的区别。可以说是等价的。


设计者在设计的时候的思路是：
1. null 从逻辑角度上看，其表示一个空对象指针。
这也是为什么typeof(null)  是一个object 的原因。

    典型用法：
        
        变量赋初值可以为null。
    
    经典场景：
    
        (javascript)
        var a ;
        console.log(a);//undefined;
        a = null;
        console.log(a);//null;
        

2. undefined 声明但是还没初始化。

    经典场景：
        
         (javascript)
        var a ;
        console.log(a);//undefined;
        
### 注意


```javascript
null == undefined //true

null === undefined //false
```

由于==的时候会把他们都转换成false

而在===的时候是判断类型和值是否都相等。





