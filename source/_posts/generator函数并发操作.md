---
title: generator函数并发操作
date: 2019-09-05 12:00:52
categories:
tags:
---
generator函数把异步操作写成同步形式，下一步操作会等待上一步。那如果想并发操作，要怎么写呢？

<!--more-->

我们使用co库来执行generator函数，注意使用的版本是3.1.0，和最新的4.0版本有些不同。
先创建两个异步函数，一个返回Promise，另一个是Thunk函数：

```javascript
var readFile1 = function(fileName) {
    return new Promise(function(resolve, reject) {
        setTimeout(() => {
            resolve(fileName)
        }, 1000)
    })
};

var readFile2 = function(fileName) {
    return function (cb) {
        setTimeout(() => {
            cb(null, fileName)
        }, 1000)
    }
};
```
执行一下，我们先试试readFile1，即Promise：

```javascript
var co = require("co");

function run() {
    return co(function*() {
        try {
            console.time('time')
            //并发的重点是这里
            let a =  readFile1("./a.txt")
            let b = readFile1("./b.txt")
            let info = {
                a: yield a,
                b: yield b
            }
            console.timeEnd('time')
            console.log('res:', info);

        } catch (error) {
            console.error("happen error!");
        }
        return "done";
    })();
}
run()
```

打印出来的执行结果：

```javascript
//time: 1006.001ms
//res: { a: './a.txt', b: './b.txt' }
```

很好，是并发执行。注意，如果你写成以下形式，是不能并发：

```javascript
let info = {
    a: yield readFile1("./a.txt")
    b: yield readFile1("./b.txt")
}
//打印结果：
//time: 2009.424ms
//res: { a: './a.txt', b: './b.txt' }
```


再试试readFile2：

```javascript
function run() {
    return co(function*() {
        try {
            console.time('time')
            //并发的重点是这里
            let a =  readFile2("./a.txt")
            let b = readFile2("./b.txt")
            let info = {
                a: yield a,
                b: yield b
            }
            console.timeEnd('time')
            console.log('res:', info);

        } catch (error) {
            console.error("happen error!");
        }
        return "done";
    })();
}
run()

//打印结果：
//time: 2009.930ms
//res: { a: './a.txt', b: './b.txt' }
```

对不起，并不能并发执行。
那有没有写法可以既支持Promise，又支持Thunk函数呢？
答案是有的：

```javascript
function run() {
    return co(function*() {
        try {
            console.time('time')
            //划重点
            let info = yield {
                a: readFile2("./a.txt"),
                b: readFile2("./b.txt")
            }
            
            console.timeEnd('time')
            console.log('res:', info);

        } catch (error) {
            console.error("happen error!");
        }
        return "done";
    })();
}
run()

//打印结果：
//time: 1004.395ms
//res: { a: './a.txt', b: './b.txt' }
```

如果只用Promise的话，也可以使用Promise.all，只不过返回的是数组：

```javascript
function run() {
    return co(function*() {
        try {
            console.time('time')

            let tasks = [readFile1("./a.txt"), readFile1("./b.txt")]
            let info = yield Promise.all(tasks)

            console.timeEnd('time')
            console.log('res:', info);

        } catch (error) {
            console.error("happen error!");
        }
        return "done";
    })();
}
run()

//打印结果：
//time: 1003.025ms
//res: [ './a.txt', './b.txt' ]
```