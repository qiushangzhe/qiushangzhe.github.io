---
layout: post
title: 游戏开发之状态机
date: 2016-12-15 00:00:00
category: talk
tags: [游戏开发,状态机]
---

## 状态机

### 什么是状态机

全称是有限状态机。又称有限状态自动机，简称状态机，是表示有限个状态以及在这些状态之间的转移和动作等行为的数学模型。

### 状态机的特点
- 状态总数是有限的。
- 任意时刻，只处在一种状态中。
- 某种条件下，会从一种状态切换到另一种状态中。
<!-- more -->

### 简单模型：
> 在什么情况下，切换到什么状态，执行什么行为。


#### 状态机实例（词法分析器）
```
function state_machine()
{
    this.state=_1;
    this.resault=[];
    function _1(c)
    {
        if(c!='/')
        {
            this.resault.push(c);
            return _1;
        }
        else
        {
            return _2;
        }    
    }
    function _2(c)
    {
        this.resault.push('/'+c);
        return _1;
    }
    this.change=function(c)
    {
        this.state=this.state(c);
    };
}
var sm=new state_machine();
var queue=("a//sd/jh/ds").split('');

for(var i=0;i<queue.length;i++)
   sm.change(queue[i]);

console.log(sm.resault);
//["a", "//", "s", "d", "/j", "h", "/d", "s"]
```

### 状态机+面向对象思想应用于游戏AI开发

#### （简单AI的实现）狗的行为
```
对象：
    狗

属性：

- 疲劳值（100/100）
- 饥饿值（100/100）
- 生命值（100/100）    

方法：

1. 休息（减少疲劳值）
2. 吃东西（减少饥饿值）
3. 玩耍（增加疲劳和饥饿值）

状态：

1. 疲惫状态 =>（休息）
2. 饥饿状态 =>（吃东西）
3. 活力状态 =>（玩耍）

状态切换条件：

1. 疲劳值为100切换到疲劳状态
2. 饥饿值低于100切换到饥饿状态
3. 其他时候就是活力状态
```

#### 02 lol中野怪AI（简易版本）

```
对象：

	蓝buff巨人

属性：

1. 生命值(1500/1500)
2. 追击定时器(5/5)

方法：

1. 攻击敌人
2. 追逐敌人
3. 恢复生命
4. 返回出生点

状态：

1. 待机状态		==>		方法：无
2. 反击状态		==>		方法：攻击敌人
3. 追击状态		==>		方法：追逐敌人 并 开启追逐定时器
4. 失落状态		==>		方法：恢复生命／返回出生点

状态切换条件：

1. 默认创建以后就是待机状态。
2. 被攻击后切换成反击状态。		
3. 处于反击状态。				
	- 攻击不到敌人（攻击距离不够），切换到追击状态。
4. 追击状态
	- 打到敌人 切换至 反击状态。
	- 追击定时器归零 切换至 失落状态
```

#### 一个对狗的行为模拟的状态机代码

```javascript
//状态参数
var s_map = {
    STATUS_NULL:0,//白痴ing
    STATUS_TIRED:1,//疲惫状态
    STATUS_HUNGER:2,//饥饿状态
    STATUS_HAPPY:3//活力状态
}
var dog = {};
//状态标记位
dog.status = s_map.STATUS_HAPPY;
//属性
dog.tiredNum = 0;
dog.hungerNum = 0;
dog.hp = 100;
//方法
dog.sleep = function(){
    console.log("睡觉呢....");
    this.tiredNum -=20 ;
    this.check();
    console.log(this.tiredNum);
}
dog.eat = function(){
    console.log("吃饭呢....");
    this.hungerNum -=20 ;
    this.check();
    console.log(this.hungerNum);
}
dog.play = function(){
    console.log("到处乱走....");
    this.tiredNum += parseInt(Math.random()*20);
    this.hungerNum += parseInt(Math.random()*20);
    this.check();
}
dog.update = function(){
    switch (this.status) {
        case s_map.STATUS_TIRED:this.sleep();break;
        case s_map.STATUS_HUNGER:this.eat();break;
        case s_map.STATUS_HAPPY:this.play();break;
        default:
    }
}
dog.check = function(){
    if(this.tiredNum > 100) this.status = s_map.STATUS_TIRED;
    else if(this.hungerNum > 100) this.status = s_map.STATUS_HUNGER;
    else if(this.tiredNum < 0 || this.hungerNum < 0)this.status = s_map.STATUS_HAPPY;
}


var doooog = Object.create(dog);
setInterval(function(){
    doooog.update();
},100);

```

#### 状态机+继承 的例子

```
### 基类
	名称：生物
	属性：
		- 饥饿值
		- 疲劳值
	方法：
		- 进食
		- 休息
	状态：
		- 饥饿状态
		- 疲倦状态
	状态切换：
		略。。。    
### 子类001						
	名称：人类
	属性：继承父类
	方法重写：
		- 去厨房做饭吃／点外卖
		- 睡觉／喝杯咖啡
	状态：继承父类
	状态切换：继承父类
### 子类002
	名称：🐶狗🐶
	属性：继承父类
	方法重写：
		- 觅食，看见可能是吃的的东西就吃掉。
		- 倒地就睡。
	状态：继承父类
	状态切换：继承父类
```
