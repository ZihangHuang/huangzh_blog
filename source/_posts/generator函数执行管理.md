---
title: generator函数执行管理
date: 2019-03-11 10:33:03
categories: 技术
tags: [前端]
---

generator 函数的两种执行管理方式：

### 基于 Thunk 函数

<!--more-->

```javascript
function thunkify(fn) {
    return function() {
        var args = new Array(arguments.length);
        var ctx = this;

        for (var i = 0; i < args.length; ++i) {
            args[i] = arguments[i];
        }

        return function(cb) {
            var called;

            args.push(function() {
                if (called) return; //回调函数只执行一次
                called = true;
                cb.apply(null, arguments);
            });

            try {
                fn.apply(ctx, args);
            } catch (err) {
                cb(err);
            }
        };
    };
}

var readBook = function(fileName, callback) {
    setTimeout(function() {
        try {
            throw new Error("I am die");
        } catch (err) {
            callback(err);
            return;
        }
        callback(null, "aa:" + fileName);
    }, 1000);
};

var readBookThunk = thunkify(readBook);

//或者不借助Thunk函数，手动包装自己的函数
// var readBookThunk = function(fileName) {
//     return function(callback) {
//         try {
//             throw new Error("I am die");
//         } catch (err) {
//             callback(err);
//             return;
//         }
//         setTimeout(function() {
//             callback(null, "aa:" + fileName);
//         }, 1000);
//     };
// };

var gen = function*() {
    try {
        var r1 = yield readBookThunk("/etc/fstab");
        console.log(r1.toString());

        var r2 = yield readBookThunk("/etc/shells");
        console.log(r2.toString());
    } catch (err) {
        console.error("happer err");
    }
};

//执行方式1：手动执行
// var g = gen();

// var r1 = g.next();
// r1.value(function(err, data) {
//     if (err) return g.throw(err);
//     var r2 = g.next(data);

//     r2.value(function(err2, data) {
//         if (err2) return g.throw(err2);
//         g.next(data);
//     });
// });

//执行方式2：自动管理执行
function run(gen) {
    var g = gen();

    function next(err, data) {
        if (err) return g.throw(err);

        var result = g.next(data);
        if (result.done) return;
        result.value(function(err2, data) {
            next(err2, data);
        });
    }

    next();
}

run(gen);
//过1秒输出happer err
```

### 基于 Promise

```javascript
var readFile = function(fileName) {
    return new Promise(function(resolve, reject) {
        setTimeout(function() {
            resolve("aa:" + fileName);
        }, 1000);
    });
};

var gen2 = function*() {
    try {
        // var f1 = yield readFile('/etc/fstab');
        // var f2 = yield readFile('/etc/shells');

        // console.log(f1.toString());
        // console.log(f2.toString());

        var obj = {
            a: yield readFile("/etc/fstab"),
            b: yield readFile("/etc/shells")
        };

        console.log(obj);
    } catch (err) {
        console.error("happer err");
    }
};

function run2(gen) {
    var g = gen();

    function next(err, data) {
        if (err) return g.throw(err);

        var r = g.next(data);
        if (r.done) return;

        r.value
            .then(data => {
                next(null, data);
            })
            .catch(err => {
                next(err);
            });
    }

    next();
}

run2(gen2);
//过2秒输出{ a: 'aa:/etc/fstab', b: 'aa:/etc/shells' }
```
