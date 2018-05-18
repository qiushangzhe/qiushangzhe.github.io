---
layout: post
title: gulp学习笔记
date: 2017-05-01
category: code
tags: [gulp]
---

# gulp 

## gulp是什么

Gulp.js 是一个自动化构建工具,开发者可以使用它在项目开发过程中自动执行常见任务

<!-- more -->

## gulp的工作方式

gulp中使用的是node.js中的stream，首先获取到需要的stream，然后可以通过stream的pipe方法把流导入到你想要的地方，比如gulp的插件中，经过插件处理过的流又可以导入其他的插件中，当然也可以把流写入到文件中。

所以gulp是以stream为媒介的，他不需要频繁的生成临时文件，这是使用gulp的一个主要原因。

#### 工作流程

- gulp的工作流程一般是：首先通过gulp.src()方法获得想要处理的文件流
- 然后把文件流通过pipe方法导入到gulp插件中。
- 最后吧经过插件处理的流再通过pipe方法导入到gulp.dest()中，gulp.dest()方法则把流中的内容写入到文件中。

例如
    
    var gulp = require('gulp');
    gulp.src('script/main.js').
    pipe(gulp.dest(dist/yo.js));   

## globs匹配规则

gulp内部使用了node-glob模块来实现其文件的匹配功能。可以使用一些特殊的字符来匹配我们想要的文件。


匹配符 | 说明
-------------|-------------
 * | 匹配文件路径中的一个或者多个字符,   但是不会匹配路径分割字符,   除非路径分割字符在末尾。
 ** | 匹配路径中的0个或多个目录及其子目录  ，需要单独出现，即左右不能有其他的东西了。  如果出现 在末尾，也可以匹配文件。
 ？ |   匹配文件路径中的一个字符 不会匹配路径分割符号
 [...] | 匹配方括号中出现的字符中的任意一个，    当方括号中出现了!或者^,代表方括号中 不出现这些字符。
 !(xx) | 匹配括号中任何一个模式都不匹配的
 ？ | 0或者1次
 +  | 至少一次
 *  | 0次或多次
 @  | 任意模式一次

## 一.gulp.src()

### src是干嘛的？

用于获取一个虚拟文件流，其中包括原始文件路径，文件名，内容等信息。

### 具体语法：
`gulp.src(globs,[,options])`

- globs 是文件匹配模式，类似正则表达式，也可以直接填写某个文件的路径。如果是多个匹配模式的时候，也可以为一个数组。
- options是一个可选参数：
   1. options.buffer   
   `boolen 默认：true`  
    如果这个被设置为了false，那么将会以stream的方式返回file.contents，而不是以文件的buffer的形式返回。 
   2. options.read  
   `boolen 默认值：true`  
    如果是false，file.content会返回null也就是不回去读取文件。
    
    3. options.base
    `string`
    设置输出路径以某个路径的某个组成部分为基础向后拼接。
    请想像一下在一个路径为 client/js/somedir 的目录中，有一个文件叫 somefile.js ：
    
	```javascript
	gulp.src('client/js/**/*.js') // 匹配 'client/js/somedir/somefile.js' 现在 `base` 的值为 `client/js/`
	  .pipe(minify())
	  .pipe(gulp.dest('build'));  // 写入 'build/somedir/somefile.js' 将`client/js/`替换为build
	gulp.src('client/js/**/*.js', { base: 'client' }) // base 的值为 'client'
	  .pipe(minify())
	  .pipe(gulp.dest('build'));  // 写入 'build/js/somedir/somefile.js' 将`client`替换为build
	```

## 二.gulp.watch
### 干啥的
gulp.watch()用来监视文件的变化，当文件发生变化后，我们可以利用它来执行相应的任务，例如文件压缩等

### 怎么使用


```javascript
gulp.watch(glob[, opts], tasks); 
```

1. glob是匹配要监视那些文件。
2. tasks是如果监视到了变化，要执行哪些任务。是一个数组。
3. opts没什么卵用。

### 另一种用法：


```javascript
gulp.watch(glob[, opts, cb]);
```

glob和opts参数与第一种用法相同;

cb参数为一个函数。每当监视的文件发生变化时，就会调用这个函数,并且会给它传入一个对象，该对象包含了文件变化的一些信息，type属性为变化的类型，可以是added,changed,deleted；path属性为发生变化的文件的路径。


```javascript
gulp.watch('js/**/*.js', function(event){
    console.log(event.type); //变化类型 added为新增,deleted为删除，changed为改变 
    console.log(event.path); //变化的文件的路径
}); 
```

## 三.gulp.task
### 是啥：
gulp.task方法用来定义任务，内部使用的是Orchestrator(用于排序、执行任务和最大并发依赖关系的模块)

### 怎么用：

```javascript
gulp.task(name[, deps], fn)
```

- name 为任务名；

- deps 是当前定义的任务需要依赖的其他任务，为一个数组。当前定义的任务会在所有依赖的任务执行完毕后才开始执行。如果没有依赖，则可省略这个参数；

- fn 为任务函数，我们把任务要执行的代码都写在里面。该参数也是可选的。

当你定义一个简单的任务时，需要传入任务名字和执行函数两个属性。



你也可以定义一个在gulp开始运行时候默认执行的任务，并将这个任务命名为“default”：


```javascript
gulp.task('default', function () {
   // Your default task
});
```

## 四.gulp.run
### 什么是run
gulp.run()表示要执行的任务。可能会使用单个参数的形式传递多个任务

### 如何使用


```javascript
gulp.task('end',function(){
gulp.run('task1','task3','task2');
});
```

注意：任务是尽可能多的并行执行的，并且可能不会按照指定的顺序运行。

## 五.gulp.dest
### 干嘛的？
写文件用的。

### 用法？

`gulp.dest(path[,options])`


```
path为写入文件的路径；

options为一个可选的参数对象，以下为选项参数：
```

1. option.cwd()
    
    类型String 默认值process.cwd()
    
    输出目录的cwd参数，只在所给路径是相对路径的时候有效。

2. options.mode
    
    类型String 默认值：0777

    八进制权限字符，用于定义所有在输出目录中所创建目录的权限。
    
    
### 补充   
下面再说说生成的文件路径与我们给_gulp.dest()_方法传入的路径参数之间的关系。

_gulp.dest(path)_生成的文件路径是我们传入的_path_参数后面再加上_gulp.src()_中有通配符开始出现的那部分路径。例如：

```javascript
var gulp = reruire('gulp');
//有通配符开始出现的那部分路径为 **/*.js
gulp.src('script/**/*.js')
    .pipe(gulp.dest('dist')); //最后生成的文件路径为 dist/**/*.js
//如果 **/*.js 匹配到的文件为 jquery/jquery.js ,则生成的文件路径为 dist/jquery/jquery.js
```
　　
用gulp.dest()把文件流写入文件后，文件流仍然可以继续使用。

### 常用插件
## browser-sync

官方给出的解释：


```
省时的浏览器同步测试工具
```

## 为什么使用它


```
Browsersync能让浏览器实时、快速响应您的文件更改（html、js、css、sass、less等）并自动刷新页面。  
更重要的是 Browsersync可以同时在PC、平板、手机等设备下进项调试。
```

## 如何使用

结合Gulp自动化构建工具进行使用。

步骤如下：

1. npm安装 browser-sync
2. 在gulpfile.js中使用：
    
```javascript
var gulp        = require('gulp');
var browserSync = require('browser-sync').create();

// 静态服务器
gulp.task('browser-sync', function() {
    browserSync.init({
        server: {
            baseDir: "./"
        }
    });
});
```

3. 例子（更改js文件浏览器刷新）


```javascript
// 处理完JS文件后返回流
gulp.task('js', function () {
    return gulp.src('js/*js')
        .pipe(browserify())
        .pipe(uglify())
        .pipe(gulp.dest('dist/js'));
});

// 创建一个任务确保JS任务完成之前能够继续响应
// 浏览器重载
gulp.task('js-watch', ['js'], browserSync.reload);

// 使用默认任务启动Browsersync，监听JS文件
gulp.task('serve', ['js'], function () {

    // 从这个项目的根目录启动服务器
    browserSync.init({
        server: {
            baseDir: "./"
        }
    });

    // 添加 browserSync.reload 到任务队列里
    // 所有的浏览器重载后任务完成。
    gulp.watch("js/*.js", ['js-watch']);
});
```



